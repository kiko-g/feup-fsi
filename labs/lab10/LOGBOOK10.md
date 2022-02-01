# Cross-Site Scripting (XSS) Attack Lab
## Logbook Week 10

This document describes the conclusions made after completing the Seed Labs tutorial on Cross-Site Scripting (XSS).

The following is an academic analysis made by students of master in Computing Engineering at the University of Porto.

The document follows the suggested division of the Seed Lab tutorial.

### Web Security and XSS

The internet is a hostile environment. It was designed to allow everybody to reach everybody. This philosophy has the advantage of connecting the world in a way ancestors never thought could be possible. 

The most used service of this messy and complex system is the web protocol. The web is already built on top of a network where security is not a stronghold, but it extends this issue to a higher level since it is a client-server architecture, where clients are required to execute code coming from unknown servers on their local machines with their local privileges.

To mitigate the threat the Web architecture creates itself, the software where clients execute this protocol, known as browsers have become a complex and efficient tool of sandboxing. Their philosophy is correct, we will never be able to control the quality of content following in the network, for that reason we should always apply strict rules of isolation, and if the content proves itself valid and legit, increase the number of resources it can use.

There are many types of attacks typical of the web, many of them were already antiquated by countermeasures enforced in modern browsers, but others like XSS keep hard to fix it completely. A silver bullet was not yet found.

[According to OWASP](https://owasp.org/www-community/attacks/xss/), an XSS attack is:

* Cross-Site Scripting (XSS) attacks are a type of injection, in which malicious scripts are injected into otherwise benign and trusted websites. XSS attacks occur when an attacker uses a web application to send malicious code, generally in the form of a browser side script, to a different end-user. 

* Flaws that allow these attacks to succeed are quite widespread and occur anywhere a web application uses input from a user within the output it generates without validating or encoding it.

### Types of XSS

There are two types of XSS, both compose an injection of code, typically in the context of the web is Javascript.

* Reflected XSS: When a client loads a script from the web, it is executed in its local machine with its privileges. When a client connects to a malicious server, it loads some lines of malicious code in the middle of thousands of valid content, that in the background are executing a request for a valid third party, for example, the client's bank to steal money from that agent.

* Stored XSS: A malicious agent can inject malicious code in the database of a server that, when it is fetched will be executed in a client.

**To sum up:** 

* XSS is an injection of code, usually of Javascript to execute malicious code silences to dump any type of valuable information.
* These attacks could theoretically be avoided if the input from third parties were always sanitized.
* There are unlimited forms of creating a malicious string, for that reason, there is no sanitization that performs as a silver bullet.

### Samy Worm

This Seedlab was inspired by the Samy Worm incident that attacked MySpace Users in 2005.
This attack was an XSS of the type stored.

MySpace didn't perform a proper sanitization of a high-risk input that allowed users to inject web code to style their webpage. Samy was able to inject a javascript code that was executed and propagated after users visit Samy profile. 

### Environment Setup

Seed lab week 10 set up a docker environment compressing a web server and a MySQL database that supports the information stored in that platform.

This web application is an open-source social network named Elgg where some countermeasures were relaxed to be able to replicate the Samy Worm of 2005 in MySpace.

It is suggested to install a tool that allows an easy inspection of the HTTP requests made in the background while browsing the Firefox Browser. **HTTP Header Live**


### Task 1: Posting a Malicious Message to Display an Alert Window

The handout tells us to inject this inline javascript code in the brief description field under the profile edition. If the input is not properly sanitized

```javascript
<script>alert(’XSS’);</script>
```

The result was the javascript alert appearing in the browser frame. This means we can use a brief description to perform XSS stored attacks.

### Task 2: Posting a Malicious Message to Display Cookies

In task 1 we simply injected a line of code that does nothing harmful, simply display the 'XSS' in a Javascript alert. Nevertheless, we are facing a vulnerability that can be exploited for bad.

In task 2 we want to increase the severity of the attacks using a brief description of input vulnerability. We will output in an alert the cookie variable content in the javascript alert

By changing the input of task 1 to the following line:
```javascript
<script>alert(document.cookie);</script>
```

The output was the following:
<div style="text-align:center">
    
![](https://i.imgur.com/P7fS4jt.png)

</div>

### Task 3: Stealing Cookies from the Victim’s Machine

One common attack described extensively in the theoretical class, was the use of XSS to trigger a connection to the browser on the client-side that will be interpreted as intended by the user, and therefore will execute with the client privileges, namely it will send the cookies associated with the destination website.

Then by performing usually eavesdropping on the network, in this case by targeting the request towards a malicious Socket connection with the cookie as payload. The attacker can collect sensitive information about the user.

The simplicity of this attack required platforms to enforce measures against the privileges somebody with a session cookie could access, since easily, as we saw, it can be stolen.


### Task 4: Becoming the Victim’s Friend


The goal of this task is the exploitation of XSS vulnerability to imitate part of behaviour that was presented in the already described Sandy Worm. When somebody visits the profile of the user Sandy of Elgg, that person becomes automatically a friend of Sandy.

#### Understand How Friend Requests are Made

To understand how we can handle this task, Seedlab tells us to first verify how friend requests are made. What is the URL and payload sent to a user during a friend request? **To achieve this goal we made use of HTTP Header Live**.

After opening HTTP Header Live, we sent a friend request from Alice towards Charlie. The URL that constituted this request was:

<div style="text-align:center">
    
![](https://i.imgur.com/HOIePD2.png)

</div>

A HTTP GET request for 

* Path : **/action/friends/add**
* Query String:
    * friend=friend_id
    * elgg_ts(timestamp)=timestamp
    * elgg_token=cookie


The timestamp and the token which is the session cookie provided by the Elgg server upon login are replicated twice to perform a syntactically well-formed friend request in this application.

#### Exploit

To trigger a GET request dynamically, we will inject code that could perform an Ajax request to correct the route. 

By performing a previous friend request to Samy we determined that **Samy id was 59**. With this information and using the example code provided already by Seedlabs, we reached out to the following code.

We added an extra alert() so that it was easier to detect the code was taking effect.

<div style="text-align:center">
    
![](https://i.imgur.com/ka29hoz.png)

</div>

With the payload crafted, we then introduced it as suggested in the handout in the about me field on Samy profile. It was necessary to select the option HTML mode before submitting the payload to it to take effect(See question 2 below).

After that we open Alice account, we went to Samy profile and the effect was the following.
<div style="text-align:center">
    
![](https://i.imgur.com/9RyDexF.png)

</div>


Question Answering:

* Question 1: Line 1 and 2 are required to perform a well-formed friend request. Before crafting the payload we made a manual friend request to understand what the syntax was, and it demanded inserting the pair, timestamp-token, twice in the request towards the server. Otherwise, the request was malformed and the server would ignore it. Lines 1 and 2 read those required values that are private and unique between users(and for that reason cannot be hardcoded), require a dynamic extraction from the URL code of that information.


* Question 2: No, we didn't notice that (we didn't read it completely) and the attack was not taking place the first time we tried it. We then recalled to the Seedlab and this time we notice this question and fixed it. After submitting in HTML form we completed the exploit. We bet that internally authors have two distinct pipelines, the text mode that dumps the content in HTML raw text, and the HTML mode that interprets it as executable HTML, a clear wrong decision from developers.


## CTF Week 10

CTF week 10 has two different tasks in nature:

* Chanlendge 1 - That targets a XSS reflexive attack.
* Challenge 2 - That aims one more time at attacks of type buffer overflow.

### Chanlendge 1: XSS Reflexive

This task is simple but it was tricky. We were required to read multiple times the specification. And we only got it because it was almost certain it needed to target the XSS topic. Because the range of options was wide. It could be:

* SQL Injection (That was already evaluated in a CTF)
* Shell Code injection(That was also already evaluated in a CTF)

Then we focus our attention on understanding how it could fit an XSS attack, and hopefully, we reached an answer.

According to the specification:

* "O adminstrador vê todos os pedidos
* "Quando o teu pedido é visto por um administrador"
* "teu pedido pode demorar até 2 minutos"

The trick is to connect "visto" with the "read" on the *random* form that is dumped in the HTML

<div style="text-align:center">
    
![](https://i.imgur.com/j2IHQBq.png)

</div>


We from the beginning understood that the input was not being sanitized, and therefore we were allowed to inject javascript code into the admin.

If we launched the particular command alert(), the first reload became white that was our turning point, **the admin was reading this request** it also saw an alert. 

After he clicks that disabled(for us, clients) HTML button "Mark request as read" we received the message that it denied the request.


<div style="text-align:center">
    
![](https://i.imgur.com/rDwK8gp.png)

</div>

#### The Exploit

Instead of injecting a harmless alert() we needed to inject a javascript that would submit the form associated with the button "Give the flag" when he clicks the "Mark as reading button"

**The malicious payload was simple DOM manipulation:**

```javascript
<script>document.querySelector("[role='formRead']").onsubmit = function(){ 
    document.querySelector("[role='form']").submit(); return false;
}</script>
```

### Chanlendge 2: Buffer Overflow

During the FSI course, we have already performed various Buffer overflow attacks. Nevertheless, none of them was as complex as this one.

The main complexity of this challenge was detected after running the checksec command on the program

<div style="text-align:center">
    
![](https://i.imgur.com/TlAwhOW.png)

</div>

#### Checksec

As we can notice below, the execution of checksec in the executable above shows that the PIE countermeasure is active. It is the first time we were asked to perform a CTF when there were countermeasures active.

##### **What is PIE**? 

Position-independent code is a countermeasure against buffer exploitation that allows code to execute in a random address in memory in each execution. Making it a hard task to predict where the code will be placed and therefore guess the value of the memory addresses attackers need to guess to easily exploit the vulnerability.

<div style="text-align:center">
    
![](https://i.imgur.com/UONuAUX.png)
    
</div>


* **NX** is disabled, this means writable memory can also be executed. This point is important to understand how we successfully performed this attack.


Question answering:

* Question 1: The line of code that allows the attack to take shape is the use of gets(). gets() is a C function that reads a byte from the stdin until \0 or \n is found. Since we are reading towards a 100 bytes buffer, easily we can inject a payload bigger than 100 bytes overwriting the value of the return address, inject shellcode controlled by ourselves, and then jump to that code, which will be executed automatically.

* Question 2: Exploiting gets() we can rewrite the content of the return address to a place in memory with a lower address where we have deposited a large payload of shellcode and NOPs. By injecting the shellcode that returns us a bash shell we can simply perform a cat flag.txt command and the flag content will be dumped in the stdout.

The line where it is printed the buffer address using a %p printf() is very important. Without this line, an attacker was forced to guess the location where that buffer would be randomized in memory. With %p, our task was simplified.

The goal is to exploit this %p together with the NX disabled to inject in the address printed dynamically(that change at each execution) the malicious payload with shellcode and NOPs.

##### The Payload

We used the shellcode provided by Seedlabs in week 5, the assembly code in little-endian corresponding to the execution of the /bin/bash command.


##### GDB - Determine Memory Offsets

Similar to all the tasks involving buffer overflow exploitation, the next task was to determine the offset between ebp and buffer in memory. We determined, as expected since there were no extra variables or canaries, that distance was 104 bytes. Meaning that the return address is 108 bytes from the beginning of the buffer

<div style="text-align:center">
    
![](https://i.imgur.com/4TNNAqR.png)

</div>


##### Read the %p dynamically

This topic composed the main problem we had with this CTF task, we weren't familiar enough with the library pwdtools to understand how to read dynamically that value. We found on GitHub user from Texas that solved a similar CTF problem that guide us in the solution of our issue with pwtools.

The final content of the exploit that allowed us to collect the flag was:

<div style="text-align:center">
    
![](https://i.imgur.com/OCg24OL.png)

</div>

1. First we use pwtools to parse with a readline the content outputted by %p
2. Then we copy the seed lab shell code already assembled in little-endian
3. We injected 200 bytes of NOPs and the shellcode at the trailer of the payload.
4. Then we specified that the return address should be the buffer itself and that value should be placed 108 bytes above.
