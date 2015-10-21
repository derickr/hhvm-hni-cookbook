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
		-D CMAKE_CXX_COMPILER="/usr/lib/ccache/g++" \
		-D CMAKE_C_COMPILER="/usr/lib/ccache/gcc" \
		.

If you get strange errors, remove ``CMakeCache.txt`` first. The above will
instruct ``cmake`` to use ``ccache`` as the compiler. It does not support ASM
compilation caching, so we need to set that to the standard compiler.

With ccache enabled, a ``make clean && make -j5`` now takes about 90 seconds,
instead of nearly two hours.

.. _ccache: https://ccache.samba.org/

Debug Builds of HHVM
--------------------

In order to make a debug build, you need to pass yet another flag to
``cmake``::

	cmake \
		-DCMAKE_BUILD_TYPE=Debug \
		.

This new flag can be combined with the flags from the ccache section to create
a debug build of HHVM. This sets the ``-Og`` flag instead of ``-O3`` among
other things.

Install Prefix
--------------

You can set an install prefix by running cmake with a flag to ``cmake`` too::

	cmake \
		-DCMAKE_INSTALL_PREFIX=/usr/local/hhvm/3.9 \
		.

Default php.ini File Location
-----------------------------

Since HHVM 3.10, you can specify the default location for the ``php.ini``
file. This is handy if you build/run more than one version of HHVM. The flag
is::

	cmake \
		-DDEFAULT_CONFIG_DIR=/etc/hhvm/3.10 \
		.

Full Build
----------

With all the flags, the ``cmake`` incantation is::

	cmake \
		-DCMAKE_INSTALL_PREFIX=/usr/local/hhvm/3.9 \
		-DCMAKE_CXX_COMPILER="/usr/lib/ccache/g++" \
		-DCMAKE_C_COMPILER="/usr/lib/ccache/gcc" \
		-DCMAKE_BUILD_TYPE=Debug \
		.

Debug Builds of HHVM extensions
-------------------------------

Sadly, this still does not create easy to debug builds - especially when you
are making an HHVM extension. I had to modify the
``CMakeFiles/mongodb.dir/flags.make`` file **after** running ``hphpize``. I
changed the ``-Og`` to ``-O0 -ggdb3``. After compilation, I now have a debug
build that works well with ``valgrind`` and ``gdb``.

Please note, that you have to do this every single time after you have run
``hphpize``.

An alternative, is to modify the installed
``/usr/local/lib/hhvm/CMake/HPHPCompiler.cmake`` and (in vim) run::

	:%s/"-Og -g"/""

When this is done, you can run cmake in the extension's directory (after
``hphpize``)::

	cmake \
		-DCMAKE_C_FLAGS="-O0 -ggdb3" \
		-DCMAKE_CXX_FLAGS="-O0 -ggdb3" \
		.

Verbose Builds
--------------

In order to see what ``make`` actually runs, you can invoke it with verbosity
turned on::

	make -j5 VERBOSE=1

This then shows the full command to compile each file, and is useful for
figuring out if things go wrong, and whether specific compiler flags are set.

HHVM make errors
----------------

Once in a while, ocaml gets confused about compiled files -- for example when
a file is renamed. The error looks like something like::

	[ 46%] Built target hphp_analysis
	Finished, 14 targets (14 cached) in 00:00:00.
	+ /usr/bin/ocamlc.opt -c -g -w A -warn-error A -w -27 -w -4-6-29-35-44-48 -I server -I dfind -I utils -I format -I stubs -I socket -I procs -I parsing -I hhi -I h2tp -I typing -I fsnotify_linux -I naming -I search -I client -I globals -I deps -I heap -I h2tp/test -I h2tp/unparser -I h2tp/mapper -I h2tp/common -I third-party/inotify -I third-party/avl -I third-party/core -o server/serverConfig.cmi server/serverConfig.mli
	File "server/serverConfig.mli", line 1:
	Error: The files utils/relative_path.cmi and utils/typecheckerOptions.cmi
		   make inconsistent assumptions over interface Utils
	Command exited with code 2.
	Compilation unsuccessful after building 84 targets (83 cached) in 00:00:00.
	Makefile:118: recipe for target '_build/hh_server.native' failed
	make[3]: *** [_build/hh_server.native] Error 10
	hphp/hack/CMakeFiles/hack.dir/build.make:49: recipe for target 'hphp/hack/CMakeFiles/hack' failed
	make[2]: *** [hphp/hack/CMakeFiles/hack] Error 2
	CMakeFiles/Makefile2:1226: recipe for target 'hphp/hack/CMakeFiles/hack.dir/all' failed
	make[1]: *** [hphp/hack/CMakeFiles/hack.dir/all] Error 2
	Makefile:116: recipe for target 'all' failed
	make: *** [all] Error 2

You can fix this, by running::

	git clean -fdx hphp/hack/

And rebuild.

HHVM and GCC 5
--------------

These don't work well together yet, instead, you need to compile with::

	cmake \
		-DCMAKE_BUILD_TYPE=Debug \
		-DCMAKE_CXX_COMPILER=`which g++-4.9` \
		-DCMAKE_C_COMPILER=`which gcc-4.9` \
		-DCMAKE_ASM_COMPILER=`which gcc-4.9` \
		-DCMAKE_INSTALL_PREFIX=/usr/local/hhvm/3.10.0 \
		.

This is not all though, you also need special versions of Boost and Google
Log, if you're using a really new set of Debian packages. There is extra
information at http://derickrethans.nl/hhvm-gcc-52.html
