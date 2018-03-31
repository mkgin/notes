# VM image management...

* mount partition from raw img file
 * http://madduck.net/blog/2006.10.20:loop-mounting-partitions-from-a-disk-image/

* raw to qcow
 * ```qemu-img convert -c -f raw -O qcow2 dev_sda.raw dev_sda.qcow2```
 * see also zerofree virt-sparsify
