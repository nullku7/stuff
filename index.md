# Dahua NVR - Multiple Exposures

### Introduction

My little peek at the new Dahua NVR software, which is the latest as of this post.
Responsible Disclosure

- 2016-10-10: Sent to both the Australian Importer and Dahua, everything mentioned here plus more.  The Importer was also called to discuss the issues.
- 2016-10-17 (approx): Followed up verbally with the importer a couple weeks later (phone)
- 2016-11-14: Both the Importer and Dahua for an update.
- Apart from when I rang the Importer directly, no one has responded.
- 2017-02-24: Public Disclosure

### Changes since First Reporting on 2016-10-10

The camera's now force a password to be set, which to be honest, does nothing!
Older CVE's worth noting

CVE-2013-6117 is still worth reading to understand the port 37777 service.  While it's vulnerabilities are no longer directly exploitable, the protocol is unchanged for the most part.
https://depthsecurity.com/blog/dahua-dvr-authentication-bypass-cve-2013-6117

### The Hardware

- Model: DHI-HCVR7216A-S3
- Firmware: ????
- SmartPSS Software (v 1.16.1 Build Date 2017-01-19)

### Camera Names

The first interesting thing when loading the webpage is a cap.js file.
While this gives us nothing yet, we shall take log of what it has.

```js
var talkTypes='2&1&4&';
var devType='DHI-HCVR7216A-S3';

var userInfo='This is user info!';
var streamCap=19;
var channelNames='Shop&&Shop&&Warehouse&&Entry Gate&&<<<redacted>>>';
var rtspport = 554;
var ClientType = 0;
var capTcpPort = 37777;
var radius = false;
```

So we know the names of the camera's, but not much more yet.

> The entire DVR/NVR interface is already loaded on the login screen

### Obtaining Camera Passwords

As it turns out, the Web Page, Mobile Application, and Desktop Application are all nice enough to transfer all camera details in clear text.

The following TCP Stream contains ALL camera's, their IP's, and Username/Password

These are the camera's that Dahua now Force a password change on!
This is a huge risk as the camera's are usually isolated via the NVR from the main network, meaning accessing them required using the NVR as a pivot.
Due to most installers sharing a common password for the entire system, the chance the admin password for the NVR would be revealed would be very high.

Screen Shot 2017-02-24 at 10.26.33 am.png Captured Usernames and Password

As can be seen from this Captured TCP Stream, the client software requests all of the camera channels, and the NVR responds with this.

We have the the Requested DisChn
This alligns with with the Cammera Channels from cap.js.

So we now have.
- Camera Name
- Camera IP
- Camera Username
- Camera Password
- Camera Type

 
 
### New Users

Whenever a new user (of any level permission) is added, the username and password is transmitted in clear text

Screen Shot 2017-02-24 at 10.39.14 am.png Username: test   Password: NotSecure

This packet  a6 00 00 00 17 00 00 00 06 is surprisingly identical to that identified in cve-2013-6117
If that's the case, 17 is a checksum
06 is what I seen on a new user, while 0a is identified in the above cve for changing passwords.

23 char = 0x17 ... looks good.

> Upon further investigation of  CVE-2013-6117, the entire protocol is identical.  This results in another exposure, the ability to capture reset passwords, not only the new one, but the old as well.

### Injecting a new user

So testing the known CVE-2013-6117, as well as writing my own, does not work.  The NVR closes the port the moment the packet is sent, it must be blocking the connection.

At this stage, it appears each TCP Connect needs to authenticate, then the binary protocol should work exactly as it used to.  This means that the authentication bypass methods should no longer work.

> Anyone who wants to assist in solving this, please feel free to contact me.
> I will spend more time on this

### Auto Login despite NO login

When launching SmartPSS Software (v 1.16.1 Build Date 2017-01-19)
It logs into the NVR before any user interaction with the software.
Upon logging in, the above camera information is divulged, as well as the Admin Hash.
This alone may be enough information to gain access, if not, we need to go deeper.
Hashes

Below is the hash captured when the software is launched.

Screen Shot 2017-02-24 at 2.37.47 pm.png

This hash alone is not that useful, but everything needed to make it useful is presented just before.

Screen Shot 2017-02-24 at 2.39.58 pm.png

So with this we have
- Account Name: admin
- Realm: Login to 2FO3611PAEG9265
- Random: 1331847305
- Account Hash: B43EA4316B4B391FD34B6A85E687B628E3FE7A9CBF8405F1452EF497E53899E3

The method of calculating this hash is unknown for Port 37777 Binary Protocol
It appears to be SHA256, which is not a known Dahua Hash, and it not present in the web interface JavaScript.

For the Web it is simply this

part 1: MD5(username:realm:password)
MD5(username:random:part1)

Older Dahua had an infamous 48bit algorithm

Looking at source code, it seems come even only did a base64encode

This is a great improvement over the past, but unfortunately, you can still sign in using the MD5 hash via the web interface.'

Below is the JS which creates the Hash for the web interface, none of these appear to be valid.

```js
function getAuthByType(a, b, c, d, e, f) {
switch (d) {
case "Basic":
return Base64.encode(a + ":" + b);
case "OldDigest":
return superEncipherment(b);
case "Default":
if (c)
return hex_md5(a + ":" + e + ":" + b);
var g = hex_md5(a + ":" + e + ":" + b);
return hex_md5(a + ":" + f + ":" + g);
default:
return b
}
}
function superEncipherment(a) {
for (var b = "", c = [], d = hex_md5(a), e = [], f = 0; 16 > f && f < d.length / 2; f++) e[f] = parseInt(d.slice(2 * f, 2 * f + 2), 16); for (var f = 0; 7 >= f; f++)
c[f] = (e[2 * f] + e[2 * f + 1]) % 62,
c[f] += c[f] >= 0 && c[f] <= 9 ? 48 : c[f] >= 10 && c[f] <= 35 ? 55 : 61,
b += String.fromCharCode(c[f]);
return b
}
```
