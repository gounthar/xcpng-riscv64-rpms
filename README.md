# xcpng-riscv64-rpms

Experimental riscv64 RPMs of the XCP-ng toolstack (Xen and xapi) for an AlmaLinux 10 Kitten
dom0. They are what the RISC-V dom0 in our QEMU test environment is built from.

**This is not an XCP-ng release.** The packages are unsigned, built from development
branches, and only tested in QEMU (TCG). Use them to reproduce or work on XCP-ng on RISC-V,
not to run anything you care about.

## What is in a release

Each release is one tarball, named after the build round, holding:

- `rpms/`: the binary RPMs, with dnf metadata (`repodata/`)
- `srpms/`: the source RPMs they were built from
- `MANIFEST.tsv`: name, version-release, arch and licence of each binary RPM
- `SHA256SUMS`: checksums of every RPM

Round 14 has 39 binary RPMs: Xen 4.18 for RISC-V (hypervisor, tools, dom0 libraries, OCaml
bindings), the xapi toolstack (xapi, xe, xenopsd, xcp-networkd, xcp-rrdd and the rest), `qemu`
(only as the Xen PV backend behind a VM's graphical console in XO), and `busybox`, `vncterm`,
`xcp-featured`, `xcp-python-libs`, `xxhash`, `yajl` and `libempserver`, which Kitten has no
riscv64 build of. Each release's notes say what changed since the previous round and what ran.

## Using them

```bash
gh release download round-14 -R gounthar/xcpng-riscv64-rpms
sha256sum -c xcpng-riscv64-rpms-round-14.tar.sha256
tar -xf xcpng-riscv64-rpms-round-14.tar
cd xcpng-riscv64-rpms-round-14 && sha256sum -c SHA256SUMS
```

As a local dnf repository, on a riscv64 Kitten system or in a Kitten root:

```bash
dnf --repofrompath=xcpng-rv,file://$PWD/rpms --setopt=xcpng-rv.gpgcheck=0 \
    --enablerepo=xcpng-rv install xapi-core xapi-xe xenopsd-xc xen-tools
```

Add `qemu` to that list for the graphical console. Round 14 installs over round 13 with `dnf upgrade`.

These RPMs are one input to the dom0 image of our QEMU test environment, not all of it. The
image builder, `build-rootfs-rpm.sh` in
[baptleduc/hypervisor-dev](https://github.com/baptleduc/hypervisor-dev/tree/xapi-riscv/docker/riscv/kitten-dom0/rpm),
branch `xapi-riscv`, also needs a payload of RISC-V-specific files (hotplug scripts, a storage
driver, the guest kernel) built from the Xen and xen-api trees. So this repository alone does
not give you a dom0.

Our test dom0 runs round 13's xapi packages and `qemu`, installed with dnf, on Xen packages from
an earlier build (`9ede04b70e`). A dom0 image built only from round 14's RPMs has also run a guest (see the round 14 notes).

## How they were built

With the [meta-xcpng](https://github.com/xcp-ng/meta-xcpng) BitBake layer (branch
`ydi/meta-xcpng`, plus the riscv64 changes on
[gounthar/meta-xcpng `riscv64/meta-xcpng`](https://github.com/gounthar/meta-xcpng/tree/riscv64/meta-xcpng)), inside a riscv64 AlmaLinux Kitten container under
qemu-user on an x86_64 host. There is no cross-compilation: every compiler ran emulated.

| Package | Source |
|---|---|
| `xen-*` | [Baptiste Le Duc's Xen tree](https://gitlab.com/xen-project/people/baptleduc/xen), branch `xapi/investigation`, plus RISC-V fixes, at `0ef4cdf884` |
| `xapi-*`, `xenopsd*`, `xcp-*` | [baptleduc/xen-api](https://github.com/baptleduc/xen-api), branch `riscv`, plus RISC-V fixes; rounds 13 and 14 at `245b33a319` ([gounthar/xen-api `riscv64-rpms-round-13`](https://github.com/gounthar/xen-api/tree/riscv64-rpms-round-13)) |
| `qemu` | XCP-ng's QEMU 10.1.0 ([xcp-ng-rpms/qemu](https://github.com/xcp-ng-rpms/qemu), branch `jvr/9-arm`), with two patches for a riscv64 host |
| `yajl`, `xxhash`, `busybox` | the AlmaLinux Kitten and EPEL 10 source RPMs, rebuilt unchanged or nearly |
| `vncterm`, `xcp-featured`, `xcp-python-libs`, `xcp-ng-release`, `libempserver` | [xcp-ng-rpms](https://github.com/xcp-ng-rpms) and XCP-ng's own repositories, with riscv64 fixes |

The exact sources, patches and spec files are in the source RPMs.

## Licences

Each package keeps its own licence, listed in `MANIFEST.tsv` and in the RPM header. This
repository's own text is under CC-BY-4.0.
