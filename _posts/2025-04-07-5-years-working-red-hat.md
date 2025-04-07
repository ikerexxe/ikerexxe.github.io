---
layout:     post
title:      "5 years working at Red Hat"
date:       2025-04-07 09:00:00 +0200
categories: work
background: '/assets/figures/2025-04-07-5-years-working-red-hat-1.png'
---

# My Routine at Red Hat

Continuing with the theme of anniversaries, I recently celebrated 5 years at Red Hat! So, I thought I'd share a glimpse into my daily routine at this amazing company. As you all know, I'm a software engineer, which is something I truly enjoy. More specifically, I work in the Identity Management department. This role is quite different from my previous ones, as I used to work in the embedded software world, and now I'm in IT. But I'm getting ahead of myself. Let's take it one step at a time, as Jack the Ripper used to say.

# Development

This is the part that's most similar across all companies. At Red Hat, I'm on the SSSD team, whose main goal is the development of this software package. [SSSD](https://github.com/SSSD/sssd) simplifies the administration of access to different identity providers, such as Microsoft's Active Directory (AD), LDAP, Kerberos, and others.

Apart from that, I also dedicate myself to the development of [shadow](https://github.com/shadow-maint/shadow), [Linux-PAM](https://github.com/linux-pam/linux-pam), [libeconf](https://github.com/openSUSE/libeconf), [pam_radius](https://github.com/FreeRADIUS/pam_radius), and [python-PAM](https://github.com/FirefighterBlu3/python-pam). In general, all these packages have a common goal: the management of authentication and security in Linux systems. In addition, I also frequently participate in the development of [pytest-mh](https://github.com/next-actions/pytest-mh), [sssd-test-framework](https://github.com/SSSD/sssd-test-framework/), [sssd-ci-containers](https://github.com/SSSD/sssd-ci-containers), etc., which are part of the test suite we use for the aforementioned projects. Finally, I also have some changes in projects like [freeipa](https://github.com/freeipa/freeipa), [gdm](https://gitlab.gnome.org/GNOME/gdm/), or [leapp-repository](https://github.com/oamg/leapp-repository/), among others.

When I say that I develop, I mean that I'm in charge of implementing new features, such as those related to passkey ([authentication](https://sssd.io/design-pages/passkey_authentication.html) and [kerberos integration](https://sssd.io/design-pages/passkey_kerberos.html)) or the [integration of passwordless authentication methods with the graphical interface](https://github.com/SSSD/sssd.io/pull/79). And I'm also in charge of fixing bugs reported to these projects.

# Downstream Maintainer

Here comes one of the big differences compared to other companies: maintenance. Red Hat follows a policy in which all its software is open source. This means that first, you have to release the code in the respective development repositories (upstream), and then you can port (downstream) these solutions to the distribution repositories for Fedora, CentOS Stream, and Red Hat Enterprise Linux (RHEL). At first glance, it's more complicated to explain than to understand, so I think it's better to understand it with an image:

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-04-07-5-years-working-red-hat-2.png){: .img-fluid}
| *A side-by-side look at the shadow project's git trees, revealing how they are structured for development and distributions.* |
</div>

Next, I'm going to explain the points marked with numbers in purple:

1.  Commits made in the development repository can be manually ported to any distribution in a process popularly known as backporting. In this specific case, `Commit1` is backported to Fedora 40.
2.  New versions of a project can also be ported to distributions (sometimes even automatically) in a process called rebasing. This means that **all changes** included up to the release of a version are ported to the distribution. In this case, version `4.15.1` is ported to Fedora 40.
3.  At some point in the development of the distribution, usually before the distribution version is released, the branch is forked to continue development. In this case, Fedora 40 and 41 are separated. This is done so that Fedora 40 stabilizes before its release, while more extreme development, which can lead to breakages, is done in Fedora 41.
4.  This forking process is also carried out between Fedora and CentOS Stream, which leads to the birth of CentOS Stream 10, which is a child of Fedora 40.
5.  Backports of a commit can be made to several distributions, such as Fedora 40, Fedora 41, and CentOS Stream 10.
6.  RHEL is also born from a fork; in this case, CentOS Stream 10 gives rise to RHEL 10 and all its minor versions (i.e., 10 Beta, 10.0, 10.1).
7.  It can also happen that a backport is made directly to the CentOS Stream branch without having gone through Fedora first.
8.  Normally, backports are not made to RHEL directly; instead, they are ported to CentOS Stream, and from there, RHEL inherits them. Although there are exceptions.

In my specific case, I'm in charge of doing downstream maintenance in Fedora, CentOS Stream, and RHEL for shadow, Linux-PAM, libeconf, python-PAM, and pam_radius. Yes, all of them!

# Upstream Maintainer

In shadow and SSSD, I also act as an upstream maintainer, that is, I maintain the development repositories. This means that apart from fixing bugs and making new implementations, I'm also in charge of doing code reviews, merging changes, maintaining the infrastructure, generating documentation, and in general, maintaining the good health of these projects.

# Supporting Support

At Red Hat, we have excellent support teams that are responsible for helping customers. Unfortunately, sometimes their extensive knowledge of the products is not enough, and that's where we software engineers come in. As the people who develop the projects, we are the people with the most technical knowledge, and that's why we sometimes help the support team to resolve their doubts and those of the customers.

# Mentor

Red Hat takes the continuous learning and growth of its employees very seriously. Because of this, there are several mentoring programs. I'm currently mentoring another team member in their goal of improving their technical and programming skills. We've been working on it for several months now, and little by little, the progress is noticeable.

# Team

As in any other company, teams have to organize themselves, and that's why we have two weekly meetings all together. We also have separate meetings for each new feature we are implementing. To all this, we must also add the mentoring meetings and those that may arise sporadically. At first glance, it may seem like a lot, but there are fewer meetings than in other companies, and the fact that there is an agenda in all of them helps us to focus the meetings a lot and dedicate ourselves to discussing what is truly important.

# Communities

Working at Red Hat or any other open source company requires you to work with many other people with very different profiles. Apart from participating in the communities of the different software projects, I also participate in the communities of the different distributions. In addition, I write on this blog, and I have also written several posts on Red Hat's blog, such as the one on [passkey](https://www.redhat.com/en/blog/passkey-with-rhel) and the one on [passwordless authentication](https://github.com/SSSD/sssd.io/pull/79) (WIP). Finally, I have participated in several conferences as an attendee, speaker, and organizer: [FOSDEM](https://archive.fosdem.org/2024/schedule/event/fosdem-2024-2169-passwordless-authentication-in-the-gui/), [devconf.cz](https://devconfcz2023.sched.com/event/1MYgz/fido2-authentication-for-centrally-managed-users), and [DOKSummit](https://doksummit.com/programa/). All this makes for a very public profile.

# Conclusion

Working with a talented team and having the opportunity to share knowledge at conferences and blogs is something I value tremendously. However, I aspire to grow as a **software engineer** by exploring areas such as low-level and kernel development, where I can apply and deepen my knowledge of system architecture and performance optimization. Ideally, I would like to collaborate with other engineers on projects that drive innovation within the company, rather than focusing on the direct needs of the customer. I believe that this approach, with a greater emphasis on core software engineering, would allow me to develop my full potential and contribute to the creation of more robust and efficient technologies.
