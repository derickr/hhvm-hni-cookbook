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
