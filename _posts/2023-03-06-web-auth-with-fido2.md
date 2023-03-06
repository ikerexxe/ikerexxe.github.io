---
layout:     post
title:      "Web authentication with FIDO2 keys"
date:       2023-03-06 12:00:00 +0100
categories: idm
---

# Introduction

I started using [bitwarden](https://bitwarden.com/) several months ago and one of the weaknesses I [identified](https://ikerexxe.github.io/idm/2022/11/06/bitwarden.html) was that it created a single point of attack to retrieve all my passwords. That's why I have decided to start using FIDO2 keys to increase the security level.

# Analysis

So far I have registered keys for three services and the process has been very simple. It's a matter of opening the security options and enabling FIDO2 WebAuthn. Then you register the key by giving it a name, and when prompted, tapping the device. That's it! From now on you can use the token to authenticate.

By the way, the FIDO2 key that I'm using also provides other authentication protocols, and I'm using OATH for a service that specifically requests it. So I'm killing two birds with one stone by using this type of keys.

One thing I don't like is how these services provide MFA. They all ask for the password, and then FIDO2 authentication. In my opinion it would be better to force the user-verification embedded in the FIDO2 key, and forget about the password. This would allow a move to a better integrated authentication workflow, where the user is only asked to enter the FIDO2 PIN, and then touch the device (which may have another authentication mechanism in the form of a biometric sensor). The difference seems subtle, but in my opinion it would make a big difference.

Finally, I'd like to mention that I have two FIDO2 keys. I use one for day-to-day use and keep the other as a backup. Note that for some services, if you lose or damage the single FIDO2 key, you will no longer be able to log in. Better to be safe than sorry by buying a second token.

# Conclusion

I recommend using FIDO2 keys to increase the security of your accounts. Once the improvement I have identified is implemented, FIDO2 authentication will be unstoppable.
