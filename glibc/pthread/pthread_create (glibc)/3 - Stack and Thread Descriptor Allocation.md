#glibc #Breakdown #thread #pthread

```c
  struct pthread *pd = NULL;
  int err = allocate_stack (iattr, &pd, &stackaddr, &stacksize);
  int retval = 0;

  if (__glibc_unlikely (err != 0))
    /* Something went wrong.  Maybe a parameter of the attributes is
       invalid or we could not allocate memory.  Note we have to
       translate error codes.  */
    {
      retval = err == ENOMEM ? EAGAIN : err;
      goto out;
    }

```