---
title: Login
---
 
"ffxivlogin.exe" is a IE based broweser that's used to render the Login page and read the session ID from the header once it's populated
 
# Table of contents
 
{{< toc >}}
 
## Login Page
 
The original Login Page URL was ```http://account.square-enix.com/account/content/ffxivlogin``` it's present at the following offset in the binary file
 
```
0x53EA0
```
 
The URL is encoded and not present as a plain text
 
### URL Encoding
Source : https://bitbucket.org/Ioncannon/project-meteor-server/src/master/Launcher%20Editor/Program.cs
```csharp
public byte[] FFXIVLoginStringEncode(uint key, string text)
    {
        key = key & 0xFFFF;
 
        uint count = 0;
        byte[] asciiBytes = Encoding.ASCII.GetBytes(text);
        byte[] result = new byte[4 + text.Length];
        for (count = 0; count < text.Length; count++)
        {
            result[result.Length - count - 1] = (byte)(asciiBytes[asciiBytes.Length - count - 1] ^ (key & 0xFF));
            key += 0x22AF;
            key &= 0xFFFF;
            key = RotateLeft(key, 1);
            key &= 0xFFFF;
        }
 
        count = count ^ key;
        result[3] = (byte)(count & 0xFF);
 
        key += 0x22AF;
        key &= 0xFFFF;
        key = RotateLeft(key, 1);
        key &= 0xFFFF;
 
        result[2] = (byte)(key & 0xFF);
 
        key += 0x22AF;
        key &= 0xFFFF;
        key = RotateLeft(key, 1);
        key &= 0xFFFF;
 
        result[1] = (byte)(key & 0xFF);
        result[0] = (byte)((key >> 8) & 0xFF);
 
        return result;
    }
```
 
### URL Decoding
Source : https://bitbucket.org/Ioncannon/project-meteor-server/src/master/Launcher%20Editor/Program.cs
```csharp
 public string FFXIVLoginStringDecode(byte[] data)
    {
        Console.OutputEncoding = System.Text.Encoding.UTF8;
        while (true)
        {
            string result = "";
            uint key = (uint)data[0] << 8 | data[1];
            uint key2 = data[2];
            key = RotateRight(key, 1) & 0xFFFF;
            key -= 0x22AF;
            key &= 0xFFFF;
            key2 = key2 ^ key;
            key = RotateRight(key, 1) & 0xFFFF;
            key -= 0x22AF;
            key &= 0xFFFF;
            uint finalKey = key;
            key = data[3];
            uint count = (key2 & 0xFF) << 8;
            key = key ^ finalKey;
            key &= 0xFF;
            count |= key;
 
            int count2 = 0;
            while (count != 0)
            {
                uint encrypted = data[4 + count2];
                finalKey = RotateRight(finalKey, 1) & 0xFFFF;
                finalKey -= 0x22AF;
                finalKey &= 0xFFFF;
                encrypted = encrypted ^ (finalKey & 0xFF);
 
                result += (char)encrypted;
                count--;
                count2++;
            }
 
            return result;
        }
    }
```
## Header
 
After the Player enters the username&password and get's authenticated, the Login server will return the following header
 
```html
<x-sqexauth sid="" lang="" region="" utc="" />
```
* sid: The sessionID of the logged-in user
* lang: The default language of the user
* region: The region of the Player's Square Enix Account
* utc: The key used for the Blowfish
 
once the "ffxivlogin.exe" detects the header in the page , it will construct the following string
```cpp
T =%d /LANG =%s /REGION =%d /SERVER_UTC =%s /SESSION_ID =%s
```
* T: current tick count
The rest is retrieved from the header
 
The string then will be Encrypted with Blowfish after that converted to Base64. After the conversion, it replaces the following from the base64 string
 
```csharp
String.Replace('+','-');
String.Replace('/','_');
```
 
Then ffxivlogin will launch the game with the following argument.
 
```
{workingdirectory}\ffxivgame.exe sqex0002{base64string}////
```


