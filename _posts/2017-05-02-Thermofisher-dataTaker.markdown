---
layout: post
title:  "Thermofisher dataTaker - Insecure by Design"
date:   2017-05-02 11:00:00 +11005
categories: exposure industrial
---

### Introduction

Thermofisher dataTaker DT8x devices offer little security, and clear text configuration visible to users without any log in process.  
The products website: http://www.datataker.com/
  
  
### Vulnerable Versions  

While only DT8x devices have been inspected, it would be unlikely that other models are not also affected by this design issue.  
Only Firmware 1.72.007 was observed in the wild.
  
  
### Details

This is a very short stub of of the issue.
There has been NO testing done on these devices, only a quick overview of the device and their state in the wild.

**Popularity**  
At the time of this posting, according to Shodan, there are at least 331 of these devices publically exposed on the internet.  
A small sample of 16 devices were inspected, and only 2 prompted for a login.  

![img]({{ site.github.url }}/images/tf-shodan.png){:class="img-responsive"}

**Risks**  
These devices are fully configurable, where programs can be written in them, using the web interface.  
Schedules can be created, to send emails, connect to servers, and alike.  
Many devices are configured to send emails.  Some even connect to ftp servers for uploading data.  
All config is removed in this screenshot for privacy reasons.  
![img]({{ site.github.url }}/images/tf-config.png){:class="img-responsive"}
  
Due to the clear text, unauthenticated manner of accessing these devices, and the configuration method, this exposes usernames and passwords of the owners system.  
For example, to log to ftp, the following command is stored in the config.  
`DO{copyd dest="ftp://username:password@1.2.3.4:21/folder/file(timestamp).csv?priority=NORMAL&interface=MODEM" format=CSV sched=A data=Y alarms=N start=-1T}`  

In addition, when configuring the devices, there is the ability to connect directly to the device, obviously needing a username and password.  
Because it is possible to view this configuration without logging in, the username and password can be gathered.  
![img]({{ site.github.url }}/images/tf-user.png){:class="img-responsive"}  


**Bad Advice from Manufacturer**  
With any IoT style device, manufacturers should be doing their best to advise users how to use them securely.  
Thermofisher are doing the exact opposite, reccomending users to port forward access to these devices.  
![img]({{ site.github.url }}/images/tf-rec.png){:class="img-responsive"}  
This is sourced from an Application note on the datataker website.  
http://www.datataker.com/documents/communications/application-notes/AN-0023-A0%20-%20Energy%20Resource%20Monitoring.pdf  

  
### Mitigation  

Get these devices OFF the internet, they are not secure, and clearly not designed to ever be accessed directly over the internet.   
  
This is not unusual for industrial equipment, which is rarely built with security in mind.  
  
If accessing these remotely is required, a secure encrypted VPN should need used for access.  
  
As these devices do not contain any sign of supporting UPNP, the only way these devices could be exposed is by users intentially exposing them.  

## Update  

As of 2017-05-21, no one has responded to my warnings.  
The manufacturer has completely ignored all forms of contact.  

There were several users also warned, while none have responded, the affected devices has been reduced from an estimated 331 devices to 214 globally.  

While it is good to see the numbers reduced, it is a disguisting attitude from the manufacturer.

I have contacted some of the warned companies to see if Thermofisher have contacted users in private.
