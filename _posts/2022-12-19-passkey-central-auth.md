---
layout:     post
title:      "Local authentication in a centralized management system with passkey"
date:       2022-12-19 12:00:00 +0100
categories: idm
---

# Introduction

I have been working on a feature that will allow centrally managed users to log in locally to a computer with passkeys. For the purpose of this work, passkey is a FIDO2 compatible device supported by the libfido2 library. As the feature is still under development I've prepared an environment to test it.

## Requirements

First of all a FIDO2 key is needed. I'm using a Yubikey 5 series, but you can use any FIDO2 compliant key. Besides, the test environment uses docker/podman to set up the containers and ansible to provision them. On top of that, yubikey-manager is needed to manage the FIDO2 device options. Moreover, I've created a [COPR repository](https://copr.fedorainfracloud.org/coprs/ipedrosa/passkey-auth/) to deliver the patched versions of SSSD and IPA that will enable this feature. Finally, I've prepared a new [sssd-ci-container branch](https://github.com/ikerexxe/sssd-ci-containers/tree/passkey) to automate the provision of everything. The following sections will explain how to set up the environment and how to test it.

## Passwords

All passwords are set to `Secret123`.

# How to set up the environment

* Clone sssd-ci-containers locally and checkout the passkey branch:

```console
$ git clone https://github.com/ikerexxe/sssd-ci-containers/
$ cd sssd-ci-containers
$ git checkout --track origin/passkey
```

* Host package installation and configuration. The following commands should be executed only once:

```console
$ sudo dnf install -y podman podman-docker docker-compose yubikey-manager fido2-tools
$ sudo systemctl enable --now podman.socket
$ sudo setsebool -P container_manage_cgroup true
$ cp env.example .env
```

* Check if the ansible community.general module is installed:

```console
$ ansible-galaxy collection list | grep community.general
community.general             4.8.2
```

* If it isn't installed then do it:

```console
$ sudo ansible-galaxy collection install community.general
```

* If you haven't done so, connect the key to the computer.

* Set up the containers (make up) and update the packages with the passkey patches (make passkey). For the latter command to run correctly you will need to input the password. It will take some time to execute as it needs to update the dnf cache and then update the sssd packages.

```console
$ sudo make up
$ sudo make passkey
```

This way the environment is ready for testing. From now on you can use the following command to connect to the client container:

```console
$ sudo podman exec -it client /bin/bash
```

# Configure sssd

* Edit `/etc/sssd/sssd.conf` to enable passkey authentication:

```
...
[pam]
pam_passkey_auth = true
...
```

* Restart sssd service:

```console
$ systemctl restart sssd
```

# Additional configuration

It is advisable to add a PIN to the key to reinforce the security. You only need to do this once.

```console
$ ykman fido access change-pin
```

# IPA

## Add a user

* First of all obtain a kerberos ticket:

```console
$ kinit admin@IPA.TEST
```

* Add the user:

```console
$ ipa user-add joe --first=joe --last=doe
```

## Register a key

### Client

* Register the key with sssctl:

```console
$ sssctl passkey-exec --register --username=joe --domain=ipa.test
```

**Note**: If you'd like the full logs to be printed in the command line then append `--debug-level=9 --logger=stderr`

* Next, the PIN will be requested and once it is introduced the key will blink indicating that you need to touch it.

* Finally, this will output a string with the following format:

```
PASSKEY:credentialId,publicKey
```

Example:
```
passkey:aEgemlnC6a/WOoEZ8qU1YMwsTW9+uwmMsJnrgOXwTID0qIBHirzHp6d+e1d3WBhcSf7t9Ji8fl3AdSPtlbdN5Q==,MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAENwDQHwyZmnYaUEp0UNqqnw0tGOGnqOMBGdds6O3+JKbmmJGTn0vo7sKNNcDWDsFhJFU/RLWXmHXglxSo+yw9iQ==
```

### Server

* Add the attribute to the user:

```console
$ ipa user-add-passkey joe passkey:aEgemlnC6a/WOoEZ8qU1YMwsTW9+uwmMsJnrgOXwTID0qIBHirzHp6d+e1d3WBhcSf7t9Ji8fl3AdSPtlbdN5Q==,MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAENwDQHwyZmnYaUEp0UNqqnw0tGOGnqOMBGdds6O3+JKbmmJGTn0vo7sKNNcDWDsFhJFU/RLWXmHXglxSo+yw9iQ==
------------------------------------
Added passkey mappings to user "joe"
------------------------------------
  User login: joe
  Passkey mapping: passkey:aEgemlnC6a/WOoEZ8qU1YMwsTW9+uwmMsJnrgOXwTID0qIBHirzHp6d+e1d3WBhcSf7t9Ji8fl3AdSPtlbdN5Q==,MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAENwDQHwyZmnYaUEp0UNqqnw0tGOGnqOMBGdds6O3+JKbmmJGTn0vo7sKNNcDWDsFhJFU/RLWXmHXglxSo+yw9iQ==
```

* Checking that it has been added correctly:

```console
$ ipa user-show joe
  User login: joe
  First name: joe
  Last name: doe
  Home directory: /home/joe
  Login shell: /bin/sh
  Principal name: joe@IPA.TEST
  Principal alias: joe@IPA.TEST
  Email address: joe@ipa.test
  UID: 805400005
  GID: 805400005
  Passkey mapping: passkey:aEgemlnC6a/WOoEZ8qU1YMwsTW9+uwmMsJnrgOXwTID0qIBHirzHp6d+e1d3WBhcSf7t9Ji8fl3AdSPtlbdN5Q==,MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAENwDQHwyZmnYaUEp0UNqqnw0tGOGnqOMBGdds6O3+JKbmmJGTn0vo7sKNNcDWDsFhJFU/RLWXmHXglxSo+yw9iQ==
  Account disabled: False
  Password: False
  Member of groups: ipausers
  Kerberos keys available: False
```

## User authentication

* Change to `ci` user, as `root` will always authenticate as another user:

```console
$ su - ci
```

* Now you can proceed to authenticate as the newly created user using the passkey. Remember to enter the PIN when requested and to touch the device when the LED starts blinking:

```console
$ su - joe@ipa.test
Insert your passkey device, then press ENTER.
Enter PIN:
Creating home directory for joe@ipa.test.
```

* Check the user id:

```console
$ id
uid=805400005(joe@ipa.test) gid=805400005(joe@ipa.test) groups=805400005(joe@ipa.test)
```

Congratulations! You've succesfully authenticate as `joe` using the passkey.

# LDAP

This part is more complicated than IPA as a custom schema needs to be created. This schema will be used to hold the passkey data.

## Register a key

### Client

* As with the IPA server we also need to register the key with sssctl:

```console
$ sssctl passkey-exec --register --username=alice --domain=ldap.test
```

**Note**: If you'd like the full logs to be printed in the command line then append `--debug-level=9 --logger=stderr`

* Next, the PIN will be requested and once it is introduced the key will blink indicating that you need to touch it.

* Finally, this will output the passkey string:

```
passkey:oducA9WSTrzBHX2gUKylRNl2PD2XCb4a7V0XJOtahqIX7wGcAugflvrVjbWG2JPTsLlVf+j/dmia7SNIVhK5AA==,MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEGEa7EktmUw4AOR6Y6r1W2zxXptQh3YaDNdvQEifZ3NpgRosVv+GS85uR3h6Ed1E7FtgfugwsZYeR8+9+GM6h8g==
```

### Server

* You will need to connect to the LDAP container for the next steps:

```console
$ sudo podman exec -it ldap /bin/bash
```

* Edit `/etc/dirsrv/slapd-localhost/schema/60base.ldif` to add the custom schema:

```console
dn: cn=schema
attributeTypes: ( 2.16.840.1.113730.3.8.24.27 NAME 'passkey' DESC 'Passkey mapping' EQUALITY caseExactMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 )
objectclasses: ( 2.16.840.1.113730.3.8.24.9 NAME 'passkeyUser' DESC 'IPA passkey user' AUXILIARY MAY passkey)
```

* Dynamically reload the schemas:

```console
$ dsconf -D "cn=Directory Manager" localhost schema reload
```

* Create the user and its attributes in `user.ldif`:

```console
dn: uid=alice,dc=ldap,dc=test
mail: alice@ldap.test
uid: alice
givenName: Alice
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: inetorgperson
objectClass: posixAccount
objectClass: inetuser
sn: Moreau
cn: Alice Moreau
uidNumber: 2000
gidNumber: 2000
homeDirectory: /home/alice
loginShell: /bin/bash
gecos: alice
passkey: passkey:oducA9WSTrzBHX2gUKylRNl2PD2XCb4a7V0XJOtahqIX7wGcAugflvrVjbWG2JPTsLlVf+j/dmia7SNIVhK5AA==,MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEGEa7EktmUw4AOR6Y6r1W2zxXptQh3YaDNdvQEifZ3NpgRosVv+GS85uR3h6Ed1E7FtgfugwsZYeR8+9+GM6h8g==
objectclass: passkeyUser
```

* Add the user to the LDAP database:

```console
$ ldapadd -D "cn=Directory Manager" -w Secret123 -H ldap://localhost -x -f user.ldif
```

## User authentication

* From the client machine change to `ci` user, as `root` will always authenticate as another user:

```console
$ su - ci
```

* Now you can proceed to authenticate as the newly created user using the passkey. Remember to enter the PIN when requested and to touch the device when the LED starts blinking:

```console
$ su - alice@ldap.test
Insert your passkey device, then press ENTER.
Enter PIN:
Creating home directory for alice@ldap.test.
```

* Check the user id:

```console
$ id
uid=2000(alice@ldap.test) gid=2000 groups=2000
```

Congratulations! You've succesfully authenticate as `alice` using the passkey.

# AD

You could also use Active Directory to manage the user. The specific instructions are out of the scope of this post, but you should follow the [instructions to set up the Windows VM](https://github.com/SSSD/sssd-ci-containers#using-real-active-directory-instance), create the user and add the passkey to the `altSecurityIdentities` attribute.


# Cleaning up the environment

First you need to exit the container and then execute the clean up command (make down).

```console
$ exit
$ sudo make down
```

# Additional information

More information regarding the sssd-ci-containers can be found [here](https://github.com/SSSD/sssd-ci-containers).

There are several options to tune the SSSD behaviour in the `sssd.conf` file. I'd recommend you to read the man page to learn how to do it.

**Edit**: Install ansible module for `root`. Thanks to [Thorsten Scherf](https://github.com/tscherf) for catching it.
