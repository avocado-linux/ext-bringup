# ext-bringup

Avocado extension for a board with nothing provisioned yet: a set of hardware bringup tools and a serial console
that logs in as root without a password. **Do not use it in production.**

## Supported targets

All targets. The tool packages were checked against the 2026/next feeds for jetson-orin-nano, jetson-agx-orin,
qemux86-64, raspberrypi5 and imx93-frdm; none was missing. The 2024 feed was not checked by hand. The `test.yml`
workflow builds the extension against the live 2026/next and 2024/next qemux86-64 feeds on every pull request.

## What's included

- **Tools**: `ethtool`, `iproute2`, `iputils`, `tcpdump` (network); `i2c-tools`, `libgpiod`, `libgpiod-tools`,
  `usbutils`, `pciutils`, `devmem2` (buses and memory); `e2fsprogs`, `dosfstools`, `parted` (storage); `dtc`
  (device tree); `strace`, `procps`, `htop`, `curl`.
- **Serial autologin**: a drop-in for `serial-getty@.service` that runs `agetty --autologin root`. It applies to
  every serial console, so it does not name a tty. The extension restarts the running getties when it merges.
  Every merge reruns the hook, so a refresh or deploy restarts the serial getty and ends an open console session.

## Why the console and not SSH

The base rootfs ships root locked (`root:*`), and `/root` is not writable before the first boot, so a fresh board
has no credentials to log in with. If you can seed the var partition at build time, install the key that way: add
a `var_files` entry to the runtime that copies a file named `root` into `lib/ssh/authorized_keys.d/` (see the
Avocado guide "Seeding the var partition"). The console is the way in for a board with no key yet. From it, add
your public key to the per-user file `avocado-ext-sshd-dev` reads authorized keys from:

```sh
mkdir -p /var/lib/ssh/authorized_keys.d
echo 'ssh-ed25519 AAAA... you@host' >> /var/lib/ssh/authorized_keys.d/root
```

The directory and the file must be owned by root and not group- or world-writable, because sshd checks their
ownership and permissions and ignores the key otherwise. This location requires `avocado-ext-sshd-dev` 0.1.1 or
newer.

## Using this extension

Declare it as a package-sourced extension in your `avocado.yaml` and add it to a runtime, next to the other
development extensions:

```yaml
extensions:
  avocado-ext-bringup:
    source:
      type: package
      version: "*"        # or pin an exact version

runtimes:
  dev:
    extensions:
      - avocado-ext-dev
      - avocado-ext-sshd-dev
      - avocado-ext-bringup
```

Then install and build:

```sh
avocado install
avocado build
```

## Notes

- Not yet verified through the stock `serial-getty@.service` start path.
- Anyone with physical or serial access gets a root shell. That is the point of the extension, and why it must not
  ship in a production runtime.

## Releasing

Versions are plain semver. Open a pull request that sets the new version in `avocado.yaml` and `CHANGELOG.md`; the
`test.yml` workflow runs against the live feeds. After it merges, tag the commit with the exact `avocado.yaml`
version (`git tag 0.1.0 && git push --tags`) and `release.yml` packages and publishes it.
