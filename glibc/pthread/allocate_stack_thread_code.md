
```c
/* Returns a usable stack for a new thread either by allocating a
   new stack or reusing a cached stack of sufficient size.
   ATTR must be non-NULL and point to a valid pthread_attr.
   PDP must be non-NULL.  */
static int
allocate_stack (const struct pthread_attr *attr, struct pthread **pdp,
		void **stack, size_t *stacksize)
{
  struct pthread *pd;
  size_t size;
  size_t pagesize_m1 = __getpagesize () - 1;
  size_t tls_static_size_for_stack = __nptl_tls_static_size_for_stack ();
  size_t tls_static_align_m1 = GLRO (dl_tls_static_align) - 1;

  assert (powerof2 (pagesize_m1 + 1));
  assert (TCB_ALIGNMENT >= STACK_ALIGN);

  /* Get the stack size from the attribute if it is set.  Otherwise we
     use the default we determined at start time.  */
  if (attr->stacksize != 0)
    size = attr->stacksize;
  else
    {
      lll_lock (__default_pthread_attr_lock, LLL_PRIVATE);
      size = __default_pthread_attr.internal.stacksize;
      lll_unlock (__default_pthread_attr_lock, LLL_PRIVATE);
    }

  /* Get memory for the stack.  */
  if (__glibc_unlikely (attr->flags & ATTR_FLAG_STACKADDR))
    {
      uintptr_t adj;
      char *stackaddr = (char *) attr->stackaddr;

      /* Assume the same layout as the _STACK_GROWS_DOWN case, with struct
	 pthread at the top of the stack block.  Later we adjust the guard
	 location and stack address to match the _STACK_GROWS_UP case.  */
      if (_STACK_GROWS_UP)
	stackaddr += attr->stacksize;

      /* If the user also specified the size of the stack make sure it
	 is large enough.  */
      if (attr->stacksize != 0
	  && attr->stacksize < (tls_static_size_for_stack
				+ MINIMAL_REST_STACK))
	return EINVAL;

      /* Adjust stack size for alignment of the TLS block.  */
#if TLS_TCB_AT_TP
      adj = ((uintptr_t) stackaddr - TLS_TCB_SIZE)
	    & tls_static_align_m1;
      assert (size > adj + TLS_TCB_SIZE);
#elif TLS_DTV_AT_TP
      adj = ((uintptr_t) stackaddr - tls_static_size_for_stack)
	    & tls_static_align_m1;
      assert (size > adj);
#endif

      /* The user provided some memory.  Let's hope it matches the
	 size...  We do not allocate guard pages if the user provided
	 the stack.  It is the user's responsibility to do this if it
	 is wanted.  */
#if TLS_TCB_AT_TP
      pd = (struct pthread *) ((uintptr_t) stackaddr
			       - TLS_TCB_SIZE - adj);
#elif TLS_DTV_AT_TP
      pd = (struct pthread *) (((uintptr_t) stackaddr
				- tls_static_size_for_stack - adj)
			       - TLS_PRE_TCB_SIZE);
#endif

      /* The user provided stack memory needs to be cleared.  */
      memset (pd, '\0', sizeof (struct pthread));

      /* The first TSD block is included in the TCB.  */
      pd->specific[0] = pd->specific_1stblock;

      /* Remember the stack-related values.  */
      pd->stackblock = (char *) stackaddr - size;
      pd->stackblock_size = size;

      /* This is a user-provided stack.  It will not be queued in the
	 stack cache nor will the memory (except the TLS memory) be freed.  */
      pd->user_stack = true;

      /* This is at least the second thread.  */
      pd->header.multiple_threads = 1;

#ifdef NEED_DL_SYSINFO
      SETUP_THREAD_SYSINFO (pd);
#endif

      /* Don't allow setxid until cloned.  */
      pd->setxid_futex = -1;

      /* Allocate the DTV for this thread.  */
      if (_dl_allocate_tls (TLS_TPADJ (pd)) == NULL)
	{
	  /* Something went wrong.  */
	  assert (errno == ENOMEM);
	  return errno;
	}


      /* Prepare to modify global data.  */
      lll_lock (GL (dl_stack_cache_lock), LLL_PRIVATE);

      /* And add to the list of stacks in use.  */
      list_add (&pd->list, &GL (dl_stack_user));

      lll_unlock (GL (dl_stack_cache_lock), LLL_PRIVATE);
    }
  else
    {
      /* Allocate some anonymous memory.  If possible use the cache.  */
      size_t guardsize;
      size_t reported_guardsize;
      size_t reqsize;
      void *mem;
      const int prot = (PROT_READ | PROT_WRITE
			| ((GL(dl_stack_flags) & PF_X) ? PROT_EXEC : 0));

      /* Adjust the stack size for alignment.  */
      size &= ~tls_static_align_m1;
      assert (size != 0);

      /* Make sure the size of the stack is enough for the guard and
	 eventually the thread descriptor.  On some targets there is
	 a minimum guard size requirement, ARCH_MIN_GUARD_SIZE, so
	 internally enforce it (unless the guard was disabled), but
	 report the original guard size for backward compatibility:
	 before POSIX 2008 the guardsize was specified to be one page
	 by default which is observable via pthread_attr_getguardsize
	 and pthread_getattr_np.  */
      guardsize = (attr->guardsize + pagesize_m1) & ~pagesize_m1;
      reported_guardsize = guardsize;
      if (guardsize > 0 && guardsize < ARCH_MIN_GUARD_SIZE)
	guardsize = ARCH_MIN_GUARD_SIZE;
      if (guardsize < attr->guardsize || size + guardsize < guardsize)
	/* Arithmetic overflow.  */
	return EINVAL;
      size += guardsize;
      if (__builtin_expect (size < ((guardsize + tls_static_size_for_stack
				     + MINIMAL_REST_STACK + pagesize_m1)
				    & ~pagesize_m1),
			    0))
	/* The stack is too small (or the guard too large).  */
	return EINVAL;

      /* Try to get a stack from the cache.  */
      reqsize = size;
      pd = get_cached_stack (&size, &mem);
      if (pd == NULL)
	{
	  /* If a guard page is required, avoid committing memory by first
	     allocate with PROT_NONE and then reserve with required permission
	     excluding the guard page.  */
	  mem = __mmap (NULL, size, (guardsize == 0) ? prot : PROT_NONE,
			MAP_PRIVATE | MAP_ANONYMOUS | MAP_STACK, -1, 0);

	  if (__glibc_unlikely (mem == MAP_FAILED))
	    return errno;

	  /* Do madvise in case the tunable glibc.pthread.stack_hugetlb is
	     set to 0, disabling hugetlb.  */
	  if (__glibc_unlikely (__nptl_stack_hugetlb == 0)
	      && __madvise (mem, size, MADV_NOHUGEPAGE) != 0)
	    return errno;

	  /* SIZE is guaranteed to be greater than zero.
	     So we can never get a null pointer back from mmap.  */
	  assert (mem != NULL);

	  /* Place the thread descriptor at the end of the stack.  */
#if TLS_TCB_AT_TP
	  pd = (struct pthread *) ((((uintptr_t) mem + size)
				    - TLS_TCB_SIZE)
				   & ~tls_static_align_m1);
#elif TLS_DTV_AT_TP
	  pd = (struct pthread *) ((((uintptr_t) mem + size
				    - tls_static_size_for_stack)
				    & ~tls_static_align_m1)
				   - TLS_PRE_TCB_SIZE);
#endif

	  /* Now mprotect the required region excluding the guard area.  */
	  if (__glibc_likely (guardsize > 0))
	    {
	      char *guard = guard_position (mem, size, guardsize, pd,
					    pagesize_m1);
	      if (setup_stack_prot (mem, size, guard, guardsize, prot) != 0)
		{
		  __munmap (mem, size);
		  return errno;
		}
	    }

	  /* Remember the stack-related values.  */
	  pd->stackblock = mem;
	  pd->stackblock_size = size;
	  /* Update guardsize for newly allocated guardsize to avoid
	     an mprotect in guard resize below.  */
	  pd->guardsize = guardsize;

	  /* We allocated the first block thread-specific data array.
	     This address will not change for the lifetime of this
	     descriptor.  */
	  pd->specific[0] = pd->specific_1stblock;

	  /* This is at least the second thread.  */
	  pd->header.multiple_threads = 1;

#ifdef NEED_DL_SYSINFO
	  SETUP_THREAD_SYSINFO (pd);
#endif

	  /* Don't allow setxid until cloned.  */
	  pd->setxid_futex = -1;

	  /* Allocate the DTV for this thread.  */
	  if (_dl_allocate_tls (TLS_TPADJ (pd)) == NULL)
	    {
	      /* Something went wrong.  */
	      assert (errno == ENOMEM);

	      /* Free the stack memory we just allocated.  */
	      (void) __munmap (mem, size);

	      return errno;
	    }


	  /* Prepare to modify global data.  */
	  lll_lock (GL (dl_stack_cache_lock), LLL_PRIVATE);

	  /* And add to the list of stacks in use.  */
	  __nptl_stack_list_add (&pd->list, &GL (dl_stack_used));

	  lll_unlock (GL (dl_stack_cache_lock), LLL_PRIVATE);


	  /* There might have been a race.  Another thread might have
	     caused the stacks to get exec permission while this new
	     stack was prepared.  Detect if this was possible and
	     change the permission if necessary.  */
	  if (__builtin_expect ((GL(dl_stack_flags) & PF_X) != 0
				&& (prot & PROT_EXEC) == 0, 0))
	    {
	      int err = __nptl_change_stack_perm (pd);
	      if (err != 0)
		{
		  /* Free the stack memory we just allocated.  */
		  (void) __munmap (mem, size);

		  return err;
		}
	    }


	  /* Note that all of the stack and the thread descriptor is
	     zeroed.  This means we do not have to initialize fields
	     with initial value zero.  This is specifically true for
	     the 'tid' field which is always set back to zero once the
	     stack is not used anymore and for the 'guardsize' field
	     which will be read next.  */
	}

      /* Create or resize the guard area if necessary.  */
      if (__glibc_unlikely (guardsize > pd->guardsize))
	{
	  char *guard = guard_position (mem, size, guardsize, pd,
					pagesize_m1);
	  if (__mprotect (guard, guardsize, PROT_NONE) != 0)
	    {
	    mprot_error:
	      lll_lock (GL (dl_stack_cache_lock), LLL_PRIVATE);

	      /* Remove the thread from the list.  */
	      __nptl_stack_list_del (&pd->list);

	      lll_unlock (GL (dl_stack_cache_lock), LLL_PRIVATE);

	      /* Get rid of the TLS block we allocated.  */
	      _dl_deallocate_tls (TLS_TPADJ (pd), false);

	      /* Free the stack memory regardless of whether the size
		 of the cache is over the limit or not.  If this piece
		 of memory caused problems we better do not use it
		 anymore.  Uh, and we ignore possible errors.  There
		 is nothing we could do.  */
	      (void) __munmap (mem, size);

	      return errno;
	    }

	  pd->guardsize = guardsize;
	}
      else if (__builtin_expect (pd->guardsize - guardsize > size - reqsize,
				 0))
	{
	  /* The old guard area is too large.  */

#if _STACK_GROWS_DOWN
	  if (__mprotect ((char *) mem + guardsize, pd->guardsize - guardsize,
			prot) != 0)
	    goto mprot_error;
#elif _STACK_GROWS_UP
         char *new_guard = (char *)(((uintptr_t) pd - guardsize)
                                    & ~pagesize_m1);
         char *old_guard = (char *)(((uintptr_t) pd - pd->guardsize)
                                    & ~pagesize_m1);
         /* The guard size difference might be > 0, but once rounded
            to the nearest page the size difference might be zero.  */
         if (new_guard > old_guard
             && __mprotect (old_guard, new_guard - old_guard, prot) != 0)
	    goto mprot_error;
#endif

	  pd->guardsize = guardsize;
	}
      /* The pthread_getattr_np() calls need to get passed the size
	 requested in the attribute, regardless of how large the
	 actually used guardsize is.  */
      pd->reported_guardsize = reported_guardsize;
    }

  /* Initialize the lock.  We have to do this unconditionally since the
     stillborn thread could be canceled while the lock is taken.  */
  pd->lock = LLL_LOCK_INITIALIZER;

  /* The robust mutex lists also need to be initialized
     unconditionally because the cleanup for the previous stack owner
     might have happened in the kernel.  */
  pd->robust_head.futex_offset = (offsetof (pthread_mutex_t, __data.__lock)
				  - offsetof (pthread_mutex_t,
					      __data.__list.__next));
  pd->robust_head.list_op_pending = NULL;
#if __PTHREAD_MUTEX_HAVE_PREV
  pd->robust_prev = &pd->robust_head;
#endif
  pd->robust_head.list = &pd->robust_head;

  /* We place the thread descriptor at the end of the stack.  */
  *pdp = pd;

  void *stacktop;

#if TLS_TCB_AT_TP
  /* The stack begins before the TCB and the static TLS block.  */
  stacktop = ((char *) (pd + 1) - tls_static_size_for_stack);
#elif TLS_DTV_AT_TP
  stacktop = (char *) (pd - 1);
#endif

  *stacksize = stacktop - pd->stackblock;
  *stack = pd->stackblock;

  return 0;
}

```

