## 1. File Concept
- This is the smallest allocation unit for the user.
- File system abstracts how they are mapped into secondary storage (hard disk).
- Similar to process, file have contiguous logical address space (but not physical).
- Different types: data file and program files (executable).
- Contents defined by file's creator.
- A file has some attributes regardless the platform:
	- Name
	- Identifier (unique number within the file system)
	- Type
	- Location
	- Size
	- Protection
	- Time, date and user identification 
## 2. Open a file
- Providing a File System implies that the OS needs to maintain information, including the open file:
	- Tracks open file
	- File pointer to point to las read/write location, per process that has the file open.
	- File open count: counter of number of times a file is open - to allow removal of data from open-file table when last process closes it.
	- Disk location of the file: cache of data access information (for saving time).
	- Access rights: per-process access mode information (read only, read and write,...).

- Other things to do with a file are **Operations**.

## 3. Access method to a file
- Access method
	- Sequential access with read next, write next and reset (like traversal through a linked list).
	- Direct access, the file is fixed length logical record
		- read n
		- write n
		- position to n
			- read next
			- write next
		- rewrite n(with n is the relative block number, equals to 0, being the beginning of the file)
- Which is the appropriate size of a block?
	- If too large:
		- Less blocks to read, which leads to faster reading
		- But more wasted space, when the file size is smaller than the capacity of a block.
	- If too small
		- We have more blocks to read, resulting in slower reading.
		- But less space wasted!
- For a block size, want to maximum the speed (speed test or something), try to use the smaller block size. In the contrast, use the bigger to leverage the disk space utilization.
	![[Block size, data rate and disk space utilization.png]]

## 4. Disk structure
We will have 2 kind, the MBR (master boot record) and the UEFI (Unified Extensible Firmware Interface)
![[MBR Layout.png]]
For the MBR, the first sector (sector 0) contains the information for the booting process of the computer. At the end, it also contains the partition table. 
The limitations of this structure
- Slow, with 512 bytes in total.
- Wants 15-bit real mode on start up with no paging.
- Max 2TB support (because it has 32 bits for address, 32 for size with sector of 512 bytes).

With the UEFI system (GPT is Global Unique Identifier - G Partition Table)
![[UEFI Layout.png]]
- UEFI Firmware is powerful enough to read a specific partition, called EFI System Partition (ESP).
- This is a proper file system with programs and configuration file.
- Can load and execute programs in a specific format (PE - Portable executable).