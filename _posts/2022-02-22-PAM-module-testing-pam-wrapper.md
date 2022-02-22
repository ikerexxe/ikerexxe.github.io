---
layout:     post
title:      "PAM pam module testing with pam_wrapper"
date:       2022-02-22 20:00:00 +0100
categories: idm
---

# Context

In general, most [PAM](hhttps://en.wikipedia.org/wiki/Pluggable_authentication_module) modules aren't thread-safe and they fail some way or another when used concurrently. In the specific case that I will define in this post, pam_keyinit was failing because the system call for modifying user and group IDs changes those attributes for the whole process, instead of for a single thread. This caused that any system call in execution from that process failed with `EINTR`. So recently, I codeveloped a [solution to make pam_keyinit thread-safe](https://github.com/linux-pam/linux-pam/pull/432).

Once I had the solution ready I wanted to check it and this had to be done by executing two actions concurrently. On the one hand I needed to block a thread execution in a system call. On the other hand, another thread had to open a session with pam_keyinit. The first part was easy, but the second involved creating a fully configured environment for the PAM module.

# pam_wrapper to the rescue

The setup of that environment can be avoided by using [pam_wrapper](https://cwrap.org/pam_wrapper.html), which in conjunction with [cmocka](https://cmocka.org/) and the pamtest library, can be used to easily test a PAM module. All that is needed is a test PAM stack file, the test code and some tweaks when running the test to preload pam_wrapper and configure it.

# PAM stack

Create a simple PAM stack file called `myapp`, that contains the session opening for `pam_keyinit.so`:

```
session		required		{PAM_MODULE_PATH}/pam_keyinit.so
```

Note: the `{PAM_MODULE_PATH}` should be modified to include the absolute path to the system library  on your system or to the location in the linux-pam project.

# Test code

A simple test would involve calling `run_pamtest()` with the following arguments: the PAM stack file name, the username, the conversation data, the PAM actions and a pointer to a pam_handle structure. My actual test is more complex because I want to check that the module is thread-safe.

Briefly explained, I create two threads and check their return code to see if they failed.

```
static void test_thread_pam_session(void **state)
{
    int i;
    pthread_t thread_id[MAX_THREADS];
    int ret;

    for (i = 0; i < MAX_THREADS; i++) {
        if (i == 0) {
            pthread_create(&thread_id[i], NULL, change_uids_and_sleep, &ret);
        } else {
            pthread_create(&thread_id[i], NULL, open_session, &ret);
        }
    }

    for (i = 0; i < MAX_THREADS; i++) {
        pthread_join(thread_id[i], NULL);
        if (i == 0) {
            assert_int_not_equal(ret, EINTR);
        } else {
            assert_int_equal(ret, PAMTEST_ERR_OK);
        }
    }
}
```

The first thread blocks in a sleep.

```
static void *change_uids_and_sleep(void *param)
{
    int ret;

    ret = sleep(3);

    *(int*)param = errno;
}
```

The second thread changes the user and group ID to the user `nobody` for itself. Finally, it opens a pam_keyinit session for the logged-in user by calling `run_pamtest()`, which loads the PAM stack defined in the previous section.

```
static void *open_session(void *param)
{
    *(int*)param = perr;
    
    int ret;
    char username[MAX_USERNAME_SIZE];
    struct passwd *pw;
    enum pamtest_err perr;
    struct pam_testcase tests[] = {
        pam_test(PAMTEST_OPEN_SESSION, PAM_SUCCESS),
    };

    pw = getpwnam("nobody");
    assert_non_null(pw);
    ret = getlogin_r(username, MAX_USERNAME_SIZE);
    assert_int_equal(ret, 0);
    ret = pam_setregid(pw->pw_uid, -1);
    assert_int_equal(ret, 0);
    ret = pam_setreuid(pw->pw_gid, -1);
    assert_int_equal(ret, 0);

    perr = run_pamtest("myapp", username, NULL, tests, NULL);

    *(int*)param = perr;
}
```

When pam_keyinit is executed the `sleep()` shouldn't be interrupted with `EINTR`. If it does, then the module isn't thread-safe.

# Test execution

In order to execute the test I preload pam_wrapper, enable it, define the location of the testing PAM stack file and call the testing binary:

```
LD_PRELOAD=libpam_wrapper.so \
    PAM_WRAPPER=1 \
    PAM_WRAPPER_SERVICE_DIR=./myapp \
    ./testprog
```

Note: as mentioned before the test changes the user ID, so it needs to be run as a privileged user.

# Additional options

Other options can be used to help debugging any problem encountered.

* PAM_WRAPPER_DEBUGLEVEL: enables additional logging for pam_wrapper. Four choices are available: ERROR, WARNING, DEBUG and TRACE.
* PAM_WRAPPER_KEEP_DIR: set to 1 to disable the deletion of the temporary directory. This is usually located in `/tmp/` and it's printed during the test execution.

# Conclusion

pam_wrapper eases the task of writing a test for a given PAM module as it enables you to focus on actually writing the test and forgetting about setting or tearing down the environment.

# Acknowledgements

To [Andreas Schneider](https://github.com/cryptomilk) for helping me setup my first pam_wrapper test. 

# Additional information

* [A lwn.net article written by the authors of pam_wrapper (Andreas Schneider and Jakub Hrozek) explaining how to use it.](https://lwn.net/Articles/671094/)
* [The repository containing the test.](https://github.com/ikerexxe/test_pam_keyinit)

