
AC_INIT(lnav, 0.7.0, lnav@googlegroups.com)
AC_CONFIG_SRCDIR([src/lnav.cc])
AM_INIT_AUTOMAKE([foreign subdir-objects])
AM_SILENT_RULES([yes])

AC_PREFIX_DEFAULT(/usr)

AC_CANONICAL_HOST

for defdir in /opt/local /usr/local /usr /; do
    if test -d "$defdir/include"; then
        CPPFLAGS="$CPPFLAGS -I$defdir/include"
    fi

    if test -d "$defdir/lib"; then
        LDFLAGS="$LDFLAGS -L$defdir/lib"
    fi
    if test -d "$defdir/lib64"; then
        LDFLAGS="$LDFLAGS -L$defdir/lib64"
    fi

    if test -d "$defdir/lib/x86_64-linux-gnu"; then
        LDFLAGS="$LDFLAGS -L$defdir/lib/x86_64-linux-gnu"
    fi
done

dnl abssrcdir is the absolute path to the source base (regardless of where
dnl you are building it)
case x$srcdir in
x/*)
	abssrcdir=$srcdir
	;;
*)
	abssrcdir=`pwd`/$srcdir
	;;
esac
AC_SUBST(abssrcdir)

AC_PROG_CXX

CPPFLAGS="$CPPFLAGS -D_ISOC99_SOURCE -D__STDC_LIMIT_MACROS"

# CFLAGS=`echo $CFLAGS | sed 's/-O2//g'`
# CXXFLAGS=`echo $CXXFLAGS | sed 's/-O2//g'`

AC_ARG_ENABLE([profiling],
              AS_HELP_STRING([--enable-profiling],
                             [Compile with gprof(1) profiling support]))

AC_MSG_CHECKING(gprof(4) profiling support)
if test x"${enable_profiling}" = x"yes" ; then
	CFLAGS="$CFLAGS -pg -gstabs"
	CXXFLAGS="$CXXFLAGS -pg -gstabs"
	LDFLAGS="$LDFLAGS -pg"
else
	enable_profiling=no
fi
AC_MSG_RESULT($enable_profiling)

AC_SUBST(CFLAGS_PG)

AC_PROG_INSTALL
AC_PROG_RANLIB
AC_PROG_LN_S
AC_PROG_MAKE_SET

AM_PATH_PYTHON([2.4])

AC_PATH_PROG(SQLITE3_CMD, [sqlite3])

if test x"$SQLITE3_CMD" = x""; then
    AC_MSG_ERROR([The sqlite3 command is required])
fi

AC_PATH_PROG(BZIP2_CMD, [bzip2])

AC_CHECK_SIZEOF(off_t)
AC_CHECK_SIZEOF(size_t)

AC_STRUCT_TIMEZONE

AC_SEARCH_LIBS(openpty, util)
AC_SEARCH_LIBS(gzseek, z, [], [AC_MSG_ERROR([libz required to build])])
AC_SEARCH_LIBS(BZ2_bzopen, bz2)
AC_SEARCH_LIBS(dlopen, dl)
AC_SEARCH_LIBS(backtrace, execinfo)

# Sometimes, curses depends on these libraries being linked in...
AC_SEARCH_LIBS(cur_term, tinfo)
AC_SEARCH_LIBS(Gpm_Open, gpm)

AC_CHECK_HEADERS(execinfo.h pty.h util.h zlib.h bzlib.h libutil.h sys/ttydefaults.h)

AX_WITH_CURSES

if test "x$ax_cv_curses" != xyes; then
    AC_MSG_ERROR([requires an X/Open-compatible Curses library with color])
fi

AX_PATH_LIB_PCRE([], [AC_MSG_ERROR([pcre required to build])])
AX_PATH_LIB_READLINE([], [AC_MSG_ERROR([readline required to build])])

AX_LIB_SQLITE3("3.0.0")

if test x"$SQLITE3_LIBS" = x""; then
    AC_MSG_ERROR([The sqlite3 developement package must be installed.])
fi

OLD_LIBS="$LIBS"
LIBS="$LIBS ${SQLITE3_LIBS}"
AC_CHECK_FUNC(sqlite3_stmt_readonly,
    AC_DEFINE([HAVE_SQLITE3_STMT_READONLY], [], [Have the sqlite3_stmt_readonly function]))
LIBS="$OLD_LIBS"

case "$host_os" in
    *)
	# AC_DEFINE([_XOPEN_SOURCE], [500], [Need pread])
	AC_DEFINE([_BSD_SOURCE], [1], [Need pread])
	;;
esac

ALL_LDFLAGS="$LDFLAGS $SQLITE3_LDFLAGS"

static_lib_list="libncurses.a libreadline.a libsqlite3.a libz.a libtinfo.a"
static_lib_list="$static_lib_list libpcre.a libpcrecpp.a libncursesw.a libbz2.a"
static_lib_list="$static_lib_list libgpm.a"

AC_ARG_ENABLE([static],
              AS_HELP_STRING([--disable-static],
                             [Disable static linking]))
if test x"${enable_static}" != x"no"; then
    case "$host_os" in
    darwin*)
        STATIC_LDFLAGS="$STATIC_LDFLAGS -Wl,-search_paths_first"
        ;;
    esac

    STATIC_LDFLAGS="$STATIC_LDFLAGS -L`pwd`/src/static-libs"
    # This is a hack to link against static libraries instead of shared
    # so that we can build a mostly statically link exe that can
    # be downloaded and used right away.  This is required for OS X and
    # will, hopefully, make a static binary that is compatible with
    # many different versions of Linux.
    mkdir -p src/static-libs
    rm -f src/static-libs/*.a
    for libflag in $ALL_LDFLAGS; do
        case $libflag in
	    -Lstatic-libs)
		;;
	    -L*)
                libdir=`echo $libflag | sed -e 's/-L//'`
                for slib in $static_lib_list; do
                    if test -e "$libdir/$slib"; then
                        ln -sf "$libdir/$slib" src/static-libs/.
                    fi
                done
		;;
        esac
    done
fi
AC_SUBST(STATIC_LDFLAGS)

AC_CONFIG_HEADERS([src/config.h])
AC_CONFIG_FILES([Makefile])
AC_CONFIG_FILES([TESTS_ENVIRONMENT])
AC_CONFIG_FILES([src/Makefile])
AC_CONFIG_FILES([test/Makefile])

AC_OUTPUT
