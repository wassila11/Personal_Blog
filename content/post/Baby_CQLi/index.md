---
title: Securinets ISI Mini-CTF 2023
# Summary for listings and search engines
summary: Medium/Hard Web Challenges Writeup 
# Link this post with a project
# projects: []

# Date published
date: "2023-12-13T00:00:00Z"

# Date updated
#lastmod: "2023-12-13T00:00:00Z"

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
- SQLi
- RCE
- Sqlite3

---
# Securinets ISI Mini-CTF

### Introduction 
Securinets is a Tunisian university cybersecurity association, founded in INSAT (National Institute of Applied Sciences and Technology) and since 
there was a branch in my university ISI (Higher Institute of Computer Science), I got invited to participate as an author for the web challenges. 
I wrote one hard and one medium challenge and I decided to share their write-ups in my blog. 

### Baby_CQLi - 500 points - Hard (2 solves)
This challenge was basically a chain of vulnerabilities leading to RCE. You start by a simple IDOR. 

> IDOR (Insecure Direct Object Reference) is a web application security
> flaw where attackers can access unauthorized data by manipulating
> direct references to internal objects like files or database entries.
> For instance, they might change a URL parameter from 'user_id=123' to
> 'user_id=124' in an attempt to access another user's data.

Hitting on the Play Minions Game button you get redirected to this path http://localhost:1234/1/minions-game.php having a 404 error page. Noticing 
the parameters here, you change it to http://localhost:1234/0/minions-game.php and here you go, the fun is about to begin ! 



![alt text](https://i.imgur.com/SuNoLh8.png)


The interface had a jump game similar to the Google Chrome offline dino game. It had the following features:

- You start by hitting on a space.
- When game is over:
	-  You are prompted to enter your username which is then stored in localStorage.
	- A POST request to save your score along with the associated entered username is sent. 

- A button to view your previous scores using a GET request and taking the username as a parameter.

There was also the source code for the page value-score.php that fetches the previous scores. Let's understand the code first. So since there was 
some sort of interaction with the database through storing the score data, we can start by looking for how is the query being handled. 

![alt text](https://i.imgur.com/cGX2o8X.png)


The username is being concatenated to a SELECT stamement and then the query is executed using a custom function called queryLite instead of 
utilizing the built-in functions. hmm.. sounds fishy, right ? :wink:

Lets check this function and try understand it line by line.

![alt text](https://i.imgur.com/stqgRU1.png)

This line here defines a shell command that is running sqlite3 command on the database minions.db
> $command = 'sqlite3 '  .  escapeshellarg($dbPath);

Then we are utilizing the proc_open built in function to execute this command on the system.

`$descriptorspec` sets up file descriptors for input, output, and error handling which are pointed by the `$pipes` argument. For a more 
understanding of this function you can check this [Link](https://www.php.net/manual/fr/function.proc-open.php).


And we are passing here the sql query as the input which is the argument of the sqlite3 command. 

> if (is_resource($process)) {
> 
>fwrite($pipes[0], $sql);
>
> fclose($pipes[0]);

The result of this query is saved to $output which is then parsed and displayed.

What we know so far: 
- There is an SQLi in the SELECT statement.
- The query is executed by a command line: `sqlite3`
- The flag is in the filesystem. *(from the description)*

So taking advantage of the query being executed by a shell command line, we need to get an RCE to access the flag.


![alt text](https://i.imgur.com/zkvQTod.png)

Simply googling sqlite3 shell, shows us the [documentation](https://www.php.net/manual/fr/function.proc-open.php) of command line shell of 
sqlite3. Scrolling through the page, we notice some special dot-commands, and here we can find `.shell` and `.system` which allow us to execute 
shell on the system machine aaaand boom! Here's our foothold. 

Having this in mind, we can try testing this out locally in order to understand how to link what we already have with the challenge. 

![alt text](https://i.imgur.com/1gji1LZ.png)

We simulate how the query is being executed and try out .shell command in order to list files in the current directory. 
We can conclude that by concatenating a **\n** to start a new line after the select statement we can successfully execute the .shell command. And 
to balance the query we can add a `select "` to handle the unclosing quotes error or simply using `/*` to comment the rest.

Now trying to find the path to the flag, we find that our input is being blocked. 
Lets get back to the code:

![alt text](https://i.imgur.com/X7OIDXw.png)

We see here that both the input and output are being compared to a bunch of backlisted words. In this case, to use the command `ls` we can for 
example insert  `$@` between characters to bypass the filter. 

![alt text](https://i.imgur.com/EvL8Ytd.png)

The .secret file is indeed the flag file. Trying to cat it, since it contains the word flag and the output is being filtered, we have to find a 
way to bypass this check too.

In this case we have two options: either to encode the output and have it displayed 

![alt text](https://i.imgur.com/H4ie68E.png)

or taking advantage of the fact that the stderr is printed out whenever there's an sql error.

> if ($returnValue !== 0) { throw  new  Exception("Error executing
> SQLite command: $errorOutput"); }

![alt text](https://i.imgur.com/lVAudBJ.png)


Unintended solutions: 

Sooo as I forgot to add an exit after filtering the input, the application contunied to run the rest of the code and was not stopping when 
detecting the blacklisted words. Some players could bypass the blacklist by having the application display the error message and then executing 
their payload x). gj fellas, you got me haha. 

I'll correct this one and upload the Dockerfile on GitHub so everyone can enjoy the challenge.  

### FruitDB - 500 points - Hard (0 solves)
The players didn't find enough time to try out this challenge properly, but I'm pretty sure that they would've solved it.

Thus I won't be providing a write-up at this moment to allow them the opportunity to attempt it first. I'll be sharing the Dockerfile for it on 
GitHub too so feel free to share the solution with me if you manage to solve it.

