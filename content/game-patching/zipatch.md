---
title: ZiPatch Format
tags: ["zipatch"]
---

A ZiPatch file is the compressed format used by FFXIV to patch the games files. A newer version of this same format is used by FFXIV: ARR. 

This documentation is for the ZiPatch format which is used by FFXIV 1.0 - 1.23b.

**Note: ZiPatch files are all Big Endian.**

# Table of contents

{{< toc >}}

## File Information

- **Location**: `Downloaded from patch server`
- **Extensions**: `patch`

## File Header

The file header is just 16 bytes long and it contains the following information:

```cpp
struct header_t {
  char file_type[8] // ALways ‘ZIPATCH
  char unknown[8] // Unknown bytes
}
```

## File Footer

The last 8 bytes of the file are the CRC32 of the file.

## Patch Blocks

In with sections contains all the data related to the ZiPatch Blocks.

### Patch Block Structure

```cpp
  struct block_t {
    char block_type[4] // Type of the block (FHDR, APLY. APFS, ADIR, DLED, ETRY)
    block_entry_t block_data[];
  }
```

`block_t` information
: Is a struct that matches one of the types below.


#### FHDR

The over all changes that occur within the patch file such as number of files, number of directories added, and removed.

```cpp
  struct fhdr_t {
    char version[4] // Version of the patch file always [00, 00, 02, 00]
    char result[4] // "DIFF" or "HIST"
    uint32 num_Entry_Files; // Number of files being patched
    uint32 num_Dirs_Added; // Number of directories added
    uint32 num_Dirs_Deleted; // Number of directories deleted
  }
```

#### APLY

Two APLY always follow a `FHDR` block, it's currently unknown what there purpose is.

```cpp
  struct aply_t {
    uint32 unknown1;
    uint32 unknown2;
    uint32 unknown3;
  }
```

#### APFS

**Note: This block is never actually used because FFXIV 1.X never released on anything other then PC**

Unknown to the purpose of this block but it is assumed to handle filesystem architecture in some way, maybe because the PS3 uses FAT32 and Windows uses NTFS.

```cpp
  struct apfs_t { 0 };
```

#### ADIR and DLED

These blocks are basically the same thing, ADIR is to add a directory and DLED is to delete a directory.

```cpp
  struct adir_t {
    uint pathLength;
    char path[];
  }

  struct dled_t {
    uint pathLength;
    char path[];
  }
```

#### ETRY

These blocks contains `chunks` of data that are to be used during patching.

```cpp
  struct etry_t {
    uint32 filePathLength;
    char filePath[];
    uint32 totalChunks;
    chunk_t chunks[];
    char padding[8];
  }
```


##### ETRY Chunk

The `chunk` of data that is apart of a `ETRY` block.

```cpp
  struct chunk_t {
    uint32 mode; // 0x41 = Add, 0x44 = Delete, 0x4D = Modify
    char previous_hash[20]; // Hash of the previous file
    char new_hash[20]; // Hash of the new file
    int compression_Mode; // 0x4E = Uncompressed, 0x54 Compressed (zlib)
    uint32 file_Size; // Size of the file based on the compression_mode
    uint32 previous_File_Size; // Size of the previous file
    uint32 new_File_Size; // Size of the new file
    char data[]; // The actual data size is based on the file_size
  }
```

Handling `mode` of a chunk
: `0x41`: Create a new file with the data
: `0x44`: Delete the given file
: `0x4D`: Modify the given file (Basically overwrite it with the new file)

Handling hashes
: When a file is being added `previous_hash` is set to 0x00
: When deleting a file `new_hash` is set to 0x00 and the data is set to 0x00

Handling `compression_mode`
: `0x4E`: Data stream is uncompressed
: `0x54`: Data stream is compressed with zlib