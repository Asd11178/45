#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

# $Id$

AC_PREREQ([2.63])

AC_INIT([libtorrent-rasterbar],[1.2.1],[arvid@libtorrent.org],
        [libtorrent-rasterbar],[http://www.libtorrent.org])
AC_CONFIG_SRCDIR([src/torrent.cpp])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([m4])

# Silencing build output (automake-1.11)
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

# Change ar argument to 'rc' to silence the warning:
#     ar: `u' modifier ignored since `D' is the default (see `U')
m4_divert_text([DEFAULTS], [: "${AR_FLAGS=rc}"])

###############################################################################
dnl ---------------------------------------------------------------------------
dnl interface version info
dnl ---------------------------------------------------------------------------
dnl Advanced information about versioning:
dnl   * "Writing shared libraries" by Mike Hearn
dnl         http://navi.cx/~mike/writing-shared-libraries.html
dnl   * libtool.info chapter "Versioning"
dnl   * libtool.info chapter "Updating library version information"
dnl ---------------------------------------------------------------------------
dnl Versioning:
dnl  - CURRENT (Major):  Increment if the interface has changes. AGE is always
dnl                      *changed* at the same time.
dnl  - AGE (Micro):      Increment if any interfaces have been added; set to 0
dnl                      if any interfaces have been removed. Removal has
dnl                      precedence over adding, so set to 0 if both happened.
dnl                      It denotes upward compatibility.
dnl  - REVISION (Minor): Increment any time the source changes; set to
dnl                      0 if you incremented CURRENT.
dnl
dnl  To summarize. Any interface *change* increment CURRENT. If that interface
dnl  change does not break upward compatibility (ie it is an addition),
dnl  increment AGE, Otherwise AGE is reset to 0. If CURRENT has changed,
dnl  REVISION is set to 0, otherwise REVISION is incremented.
dnl ---------------------------------------------------------------------------

m4_define([VERSION_INFO_CURRENT],[10])
m4_define([VERSION_INFO_REVISION],[0])
m4_define([VERSION_INFO_AGE],[0])
INTERFACE_VERSION_INFO=VERSION_INFO_CURRENT:VERSION_INFO_REVISION:VERSION_INFO_AGE
AC_SUBST(INTERFACE_VERSION_INFO)
###############################################################################


###############################################################################
# Start
###############################################################################

AS_ECHO
AS_ECHO "Building $PACKAGE_STRING"


###############################################################################
# Performing some basic checks and initializing the build system
###############################################################################

AS_ECHO
AS_ECHO "Checking for a C/C++ compiler to use:"
AC_PROG_CC
AC_PROG_CPP
AC_PROG_CC_C_O
AC_PROG_CXX
AC_PROG_CXXCPP
AC_PROG_CXX_C_O

AS_ECHO
AS_ECHO "Checking system type:"
AC_CANONICAL_BUILD
AC_CANONICAL_HOST
AC_CANONICAL_TARGET

AS_ECHO
AS_ECHO "Initializing Automake:"
AM_INIT_AUTOMAKE([1.11 foreign])
AM_MAINTAINER_MODE([enable])

AS_ECHO
AS_ECHO "Initializing Libtool:"
LT_PREREQ([2.2.6])
LT_INIT

AS_IF([test "$SYS" = linux],[
    AC_PREPROC_IFELSE([AC_LANG_PROGRAM(
        [[#ifndef __ANDROID__
         # error Not Android
         #endif
        ]],[[;]])
    ],[
      HAVE_ANDROID="1"
      AC_MSG_RESULT([yes])
    ],[
      AC_MSG_RESULT([no])
    ])
])
AM_CONDITIONAL(HAVE_ANDROID, test "${HAVE_ANDROID}" = "1")

HAVE_WINDOWS=0
case "${host_os}" in
    *mingw32* | *cygwin*)
        HAVE_WINDOWS=1
        ;;
esac
AM_CONDITIONAL(HAVE_WINDOWS, test "${HAVE_WINDOWS}" = "1")

###############################################################################
# Checking for needed base libraries
###############################################################################

AS_ECHO
AS_ECHO "Checking for posix thread support:"

AX_PTHREAD()

LIBS="$PTHREAD_LIBS $LIBS"
CFLAGS="$PTHREAD_CFLAGS $CFLAGS"
CC="$PTHREAD_CC"
CXXFLAGS="$CXXFLAGS -ftemplate-depth=512 -Wno-format-zero-length"

AS_ECHO "Checking for visibility support:"
AC_CACHE_CHECK([for __attribute__((visibility("hidden")))],
    ac_cv_hidden_visibility_attribute, [
    echo 'int __attribute__ ((visibility ("hidden"))) foo (void) { return 1; }' > visibility_conftest.c
    ac_cv_hidden_visibility_attribute=no
    if AC_TRY_COMMAND(${CC-cc} -fvisibility=hidden -S visibility_conftest.c -o visibility_conftest.s 1>&AS_MESSAGE_LOG_FD);
    then
        AS_ECHO "found"
        ac_cv_hidden_visibility_attribute=yes
    fi
    rm -f visibility_conftest.*
])

AS_ECHO
AS_ECHO "Checking for boost libraries:"

AX_BOOST_BASE([1.58])

AX_CXX_COMPILE_STDCXX_11([noext], [mandatory])

AX_BOOST_SYSTEM()
AS_IF([test -z "$BOOST_SYSTEM_LIB"],
      [AC_MSG_ERROR(Boost.System library not found. Try using --with-boost-system=lib)])

CPPFLAGS="$BOOST_CPPFLAGS $CPPFLAGS"
LDFLAGS="$BOOST_LDFLAGS $LDFLAGS"
LIBS="$BOOST_SYSTEM_LIB $LIBS"

###############################################################################
# Checking for functions and other stuffs
###############################################################################


AS_ECHO
AS_ECHO "Checking for pkg-config:"

PKG_PROG_PKG_CONFIG([0.20])

AS_ECHO
AS_ECHO "Checking for functions and headers:"

AC_SYS_LARGEFILE
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET

AC_CHECK_FUNCS([clock_gettime], [],
    [AC_CHECK_LIB([rt], [clock_gettime], [],
      [AC_CHECK_LIB([posix4], [clock_gettime], [],
      [AC_MSG_WARN([clock_gettime function not found.])])])]
)


dnl Pass some build options to .pc file
COMPILETIME_OPTIONS=""


###############################################################################
# Setting configure options
###############################################################################

AC_ARG_ENABLE(
  [logging],
  [AS_HELP_STRING(
    [--enable-logging],
    [enable logging alerts [default=yes]])],
  [[ARG_ENABLE_LOGGING=$enableval]],
  [[ARG_ENABLE_LOGGING=yes]]
)

AC_ARG_ENABLE(
  [debug],
  [AS_HELP_STRING(
    [--enable-debug],
    [enable debug build [default=no]])],
  [
   ARG_ENABLE_DEBUG=$enableval
   ac_arg_enable_debug=$enableval
  ],
  [
   ARG_ENABLE_DEBUG=no
   ac_arg_enable_debug=no
  ]
)

AC_ARG_ENABLE(
  [dht],
  [AS_HELP_STRING(
    [--enable-dht],
    [enable dht support [default=yes]])],
  [[ARG_ENABLE_DHT=$enableval]],
  [[ARG_ENABLE_DHT=yes]]
)

AC_ARG_ENABLE(
  [encryption],
  [AS_HELP_STRING(
    [--enable-encryption],
    [enable encryption support (requires OpenSSL to be installed on your system, you can use --with-openssl to set the path) [default=yes]])],
  [[ARG_ENABLE_ENCRYPTION=$enableval]],
  [[ARG_ENABLE_ENCRYPTION=yes]]
)

AC_ARG_ENABLE(
  [export-all],
  [AS_HELP_STRING(
    [--enable-export-all],
    [export all symbols from libtorrent, including non-public ones [default=no]])],
  [[ARG_ENABLE_FULL_EXPORT=$enableval]],
  [[ARG_ENABLE_FULL_EXPORT=no]]
)

AC_ARG_ENABLE(
  [invariant-checks],
  [AS_HELP_STRING(
    [--enable-invariant-checks],
    [enable invariant checks (use value "full" to turn on extra expensive invariant checks) @<:@default=yes if debug is enabled, no otherwhise@:>@])],
  [[ARG_ENABLE_INVARIANT=$enableval]],
  [
   AS_IF([test "x$ac_arg_enable_debug" = "xyes"],
         [ARG_ENABLE_INVARIANT=yes],
         [ARG_ENABLE_INVARIANT=no])
  ]
)

AC_ARG_ENABLE(
  [deprecated-functions],
  [AS_HELP_STRING(
    [--enable-deprecated-functions],
    [enable deprecated functions in the API [default=yes]])],
  [[ARG_ENABLE_DEPRECATED=$enableval]],
  [[ARG_ENABLE_DEPRECATED=yes]]
)

AC_ARG_ENABLE(
  [examples],
  [AS_HELP_STRING(
    [--enable-examples],
    [build example files [default=no]])],
  [[ARG_ENABLE_EXAMPLES=$enableval]],
  [[ARG_ENABLE_EXAMPLES=no]]
)

AC_ARG_ENABLE(
  [tests],
  [AS_HELP_STRING(
    [--enable-tests],
    [build test files [default=no]])],
  [[ARG_ENABLE_TESTS=$enableval]],
  [[ARG_ENABLE_TESTS=no]]
)

AC_ARG_ENABLE(
  [python-binding],
  [AS_HELP_STRING(
    [--enable-python-binding],
    [build python bindings [default=no]])],
  [[ARG_ENABLE_PYTHON_BINDING=$enableval]],
  [[ARG_ENABLE_PYTHON_BINDING=no]]
)

AC_ARG_WITH(
  [libiconv],
  [AS_HELP_STRING(
    [--with-libiconv],
    [enable linking against system libiconv [default=no]])],
  [[ARG_WITH_LIBICONV=$withval]],
  [[ARG_WITH_LIBICONV=no]]
)

###############################################################################
# Checking configure options
###############################################################################

AS_ECHO
AS_ECHO "Checking build options:"

AC_MSG_CHECKING([whether deprecated functions should be enabled])
AS_CASE(["$ARG_ENABLE_DEPRECATED"],
  ["yes"|"on"], [
      AC_MSG_RESULT([yes])
    ],
  ["no"|"off"], [
      AC_MSG_RESULT([no])
      AC_DEFINE([TORRENT_NO_DEPRECATE],[1],[Define to exclude all deprecated functions from the API.])
      COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DTORRENT_NO_DEPRECATE "
    ],
  [AC_MSG_RESULT([$ARG_ENABLE_DEPRECATED])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_DEPRECATED". Use either "yes" or "no".])]
)

AC_MSG_CHECKING([whether debug build should be enabled])
AS_CASE(["$ARG_ENABLE_DEBUG"],
  ["yes"], [
      AC_MSG_RESULT([yes])
      AC_DEFINE([TORRENT_USE_ASSERTS],[1],[Define to enable debug code.])
      COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DTORRENT_USE_ASSERTS"
      DEBUGFLAGS="-Og"
   ],
  ["no"], [
      AC_MSG_RESULT([no])
      AC_DEFINE([NDEBUG],[1],[Define to disable debug code.])
      #COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DNDEBUG "
      DEBUGFLAGS="-g0 -Os"
   ],
  [AC_MSG_RESULT([$ARG_ENABLE_DEBUG])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_DEBUG". Use either "yes" or "no".])]
)

AC_MSG_CHECKING([whether invariant check should be enabled]) #depends: $ac_arg_enable_debug
AS_CASE(["$ARG_ENABLE_INVARIANT"],
  ["yes"|"on"], [
      AC_MSG_RESULT([yes])
      AC_DEFINE([TORRENT_USE_INVARIANT_CHECKS],[1],[Define to enable internal invariant checks.])
    ],
  ["no"|"off"], [
      AC_MSG_RESULT([no])
      AC_DEFINE([TORRENT_USE_INVARIANT_CHECKS],[0],[Define to disable internal invariant checks. Asserts are still enabled while debug is on.])
    ],
  ["full"], [
      AC_MSG_RESULT([full])
      AC_DEFINE([TORRENT_USE_INVARIANT_CHECKS],[1],[Define to enable internal invariant checks.])
      AC_DEFINE([TORRENT_EXPENSIVE_INVARIANT_CHECKS],[1],[Define to enable extra expensive invariant checks.]),
    ],
  [AC_MSG_RESULT([$ARG_ENABLE_INVARIANT])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_INVARIANT". Use either "yes", "no" or "full".])]
)

AC_MSG_CHECKING([whether logging to disk should be enabled])
AS_CASE(["$ARG_ENABLE_LOGGING"],
  ["yes"|"default"], [
      AC_MSG_RESULT([yes])
    ],
  ["no"|"none"], [
      AC_MSG_RESULT([no])
      AC_DEFINE([TORRENT_DISABLE_LOGGING],[1],[Define to disable support for logging alerts])
      COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DTORRENT_DISABLE_LOGGING "
    ],
  [AC_MSG_RESULT([$ARG_ENABLE_LOGGING])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_LOGGING". Use either "yes" or "no"])]
)

AS_ECHO
AS_ECHO "Checking features to be enabled:"

AC_MSG_CHECKING([whether encryption support should be enabled])
AS_CASE(["$ARG_ENABLE_ENCRYPTION"],
  ["yes"|"on"], [
      AC_MSG_RESULT([yes])
      AC_MSG_NOTICE([encryption support: now checking for the OpenSSL library...])

      AX_CHECK_OPENSSL([
          AC_DEFINE([TORRENT_USE_OPENSSL],[1],[Define to use OpenSSL support.])
          AC_DEFINE([TORRENT_USE_LIBCRYPTO],[1],[Define to use libcrypto from OpenSSL.])
          COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DTORRENT_USE_OPENSSL -DTORRENT_USE_LIBCRYPTO "
        ], [
          AC_MSG_ERROR([OpenSSL library not found. Try using --with-openssl=DIR or disabling encryption at all.])
      ])
    ],
  ["no"|"off"], [
      AC_MSG_RESULT([no])
      AC_DEFINE([TORRENT_DISABLE_ENCRYPTION],[1],[Define to disable any encryption support and avoid linking against OpenSSL.])
      COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DTORRENT_DISABLE_ENCRYPTION "
    ],
  [AC_MSG_RESULT([$ARG_ENABLE_ENCRYPTION])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_ENCRYPTION". Use either "yes" or "no".])]
)

AC_MSG_CHECKING([whether dht support should be enabled])
AS_CASE(["$ARG_ENABLE_DHT"],
  ["yes"|"on"], [
      AC_MSG_RESULT([yes])
    ],
  ["no"|"off"], [
      AC_MSG_RESULT([no])
      AC_DEFINE([TORRENT_DISABLE_DHT],[1],[Define to disable the DHT support.])
      COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DTORRENT_DISABLE_DHT "
    ],
  [AC_MSG_RESULT([$ARG_ENABLE_DHT])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_DHT". Use either "yes" or "no".])]
)

AS_IF([test "x$ac_cv_hidden_visibility_attribute" = "xyes"], [
  AS_IF([test "x$ARG_ENABLE_FULL_EXPORT" = "xno"], [
     CXXFLAGS="$CXXFLAGS -fvisibility=hidden -fvisibility-inlines-hidden"
     CFLAGS="$CFLAGS -fvisibility=hidden"
     LDFLAGS="$LDFLAGS -fvisibility=hidden -fvisibility-inlines-hidden"
  ])
])

AS_ECHO
AS_ECHO "Checking for extra build files:"

AC_MSG_CHECKING([whether example files should be built])
AS_CASE(["$ARG_ENABLE_EXAMPLES"],
  ["yes"], [AC_MSG_RESULT([yes])],
  ["no"],  [AC_MSG_RESULT([no])],
  [AC_MSG_RESULT([$ARG_ENABLE_EXAMPLES])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_EXAMPLES". Use either "yes" or "no".])]
)

AC_MSG_CHECKING([whether test files should be built])
AS_CASE(["$ARG_ENABLE_TESTS"],
  ["yes"], [
      AC_MSG_RESULT([yes])
      AC_DEFINE([TORRENT_EXPORT_EXTRA],[1],[Define to export additional symbols for unit tests.])
      COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DTORRENT_EXPORT_EXTRA "
    ],
  ["no"],  [AC_MSG_RESULT([no])],
  [AC_MSG_RESULT([$ARG_ENABLE_TESTS])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_TESTS". Use either "yes" or "no".])]
)

AC_MSG_CHECKING([whether python bindings should be built])
AS_CASE(["$ARG_ENABLE_PYTHON_BINDING"],
  ["yes"], [
      AC_MSG_RESULT([yes])

      AM_PATH_PYTHON([2.4], [], AC_MSG_ERROR([Python interpreter not found.]))
      AX_PYTHON_DEVEL([>= '2.4'])
      AX_BOOST_PYTHON()

      AS_IF([test -z "$BOOST_PYTHON_LIB"],
            [AC_MSG_ERROR([Boost.Python library not found. Try using --with-boost-python=lib.])])

      BOOST_PYTHON_LIB="-l$BOOST_PYTHON_LIB"
    ],
  ["no"], [
      AC_MSG_RESULT([no])
    ],
  [AC_MSG_RESULT([$ARG_ENABLE_PYTHON_BINDING])
   AC_MSG_ERROR([Unknown option "$ARG_ENABLE_PYTHON_BINDING". Use either "yes" or "no".])]
)

AS_ECHO
AS_ECHO "Checking for external libraries:"

AC_MSG_CHECKING([whether to link against system libiconv])
AS_CASE(["$ARG_WITH_LIBICONV"],
  ["yes"], [
      AC_MSG_RESULT([yes])
      AM_ICONV()
      AS_IF([test "x$am_cv_func_iconv" = "xyes"], [
           ICONV_LIBS=$LIBICONV
           AC_SUBST([ICONV_LIBS])
           LIBS="$ICONV_LIBS $LIBS"
      ], [
           AC_MSG_ERROR([Iconv library not found.])
      ])
   ],
  ["no"], [
        ARG_WITH_LIBICONV="no"
   ],
  [AC_MSG_RESULT([$ARG_WITH_LIBICONV])
   AC_MSG_ERROR([Unknown option "$ARG_WITH_LIBICONV". Use either "yes" or "no".])]
)

###############################################################################
# Setting conditional variables for Makefiles
###############################################################################

AM_CONDITIONAL([ENABLE_DHT], [test "x$ARG_ENABLE_DHT" = "xyes" -o "x$ARG_ENABLE_DHT" = "xlogging" ])
AM_CONDITIONAL([ENABLE_EXAMPLES], [test "x$ARG_ENABLE_EXAMPLES" = "xyes"])
AM_CONDITIONAL([ENABLE_TESTS], [test "x$ARG_ENABLE_TESTS" = "xyes"])
AM_CONDITIONAL([ENABLE_PYTHON_BINDING], [test "x$ARG_ENABLE_PYTHON_BINDING" = "xyes"])

AM_CONDITIONAL([WITH_OPENSSL], [test "x$ARG_ENABLE_ENCRYPTION" = "xyes" -o "x$ARG_ENABLE_ENCRYPTION" = "xon" ])


###############################################################################
# Other useful stuff
###############################################################################

AC_DEFINE([BOOST_ASIO_HAS_STD_CHRONO],[1],[Define to make sure asio uses std::chrono.])
COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DBOOST_ASIO_HAS_STD_CHRONO=1 "

AC_DEFINE([BOOST_EXCEPTION_DISABLE],[1],[Define to disable the boost.exception features.])
COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DBOOST_EXCEPTION_DISABLE "

AC_DEFINE([BOOST_ASIO_ENABLE_CANCELIO],[1],[Define to enable cancel support in asio on windows XP and older.])
COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DBOOST_ASIO_ENABLE_CANCELIO "

dnl Use possibly specific python install params
AC_ARG_VAR([PYTHON_INSTALL_PARAMS], [Set specific install parameters for python bindings.])
AS_IF([test "x$PYTHON_INSTALL_PARAMS" = "x"],
      [PYTHON_INSTALL_PARAMS='--prefix=$(DESTDIR)$(prefix)'])

dnl Set some defines if we are building a shared library
AS_IF([test "x$enable_shared" = "xyes"],
      [AC_DEFINE([TORRENT_BUILDING_SHARED],[1],[Define to exports functions and classes with default visibility in GCC.])
       COMPILETIME_OPTIONS="$COMPILETIME_OPTIONS -DTORRENT_LINKING_SHARED "])

AC_SUBST(DEBUGFLAGS)
AC_SUBST(PYTHON_INSTALL_PARAMS)
AC_SUBST(COMPILETIME_OPTIONS)

# Disable deprecated warnings for the Python binding builds.
PYTHON_CXXFLAGS="${PYTHON_CXXFLAGS} -Wno-deprecated-declarations"
AC_SUBST(PYTHON_CXXFLAGS)

# Try to guess real git revision if any, fallback to hardcoded otherwise
GIT_REVISION=`git log -1 --format=format:%h 2>/dev/null`
AS_IF([test -z "$GIT_REVISION"],
      [GIT_REVISION=`sed -n -e "s/^#define LIBTORRENT_REVISION \"\([0-9a-z]*\)\"$/\1/p" $(dirname $0)/include/libtorrent/version.hpp`])


###############################################################################
# Generating Makefiles
###############################################################################

AS_ECHO
AS_ECHO "Generating Makefiles:"

AC_CONFIG_FILES(
  [Makefile]
  [src/Makefile]
  [include/libtorrent/Makefile]
  [examples/Makefile]
  [test/Makefile]
  [tools/Makefile]
  [bindings/Makefile]
  [bindings/python/Makefile]
  [bindings/python/link_flags]
  [bindings/python/compile_flags]
  [bindings/python/compile_cmd]
  [libtorrent-rasterbar.pc]
)

AC_OUTPUT


###############################################################################
# Generating config.report
###############################################################################

AS_ECHO
AS_ECHO "Configure script has finished system check."
AS_ECHO

cat > config.report << END
   Config results:
  -=-=-=-=-=-=-=-=-

Package:
  name:                 ${PACKAGE_NAME}
  version:              ${PACKAGE_VERSION}
  git revision:         ${GIT_REVISION}

Build environment:
  build system:         ${build}
  host system:          ${host}
  target system:        ${target}

Compiler and linker flags:
  CPPFlags:             ${CPPFLAGS}
  CFlags:               ${CFLAGS}
  CXXFlags:             ${CXXFLAGS}
  LDFlags:              ${LDFLAGS}
  Libs:                 ${LIBS}
  Defs:                 ${DEFS}

Build options:
  deprecated functions: ${ARG_ENABLE_DEPRECATED:-yes}
  debug build:          ${ARG_ENABLE_DEBUG:-no}
  invariant checks:     ${ARG_ENABLE_INVARIANT:-no}
  logging support:      ${ARG_ENABLE_LOGGING:-yes}

Features:
  encryption support:   ${ARG_ENABLE_ENCRYPTION:-yes}
  dht support:          ${ARG_ENABLE_DHT:-yes}

Extra builds:
  examples:             ${ARG_ENABLE_EXAMPLES:-no}
  tests:                ${ARG_ENABLE_TESTS:-no}
  python bindings:      ${ARG_ENABLE_PYTHON_BINDING:-no}

Pthread library:
  CFlags:               ${PTHREAD_CFLAGS}
  Libs:                 ${PTHREAD_LIBS}

Boost libraries:
  version:              ${BOOST_VERSION}
  CPPFlags:             ${BOOST_CPPFLAGS}
  LDFlags:              ${BOOST_LDFLAGS}
  boost.system:         ${BOOST_SYSTEM_LIB}
END

AS_IF([test "x$ARG_ENABLE_PYTHON_BINDING" = "xyes"], [
cat >> config.report << END
  boost.python:         ${BOOST_PYTHON_LIB}

Python environment:
  -automake-
  binary:               ${PYTHON}
  version:              ${PYTHON_VERSION}
  platform:             ${PYTHON_PLATFORM}
  prefix:               ${PYTHON_PREFIX}
  exec_prefix:          ${PYTHON_EXEC_PREFIX}
  pythondir:            ${pythondir}
  pkgpythondir:         ${pkgpythondir}
  pyexecdir:            ${pyexecdir}
  pkgpyexecdir:         ${pkgpyexecdir}

  -m4-
  cppflags:             ${PYTHON_CPPFLAGS}
  ldflags:              ${PYTHON_LDFLAGS}
  extra libs:           ${PYTHON_EXTRA_LIBS}

END
])

cat >> config.report << END

External libraries:
  system libiconv:      ${ARG_WITH_LIBICONV:-no}
END

AS_IF([test "x$ARG_WITH_LIBICONV" = "xyes"], [
cat >> config.report << END

Iconv library:
  Iconv Libs:           ${ICONV_LIBS}
END
])

AS_IF([test "x$ARG_ENABLE_ENCRYPTION" = "xyes"], [
cat >> config.report << END

OpenSSL library:
  OpenSSL Libs:         ${OPENSSL_LIBS}
  OpenSSL LDFlags:      ${OPENSSL_LDFLAGS}
  OpenSSL Includes:     ${OPENSSL_INCLUDES}
END
])

cat config.report

AS_ECHO
AS_ECHO "Type 'make' to compile $PACKAGE_STRING"
AS_ECHO "or type 'make V=1' for verbose compiling"
AS_ECHO "and then 'make install' to install it into $prefix"
AS_ECHO
