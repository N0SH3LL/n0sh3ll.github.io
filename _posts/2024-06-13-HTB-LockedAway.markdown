---
layout: post
title:  "HTB: Locked Away Challenge"
date:   2024-06-13 13:08:01 -0400
categories: CTF
---


Locked Away is a "very easy" challenge on Hack the Box. I've been tackling some of these in the mornings as I drink coffee and try to get the brain to warm up. This challenge is pretty straightforward, we get to the guess-and-check piece pretty quickly, and the answer isn't any great twist. Basically, we have a server accepting python commands and we have to somehow call a function without triggering the blacklist items. 

CHALLENGE DESCRIPTION

A test! Getting onto the team is one thing, but you must prove your skills to be chosen to represent the best of the best. They have given you the classic - a restricted environment, devoid of functionality, and it is up to you to see what you can do. Can you break open the chest? Do you have what it takes to bring humanity from the brink?


## Enumeration

Visiting the IP given to us shows us some ASCII art and some text. There isn't any javascript being run on the page, and there isn't a way to pass commands directly throught the webpage, so we have to look at the files given as part of the challenge. 

![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/6317aa00-9e40-41c2-9c1f-541bc418fa98)

We've got a dockerfile, a build script for the file, and a python file.

![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/7ccbe44a-287d-4225-a306-5fd85f2da9e5)
![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/6baf9a80-158d-4ac3-a540-59ff6e6ae627)

Inside the dockerfile we see the challenge being set up. 

![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/237f7e49-b1f3-47e4-93a8-c1faabbe89c9)

{% highlight bash %}
ADD --chown=ctf challenge/* /home/ctf/
{% endhighlight%}

{% highlight bash %}
ENTRYPOINT ["socat", "TCP-LISTEN:1337,fork,reuseaddr", "EXEC:'python3 main.py'"]
{% endhighlight%}

Tells us that the the challenge folder is added to the container, and that main.py is executed. 

![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/1be6357b-ef97-4262-9bd2-a542471a676d)

Main.py looks awfully familiar. It's got the same banner and printed message as the web page. I think its safe to assume at this point that the box is running the docker container and python file we were given. The logic in the file is fairly straightforward: There is one function that accepts an input, checks for any values in the blacklist, and either blocks or executes the command. There is also an uncalled function at the top that opens flag.txt. So we have to pass a command that does not trigger the blacklist, but still either triggers open_chest, or lets us read flag.txt. 

Some tricks that immediately came to mind were- 

-passing an encoded command to be decoded
-passing a command to alter the blacklist itself
-access elements of the blacklist itself (it's an array) to build the command I can't type directly
-passing commands to the container itslef to just read the flag instead of triggering the function
-concatenate different pieces together to assemble a full command

But first, we have to find out how to pass the commands to the box. 

{% highlight bash %}
nc 94.237.49.178 46920
{% endhighlight%}

That was easy. 

In short order, I tested the above ideas and quickly found that the blacklist was annoyingly well thought out. Import, os, sys really limit our options. And even quotation marks and brackets are off limits. I got the following results- 

-passing an encoded command to be decoded > Lacking import prevented standard base64 encoding, and most encoding commands require quotes somewhere.
-passing a command to alter the blacklist itself > Couldn't really interface with the array without square brackets. 
-access elements of the blacklist itself (it's an array) to build the command I can't type directly > Same issue. I can't do anything like blacklist[1] because of the bracket limitation. 
-passing commands to the container itself to just read the flag instead of triggering the function > Importing and os/sys limitations prevent the standard ways I know to trigger os commands.
-concatenate different pieces together to assemble a full command > Again, brackets and quotes prevent the standard layout of most of these commands. 

I feel like the answer is some python stuff I'm not enough of a programmer to understand. 

After some wandering through google, I came to this clue: 
https://www.kdnuggets.com/2023/03/introduction-getitem-magic-method-python.html

The syntax for the __getitem__ method is as follows:

![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/31de20e7-23c3-4bd8-b33b-3386dac5d780)


So, I would expect blacklist.__getitem__(1) to return "import", so we've gotten past some of the bracket issues. 
I then googled "how to list all funcitons in a python file" and found the wonderful dir command, which returns an array of objects in the file. So I added the line print(dir()), found that open_chest was the twelth item, and carefully crafted the following command: 

{% highlight bash %}
dir().__getitem__(11)
{% endhighlight%}

Brilliant right? I even remembered python starts counting from 0. 

.....Except dir is on the list too. 
Hopefully this means we are on the right track though. 

Some further searches told me that var() is similar, and printing it gives us the following if we modify and run the python file we have: 

![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/a56c7a62-d54b-4bf2-8ab0-fe219fa0760e)

And we can see the function we need as part of a key pair. 
So now we need to use the key open_chest and call the function. 

print(vars().__getitem__(open_chest)()) successfully uses the key and returns us the function, but of course triggers the blacklist. I tried multiple ways to get it by position, but to no avail. So we need an alternate way to construct the key so that getitem accepts it. We still can't decode, but the time I initally spent trying to obscure commands earlier paid off. Python has the chr(number) function, which represents an ASCII character, based on the number passed to it. 

![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/2e02e75c-2bb6-4382-92d7-822276d1ba63)

So, I need to translate open_chest to these values, concatenate, and hope it works. 

{% highlight bash %}
vars().__getitem__(chr(111)+chr(112)+chr(101)+chr(110)+chr(95)+chr(99)+chr(104)+chr(101)+chr(115)+chr(116))()
{% endhighlight%}

And we have liftoff. 

![image](https://github.com/N0SH3LL/n0sh3ll.github.io/assets/107323047/ba4e1344-1320-42d1-b27d-26f0dc226f2c)

