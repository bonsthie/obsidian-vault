#glibc #Breakdown #thread #pthread

```c
  const struct pthread_attr *iattr = (struct pthread_attr *) attr;
  union pthread_attr_transparent default_attr;
  bool destroy_default_attr = false;
  bool c11 = (attr == ATTR_C11_THREAD);
  if (iattr == NULL || c11)
    {
      int ret = __pthread_getattr_default_np (&default_attr.external);
      if (ret != 0)
        return ret;
      destroy_default_attr = true;
      iattr = &default_attr.internal;
    }

```
