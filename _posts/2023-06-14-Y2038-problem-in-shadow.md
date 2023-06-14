---
layout:     post
title:      "Y2038 problem in shadow"
date:       2023-06-14 12:00:00 +0100
categories: idm
---

# Introduction

The following investigation is similar to the one provided by Thorsten Kukuk's on his blog: [Y2038, glibc and utmp/utmpx on 64bit architectures](https://www.thkukuk.de/blog/Y2038_glibc_utmp_64bit/) and [Y2038, glibc and /var/log/lastlog on 64bit architectures](https://www.thkukuk.de/blog/Y2038_glibc_lastlog_64bit/). In this investigation I'll try to go deeper into the dependencies that shadow-utils has with respect to the interfaces affected by the Y2038 glibc problem.

For additional information regarding the Year 2038 problem I'd suggest you to read the [wikipedia article](https://en.wikipedia.org/wiki/Year_2038_problem) with that title.

# Investigation

## utmp/utmpx/wtmp/btmp

* login: reads and writes utmp/wtmp data. The reading is done to ensure that the user has an open session by calling `get_current_utmp()`. It also makes sure that the user has not exceeded the maximum number of logins by calling `check_logins()`. The writing is done to create or update the utmp entry by calling `setutmp()`. It also logs the login failures by calling `failtmp()`. Finally, it uses the utmp structure to define the maximum length for the username when using the `USER_NAME_MAX_LENGTH` macro.
* logoutd: reads the utmp file to check if a user is allowed to login, and, once logged in, whether the user can remain logged in.
* su: reads utmp data to ensure that the user has not exceeded the maximum number of logins by calling `check_logins()`.

* newusers: It uses the utmp structure to define the maximum length for the username when calling `is_valid_user_name()`.
* pwck: It uses the utmp structure to define the maximum length for the username when calling `is_valid_user_name()`.
* useradd: It uses the utmp structure to define the maximum length for the username when calling `is_valid_user_name()`.
* usermod: It uses the utmp structure to define the maximum length for the username when calling `is_valid_user_name()`.

* usermod.8.xml: lost reference to utmp.

## lastlog

* lastlog: prints the content of the lastlog database.

* login: when PAM isn't used it logs the lastlog data in the database by calling `dolastlog()`.
* useradd: add the user to the lastlog database by calling `lastlog_reset()`.
* usermod: update the lastlog database when changing the UID by calling `update_lastlog()`.

# Solutions

There are too many tools and packages using the {u,w,b}tmp and lastlog interfaces, and all of them need to change from the old interfaces to the new ones at the same time. Other options would break some functionality related to these interfaces at some point or the other. That's why we need to provide distribution maintainers with a way to make changes smoothly.

## utmp/utmpx/wtmp/btmp

All the utmp function dependencies should be replaced by systemd-logind, but this needs to be done with care. That is why I propose to create a configuration option to control these changes (i.e. `SESSION_MANAGEMENT`=`UTMP` or `LOGIND`). The default value should be `LOGIND`, but shadow-utils should provide `UTMP` for some time to maintain backward compability. An abstraction layer should be implemented to keep both implementations at the same time.

The logoutd binary will be deprecated in the next release and, if there are no objections, will be removed in the following one.

Regarding the username length, [POSIX doesn't define a maximum value](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html#tag_03_437). The kernel does provide this value and it can be obtained using the `sysconf(_SC_LOGIN_NAME_MAX)` interface. So we'll use it.

Finally, there's a lost reference to utmp in usermod's man page that should be removed.

## lastlog

All the code interacting with the lastlog database should be conditionally compiled, including the lastlog binary itself. A new configuration option should be provided to enable the lastlog implementation provided by shadow-utils. If disabled, default case, this implementation shoulnd't be compiled.

# Acknowledgments

To [Thorsten Kukuk](https://github.com/thkukuk) and [Serge Hallyn](https://github.com/hallyn) for the feedback provided during the [investigation](https://github.com/shadow-maint/shadow/issues/674).
