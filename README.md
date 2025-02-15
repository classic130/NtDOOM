# NtDOOM

Doom running in the NT kernel. [Only tested on NT 10.0.22621.525 as that is what I had on hand]

https://user-images.githubusercontent.com/51860844/235435027-c4137000-5efd-435f-a25f-5f90943ee9be.mp4



# How to run it?

1. Enable test signing and reboot

```
bcdedit /set testsigning on
shutdown /r /t 0
```

2. Create a service using SC

```
sc create NtDOOM binPath=C:\where\ever\the\driver\is\NtDOOM.sys type=kernel start=demand
```

3. Place a copy of DOOM on the root of the C: drive with the name "DOOM.WAD" 

4. Run it!

```
sc start NtDOOM
```

# Possible problems

## If the driver fails to load
Try restarting `explorer.exe`. If that did not work then restart your system. Do not try to rerun the driver if restarting `explorer.exe` did not work, There is a good chance the system will BSOD.

# How does this work?

Many parts of the Win32 API are usually handled in the kernel side. These include GDI, getting inputs, etc. Many of these functions have syscalls for them. These syscalls are handled by `win32k.sys` [`win32kbase.sys` and `win32kfull.sys`]. 

Here the kernel driver just calls the handler for these syscalls present in win32k. 

However one can't simply get the addresses to these functions and call them directly. 

If you have experience with [how syscalls in Windows work](https://alice.climent-pommeret.red/posts/a-syscall-journey-in-the-windows-kernel/) then you know that not every type of thread can call these GUI syscalls directly. The thread must be running under a Win32 GUI application. 

To bypass that we spoof the kernel thread as if it is a thread from `explorer.exe`. Due to this spoofing process the driver can only run on certain NT kernel versions as the internal data structures for `KTHREAD` and `EPROCESS` usually change.

The doom port is based off of [PureDoom](https://github.com/Daivuk/PureDOOM) with slight changes.

# Credits

[ReactOS Project](doxygen.reactos.org/) for providing documentations for these raw syscalls.

[Kernel Drawing](https://github.com/Sentient111/KernelDrawing) for figuring out how to use win32k functions in a kernel driver

[Pure Doom](https://github.com/Daivuk/PureDOOM) for providing the base for this doom port
