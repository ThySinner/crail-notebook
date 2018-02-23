# Deployment



## Configure Hugepage

### Enabling HugeTlbPage

Currently, there is no standard way to enable HugeTLBfs, mainly because the FHS has no provision for such kind of virtual file system, see 572733 . (Fedora mounts it in /dev/hugepages/, so don't be surprised if you find some example on the web that use this location)

Linux support "Huge page tables" (HugeTlb) is available in Debian since DebianLenny (actually, since 2.6.23). A good introduction to large pages is available from ibm.com.

1. Create a group for users of hugepages, and retrieve it's GID (is this example, 2021) then add yourself to the group.Note: this should not be needed for libvirt (see /etc/libvirt/qemu.conf)

	```bash
	% groupadd my-hugetlbfs
	
	% getent group my-hugetlbfs
	my-hugetlbfs:x:2021:
	
	% adduser franklin my-hugetlbfs
	Adding user `franklin' to group `my-hugetlbfs' ...
	Adding user franklin to group my-hugetlbfs
	Done.
	```
	
2. Edit /etc/sysctl.conf and add this text to specify the number of pages you want to reserve (see pages-size)

	```bash
	# Allocate 256*2MiB for HugePageTables (YMMV)
	vm.nr_hugepages = 256
	
	# Members of group my-hugetlbfs(2021) can allocate "huge" Shared memory segment 
	vm.hugetlb_shm_group = 2021
	```

3. Create a mount point for the file system

	```bash
	% mkdir /hugepages
	```

4. Add this line in /etc/fstab (The mode of 1770 allows anyone in the group to create files but not unlink or rename each other's files.1)

	```bash
	hugetlbfs /hugepages hugetlbfs mode=1770,gid=2021 0 0
	```

5. Reboot (This is the most reliable method of allocating huge pages before the memory gets fragmented. You don't necessarily have to reboot. You can try to run sysctl -p to apply the changes. if grep "Huge" /proc/meminfo don't show all the pages, you can try to free the cache with sync ; echo 3 > /proc/sys/vm/drop_caches (where "3" stands for "purge pagecache, dentries and inodes") then try sysctl -p again. 2)

### limits.conf

You should configure the amount of memory a user can lock, so an application can't crash your operating system by locking all the memory. Note that any page can be locked in RAM, not just huge pages. You should allow the process to lock a little bit more memory that just the the space for hugepages.

```bash
## Get huge-page size:
% grep "Hugepagesize:" /proc/meminfo
Hugepagesize:       4096 kB

## What's the current limit
% ulimit -H -l
64

## Just add them up... (how many pages do you want to allocate?)
```

See Limits (ulimit -l and memlock in /etc/security/limits.conf).
