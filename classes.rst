Classes and Properties
======================

Instantiating Classes
---------------------

If you need to instantiate a class, you can do that through
``Object{}``. It wants a ``Class*`` as argument, which you can
obtain by calling ``Unit::lookupClass()``.

In full, you'd have to do something like::

	const StaticString s_MongoDriverWriteResult_className("MongoDB\\Driver\\WriteResult");

	...

	{
		static Class* c_foobar;
		...

		c_foobar = Unit::lookupClass(s_MongoDriverWriteResult_className.get());
		Object obj = Object{c_foobar};

		return obj;
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

Calling Methods
---------------

In order to call a method, you first need to obtain the HHVM equivalent to a
zend_class_entry::

	Object v;
	Class *cls;
	
	cls = v.get()->getVMClass();

On this class object you then run ``lookupMethod`` to obtain a handle to the
function/method::

	Func *m;
	const StaticString s_MongoDriverBsonSerializable_functionName("bsonSerialize");

	m = cls->lookupMethod(s_MongoDriverBsonSerializable_functionName.get());

Arguments are defined in an array of ``TypedValue`` variables::

	TypedValue args[1] = {
		*(Variant(v)).asCell()
	};

In my example, I convert my ``v`` Object to a Variant:: ``Variant(v)`` and on
this Variant I call ``asCell()`` to create a TypedValue. The pointer to this
TypedValue is then placed in the ``args`` array.

The obtained method handle can be executed by calling ``invokeFuncFew`` on the
global context ``g_context``. You need to include the ``execution-context.h``
header for that::

	#include "hphp/runtime/base/execution-context.h"

With this global context, you then call the function::

	Variant result;

	g_context->invokeFuncFew(
		result.asTypedValue(), // the by-ref result
		m,                     // the method handle
		v.get(),               // the object data, providing context
		nullptr,               // a null pointer (should only be non-NULL)
		                       // when calling __call or __callStatic
		1, args                // the number of arguments, and the arguments in
		                       // an array
	);


Obtaining Object Properties as Array
------------------------------------

In the simplest form, you can get all the properties of an object as an Array
by just calling ``toArray()`` on the object::

	Array properties;
	Object v;

	properties = result.toArray();

However, this also includes private and protected properties. If you do not
want to include those in the resulting array, you need to iterate over the
properties in a specific context.

The iteration and conversion to Array can be done with the ``o_toIterArray()``
method on an ``Object``. This method accepts two arguments. The first one is
the context—the class name as a string. The second one a set of options
enumerated by ``IterMode``: ``EraseRefs``, ``CreateRefs`` or ``PreserveRefs``.

In the following example, we are using ``null_string`` as the class context.
That means that we will never get ``protected`` or ``private`` properties in
the resulting array. We are also just preserving references::

	Array document;
	Object v;

	document = v->o_toIterArray(null_string, ObjectData::PreserveRefs);

Checking whether a Class is of a Certain Type
---------------------------------------------

If you want to find out whether a ``Class`` is a normal class, or something
else, there are a few methods that you can call on a ``Class*`` to find out.
For example, to find out if a class is a "concrete class", and not an
interface, trait, or an enum), you can use::

	Class *cls;

	isNormalClass(cls);

There is also ``isTrait()``, ``isEnum()``, ``isInterface()`` and
``isAbstract()``.

Please note that ``isNormalClass()`` also allows for abstract classes, so if
you want to check whether a class is a real concrete class, you will need to
check this with::

	if (isNormalClass(cls) && !isAbstract(cls)) {
