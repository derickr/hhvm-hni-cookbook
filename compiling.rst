Compiling
=========

In order to make debugging work, and compiling faster, there are a few things
that you need to configure.

ccache
------

Repeatedly compiling HHVM can benefit enormously from having ccache_ enabled.
It was a bit tricky to get going, so hence I am documenting it.

First of all, install ccache_ (through ``apt-get``) or something.
Then, when running ``cmake`` for HHVM, add a few flags::

	cmake \
		-D CMAKE_CXX_COMPILER="ccache" -D CMAKE_CXX_COMPILER_ARG1="g++" \
		-D CMAKE_C_COMPILER="ccache" -D CMAKE_C_COMPILER_ARG1="gcc" \
		-D CMAKE_ASM_COMPILER=/usr/bin/gcc \
		.

If you get strange errors, remove ``CMakeCache.txt`` first. The above will
instruct ``cmake`` to use ``ccache`` as the compiler. It does not support ASM
compilation caching, so we need to set that to the standard compiler.

With ccache enabled, a ``make clean && make -j5`` now takes about 90 seconds,
instead of nearly an hour.

.. _ccache: https://ccache.samba.org/

Debug Builds
------------

In order to make a debug build, you need to pass yet another flag to
``cmake``::

	cmake \
		-DCMAKE_BUILD_TYPE=Debug \
		.

This new flag can be combined with the flags from the ccache section to create
a debug build of HHVM. This sets the ``-Og`` flag instead of ``-O3`` among
other things.

Sadly, this still does not create easy to debug builds - especially when you
are making an HHVM extension. I had to modify the
``CMakeFiles/mongodb.dir/flags.make`` file **after** running ``hphpize``. I
changed the ``-Og`` to ``-O0 -ggdb3``. After compilation, I now have a debug
build that works well with ``valgrind`` and ``gdb``.

Please note, that you have to do this every single time after you have run
``hphpize``.

Verbose Builds
--------------

In order to see what ``make`` actually runs, you can invoke it with verbosity
turned on::

	make -j5 VERBOSE=1

This then shows the full command to compile each file, and is useful for
figuring out if things go wrong, and whether specific compiler flags are set.
