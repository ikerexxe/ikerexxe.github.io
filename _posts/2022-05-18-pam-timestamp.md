---
layout:     post
title:      "pam_timestamp usage"
date:       2022-05-18 12:00:00 +0100
categories: idm
---

# What is pam_timestamp?

It is a pluggable authentication module (PAM) that allows you to use a recent successful authentication attempt as the basis for authentication. This mechanism is similar to the one used by **sudo**, that prompts for the password once, and then authorizes all the following authentication attempts in a given period of time without any password input.

# How to configure it?

## PAM stack

First of all you will need to update the pam stack files (*/etc/pam.d/system-auth* or */etc/pam.d/password-auth*) to include the pam_timestamp module.

As an example this is an excerpt from the first file:

```
...
auth        [default=1 ignore=ignore success=ok]         pam_localuser.so
auth        sufficient                                   pam_timestamp.so verbose
auth        sufficient                                   pam_unix.so nullok
...
session     required                                     pam_unix.so
session     optional                                     pam_timestamp.so
session     optional                                     pam_sss.so
```

## Authentication configuration file

Starting with [version 1.5.2 of PAM](https://github.com/linux-pam/linux-pam/releases/tag/v1.5.2) there is a new [configuration](https://github.com/linux-pam/linux-pam/blob/master/modules/pam_timestamp/pam_timestamp.8.xml#L55), that can be tuned to increase the security of this module by changing the cryptographic algorithm in charge of caching the successful authentication attempts. The internal pam\_timestamp SHA1 implementation has been replaced by the one provided by the OpenSSL library.

This configuration can be changed in the */etc/login.defs* file in the **HMAC_CRYPTO_ALGO** option. The default value is set to SHA512, but other values like SHA256 or SHA3-512 can be used.

**Note**: This change is available since Fedora 36 and the newly released RHEL9.0.

# How to test it?

A simple way to test it is to use **su** to authenticate as another user, then logout from that user and then authenticate again. The first attempt should prompt for the password while the second one shouldn't.

```
$ su - testuser
Password: 
$ exit
logout
$ su - testuser
Access has been granted (last access was 3 seconds ago).
$ id
uid=1001(testuser) gid=1001(testuser) groups=1001(testuser) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

