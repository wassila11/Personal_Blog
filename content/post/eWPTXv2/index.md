---
title: Yogosha Christmas CTF 2023
# Summary for listings and search engines
summary: Web Challenges Writeup 
# Link this post with a project
# projects: []

# Date published
date: "2024-01-01T00:00:00Z"

# Date updated
#lastmod: "2020-12-13T00:00:00Z"

# Is this an unpublished draft?
#draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
 
 
 # placement: 2
  #preview_only: false

authors:
- Wassila Chtioui

tags:
- Web Exploitation
- Penetration test

categories:
- Web
- eWPTXv2
- ine

---

### Introduction

The eWPTXv2 *(eLearnSecurity Web Application Penetration Tester eXtreme)* certification is an advanced certification in web application 
penetration testing. It's basically a demonstration of a black box penetration test where you can asses your skills in enumeration,
vulnerability assessment, filters bypass and report generation.

I hope this article can provide you with some useful pointers to kickstart your journey in succeeded your eWPTXv2 certificate, and get familiar with some of the techniques needed.

### Overview
You have two weeks period to pass the exam during which only one week where the lab is accessible. So basically, you need to take as much screenshots and notes to write your report in the second week. 

Your goal is to read a file from an internal server and achieve an RCE on the two localhost servers running on one of the assets. 

Of course only achieving this goal, won't allow you to succeed the exam. The lab has plenty of other vulnerabilities that you have to find and mention in your report. 

Some exploits or commands may not work on the first attempt sometimes, so anytime this happens and you're sure about your payload, just reset the lab and try again. *(there is a daily restart limit btw)*


### Certification Exam Insights and Review

In my opinion, the exam scenario was quite fun and interesting. You get to assess your skills in building custom exploits and bypassing Web Application Firewalls (WAF). There was various vulnerabilities to exploit and some of them were tricky. In fact, It's similar to a real world pentest so it's a great opportunity to see how this looks like in reality and learn what you should and shouldn't do, despite the occasional instability in the lab environment. For Mac users, I recommend installing Tunnelblick to ensure seamless VPN functionality. Let me highlight here, that enumeration is key.

In addition, I learnt to think out of the box and to have a strategic plan of how to identify vulnerabilities. Closely observing the server response and behavior is crucial. I noticed that I need to read more blogs and practice more real life scenarios to improve my methodology of how to check for flaws and better identify the attack surface. 

All in all, it's a great opportunity to put your skills into test and definitely worth giving it a shot. 


### Roadmap to Exam Success: Strategic Resource Selection

Now, how can you build your way to succeeding the exam ? 

Maybe you already know the resources I'll list, but this is just to highlight how important these resources are and to tell you that it's about quality not quantity. Focus on targeted resources rather than overwhelming yourself with countless options.

Tailor your study plan to your learning style whether it's structured schedules and note-taking or a more flexible approach, yet most importantly ensure having a continuous progress.

Start by building a solid foundation in web application vulnerabilities and practice them as much as you can through labs. Here are some recommended resources:

* Portswigger:  If you're still a beginner and still learning about web app vulnerabilities or if you would like to tackle a specific topic for the first time then this is a very great resource for you. You can find labs ranging from friendly to expert along with very explained course to read. 

* Hackthebox: This has more advanced labs but definitely worth checking. The web challenges ideas are very interesting and you can practice the machines as well. Why I would suggest HTB machines, is because you'll need enumeration skills during the exam and this is where you can practice them. Also it may be beneficial to save some of the payloads you use.

* Pentesterlab: This is a very good resource too. It's a little bit similar to Portswigger but with more advanced stuff. I would suggest this if you're already familiar with web vulnerabilities and you're looking for the push to get you to the next step. 

* Capture The Flag contests: Why CTFs are quite important here although the exam is not a CTF style at all and should think of it as a penetration test. Generally, in security platform, you know what vulnerability you're focusing on, but in real life cases (and in the exam), you have to identify the vulnerability you're going to exploit. CTFs are where you are able to learn this skill. This case is also the same for HTB web challenges. 

Which resources are best, it depends on the person that's why I listed all the resources I find interesting. For me, I didn't bye the INE course, and when I started the preparation I already had the basic knowledge for web app vulnerabilities (I always practiced through Portwsigger). So what I did was to read reviews about the certificate, read various penetration testing reports/CTF writeups to grasp the methodology used and practice through CTFs as well as HTB machines and web challenges. 

Identify your position, what topics you should practice more, make sure you have the basics and then check more advanced stuff according to your pace and need. 

Maybe once you have covered all basic web vulnerabilities and you feel confident playing labs then I would say that you're ready by then. 

The duration of a week is sufficient to achieve the necessary requirements to succeed so take your time. It took me 3 days to achieve the first goal of the exam, thus don't panic if you started and still didn't find anything, just keep going and note anything you covered along the way as these are as important as achieving the exam goals.

For a better organization, I would suggest to take screenshots and write a note (specifying the vulnerable endpoint and the payload used) once you identify a vulnerability, so you don't forget to do this afterward in case you got caught up into the hacking excitement and especially since the lab is not available during the second week. Include everything you find in your report, don't miss out anything. 


### Conclusion
I hope that I've covered all essential aspects in this blog post, but in case I've overlooked anything or if you have additional questions, please feel free to contact me, I'll be more than happy to assist.


![](https://i.imgur.com/5zxZRAC.png)

