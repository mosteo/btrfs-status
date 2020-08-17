# btrfs stability/maturity status - btrfs-status
My personal assessment on the btrfs stability/maturity status vs the official one at https://btrfs.wiki.kernel.org/index.php/Status

## Foreword
After some years of using btrfs and following the official mailing list, I still see weekly horror stories posted there from unsuspecting users. I have come to have a personal idea of what stable means in the btrfs world, which I offer here to any interested party. 

Sure, this is my subjective interpretation; I may be wrong or unfairly critical, refer to btrfs experts if in doubt. Also note that, despite the apparent criticism, I do use btrfs in my systems when I consider the advantages outweight the risks.

This list is likely to become outdated as btrfs evolves since I'm not 100% on top of it. Corrections welcomed.

## The main gotcha with btrfs
Since the [official recommendation](https://btrfs.wiki.kernel.org/index.php/Main_Page#Stability_status) is to use the "most modern kernel possible", be ready to be friendly [scolded](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg61252.html) when you report a problem while not using it. That's fine, I can understand why developers say so and too why users may not able to follow the recommendation. However, that's not the problem, but that when you obey and use the bleeding edge kernels, you better be ready to find new bugs and possibly have to compile patched kernels. 

So there you have it. Run "old" kernels with known bugs or recent ones with unknown ones. And that is my main gripe with btrfs at present.

## Features

Only features where I disagree with the [official assessment](https://btrfs.wiki.kernel.org/index.php/Status).

*Feature* | *My Status* | *Notes* | *References* 
--- | --- | --- | --- 
Subvolumes, snapshots | Mostly OK | Don't count on having lots (>1k?) of them | [1](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg45295.html), [2](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg24469.html), [3](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg38289.html), [4](http://www.spinics.net/lists/linux-btrfs/msg52881.html), [20160216](http://www.spinics.net/lists/linux-btrfs/msg52131.html) 
Subvolumes, snapshots | Gotcha | No recursive snapshotting | [20140708](http://stackoverflow.com/questions/24625712/how-to-take-a-recursive-snapshot-of-a-btrfs-subvol), [2014](http://linux-btrfs.vger.kernel.narkive.com/A2x0iFeW/planning-for-subvolumes-of-subvolumes-and-btrfs-send-receive), [20160228](https://www.mail-archive.com/linux-btrfs@vger.kernel.org/msg51115.html)
Send | Gotcha | No recursive sending | [20160227](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg51113.html)
Receive | Gotcha | Read-only snapshots are writable during receive | [20170131](https://www.spinics.net/lists/linux-btrfs/msg62524.html)
Receive+Quotas | Unstable | Can cause corruption | [20170731](https://www.spinics.net/lists/linux-btrfs/msg67788.html)
RAID1+NODATACOW | Unstable | Data corruption likely on transient device failure | [20180629](https://www.mail-archive.com/linux-btrfs@vger.kernel.org/msg78254.html)
RAID1, RAID10 | Gotcha | RAID1, RAID10 in btrfs has a different meaning | [20170601](https://www.spinics.net/lists/linux-btrfs/msg66115.html), [20161201](https://www.spinics.net/lists/linux-btrfs/msg61074.html), [20160603](http://www.spinics.net/lists/linux-btrfs/msg55829.html), [20140103](http://www.spinics.net/lists/linux-btrfs/msg30373.html), [20131119](http://www.spinics.net/lists/linux-btrfs/msg29282.html)
Quotas | Unstable | May eat lots of RAM/CPU, hang computer because of OOM. | [20175024](https://www.spinics.net/lists/linux-btrfs/msg65796.html) ("it is now commomn knowledge", yet "mostly OK" in official assesment), [20170131](https://www.spinics.net/lists/linux-btrfs/msg62508.html) (This happened to me once, had to use a live 4.9 USB to disable quotas, took many hours)
Quotas | Unstable | May grealy slow down balance operations | [20170206](https://mail-archive.com/linux-btrfs@vger.kernel.org/msg61255.html)
Balance | Unstable | Incomplete balances may cause unescapable read-only mount | [1](https://btrfs.wiki.kernel.org/index.php/Gotchas#Incomplete_chunk_conversion) (personally bitten when reshaping a single to RAID1, IIRC)
Balance | Unstable | Interrupting a balance might leave the system unmountable | [20170607](https://www.spinics.net/lists/linux-btrfs/msg66298.html)
Repair | Unstable | Different features/bugs between normal/lowmemory, can use large amounts of RAM | [20180629](https://www.mail-archive.com/linux-btrfs@vger.kernel.org/msg78315.html)
Free space | Mostly OK | You can get "no free space left" even with plenty (many GBs) of it. A balance, or deletion of files + balance, should fix it | TBD, but experienced as recently as 2018-10
btrfsck | Gotcha | Using `--repair` can be destructive, and there is no warning unlike for full balances | TBD

## Fixed issues (for future reference)
Those should be OK now.

*Feature* | *My Status* | *Notes* | *References* 
--- | --- | --- | --- 
RAID1 | Mostly OK | Failure may give you only one chance at mounting read-write for rebuild | [1](https://btrfs.wiki.kernel.org/index.php/Gotchas#raid1_volumes_only_mountable_once_RW_if_degraded)

## Horror diary
* 2020-08-17: As closing to the previous entry: after trying all I could think of and backing up the data, I dared to use `btrfsck --rescue` and guess what, it worked. Not as solidly as one would want: the first time it found and fixed things, the second time it found and fixed _more_ things, and thereafter it was silent and happy. It did what your regular `fsck` used to do in FAT times: found a bunch of dangling files and put them in `lost+found`. After that, I could remove the devices without trouble. And, since I'm daring that way, I'm still using the same filesystem without recreating it.
* 2020-07-29: Oh sad day. After a long time not being bitten by any surprises, yesterday had an HDD start announcing bad sectors via SMART. Time for btrfs to shine! I was ready, being this disk part of a two disks filesystem. No dice. When doing the remove (I had plenty of space to spare), at the time of moving out the metadata blocks, btrfs falls on its sword. `dmesg` info points to corrupted data in the metadata (not unable to read sectors with metadata). A few funny observations:
   * The filesystem was created on May 2017 (snapshot info says this). Too old to live?
   * There were a couple of files with I/O errors on the bad sectors that I could delete without issue by mapping the inode reported by scrub on dmesg.
   * Scrub says the fs is clean (oh sweet summer child).
   * `btrfs check` says the roots contain errors (that's more like it).
   * The drive can be `btrfs replace`d, but the new pristine drive cannot be then removed anyway, with the same errors. It seems `replace` is dutifully copying whatever error is encroached in the metadata to the replacement drive.
   * The fs has single profile data, so I cannot mount without the failing drive in rw mode and go from there, losing the scraps that remain in the bad device (it's scraps, because `btrfs dev del` moves everything out except the last metadata block, which furthermore is raid1!).
   * For posterity, here is a pastebin with the [errors](https://pastebin.com/QK0vZUzB).
* 2018-09-19: An [eye-opener post](https://www.mail-archive.com/linux-btrfs@vger.kernel.org/msg80651.html) by a btrfs developer on recommended practices to keep your btrfs stable.
* 2018-02-15: Not a personal experience but [these kind of threads](https://www.spinics.net/lists/linux-btrfs/msg74892.html) bring tears to my eyes.
* 2017-08-24: [Red Hat has recently announced](https://www.phoronix.com/scan.php?page=news_item&px=Red-Hat-Deprecates-Btrfs-Again) that they are giving up on btrfs. I will say I'm not entirely surprised, since what has amounted to relatively small sporadic frustrations to me must be a maintenance hell on large scales. I worry though that they'll simply reinvent another wheel and push back the timeframe for a robust and flexible solution some years more. Or maybe not, we'll see.
* 2017-07-20: [Not my own experience](https://www.spinics.net/lists/linux-btrfs/msg67489.html), but a nice btrfs scare with a happy ending. Remember, don't do "several big things simultaneously" in btrfs world or else the Unholy Locking Mess™ can come and bite you. Also contains such pearls as "choose your kernel versions wisely. Right now, the 4.9 LTS kernel is a good bet" (c.f. the "use the most modern kernel" official advice). BTW: these cites are from a list member whose bg I don't know, and I'm not blaming him or taking his words as official or The Truth™. It's just a daily scene of the btrfs perception on-list.
* 2017-02-22: After removing a device from a three-deviced fs, the `btrfs dev remove` command never finished, when I/O activity and `btrfs fi sh` suggested that many hours had gone after completion. Fortunately, it seems everything but the last wiping of signatures did complete. This was my `/home`, and for some reason it mounts normally when I do it from recovery, but won't automatically mount in a normal boot. (For the curious: this started as a single device fs; I added two more (withouth changing profiles, so the single/dup data/meta profiles became automatically single/raid1) and afterwards tried to remove the original device. Then is when `btrfs dev remove` got stuck. Kernel is 4.8.0-36 in Ubuntu 16.04.  
  * Update: doing a `wipefs -a` on the stale device made things get back to normal. Reassuring. Also, I just read http://www.virtualtothecore.com/en/2016-btrfs-really-next-filesystem/ and couldn't but laugh. I guess Facebook has more redundancy in place than your typical home user... (not that what I do to btrfs is typical from home users either).
* 2017-02-14: After a hard freeze (probably graphics-driver related) and reboot, my root partition, a plain, defaults, single profile btrfs panics at mount time. Unbootable system, my first where I tried to use btrfs for the root partition, lasted less than a month :'(. Home seems healthy though. I haven't tried yet to repair the filesystem. Kernel is 4.4.0 from Ubuntu 16.04.1 LTS.  
  * Update 2017-02-15: Mounting from a live 4.8 kernel made the issue go away...
* 2017-01-09: Quotas in a backup drive (used to identify individual space usage in snapshots) made mount/unmount take ages, 100% IO load during hours. Solved by disabling quotas from live rolling release OpenSuse usb (couldn't with the 16.04.1 Ubuntu installation).

## Final admonitory
If your filesystem some day refuses to mount, you might be tempted to jump into btrfsck. Instead, a cursory search will point out that there is a preferred path of solutions to attemp from less to more damage, that don't start at btrfsck. For example:
* http://blog.tinola.com/?e=43
* https://bbs.archlinux.org/viewtopic.php?id=182505

There is a funny quote in that last link: *"... you should not run btrfsck with the --repair flag unless told to do so by the developers.  This of course isn't very intuitive, but at the moment is just the way it is"*. This was three years ago (2014), however.

On that note, even repairing seems to be provoking headaches as recently as [20170521](https://www.spinics.net/lists/linux-btrfs/msg65720.html) due to memory requeriments.

Also do not forget to check the [official Gotchas page](https://btrfs.wiki.kernel.org/index.php/Gotchas)
