﻿#summary Common mod\_wsgi installation problems.

_**If you are using mod\_wsgi, please consider making a
[donation](HowToContributeBack.md).**_

# Installation Issues #

Although mod\_wsgi is not a large package in itself, it depends on both
Apache and Python to get it compiled and installed. Because Apache and
Python are complicated systems in their own right, various problems can
come up during installation of mod\_wsgi. These problems can arise for
various reasons, including an incomplete or suboptimal Python installation
or presence of multiple Python versions.

The purpose of this document is to capture all the known problems that can
arise regarding installation, including workarounds if available.

If you are having a problem which doesn't seem to be covered by this
document, also make sure you see:

  * [Configuration Issues](ConfigurationIssues.md)
  * [Application Issues](ApplicationIssues.md)

## Missing Python Header Files ##

In order to compile mod\_wsgi from source code you must have installed the
full Python distribution, including header files. On a Linux distribution
where binary Python packages are split into a runtime package and a
developer package, the developer package is often not installed by default.
This means that you will be missing the header files required to compile
mod\_wsgi from source code. An example of the error messages you will see
if the developer package is not installed are:

```
mod_wsgi.c:113:20: error: Python.h: No such file or directory
mod_wsgi.c:114:21: error: compile.h: No such file or directory
mod_wsgi.c:115:18: error: node.h: No such file or directory
mod_wsgi.c:116:20: error: osdefs.h: No such file or directory
mod_wsgi.c:119:2: error: #error Sorry, mod_wsgi requires at least Python 2.3.0.
mod_wsgi.c:123:2: error: #error Sorry, mod_wsgi requires that Python supporting thread.
```

To remedy the problem, install the developer package for Python
corresponding to the Python runtime package you have installed. What the
name of the developer package is can vary from one Linux distribution to
another. Normally it has the same name as the Python runtime package with
'-dev' appended to the package name. You will need to lookup up list of
available packages in your packaging system to determine actual name of
package to install.

## Lack Of Python Shared Library ##

In the optimal case, when mod\_wsgi is compiled the resulting Apache module
should be less than 250 Kbytes in size. If this is not the case and the
module is over 1MB in size, it indicates that the version of Python being
used was not originally configured so as to produce a Python shared library
and is instead only producing a static library.

Although the existance of only a static library for Python doesn't normally
cause compilation of mod\_wsgi to fail, it does mean that when 'libtool' is
used to generate the mod\_wsgi Apache module, that it has to embed the
actual static library objects into the Apache module instead of it being
used as a shared library.

The consequences of this are that when the mod\_wsgi Apache module is loaded
by Apache, the operating system dynamic linker has to perform address
relocations on the Python library component of the mod\_wsgi Apache module.
Because these relocations require memory to be modified, the full Python
library then becomes private memory to the process and not shared.

On a Linux system this need to perform the address relocations at runtime
will immediately cause each Apache child process to bloat out in size by
between 1 and 2MB. On a Solaris system, depending on which compiler is
being used and which options, the amount of additional memory used can be
5MB or more.

To determine whether the compiled mod\_wsgi module is making use of a
shared library for Python, many UNIX systems provide the 'ldd'
program. The output from running this on the 'mod\_wsgi.so' file would
be something like:

```
$ ldd mod_wsgi.so
 linux-vdso.so.1 =>  (0x00007fffeb3fe000)
 libpython2.5.so.1.0 => /usr/local/lib/libpython2.5.so.1.0 (0x00002adebf94d000)
 libpthread.so.0 => /lib/libpthread.so.0 (0x00002adebfcba000)
 libdl.so.2 => /lib/libdl.so.2 (0x00002adebfed6000)
 libutil.so.1 => /lib/libutil.so.1 (0x00002adec00da000)
 libc.so.6 => /lib/libc.so.6 (0x00002adec02dd000)
 libm.so.6 => /lib/libm.so.6 (0x00002adec0635000)
 /lib64/ld-linux-x86-64.so.2 (0x0000555555554000)
```

Note how there is a dependency listed on the '.so' file for Python. If
this is not present then mod\_wsgi is using a static Python library.

Although mod\_wsgi will still work when compiled against a version of Python
which only provides a static library, you are highly encouraged to ensure
that your Python installation has been configured and compiled with the
'--enable-shared' option to enable the production and use of a shared
library for Python.

If rebuilding Python to generate a shared library, do make sure that the
Python shared library, or a symlink to it appears in the Python 'config'
directory of your Python installation. If the shared library doesn't appear
here next to the static version of the library, 'libtool' will not be able
to find it and will still use the static version of the library. It is
understood that the Python build process may not actually do this, so you
may have to do it by hand.

To check, go to the Python 'config' directory of your Python installation
and do a directory listing.

```
$ ls -las

   4 drwxr-sr-x  2 root staff    4096 2007-11-29 23:26 .
  20 drwxr-sr-x 21 root staff   20480 2007-11-29 23:26 ..
   4 -rw-r--r--  1 root staff    2078 2007-11-29 23:26 config.c
   4 -rw-r--r--  1 root staff    1446 2007-11-29 23:26 config.c.in
   8 -rwxr-xr-x  1 root staff    7122 2007-11-29 23:26 install-sh
7664 -rw-r--r--  1 root staff 7833936 2007-11-29 23:26 libpython2.5.a
  40 -rw-r--r--  1 root staff   38327 2007-11-29 23:26 Makefile
   8 -rwxr-xr-x  1 root staff    7430 2007-11-29 23:26 makesetup
   8 -rw-r--r--  1 root staff    6456 2007-11-29 23:26 python.o
  20 -rw-r--r--  1 root staff   17862 2007-11-29 23:26 Setup
   4 -rw-r--r--  1 root staff     368 2007-11-29 23:26 Setup.config
   4 -rw-r--r--  1 root staff      41 2007-11-29 23:26 Setup.local
```

If you only see a '.a' file for Python library, then either Python wasn't
installed with the shared library, or the shared library was placed
elsewhere. What appears to normally happen is that the shared library is
actually placed in the 'lib' directory two levels above the Python 'config'
directory. In that case you need to create a symlink in the 'config'
directory to where the shared library is actually installed.

```
$ ln -s ../../libpython2.5.so .
```

Apart from the additional memory consumption when using a static library,
it is also preferable that a shared library be used where it is possible
that you will upgrade your Python installation to a newer patch revision.
This is because if you upgrade Python to a newer patch revision but do
not recompile mod\_wsgi, mod\_wsgi will still incorporate the older static
Python library and will not pick up any changes from the newer version
of Python. This will result in undefined behaviour as the Python library
code may not match up with the Python code modules or external modules
in the Python installation. If a Python shared library is used, this will
not be a problem.

## Multiple Python Versions ##

Where there are multiple versions of Python installed on a system and it is
necessary to ensure that a specific version is used, the '--with-python'
option can be supplied to 'configure' when installing mod\_wsgi.

```
./configure --with-python=/usr/local/bin/python2.5
```

This may be necessary where for example the default Python version supplied
with the system is an older version of Python. More specifically, it would
be required where it isn't possible to replace the older version of Python
outright due to operating system management scripts being dependent on the
older version of Python and not working with newer versions of Python.

Where multiple versions of Python are present and are installed under the
same directory, this should generally be all that is required. If however
the newer version of Python you wish to use is in a different location, for
example under '/usr/local', it is possible that when Apache is started that
it will not be able find the Python library files for the version of Python
you wish to use.

This can occur because the Python library when initialised determines where
the Python installation resides by looking through directories specified in
the 'PATH' environment variable for the 'python' executable and using that
as base location for calculating installation prefix. Specifically, the
directory above the directory containing the 'python' executable is taken
as being the installation prefix.

When the Python which should be used is installed in a non standard
location, then that 'bin' directory is unlikely to be in the 'PATH' used by
Apache when it is started. As such, rather than find
'/usr/local/bin/python' it would instead find '/usr/bin/python' and so use
'/usr' rather than the directory '/usr/local/' as the installation prefix.

When this occurs, if under '/usr' there was no Python installation of the
same version number as Python which should be used, then normally:

```
'import site' failed; use -v for traceback
```

would appear in the Apache error log file when Python is first being
initialised within Apache. Any attempt to make a request against a WSGI
application would also result in errors as no modules at all except for
inbuilt modules, would be able to be found when an attempt is made to
import them.

Alternatively, if there was a Python installation of the same version,
albeit not the desired installation, then there may be no obvious issues on
startup, but at run time you may find modules cannot be found when being
imported as they are installed into a different location than that which
was being used. Even if equivalent module is found, it could fail at run
time in subtle ways if the two Python installations are of same version but at
the different locations are compiled in different ways, or if it is a third
party module and they are different versions and so API is different.

In this situation it will be necessary to explicitly tell mod\_wsgi
where the Python executable for the version of Python which should be
used, is located. This can be done using the WSGIPythonHome directive.

```
WSGIPythonHome /usr/local
```

The value given to the WSGIPythonHome directive should be a normalised
path corresponding to that defined by the Python `sys.prefix` variable
for the version of Python being used and passed to the '--with-python'
option when configuring mod\_wsgi.

```
>>> import sys
>>> sys.prefix
'/usr/local'
```

An alternative, although less desirable way of achieving this is to set the
'PATH' environment variable in the startup scripts for Apache. For a standard
Apache installation using ASF structure, this can be done by editing the
'envvars' file in same directory as the Apache executable and adding the
alternate bin directory to the head of the 'PATH':

```
PATH=/usr/local/bin:$PATH
export PATH
```

If there are any concerns over what Python installation directory is being
used and you want to verify what it is, then use a small test WSGI script
which outputs the values of 'sys.prefix' and 'sys.path'. For example:

```
import sys

def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    print >> sys.stderr, 'sys.prefix = %s' % repr(sys.prefix)
    print >> sys.stderr, 'sys.path = %s' % repr(sys.path)

    return [output]
```

## Using ModPython and ModWsgi ##

Although mod\_python and mod\_wsgi both provide the ability to work using
Python under Apache, the way in which mod\_wsgi has been implemented is such
that it is possible for both modules to coexist.

When using the two modules together like this however, it is vitally
important that the Python installation being used was configured and
compiled such that a shared library is produced for the Python library and
not just a static library. If this is not the case and either mod\_python or
mod\_wsgi was built against a version of Python which provided a static
library only, then more than likely one or the other when used will crash
the Apache child process handling the request or cause Apache to be unstable.

Note that this is not the fault of either mod\_python or mod\_wsgi but is a
direct result of only a static library being available for Python. The
reason the crash can occur is that when only a static Python library is
available, the 'libtool' program used to create the dynamically loadable
Apache module has no choice but to embed the object files from the static
Python library into the Apache module. When both mod\_python and mod\_wsgi
are then subsequently loaded there will actually be two copies of all
object files for the Python runtime in memory. Depending on how the runtime
dynamic linker merges the duplicate symbols from these modules, there can
be a mixture of code and data from each set of modules. If different code
objects then access different copies of the same data where one wasn't
initialised, a crash will likely result.

Linux distributions where this is known to be a problem are older Red Hat
and Fedora distributions. Other distributions such as Ubuntu do not have a
problem as they use a shared library for the Python library.

The default options for compiling Python from source code on most platforms
also results in no shared library being produced. Thus if compiling Python
from source code, ensure the '--enable-shared' option is supplied to the
'configure' script. After installation, it may also be necessary to ensure
that a copy or link to the Python shared library is placed in a standard
library directory on your system so it can be found at runtime.

If it is not possible to rebuild Python so as to use a shared library, a
workaround is to fiddle with the Makefile for building mod\_wsgi after
having run 'configure' but before running 'make'. The change required is to
modify the 'LDLIBS' definition so that the Python library is not actually
linked with the mod\_wsgi module. For example:

```
#LDLIBS = -lpython2.3 -lpthread -ldl  -lutil
LDLIBS = -lpthread -ldl  -lutil
```

The result of doing this is that a copy of the objects from the Python
static library will not be embedded in the mod\_wsgi module. Having done
this, when mod\_python and mod\_wsgi are used together, the only copy of the
objects from the Python static library will be in the mod\_python module and
so the problems resulting from duplicates will not arise. It does however
mean that mod\_python must always be loaded into Apache when this mod\_wsgi
module is being used. If at some later point you stop using mod\_python and
only want to use mod\_wsgi, you will need to recompile mod\_wsgi again
without the fiddle.

## Python Version Mismatch ##

When both mod\_python and mod\_wsgi are being used at the same time, what
will happen is that mod\_wsgi will allow mod\_python to perform the initial
initialisation of the Python interpreter.

One problem which can arise though with this scenario is that when Apache
is started you will see error messages of the form:

```
mod_wsgi: Compiled for Python/2.5.1.
mod_wsgi: Runtime using Python/2.2.3.
```

This can occur even though both versions of Python are installed under the
same root directory.

If this occurs, the problem would generally be due to the version of
mod\_python having being originally compiled against the older version of
Python. The mismatch comes about because mod\_wsgi is allowing mod\_python to
perform the initilisation of the Python interpreter and it is therefore
using the version it wants and not that which mod\_wsgi requires.

The only workaround for this problem is to not use mod\_python at all, or
recompile mod\_python to use the same version of Python that mod\_wsgi is
using.

Note that if removing mod\_python from the set of modules that Apache loads,
or replacing it with a version compiled against a newer version of Python,
ensure that you fully 'stop' the Apache web server and not just 'restart'
it. This is necessary as the way that mod\_python initialises the Python
interpreter means that on some platforms it may not correctly shutdown
and unload the Python library. As a result, you may still see the errors
above after a 'restart'.

Even when only mod\_wsgi is being loaded and you have recompiled it for a
newer version of Python or even if replacing it with a newer version of
mod\_wsgi, it is still good practice to properly 'stop' the Apache web
server rather than just 'restart' it.

## Python Patch Level Mismatch ##

If the Python package is upgraded to a newer patch level revision, one
will likely see the following warning messages in the Apache error log
when Python is being initialised.

```
mod_wsgi: Compiled for Python/2.4.1.
mod_wsgi: Runtime using Python/2.4.2.
```

The warning is indicating that a newer version of Python is now being
used than what mod\_wsgi was originally compiled for.

This would generally not be a problem provided that both versions of Python
were originally installed with the '--enable-shared' option supplied to
'configure'. If this option is used then the Python library will be linked
in dynamically at runtime and so an upgrade to the Python version will be
automatically used.

If '--enable-shared' was however not used and the Python library is
therefore embedded into the actual mod\_wsgi Apache module, then there is a
risk of undefined behaviour. This is because the version of the Python
library embedded into the mod\_wsgi Apache module will be older than the
corresponding Python code modules and extension modules being used from the
Python library directory.

Thus, if a shared library is not being used for Python it will be necessary
to rebuild mod\_wsgi against the newer patch level revision of mod\_wsgi and
reinstall it.

## Incompatible ModPython Versions ##

The dual loading of mod\_python and mod\_wsgi together has only been tested
and verified as working for mod\_python 3.3.1. It is possible that
mod\_python 3.2.X may also work, but older mod\_python versions 2.7.X and
3.1.X may give problems.

In particular, it has been noted that dual loading of mod\_python 3.1.4 will
cause mod\_wsgi daemon mode to fail. This occurs because older mod\_python
versions setup threads differently when initialising the Python
interpreter. This appears to cause the mod\_wsgi daemon processes to
deadlock on the Python GIL at startup. This would be evident through any
request to a WSGI application delegated to that daemon process appearing
to hang and the browser client having to timeout.

If mod\_python needs to be available at the same time, it is recommended
that mod\_python 3.3.1 be used. If mod\_python is not actually required it
should not be loaded at all, as by loading it, it actually causes some
features of mod\_wsgi not to work, with equivalent configuration having to
be done through mod\_python with some loss of functionality as a result.

## Mixing 32 Bit And 64 Bit Packages ##

When attempting to compile mod\_wsgi on a Linux system using an X86 64 bit
processor, the following error message can arise:

```
/bin/sh /usr/lib64/apr/build/libtool --silent --mode=link gcc -o \
  mod_wsgi.la -I/usr/local/include/python2.4 -DNDEBUG  -rpath \
  /usr/lib64/httpd/modules -module -avoid-version mod_wsgi.lo \
  -L/usr/local/lib/python2.4/config -lpython2.4 -lpthread -ldl -lutil
/usr/bin/ld: /usr/local/lib/python2.4/config/
libpython2.4.a(abstract.o): relocation R_X86_64_32 against `a local
symbol' can not be used when making a shared object; recompile with -fPIC
/usr/local/lib/python2.4/config/libpython2.4.a: could not read symbols: Bad value
collect2: ld returned 1 exit status
apxs:Error: Command failed with rc=65536
.
make: *** [mod_wsgi.la] Error 1
```

This error is believed to be result of the version of Python being used
having been originally compiled for the generic X86 32 bit architecture
whereas mod\_wsgi is being compiled for X86 64 bit architecture. The actual
error arises in this case because 'libtool' would appear to be unable to
generate a dynamically loadable module for the X86 64 bit architecture from
a X86 32 bit static library. Alternatively, the problem is due to 'libtool'
on this platform not being able to create a loadable module from a X86 64
bit static library in all cases.

If the first issue, the only solution to this problem is to recompile
Python for the X86 64 bit architecture. When doing this, it is preferable,
and may actually be necessary, to ensure that the '--enable-shared' option
is provided to the 'configure' script for Python when it is being compiled
and installed.

If rebuilding Python to generate a shared library, do make sure that the
Python shared library, or a symlink to it appears in the Python 'config'
directory of your Python installation. If the shared library doesn't appear
here next to the static version of the library, 'libtool' will not be able
to find it and will still use the static version of the library. It is
understood that the Python build process may not actually do this, so you
may have to do it by hand.

If the version of Python being used was compiled for X86 64 bit
architecture and a shared library does exist, but not in the 'config'
directory, then adding the missing symlink may be all that is required.

## Unable To Find Python Shared Library ##

When mod\_wsgi is built against a version of Python providing a shared
library, the Python shared library must be in a directory which is searched
for libraries at runtime by Apache. If this isn't the case the Python
shared library will not be able to be found when loading the mod\_wsgi
module in to Apache. The error in this situation will be similar to:

```
error while loading shared libraries: libpython2.4.so.1.0: \
 cannot open shared object file: No such file or directory
```

A number of alternatives exist for resolving this problem. The preferred
solution would be to copy the Python shared library into a directory which
is searched for dynamic libraries at run time. Directories which would
generally always be searched are '/lib' and '/usr/lib'.

For some systems the directory '/usr/local/lib' may also be searched, but
this may depend on the directory having been explicitly added to the
approrpiate system file listing the directories to be searched. The name
and location of this configuration file differs between platforms. On Linux
systems it is often called '/etc/ld.so.conf'. If changes are made to the
file on Linux systems the 'ldconfig' command also needs to be run. See the
manual page for 'ldconfig' for further details.

Rather than changing the system wide list of directories to search for
shared libraries, additional search directories can be specified just
for Apache. On Linux this would entail setting the 'LD\_LIBRARY\_PATH'
environment variable to include the directory where the Python shared
library is installed.

The setting and exporting of the environment variable would be placed in
the Apache 'envvars' file, for a standard Apache installation, located in
the same directory as the Apache web server executable. If using a
customised Apache installation, such as on Red Hat, the 'envvars' file may
not exist. In this case you would need to add this into the actual startup
script for Apache. For Red Hat this is '/etc/sysconfig/httpd'.

A final alternative on some systems is to embed the directory to search
for the Python shared library into the mod\_wsgi Apache module itself. On
Linux systems this can be done by setting the environment variable
'LD\_RUN\_PATH' to the directory containing the Python shared library when
initially building the mod\_wsgi source code.

## GNU C Stack Smashing Extensions ##

Various Linux distributions are starting to ship with a version of the GNU
C compiler which incorporates an extension which implements protection for
[stack-smashing](http://en.wikipedia.org/wiki/Stack-smashing_protector).
In some instances where such a compiler is used to build mod\_wsgi, the
module is unable to then be loaded by Apache. The specific problem is that
the symbol `__stack_chk_fail_local` is being flagged as undefined.

```
$ invoke-rc.d apache2 reload
apache2: Syntax error on line 190 of /etc/apache2/apache2.conf: \
 Cannot load /usr/lib/apache2/modules/mod_wsgi.so into server: \
 /usr/lib/apache2/modules/mod_wsgi.so: \
 undefined symbol: __stack_chk_fail_local failed!
invoke-rc.d: initscript apache2, action "reload" failed.
```

The exact reason for this is not known but it is speculated to be caused
when the system libraries or Apache itself has not been compiled with a
version of the GNU C compiler incorporating the extension.

To workaround the problem, modify the 'Makefile' for mod\_wsgi and change
the value of 'CFLAGS' to:

```
CFLAGS = -Wc,-fno-stack-protector
```

Perform a 'clean' in the directory and then rebuild and reinstall the
mod\_wsgi module.

## Undefined 'forkpty' On Fedora 7 ##

On Fedora 7, the provided binary version of Apache is not linked against
the 'libutil' system library. This causes problems when Python is initialised
and the 'posix' module imported for the first time. This is because the
'posix' module requires functions from 'libutil' but they will not be present.
The error encountered would be similar to:

```
httpd: Syntax error on line 54 of /etc/httpd/conf/httpd.conf: Cannot \
 load /etc/httpd/modules/mod_wsgi.so into server: \
 /etc/httpd/modules/mod_wsgi.so: undefined symbol: forkpty 
```

This problem can be fixed by adding '-lutil' to the list of libraries to
link mod\_wsgi against when it is being built. This can be done by adding
'-lutil' to the 'LDLIBS' variable in the mod\_wsgi 'Makefile' after having
run 'configure'.

An alternative method which may work is to edit the 'envvars' file, if it
exists and is used, located in the same directory as the Apache 'httpd'
executable, or the Apache startup script, and add:

```
LD_PRELOAD=/usr/lib/libutil.so
export LD_PRELOAD
```

## Missing Include Files On SUSE ##

SUSE Linux follows a slightly different convention to other Linux
distributions and has split their Apache "dev" packages in a way as to
allow packages for different Apache MPMs to be installed at the same time.
Although the resultant mod\_wsgi module isn't strictly MPM specific, it
does indirectly include the MPM specific header file "mpm.h". Because the
header file is MPM specific, when configuring mod\_wsgi, it is necessary to
reference the version of "apxs" from the MPM specific "dev" package else
the "mpm.h" header file will not be found at compile time. These errors
are:

```
In file included from mod_wsgi.c:4882: /usr/include/apache2/mpm_common.h:46:17: error: mpm.h: No such file or directory 
...
mod_wsgi.c: In function 'wsgi_set_accept_mutex': 
mod_wsgi.c:5200: error: 'ap_accept_lock_mech' undeclared (first use in this function) 
mod_wsgi.c:5200: error: (Each undeclared identifier is reported only once 
mod_wsgi.c:5200: error: for each function it appears in.) 
apxs:Error: Command failed with rc=65536 
```

To avoid this problem, when configuring mod\_wsgi, it is necessary to use
the "--with-apxs" option to designate that either "apxs2-worker" or
"apxs2-prefork" should be used. Thus:

```
./configure --with-apxs=/usr/sbin/apxs2-worker
```

or:

```
./configure --with-apxs=/usr/sbin/apxs2-prefork
```

Although which is used is not important, since mod\_wsgi when compiled isn't
specific to either, best to use that which corresponds to the version of
Apache being used.

## Apache Maintainer Mode ##

When building mod\_wsgi from source code, on UNIX systems there should be
minimal if no compiler warnings. If you see a lot of warnings, especially
complaints about `ap_strstr`, then your Apache installation has been
configured for maintainer mode.

```
mod_wsgi.c: In function 'wsgi_process_group':
mod_wsgi.c:722: warning: passing argument 1 of 'ap_strstr' discards
qualifiers from pointer target type
mod_wsgi.c:740: warning: passing argument 1 of 'ap_strstr' discards
qualifiers from pointer target type
```

Specifically, whoever built the version of Apache being used supplied the
option '--enable-maintainer-mode' when configuring Apache prior to
installation. You would be able to tell at the time of compiling mod\_wsgi
if this has been done as the option '-DAP\_DEBUG' would be supplied to the
compiler when mod\_wsgi source code is compiled.

These warnings can be ignored, but in general you shouldn't run Apache in
maintainer mode.

A further reason for not running Apache in maintainer mode is that certain
situations can cause Apache to fail an internal assertion check when using
mod\_wsgi. The specific error message is:

```
[crit] file http_filters.c, line 346, assertion "readbytes > 0" failed
[notice] child pid 18551 exit signal Aborted (6)
```

This occurs because the Apache code has an overly agressive assertion
check, which is arguably incorrect. This particular assertion check will
fail when a zero length read is perform on the Apache 'HTTP\_IN' input
filter.

This scenario can arise in mod\_wsgi due to a workaround in place to get
around a bug in Apache related to generation of '100-continue' response.
The Apache bug is described in:

> https://issues.apache.org/bugzilla/show_bug.cgi?id=38014

The scenario can also be triggered as a result of a WSGI application
performing a zero length read on 'wsgi.input'.

Changes to mod\_wsgi are being investigated to see if zero length reads can
be ignored, but due to the workaround for the bug, this would only be able
to be done for Apache 2.2.8 or later.

The prefered solution is simply not to use Apache with maintainer mode
enabled for systems where you are running real code. Unfortunately, it
looks like some Linux distributions, eg. SUSE, accidentally released Apache
binary packages with this mode enabled by default. You should update to a
Apache binary package that doesn't have the mode enabled, or compile from
source code.