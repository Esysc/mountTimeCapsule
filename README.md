# mountTimeCapsule

Mount legacy Apple Time Capsule drives from Linux (via an isolated Vagrant VM)
or from macOS (native SMB/Afp where available).

IMPORTANT: Time Capsule devices may require SMBv1 (NT1) to access their
shares. SMBv1 is insecure — this project isolates SMBv1 access inside a
Vagrant VM so your host does not need to enable SMBv1. Prefer modern storage
when possible.

## Development

This repository uses [pre-commit](https://pre-commit.com/) for code quality
checks.

### Setup

```bash
pip install pre-commit
pre-commit install
```

## Quick Start (Linux)

Prerequisites: `vagrant` and a VirtualBox provider installed on the host.

1. Start or reprovision the VM (the `Vagrantfile` will install `samba` inside
   the VM):

```bash
vagrant up --provision
# or, if the VM is running already:
vagrant reload --provision
```

2. Run the mount script (it will mount the Time Capsule inside the VM using
   SMBv1 and re-share the mount to the host with SMB3):

```bash
./mountTimecapsule -i 192.168.1.100 -v Data -m /mnt/timecapsule
```

3. Unmount and destroy the VM when finished:

```bash
./mountTimecapsule --umount
# this will attempt to unmount and run `vagrant destroy -f` on Linux
```

## Installation (short)

- Linux: install `vagrant` and VirtualBox via your package manager or from
  upstream downloads.
- macOS: no extra tools are required for native mounts; the script will
  attempt to use `mount_smbfs` on macOS.

## Configuration

Set these environment variables (optional; script defaults are used otherwise).
They can be overridden by the corresponding command-line options shown below.

- `TIMECAPSULE_IP` — IP address of the Time Capsule (default: 192.168.0.3)
- `TIMECAPSULE_VOLUME` — Share name on the Time Capsule (default: Data)
- `TIMECAPSULE_USER` — Your Time Capsule username (optional)
- `TIMECAPSULE_PASS` — Your Time Capsule password (optional)
- `TIMECAPSULE_MOUNT_POINT` — Host mount point for the re-shared VM share
  (default: $HOME/TimeCapsule)

Command-line options (override environment variables):

- `-i IP` — Set Time Capsule IP
- `-v VOLUME` — Set share name (e.g., Data)
- `-u USERNAME` — Set username
- `-p PASSWORD` — Set password
- `-m MOUNT_POINT` — Set host mount point
- `--umount` — Unmount and (on Linux) destroy the Vagrant VM

Examples:

```bash
# Using environment variables
export TIMECAPSULE_IP="192.168.1.100"
export TIMECAPSULE_USER="myuser"
export TIMECAPSULE_PASS="mypass"
export TIMECAPSULE_MOUNT_POINT="$HOME/TimeCapsule"
./mountTimecapsule

# Using command-line options
./mountTimecapsule -i 192.168.1.100 -v Data -u myuser -p mypass -m /mnt/tc

# Unmount and destroy the VM (Linux)
./mountTimecapsule --umount
```

## How it works (brief)

1. On Linux, the script starts a Vagrant VM (Ubuntu) and mounts the Time
   Capsule inside the VM using SMBv1 (NT1) if required.
2. The VM runs `smbd` and re-shares the mounted folder to the host with a
   modern protocol (SMB2/SMB3). The host mounts the VM share using SMB3 so the
   host never needs to enable SMBv1.

## Troubleshooting

- If host mount fails with CIFS errors, check the VM IP and connectivity:

```bash
# get host-accessible VM IP (skips NAT 10.0.2.x)
vagrant ssh -c "ip -4 addr show scope global | awk '/inet /{print \$2}' | cut -d/ -f1 | grep -v '^10\\.0\\.2\\.' | head -n1"

# from host: test TCP reachability to Samba
nc -vz VM_IP 445
nc -vz VM_IP 139

# list shares from host (SMB3)
smbclient -L //VM_IP -N -m SMB3
```

- Inside the VM: check `smbd` and logs:

```bash
vagrant ssh -c "sudo systemctl status smbd --no-pager --no-hostname"
vagrant ssh -c "sudo journalctl -u smbd -n 200 --no-hostname"
vagrant ssh -c "sudo testparm -s"
```

- If the script still targets a NAT IP like `10.0.2.15`, re-run
  `vagrant reload --provision` after updating the `Vagrantfile` (the VM is
  configured to use a host-accessible private IP so the host can reach the
  Samba service).

## Security

- SMBv1 is insecure. This project isolates SMBv1 inside a VM to reduce host
  exposure, but you should migrate data off Time Capsule hardware when
  possible.

## Changes

- Linux uses a Vagrant VM to isolate legacy SMBv1 access
- VM re-shares the mounted Time Capsule using SMB2/3 to the host
- `--umount` on Linux now destroys the VM after unmounting
- README updated with Quick Start and troubleshooting
