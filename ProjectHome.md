## What Is mod\_wsgi? ##

The aim of mod\_wsgi is to implement a simple to use [Apache](http://httpd.apache.org) module which can host any [Python](http://www.python.org) application which supports the Python [WSGI](http://www.wsgi.org) interface. The module would be suitable for use in hosting high performance production web sites, as well as your average self managed personal sites running on web hosting services.

## Security Issues ##

Due to security issues in versions of mod\_wsgi up to and including version 3.4, it is recommended that version 3.5 or later be used. See the release notes for the respective versions:

  * [Changes in Version 3.4](ChangesInVersion0304.md)
  * [Changes in Version 3.5](ChangesInVersion0305.md)

## This Site is Moving! ##

Note that this site is going to be progressively moved elsewhere.

Source code is now being hosted on GitHub at:

  * https://github.com/GrahamDumpleton/mod_wsgi

Downloads are available at:

  * https://github.com/GrahamDumpleton/mod_wsgi/releases

and the new issue tracker at:

  * https://github.com/GrahamDumpleton/mod_wsgi/issues

Documentation will eventually be relocated to ReadTheDocs as it is cleaned up:

  * http://modwsgi.readthedocs.org/

## Modes of Operation ##

When hosting WSGI applications using mod\_wsgi, one of two primary modes of operation can be used. In 'embedded' mode, mod\_wsgi works in a similar way to mod\_python in that the Python application code will be executed within the context of the normal Apache child processes. WSGI applications when run in this mode will therefore share the same processes as other Apache hosted applications using Apache modules for PHP and Perl.

An alternate mode of operation available with Apache 2.X on UNIX is 'daemon' mode. This mode operates in similar ways to FASTCGI/SCGI solutions, whereby distinct processes can be dedicated to run a WSGI application. Unlike FASTCGI/SCGI solutions however, neither a separate process supervisor or WSGI adapter is needed when implementing the WSGI application and everything is handled automatically by mod\_wsgi.

Because the WSGI applications in daemon mode are being run in their own processes, the impact on the normal Apache child processes used to serve up static files and host applications using Apache modules for PHP, Perl or some other language is much reduced. Daemon processes may if required also be run as a distinct user ensuring that WSGI applications cannot interfere with each other or access information they shouldn't be able to.

Note that although mod\_wsgi has features similar to FASTCGI/SCGI solutions, it isn't intended to be a replacement for those hosting mechanisms in all situations for Python web hosting. Specifically, mod\_wsgi is not designed for nor intended for use in over allocated shared mass virtual hosting setups for different users on a single Apache instance. For such mass virtual hosting arrangements, FASTCGI in particular would still be the preferred choice in most situations.

## Server Performance ##

The mod\_wsgi module is written in C code directly against the internal Apache and Python application programming interfaces. As such, for hosting WSGI applications in conjunction with Apache it has a lower memory overhead and performs better than existing WSGI adapters for mod\_python or alternative FASTCGI/SCGI/CGI or proxy based solutions.

Although embedded mode can technically perform better, daemon mode would generally be the safest choice to use. This is because to get the best performance out of embedded mode you must tune the Apache MPM settings, which in their default settings are biased towards serving of static media and hosting of PHP applications. If the Apache MPM settings are not set appropriately for the type of application being hosted, then worse performance can be seen rather than better performance.

Thus, unless you are adept at configuring Apache, always use daemon mode when available. Overall, for large Python web applications you wouldn't normally expect to see any significant difference between daemon mode and embedded mode, as the bottlenecks are going to be in the Python web application or any database access.

## Supported Applications ##

As mod\_wsgi supports the WSGI interface specification, any Python web framework or application which is compatible with the WSGI interface specification should be able to be hosted on top of mod\_wsgi.

Major Python web frameworks and toolkits which are known to work include CherryPy, Django, Pylons, TurboGears, Pyramid, web.py, Werkzeug, Web2Py and Zope. Major Python web applications which are known to work include MoinMoin, PyBlosxom and Trac.

## System Requirements ##

The mod\_wsgi package can be compiled for and used with either Apache 1.3, 2.0, 2.2 or 2.4 on UNIX systems (including Linux), as well as Windows. Either the single threaded 'prefork' or multithreaded 'worker' Apache MPMs can be used when running on UNIX.

Daemon mode of mod\_wsgi will however only be available on Apache 2.X running on UNIX, and only when the Apache runtime library underlying Apache has been compiled with support for threading.

Python 2.X is required with a minimum of Python 2.3. The Python installation being used must have been compiled with support for threading.

The original WSGI specification (PEP 333) only supports Python 2.X. There is support in mod\_wsgi for Python 3.X which is based on guesses as to what the WSGI specification would look like for Python 3.X. The new WSGI specification (PEP 3333) has finally now been accepted and although some tweaks need to be made to mod\_wsgi to make it more strict, if you write your Python 3 WSGI application according to PEP 3333, it will work perfectly fine on mod\_wsgi. If you wish to experiment with Python 3.X, you will need to use Python 3.1 or later.

## Getting Started ##

Ensure that you first read the documentation covering [installation and configuration](http://code.google.com/p/modwsgi/wiki/InstallationInstructions). Additional [guidelines](http://code.google.com/p/modwsgi/wiki/DeveloperGuidelines) are also provided for developers working with applications on top of mod\_wsgi so as to get the most out of mod\_wsgi, but also to assist in debugging problems or issues that arise with your application.

If you don't understand a problem you are having with your application, or believe you have found a problem with mod\_wsgi, raise the issue on the [mod\_wsgi user group](http://groups.google.com/group/modwsgi). Make sure you read the guide on [where to get help](http://code.google.com/p/modwsgi/wiki/WhereToGetHelp) first though.

## Contributing ##

Developing software as Open Source is more often than not a thankless task. If you are using mod\_wsgi, please consider making a [donation](HowToContributeBack.md). By giving a donation you help to ensure that mod\_wsgi will continue to be developed and the documentation further improved.

Also please note that contrary to what is being reported in some forums, this project is in no way affiliated with Google or funded by Google (or any other company) in any way. The only connection with Google is that the project source code and documentation is hosted on the free and public Google code hosting site. All development on this project is done at my own cost in personal time.

Enjoy.

**Graham Dumpleton**