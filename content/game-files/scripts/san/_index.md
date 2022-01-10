---
title: SAN (Static Actors)
tags: ["san", "static actors"]
weight: 1
---

The SAN file is what stores all of the games static actors. A static actor is mapped to a 
[LPD file]({{<ref "game-files/scripts/lpd/_index.md">}}) to handle that given actors logic.

# Table of contents

{{< toc >}}

## File Information

- **Location**: `$GAMEDIR/client/script`
- **Extensions**: `san`
- **Encrypted File Name**: `rq9q1797qvs.san`
- **Decrypted File Name**: `StaticActor.san`

## File Header

The SAN file header is a simple struct which follows the following format:

```cpp
struct header_t = {
  char file_type[4]; // Always "SANE"
  char unknown[8]; // Unknown
  char padding;
}
```

## File Format

The SAN file format is fairly straight forward it contains a basic file header following with a list of all known actors within the game.

```cpp
struct lpd_t = {
  header_t header;
  actor_t actors[];
}

struct actor_t = {
  uint32 actor_id; // Actor ID as BE processed with a logical or of 0xA0F00000
  char actor_name[...0x0]; // This is the name of the actor which is null terminated
  char padding;
}
```

### Actors section

The actors is an "encrypted" and can be decrypted using the following algorithm:

```cpp
  char decoded = encoded ^ 0x73
```