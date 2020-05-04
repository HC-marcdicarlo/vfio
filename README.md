# vfio
notes on my personal vfio setup and specs

## gettings started
## requirements
## specs

## notes
commands and notes

### find gpu device numbers

```shell
bash ./iommu.sh
```

```
...
IOMMU Group 14:
	08:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP106 [GeForce GTX 1060 6GB] [10de:1c03] (rev a1)
	08:00.1 Audio device [0403]: NVIDIA Corporation GP106 High Definition Audio Controller [10de:10f1] (rev a1)
IOMMU Group 15:
	09:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:1e81] (rev a1)
	09:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:10f8] (rev a1)
	09:00.2 USB controller [0c03]: NVIDIA Corporation Device [10de:1ad8] (rev a1)
	09:00.3 Serial bus controller [0c80]: NVIDIA Corporation Device [10de:1ad9] (rev a1)
...
```

our ids: 10de:1e81,10de:10f8,10de:1ad8,10de:1ad9



### make vfio-pci module aware of our gpu ids

This will let the vfio-pci module know which devices to overide te drivers for.

```shell
sudo vim /etc/initramfs-tools/modules
```
```
# List of modules that you want to include in your initramfs.
# They will be loaded at boot time in the order below.
#
# Syntax:  module_name [args ...]
#
# You must run update-initramfs(8) to effect this change.
#
# Examples:
#
# raid1
# sd_mod
vfio vfio_iommu_type1 vfio_virqfd vfio_pci ids=10de:1e81,10de:10f8,10de:1ad8,10de:1ad9
```

```shell
sudo vim /etc/modules
```

```
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
vfio vfio_iommu_type1 vfio_pci ids=10de:1e81,10de:10f8,10de:1ad8,10de:1ad9
```

```shell
sudo vim /etc/modprobe.d/nvidia.conf
```

```softdep nouveau pre: vfio-pci
softdep nvidia pre: vfio-pci
softdep snd_hda_intel pre: vfio-pci
softdep xhci_hcd pre: vfio-pci
softdep nvidia-gpu pre: vfio-pci
```

```shell
sudo vim /etc/modprobe.d/vfio.conf
```

```
options vfio-pci ids=10de:1e81,10de:10f8,10de:1ad8,10de:1ad9
```

```shell
sudo update-initramfs -u -k all
```

### verify gpu driver

```shell
lspci -nnk|grep "09:" -a3
```
```
	Subsystem: PNY GP106 High Definition Audio Controller [196e:119f]
	Kernel driver in use: snd_hda_intel
	Kernel modules: snd_hda_intel
09:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:1e81] (rev a1)
	Subsystem: ASUSTeK Computer Inc. Device [1043:8712]
	Kernel driver in use: nvidia
	Kernel modules: nvidiafb, nouveau, nvidia_drm, nvidia
09:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:10f8] (rev a1)
	Subsystem: ASUSTeK Computer Inc. Device [1043:8712]
	Kernel driver in use: snd_hda_intel
	Kernel modules: snd_hda_intel
09:00.2 USB controller [0c03]: NVIDIA Corporation Device [10de:1ad8] (rev a1)
	Subsystem: ASUSTeK Computer Inc. Device [1043:8712]
	Kernel driver in use: xhci_hcd
09:00.3 Serial bus controller [0c80]: NVIDIA Corporation Device [10de:1ad9] (rev a1)
	Subsystem: ASUSTeK Computer Inc. Device [1043:8712]
	Kernel driver in use: nvidia-gpu
	Kernel modules: i2c_nvidia_gpu

```