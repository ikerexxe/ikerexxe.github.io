---
layout:     post
title:      "SpacemiT K3 performance revisited"
date:       2026-10-13 09:00:00 +0200
categories: test
---

In my [K1 vs. K3 comparison](/test/2026/09/14/spacemit-k1-k3-performance-comparison), the K3 surprisingly needed much longer than the K1 to run the `shadow` test suite (29 minutes 7 seconds vs. 9 minutes 24 seconds), despite its superior hardware. The most likely explanation was the network latency of the remote facility hosting that K3.

I have now repeated the tests on a K3 that sits on the **same local network**, removing that limitation. Everything else remained unchanged: **Fedora 44 Omni**, **`shadow-utils-4.19.0-7`**, and the same `pytest-mh` configuration.

# Results

| Metric             | K1             | K3 (remote)     | K3 (local) |
|--------------------|----------------|-----------------|------------|
| **Network**        | Local          | Remote facility | Local      |
| **Tests**          | 129            | 147             | 154        |
| **Execution time** | 9m 24s         | 29m 7s          | **3m 44s** |

# Analysis

With the network variable removed, the picture changes completely:

- The local K3 executed 154 tests in **3 minutes 44 seconds**, roughly **8x faster per test** than the remote run.
- It is also about **3x faster per test than the K1**, which is what the hardware difference suggested all along.

# Conclusion

These results confirm that network latency, not the SoC, was the bottleneck in the previous comparison. Under comparable network conditions, the K3 clearly outperforms the K1, and its test execution time is finally consistent with the architectural advantages described in the [original post](/test/2026/09/14/spacemit-k1-k3-performance-comparison).
