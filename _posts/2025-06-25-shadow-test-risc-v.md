---
layout:     post
title:      "Running shadow system tests in RISC-V"
date:       2025-06-25 09:00:00 +0200
categories: test
background: '/assets/figures/2025-06-25-shadow-test-risc-v-1.png'
---

Lately, my brain has been a playground for two seemingly disparate obsessions: the rigorous discipline of shadow system testing and the bleeding-edge architecture of RISC-V. You might reasonably ask, "What's the connection?" Truthfully, on the surface, there isn't one. But since I wanted to play with the RISC-V architecture I decided to see if I could run the shadow system tests on a RISC-V platform.

# Prepare the RISC-V system

## Setup RISC-V with QEMU

Before we dive into the shadow testing on RISC-V, we need a working environment. For this, we'll be using QEMU to emulate a RISC-V system. The Fedora project provides excellent resources for this, and I've found their guides to be invaluable.

First, we need to prepare our host environment by installing the necessary dependencies. The official Fedora documentation provides a [comprehensive guide](https://fedoraproject.org/wiki/Architectures/RISC-V/QEMU#Host_setup), which I recommend following closely. This step ensures that QEMU has all the tools it needs to run smoothly.

Once the host environment is ready, the next step is to download the QEMU image. Again, the Fedora project has you covered with a pre-built image and another [guide](https://fedoraproject.org/wiki/Architectures/RISC-V/QEMU).

**Note**: The initial startup of the QEMU image can take a significant amount of time, especially depending on your host system's resources. Don't panic if it seems to be hanging; just be patient.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2025-06-25-shadow-test-risc-v-2.jpg){: .img-fluid}
</div>

After the image has booted, you can log in with the following credentials:

* login: fedora
* password: linux

## Setup ssh

Setting up SSH access to our RISC-V QEMU guest presented an unexpected challenge. Initially, password-based authentication failed to function as expected, despite correct configuration. To circumvent this issue and establish a robust connection, we'll configure key-based authentication with root login enabled. This approach offers a reliable and secure method for remote access, essential for executing our shadow system tests.

First, we need to make some adjustments to the SSH server configuration within the guest system. Start by installing vim (if it's not already present) and opening the `sshd_config` file:

```console
dnf install -y vim
vim /etc/ssh/sshd_config
```

Within the `sshd_config` file, locate the `PermitRootLogin` directive and ensure it's set to `yes`. This allows root login, which is necessary for our workaround. Save your changes and restart the SSH daemon:

```console
systemctl restart sshd
```

Now, let's generate and copy your SSH public key from your host system. In your host terminal, run:

```console
cat ~/.ssh/id_rsa.pub
```

Copy the output of this command.

Next, switch back to your RISC-V guest system. Before modifying the `authorized_keys` file, it's always a good practice to create a backup:

```console
cp /root/.ssh/authorized_keys /root/.ssh/authorized_keys.backup
```

Now, open the `authorized_keys` file for editing:

```console
vim /root/.ssh/authorized_keys
```

Paste the SSH public key you copied earlier into this file. Save and close the file.

Finally, you should be able to SSH into your RISC-V guest from your host system. Use the following command, replacing `192.168.124.137` with the actual IP address of your guest:

```console
ssh root@192.168.124.137
```

If everything is configured correctly, you should now have secure SSH access to your RISC-V guest. This will enable running shadow's system test framework.

# Run tests

With our RISC-V QEMU environment set up and SSH access configured, we're finally ready to run our shadow system tests. This section will guide you through the process of configuring and executing the tests.

First, we need to configure the `mhc.yaml` file on your host system. This file defines the target environment for our tests. Open `mhc.yaml` in your preferred text editor and ensure it contains the following configuration:

```yaml
provisioned_topologies:
- shadow
domains:
- id: shadow
    hosts:
    - hostname: shadow.test
    role: shadow
    conn:
        type: ssh
        host: 192.168.124.137
        user: root
    artifacts:
    - /var/log/*
```

**Important**: Remember to replace `192.168.124.137` with the actual IP address of your RISC-V guest system.

During my testing, I encountered an issue with the `tests/system/tests/test_useradd.py` script. The user ID 1000 was already assigned to the `fedora` user in the QEMU image. Therefore, I modified the assertions in the test to use user ID 1001 instead. If you encounter a similar issue, you'll need to make the same adjustments.

Assuming you have the [shadow](https://github.com/shadow-maint/shadow) repository cloned on your host system, navigate to the tests/system/ directory:

```console
cd $(shadow)/tests/system/
```

Next, create a virtual environment to isolate the test dependencies:

```console
python -m venv .venv
source .venv/bin/activate
```

Install the necessary dependencies:

```console
pip install -r ./requirements.txt
```

Finally, execute the `test_useradd__add_user test` using pytest. We'll use the `--mh-config` flag to specify our configuration file, `--mh-lazy-ssh` to optimize SSH connections, and `-v` for verbose output:

```console
pytest --mh-config=mhc.yaml --mh-lazy-ssh -v -k test_useradd__add_user
```

**Note**: The test execution can take a considerable amount of time. Be patient.

If the test runs successfully, you should see output similar to this:

```console
collected 7 items / 6 deselected / 1 selected

tests/test_useradd.py::test_useradd__add_user (shadow) PASSED                                              [100%]
```

This confirms that our shadow test has successfully executed on the RISC-V QEMU environment.

# Conclusion

I'm thrilled to have successfully deployed and executed the shadow system test framework on RISC-V. It's a testament to the framework's adaptability and the growing maturity of the RISC-V ecosystem. This achievement fuels my desire to tackle even more challenging integration and testing scenarios. Let the next challenge begin!
