# CMake build for the Apache APR library.
# SEE WARNING BELOW.
#
# Copyright (C) 2012 LuaDist.
# Created by David Manura (CMakeLists.txt).
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.
# Note: Based partly on configure.in and apr.h.in in Apache APR (which has its own
# copyright--see LICENSE).

project ( apr C )

cmake_minimum_required ( VERSION 2.8 )
include ( dist.cmake )

message ( WARNING
"
WARNING!!!
THE CMAKE BUILD FOR THIS LIBRARY IS PRELIMINARY.
IT LIKELY WON'T WORK AS IS.
PLEASE FIX ANY PROBLEMS YOU FIND.
It was adapted from the autoconf build,
which is fairly complex.
")

include ( CheckIncludeFile )
include ( CheckFunctionExists )

macro ( my_check_include_file path var )
  check_include_file ( ${path} ${var} )
  if ( NOT ${var} )
    set ( ${var} 0 )
  endif ()
  # message ( "${path} ${var} ${${var}}" )
endmacro ()

# from configure.in & apr.h.in
macro ( APR_FLAG_HEADERS )
  foreach ( path ${ARGV} )
    string ( REPLACE "." "" _var "${path}" )
    string ( REPLACE "/" "_" _var "${_var}" )
    my_check_include_file ( ${path} ${_var} )
  endforeach ()
endmacro ()
APR_FLAG_HEADERS (
    ByteOrder.h
    conio.h
    crypt.h
    ctype.h
    dir.h
    dirent.h
    dl.h
    dlfcn.h
    errno.h
    fcntl.h
    grp.h
    io.h
    limits.h
    mach-o/dyld.h
    malloc.h
    memory.h
    netdb.h
    osreldate.h
    poll.h
    process.h
    pwd.h
    semaphore.h
    signal.h
    stdarg.h
    stddef.h
    stdio.h
    stdlib.h
    string.h
    strings.h
    sysapi.h
    sysgtime.h
    termios.h
    time.h
    tpfeq.h
    tpfio.h
    unistd.h
    unix.h
    windows.h
    winsock2.h
    arpa/inet.h
    kernel/OS.h
    net/errno.h
    netinet/in.h
    netinet/sctp.h
    netinet/sctp_uio.h
    sys/file.h
    sys/ioctl.h
    sys/mman.h
    sys/param.h
    sys/poll.h
    sys/resource.h
    sys/select.h
    sys/sem.h
    sys/sendfile.h
    sys/signal.h
    sys/socket.h
    sys/sockio.h
    sys/stat.h
    sys/sysctl.h
    sys/syslimits.h
    sys/time.h
    sys/types.h
    sys/uio.h
    sys/un.h
    sys/wait.h)
APR_FLAG_HEADERS (  netinet/tcp.h ) # TODO: IRIX (see configure.in)

# improve? see configure.in
my_check_include_file ( stdint.h stdint )

set ( voidp_size ${CMAKE_SIZEOF_VOID_P} )

# from configure.in
# Checks for integer size
include ( CheckTypeSize )
check_type_size ( char _sizeof_char )
check_type_size ( int _sizeof_int )
check_type_size ( long _sizeof_log )
check_type_size ( short _sizeof_short )
check_type_size ( "long long" _sizeof_long_long )
if ( _sizeof_short EQUAL 2 )
  set ( short_value short )
endif ()
if ( _sizeof_int EQUAL 4 )
  set ( int_value int )
endif ()
if ( _sizeof_int EQUAL 8 )
  set ( int64_literal "#define APR_INT64_C(val) (val)" )
  set ( uint64_literal "#define APR_UINT64_C(val) (val##U)" )
  set ( int64_t_fmt "#define APR_INT64_T_FMT \"d\"" )
  set ( uint64_t_fmt "#define APR_UINT64_T_FMT \"u\"" )
  set ( uint64_t_hex_fmt "#define APR_UINT64_T_HEX_FMT \"x\"" )
  set ( int64_value "int" )
  set ( long_value int )
  set ( int64_strfn "strtoi" )
elseif ( _sizeof_long EQUAL 8 )
  set ( int64_literal "#define APR_INT64_C(val) (val##L)" )
  set ( int64_literal "#define APR_UINT64_C(val) (val##UL)" )
  set ( int64_t_fmt "#define APR_INT64_T_FMT \"ld\"" )
  set ( uint64_t_fmt "#define APR_UINT64_T_FMT \"lu\"" )
  set ( uint64_t_hex_fmt "#define APR_UINT64_T_HEX_FMT \"lx\"" )
  set ( int64_value "long" )
  set ( long_value long )
  set ( int64_strfn "strtol" )
elseif ( _sizeof_long_long EQUAL 8 ) 
  set ( int64_literal "#define APR_INT64_C(val) (val##LL)" )
  set ( uint64_literal "#define APR_UINT64_C(val) (val##ULL)" )
  set ( int64_t_fmt "#define APR_INT64_T_FMT \"lld\"" )
  set ( uint64_t_fmt "#define APR_UINT64_T_FMT \"llu\"" )
  set ( uint64_t_hex_fmt "#define APR_UINT64_T_HEX_FMT \"llx\"" )
  set ( int64_value "long long" )
  set ( long_value "long long" )
  set ( int64_strfn "strtoll" )
# FIX? elseif ( _sizeof_longlong EQUAL 8 )
# set ( int64_literal "#define APR_INT64_C(val) (val##LL)" )
# set ( uint64_literal "#define APR_UINT64_C(val) (val##ULL)" )
# set ( int64_t_fmt "#define APR_INT64_T_FMT \"qd\"" )
# set ( uint64_t_fmt "#define APR_UINT64_T_FMT \"qu"" )
# set ( uint64_t_hex_fmt "#define APR_UINT64_T_HEX_FMT \"qx\"" )
# set ( int64_value "__int64" )
# set ( long_value "__int64" )
# set ( int64_strfn "strtoll" )
else ()
  message ( FATAL "could not detect a 64-bit integer type" )
endif ()


# from configure.in
macro ( my_check_function_exists func var )
  check_function_exists ( ${func} ${var} )
  if ( NOT ${var} )
    set ( ${var} 0 )
  endif ()
endmacro ()
my_check_function_exists ( strnicmp have_strnicmp )
my_check_function_exists ( strncasecmp have_strncasecmp )
my_check_function_exists ( stricmp have_stricmp )
my_check_function_exists ( strcasecmp have_strcasecmp )
my_check_function_exists ( strdup have_strdup )
my_check_function_exists ( strstr have_strstr )
my_check_function_exists ( memchr have_memchr )
#my_check_function_exists ( $int64_strfn, have_int64_strfn="1", have_int64_strfn="0" )

my_check_function_exists ( memmove have_memmove )
message("HM: ${have_memmove}")
check_function_exists ( getpass HAVE_GETPASS )
check_function_exists ( getpassphrase HAVE_GETPASSPHRASE )
check_function_exists ( gmtime_r HAVE_GMTIME_R )
check_function_exists ( localtime_r HAVE_LOCALTIME_R )
check_function_exists ( mkstemp HAVE_MKSTEMP )

set ( rand 0 ) # TOOD: see configure.in

# TODO: see configure.in
set ( threads 0 )
set ( pthreadh 0 )
set ( pthreadser 0 )

# from apr.h.in
#TODO:improve
set ( havemmaptmp 0 )
set ( havemmapshm 0 )
set ( havemmapzero 0 )
set ( haveshmgetanon 0 )
set ( haveshmget 0)
set ( havemmapanon 0)
set ( havebeosarea 0)
set ( usemmaptmp 0)
set ( usemmapshm 0)
set ( usemmapzero 0)
set ( useshmgetanon 0)
set ( useshmget 0)
set ( usemmapanon 0)
set ( usebeosarea 0)
set ( flockser 0) 
set ( sysvser 0)
set ( posixser 0)
set ( fcntlser 0)
set ( procpthreadser 0)
set ( pthreadser 0)
set ( hasflockser 0)
set ( hassysvser 0)
set ( hasposixser 0)
set ( hasfcntlser 0)
set ( hasprocpthreadser 0)
set ( proclockglobal 0)
set ( have_corkable_tcp 0)
set ( have_getrlimit 0)
set ( have_in_addr 1)
set ( have_inet_addr 0)
set ( have_inet_network 0)
set ( have_ipv6 0)
# see above: set ( have_memmove 0)
set ( have_setrlimit 0)
set ( have_sigaction 0)
set ( have_sigsuspend 0)
set ( have_sigwait 0)
set ( have_sa_storage 0)
set ( have_strcasecmp 0)
set ( have_strdup 0)
set ( have_stricmp 0)
set ( have_strncasecmp 0)
set ( have_strnicmp 0)
set ( have_strstr 0)
set ( have_memchr 0)
set ( struct_rlimit 0)
set ( have_union_semun 0)
set ( have_sctp 0)
set ( have_iovec 0)

#FIX
set ( apr_tcp_nopush_flag TCP_CORK )
set ( tcp_nodelay_inherited 1 )
set ( o_nonblock_inherited 0 )

#TODO:FIX
set ( proc_mutex_is_global 0 )

# FIX
set ( file_as_socket 0 )

include ( TestBigEndian )
test_big_endian ( _big_endian )
if ( _big_endian )
  set ( bigendian 1 )
else ()
  set ( bigendian 0 )
endif ()

#TODO (configure.in)
#set ( off_t_fmt "#define APR_OFF_T_FMT APR_INT64_T_FMT" )
#set ( off_t_value "off64_t" )
#set ( off_t_strfn "apr_strtoi64" )
set ( off_t_value long )
set ( off_t_fmt "#define APR_OFF_T_FMT \"ld\"" )
set ( off_t_strfn "strtol" )

# from configure.in
include ( CheckTypeSize )
set ( CMAKE_EXTRA_INCLUDE_FILES sys/types.h sys/uio.h )
check_type_size ( "struct iovec" _iovec )
if ( HAVE__iovec )
  set ( have_iovec 1 )
else ()
  set ( have_iovec 0 )
endif ()
set ( CMAKE_EXTRA_INCLUDE_FILES )

# APR Feature Macros (from configure.in / apr.h.in)
# TODO: improve
set ( sharedmem 0 )
set ( threads 0 )
set ( sendfile 0 )
set ( mmap 0 )
set ( fork 0 )
set ( rand 0 )
set ( oc 0 )
set ( aprdso 0 )
set ( acceptfilter 0 )
set ( have_unicode_fs 0 )
set ( have_proc_invoked 0 )
set ( apr_has_user 1 )
set ( aprlfs 0 )
set ( apr_has_xthread_files 0 )
set ( osuuid 0 )
set ( apr_procattr_user_set_requires_password 0 )
set ( apr_thread_func "" )

# TODO: improve
set ( apr_charset_ebcdic 0 )

configure_file ( ${CMAKE_CURRENT_SOURCE_DIR}/include/arch/unix/apr_private.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/apr_private.h )
configure_file ( ${CMAKE_CURRENT_SOURCE_DIR}/include/apr.h.in ${CMAKE_CURRENT_BINARY_DIR}/apr.h @ONLY )

include_directories ( ${CMAKE_CURRENT_BINARY_DIR} )
include_directories ( ${CMAKE_CURRENT_SOURCE_DIR}/include )
include_directories ( ${CMAKE_CURRENT_SOURCE_DIR}/include/arch/unix )

# based on build-outputs.mk, in turn generated by gen-build.py

set ( SRC_all
  passwd/apr_getpass.c
  #FIX strings/apr_cpystrn.c
  strings/apr_fnmatch.c
  strings/apr_snprintf.c
  strings/apr_strings.c
  strings/apr_strnatcmp.c
  strings/apr_strtok.c
  tables/apr_hash.c
  tables/apr_tables.c
)

set ( SRC_dso_unix
  dso/unix/dso.c
)

set ( SRC_file_io_unix
  file_io/unix/buffer.c
  file_io/unix/copy.c
  file_io/unix/dir.c
  file_io/unix/fileacc.c
  file_io/unix/filedup.c
  file_io/unix/filepath.c
  file_io/unix/filepath_util.c
  file_io/unix/filestat.c
  #FIX file_io/unix/flock.c
  file_io/unix/fullrw.c
  file_io/unix/mktemp.c
  file_io/unix/open.c
  file_io/unix/pipe.c
  file_io/unix/readwrite.c
  file_io/unix/seek.c
  file_io/unix/tempdir.c
)


set ( SRC_locks_unix
  locks/unix/global_mutex.c
  locks/unix/proc_mutex.c
  locks/unix/thread_cond.c
  locks/unix/thread_mutex.c
  locks/unix/thread_rwlock.c
)

set ( SRC_memory_unix
  memory/unix/apr_pools.c
)

set ( SRC_misc_unix
  misc/unix/charset.c
  misc/unix/env.c
  misc/unix/errorcodes.c
  misc/unix/getopt.c
  misc/unix/otherchild.c
  misc/unix/rand.c
  misc/unix/start.c
  misc/unix/version.c
)

set ( SRC_mmap_unix
  mmap/unix/common.c
  mmap/unix/mmap.c
)

set ( SRC_network_io_unix
  network_io/unix/inet_ntop.c
  network_io/unix/inet_pton.c
  network_io/unix/multicast.c
  network_io/unix/sendrecv.c
  network_io/unix/sockaddr.c
  network_io/unix/socket_util.c
  network_io/unix/sockets.c
  network_io/unix/sockopt.c
)

set ( SRC_poll_unix
  poll/unix/epoll.c
  poll/unix/kqueue.c
  poll/unix/poll.c
  poll/unix/pollcb.c
  poll/unix/pollset.c
  poll/unix/port.c
  poll/unix/select.c
)

set ( SRC_random_unix
  random/unix/apr_random.c
  random/unix/sha2.c
  random/unix/sha2_glue.c
)

set ( SRC_shmem_unix
  shmem/unix/shm.c
)

set ( SRC_support_unix
  support/unix/waitio.c
)

set ( SRC_threadproc_unix
  threadproc/unix/proc.c
  threadproc/unix/procsup.c
  threadproc/unix/signals.c
  threadproc/unix/thread.c
  threadproc/unix/threadpriv.c
)

set ( SRC_time_unix
  #FIX time/unix/time.c
  #FIX time/unix/timestr.c
)

set ( SRC_user_unix
  #FIX user/unix/groupinfo.c
  #FIX user/unix/userinfo.c
)

set ( SRC_atomic_unix
  atomic/unix/builtins.c
  atomic/unix/ia32.c
  atomic/unix/mutex.c
  atomic/unix/ppc.c
  atomic/unix/s390.c
  atomic/unix/solaris.c
)

set ( SRC_unix
  ${SRC_all} ${SRC_atomic_unix} ${SRC_dso_unix}
  ${SRC_fix} ${SRC_locks_unix} ${SRC_memory_unix}
  ${SRC_misc_unix} ${SRC_mmap_unix} ${SRC_network_io_unix}
  ${SRC_poll_unix} ${SRC_random_unix} ${SRC_shmem_unix}
  ${SRC_support_unix} ${SRC_threadproc_unix} ${SRC_time_unix} ${SRC_user_unix}
)

set ( SRC_dso_aix
  dso/aix/dso.c
)

set ( SRC_aix
  ${SRC_all} ${SRC_atomic_unix} ${SRC_dso_aix} ${SRC_file_io_unix}
  ${SRC_locks_unix}
  ${SRC_memory_unix} ${SRC_misc_unix} ${SRC_mmap_unix}
  ${SRC_network_io_unix} ${SRC_poll_unix} ${SRC_random_unix}
  ${SRC_shmem_unix} ${SRC_support_unix} ${SRC_threadproc_unix}
  ${SRC_time_unix} ${SRC_user_unix}
)

set ( SRC_dso_beos
  dso/beos/dso.c
)

set ( SRC_locks_beos
  locks/beos/proc_mutex.c
  locks/beos/thread_cond.c
  locks/beos/thread_mutex.c
  locks/beos/thread_rwlock.c
)

set ( SRC_network_io_beos
  network_io/beos/sendrecv.c
  network_io/beos/socketcommon.c
)

set ( SRC_shmem_beos
  shmem/beos/shm.c
)

set ( SRC_threadproc_beos
  threadproc/beos/apr_proc_stub.c
  threadproc/beos/proc.c
  threadproc/beos/thread.c
  threadproc/beos/threadpriv.c
  threadproc/beos/threadproc_common.c
)

set ( SRC_beos
  ${SRC_all} ${SRC_atomic_unix} ${SRC_dso_beos} ${SRC_file_io_unix}
  ${SRC_locks_beos} ${SRC_memory_unix} ${SRC_misc_unix}
  ${SRC_mmap_unix} ${SRC_network_io_beos} ${SRC_poll_unix}
  ${SRC_random_unix} ${SRC_shmem_beos} ${SRC_support_unix}
  ${SRC_threadproc_beos} ${SRC_time_unix} ${SRC_user_unix}
)

set ( SRC_dso_os2
  dso/os2/dso.c
)

set ( SRC_file_io_os2
  file_io/os2/buffer.c
  file_io/os2/copy.c
  file_io/os2/dir.c
  file_io/os2/dir_make_recurse.c
  file_io/os2/fileacc.c
  file_io/os2/filedup.c
  file_io/os2/filepath.c
  file_io/os2/filepath_util.c
  file_io/os2/filestat.c
  file_io/os2/filesys.c
  file_io/os2/flock.c
  file_io/os2/fullrw.c
  file_io/os2/maperrorcode.c
  file_io/os2/mktemp.c
  file_io/os2/open.c
  file_io/os2/pipe.c
  file_io/os2/readwrite.c
  file_io/os2/seek.c
  file_io/os2/tempdir.c
)

set ( SRC_locks_os2
  locks/os2/proc_mutex.c
  locks/os2/thread_cond.c
  locks/os2/thread_mutex.c
  locks/os2/thread_rwlock.c
)

set ( SRC_network_io_os2
  network_io/os2/inet_ntop.c
  network_io/os2/inet_pton.c
  network_io/os2/os2calls.c
  network_io/os2/sendrecv.c
  network_io/os2/sendrecv_udp.c
  network_io/os2/sockaddr.c
  network_io/os2/socket_util.c
  network_io/os2/sockets.c
  network_io/os2/sockopt.c
)

set ( SRC_poll_os2
  poll/os2/poll.c poll/os2/pollset.c
)

set ( SRC_shmem_os2
  shmem/os2/shm.c
)

set ( SRC_threadproc_os2
  threadproc/os2/proc.c
  threadproc/os2/signals.c
  threadproc/os2/thread.c
  threadproc/os2/threadpriv.c
)

set ( SRC_os2
  ${SRC_all} ${SRC_atomic_unix} ${SRC_dso_os2} ${SRC_file_io_os2}
  ${SRC_locks_os2} ${SRC_memory_unix} ${SRC_misc_unix} 
  ${SRC_mmap_unix} ${SRC_network_io_os2} ${SRC_poll_os2}
  ${SRC_random_unix} ${SRC_shmem_os2} ${SRC_support_unix}
  ${SRC_threadproc_os2} ${SRC_time_unix} ${SRC_user_unix}
)

set ( SRC_dso_os390
  dso/os390/dso.c
)

set ( SRC_atomic_os390
  atomic/os390/atomic.c
)

set ( SRC_os390
  ${SRC_all} ${SRC_atomic_os390} ${SRC_dso_os390} ${SRC_file_io_unix}
  ${SRC_locks_unix} ${SRC_memory_unix} ${SRC_misc_unix}
  ${SRC_mmap_unix} ${SRC_network_io_unix} ${SRC_poll_unix}
  ${SRC_random_unix} ${SRC_shmem_unix} ${SRC_support_unix}
  ${SRC_threadproc_unix} ${SRC_time_unix} ${SRC_user_unix}
)

set ( SRC_dso_win32
  dso/win32/dso.c
)

set ( SRC_file_io_win32
  file_io/unix/copy.c
  file_io/unix/fileacc.c
  file_io/unix/filepath_util.c
  file_io/unix/fullrw.c
  file_io/unix/mktemp.c
  file_io/unix/tempdir.c
  file_io/win32/buffer.c
  file_io/win32/dir.c
  file_io/win32/filedup.c
  file_io/win32/filepath.c
  file_io/win32/filestat.c
  file_io/win32/filesys.c
  file_io/win32/flock.c
  file_io/win32/open.c
  file_io/win32/pipe.c
  file_io/win32/readwrite.c
  file_io/win32/seek.c
)

set ( SRC_locks_win32
  locks/win32/proc_mutex.c
  locks/win32/thread_cond.c
  locks/win32/thread_mutex.c
  locks/win32/thread_rwlock.c
)

set ( SRC_misc_win32
  misc/unix/errorcodes.c
  misc/unix/getopt.c
  misc/unix/otherchild.c
  misc/unix/version.c
  misc/win32/charset.c
  misc/win32/env.c
  misc/win32/internal.c
  misc/win32/misc.c
  misc/win32/rand.c
  misc/win32/start.c
  misc/win32/utf8.c
)

set ( SRC_mmap_win32
  mmap/unix/common.c
  mmap/win32/mmap.c
)

set ( SRC_network_io_win32
  network_io/unix/inet_ntop.c
  network_io/unix/inet_pton.c
  network_io/unix/multicast.c
  network_io/unix/sockaddr.c
  network_io/unix/socket_util.c
  network_io/win32/sendrecv.c
  network_io/win32/sockets.c
  network_io/win32/sockopt.c
)

set ( SRC_shmem_win32
  shmem/win32/shm.c
)

set ( SRC_threadproc_win32
  threadproc/win32/proc.c
  threadproc/win32/signals.c
  threadproc/win32/thread.c
  threadproc/win32/threadpriv.c
)

set ( SRC_time_win32
  time/win32/time.c
  time/win32/timestr.c
)

set ( SRC_user_win32
  user/win32/groupinfo.c
  user/win32/userinfo.c
)

set ( SRC_atomic_win32
  atomic/win32/apr_atomic.c
)

set ( SRC_win32
  ${SRC_all} ${SRC_atomic_win32} ${SRC_dso_win32} ${SRC_file_io_win32}
  ${SRC_locks_win32} ${SRC_memory_unix} ${SRC_misc_win32}
  ${SRC_mmap_win32} ${SRC_network_io_win32} ${SRC_poll_unix}
  ${SRC_random_unix} ${SRC_shmem_win32} ${SRC_support_unix}
  ${SRC_threadproc_win32} ${SRC_time_win32} ${SRC_user_win32}
)

#set ( SOURCE_DIRS random/unix misc/win32 mmap/win32 dso/os2
#    time/unix network_io/win32 dso/win32 locks/unix user/unix
#    time/win32 locks/beos tables support/unix file_io/unix mmap/unix
#    atomic/unix threadproc/win32 poll/os2 atomic/win32 dso/os390
#    atomic/os390 dso/beos poll/unix passwd network_io/beos threadproc/os2
#    network_io/os2 shmem/win32 threadproc/beos shmem/unix network_io/unix
#    file_io/os2 dso/aix file_io/win32 threadproc/unix misc/unix locks/win32
#    shmem/beos dso/unix locks/os2 user/win32 shmem/os2
#    memory/unix strings ${EXTRA_SOURCE_DIRS} )
#set ( BUILD_DIRS atomic atomic/os390 atomic/unix atomic/win32 dso
#    dso/aix dso/beos dso/os2 dso/os390 dso/unix dso/win32 file_io
#    file_io/os2 file_io/unix file_io/win32 locks locks/beos locks/os2
#    locks/unix locks/win32 memory memory/unix misc misc/unix misc/win32
#    mmap mmap/unix mmap/win32 network_io network_io/beos network_io/os2
#    network_io/unix network_io/win32 passwd poll poll/os2 poll/unix random
#    random/unix shmem shmem/beos shmem/os2 shmem/unix shmem/win32
#    strings support support/unix tables threadproc threadproc/beos
#    threadproc/os2 threadproc/unix threadproc/win32 time time/unix
#    time/win32 user user/unix user/win32 )

if ( WIN32 )
  set (SRC ${SRC_win32} )
else ()
  set (SRC ${SRC_unix} )
endif ()
# FIX: more

add_library( apr SHARED ${SRC} )

install_library( apr )
install_header ( ${CMAKE_CURRENT_BINARY_DIR}/apr.h )
install_doc( CHANGES README docs/)

