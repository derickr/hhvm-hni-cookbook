Adding class storage to store C-structs
=======================================

This explains how to store C-based structures with your HHVM Native classes —
akin to Zend Engine's "internal" member of the object structure.

Definitions
-----------

For each class, you need to implement a corresponding ``Data`` class. For
example if your class is ``MongoDBManager``, then you define a
``MongoDBManagerData`` class in the ``HPHP`` namespace::

	class MongoDBManagerData
	{
		public:
			static Class* s_class;
			static const StaticString s_className;

			/* Your own members */

			static Class* getClass();

			void sweep() {
				/* Here you destroy your resources */
			};

			~MongoDBManagerData() {
				sweep();
			}
	};

The ``sweep()`` method is called at the end of the request if the Data class
has not gone out of scope yet, and the destructor is called when (of course),
this Data object goes out of scope.

To initialise the class, you also need to do::

	Class* MongoDBManagerData::s_class = nullptr;
	const StaticString MongoDBManagerData::s_className("MongoDBManager");

And implement the ``getClass()`` method. As every ``getClass()`` method is the
same, you can use the following macro instead::

	#define IMPLEMENT_GET_CLASS(cls) \
		Class* cls::getClass() { \
			if (s_class == nullptr) { \
				s_class = Unit::lookupClass(s_className.get()); \
				assert(s_class); \
			} \
			return s_class; \
		} 

And then call it like::

	IMPLEMENT_GET_CLASS(MongoDBManagerData);

Registering
===========

In the ``moduleInit`` of your extension, you need to register this
``NativeData`` class with::

	Native::registerNativeDataInfo<MongoDBManagerData>(MongoDBManagerData::s_className.get());

In the class definition (in ``ext_mongodb.php``) you would also need the
``_NativeData`` strata before your class name::

	<<__NativeData("MongoDBManager")>>
	class Manager {
		<<__Native>>
		function __construct(string $dsn = "localhost", array $options = array(), array $driverOptions = array()): void;
	}

**Warning:** If you forget the ``<<__NativeData("MongoDBManager")>>``
annotation for the class, you are going to be in for a long frustrating ride
figuring out why data is suddenly changing.
