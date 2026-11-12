# Running vphone-linux on Windows 10 (WSL2)

Same VM as on Windows 11: everything runs inside WSL2 Ubuntu
(`./setup/setup.sh`, then `./setup/create-vm.sh`). The scripts detect WSL
and adapt. This page lists what Windows 10 needs that is different from
[docs/WSL.md](WSL.md).

## Minimum Windows 10

- **Windows 10 version 21H2 (build 19044) or newer.** Older builds have
  no WSLg or systemd support — update Windows first.
- **WSL from the Microsoft Store** (`wsl --install` / `wsl --update` in
  PowerShell). The Store build is what carries WSLg and systemd on
  Windows 10; the old in-box WSL does not have them.
- **WSL 2 with systemd** (the default for new Ubuntu installs; otherwise
  add `[boot]` / `systemd=true` to `/etc/wsl.conf`).
- Check yours in PowerShell: `wsl --version` and `winver`. `wsl -l -v`
  must show Version 2 for the Ubuntu distro.

## Display: WSLg, with an X-server fallback

WSLg (Linux GUI apps on the Windows desktop) works on Windows 10 21H2+
with the Store WSL, so the iPhone window (`WINDOW=1`) works the same as
on Windows 11. If no window appears:

1. `wsl --update` in PowerShell, then `wsl --shutdown` and reopen.
2. Check inside WSL that a display is set: `echo $WAYLAND_DISPLAY $DISPLAY`.
   `tools/check-host.sh` reports "no WSLg display" when neither is set.
3. Still nothing (older build, or WSLg disabled): install VcXsrv on
   Windows, start it with "Disable access control", and in WSL export
   `DISPLAY=$(ip route | awk '/default/{print $3}'):0` before running
   `run-vphone.sh`. This is the pre-WSLg path; WSLg is preferred.

## Memory and cores

Same as Windows 11: the guest needs `MEM` (8 GB) in `/dev/shm` plus room
for QEMU and lavapipe — give WSL 16 GB or more in
`%UserProfile%\.wslconfig` (`memory=16GB` minimum, `processors=` to what
the PC has). Then `wsl --shutdown` and reopen.

A PC with less than 16 GB of RAM cannot meet this: the guest alone wants
8 GB in `/dev/shm`, and Windows itself needs room too. On such a PC the
VM may fail to start (`/dev/shm` full) or swap itself to a crawl — that is
a hardware limit, not a setup bug.

## The rest is identical to Windows 11

Kernel-tree build (`tools/wsl-kernel-tree.sh`), usbip, vhost-vsock,
lavapipe rendering, and the known problems (first-boot watchdog panic,
restore USB wait) are the same — see [docs/WSL.md](WSL.md). On a 2-core
host, run with `SMP=2` and scale 1, and expect the first boot to be very
slow.
