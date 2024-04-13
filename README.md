# FirmwareDecompilingNotes

First case study: squash file system.
The ones who knows me knows that sometime as a hobby i like to "study cybersecurity on real life cases"... 😜

i got a device with a firmware that didnt allow me to do what i needed, so this is my starting point: .bin containing the firmware (firmware_v1.0.bin)

The first thing we need is to find out some info about the file. To achieve that you can use binwalk

\#binwalk firmware_v1.0.bin

the output will be something like this

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
1XXXX         0x47XXX         Unix path: XXXXXXXXXXXXXXX
XXXXX         0x1XXXX        Unix path: XXXXX
XXXXX         0xXXXXX         CRC32 polynomial table, little endian
XXXXX         0x1XXXXX        xz compressed data
XXXXX         0x3XXXXX        uImage header, header size: 64 bytes, header CRC: XXXXX, created: XXXXX, image size: 98xxx bytes, Data Address: 0x0, Entry Point: 0x0, data CRC: XXXXX, OS: Firmware, CPU: ARM, image type: OS Kernel Image, compression type: lzma, image name: "XXXXXXVM"
XXXXX        0xXXXXX        xz compressed data
XXXXX        0xXXXXX        uImage header, header size: 64 bytes, header CRC: 0xFA5xxxx, created: XXXXX, image size: 161xxxx bytes, Data Address: 0xXXXXX, Entry Point: 0xXXXXX, data CRC: 0xXXXXX, OS: Linux, CPU: ARM, image type: OS Kernel Image, compression type: lzma, image name: "XXXXX"
XXXXX        XXXXX         xz compressed data
YYYYYY       0xZZZZZ      Squashfs filesystem, little endian, version 4.0, compression:xz, size: jjjjjj bytes, 5xx inodes, blocksize: hhhhhh bytes, created: XXXXX
XXXXX       0xXXXXX        Squashfs filesystem, little endian, version 4.0, compression:xz, size: jjjjjjjjj bytes, 4xx inodes, blocksize: hhhhhh bytes, created: XXXXX

i knew from some research that i would have found something like a Squash file system. So the thing that interested me is whatever is inside the file at the YYYYYY decimal address.

to extract the file system we can use dd.

\#dd if=firmware_v1.0.bin of=rootfs.squashfs bs=1 skip=YYYYYY

to depack the extracted file we can use unsquashfs.
\#unsquashfs -d rootfs rootfs.squashfs

a #ls inside rootfs write: bin  bootconfig  config  customer  dev  etc  home  lib  linuxrc  mnt  proc  sbin  sd  sys  tmp  usr  var

clearly a standard unix/linux file structure...

here i have made the modifications i needed, i will not go into much details. Lets jump to how you can repack the edited structure.
without too much explaining:

repacking the squashfs:
\#mksquashfs rootfs rootfs_new.squashfs -comp xz

injecting the edited squashfs into the original firmware
\#dd if=rootfs_new.squashfs of=firmware_v1.0.bin bs=1 seek=YYYYYY conv=notrunc

and to finish the work it was needed to edit a little bit the hex code of the bin to make it look like an update of the previous firmware, i will not go into much details of this part right now, maybe i will update this in the future with more tutorials and more detailed informations :)
