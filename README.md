# DEB12-MDB7-UNF

Install script and instructions for installing MongoDB 7.0 and the UniFi Network Application on Debian 12 (Bookworm).

## Description

This repository contains the installation commands used to install MongoDB 7.0 (mongodb-org) and the UniFi Network Controller on Debian 12. The commands are intended to be run with root privileges (either as root or using sudo).

## Requirements

- Debian 12 (bookworm)
- Root privileges (sudo)
- Internet access to download packages and keys

## Important notes / Warnings

- These steps add external package repositories (MongoDB and Ubiquiti). Only add repositories you trust.
- The examples assume you run commands as root. Prefix commands with `sudo` if you are not root.
- The original install commands contained a typo (`systemclt`). The correct command is `systemctl`.

## Installation steps

1. Update package lists and install small prerequisites

```
sudo apt-get update
sudo apt-get install -y curl wget apt-transport-https ca-certificates gnupg
```

2. Install MongoDB 7.0 repository and package

```
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg

echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/7.0 main"  
  | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list >/dev/null

sudo apt-get update
sudo apt-get install -y mongodb-org

sudo systemctl start mongod
sudo systemctl enable mongod
```

3. Install UniFi Network Controller

```
wget -qO - https://dl.ui.com/unifi/unifi-repo.gpg | sudo tee /usr/share/keyrings/unifi-repo.gpg >/dev/null

echo "deb [signed-by=/usr/share/keyrings/unifi-repo.gpg] https://www.ui.com/downloads/unifi/debian stable ubiquiti"  
  | sudo tee /etc/apt/sources.list.d/unifi.list >/dev/null

sudo apt-get update
sudo apt-get install -y unifi

sudo systemctl start unifi
sudo systemctl enable unifi
```

## Verify services

Check the service statuses:

```
sudo systemctl status mongod
sudo systemctl status unifi
```

Or:

```
service mongod status
service unifi status
```

## Troubleshooting

- If apt cannot find packages, ensure the repository files exist in `/etc/apt/sources.list.d/` and re-run `sudo apt-get update`.
- If a GPG key import fails, ensure curl/wget and gnupg are installed and reachable from the server.
- Check journal logs for service errors:
  `sudo journalctl -u mongod -b` or `sudo journalctl -u unifi -b`

## Uninstall (quick)

Remove packages and repository entries (example — test before running in production):

```
sudo apt-get remove --purge -y mongodb-org unifi
sudo rm /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo rm /etc/apt/sources.list.d/unifi.list
sudo rm /usr/share/keyrings/mongodb-server-7.0.gpg
sudo rm /usr/share/keyrings/unifi-repo.gpg
sudo apt-get update
```

## Example: create an executable installer script

Save the sanitized commands into `install.sh`, make it executable, and run:

```
chmod +x install.sh
sudo ./install.sh
```
