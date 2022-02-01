# FSI Lab 8/9 - Web SQL Injection

## Task 1: Get Familiar with SQL Statements

First, we need to get control of the MySQL container

```bash
docker-compose up --build
docksh $(dockps | grep mysql | head -c 2) # access mysql container
```

After that, we use the following commands to get started with MySQL

```bash
# Inside MySQL container
mysql -u root -pdees
use sqllab_users;
show tables;
```

Finally, we can perform queries like below

```sql
SELECT * FROM credential;
SELECT name, salary FROM credential;
SELECT name, salary FROM credential WHERE salary > 100000;
```

The query below retrieves all the information of the employee Alice

```sql
SELECT * FROM credential WHERE NAME = 'Alice';
```

<div style="text-align:center">
    
![](https://i.imgur.com/OLqngJT.png)
    
</div>


## Task 2: SQL Injection Attack on SELECT Statement

### Task 2.1: SQL Injection Attack from webpage

In this task, we only know the username of the user that we want to authenticate as, which is `admin`. That being said, we can manipulate the authentication due to a vulnerability on the following piece of code.

```php
WHERE name= '$input_uname' and Password='$hashed_pwd'";
```

If we insert a username string with the format `username'#` we will cause the string to be closed and the rest of the code to be commented up until the closing quote of the query string, leaving the code with no errors. Below we can see more clearly what will happen at compile time if we input `admin'#`. This will allow us to log in to `admin` without providing a password input.

<div style="text-align:center">
    
![](https://i.imgur.com/EF53iNB.png)

</div>


As a result of that input, we get the page below.

<div style="text-align:center">
    
![](https://i.imgur.com/LGuuBLk.png)
    
</div>

### Task 2.2: SQL Injection Attack from the command line

For this task, we have to repeat the process above but this time we can't use the webpage. Using a curl command with the characters `'#` replaced as `%27%23` we should be able to retrieve the HTML page in the command line.

```bash
curl 'www.seed-server.com/unsafe_home.php?username=admin%27%23&Password='
```

<div style="text-align:center">
    
![](https://i.imgur.com/5VVLIh4.png)
    
</div>


### Task 2.3: Append a new SQL statement.

Following what we did on the previous 2 tasks, it is intuitive to think that a SQL command injected in between the `'` and the `#` of the username string (`username'#`) would work but upon trying an input like the `admin'; DELETE FROM credential WHERE name='Boby'";#` we get the following error.

<div style="text-align:center">
    
![](https://i.imgur.com/Y7J5URA.png)
    
</div>


This error is raised by a countermeasure implemented in `unsafe_home.php` which is the usage of `mysqli::query()`, from the PHP MySQLi extension. This means that we cannot perform SQL injection against MySQL through this method, since the `mysqli::query()` API does not allow multiple queries to be run in the database server, due to the concern of potential SQL injection.

```php
$conn = new mysqli($dbhost, $dbuser, $dbpass, $dbname);
$result = $conn->query($sql)
```

## Task 3: SQL Injection Attack on UPDATE Statement

### Task 3.1: Modify your own salary

The purpose of this task is to change Alice's value salary because she's unhappy with her compensation. To achieve that we logged in as Alice using the interface with the username input `alice'#` (we could have used the password if we were Alice). Then we went to the edit profile page and updated the nickname field with the string `', salary='1000000`, which will change Alice's salary to 1 million. This input will set Alice's nickname to an empty string and update the salary value to `'1000000'`. Below we can see more clearly what query will be run as well as the results on the webpage.

<div style="text-align:center">
    
![](https://i.imgur.com/esXyiYk.png)
    
</div>

| Alice's profile edit form  | Alice's profile after |
|:-------------------------:|:-------------------------:|
|![](https://i.imgur.com/RQFM4wy.png)  |  ![](https://i.imgur.com/9ejHOCb.png)|


### Task 3.2: Modify other people’ salary

The purpose of this task is to set Boby's salary to 1, seen as though Alice is still unhappy, this time with her boss's (Boby) compensation. To achieve this we must go to Alice's edit profile page, and input `', salary=1 WHERE name='Boby';#` on the nickname field, for example. This input will set Boby's nickname to an empty string and change his salary since the `#` at the end will hide the query that is related to Alice. To check if everything went according to plan we must log in into Boby's account using the username `Boby'#`. Below is the query that will be executed upon our input as well as the results.

<div style="text-align:center">
    
![](https://i.imgur.com/DvCDgBF.png)
    
</div>


| Alice's profile edit form  | Boby's profile after |
|:-------------------------:|:-------------------------:|
|![](https://i.imgur.com/eLsZvyx.png)  | ![](https://i.imgur.com/XYWNVWx.png) |


---


## CTF Week 8/9:

CTF for weeks eight and nine compressed two different challenges targeting injections attacks in services on web services.

The first challenge we want to overpass a simple login system. In the second one, we want to exploit a vulnerability that would allow us to seize control over the system where the authentication is stored.

Seed lab for this week target only SQL injections, CTF challenge 2 is a more general attack that inserts itself into injections but rather than inject SQL, it injects valid shell strings.

### Challenge 1

The first challenge is a white box attack since the source code is available for analysis. 

<div style="text-align:center">
    
![](https://i.imgur.com/RZmEQQ2.png)
    
</div>

In the line underlined in white, we can notice that there is in this code a string without sanitization being constructed directly from the user input. If this user is a malicious party, this element can be easily exploited to warm system integrity.

More specifically, this underline code segment allows us to authenticate into the system and after that, the content of the flag is dumped.

One important element, that makes this challenge 1 pretty unique from the general type of injection in web servers is the fact that there isn't any content rather the flag being dumped in the HTML. This means this challenge is a simple educational example, not a clone of a real example attack, where the gather of information about the system is collected incrementally, using the output dumped in the HTML.

Furthermore, there is an additional detail we can extract from this print screen. The PHP code interacts with an SQLite 3 database. This element is relevant since every SQL distro has its syntax.

Moodle specification also ensure that there is a valid username that we should use, **admin**. Without this information, we would be forced to architecture a more general attack that would allow us to bypass not only the value of the password but also the username value.

We were able to simulate a valid authentication of the admin username by performing a SQL Injection. Exploiting the vulnerability of not performing any type of sanitization of the SQL Query, we inject the following credentials:

    username: admin
    password:' OR 1=1--'

The trick is focused on the password field. If we pay attention to the print screen above we can extract that password field is inserted inside **''**, a PHP string. To perform the typical SQL injection payloads, like **OR 1=1** we need to interrupt this string, otherwise, the malicious payload would be interpreted as a string and not as SQL Syntax.

Then we should insert to help us avoid problems with the end of the string the **- -** syntax, which represents an inline comment and inject **'** to close the string.


### Challenge 2

Challenge 2 was easy to perform because of destiny. We were facing problems connecting to FEUP VPN and we have opened a windows shell to perform the command: 

    ping ctf-fsi.fe.up.pt

When we explored the webpage and found that there was a field to ping to a user-defined host, we easily understood that *under the hood* we were interacting with a Linux shell.

PHP has instructions that allow developers to execute inline shell operations. It uses C/C++ **system()** calls at the low level to achieve that result.

Moodle handout ensures us that the CTF flag is located in a file in the same path as the website.

If we can interact with a Linux shell and the content of the flag is located inside a flag.txt file, the mean of exploit must turn around the command cat [flag.txt].

We injected in the ping functionality the following malicious payload:

    8.8.8.8; cat flag.txt

Shell will first ping 8.8.8.8, Google IPV4 address and then execute the extra **cat** command that dumped to us the content of the flag.