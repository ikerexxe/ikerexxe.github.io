---
layout:     post
title:      "SSSD: authentication method policy"
date:       2023-11-13 10:00:00 +0100
categories: idm
---

# Introduction

SSSD has recently released version [2.9.2](https://sssd.io/release-notes/sssd-2.9.2.html), which adds a new option to control the authentication method policy. The need for this new option comes from adding methods which may work with the configured online (or server-side) authentication and can securely be used for offline (or local) authentication without interacting with the server and adding new authentication methods such as passkey[1][2]. It is possible that more authentication methods will emerge in the future, so it seems necessary to add a way to control the policy governing them.

# Existing policy before v2.9.2

When several authentication mechanisms were available SSSD defaulted to try methods that would return a Kerberos ticket. If the server was unavailable, then SSSD would try an offline authentication. For this purpose, it would match the preferred online authentication method, i.e. if the server-side authentication would be performed via password, the local authentication would be the same.

Note: Take into account that the local password authentication workflow requires to set `cache_credentials` to `true` to enable the user cache credential caching. This is potentially security risky, as a malicious actor with access to the cache file (normally requires privileged access) can try to break the password using brute force attack. Thus, it would be recommended to use passkey or smartcard mechanisms for local authentication. You can check how to do that below.

# New policy after v2.9.2

The default new policy is the same as before, but the behaviour can be tuned using the `local_auth_policy` domain option. `local_auth_policy` is set to `match` by default, which means that the online and offline methods are matched. As an example, if the password, passkey and smartcard methods are available for server-side authentication, then the local available methods will be the same with some ifs and buts. As mentioned before, password mechanism needs to set `cache_credentials = true`. In the case of smartcard, you'll need a successful online authentication as well to store the proper mapping in the cache. As for passkey, you'll need the user mapping credentials cached, this can be obtained by trying to authentication online once.

`local_auth_policy` can also be set to `only`. In this case, even if the server is available, the authentication will be performed offline using passkey or smartcard mechanisms.

Finally, the last option for `local_auth_policy` is `enable`. This allows to define a comma-separated list of the enabled offline authentication mechanisms. For example, `enable:passkey, enable:smartcard` will enable local authentication with passkey and smartcard.

## Example

The following snippet allows for offline authentication with passkey.

```
[domain/ldap.test]
id_provider = ldap
ldap_uri = _srv_
ldap_tls_reqcert = demand
ldap_tls_cacert = /data/certs/ca.crt
dns_discovery_domain = ldap.test
use_fully_qualified_names = true
local_auth_policy = enable:passkey
```

# Conclusion

The inclusion of new authentication mechanisms makes it necessary to also include a policy to govern them. The new option explained in this post allows this to be done through a simple mechanism while allowing the policy to be tailored to the user's needs.

# References

1. [SSSD: Passkey authentication in a centralized environment](https://sssd.io/design-pages/passkey_authentication.html)
2. [SSSD: Passkey authentication Kerberos integration](https://sssd.io/design-pages/passkey_kerberos.html)
