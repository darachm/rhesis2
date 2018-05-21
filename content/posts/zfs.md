---
title: "zfs"
date: 2018-05-20T10:01:23-04:00
draft: true
---


Tried using zfs. That was a mess. Seems great and all.

One major issue, and I think it's a bug to not document all your
flags and methods to deal with common problems even in your online
documentation, was when I woke up after scheduling a large transfer
and saw my external disk unplugged from the computer. Maybe it was
a cat.

This triggers the error `zpool: pool I/O is currently suspended`.
Which is a problem. What's going on is that because I added the
pool by specifying `/dev/sdb` and `/dev/sdc`, well those just changed
because someone pulled the plug. And that means those are now at
`/dev/sdf` and `/dev/sdg`.

[^zfs104]

[^docz]

So the short term fix is to link them, with 
`link /dev/sdf /dev/sdb`, `link /dev/sdf1 /dev/sdb1`, and
`link /dev/sdf9 /dev/sdb9`. This links all the partitions over, and
then it can continue to `zpool clear` and `zpool export` the disk.

The long term fix is to ALWAYS use 
`sudo zpool import -d /dev/disk/by-id that_pool` when importing.
So my intuition is that zfs is then looking in that directory and
reading all disks to look for it's filesystems and IDs on the disk,
and then mounting them. Importantly, when it looks in there then
it remembers the disk ID, which doesn't change when you unplug and
replug it. Then a `zpool clear` fixed it (for me).


[zfs104]: https://github.com/openzfsonosx/zfs/issues/104#issuecomment-30363146

[docz]: https://github.com/zfsonlinux/zfs/wiki/faq#changing-dev-names-on-an-existing-pool
