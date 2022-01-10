---
title: LPD (Lua Program Data)
tags: ["lpd", "lua", "scripts", "luac"]
weight: 0
---

LPD files are what store the games LUA logic.

# Table of contents

{{< toc >}}

## File Information

- **Location**: `$GAMEDIR/client/script`
- **Extensions**: `lpd`

## File Header

The LPD file header is a simple struct which follows the following format:

```cpp
struct header_t = {
  char file_type[4]; // Always "RLE\f"
  uint32 version; // Always 1
  uint32 file_size; // Unencrypted file size
  char padding;
}
```

## File Format

The LPD file format is fairly straight forward it contains a basic file header follow the above format,
then followed by the **compiled** lua script.

```cpp
struct lpd_t = {
  header_t header;
  char script[];
}
```

### Script section

The script is an "encrypted" form of the lua byte-code. The script can be decrypted using the following algorithm:

```cpp
  char decoded = encoded ^ 0x73
```

## File Name Format

File names are formatted as the following:
```
Normal Script: <script_name>.<le|be>.lpd
Program Script: <script_name>_p.<le|be>.lpd
```
_Note: LE means Little Endian and BE means Big Endian_

### Normal Script

A script is a script that can be required by a Program Script.

### Program Script

A program script is what the client loads to handle the actual game logic for the lua script. In the client it converts the `_p` to `.p` on file load.

## File Name Conversion

All file names in the `scripts` folder are encrypted, I haven't found the actual algorithm used, to encrypte them. But I did discover that it is a basic substitution cipher, with the following key:

_Note: This doesn't map all possible characters only the ones that I have observed to be used by the game._

```
Character | Substituted Character
0 | j
1 | i
2 | h
a | 9
b | 8
c | 7
d | 6
e | 5
f | 4
g | 3
h | 2
i | 1
j | 0
k | z
l | y
m | x
n | w
o | v
p | u
q | t
r | s
s | r
t | q
u | p
v | o
w | n
y | l
z | k
```

## Notes

- All LUA files are using LUA 5.1 and can be recovered using something like `LuaDec`.
- All Lua scripts are compiled using Little Endian.