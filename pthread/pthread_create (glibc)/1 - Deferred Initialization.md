#glibc #Breakdown #thread #pthread

```c
    /* Avoid a data race in the multi-threaded case, and call the
     deferred initialization only once.  */
  if (__libc_single_threaded_internal)
    {
      late_init ();
      __libc_single_threaded_internal = 0;
      /* __libc_single_threaded can be accessed through copy relocations, so
         it requires to update the external copy.  */
      __libc_single_threaded = 0;
    }

```
