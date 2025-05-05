# Ave Rootkit for Linux
## Guidance
## General Concept
Ave is a loadable kernel module for Linux (5.18+) capable of hiding processes, files and attached sockets at the kernel level, plus spoofing system calls and encrypting traffic. It works on x86-64 and ARM64. In short, it's a universal tool for stealthily hiding any object on your system, with additional SELinux/AppArmor bypass, antidebugging and a whole bunch of other features.

### Installation:
### 1. Pre-requisites
To build Ave, you first need to install the packages:

sudo apt install cmake gcc gcc g++ llvm-dev llvm-tools python3-pip qemu-system-x86 socat netcat libssl-dev
pip3 install lit
sudo ln -s ~/.local/bin/lit /usr/bin/llvm-lit

### 2. Build

#### Local build
1. Prepare a folder for builds:
   mkdir build && cd build
   cmake ../ -DCMAKE_C_COMPILER=gcc -DPROCNAME="interface_name”

2. Run the build:
   make


#### Cross-compile
If you need to build for another kernel or another machine:
cmake ../ -DKERNEL_DIR=/path to kernel headers -DKOVID_LINUX_VERSION=5.10 -DCMAKE_C_COMPILER=gcc
make

### Basic Functionality:

### 1. Module cloaking
Ave removes itself from lsmod, /sys and other places where the module is normally visible. In DEPLOY mode this all works out of the box.

### 2. Hiding files and directories
The filldir and filldir64 hooks are used:
- Anything that is hidden is out of output from ls, ps, top, etc.
- No traces in normal utilities.

### 3. System calls
A mix of Ftrace hooks and direct editing of the system call table:
- Dynamically updates hooks to match kernel changes.
- Substitutes e.g. read, kill, clone, etc.

### 4. Working with the network stack
Ave monitors connections via netfilter and generates hidden signatures. Everything is encrypted with AES.

### 5. Bypass kernel protections
SELinux, AppArmor and relro, pie, nx protections are bypassed by Ave by adjusting to the specific system config.

### 6. Antidebug
- Filters and blocks ptrace, strace and other debuggers.
- Data in memory is encrypted to resist unloading and analysis.
- If debugging is attempted, the module starts spoofing.

### 7. Persistence
- Patches ELF files for autoloading.
- Sticks in system initialization mechanisms to start at startup.

### 8. Backdoors
Choice:
- Netcat: applying port-knocking to remotely open hidden access.
- OpenSSL: to have an encrypted channel at once.
- Socat: universal, for any type of sockets.

### 9. Logs
- Removes traces from dmesg.
- Masks TCP/UDP connections.
- Hides itself from /rgos.

### Usage:
### Enabling the /proc interface
kill -SIGCONT 31337

Then /proc/interface_name starts listening to your commands.

### Getting root
kill -SIGCONT 666
su
(Automatic escalation on the local.)

### Hide process
echo <PID> > /proc/interface_name


### Hide files and folders
- In the current directory:
  echo hide-file=README.txt > /proc/interface_name

- In the absolute path:
  echo hide-directory=/home/user/docs > /proc/interface_name


### Backdoors
To check what backdoors there are and what is available:
utils/bd_connect.sh

### Technical background: 
### Architecture.
- x86-64 and ARM64 are the main focus.
- Kernels: 5.18+.
- Automatic adaptation to kernel settings so hooks don't break after updates.

### Security
- Completely removes hidden processes and connections from monitoring.
- The module blocks unloading and tries to make a backup if you try to pull it.

### Automation
- Scripts for quick build and deployment of rootkit.
- Key generation for encrypted channels.
- Modular infecting machinery for ELF files autoconnect.

## Testing: 
Tested on:
- Debian 12.8 (Bookworm): kernel 6.1.0-10-10-amd64 (LTS).
- Ubuntu 22.04.3 LTS: kernel 5.19.0-32-generic (HWE).
- Ubuntu 23.10: kernel 6.5.0-25-generic.

### Running tests
1. local tests:
   make check-ave -j1

2. Cross tests:
   cmake ../ -DKERNEL_DIR=/path to kernel headers -DCROSS_TESTS=ON
   make check-ave
