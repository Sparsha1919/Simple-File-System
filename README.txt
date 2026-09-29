CSE321 Lab Term Project - Summer 2026
SimpleFS: Implementation of a Simple File System in C
========================================================

Group Number: 6

Group Members:
  1. Md. Wasifur Rahman   - 24101474
  2. Mehedi Hasan Rowdro  - 24101226
  3. Md. Abu Zubayer      - 22301218


FILES SUBMITTED
---------------
  simplefs.h          - Fixed structures and constants (unmodified)
  simplefs_builder.c  - Creates and initializes an empty SimpleFS image
  simplefs_adder.c    - Adds a regular file into an existing SimpleFS image
  README.txt          - This file


COMPILATION COMMANDS
---------------------
  gcc -Wall -Wextra -std=c11 simplefs_builder.c -o simplefs_builder
  gcc -Wall -Wextra -std=c11 simplefs_adder.c -o simplefs_adder

Both programs compile cleanly with no warnings under -Wall -Wextra -std=c11.


EXECUTION EXAMPLES
--------------------
1) Create a fresh, empty SimpleFS image:
   ./simplefs_builder --image disk.img
   ls -l disk.img
   (Expected: disk.img is exactly 262144 bytes)

2) Add a file to the image:
   ./simplefs_adder --input disk.img --file test1.txt

3) Add multiple files:
   ./simplefs_adder --input disk.img --file test1.txt
   ./simplefs_adder --input disk.img --file test2.txt
   ./simplefs_adder --input disk.img --file test3.txt

4) Inspect the resulting image (Linux):
   xxd disk.img
   or
   hexdump -C disk.img

BRIEF DESCRIPTION OF IMPLEMENTATION
-------------------------------------
simplefs_builder.c
  Creates a 262144-byte (64 x 4096) image and zero-fills it. It then:
    - Fills and writes the superblock (magic number, block size, total
      blocks, inode count, and the block locations of every metadata
      region) to Block 0.
    - Sets bit 0 of the inode bitmap (Block 1) to mark inode 1 (root)
      as allocated.
    - Sets bit 0 of the data bitmap (Block 2) to mark Block 4 (root's
      data block) as allocated.
    - Initializes root inode 1 (type = directory, links = 2, size = 128,
      direct[0] = 4) and writes it to its slot in the inode table
      (Block 3).
    - Creates the "." and ".." directory entries (both pointing to
      inode 1) and writes them at the start of Block 4.

simplefs_adder.c
  Opens an existing SimpleFS image and validates its magic number, then
  adds one regular file from the current working directory:
    - Rejects files larger than 12288 bytes (3 direct blocks x 4096)
      and file names longer than 58 characters.
    - Searches the root directory for a duplicate file name.
    - Uses first-fit allocation to find a free inode (inode bitmap) and
      the required number of free data blocks (data bitmap).
    - Copies the source file's bytes into the allocated data blocks
      using zero-filled 4096-byte buffers, so any unused tail bytes in
      the final block remain zero.
    - Builds the new file's inode (type = file, links = 1, actual byte
      size, and up to 3 absolute direct block numbers) and writes it to
      the inode table.
    - Marks the new inode allocated in the inode bitmap and the newly
      used blocks allocated in the data bitmap.
    - Adds a new 64-byte directory entry (name -> inode number) in the
      first free slot of the root directory (entries 2-63).
    - Increases the root inode's size by 64 bytes (sizeof(dirent_t))
      for the new entry.

TESTING PERFORMED
--------------------
The following test cases from the project specification were run and
verified using od/xxd/hexdump against the exact expected byte values:

  - Test Case 1  : Empty image is exactly 262144 bytes
  - Test Case 2  : Superblock fields correct at Block 0
  - Test Case 3  : Initial inode bitmap = 01
  - Test Case 4  : Initial data bitmap = 01
  - Test Case 5  : Root directory contains "." and ".." entries
  - Test Case 6  : Small one-block file added and content verified
  - Test Case 7  : Inode bitmap = 03 after one file
  - Test Case 8  : Data bitmap = 03 after one small file
  - Test Case 9  : File content verified byte-for-byte in Block 5
  - Test Case 10 : Two-block (5000-byte) file -> data bitmap = 07,
                   direct[0] = 5, direct[1] = 6
  - Test Case 11 : 12288-byte file accepted; 12289-byte file rejected
  - Test Case 12 : Three files added sequentially receive inodes 2, 3, 4
                   under first-fit allocation
  - Test Case 13 : Duplicate file name rejected with correct error
  - Test Case 14 : Missing source file rejected with correct error
  - Test Case 15 : Missing image file rejected with correct error,
                   no crash

CONTRIBUTION OF EACH GROUP MEMBER
------------------------------------
Md. Wasifur Rahman (24101474)
  - Implemented simplefs_builder.c in full (superblock initialization,
    inode/data bitmap setup, root inode initialization, "." and ".."
    directory entry creation).
  - Verified builder output against the specification's Test Cases 1-5
    using byte-level inspection.

Mehedi Hasan Rowdro (24101226)
  - Implemented simplefs_adder.c (first-fit inode/data-block allocation,
    duplicate-name and free-directory-entry search, file-size
    calculation).

Md. Abu Zubayer (22301218)
  - Implemented simplefs_adder.c (data copying into allocated blocks,
    new-file inode initialization, bitmap updates, directory-entry
    creation, root inode size update, and required error handling).
  - Verified adder output against the specification's Test Cases 6-15
    using byte-level inspection.