---
title: "Dirty Frag: A Nine-Year-Old Kernel Bug Just Became Every Linux Distro's Problem"
categories: [Blog]
tags: [cve, LPE]
render_with_liquid: false
media_subpath: /images/blog_dirtyfrag/
image:
  path: banner.png
---

[![Link](header.png){: width="1200" height="600" }](https://dirtyfrag.io/)

A universal Linux LPE chaining two vulns in `xfrm-ESP` and `RxRPC`. A successor class to Dirty Pipe & Copy Fail.
Ubuntu / RHEL / Fedora / openSUSE / CentOS / AlmaLinux, and more.

# Dirty Frag: Two Kernel Bugs, One Root Shell, Zero Patches

Just when Linux admins were finishing up with [Copy Fail](https://nginx0.github.io/posts/blog-copy_fail/), here comes another one. Dirty Frag is a newly disclosed privilege escalation exploit that works on Ubuntu, RHEL, Fedora, CentOS, AlmaLinux, openSUSE and more. It requires no special permissions, no lucky timing, and it succeeds on the very first try. There is no patch. There is no CVE. And the researcher who found it never got the chance to release it on his own terms.

Think of it as the next evolution of [Dirty Pipe](https://dirtypipe.cm4all.com/). The target is the same, the Linux kernels page cache, which is the copy of your files that the kernel holds in memory to speed things up. Getting the kernel to corrupt that in-memory copy, without ever touching the actual file on disk and without any write permissions, is exactly what Dirty Frag pulls off. Twice, actually, using two completely separate vulnerabilities working in tandem.

![](demo.gif){: width="1280" height="720"}

## The Two Bugs Under the Hood

The older of the two has been hiding in the IPsec/ESP networking stack since January 2017. Kernel commit `cac2661c53f3` is where it started. Deep inside `esp_input()`, when a non-linear socket buffer carrying a splice-pinned page cache reference bypasses the `skb_cow_data()` copy-on-write check, the ESP decryption path ends up writing 4 bytes straight into the page cache. The attacker chooses both the offset and the value. That is a clean arbitrary-write primitive sitting in the kernel for nearly a decade.

The newer bug arrived in June 2023 inside the RxRPC/rxkad subsystem, in a function called `rxkad_verify_packet_1()`. It performs an 8-byte in-place `pcbc(fcrypt)` decryption directly onto a splice-pinned page, and it does not ask for namespace privileges to do it. What makes this one especially clean is that the attacker works out the right decryption key entirely in user space before the exploit even touches the kernel. By the time anything happens at the kernel level, the result is already locked in.

Neither bug is a race condition. No millisecond timing windows to hit, no kernel crashes on a bad attempt. Logic bugs, both of them. They either work or they do not, and they work.

## Why One Bug Was Never Going to Be Enough

Each vulnerability has a gap the other fills. The ESP bug needs unprivileged user namespace creation, something a hardened Ubuntu system can shut down through AppArmor. That would stop the ESP path completely. The RxRPC bug sidesteps namespaces entirely, which sounds ideal, except the `rxrpc.ko` module does not load automatically on most distributions. Ubuntu is the exception. It loads the module by default, which is exactly why the RxRPC path works so well there.

Put both exploits together and the gaps disappear. A single binary walks through Ubuntu 24.04.4, RHEL 10.1, CentOS Stream 10, AlmaLinux 10, Fedora 44, and openSUSE Tumbleweed, covering kernels all the way to 7.0.x, and drops a root shell every time.

One more thing worth knowing: if you already blocked `algif_aead` as a Copy Fail mitigation, that does absolutely nothing here. Dirty Frag operates in completely different parts of the kernel. The two exploits do not overlap at all.

## How This Went Public

Researcher Hyunwoo Kim found both bugs and was following the standard responsible disclosure process. He submitted the RxRPC patch to the netdev mailing list on April 29, 2026. The linux-distros embargo was scheduled to end on May 12, giving distributions time to get patches out first. On May 7, an unrelated third party published the ESP exploit publicly and blew the whole timeline apart. Kim disclosed everything the next day. No distribution had a fix ready. None do today.

## The Only Thing You Can Do Right Now

There is no kernel update to apply. The only mitigation available is removing the vulnerable modules entirely.

```sh
sh -c "printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' \
> /etc/modprobe.d/dirtyfrag.conf; rmmod esp4 esp6 rxrpc 2>/dev/null; true"
```

Running that knocks out IPsec ESP and RxRPC. Most systems will not notice, if yours does not depend on IPsec VPN tunnels over ESP mode, apply it without hesitation. If it does, weigh that against leaving a guaranteed root exploit sitting open on your box.

The PoC is available on [GitHub](https://github.com/V4bel/dirtyfrag) for verification purposes. Kim published everything in his [official disclosure post](https://www.openwall.com/lists/oss-security/2026/05/07/8) on May 8, after the embargo collapsed the day before.