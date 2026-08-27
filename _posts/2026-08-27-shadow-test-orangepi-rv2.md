---
layout:     post
title:      "Running shadow system tests in the OrangePi RV2"
date:       2026-08-27 09:00:00 +0200
categories: test
background: '/assets/figures/2026-08-27-shadow-test-orangepi-rv2.png'
---

Since I had already merged the [SD card driver for the OrangePi RV2](/kernel/2026/03/30/OrangePi-RV2-SD-card), I decided to revisit the shadow system tests, which I had previously run in [a RISC-V emulated environment](/test/2025/06/25/shadow-test-risc-v), and execute them directly on real hardware using the OrangePi RV2. The OrangePi RV2, a RISC-V-based single-board computer, provides an ideal platform for testing low-level system utilities like `shadow`.

For this test, I set up the environment using the `Fedora 44 Omni` image, running `shadow-utils-4.19.0-7`.

# Prepare the system

## Set up SSH

SSH access to the OrangePi is required for the tests to run. Password-based authentication initially failed, despite a valid configuration. To work around this, I configured key-based authentication with root login enabled.

First, modify the SSH server configuration in the guest system. Install `vim` if necessary, then open `sshd_config`:

```console
dnf install -y vim
vim /etc/ssh/sshd_config
```

Ensure that the `PermitRootLogin` directive is set to `yes`, save the file and restart the SSH daemon:

```console
systemctl restart sshd
```

Allow the `fedora` user to SSH as `root` using their public key:

```console
ssh-keygen -t rsa -f ~/.ssh/id_rsa -N "" -q
cat ~/.ssh/id_rsa.pub | sudo tee -a /root/.ssh/authorized_keys > /dev/null
ssh-keyscan -H localhost >> ~/.ssh/known_hosts 2>/dev/null
```

# Run tests

Navigate to the test directory and generate `mhc-orangepi.yaml`. The `mhc-orangepi.yaml` file defines the test topology and connection details for `pytest-mh`, specifying the host, user, and artifacts to collect.

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
      host: localhost
      user: root
    artifacts:
    - /var/log/*
```

Prepare the virtual environment:

```console
cd $(SHADOW_REPO)/tests/system/
python -m venv .venv
source .venv/bin/activate
pip install -r ./requirements.txt
```

Run the tests:

```console
pytest --mh-config=mhc-orangepi.yaml --mh-lazy-ssh -v
```

Unfortunately, execution hit a fatal crash during the environment restoration phase:

```console
Fatal Python error: Segmentation fault

Current thread 0x0000003fab49d040 [pytest] (most recent call first):
  File "/home/fedora/shadow/tests/system/.venv/lib64/python3.14/site-packages/pytest_mh/conn/ssh.py", line 328 in _wait_for_rc
  ...
[ 2522.854714] pytest[5527]: unhandled signal 11 code 0x1 at 0x0000000000000260 in libgcc_s-16-20260515.so.1
[ 2522.868807] CPU: 7 UID: 1000 PID: 5527 Comm: pytest Not tainted 7.1.5-201.0.riscv64.omni.fc44.riscv64 #1 PREEMPT(lazy)
[ 2522.893568] epc : 0000003f9aee2508 ra : 0000003f9aee222a sp : 0000002ae585bda0
[ 2523.003440] status: 8000000200004620 badaddr: 0000000000000260 cause: 000000000000000d
...
Segmentation fault (core dumped) pytest --mh-config=mhc-orangepi.yaml --mh-lazy-ssh -v
```

The kernel log output, mixed into the traceback, showed that the crash did not actually occur in Python itself. It triggered a load page fault (`badaddr: 0x260`) in `libgcc_s`. This appears to be an issue during stack unwinding or thread cleanup while `pytest-mh` was waiting for SSH commands to complete.

I attempted to debug the issue using `gdb`, but encountered an obstacle: critical `debuginfo` packages are still missing in the RISC-V repositories for Fedora 44. I lacked the time to rebuild everything with debug symbols, so I had to set this aside for now, until proper symbol packages become available for this target.

# Run the tests remotely

To avoid the segmentation fault on the target hardware, I opted to run the test suite remotely from my laptop instead.

First, copy the SSH public key to the board:

```console
ssh-copy-id root@orange.pi
```

Navigate to the test directory in the local shadow repository and update `mhc-orangepi.yaml` to point to the host:

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
      host: orange.pi
      user: root
    artifacts:
    - /var/log/*
```

Set up the virtual environment on the host machine:

```console
cd $(SHADOW_REPO)/tests/system/
python -m venv .venv
source .venv/bin/activate
pip install -r ./requirements.txt
```

Run the tests and time the execution:

```console
time pytest --mh-config=mhc-orangepi.yaml --mh-lazy-ssh -v
```

The results were as follows:
- Of the 129 total tests, 126 executed (the 3 skipped tests belonged to a different topology).
- All executed tests passed, except for `newgrp`, `vipw`, and `interactive`. The failures in `newgrp`, `vipw`, and `interactive` are likely due to these tests requiring interactive user input.
- The entire test run completed in 9 minutes, 24 seconds.
