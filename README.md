# How to Build

## Install RHEL 10 on WSL2
- [Create a new Red Hat Developer Account](https://developers.redhat.com/register)
- [Get latest RHEL 10 WSL2 Image](https://access.redhat.com/downloads/content/479/ver=/rhel---10/10.0/x86_64/product-software)
- Install WSL2
  ```Powershell
  PS> wsl --install
  ```
- Create WSL2 Directory
  ```Powershell
  PS> New-Item -ItemType Directory -Path "C:\WSL\RHEL10"
  ```
- Import the downloaded image
  ```Powershell
  PS> wsl --import "RHEL10" "C:\WSL\RHEL10" "$env:USERPROFILE\Downloads\rhel-10.0-x86_64-wsl2.tar.gz" --version 2
  The operation completed successfully.
  ```
- Verify installation
  ```Powershell
    PS> wsl --list --verbose
    NAME      STATE           VERSION
  * Ubuntu    Stopped         2
    RHEL10    Stopped         2
  ```
- Change default to RHEL 10
  ```Powershell
  PS> wsl --set-default RHEL10
  PS> wsl --list --verbose
    NAME      STATE           VERSION
  * RHEL10    Stopped         2
    Ubuntu    Stopped         2
  ```

## Configure RHEL 10
- Start RHEL 10
  ```Powershell
  PS> wsl
  [root@Windows user]#
  ```
- Register RHEL Installation
  ```shell
  [root@Windows user]# subscription-manager register
  Registering to: subscription.rhsm.redhat.com:443/subscription
  Username:
  Password:
  The system has been registered with ID: (UUID)
  The registered system name is: Windows
  ```
- Upgrade System and Install Basic Apps
  ```shell
  [root@Windows user]# dnf update -y
  [root@Windows user]# dnf install -y git wget curl ncurses nano dnf-plugins-core
  ```
- Create non-root user account
  ```shell
  [root@Windows user]# useradd -m -G wheel ordinaryuser
  [root@Windows user]# passwd ordinaryuser
  New password:
  Retype new password:
  passwd: password updated successfully
  ```
