---
title: Client Patching
---

# Table of contents
{{< toc >}}

## Version Tracking Server
Server Path: `/patch/vercheck/ffxiv/win32/release/{type}/{currentVersion}`

The version tracking server `ver01.ffxiv.com` is used to get the latest version of the client. This also serves as the download point for all the torrent files needed to download the proper patch files.

The server returns certain headers which are used to both verify the patch is from from `ver01.ffvix.com` and get the latest version.

Type hash mapping:
: `2d2a390f`: Boot
: `48eca647`: Game

### 200 (Updating)

#### Headers
```
  Content-Location: ffxiv/{type}/vercheck.dat
  Content-Type: multipart/mixed; boundary=477D80B1_38BC_41d4_8B48_5273ADB89CAC
  X-Repository: ffxiv/win32/release/boot
  X-Patch-Module: ZiPatch
  X-Protocol: torrent // Only torrent is supported
  X-info-url: http://example.com // Never gets used
  X-Latest-Version: {latestVersion} // Latest version of the given {type}
  Connection: keep-alive
```

#### Body
```
  --477D80B1_38BC_41d4_8B48_5273ADB89CAC
  Content-Type: application/octet-stream
  Content-Location: ffxiv/{type}/metainfo/D{version}.torrent
  X-Patch-Length: {size} // Size of the .patch file to download from the torrent
  X-Signature: jqxmt9WQH1aXptNju6CmCdztFdaKbyOAVjdGw_DJvRiBJhnQL6UlDUcqxg2DeiIKhVzkjUm3hFXOVUFjygxCoPUmCwnbCaryNqVk_oTk_aZE4HGWNOEcAdBwf0Gb2SzwAtk69zs_5dLAtZ0mPpMuxWJiaNSvWjEmQ925BFwd7Vk= // Signature of the response body
  // Empty Line
  ... Binary Data or torrent file ...
```
Once you loop over versions starting at the request version to the latest verison you must end the request with 
` --477D80B1_38BC_41d4_8B48_5273ADB89CAC`

### 204 (No Update)
```
  Content-Location: ffxiv/{type}/vercheck.dat
  Content-Type: multipart/mixed; boundary=477D80B1_38BC_41d4_8B48_5273ADB89CAC
  X-Repository: ffxiv/win32/release/boot
  X-Patch-Module: ZiPatch
  X-Protocol: torrent // Only torrent is supported
  X-info-url: http://example.com // Never gets used
  X-Latest-Version: {latestVersion} // Latest version of the given {type}
  Connection: keep-alive
```
