# LIFS
Linux InitRAMFS from Scratch

Linux Initramfs From Scratch (LIFS) is a project that provides step-by-step instructions for building a customized Linux initramfs image entirely from source.

<i>Originally developed for [CMLFS](https://github.com/dslm4515/CMLFS)  and [MLFS](https://github.com/dslm4515/Musl-LFS) that use my [S6+S6-rc bootscripts](https://github.com/dslm4515/MLFS-S6-Bootscripts).</i>

## Goals
<ul>
 <li> [ ] Apply CPU microcode at early boot</li>
 <li> [ ] Provide firmware for kernel - For example firmware blobs for AMD GPU's</li>
 <li> [ ] Provide a 'rescue shell'</li>
 <li> [ ] Stay simple: Utilize Busybox</li>
 <li> [ ] Switch from initramfs to real system root and boot the rest of the system </li>
 <li> [ ] Distro neutral</li>
</ul>


