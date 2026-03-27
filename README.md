# CredAgent Installation Guide

## Overview
CredAgent is a Zero-Trust FUSE Credential Shield that protects legacy CLI tools from credential theft by compromised AI Agents. It consists of three main components:
- **enclave**: The daemon that manages credential protection policies
- **monitor**: The FUSE filesystem that intercepts credential access
- **protector**: The CLI tool for protecting credentials

## Prerequisites
Before installation, you need to install FUSE (Filesystem in Userspace) on your system:

### Ubuntu/Debian:
```bash
sudo apt update
sudo apt install fuse libfuse-dev
# On some systems, you may need to add your user to the fuse group (may require logout/login to take effect)
# Check if this is required: if you get permission errors when mounting FUSE filesystems, run:
# sudo usermod -a -G fuse $USER
```

### CentOS/RHEL/Fedora:
```bash
sudo yum install fuse fuse-devel
# Or for newer versions:
sudo dnf install fuse fuse-devel
```

## Installation
1. Ensure you have installed FUSE as described above
2. Run the installation script:
```bash
sudo ./install.sh
```

## Configuration
After installation, you can configure your LLM API settings in the configuration files:
[notice] If you only use BASIC strategy, you can skip this step.
- `/usr/local/etc/CredAgent/engine/config.json` - For the enclave daemon
- `/usr/local/etc/CredAgent/protector/config.json` - For the protector CLI

Update the following fields in both files:
- `api_url`: Your LLM API endpoint
- `api_key`: Your API key
- `model_name`: The model name to use

## Usage

### 1. Initialize the Enclave
First, create a master key for the enclave:
```bash
sudo enclave init
```

### 2. Start the Enclave Daemon
Start the enclave daemon to manage credential protection:
```bash
sudo enclave run &
```

### 3. Mount the Protected Filesystem
Fuse system will be mounted at the location in config file where the monitor.mount_point specify:
```bash
sudo monitor &
```

### 4. Protect Credentials
Use the protector CLI to protect your credential files:
```bash
# Protect a credential file
protector protect --strategy BASIC --allowed-binary /usr/local/bin/aliyun --measure-binary ~/.aliyun/config.json --max-open-count 1

# List protected files
protector list

# Release protection on a file
protector release ~/.aliyun/config.json
```

## Components Details

### Enclave
The enclave daemon manages the credential protection policies and acts as the central authority for access control decisions. It uses an LLM to make intelligent decisions about credential access requests.

#### Usage:
```bash
# Initialize the enclave (creates master key)
sudo enclave init [options]

# Run the enclave daemon
sudo enclave run [options]

# Options:
# -c, --config string          Path to the configuration file (default "/usr/local/etc/CredAgent/engine/config.json")
# -k, --key-file string        Path to the master key file (default "/usr/local/etc/CredAgent/master.key")
# -s, --socket-file string     Path to the Unix Domain Socket file (default "/run/cred_protector/guard.sock")
# -b, --baseline-file string   Path to the encrypted baseline store file
# -l, --log-level string       Log level (debug, info, warning, error) (default "info")
```

### Monitor
The monitor implements a FUSE filesystem that intercepts access to protected credentials and enforces access policies.

#### Usage:
```bash
# Mount the shielded filesystem
sudo monitor --mount-point /path/to/mount [options]

# Options:
# -c, --config string          Path to the configuration file (default "/usr/local/etc/CredAgent/monitor/config.json")
# -m, --mount-point string     Mount point for the FUSE filesystem (default "/mnt/shielded")
# -s, --socket-file string     Path to the Unix Domain Socket file (default "/run/cred_protector/guard.sock")
# -l, --log-level string       Log level (debug, info, warning, error) (default "info")
```

### Protector
The protector is a CLI tool for managing protected credentials, including adding/removing protection and viewing the protection status.

#### Usage:
```bash
# Protect a file
protector protect [filepath] [options]

# List protected files
protector list [options]

# Release protection on a file
protector release [filepath] [options]

# Options:
# -c, --config string          Path to the configuration file (default "/usr/local/etc/CredAgent/protector/config.json")
# -s, --socket-file string     Path to the Unix Domain Socket file (default "/run/cred_protector/guard.sock")
# -l, --log-level string       Log level (debug, info, warning, error) (default "info")
```

## Troubleshooting
- Make sure FUSE is properly installed and your user is in the fuse group
- Check that the enclave daemon is running before attempting to mount the filesystem
- Ensure API keys and endpoints are properly configured in the config files
- Check log files for error messages if components fail to start
