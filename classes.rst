Classes and Properties
======================

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

Getting Properties
------------------

Properties can be read with the ``o_get`` method called on a ``ObjectData``
object. It accepts one required argument (name), and two optional ones (throw
error, and the context).

The name is passed as a ``String``, and when you have a public property, you
can read it with::

	obj->o_get(String("pattern"));

In order to read a *private* or *protected* property, you need to provide a
context, which is a ``String`` argument. Because this is the third argument,
you also have to provide the second one (which defaults to ``true``). As an
example, to read the ``flags`` private property, you can do::


	const StaticString s_MongoDriverBsonRegex_className("MongoDB\\BSON\\Regex");
	const StaticString s_MongoDriverBsonRegex_pattern("pattern");

	String regex = v.o_get(s_MongoDriverBsonRegex_pattern, false, s_MongoDriverBsonRegex_className);

Getting a ClassName
-------------------

To retrieve the class name of an ``HPHP::Object`` you can call the
``getClassName`` method. This is a convenience method that actually does
``.get->getClassName`` — i.e., it goes through ``ObjectData`` first.

As an example::

	static Object HHVM_METHOD(MongoDBManager, executeInsert,
		const String &ns, const Variant &document, const Object &v)
	{
		std::cout << v->getClassName().c_str() << " object converted to ";
	}

However, instead of using a class name directly to compare with, it is likely
better to use ``instanceof``.

Comparing Classes (instanceof)
------------------------------

To check whether an object is of a specific class (or an inherited class), you
can use the ``instanceof`` method on ``HPHP::Object``. A simple equality test
looks like::

	const StaticString s_MongoDriverBsonRegex_className("MongoDB\\BSON\\Regex");

	if (v.instanceof(s_MongoDriverBsonRegex_className)) {
	}

This also works for interface implementations. If you consider the ``Regex``
class to implement ``Type``, then this will also match for the same objects as
in the above example::

	const StaticString s_MongoDriverBsonType_className("MongoDB\\BSON\\Type");

	void VariantToBsonConverter::convertPart(Object v)
	{
		if (v.instanceof(s_MongoDriverBsonType_className)) {
			// the class of object v implements "Type"
		}
	}

Defining Class Constants
------------------------

You can define class constants directly in the ``ext_*.php`` files, for
example as::

	class Query {
		const FLAG_NONE = 1;
	}

But when the value of the constant is defined in a library that you are
wrapping, you need to do a little bit more work.

In the ``moduleInit()`` of your extension (in the ``*.cpp`` file), you can use
``Native::registerClassConstant`` to register these constants. As an example,
you can do::

	const StaticString s_MongoDriverQuery_className("MongoDB\\Driver\\Query");

	…

	virtual void moduleInit() {
		…
		Native::registerClassConstant<KindOfInt64>(
			s_MongoDriverQuery_className.get(),
			makeStaticString("FLAG_NONE"), 
			(int64_t) MONGOC_QUERY_NONE
		);
		…

The type that you are registering with is defined in the angle brackets
``<…>``, in most cases, it's the PHP type with ``KindOf`` in front of it. In
this example, we are registering the class constant
``MongoDB\Driver\Query::FLAG_NONE`` with the value in ``MONGOC_QUERY_NONE``.
This (C-level) constant is defined in the libmongoc_ library.

.. _libmongoc: https://github.com/mongodb/mongo-c-driver
