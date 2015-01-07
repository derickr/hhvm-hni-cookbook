Instantiating a Class and Setting Properties
============================================

Instantiating Classes
---------------------

If you need to instantiate a class, you can do that through
``ObjectData::newInstance()``. It wants a ``Class*`` as argument, which you can
obtain by calling ``Unit::lookupClass()``.

In full, you'd have to do something like::

	const StaticString s_MongoDriverWriteResult_className("MongoDB\\Driver\\WriteResult");

	...

	{
		static Class* c_foobar;
		...

		c_foobar = Unit::lookupClass(s_MongoDriverWriteResult_className.get());
		ObjectData* obj = ObjectData::newInstance(c_foobar);

		return Object(obj);
	}

This works for both Natively implemented as well as PHP/Hack defined classes.

This snippet also returns the newly instantiated object. Make sure you get the
name of the class right, and that you don't get a NULL pointer back from
`lookupClass`, or otherwise you get weird errors back. In my case, a "can not
convert array to string" warning.
