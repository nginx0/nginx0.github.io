---
title: "Fragnesia: Arbitrary Kernel Page-Cache Writes Through XFRM ESP Logic Bug"
categories: [Blog]
tags: [cve, LPE]
render_with_liquid: false
media_subpath: /images/blog_fragnesia/
image:
  path: banner.png
---

[![Link](header.png){: width="1200" height="600" }](https://github.com/v12-security/)

A new Linux local privilege escalation exploit has emerged from the same subsystem as Dirty Frag and if you patched instead of mitigated, you may still be exposed.

### What is the Linux XFRM ESP-in-TCP subsystem?

XFRM is the Linux kernel's transform framework, it handles IPsec encryption and decryption of network traffic. ESP-in-TCP is a mode that allows IPsec Encapsulating Security Payload (ESP) packets to be tunneled inside a TCP stream, typically to traverse firewalls that block UDP. Because this subsystem operates deep inside the kernel's networking stack with elevated privileges, it represents a high-value attack surface.

### What is a page cache write primitive?

The Linux kernel caches file contents in memory for performance, this is the page cache. When you execute a binary like `/usr/bin/su`, the kernel reads it into the page cache and maps it into memory. Crucially, these cached pages are supposed to be read-only for unprivileged users. A page cache write primitive is the ability to write into these cached pages without touching the file on disk, without root, and without the kernel noticing anything is wrong.

The consequence: you can corrupt an executable in memory. Any subsequent execution of that binary runs your injected code instead. This is the same class of primitive exploited by the famous **Dirty Pipe** (CVE-2022-0847).

![](wednesday.png){: width="800" height="450"}

### Dirty Frag

Dirty Frag abused the XFRM ESP-in-TCP subsystem to achieve exactly this kind of page cache write. It works by splicing a target file's pages into a TCP socket's receive queue, then racing the kernel, during a specific timing window between two kernel operations to corrupt the cached page before the kernel processes it correctly.

**The race condition is the key weakness of Dirty Frag as an exploit.** Because it depends on winning a timing race, it is probabilistic. It may require dozens or hundreds of attempts. In a real attack scenario, this adds noise, increases the chance of detection and reduces reliability.

Dirty Frag was assigned **CVE-2026-43284** (XFRM-ESP) and **CVE-2026-43500** (RxRPC), with an effective kernel exposure window stretching from 2017 to May 2026, approximately nine years.

## Introducing Fragnesia

Fragnesia was discovered by [William Bowling on the V12 sec team](https://v12.sh/). It is a member of the Dirty Frag vulnerability class; same subsystem, same class of impact but it is a distinct bug in a distinct code path, assigned its own separate patch.

![](demo.gif){: width="1080" height="1080"}

Like Dirty Frag, Fragnesia:

- Abuses the Linux XFRM ESP-in-TCP subsystem
- Achieves arbitrary byte writes into the kernel page cache of read-only files
- Targets `/usr/bin/su` to escalate privileges
- Drops an interactive root shell on success
- Affects all kernel versions affected by Dirty Frag

But the mechanism is fundamentally different and in one critical way, significantly more dangerous.

## Why "Fragnesia"?

The name comes from the core bug itself, the socket buffer `(skb)` forgets that a fragment is shared during coalescing.

## How Fragnesia's write primitive works

Before the splice happens, the exploit calls `unshare(CLONE_NEWUSER | CLONE_NEWNET)` to bootstrap `CAP_NET_ADMIN` inside a user namespace, no real host privileges required. This is what makes the exploit universal, any unprivileged local user can pull this off.

Fragnesia exploits what happens when a TCP socket transitions to **espintcp ULP (Upper Layer Protocol) mode after data has already been spliced from a file into the receive queue**.

The attack sequence is as follows:

1. **User + network namespace setup.**  
   The exploit begins with `unshare(CLONE_NEWUSER | CLONE_NEWNET)`, creating a fresh user and network namespace where the process gains `CAP_NET_ADMIN`without requiring real privileges on the host. This allows unprivileged users to configure XFRM/IPsec state entirely inside the isolated namespace.

2. **ESP-in-TCP XFRM state installation.**  
   Inside the new network namespace, the exploit installs a transport-mode ESP-in-TCP security association through `NETLINK_XFRM`. The SA uses AES-128-GCM with a known key and SPI `0x100`, ensuring the resulting keystream is fully predictable.

3. **Keystream lookup generation.**  
   For AES-GCM, the counter block used for sequence position 2 has the form: `[ salt || IV || 00000002 ]` 
Encrypting this block under the chosen AES key produces the corresponding 16-byte GCM keystream block. The exploit only needs byte 0 of that stream. By varying the lower 32 bits of the 8-byte IV (nonce), all 256 possible keystream byte values become reachable within the first 65,536 nonce values. A lookup table mapping keystream byte → IV is generated once using `AF_ALG` and reused for all subsequent writes.

4. **Splice-then-ULP trigger.**  
A sender/receiver process pair is forked, each controlling one end of a TCP socket pair. The sender splices 4096 bytes from the target file beginning at the byte intended for corruption directly into the TCP stream, preceded by an ESP-in-TCP length field and a crafted ESP header containing the selected IV. The receiver intentionally delays enabling `TCP_ULP=espintcp` until the data is already queued inside the socket receive buffer.

5. **In-place page-cache corruption.**  
Once `espintcp` ULP mode is enabled, the kernel retroactively interprets the queued splice-backed data as an ESP record and performs AES-GCM decryption in place. During this process, the generated keystream is XORed directly into the underlying splice-mapped page, which is the same physical page backing the VFS page-cache entry for the target file. This produces a controlled single-byte modification in cached file data without writing through normal filesystem paths.

6. **Deterministic byte-by-byte overwrite.**  
The trigger sequence is repeated for every payload byte that differs from the desired value. For each iteration, the exploit computes the required XOR delta, selects the corresponding IV from the precomputed lookup table, increments the ESP sequence number, and executes a fresh splice/ULP cycle. The result is a reliable arbitrary page-cache overwrite primitive.

7. **Execution phase.**  
After all payload bytes have been patched and verified in the page cache, the exploit executes the modified cached binary (for example `/usr/bin/su`). Because execution occurs from the altered in-memory page-cache contents, the injected payload runs with the binary’s original privileges and calls `setresuid(0,0,0)` / `setresgid(0,0,0)` and execs `/bin/sh`.

## Dirty Frag vs Fragnesia at a glance

| Property | Dirty Frag | Fragnesia |
|---|---|---|
| Vulnerability class | Dirty Frag | Dirty Frag |
| Subsystem | XFRM ESP-in-TCP | XFRM ESP-in-TCP |
| Bug | Separate | Separate |
| Patch | Separate | Separate |
| Race condition required | Yes | **No** |
| Reliability | Probabilistic | **Deterministic** |
| Exploit complexity | Higher | Lower |
| Affected kernel range | 2017–May 2026 | Same |
| Mitigation | Module blacklist | Same module blacklist |

### Mitigation

The mitigation for both Dirty Frag and Fragnesia is the same blacklist the kernel modules that expose the vulnerable surface.

Blacklist the affected modules:

```bash
rmmod esp4 esp6 rxrpc
printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' > /etc/modprobe.d/dirtyfrag.conf
```

This takes effect immediately and persists across reboots. It disables ESP-in-TCP and RxRPC functionality. If your systems depend on these for IPsec tunneling, assess the operational impact before applying.

### Post-exploitation

After the exploit runs, `/usr/bin/su` in the **page cache** contains the injected stub. The file on disk is untouched, but any subsequent execution of `su` will re-spawn a root shell until the page is evicted from memory.

```bash
# Drop the page cache
echo 1 | tee /proc/sys/vm/drop_caches
```
Failing to do this leaves a persistent in-memory backdoor active even after you believe you have contained the incident.

The XFRM ESP-in-TCP subsystem sat largely unaudited for approximately nine years while hosting at least two exploitable page cache write primitives. Dirty Frag drew attention to the surface, Fragnesia confirms the surface was not fully understood even after that attention.

The PoC is available on the [V12 security GitHub](https://github.com/v12-security/pocs/tree/main/fragnesia). All affected kernel versions are also affected by Dirty Frag. Apply mitigations and patches as described above.