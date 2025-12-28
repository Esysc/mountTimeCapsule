# mountTimeCapsule

Mount legacy Apple Time Capsule drives from Linux or macOS using Samba/SMB.

**Note**: Apple discontinued the Time Capsule in 2018. This script is for legacy
hardware only. For modern network storage, consider alternatives like:

- Modern NAS devices (Synology, QNAP, etc.)
- Cloud storage solutions
- USB drives connected to routers
- Network shares from other devices

## Development

This repository uses [pre-commit](https://pre-commit.com/) for code quality checks.

### Setup

```bash
pip install pre-commit
pre-commit install
```

Pre-commit will run checks on shell scripts (shellcheck) and markdown files
(markdownlint) before each commit. Markdown formatting issues will be
automatically fixed.

## Usage

Everything is in the source file....

**Note about Time Capsule Shares**: The default share name is `/Data`, which is
the main storage volume on Time Capsules. If your Time Capsule has a different
share name, you can change it with `-v` or the `TIMECAPSULE_VOLUME` environment
variable.

## Configuration

Set the following environment variables before running the script (optional,
defaults provided; can be overridden by command-line options):

- `TIMECAPSULE_IP`: IP address of the Time Capsule (default: 192.168.0.3)
- `TIMECAPSULE_VOLUME`: SMB share name on the Time Capsule (default: /Data -
  the main data share)
- `TIMECAPSULE_USER`: Your Time Capsule username (default: your_username)
- `TIMECAPSULE_PASS`: Your Time Capsule password (default: Superuser)
- `TIMECAPSULE_MOUNT_POINT`: Mount point path (default: /mnt/time)

Command-line options (override environment variables):

- `-i IP`: Set IP address
- `-v VOLUME`: Set SMB share name (e.g., /Data)
- `-u USERNAME`: Set username
- `-p PASSWORD`: Set password
- `-m MOUNT_POINT`: Set mount point

Examples:

```bash
# Using environment variables
export TIMECAPSULE_IP="192.168.1.100"
export TIMECAPSULE_USER="myuser"
export TIMECAPSULE_PASS="mypass"
export TIMECAPSULE_MOUNT_POINT="/Volumes/timecapsule"
./mountTimecapsule

# Using command-line options
./mountTimecapsule -i 192.168.1.100 -v /Data -u myuser -p mypass \
  -m /mnt/tc

# Mix of both (env vars with command overrides)
export TIMECAPSULE_USER="myuser"
./mountTimecapsule -i 192.168.1.100 -p mypass -m /Volumes/tc
```

## Changes

- Extended to support both Linux and macOS
- Updated to use SMB3 for better security and compatibility
- Credentials removed from script; now read from environment variables
