
65;7006;1c
# VM image management...

* mount partition from raw img file
 * https://web.archive.org/web/20170201061418/http://madduck.net/blog/2006.10.20:loop-mounting-partitions-from-a-disk-image/
 * ~~http://madduck.net/blog/2006.10.20:loop-mounting-partitions-from-a-disk-image/~~

* raw to qcow
 * ```qemu-img convert -c -f raw -O qcow2 dev_sda.raw dev_sda.qcow2```
   * sometimes the older qcow2 format *`compat=0.10`* might be needed as opposed to *`compat=1.1`*
     * ```qemu-img convert -f raw -O qcow2 -o compat=0.10 disk.raw disk.qcow2```
 * see also virt-sparsify


* Resize filesystems with NBD
  * https://edafe.de/2025/02/shrink-optimise-and-expand-an-existing-qcow2-image/

### shrinking a filesystem with guestfish and virt-resize

```
user1@dell4:~/tmp$ ls -alh debian9-9-z4.qcow2 
-rw-r--r-- 1 user1 user1 33G Apr 16 23:39 debian9-9-z4.qcow2
user1@dell4:~/tmp$ guestfish -a debian9-9-z4.qcow2

Welcome to guestfish, the guest filesystem shell for
editing virtual machine filesystems and disk images.

Type: ‘help’ for help on commands
      ‘man’ to read the manual
      ‘quit’ to quit the shell

><fs> run
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ --:--
><fs> 
><fs> list-filesystems 
/dev/sda1: ext4
/dev/sda5: swap
><fs> list-partitions 
/dev/sda1
/dev/sda2
/dev/sda5
><fs> resize2fs
resize2fs       resize2fs-M     resize2fs-size  
><fs> resize2fs-size /dev/sda1 13G
```

This may take a while depending on the size of the filesystem.
You may need to fsck the filesystem before you can resize it... use `e2fsck`

`e2fsck device [correct:true|false] [forceall:true|false]`

```
><fs>
><fs> exit

user1@dell4:~/tmp$ qemu-img create -f qcow2 -o preallocation=metadata debian9-9-z4_resized.qcow2 14G
Formatting 'debian9-9-z4_resized.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off preallocation=metadata compression_type=zlib size=15032385536 lazy_refcounts=off refcount_bits=16
user1@dell4:~/tmp$ virt-resize --shrink /dev/sda1 debian9-9-z4.qcow2 debian9-9-z4_resized.qcow2
[   0.0] Examining debian9-9-z4.qcow2
**********

Summary of changes:

virt-resize: /dev/sda1: This partition will be resized from 39.0G to 13.0G. 
 The filesystem ext4 on /dev/sda1 will be expanded using the 
‘resize2fs’ method.

virt-resize: /dev/sda2: This partition will be left alone.

**********
[   5.2] Setting up initial partition table on debian9-9-z4_resized.qcow2
[   6.5] Copying /dev/sda1
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[ 236.6] Copying /dev/sda2
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[ 260.1] Expanding /dev/sda1 using the ‘resize2fs’ method
virt-resize: error: libguestfs error: resize2fs: e2fsck 1.47.0 (5-Feb-2023)
The filesystem size (according to the superblock) is 3407872 blocks
The physical size of the device is 3407824 blocks
Either the superblock or the partition table is likely to be corrupt!
Abort? yes

If reporting bugs, run virt-resize with debugging enabled and include the 
complete output:

  virt-resize -v -x [...]
```

I didn't allocate enough space... in the next step I made it a 0.1G larger. In the end virt-resize grew the ext4 file system to the slightly increased about of space. Maybe better to reduce the filesystem a bit more and let it be resized if you have strict space requirements. 

```
user1@dell4:~/tmp$ qemu-img create -f qcow2 -o preallocation=metadata debian9-9-z4_resized.qcow2 14.1G
Formatting 'debian9-9-z4_resized.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off preallocation=metadata compression_type=zlib size=15139759718 lazy_refcounts=off refcount_bits=16
user1@dell4:~/tmp$ virt-resize --shrink /dev/sda1 debian9-9-z4.qcow2 debian9-9-z4_resized.qcow2
[   0.0] Examining debian9-9-z4.qcow2
**********

Summary of changes:

virt-resize: /dev/sda1: This partition will be resized from 39.0G to 13.1G. 
 The filesystem ext4 on /dev/sda1 will be expanded using the 
‘resize2fs’ method.

virt-resize: /dev/sda2: This partition will be left alone.

**********
[   5.2] Setting up initial partition table on debian9-9-z4_resized.qcow2
[   6.5] Copying /dev/sda1
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[ 172.2] Copying /dev/sda2
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[ 196.0] Expanding /dev/sda1 using the ‘resize2fs’ method

virt-resize: Resize operation completed with no errors.  Before deleting 
the old disk, carefully check that the resized disk boots and works 
correctly.
user1@dell4:~/tmp$
```

Looks like I can save some more space with virt-sparsify.

```
user1@dell4:~/tmp$ ls -alh *.qcow2
-rw-r--r-- 1 user1 user1 33G Apr 16 23:55 debian9-9-z4.qcow2
-rw-r--r-- 1 user1 user1 15G Apr 17 00:24 debian9-9-z4_resized.qcow2
user1@dell4:~/tmp$
user1@dell4:~/tmp$ qemu-img info  debian9-9-z4.qcow2
image: debian9-9-z4.qcow2
file format: qcow2
virtual size: 40 GiB (42949672960 bytes)
disk size: 32.4 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
    extended l2: false
user1@dell4:~/tmp$ qemu-img info  debian9-9-z4_resized.qcow2
image: debian9-9-z4_resized.qcow2
file format: qcow2
virtual size: 14.1 GiB (15139760128 bytes)
disk size: 13 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false
user1@dell4:~/tmp$
```
    With the `--tmp .` option in the following command, I specify that the same directory should be used for tempfiles needed during the sparsifying process. My `/tmp` partition is only 20Gig.
```
user1@dell4:~/tmp$ virt-sparsify --tmp . debian9-9-z4_resized.qcow2 debian9-9-z4_resized_and_sparse.qcow2
[   0.0] Create overlay file in . to protect source disk
[   0.1] Examine source disk
[   4.7] Fill free space in /dev/sda1 with zero
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[  71.8] Clearing Linux swap on /dev/sda5
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[  77.7] Copy to destination and make sparse
[ 386.0] Sparsify operation completed with no errors.
virt-sparsify: Before deleting the old disk, carefully check that the 
target disk boots and works correctly.
user1@dell4:~/tmp$ qemu-img info  debian9-9-z4
debian9-9-z4.qcow2                     debian9-9-z4_resized_and_sparse.qcow2  debian9-9-z4_resized.qcow2
user1@dell4:~/tmp$ qemu-img info  debian9-9-z4_resized_and_sparse.qcow2 
image: debian9-9-z4_resized_and_sparse.qcow2
file format: qcow2
virtual size: 14.1 GiB (15139760128 bytes)
disk size: 5.8 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false
user1@dell4:~/tmp$ ls -alh *.qcow2
-rw-r--r-- 1 user1 user1  33G Apr 16 23:55 debian9-9-z4.qcow2
-rw-r--r-- 1 user1 user1 5.8G Apr 17 00:41 debian9-9-z4_resized_and_sparse.qcow2
-rw-r--r-- 1 user1 user1  15G Apr 17 00:24 debian9-9-z4_resized.qcow2
user1@dell4:~/tmp$
```

So, sparsifying did save some space.



# LVM

* check space used / available
  * `pvs` , `lvs`

* extend a partition
  * `lvextend -L +2G /dev/mapper/path_to_the_volume`
  * `resize2fs  /dev/mapper/path_to_the_volume` (assuming ext2/3/4)
  
