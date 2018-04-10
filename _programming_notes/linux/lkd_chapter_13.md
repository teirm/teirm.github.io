---
title: "Chapter 13: Virtual File System"
layout: page
topic: lkd 
---
This is the common interface between different file systems and
userspace.  Instead of reinventing a lot of kernel code, file systems
need only mold to the VFS layer.  It will do most of the work and does
not need to know the file system implementation details.

## Unix File Systems
Unix file systems are comprised of several objects
* Files - ordered byte string 
* Directory Entries - **dentries** or elements of a path 
* Inodes - holds the file meta data
* Superblock - holds the file system information

Additionally, through mount points, the file system provides a single name
space.  For example, local file system can be ext4 while an NFS system can
be mounted anywhere in the ext4 file system.

A non-unix interface, such as FAT32 or NTFS, can also be mounted *provided*
an interface is provided.

## VFS Objects
Expanding on the above definition of a File, in addition to being an ordered
string of bytes, the VFS layer has a file object which is associated with 
the process that opened it.  Additional VFS objects include dentries,
superblocks, and inodes.

Each of the these object types has a set of associated operations.  The VFS
layer provides a default implementation, but file system specific implementations
can be provided for some or all of the operations through a registration process
with the vfs layer (done during mount).

## The Super Block
The superblock is normally read off disk during mount or it is generated on the fly,
as is the case for *in-memory* file systems.  Operations on the superblock are 
accessible through the s_op field in the struct.

Since a file system need not implement all the VFS operations, a particular s_op
can be null.  In this case, the default implementation will be used.

All s_ops can be called in process context and all **except** dirty_inode may block.

(Expand on s_ops).

## Inodes
Read from disk and each struct has the iop field for inode operations.  

(Expand on iops).

## Dentries 
The dentry objects are any component of a path.  Unlike inodes and the superblock,
dentries are not kept on disk.  Instead, the VFS layer creates them on the fly.

Dentries can exist in three different states:
* used - correspond to a valid inode
* unusued - cached to a valid inode but not inuse
* negative - not associated with a valid inode.  The path is no longer valid or it
             is wrong.  These are kept cached to improve path resolution performance.

### The dcache
The dcache is a list of dentries linked off associated inode via i_dentry field (is 
this right? -- **DOUBLE CHECK**).  A doubly linked list of LRU unusued / negative
dentries.  Additionally a hash table and a hash function is used to quickly look
up dentries.

The dcache is essentially the front end of the inode cache.  As long as a dentry is in the
cache, the associated inode must be in memory.

## File Objects
Also does not correspond to any on disk data.  Instead it represents the view of an open
file by a process.  It points to a dentry.


# The File_System Object
Regardless of the number of mounts, there is 1 file system object for each file system.
It contains the capabilities and behavior of the file system.



