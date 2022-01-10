---
title: Boot
---

The "ffxivboot.exe" executable is the main entry point for the game.

  

# Table of contents

{{< toc >}}


## Functionality

The "ffxivboot.exe" primary function is to check the Patch server if the game's files are up to date by doing a get call to

``` http://ver01.ffxiv.com/patch/vercheck/ffxiv/win32/release/{type}/{currentVersion}```

if there are any updates, then the server will respond with the needed torrents files, and then the "ffxivboot.exe" will act as a torrent client to download the files and pass them to "ffxivupdater.exe" after it finishes the download

if there are no updates to be found, then "ffxivboot.exe" will launch "ffxivlogin.exe."

When Launching "ffxivlogin.exe" , the following command line argument is used 
```
{workingDirectory}\ffxivlogin.exe sqex0001{base64string}////
```

the values that are used to generate the base64 are still unknown

## Versions

There are two versions for ffxivboot

1. 2010.07.10.0000 ```SHA1:999EFB09D7D94C9C8106A75688CF3BED7C1FBA84```

2. 2010.09.18.0000 ```SHA1:598a0b090c496f392947055f2241e44218730668 ```

The First one gets installed through the initial game setup; as for the latter, it gets fetched and updated through the Patch server.

There is no difference in functionality between the two versions; it's possible to use the stock version with the latest login and game exe.

## RSA

When downloading the patch from the Patch Server, "ffxivboot.exe" will run a series of checks to see if the source is actually "Square Enix,"

The Offset for the RSA Function in ffxivboot

```c

Stock Version : 0x5DF50

Updated Version: 0x64310

```

  

These functions need to be patched out to bypass the checks and use a custom Patch Server.

  

Here's a recomened and tested patching value for the RSA function

``` 0xB8, 0x1F, 0x00, 0x00, 0x00, 0xC3 ```

  

along with that the function needs to return true all the time

```c

Stock Version : 0x5E32C

Updated Version: 0x646EC

Patch Value : 0x5E, 0x87, 0x48, 0x48, 0x06, 0xF8, 0x88

```

  
  

## Endpoints

  

There are references to the following endpoints within "ffxivboot.exe."

  

* "lobby01.ffxiv.com"

    * ``` Stock Version Offset : 0x8E5C6C```

    * ``` Updated Version Offset : 0x965D08```

* "secure.square-enix.com"

    * ``` Stock Version Offset : 0x90A4A0```

    * ``` Updated Version Offset : 0x99212C```

* "ver01.ffxiv.com"

    * ``` Stock Version Offset : 0x8E62DC```

    * ``` Updated Version Offset : 0x966404```

  
  

The default port for the Patch Server is "54996."

```c

Stock Version Offset : 0x8E62D4

Updated Version Offset : 0x9663FC

```

to be able to use a custom patch server, only "ver01.ffxiv.com" and the port needs to be modified