Disclamer: This has only been tested on Arch Linux Kernel: 5.10.12-zen1-1-zen so results may vary. Build instructions will be arch related as a result
===========


Introductions
===========
This Fork combines Qemu with Audio fixes from https://github.com/spheenik/qemu 
which helps improve audio performance over pulseaudio.
What is **QEMU**?

QEMU is a generic and open source machine & userspace emulator and
virtualizer.

QEMU is capable of emulating a complete machine in software without any
need for hardware virtualization support. By using dynamic translation,
it achieves very good performance. QEMU can also integrate with the Xen
and KVM hypervisors to provide emulated hardware while allowing the
hypervisor to manage the CPU. With hypervisor support, QEMU can achieve
near native performance for CPUs. When QEMU emulates CPUs directly it is
capable of running operating systems made for one machine (e.g. an ARMv7
board) on a different machine (e.g. an x86_64 PC board).

QEMU is also capable of providing userspace API virtualization for Linux
and BSD kernel interfaces. This allows binaries compiled against one
architecture ABI (e.g. the Linux PPC64 ABI) to be run on a host using a
different architecture ABI (e.g. the Linux x86_64 ABI). This does not
involve any hardware emulation, simply CPU and syscall emulation.

QEMU aims to fit into a variety of use cases. It can be invoked directly
by users wishing to have full control over its behaviour and settings.
It also aims to facilitate integration into higher level management
layers, by providing a stable command line interface and monitor API.
It is commonly invoked indirectly via the libvirt library when using
open source applications such as oVirt, OpenStack and virt-manager.

QEMU as a whole is released under the GNU General Public License,
version 2. For full licensing details, consult the LICENSE file.


***Git clone & Building***
========
build instructions are as followed 
first it requires python 3.5 and above

Arch: ```sudo pacman -Syu spice-protocol python ceph libiscsi glusterfs ninja```


 ``` 
git clone -b spheenik-qemu https://github.com/OzzyHelix/qemu.git
cd qemu
mkdir build && cd build
../configure --prefix=/opt/qemu-test --python=/usr/bin/python  --target-list=x86_64-softmmu --audio-drv-list=pa --disable-werror 
# if you have a custom location for your python3 binary do --python=/path/to/python/binary
make 
# I would use the -j flag to make the binary compiling go faster example: make -j12
sudo make install
  ```

Additional information can also be found online via the QEMU website:

* `<https://qemu.org/Hosts/Linux>`_
* `<https://qemu.org/Hosts/Mac>`_
* `<https://qemu.org/Hosts/W32>`_


# To use the patched QEMU Binary  
you will have to replace the line ```<emulator>/usr/bin/qemu-system-x86_64</emulator>```

with this line ```<emulator>/opt/qemu-test/bin/qemu-system-x86_64</emulator>```


***Using audio***
=============
these settings will allow for audio from the QEMU VM to play over pulseaudio

the end of your XML configuration file.

should look something like this

```<qemu:commandline>
    <qemu:env name='QEMU_PA_BUFFER_SIZE_OUT' value='2.5'/>
    <qemu:env name='QEMU_PA_BUFFER_SIZE_IN' value='2.5'/>
    <qemu:env name='QEMU_PA_TLENGTH' value='2.5'/>
    <qemu:env name='QEMU_PA_FRAGSIZE' value='1.0'/>
    <qemu:env name='QEMU_PA_MAXLENGTH_IN' value='2'/>
    <qemu:env name='QEMU_PA_ADJUST_LATENCY_OUT' value='0'/>
    <qemu:env name='QEMU_PA_ADJUST_LATENCY_IN' value='1'/>
    <qemu:env name='QEMU_AUDIO_DRV' value='pa'/>
    <qemu:env name='QEMU_PA_SERVER' value='/run/user/1000/pulse/native'/>
  </qemu:commandline>
</domain>
```

```
QEMU_PA_BUFFER_SIZE_OUT: integer, default = 2.5 * timer interval
  internal buffer size in frames for playback device

QEMU_PA_BUFFER_SIZE_IN: integer, default = 2.5 * timer interval
  internal buffer size in frames for recording device

QEMU_PA_TLENGTH: integer, default = 2.5 * timer interval
  playback buffer target length in frames

QEMU_PA_FRAGSIZE: integer, default = 1.0 * timer interval
 fragment length of recording device in frames

QEMU_PA_MAXLENGTH_IN: integer, default = 2 * fragsize
  maximum length of PA recording buffer in frames

QEMU_PA_ADJUST_LATENCY_OUT: boolean, default = 0
  let PA adjust latency for playback device

QEMU_PA_ADJUST_LATENCY_IN: boolean, default = 1
  let PA adjust latency for recording device

```
