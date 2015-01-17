Exceptions
==========

Throwing a Predefined Exception From C++ Land
---------------------------------------------

To throw an (SPL) InvalidArgumentException, you can add the following snippet
to your code::

	throw Object(SystemLib::AllocInvalidArgumentExceptionObject("Invalid tagSet"));

There are similar predefined methods for other standard exceptions:

Exception
	``AllocExceptionObject``

BadMethodCallException (Spl)
	``AllocBadMethodCallExceptionObject``

RuntimeException (Spl)
	``AllocRuntimeExceptionObject``

OutOfBoundsException (Spl)
	``AllocOutOfBoundsExceptionObject``

InvalidOperationException (Spl)
	``AllocInvalidOperationExceptionObject``

DOMException (Spl)
	``AllocDOMExceptionObject``

