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

Setting Properties
------------------

Properties can be set with the ``o_set`` method called on a ``ObjectData``
object. It accepts two required arguments (name, and value) and an optional
third argument (the context).

The name is passed as a ``String``, and the value as a ``Variant``. As an
example, you can include::

	obj->o_set(String("nInserted"), Variant(52));

This will set the property ``nInserted`` to the (integer) value ``52``.

The context is argument is required if you want to set the value of a
*private* or *protected* property. The context is ``String`` argument.

If setting a private property, the context should be the name of the class
that the property is defined on. As an example::

	const StaticString s_MongoDriverWriteResult_className("MongoDB\\Driver\\WriteResult");

	obj->o_set(
		String("nInserted"), 
		Variant(52),
		s_MongoDriverWriteResult_className.get()
	);

If setting a protected property, the context should either be the name of the
class on which the property is defined, or an inherited class.
