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
```
--477D80B1_38BC_41d4_8B48_5273ADB89CAC--
```
**Example Response**
```
  Content-Location: ffxiv/{type}/vercheck.dat
  Content-Type: multipart/mixed; boundary=477D80B1_38BC_41d4_8B48_5273ADB89CAC
  X-Repository: ffxiv/win32/release/boot
  X-Patch-Module: ZiPatch
  X-Protocol: torrent // Only torrent is supported
  X-info-url: http://example.com // Never gets used
  X-Latest-Version: {latestVersion} // Latest version of the given {type}
  Connection: keep-alive

  --477D80B1_38BC_41d4_8B48_5273ADB89CAC
  Content-Type: application/octet-stream
  Content-Location: ffxiv/{type}/metainfo/D{version}.torrent
  X-Patch-Length: {size} // Size of the .patch file to download from the torrent
  X-Signature: {signature} // Signature of the response body
  // Empty Line
  ... Binary Data or torrent file ...
  --477D80B1_38BC_41d4_8B48_5273ADB89CAC
  Content-Type: application/octet-stream
  Content-Location: ffxiv/{type}/metainfo/D{version}.torrent
  X-Patch-Length: {size} // Size of the .patch file to download from the torrent
  X-Signature: {signature} // Signature of the response body
  // Empty Line
  ... Binary Data or torrent file ...
  --477D80B1_38BC_41d4_8B48_5273ADB89CAC
  Content-Type: application/octet-stream
  Content-Location: ffxiv/{type}/metainfo/D{version}.torrent
  X-Patch-Length: {size} // Size of the .patch file to download from the torrent
  X-Signature: {signature} // Signature of the response body
  // Empty Line
  ... Binary Data or torrent file ...
  --477D80B1_38BC_41d4_8B48_5273ADB89CAC--
```


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

## Torrent Server

All handshake/torrent request from the client updater hash the first 16 bytes of the torrents info hash using it's own peerid. The resulting handshake from the seeder delivering the update decrypted the incoming infohash, discovers the proper torrent, and then send the handshake back encrypting the first 16 bytes with it's own peerid. 

_Note: The last 4 bytes of the incoming handshake info hash are the same as a tracked torrent_

```csharp
  var incomingHash = new byte[20];
  // This is the hash we found using the last 4 bytes
  var discoveredHash = new byte[20];
  var bf = new BlowFish("-SQ0001-{GeneratedId}");
  bf.Encrypt(discoveredHash, 0, 16);

  // Send the handshake back
```

### Some thoughts

We know the incoming hashes `Blowfish` key so we could decrypt the incoming hash and discover the proper torrent based on the full info hash. This would allow us to be more sure we are sending back the correct torrent every time.

Maybe we could update the modified version of `monotorrent` to handle this instead of always using the last 4 bytes of the info hash. This requires more research due to how the announcement server works.