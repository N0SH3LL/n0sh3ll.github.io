---
layout: post
title:  "HTB: Locked Away Challenge"
date:   2024-05-01 13:08:01 -0400
categories: CTF
---


Locked Away is a "very easy" challenge on Hack the Box. I've been tackling some of these in the mornings as I drink coffee and try to get the brain to warm up. This challenge is pretty straightforward, we get to the guess-and-check piece pretty quickly, and the answer isn't any great twist. Basically, we have a server accepting python commands and we have to somehow call a function without triggering the blacklist items. 

CHALLENGE DESCRIPTION

A test! Getting onto the team is one thing, but you must prove your skills to be chosen to represent the best of the best. They have given you the classic - a restricted environment, devoid of functionality, and it is up to you to see what you can do. Can you break open the chest? Do you have what it takes to bring humanity from the brink?


## Enumeration

Visiting the IP given to us shows us some ASCII art and some text. There isn't any javascript being run on the page, and there isn't a way to pass commands directly throught the webpage, so we have to look at the files given as part of the challenge. We've got a dockerfile, a build script for the file, and a python file.

Inside the dockerfile we see the challenge being set up. 

{% highlight bash %}
ADD --chown=ctf challenge/* /home/ctf/
{% endhighlight%}

{% highlight bash %}
ENTRYPOINT ["socat", "TCP-LISTEN:1337,fork,reuseaddr", "EXEC:'python3 main.py'"]
{% endhighlight%}

Tells us that the the challenge folder is added to the container, and the main.py is executed. 

Main.py looks awfully familiar. It's got the same banner and printed message as the web page. I think its safe to assume at this point that the box is running the docker container and python file we were given. The logic in the file is fairly straightforward: There is one function that accepts an input, checks for any values in the blacklist, and either blocks or executes the command. There is also an uncalled function at the top that opens flag.txt. So we have to pass a command that does not trigger the blacklist, but still either triggers open_chest, or lets us read flag.txt. 

Some tricks that immediately came to mind were- 

-passing an encoded command to be decoded
-passing a command to alter the blacklist itself
-access elements of the blacklist itself (it's an array) to build the command I can't type directly
-passing commands to the container itslef to just read the flag instead of triggering the function
-concatenate different pieces together to assemble a full command

![screenshot](/assets/images/Usage/nmap%20results.png)
