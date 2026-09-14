---
layout:     post
title:      "SpacemiT K1 vs. K3: performance comparison"
date:       2026-09-14 09:00:00 +0200
categories: test
background: '/assets/figures/2026-09-14-spacemit-k1-k3-performance-comparison.png'
---

Following my [recent work running shadow system tests on the K1](/kernel/2026/08/27/shadow-test-orangepi-rv2), I decided to extend the analysis to SpacemiT's **K3** SoC. The K1 and K3 belong to different generations of SpacemiT's SoCs, making them ideal candidates for evaluating how architectural advancements impact low-level system utilities like `shadow`.

For this comparison, I used **Fedora 44 Omni** with **`shadow-utils-4.19.0-7`** on both SoCs. The hardware configurations differ significantly, particularly in CPU architecture, memory, and storage:

# Hardware specifications

| Feature               | K1                          | K3                          |
|-----------------------|-----------------------------|-----------------------------|
| **CPU Cores**         | 8 x60 (in-order)            | 8 x100 (out-of-order)       |
| **RAM**               | 8GB                         | 16GB                        |
| **Storage**           | SD card                     | NVMe SSD                    |
| **Network**           | Local                       | Remote facility             |

# System preparation

## SSH configuration

As detailed in the [K1 post](/kernel/2026/08/27/shadow-test-orangepi-rv2), SSH access is required to run the tests. For both the K1 and K3, I reused the same **key-based authentication** approach with `root` login enabled. The steps are identical to those described previously:

1. Modify the SSH server configuration (`/etc/ssh/sshd_config`) to enable `PermitRootLogin`.
2. Restart the SSH daemon.

For reference, the commands are:

```console
dnf install -y vim
vim /etc/ssh/sshd_config
# Ensure PermitRootLogin is set to `yes`
systemctl restart sshd
```

# Compilation time

Before running the tests, I measured the compilation time for `shadow` on both platforms:

| Platform | Compilation Time |
|----------|------------------|
| K1       | **5 minutes, 54 seconds** |
| K3       | **1 minute, 27 seconds** |

The **K3 compiled the code ~76% faster** than the K1, which aligns with its superior CPU architecture (out-of-order execution) and NVMe storage. This performance gap highlights how both **CPU microarchitecture** and **storage technology** can impact build times.

# Test execution

## Test topology

For both SoCs, I used a similar `pytest-mh` configuration to define the test topology. The `mhc-k1.yaml` and `mhc-k3.yaml` files specify the host, user, and artifacts to collect. Below is the template used for both:

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
      host: <k1_or_k3_hostname>
      user: root
    artifacts:
    - /var/log/*
```

## Virtual environment setup

The virtual environment setup was consistent across both platforms:

```console
cd $(SHADOW_REPO)/tests/system/
python -m venv .venv
source .venv/bin/activate
pip install -r ./requirements.txt
```

## Running the tests

I executed the test suite using the following command:

```console
time pytest --mh-config=mhc-<k1_or_k3>.yaml --mh-lazy-ssh -v
```

# Results

## K1 results

- **Total tests**: 129
- **Executed**: 126 (3 skipped due to topology mismatch)
- **Passed**: 118
- **Failed**: 8 (`newgrp`, `vipw` and `interactive` related)
- **Execution time**: 9 minutes, 24 seconds

## K3 results

- **Total tests**: 147
- **Executed**: 144 (3 skipped due to topology mismatch)
- **Passed**: 136
- **Failed**: 8 (`newgrp`, `vipw` and `interactive` related)
- **Execution time**: 29 minutes, 7 seconds

# Analysis

| Metric               | K1 (In-Order, SD Card) | K3 (Out-of-Order, NVMe) |
|----------------------|------------------------|-------------------------|
| **Compilation time** | 5m 54s                 | 1m 27s                  |
| **Execution time**   | 9m 24s                 | 29m 7s                  |
| **Tests passed**     | 118/126                | 136/144                 |
| **Tests failed**     | 8                      | 8                       |

## Key takeaways

1. **Compilation performance**: 
   - The K3's out-of-order cores and NVMe storage delivered a **~76% reduction in compilation time** compared to the K1. This underscores the impact of both CPU architecture and storage technology on build performance.

2. **Test execution performance**:
   - The K3's test execution time was **longer** than the K1's, despite its superior hardware. This is likely due to the network latency introduced by the remote facility access, which overshadowed the performance gains from the NVMe SSD and out-of-order execution.

3. **Compatibility**: both K1 and K3 failed the same tests (`newgrp`, `vipw` and `interactive`), suggesting that these failures are likely due to the nature of the tests rather than hardware limitations.

# Conclusion

The comparison between SpacemiT's K1 and K3 SoCs underscores the importance of architectural choices in system-level testing. While the K3's out-of-order execution and NVMe storage significantly improved compilation performance, the remote facility access introduced network latency that impacted test execution times. For developers targeting RISC-V, these results emphasize the need to consider both CPU microarchitecture and storage technology, as well as network conditions, when designing or debugging low-level software.
