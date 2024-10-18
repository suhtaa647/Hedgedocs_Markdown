---
title: windows
tags: 14, 4days, 5days
---

# Windows

## virtualisation on linux
### INFO
[Creating Virtual Machines with Libvirt: Linux and Windows](https://manuelcortez.net/2023/08/creating-vm-libvirt-linux-windows/)

#### iso
[Download Windows 10 Disc Image (ISO File)](https://www.microsoft.com/en-us/software-download/windows10ISO) 

#### drivers from fedora
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso

## enable libvirt
[KVM - Debian Wiki](https://wiki.debian.org/KVM#Installation)

```shell
sudo apt install qemu-system libvirt-daemon-system
sudo apt install virt-manager
sudo apt install qemu-utils

sudo apt install ovmf swtpm swtpm-tools ### ??

```
### network
```shell
virsh --connect=qemu:///system net-start default
virsh --connect=qemu:///system net-autostart default
```

### add user
```shell
sudo usermod -aG libvirt $USER
newgrp libvirt
```


### create vm
```shell
sudo virt-install                                                             \
  --name windows-vm                                                           \
  --cdrom /home/qskills/MEDIA/SHARE/Win10_22H2_EnglishInternational_x64v1.iso \
  --os-variant=win10                                                          \
  --network network=default                                                   \
  --disk size=50,cache=none,bus=virtio                                        \
  --disk path=/home/qskills/MEDIA/SHARE/virtio-win-0.1.208.iso,device=cdrom   \
  --memory 4096                                                               \
  --sound default                                                             \
  --graphics spice,listen=::,password=hardpasswordforwindows                  \
  --vcpu 4                                                                    \
  --video qxl                                                                 \
  --noautoconsole
```


## usage

### INFO
[Using Ansible and Windows — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/os_guide/windows_usage.html)

#### module development
[Windows module development walkthrough — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general_windows.html#developing-modules-general-windows)
#### collection 
[Ansible.Windows — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/windows/index.html)

#### important modules

* [chocolatey.chocolatey.win_chocolatey module – Manage packages using chocolatey — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/chocolatey/chocolatey/win_chocolatey_module.html)

* [ansible.windows.win_package module – Installs/uninstalls an installable package — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_package_module.html#ansible-collections-ansible-windows-win-package-module)

* [ansible.windows.win_powershell module – Run PowerShell scripts — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_powershell_module.html#ansible-collections-ansible-windows-win-powershell-module)

* [ansible.windows.win_command module – Executes a command on a remote Windows node — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_command_module.html#ansible-collections-ansible-windows-win-command-module)

* [ansible.windows.win_shell module – Execute shell commands on target hosts — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_shell_module.html#ansible-collections-ansible-windows-win-shell-module)

* [ansible.windows.win_service module – Manage and query Windows services — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html#ansible-collections-ansible-windows-win-service-module)

#### important filter
* [ansible.windows.quote filter – Quotes argument(s) for various Windows shells — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/windows/quote_filter.html#ansible-collections-ansible-windows-quote-filter)

#### used components on windows
* WinRM [Windows Remote Management - Wikipedia](https://en.wikipedia.org/wiki/Windows_Remote_Management)
* [Windows-Remoteverwaltung - Win32 apps | Microsoft Learn](https://learn.microsoft.com/de-de/windows/win32/winrm/portal)
* MSI [Windows Installer - Wikipedia](https://en.wikipedia.org/wiki/Windows_Installer)
* [Use DISM in Windows PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/use-dism-in-windows-powershell-s14?view=windows-11)
* [Chocolatey Software | Chocolatey - The package manager for Windows](https://chocolatey.org/)