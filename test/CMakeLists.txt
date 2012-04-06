# based on Makefile.in

add_executable ( testlockperf testlockperf.c )

add_executable ( testmutexscope testmutexscope.c )

# testall

set ( SRC_TESTS
	testtime.c teststr.c testvsn.c testipsub.c testshm.c
	testmmap.c testud.c testtable.c testsleep.c testpools.c
	testfmt.c testfile.c testdir.c testfileinfo.c testrand.c
	testdso.c testoc.c testdup.c testsockets.c testproc.c
	testpoll.c testlock.c testsockopt.c testpipe.c testthread.c
	testhash.c testargs.c testnames.c testuser.c testpath.c
	testenv.c testprocmutex.c testfnmatch.c testatomic.c testflock.c
	testsock.c testglobalmutex.c teststrnatcmp.c testfilecopy.c
	testtemp.c testlfs.c testcond.c
)

add_executable ( testall abts.c testutil.c ${SRC_TESTS} )
target_link_libraries ( testall apr )

# OTHER_PROGRAMS
add_executable ( echod echod.c )
target_link_libraries ( echod apr )
add_executable ( sendfile sendfile.c )
add_executable ( sockperf sockperf.c )
target_link_libraries ( sockperf apr )

# TESTALL_COMPONENTS
add_executable ( globalmutexchild globalmutexchild.c )
target_link_libraries ( globalmutexchild apr )
add_library ( mod_test SHARED mod_test.c ) #FIX:ok?
target_link_libraries ( mod_test apr )
add_library ( libmod_test SHARED mod_test.c )
target_link_libraries ( libmod_test apr )
add_executable ( occhild occhild.c )
target_link_libraries ( occhild apr )
add_executable ( proc_child proc_child.c )
add_executable ( readchild readchild.c )
target_link_libraries ( readchild apr )
add_executable ( sockchild sockchild.c )
target_link_libraries ( sockchild apr )
add_executable ( testshmconsumer testshmconsumer.c )
add_executable ( testshmproducer testshmproducer.c )
add_executable ( tryread tryread.c )
target_link_libraries ( tryread apr )

add_test ( testlockperf testlockperf  )
add_test ( testmutexscope testmutexscope )
add_test ( testall testall )
