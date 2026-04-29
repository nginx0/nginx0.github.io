---
title: "Copy Fail: The 732-Byte Exploit Threatening the Linux Kernel"
categories: [Blog]
tags: [cve, LPE]
render_with_liquid: false
media_subpath: /images/blog_copyfail/
image:
  path: banner.webp
---

[![Link](header.png){: width="1200" height="630" }](https://copy.fail/)

The flaw has existed in the kernel since 2017 and reliably affects nearly every major distribution, including Ubuntu, Red Hat Enterprise Linux (RHEL), Amazon Linux, and SUSE.

### [CVE-2026-31431](https://github.com/theori-io/copy-fail-CVE-2026-31431)

The "Copy Fail" vulnerability represents a rare class of security flaws: a local privilege escalation (LPE) that bypasses the complexities of race conditions or heap spraying. Discovered by the Theori Xint Code team, this bug allows an unprivileged user to overwrite read-only system files in memory, turning a standard 732-byte Python script into a master key for root access.

### The Technical Core: A Collision of Three Features

The vulnerability is not found in a single line of code, but in the interaction between three separate kernel mechanisms.

## Zero-Copy Data Transfer (splice)
The Linux kernel uses the `splice()` system call to move data between file descriptors without copying it to userspace. When an attacker splices a read-only file (like `/usr/bin/su`) into a socket, the kernel doesn't create a copy; it simply passes a reference to the Page Cache, the kernel’s internal memory where it stores disk data for speed.

## The AF_ALG Interface & In-Place Crypto
The `AF_ALG` socket type allows userspace to use the kernel's internal cryptography API. In 2017, an optimization was added to the AEAD (**Authenticated Encryption with Associated Data**) implementation. This allowed the kernel to perform "in-place" operations, where the source and destination for the data are the same memory address.

## The "Scratch-Pad" Overflow 
A long-standing bug in the `authencesn` crypto wrapper commonly used for IPsec, contains a minor logic error. During decryption, it writes 4 bytes of data immediately following the output buffer as a temporary "scratch-pad".

![](demo.gif){: width="1024" height="903"}


### The Exploit Chain: From User to Root

The function `c(f, t, c)` is designed to write 4 bytes of data into a file that is supposed to be read-only.

- **Socket Setup**: It opens an `AF_ALG` socket (domain 38). This is the userspace interface for the kernel's internal crypto API.

- **Targeting the Bug**: It binds to `authencesn`. This specific crypto wrapper has a legacy bug: during decryption, it writes a small "scratch-pad" (4 bytes) immediately following the output buffer.

- **The Memory Trick (splice)**: The script uses `g.splice` to move data from the target file (/usr/bin/su) into the socket. Because of a 2017 kernel optimization, this is a Zero-Copy operation. Instead of making a copy of the file, the kernel points the crypto engine directly at the Page Cache (the version of the file sitting in RAM).

## The Overwrite (The "Copy Fail")

When `u.recv()` is called, the kernel attempts to decrypt data using the Page Cache as the "in-place" buffer.

- **The Payload**: The script sends a message that triggers the 4-bytes scratch-pad write.

- **Bypassing Integrity**: The HMAC integrity check will eventually fail (the try...except block catches this), but the kernel performs that 4-bytes write before the check is finalized.

- **Result**: Even though the file is read-only, the kernel operating in Ring 0 just wrote 4 bytes into the system's memory for that file.

## The Injection (Loop)

The variable `e` is a compressed blob of shellcode.

- **Painting the Binary**: The while loop iterates through the shellcode, calling the function c to write it 4 bytes at a time into the Page Cache where /usr/bin/su is stored. It is essentially "live-patching" the su binary in the system's RAM.

## Execution (Root Access)

- `g.system("su")`: Once the loop finishes, the script executes the `su` command.

The exploit turns the kernel efficiency against itself. By using `splice` to avoid copying data, the kernel accidentally allows its internal crypto "scratchpad" to overwrite protected system memory. Since this happens in the Page Cache, the corruption is shared across the entire system until the next reboot.

## Primary Solution: Kernel Update

The definitive fix involves applying mainline commit `a664bf3d603d`. This patch reverts the 2017 in-place optimization within algif_aead.c. By strictly separating source and destination scatterlists, the kernel prevents read-only page-cache entries from being placed in a writable cryptographic path.

Run your distribution's package manager and reboot to ensure the patched kernel is active.

## Immediate Workaround : Disabling the Vulnerable Module

If an immediate reboot or patch is not feasible, you can neutralize the attack vector by disabling the `algif_aead` module. This is safe for nearly all production environments because core services—including SSH, LUKS (disk encryption), IPsec, and OpenSSL typically interact with the kernel’s crypto API directly rather than using the `AF_ALG` userspace interface.

Execute the following to prevent the module from loading:

```console
# Blacklist the module
echo "install algif_aead /bin/false" > /etc/modprobe.d/disable-algif.conf

# Unload the module from the current session
rmmod algif_aead 2>/dev/null || true
```

Because this exploit targets the shared page cache, a compromise in one container can affect the entire host. To implement defense in depth, use seccomp policies to explicitly block the creation of `AF_ALG` sockets. This prevents the exploit from even initializing, regardless of whether the underlying kernel is patched.


> Note on Persistence: While the corruption of the page cache is ephemeral (it will be cleared and reloaded cleanly from disk upon a reboot), any root shells or persistent backdoors established by an attacker before that reboot will remain active.
{: .prompt-tip }


The Proof-of-Concept (PoC) code is available on [GitHub](https://github.com/theori-io/copy-fail-CVE-2026-31431/blob/main/copy_fail_exp.py) for verification purposes.