---
layout:     post
title:      "A Kernel story X: Submitting the patch series"
date:       2025-08-18 13:00:00 +0200
categories: kernel
---

In the last post, after a true debugging odyssey, we finally got our driver to come to life and display a shiny cube on the screen. It was a moment of victory, no doubt. But the work of a kernel developer doesn't end when the code works on their machine. The real goal has always been to share it with the community. And to do that, you have to send the patches.

Anyone who has approached Linux Kernel development knows that contributing isn't as simple as making a pull request on GitHub. The process is based on email—a system that has its own rules and complexities.

For this task, I've decided to try a tool I've heard great things about, one that promises to greatly simplify this process: B4<sup>[1](#references)</sup>. It's my first time using it, so this post will also serve as a journal of my experience and learnings with it as I prepare to take one of the most exciting steps: hitting "send".

# Preparing the patches with B4 prep

Preparation is, without a doubt, the most critical phase. This is where we ensure our work is well-packaged and ready to be reviewed by other developers. `B4` guides us step-by-step.

## Create the branch and cover letter

The first step is to create a clean working branch that contains our patches and a "cover letter." This letter is an email that heads the patch series and requires a text explaining the overall purpose of the changes.

```console
b4 prep -e drm-misc-next
```

This command creates a new branch from the `drm-misc-next` branch.

## Generate the recipient list

Who do we send the patches to? This is one of the toughest questions. `B4` has an almost magical command for this, which analyzes our changes and finds the relevant maintainers and mailing lists.

```console
b4 prep --auto-to-cc
```

After running it, `B4` will update our cover letter with the `To:` and `Cc:` fields automatically populated.

## Edit the content and recipients

Now for the most important part: describing what the patch series is about. We edit the letter again to add a summary of the changes, explain the problem they solve, and any other relevant details. Additionally, if the automated tool missed any recipient, this is the time to add them manually.

```console
b4 prep --edit-cover
```

## Check the patch quality

Before sending anything, it's essential to run our patches through the kernel's verification tools, like `checkpatch.pl`. `B4` integrates this perfectly.

```console
b4 prep --check
```

This command will review each patch for style errors, common mistakes, and other issues. Fixing the warnings it gives us is a sign of respect towards the maintainers who will review our work.

# Configuring email sending

`B4` sends emails using git's configuration. Therefore, it's essential to have our `~/.gitconfig` set up to send emails via an SMTP server. This only needs to be done once, and the configuration would look something like this:

```
[sendemail]
    from = Iker Pedrosa <ikerpedrosam@gmail.com>
    smtpserver = smtp.gmail.com
    smtpuser = ikerpedrosam@gmail.com
    smtpencryption = tls
    smtpserverport = 587
    chainreplyto = false
```

# The launch sequence with B4 send

With everything prepared, the moment of truth arrives. But there's no need to rush. `B4` allows us to be extremely cautious. I follow a three-step sequence:

## The simulation

This command generates the emails but, instead of sending them, saves them to a folder. It's the last chance to review them in their raw format and see if everything looks as expected.

```console
b4 send -o /tmp/presend
```

## The test send to yourself

The next step is to send the entire patch series only to myself. The `--reflect` option ignores the recipients in `To:` and `Cc:` and sends everything to the sender's address. It's perfect for seeing how the patches will look in a real email client.

```console
b4 send --reflect --no-sign
```

Note: I use `--no-sign` because I have chosen not to sign this series of patches with GPG.

## The real deal!

If everything looks good in the test email, there are no more excuses. The time has come. With a mix of nerves and excitement, we execute the final command.

```console
b4 send --no-sign
```

Hitting "Enter" here is an incredible feeling! Our patches are now flying across the network, on their way to the inboxes of the Kernel maintainers.

For those who are curious, [here](https://lore.kernel.org/all/20250806-st7920-v1-0-64ab5a34f9a0@gmail.com/) is the link to the patches I sent.

The journey isn't over. In fact, a new stage has just begun: the review process. Now it's time to wait for feedback, respond to comments, and hopefully, see our work integrated. But that... is a story for another day.

# Next

[A Kernel story XI: The review loop](/kernel/2025/12/17/a-kernel-story11)

# References

1. [<u>B4 end-user documentation</u>](https://b4.docs.kernel.org/en/latest/)
