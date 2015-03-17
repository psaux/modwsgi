#summary Changes in mod\_wsgi version 1.6.

_**If you are using mod\_wsgi, please consider making a
[donation](HowToContributeBack.md).**_

# Changes In Version 1.6 #

Version 1.6 of mod\_wsgi can be obtained from:

> http://modwsgi.googlecode.com/files/mod_wsgi-1.6.tar.gz

**Note that this is a quick followup to version 1.5 of mod\_wsgi to rectify
significant problem introduced by that release. You should therefore also
refer to [change notes](ChangesInVersion0105.md) for version 1.5 of mod\_wsgi.**

## Bug Fixes ##

1. Fixed problem introduced in version 1.5 of mod\_wsgi whereby use of
daemon mode would cause CGI scripts to fail.

It is quite possible that the bug could also have caused failures with other
Apache modules that relied on registering of cleanup functions against
Apache configuration memory pool.

For details see:

> http://groups.google.com/group/modwsgi/browse_frm/thread/79a86f8faffe7dcf