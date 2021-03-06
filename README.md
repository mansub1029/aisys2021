# Aisys Term project - K2S2

# Deep Learning I/O Performance: Checkpoint/Load

---

This repository is for AISYS 2021 - SNU.

In order to compare I/O difference between SATA/NVMe/PMEM, you have to either install PMEM emulator, or a physical PMEM.

## Persistent Memory Environment Settings

- Install NDCTL & PMDK for essential libraries

[NDCTL Introduction](https://docs.pmem.io/persistent-memory/getting-started-guide/what-is-ndctl)

[pmem.io: PMDK](https://pmem.io/pmdk/)

## Intel Persistent Memory Emulator

[How to Emulate Persistent Memory Using Dynamic Random-access Memory...](https://software.intel.com/content/www/us/en/develop/articles/how-to-emulate-persistent-memory-on-an-intel-architecture-server.html?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+ISNMain+%28Intel+Developer+Zone+Articles+Feed%29)

- **Linux* Kernel Configuration (4.2 or Above)**

```bash
# egrep ‘(DAX|PMEM)’ /boot/config-`uname –r`

CONFIG_X86_PMEM_LEGACY_DEVICE=y
CONFIG_X86_PMEM_LEGACY=y
CONFIG_BLK_DEV_RAM_DAX=y
CONFIG_BLK_DEV_PMEM=m
CONFIG_FS_DAX=y
CONFIG_FS_DAX_PMD=y
CONFIG_ARCH_HAS_PMEM_API=y
```

- **Enable DAX and PMEM in the Kernel**

```bash
$ make nconfig

        -> Device Drivers -> NVDIMM Support ->

                    <M>PMEM; <M>BLK; <*>BTT
$ make nconfig

        -> Processor type and features

                      <*>Support non-standard NVDIMMs and ADR protected memory

```

- **Build and Install the Kernel**

```bash
# make -jX     # where X is the number of cores on the machine
# make modules_install install
```

- **GRUB Configuration**

```bash
# memmap=nn[KMG]!ss[KMG]
# vi /etc/default/grub
GRUB_CMDLINE_LINUX="memmap=nn[KMG]!ss[KMG]"
```

- **Build a DAX-enabled File System**

```bash
# mkdir /mnt/pmemdir
# mkfs.ext4 /dev/pmem3
# mount -o dax /dev/pmem3 /mnt/pmemdir
```

## IPMCTL - for Intel Optane DC PMEM

[IPMCTL User Guide](https://docs.pmem.io/ipmctl-user-guide/)

## Recommend to use python virtual environment

### VENV setting - for PyTorch

[venv - Creation of virtual environments - Python 3.9.5 documentation](https://docs.python.org/ko/3/library/venv.html)

### R**equirements**

```bash
python3 -m venv venv
source venv/bin/activate
git clone www.github.com/mansub1029/aisys2021
pip install torch torchvision
cp serialization.py venv/lib/python3.*/site-packages/torch/
cp -r nvm venv/lib/python3.*/site-packages/
cd aisys2021

python main.py -r -p [checkpoint_path] -m [model_name] -ld false
```

## Environment Path Settings

본 코드는 3개의 device type(sata-ssd, nvem, nvm)을 이용하고 있습니다. 기본적인 shell script code에선 각 device를 mount하여 device가 사용되도록 만들어 주어야 합니다. 

Path를 지정해 주기 위해 main.py를 돌릴 때 argument로 -p checkpoint_path 를 입력해 주어야 합니다. path 뒤에는 checkpoint 디렉토리가 반드시 생성되어 있어야 하고, checkpoint 디렉토리 안에 checkpoint file이 생성됩니다. 

For example,

sata-ssd LOCATION, we've given the path as following: **/mnt/sda/checkpoint**

nvme ssd LOCATION, **./checkpoint**  (home directory is on NVMe)

pmem LOCATION, **/mnt/pmem0**. (for PMEM emulator)

### Main Code

pytorch를 사용하기 위해서 source venv/bin/activate를 먼저 수행해야 main.py코드를 에러없이 수행할 수 있습니다.

```bash
usage: test.py [-h] [--lr LR] [-m M] [-p P] [--resume] [-ld LD]

PyTorch CIFAR10 Training

optional arguments:
  -h, --help    show this help message and exit
  --lr LR       learning rate
  -m M          model name
  -p P          file path
  --resume, -r  resume from checkpoint
  -ld LD        true: load checkpoint, false: save checkpoint file
```

[save.sh] 혹은 [load.sh] 사용하시면 바로 수행해보실 수 있습니다.

[save.sh] - sata-ssd, nvme, nvm device를 각 머신러닝 모델(VGG, SimpleDLA, DPN92)들을 트레이닝하여 checkpoint 파일을 생성

[load.sh] - 각 device별 마운트된 checkpoint 파일을 이용해서 이전에 save된 checkpoint state를 load

(load.sh를 사용 전엔 checkpoint 파일을 만들어 주셔야합니다.)
