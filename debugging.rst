Debugging
=========

A few tricks for debugging structures and related things.

Value of String objects
-----------------------

In GDB, you can print the value of a String object with::

	p *named_class.m_str.m_px

	p ((StringData*) 0x7fffe3c06000).m_data

Value of Array objects
----------------------

::

	p *v.m_data.parr

	p ((MixedArray::Elm*)((MixedArray*)&$10 + 1))[0]@2


GDB utilities
-------------

In ``./hphp/tools/gdb/*.py`` you can find a lot of GDB helper utilities. You
can source these in GDB by running::

	source path/hphp/toold//gdb/hhvm.py

You can then easily print f.e. a string value with::

	p named_class

Instead of::

	(gdb) p named_class
	$1 = {m_str = {m_px = 0x7fffe0429fd0}, static MinPrecomputedInteger = -128,
	  static MaxPrecomputedInteger = 3967, static converted_integers_raw = 0x7fffe849d000,
	  static converted_integers = 0x7fffe849d400, static integer_string_data_map = 
	  {<std::unordered_map<long, HPHP::StringData const*, HPHP::int64_hash, 
	  std::equal_to<long>, std::allocator<std::pair<long const, 
	  HPHP::StringData const*> > >> = std::unordered_map with 0 elements, 
	  <No data fields>}, static npos = -1}

You will see::

	(gdb) p named_class
	$2 = (HPHP::req::ptr<HPHP::StringData>) 0x7fffe0429fd0 "FallbackClass"

