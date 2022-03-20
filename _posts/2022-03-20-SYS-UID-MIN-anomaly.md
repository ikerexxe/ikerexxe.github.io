---
layout:     post
title:      "The SYS_UID_MIN anomaly"
date:       2022-03-20 12:00:00 +0100
categories: idm
---

# Origin
The origin of this post lies in two Red Hat bugzillas that I have recently worked on: [rhbz#2004911](https://bugzilla.redhat.com/show_bug.cgi?id=2004911) for shadow-utils and [rhbz#1949137](https://bugzilla.redhat.com/show_bug.cgi?id=1949137) for PAM. Both of them have a similar root cause: low IDs, from 0 to 200, not being recognized as system accounts.

# Context
## Linux user account types
There are two different user account types in Linux systems:
* system: used for operating system defined purposes like administrative tasks or running processes.
* regular: used by a person to access the system.

System accounts, in turn, can be classified in two groups regarding their ID allocation type:
* reserved (also know as static): their ID is statically allocated in a given distribution, which means that in all systems their ID will be the same. Examples in Fedora can be found in the `/usr/share/doc/setup/uidgid` file, which contains users such as root (0), apache (48) or dbus (81).
* dynamic: their ID is dinamycally allocated when the user is created.

Regular users, on the other hand, are usually assigned dynamically.

## ID thresholds
The thresholds that can be configured in `/etc/login.defs` to assign the IDs are:
* SYS_UID_MIN: minimum ID value for system user accounts.
* SYS_UID_MAX: maximum ID value for system user accounts.
* UID_MIN: minimum ID value for regular user accounts.
* UID_MAX: maximum ID value for regular user accounts.

## Actual values for the distributions

|             | Fedora19+ | Debian9+  |
| ----------- | --------- | --------- |
| SYS_UID_MIN | 201       | 100       |
| SYS_UID_MAX | 999       | 999       |
| UID_MIN     | 1000      | 1000      |
| UID_MAX     | 60000     | 60000     |

# So, what is the anomaly referred to in the title?
As can be seen in the previous table there is a range of ID's from 0 to SYS_UID_MIN that neither belongs to the regular type nor to the system. This was causing the bugs mentioned in the introduction of this post. That's why the definition of SYS_UID_MIN should be changed a little bit: minimum ID value for dynamically allocated system user accounts. ID's below the SYS_UID_MIN threshold are system accounts but they need to be statically allocated.

# Additional information
This topic was recently discussed in a [shadow-utils pull request](https://github.com/shadow-maint/shadow/pull/492#issuecomment-1005506083), but the topic has been referenced several times over the years in pages like [the security content policy group GitHub](https://github.com/ComplianceAsCode/content/pull/1285), [a stackexchange question](https://unix.stackexchange.com/questions/80277/whats-the-difference-between-a-normal-user-and-a-system-user/80279#80279) and [the Fedora development mailing list](https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/thread/L7FHINNIJH4GK3DHHJOL23TQ2W32RLFQ/).

