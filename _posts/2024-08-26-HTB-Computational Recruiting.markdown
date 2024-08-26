---
layout: post
title:  "HTB: Computational Recruiting Challenge"
date:   2024-08-26 13:08:01 -0400
categories: CTF
---

Computational Recruiting is a "very easy" challenge on Hack the Box that focuses on solving a problem with python. We are given a dataset that we need to apply a formula to, and then sort. Nothing too crazy.
 
CHALLENGE DESCRIPTION

Not too long ago, your cyborg detective friend John Love told you he heard some strange rumours from some folks in the Establishment that he's searching into. They talked about the possible discovery of a new vault, vault 79, which might hold a big reserve of gold. Hearing of these news, youband your fellow compatriots slowly realized that with that gold reserver you could accomplish your dreams of reviving the currency of old times, and help modern civilization flourish once more. Looking at the potential location of the vault however, you begin to understand that this will be no easy task. Your team by itself is not enough. You will need some new recruitments. Now, standing in the center of Gigatron, talking and inspiring potential recruits, you have collected a big list of candidates based on skills you believe are needed for this quest. How can you decide however which ones are truly worthy of joining you?

Visiting the IP given gives us a little additional information, including the weights and formula to apply for the overall score. 

![image](https://github.com/user-attachments/assets/f0a701d4-8879-4731-8e3f-affff2f8ed72)

And looking at the dataset we are given, it appears to be column headers followed by names and numerical values. 

![image](https://github.com/user-attachments/assets/d2354521-72eb-41ad-aae6-820899629d86)

So we start by writing out the formulas
![image](https://github.com/user-attachments/assets/991edc32-0659-430f-87e5-7ba33c0a8610)

Then delete the headers and footer and split based on whitespace, assigning the names to the appropriate values. 
![image](https://github.com/user-attachments/assets/3b9fb52f-6128-44de-9f51-192f0a7491c9)

Loop and sort

![image](https://github.com/user-attachments/assets/f50b1e24-17cd-4c99-8721-4a1d73e51f72)

And print the results

![image](https://github.com/user-attachments/assets/a9117a37-df0e-483f-9b01-0f4d23b11831)

Before formatting as requested, and shooting over via nc

{% highlight bash %} echo "Jayson Enderby - 98, Malva Shreeve - 96, Randolf Raybould - 96, Shay Sheardown - 95, Koo Rue - 94, Tabina Nathon - 94, Taber Haile - 93, Constanta Rolfs - 93, Corette Bursnell - 93, Gerri Bielfelt - 92, Andy Swane - 91, Colene Vanyatin - 91, Lowe Farnan - 91, Ashlin Neely - 91" | nc 83.136.255.40 34340 {% endhighlight%}

and boom.
![image](https://github.com/user-attachments/assets/ca787e96-5a51-4337-8fc1-66e6757ea2e9)
