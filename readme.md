# Provisioning a Development VM with DigitalOcean and Cloud-Init
This guide walks you through setting up a development VM on DigitalOcean using doctl and a cloud-init configuration file (dev.yml).

# Prerequisites
## Install Cloud-Init
Ensure cloud-init is available on your base image (pre-installed on Ubuntu 22.04).
Install `doctl`

Install the DigitalOcean CLI:

On Linux/macOS:

```bash

curl -sL https://github.com/digitalocean/doctl/releases/download/v1.104.0/doctl-1.104.0-linux-amd64.tar.gz | tar -xzv
sudo mv doctl /usr/local/bin/
```

Verify: `doctl version`

## Authenticate doctl

Authenticate with your DigitalOcean account:

```bash

doctl auth init
```
Follow the prompts to enter your API token.

## Provision the VM

Create a droplet named devmachine with the following command:

```bash

doctl compute droplet create devmachine \
  --region nyc3 \
  --size s-1vcpu-1gb \
  --image ubuntu-24-04-x64 \
  --user-data-file dev.yml \
  --wait
```
- `--wait` ensures the command completes once the droplet is active.
- Increase the CPU & RAM Size ref ([Digital Ocean Doc](https://docs.digitalocean.com/reference/doctl/reference/compute/droplet/create/))
- You can also change the `region`


# List the VM

Check your droplet:

```bash

doctl compute droplet list
```

> Note the IP address of devmachine.

## Connect to the VM via SSH

Log in as the developer user:

```bash

ssh developer@{IP}
```
- Password: maldev@25

## Delete the VM

When done, delete the droplet:

```bash

doctl compute droplet delete --force devmachine
```
- --force skips confirmation.

# dev.yml Configuration

> Find it withing the `dir`

## Verify Package Installation

Manual Checks

After SSHing into the VM, verify key packages:

```bash

dpkg -l | grep gcc
dpkg -l | grep g++
dpkg -l | grep cmake
dpkg -l | grep postgresql
```
> Look for ii in the output (e.g., ii  gcc  4:11.2.0-1ubuntu1).

## Automated Check Script
Run this script to verify all packages:

```bash

#!/bin/bash
for pkg in gcc g++ gdb make cmake libboost-all-dev libssh-dev libpqxx-dev postgresql-server-dev-all libcryptopp-dev; do
  if dpkg -l | grep -q "^ii  $pkg "; then
    echo "$pkg is installed: $(dpkg -l $pkg | grep $pkg | awk '{print $3}')"
  else
    echo "$pkg is NOT installed"
  fi
done
```

1. Save as check_packages.sh.

2. Make executable: chmod +x check_packages.sh.

3. Run: ./check_packages.sh.

Expected Output Example:

```bash
gcc is installed: 4:11.2.0-1ubuntu1
g++ is installed: 4:11.2.0-1ubuntu1
gdb is installed: 12.1-0ubuntu1~22.04.2
make is installed: 4.3-4.1build1
cmake is installed: 3.22.1-1ubuntu1.22.04.2
libboost-all-dev is installed: 1.74.0.3ubuntu7
libssh-dev is installed: 0.9.6-2ubuntu0.22.04.3
libpqxx-dev is installed: 6.4.5-2build1
postgresql-server-dev-all is installed: 14+238
libcryptopp-dev is installed: 8.6.0-1
```

# Check Logs
Inspect the cloud-init process:

```bash=
sudo cat /var/log/cloud-init-debug.log
sudo cat /var/log/cloud-init-output.log
```
- Look for "Installed packages:" in cloud-init-debug.log to confirm the full list.

# Troubleshooting

- Packages Missing: If any package isn’t installed, check` /var/log/cloud-init-output.log` for errors (e.g., apt-get failures).

- Network Issues: The bootcmd ensures network readiness; verify with log entries like "Waiting for network...".

- Manual Fix: Install missing packages:

```bash

sudo apt-get update
sudo apt-get install -y <missing-package>
```

--- 

For any other quries, reach out to our discord server. 

- PR are welcome 
- Suggestions & improvements are welcome 
- Keep Learning.

> Love - Arishti Security Elearning Team.