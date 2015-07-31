Versioning
==========

This section lists changes in APIs between HHVM versions (3.8 and later), and
shows how to test for specific versions.


Checking for HHVM versions
--------------------------

HHVM defines a ``HHVM_VERSION_ID`` constant that seems like PHP's
``PHP_VERSION_ID``. Although it can be used for the same purpose, it does not
follow the ``VERSION_MAJOR * 10000 + VERSION_MINOR * 100 + VERSION_PATCH`` that
PHP follows. Instead, it uses ``VERSION_MAJOR * 65536 + VERSION_MINOR * 256 +
VERSION_PATCH``. This makes testing for HHVM version 3.8 or higher, look like::

	#if HHVM_VERSION_ID >= 198656

This is not very readable.

HHVM does define a *userland* ``HHVM_VERSION_ID`` constant, with the PHP way of
calculating it. This means that::

	<?php
	echo HHVM_VERSION_ID, "\n";
	?>

Will echo ``30800``.

As this is something we really want to replicate in an extension, it makes
sense to add the following define to your main header file::

	#define HIPPO_HHVM_VERSION (HHVM_VERSION_MAJOR * 10000 + HHVM_VERSION_MINOR * 100 + HHVM_VERSION_PATCH)

And then you can check for versions with::

	#if HIPPO_HHVM_VERSION >= 30900
