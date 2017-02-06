# btrfs-status
My personal assessment on btrfs status vs the official one at https://btrfs.wiki.kernel.org/index.php/Status

## Foreword
After some years of using btrfs and following the official mailing list, I still see weekly horror stories posted there from unsuspecting users. I have come to have a personal idea of what is stable in the btrfs world, which I offer here to any interested party. 

Additionally, some missing characteristics in a feature that I consider a must have may cause my marking them as incomplete.

Finally, this is my subjective interpretation; I may be wrong or unfairly critical, refer to btrfs experts if in doubt.

## The main gotcha with btrfs
Since the [official recommendation](https://btrfs.wiki.kernel.org/index.php/Main_Page#Stability_status) is to use the most recent kernel possible, be ready to be scolded when you report a problem while not using it. That's fine, I can understand why developers say so and too why users may not able to follow the recommendation. However, that's not the problem, but that when you obey and use the bleeding edge kernels, you better be ready to find new bugs and possibly have to compile patched kernels. 

So there you have it. Run "old" kernels with known bugs or recent ones with unknown ones. And that is my main gripe with btrfs at present.

## Features

Only features where I disagree with the [official assessment](https://btrfs.wiki.kernel.org/index.php/Status).

*Feature* | *Status* | *Notes* | *Example references* 
--- | --- | --- | --- 
Subvolumes, snapshots | Mostly OK | Don't count on having lots (>1k?) of them | [1](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg45295.html), [2](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg24469.html), [3](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg38289.html), [4](http://www.spinics.net/lists/linux-btrfs/msg52881.html), [20160216](http://www.spinics.net/lists/linux-btrfs/msg52131.html) 
Subvolumes, snapshots | Incomplete | No recursive snapshotting | [20140708](http://stackoverflow.com/questions/24625712/how-to-take-a-recursive-snapshot-of-a-btrfs-subvol), [2014](http://linux-btrfs.vger.kernel.narkive.com/A2x0iFeW/planning-for-subvolumes-of-subvolumes-and-btrfs-send-receive), [20160228](https://www.mail-archive.com/linux-btrfs@vger.kernel.org/msg51115.html)
Send | Incomplete | No recursive sending | [20160227](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg51113.html)
Receive | Mostly OK | Read-only snapshots are writable during receive | [20170131](https://www.spinics.net/lists/linux-btrfs/msg62524.html)
RAID1 | Mostly OK | Failure may give you only one chance at mounting read-write for rebuild | [1](https://btrfs.wiki.kernel.org/index.php/Gotchas#raid1_volumes_only_mountable_once_RW_if_degraded)
RAID10 | Mostly OK | RAID10 in btrfs does not mean what some people expect | [20161201](https://www.spinics.net/lists/linux-btrfs/msg61074.html), [20160603](http://www.spinics.net/lists/linux-btrfs/msg55829.html), [20140103](http://www.spinics.net/lists/linux-btrfs/msg30373.html), [20131119](http://www.spinics.net/lists/linux-btrfs/msg29282.html)
Quotas | Unstable | May eat lots of RAM/CPU, hang computer because of OOM | [20170131](https://www.spinics.net/lists/linux-btrfs/msg62508.html) (This also happened to me in once, had to use a live 4.9 USB to disable quotas, took many hours)
Balance | Unstable | Incomplete balances may cause unescapable read-only mount | [1](https://btrfs.wiki.kernel.org/index.php/Gotchas#Incomplete_chunk_conversion) (personally bitten when reshaping a single to RAID1, IIRC)

## Final admonitory
Do not forget to check the [official Gotchas page](https://btrfs.wiki.kernel.org/index.php/Gotchas)
