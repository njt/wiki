---
url: https://brownfinesecurity.com/blog/introducing-moria-and-mithril
date_fetched: 2026-09-13
---

# Moria and Mithril: IoT Firmware Extraction and Analysis Without External Dependencies

Every IoT firmware assessment starts the same way: you have a `.bin` blob and you need to turn it into a filesystem tree you can read. For years the answer has been binwalk or unblob. Both are good tools. Both also have a problem that anyone who has set up a fresh analysis box knows well.

They shell out to a small army of external extractors. Sasquatch for SquashFS, jefferson for JFFS2, ubi_reader for UBIFS, yaffshiv for YAFFS2, 7z, cramfsck, and more. Half of them are unmaintained Python from the mid-2010s, several need patched forks to handle vendor quirks, and every one is another thing to install, pin, and keep working across machines. When an extraction fails, you are debugging someone else's decade-old script instead of looking at the firmware.

I got tired of it, so I wrote two tools to replace that pipeline. They are open source and I welcome your PRs and Issues.

- **moria**identifies and unpacks firmware. It maps the bytes.
- **mithril**reads the contents of what moria unpacks: secrets, an SBOM, known CVEs, and licenses.

Both are single C++ binaries with no external extractor dependencies, and both emit clean JSON so scripts and LLM agents can drive them as easily as a person can.

## The design goal: zero external extractors

moria does not shell out. Every filesystem it supports is parsed and unpacked in-process by C++ code compiled into the binary. The only shared libraries it links are the compression codecs:

```
$ ldd build/moria
	libz.so.1 => /usr/lib/libz.so.1
	liblzma.so.5 => /usr/lib/liblzma.so.5
	liblz4.so.1 => /usr/lib/liblz4.so.1
	libzstd.so.1 => /usr/lib/libzstd.so.1
	libstdc++.so.6 => /usr/lib/libstdc++.so.6
	libm.so.6 => /usr/lib/libm.so.6
	libgcc_s.so.1 => /usr/lib/libgcc_s.so.1
	libc.so.6 => /usr/lib/libc.so.6
```
Four compression codecs and the C/C++ runtime. That is the whole list. No sasquatch, no jefferson, no ubi_reader on the PATH. Copy the binary to a box and it runs. This also means extraction needs no root: writes go through `openat` with `O_NOFOLLOW`, so a hostile archive cannot traverse out of the output directory or follow a symlink into your filesystem.

## moria: identify first, then unpack

Point moria at a raw image and by default it gives you a map, not a pile of files. Here is a TP-Link Tapo C210 camera image:

```
$ moria tapo-c210-fw.bin
OFFSET    SIZE     TYPE         TIER        NOTES
0x10000   65.7 KB  uimage       verified    big/arm lzma  "MVX4##I6B0g09280b5CM_UBT1501#XVM"
0x2dcb7   274 B    certificate  consistent  little
… +5 more certificate (8.7 KB, -A to list)
0x2ddc9   332 B    private_key  consistent  little
0x30100   192 KB   gzip         structural  little
0x60000   66.1 KB  uimage       verified    big/arm lzma  "MVX4##I6B0g09280b5CM_UBT1501#XVM"
0x80200   1.4 MB   uimage       verified    big/arm none  "Linux-4.9.84"
0x1e6200  2.3 MB   squashfs     consistent  little v4.0 xz
0x440000  3.6 MB   squashfs     consistent  little v4.0 xz
```
Every finding has a byte offset, a size, a type, and a confidence tier. `verified` means the structure passed a real structural check (here, a uImage header CRC32). `consistent` means cross-field checks agree. `structural` and below are weaker. That confidence ladder is the point: moria tells you how sure it is instead of printing a magic-byte match and hoping.

It also flagged an embedded `private_key` and a handful of X.509 certificates sitting in the raw flash before any filesystem, which is exactly the kind of thing you want to see on the first pass.

When you do want the files, `-e` unpacks everything, recursively, in under a second:

```
$ moria -e tapo-c210-fw.bin
-> extracted to tapo-c210-fw.bin.extracted/
Files analyzed:  1
Bytes analyzed:  8.0 MB
Findings:        13
Elapsed:         0.898 s
```
Each region lands in its own directory named by offset and type, and a `manifest.json` records exactly what happened, including partial extractions and any guard that tripped:

```
{
  "source": "tapo-c210-fw.bin",
  "extracted": [
    {
      "offset": 1991168,
      "type": "squashfs",
      "root": "0x1e6200-squashfs",
      "status": "ok",
      "files": 407,
      "dirs": 59,
      "symlinks": 208,
      "bytes": 5292484,
      "depth": 1
    }
  ]
}
```
The recursion is automatic. A gzip-wrapped kernel inside a uImage inside a raw image unpacks all the way down, and a UBI image is rebuilt volume by volume, with `--depth`, `--max-files`, and a decompression-ratio cap bounding hostile input.

## Rewriting the old extractors in C++

The interesting cases are the filesystems whose canonical extractors have aged out. JFFS2, YAFFS2, cramfs, romfs, and UBIFS are still everywhere in real devices, but the tools people reach for (jefferson, yaffshiv, ubi_reader) are Python, slow on large images, and in some cases effectively unmaintained.

moria reimplements these from the on-disk format up in C++. Take a VStarcam CB73 camera, a MIPS device with a JFFS2 partition:

```
$ moria vstarcam-cb73-fw.bin
OFFSET    SIZE     TYPE      TIER        NOTES
0x358c0   41.8 KB  uboot     magic       little
0x40000   1.4 MB   uimage    verified    big/mips lzma  "Linux-3.10.14__isvp_swan_1.0__"
0x1c0000  4.5 MB   squashfs  consistent  little v4.0 xz
0x64000c  1.7 MB   jffs2     verified    little x651
```
That `jffs2` region carries the writable config partition. moria walks the JFFS2 node log, replays it in order, and reconstructs the final file tree directly, no external `jefferson` and no MTD emulation:

```
$ moria -e vstarcam-cb73-fw.bin
-> extracted to vstarcam-cb73-fw.bin.extracted/
Elapsed:         0.299 s
```
The whole image, U-Boot plus MIPS kernel plus SquashFS rootfs plus JFFS2, is mapped and unpacked in 0.3 seconds. Being in-process buys more than a clean dependency list. Parsing the format in native code with one bounds-checked reader is faster and more predictable than spawning a Python interpreter per region and scraping its stdout.

## mithril: reading what moria unpacked

Once you have a rootfs tree, moria's job is done. mithril takes over and reads the contents. It runs four passes: secrets, SBOM, CVEs, and licenses. Point it at the extracted directory.

**Secrets.** mithril rides a deterministic confidence ladder: `pattern` (right shape), `structural` (a parsed PEM key or JWT), and `validated` (a recomputed checksum, such as a crypt hash or a GitHub token). On the VStarcam tree it recovered the whole root credential story with no false-positive noise:

```
$ mithril --secrets vstarcam-cb73-fw.bin.extracted/
2 secrets (2 checksum-validated) in 126 files
  empty-password  validated  0x1c0000-squashfs/etc/shadow@0    root
  empty-password  validated  0x1c0000-squashfs/etc/shadow@318  default
3 notable files
  shadow-file  0x1c0000-squashfs/etc/shadow
  wifi-config  0x64000c-jffs2/param/hostapd.conf
  wifi-config  0x64000c-jffs2/param/wpa_supplicant.conf
```
Both `root` and `default` ship with empty password hashes, parsed straight out of `/etc/shadow` and flagged. On the Tapo image the secrets pass instead surfaced a real RSA private key and classified it structurally:

```
{
  "path": "0x1e6200-squashfs/etc/tp_manage/priv-key.pem",
  "type": "private-key",
  "category": "secret",
  "confidence": 80,
  "confidence_tier": "structural",
  "label": "-----BEGIN RSA PRIVATE KEY-----",
  "evidence": "PEM private-key header"
}
```
**SBOM.** mithril builds a bill of materials from package databases (dpkg, opkg, apk, rpm), language manifests, ELF version banners, versioned libc filenames, and the kernel banner, so it finds components even in a stripped image with no package manager. The Tapo rootfs yielded 94 components, and the versions tell the story:

```
$ mithril --sbom tapo-c210-fw.bin.extracted/
94 components
  busybox   1.19.4    binary-version   pkg:generic/busybox@1.19.4
  uclibc    1.0.31    filename         pkg:generic/uclibc@1.0.31
  curl      7.29.0    binary-version   pkg:generic/curl@7.29.0
  openssl   1.0.2e    binary-version   pkg:generic/openssl@1.0.2e
  zlib      1.2.7     binary-version   pkg:generic/zlib@1.2.7
  ...
```
The SBOM is emitted as both CycloneDX and SPDX. Each component is keyed on a purl so it joins cleanly against vulnerability data.

**CVEs.** That join is the CVE pass. It reads a local mirror built from OSV, NVD, CISA KEV, and FIRST EPSS, so a scan makes no network calls. openssl 1.0.2e from 2015 lights up exactly as you would expect:

```
$ mithril --cve tapo-c210-fw.bin.extracted/
214 CVEs
...
  CVE-2015-3197     openssl@1.0.2e      nvd-cpe-range (openssl:openssl) epss=0.11
  CVE-2016-0701     openssl@1.0.2e      nvd-cpe-range (openssl:openssl) epss=0.84
  ...
  CVE-2016-2183     openssl@1.0.2e      nvd-cpe-range (openssl:openssl) epss=0.96
  CVE-2022-2068     openssl@1.0.2e      nvd-cpe-range (openssl:openssl) epss=0.96
```
Every match records its basis (exact version, affected range, or CPE) and is annotated with EPSS so you can sort by real-world exploitation probability instead of drowning in 214 undifferentiated CVEs.

The kernel gets special handling. On top of the component join, mithril runs a curated kernel-CVE checklist that is version and kconfig gated, so it cuts the kernel noise down to what actually applies to the running config. Here it is on the VStarcam tree, whose kernel banner reads 3.10.14:

```
$ mithril --cve vstarcam-cb73-fw.bin.extracted/
16 applicable kernel CVEs (of 42 in range for 3.10.14)  [4 ruled out, 22 undetermined]
  config: inferred (kallsyms; no .config)
  CVE-2016-5195   LPE   Dirty COW: COW race in core mm; universal, public exploits.  [KEV]  epss=0.84
  CVE-2021-33909  LPE   Sequoia: seq_file size_t underflow; public exploit.
  CVE-2019-13272  LPE   ptrace PTRACE_TRACEME parent cred handling; public exploit.  [KEV]  epss=0.52
  ...
  ruled out by config:
    CVE-2016-0728   requires CONFIG_KEYS (not enabled, kallsyms)
    CVE-2022-0185   requires CONFIG_USER_NS (not enabled, kallsyms)
```
Dirty COW (CVE-2016-5195) is in CISA's Known Exploited Vulnerabilities catalog and applies to this 3.10.14 kernel, so it lands at the top with an EPSS of 0.84. Below the applicable set, mithril shows what it ruled out and why: CVE-2016-0728 needs `CONFIG_KEYS`, which the kallsyms-inferred config does not have enabled. That is the difference between a checklist you can act on and a wall of kernel CVEs you have to triage by hand.

## Built for agents: -j everywhere

Both tools print a human-readable table by default and a full JSON object with `-j`. This was a first-class design goal, not an afterthought, because a lot of my firmware triage now runs through Claude Code skills, and an LLM agent parses JSON far more reliably than it scrapes a table.

Everything you saw above is available as structured output. A single moria finding, in full:

```
{
  "offset": 65536,
  "type": "uimage",
  "category": "container",
  "confidence": 99,
  "confidence_tier": "verified",
  "endian": "big",
  "arch": "arm",
  "compression": "lzma",
  "label": "MVX4##I6B0g09280b5CM_UBT1501#XVM",
  "evidence": "header CRC32 ok",
  "description": "U-Boot legacy uImage: kernel/ramdisk container with a CRC-checked header.",
  "references": [
    { "title": "U-Boot image format (image.h)", "url": "https://source.denx.de/u-boot/u-boot/-/blob/master/include/image.h" }
  ]
}
```
Every object carries a `schema_version`, offsets, confidence, and sourced references. An agent can chain `moria -j -e firmware.bin` into `mithril -j firmware.bin.extracted/` and reason over the results without a single fragile text parser in the loop. The whole pipeline is two commands:

```
$ moria -e firmware.bin            # identify + unpack  -> firmware.bin.extracted/
$ mithril firmware.bin.extracted/  # secrets + SBOM + CVEs + licenses
```
## Get the tools

Both are on GitHub, MIT licensed, and build with cmake and a C++20 compiler.

- **moria**: github.com/nmatt0/moria
- **mithril**: github.com/nmatt0/mithril

```
$ cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
$ cmake --build build -j
$ cp build/moria ~/.local/bin    # self-contained, signatures are embedded
```
moria handles the identification and extraction, mithril handles the content analysis, and neither one needs you to install a museum of aging Python extractors first.

## Contributing

These are actively developed and I want your help making them better. Contributions and bug reports are genuinely welcome:

- **Found a format or device that trips them up?**Open an issue with a sample (or a link to public firmware) on moria or mithril. Real-world firmware that breaks the extractor is the most useful thing you can send.
- **Want to add a signature, a filesystem, or a secret/CVE rule?**Pull requests are open. Adding a moria signature is often just a- `.toml`file, and both repos ship tests you can extend.
- **Want to talk it through first?**I run a- `#moria-mithril-dev`channel on my IoT Hacker Hideout Discord for questions, ideas, and work-in-progress: discord.gg/4FPrJJ8usN.

If you do IoT or firmware work, give them a try and tell me where they fall short.

### Need IoT Security Expertise?

Brown Fine Security provides expert IoT penetration testing to help secure your connected devices. From hardware analysis to cloud API testing, we uncover vulnerabilities before attackers do.

Get a Free Consultation### Want to Learn IoT Hacking?

Ready to break into IoT security? Our hands-on training courses teach real-world hardware hacking, firmware analysis, and IoT exploitation techniques used by professional pentesters.

Explore Training Courses
