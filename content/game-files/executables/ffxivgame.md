---
title: Game
---

"ffxivgame.exe" is the main game executable

# Table of contents
 
{{< toc >}}

## Lobby

Once the game starts up it will try to communicate with the lobby server ``` lobby01.ffxiv.com ``` on the port ``` 54994 ```

to be able to connect the game to a custom server , the lobby server address needs to be patched, it can be patched with either a hostname or a IP address

The lobby hostname is read from the following offset
```c
0xB90110
```
