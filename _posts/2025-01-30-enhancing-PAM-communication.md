---
layout:     post
title:      "Enhancing PAM communication: A JSON-Based approach for modern authentication"
date:       2025-01-30 10:00:00 +0100
categories: idm
background: '/assets/figures/2025-01-30-enhancing-PAM-communication-1.jpg'
---

PAM, a cornerstone of Linux security, is undergoing a transformation with the introduction of JSON-based communication between PAM modules and client applications. This innovation marks a departure from traditional, somewhat limited communication methods, opening a new era of possibilities.

Historically, PAM modules have had limited ways to convey information about their capabilities to the client. This has often led to a rigid authentication process with minimal user choice and customization. With the adoption of JSON, PAM modules can now advertise a richer set of attributes, including:

* Supported authentication mechanisms: this includes traditional methods like passwords and newer options such as passkey, EIdP and smartcard.

* Mechanism priority: modules can specify the preferred order of authentication mechanisms, allowing for dynamic adaptation based on system policies or user preferences.

* Context-aware prompts: This enables modules to provide users with specific instructions or information based on the chosen authentication method.

This new feature is still under development, but don't worry because in this post, I will explain how to test it.

I presented on this topic at the FOSDEM 2025 IAM devroom.  For a full understanding of the context and proposed solution, I recommend watching the [talk](https://fosdem.org/2025/schedule/event/fosdem-2025-4678-enhancing-pam-communication-a-json-based-approach-for-modern-authentication/).

# Design

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-01-30-enhancing-PAM-communication-2.png){: .img-fluid}
| *Sequence diagram* |
</div>

The communication is embedded in the PAM conversation, more specifically it uses the PAM binary prompt. This allows the PAM module to advertise the authentication mechanisms, the user interaction and the priority, while the PAM application can reply with the user choice and any data needed for the authentication process. For detailed information dive into the design, I recommend checking out the relevant documentation<sup>[1](#references)</sup>.

# Set Up

In order to test this feature, you'll need an SSSD client enrolled into a FreeIPA server. Moreover, you'll need to install the SSSD version that contains the feature. This can be done either through this [COPR repository](https://copr.fedorainfracloud.org/coprs/ipedrosa/passwordles-gdm/) or from the [source code](https://github.com/ikerexxe/sssd/tree/gdm_passkey_sc). Assuming you want to do it with the COPR repository, you need to run:

```console
dnf copr enable ipedrosa/passwordles-gdm 
dnf update -y sssd
```

In addition, I have created a basic [PAM rust client application](https://github.com/ikerexxe/json-pam), which uses the following [PAM rust bindings](https://github.com/1wilkens/pam/pull/46). You'll need to clone both:

```console
git clone https://github.com/ikerexxe/json-pam
cd ..
git clone https://github.com/ikerexxe/pam-rust.git
git switch binary-prompt
```

Once everything is installed, you can proceed to create a user and assign your preferred authentication mechanisms: password, passkey, EIdP or smartcard.

Now it's time to run the PAM client application, so let's move to its folder and run:

```console
cd json-pam
cargo run
```

This will download and install all the crate dependencies. From now on, you can select the user with which you want to log in:

```console
login:
iker@gdm.test
```

This will prompt the JSON message received from pam_sss, and you'll be able to select the authentication mechanism that you want to use for authentication:

```console
json-pam receives: {"auth-selection": {"mechanisms": {"password": {"name": "Password", "role": "password", "selectable": true, "prompt": "Password"}, "eidp": {"name": "Web Login", "role": "eidp", "selectable": true, "init_prompt": "Log In", "link_prompt": "Log in online with another device", "uri": "https://github.com/login/device", "code": "7481-5DFE", "timeout": 300}, "smartcard:1": {"name": "Certificate for PIV Authentication\nCN=iker,O=GDM.TEST", "role": "smartcard", "selectable": true, "init_instruction": "Insert smartcard", "pin_prompt": "Smartcard PIN"}}, "priority": ["eidp", "smartcard:1", "password"]}}
Success parsing JSON
Select authentication mechanism:
1: password
2: eidp
3: smartcard
```

Depending on your choice, you'll get a series of prompts requesting additional information, like the password or the PIN. When you've finished, the application will reply with this information to the PAM module, and you'll be authenticated.

```console
json-pam sends: {"auth-selection":{"status":"Ok","password":{"password":"Secret123"}}}
Authentication successful!
Session opened successfully!
username "iker", uid 1295600006, gid 1295600006
```

# Conclusion

As you can see, the JSON-based communication in PAM offers astounding extensibility. By providing more information to the client, PAM modules can enable more flexible and user-friendly authentication experiences. This is a significant step forward in the evolution of PAM and opens up new possibilities for innovation in the realm of authentication.

# References:

1. [<u>SSSD-GDM interface design</u>](https://github.com/SSSD/sssd.io/pull/79)

**Edit**: Add talk at FOSDEM 2025.
