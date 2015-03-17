#summary Developer guidelines for using mod\_wsgi.
#labels Phase-Deploy

_**If you are using mod\_wsgi, please consider making a
[donation](HowToContributeBack.md).**_

# Developer Guidelines #

After having installed and configured mod\_wsgi you will want to know how to
get the most out of it. This document links to documentation on this site
and elsewhere relevant to the developer of WSGI applications in general and
in particular WSGI applications being hosted on mod\_wsgi.

The most important document is the WSGI specification itself. This can
be found at:

> http://www.python.org/dev/peps/pep-0333/

This defines the interface specification that WSGI applications and WSGI
adapters for web servers, such as mod\_wsgi, are supposed to adhere to.
Additional information about WSGI in general can be found on the WSGI wiki
at:

> http://www.wsgi.org/wsgi

If you have quite specific questions about the WSGI specification, you
should ask on the Python WEB-SIG mailing list.

> http://groups.google.com/group/python-web-sig

If you are using a particular Python web framework to implement your
application, you might also want to ask your questions on the relevant
mailing lists for those packages if it relates more to the package
rather than the WSGI specification or mod\_wsgi.

If on the other hand your problems do appear to relate more to mod\_wsgi,
first off ensure that you read the following documents.

  * [Installation Issues](InstallationIssues.md)
  * [Configuration Issues](ConfigurationIssues.md)
  * [Application Issues](ApplicationIssues.md)

If none of the common issues match up with the problem you are seeing and
you have the need to perform more low level debugging, check out the
following documents.

  * [Debugging Techniques](DebuggingTechniques.md)

You can also do some basic checking of your installation and configuration
to validate that how it is setup is how you expect it to be. See the
following documents.

  * [Checking Your Installation](CheckingYourInstallation.md)

If you are still having no joy and need further assistance then ask any
questions you have on the mod\_wsgi mailing list at:

> http://groups.google.com/group/modwsgi

If you want to know more about some of the inner workings of mod\_wsgi,
how it is implemented and how it interacts with Apache read the following
documents.

  * [Processes And Threading](ProcessesAndThreading.md)
  * [Virtual Environments](VirtualEnvironments.md)

  * [Reloading Source Code](ReloadingSourceCode.md)
  * [Registering Cleanup Code](RegisteringCleanupCode.md)

  * [File Wrapper Extension](FileWrapperExtension.md)
  * [Access Control Mechanisms](AccessControlMechanisms.md)

  * [Miscellaneous Tips And Tricks](TipsAndTricks.md)

If you want to know more detail about some of the problems that can arise
when running WSGI applications on top of mod\_wsgi read the following
documents.

  * [Issues With Expat Library](IssuesWithExpatLibrary.md)
  * [Issues With Pickle Module](IssuesWithPickleModule.md)