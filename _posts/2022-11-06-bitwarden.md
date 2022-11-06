---
layout:     post
title:      "bitwarden"
date:       2022-11-06 12:00:00 +0100
categories: idm
---

# Introduction

I started using [bitwarden](https://bitwarden.com/) six months ago and these are my thoughts on it.

# Analysis

The service makes a good work to handle and secure access to web pages by storing the account information. This data is easily accessible from the web browser with an addon that recognizes the web pages that the user is visiting and provides the account information accordingly. Moreover, it contains a password generator that I used every time that I added a new web page to generate a new secure password.

In addition to the web browser addons, there are also applications for the desktop and mobile operating systems. The Linux desktop application is provided as a Flatpak, which takes a while to load, but when ready it runs smoothly. The Android app, on the other hand, is difficult to use because you have to copy the data manually, creating a cumbersome workflow. I guess there is no other option because the operating system doesn't provide it, so it's not really bitwarden's fault.

Bitwarden can also be used to store other sensitive data like credit cards, contacts and notes. I have not used them much, but they may be useful for some people.

An interesting feature is the one that allows to share an account information among different bitwarden users. I think this is only available for paying users, but still it's very interesting for people that need to share accounts.

As a final point I would like to add I have one major concern. I have moved most of the sensitive data in my head to an electronic location that can be accessed by anyone. Thus, creating a single point that anyone can attack. I know the data is protected by a passphrase, but I still feel somewhat exposed. So I think I'm going to start using an additional hardware token such as a FIDO2 key to increase the level of protection.

# Conclusion

I strongly recommend bitwarden to anyone who wants to get rid of having to remember passwords.
