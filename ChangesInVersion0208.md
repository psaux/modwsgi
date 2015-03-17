#summary Changes in mod\_wsgi version 2.8

_**If you are using mod\_wsgi, please consider making a
[donation](HowToContributeBack.md).**_

# Changes In Version 2.8 #

Version 2.8 of mod\_wsgi can be obtained from:

> http://modwsgi.googlecode.com/files/mod_wsgi-2.8.tar.gz

## Bug Fixes ##

1. Ensure that any compiler flags supplied via the CFLAGS environment variable
when running 'configure' script are prefixed by '-Wc,' before being passed to
'apxs' to build module. Without this 'apxs' will incorrectly interpret the
compiler options. For more details see:

> http://code.google.com/p/modwsgi/issues/detail?id=166