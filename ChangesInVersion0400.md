#summary Changes in mod\_wsgi version 4.0.

_**If you are using mod\_wsgi, please consider making a
[donation](HowToContributeBack.md).**_

# Changes In Version 4.0 #

Working version of mod\_wsgi 4.0 can currently only be obtained by checking
it out from the source code repository. The Mercurial command you should
use is:

```
hg clone http://modwsgi.googlecode.com/hg/ mod_wsgi-4.X
```

Note that mod\_wsgi 4.0 was originally derived from mod\_wsgi 3.1. It has
though all changes from later releases in the 3.X branch. Thus also see:

1. Changes in version [3.2](ChangesInVersion0302.md) of mod\_wsgi.

2. Changes in version [3.3](ChangesInVersion0303.md) of mod\_wsgi.

3. Changes in version [3.4](ChangesInVersion0304.md) of mod\_wsgi.

4. Changes in version [3.5](ChangesInVersion0305.md) of mod\_wsgi.

## Features Removed ##

1. No longer support the use of mod\_python in conjunction with mod\_wsgi.
When this is attempted an error is forced and Apache will not be able to
start. An error message is logged in main Apache error log.

2. No longer support the use of Apache 1.3. Minimum requirement is now
Apache 2.0.

## New Features ##

1. Add a graceful-timeout option to WSGIDaemonProcess. This option is
applied in a number of circumstances.

When maximum-requests and this option are used together, when maximum
requests is reached, rather than immediately shutdown, potentially
interupting active requests if they don't finished with shutdown timeout,
can specify a separate graceful shutdown period. If the all requests are
completed within this time frame then will shutdown immediately, otherwise
normal forced shutdown kicks in. In some respects this is just allowing a
separate shutdown timeout on cases where requests could be interrupted and
could avoid it if possible.

When cpu-time-limit and this option are used together, when CPU time limit
reached, rather than immediately shutdown, potentially interupting active
requests if they don't finished with shutdown timeout, can specify a
separate graceful shutdown period.

2. Add potentially graceful process restart option for daemon processes
when sent a graceful restart signal. Signal is usually SIGUSR1 but is
platform dependent as using same signal as Apache would use. If the
graceful-timeout option had been provided to WSGIDaemonProcess, then the
process will attempt graceful shutdown first based on the that timeout,
otherwise normal shutdown procedure used as if received a SIGTERM.