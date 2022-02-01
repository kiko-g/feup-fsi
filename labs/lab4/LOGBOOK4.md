# Environment variable and Set-UID Program LAB
## LOGBOOK 4

The purpose of this lab is to demonstrate firstly how UNIX processes deal with environmental variables. And, secondly, the way they can affect the regular execution of your code.

This document describes the conclusions made after completing the [Seed Labs tutorial on Environment Variables and Set-UID](https://https://github.com/seed-labs/seed-labs/tree/master/category-software/Environment_Variable_and_SetUID)

The following is an academic analysis made by students of master in Computing Engineering at the University of Porto.

The document follows the suggested division of the Seed Lab tutorial.

### Task 1: Manipulating Environment Variable

In the first task, it was suggested to explore the UNIX commands that print/set & unset variable environments.

At this moment our group refreshed our thoughts on what is an env variable. According to the [wiki](https://https://en.wikipedia.org/wiki/Environment_variable):

* **Environment Variable**: An environment variable is a dynamic-named value that can affect the way running processes will behave on a computer. They are part of the environment in which a process runs.

We executed the 3 commands and got the expected behaviour. 

 1. printenv -> Printed the variables in the terminal
 2. Set & Unset -> Created and deleted env. vars.


### Task 2: Passing Environment Variables from Parent Process to Child Process

In this section, seed lab suggests we explore the way env variables are inherited by child processes after executing fork() calls.

To testify its behaviour, it was presented a code whose task is to execute the printenv() function. This way, we could see in the standard output the state of that group of variables.

According to the UNIX specification, since fork simple clones the current process in two child processes, the expected behaviour related to env. variables are to see them being copied and propagated from parent to child processes.

This propagation of env. variables were successfully witnessed. In the following picture, it is printed the output in the two processes created after fork(). They are the same.

To clarify the statement above we also executed as suggested in the tutorial, the diff UNIX command that confirmed that both stdout were exactly the same.


![](https://i.imgur.com/YqPR17X.png)
![](https://i.imgur.com/rmFRTxv.png)
###### FIG 1: Outputs of running myprintenv.c after fork() calls.


### Task 3: Environment Variables and *execve()*

Task number 3 compresses the study of the relationship that the multiple types of exec() UNIX functions establish with environmental variables. It is divided into 3 sub-tasks:

1. Study of the execve() with NULL Env.Variables:

According to the specification of execve(), it allows the execution of a user-defined program without the creation of any new process, which means the current one is taken "by assault" by the one specified in the first argument of execve(). 

The particularity of the execve() is that it allows in its third parameter to specify which env variables the new execution environment should inherit.

The 3.1 section told us to use execve() to run printenv() function (Use execve to print environment variables), but it filled the parameter allocated to specify the env. variables inheritance with NULL.

The expected and, after execution, witnessed behaviour was the absence of output by printenv, more specifically the print of a NULL, thing that in C result in the absence of output.

2. Study of the execve() with environ propagation:

environ is a global variable defined in POSIX specification that stores the environment variables of a program in C struct style.

Seed labs dare us to testify what was the output of the same code described above (in 3.1) filled the third parameter of execve() with this new global variable.


The expected and, after execution, witnessed behaviour was the propagation of the default env. variables. Since the context didn't change dramatically, the output was the same as in task 1 and task 2.

![](https://i.imgur.com/1SwYlmZ.png)
![](https://i.imgur.com/t0cb6HW.png)
###### FIG 2: Outputs of running myenv.c with **NULL** (output3.txt) and **environ** (output4.txt).

### Task 4:

Task 4 comes after the exploration of the way exec() family of functions deal with environment variables.

In task 4 Seed lab tells us to testify the way an alternative way of running programs, the system() call defined in stdlib library, deals on its way with environment variables.

system() is different in nature from exec(). system() does return, but it uses exec under the hood. system() according to the specification is a concatenation of the following commands, a fork followed by an exec. A more high-level approach to the execution of functions.

We notice that both functions propagate env. variables, the main difference is that exec() cleans the current execution, while system() keeps state.

This state-keeping property of system() requires from the programmer a special attention. Since the possibility of changing env. could result in unexpected behavior.


![](https://i.imgur.com/qSzPTcN.png)
![](https://i.imgur.com/Z2QvsUA.png)
###### FIG 3: Outputs of running task4.c.


### Task 5: Environment Variable and Set-UID Programs

This task is divided into 3 parts. It is intended to explore the way root privileges affect environment variables.

One keyword was introduced in this section, **[Set-UID](https://https://en.wikipedia.org/wiki/Setuid)** We went to Wiki for clarification. According to it:
* They[Set-UID Programs] are often used to allow users on a computer system to run programs with temporarily elevated privileges to perform a specific task.


1. &  2. Verification if root mode executions change env variables.

In this subtask, seed labs required us to test if the execution of the same line of code in sudo mode and user mode, would generate different env variables out of the box.

The expectation on our side, which was further confirmed, was that it won't produce any difference.

The picture below proves that.

![](https://i.imgur.com/RxffU5o.png)
###### FIG 4: Environment variables without (output6.txt) and with (output7.txt) Set-UID permissions

##### Note: Task 2. was simply the introduction of the commands for change permissions and ownership since our group own that knowledge, we won't make further detail on it.



3. Exporting of variables & Set-UID execution.

In this task, we were asked to export some variables (**PATH**, **LD_LIBRARY_PATH** and a new one, which we called **ANY_NAME**) and as expected, since we were working on a program with root privileges on a non-root account, not all environment variables were exported correctly. 
The permissions of PATH and LD_LIBRARY_PATH variables are default defined by the OS itself because they are important vars, so, giving them other values can cause some issues, mainly in the second one. Since the ANY_NAME is a new variable, user-defined, the permissions were set at the moment, and accessible by the non-root user.

![](https://i.imgur.com/qB8UCyl.png)
###### FIG 5: Environment variables after exporting PATH (line 46), LD_LIBRARY_PATH and ANY_NAME (line 8).


### Task 6: The PATH Environment Variable and Set-UID Programs

In task 6 we conclude our environment variable journey. 

Seed labs told vaguely that the easiness of a user side manipulation of reserved variable environments like $PATH could end up in the hands of a malicious agent introduce harm to the system. It introduced the possibility of using the "ls" UNIX command as a way with the correct env. variables to perform this attack.

Then, moved by curiosity, we crafted an attack by ourselves.

1. The problem is that $PATH works with relative paths overall operating system;
2. Since we can easily modify on the user side the $Path variable, we can manipulate the endpoint in the filesystem where a relative path will end to;
3. We created a C Program that only task was to print "AHAH got you!" and compile it with -o flag with the name of "ls";
4. We then manipulated the $PATH var to point for /home/seed;
5. We moved our crafted ls binary to /home/seed
6. Then when a user calls "ls" it will execute our code, not the widely famous Linux command for listing.

This example could get worse if instead of a harmless hello world we injected malicious code. 

The worst scenario was the execution of the malicious using root privileges. This would allow an attacker to execute privileged commands at will.

Below are specified the sequence of commands to allow the execution of a previously crafted C code.


```sh 
export PATH=/home/seed:$PATH    # append /home/seed to existing PATH
gcc malicious.c -o ls           # compile malicious code as `ls` executable
mv ls /home/seed                # move ls script to home directory
gcc task6.c                     # compile script that executes system("ls")
./a.out                         # executes alt version of ls in home dir
```

![](https://i.imgur.com/HWlquFc.png)


###### FEUP 07/11/2021

---

**IMPORTANTE: Estamos a adicionar a informação sobre os CTF à posteriori tal como indicado pelo professor Manuel Barbosa, no email de dia 15/12/2021 trocado com o aluno up201704618**

## CTF Week 4 (Starting on 8/11)

### Submitted flags

- **Challenge 1**: flag{CVE-2021-34646}
- **Challenge 2**: flag{055aa413a9d380df86b33eff7f4047c5}

### Exploit context and approach

After some research we were able to sum up the context as follows:

Intel collected from navigating on the webapp

- **Wordpress version**: 5.8.1
- **Wordpress plugins**: WooCommerce @ 5.7.1 and WooCommerce Booster Plugin @ 5.4.3
- **Users**: Found `admin` and `Orval Sanford`

**Vulnerability found/chosen**: Authentication Bypass\
**CVE**: CVE-2021-34646, with CVSS Score: 9.8 (Critical)\
**Exploit Database entry**: https://www.exploit-db.com/exploits/50299\
**Using the exploit on the server**:

```sh
# saved the python script as exploit_50299.py
# found admin id through http://ctf-fsi.fe.up.pt:5001/wp-json/wp/v2/users/

python exploit_50299.py http://ctf-fsi.fe.up.pt:5001 1
```

The exploit script needs the `url` and `user_id` to be executed which are both known from the recognition phase. Executing the script will provide multiple entries for email verification, giving us authentication on the website. 

![](https://i.imgur.com/1c2VL2I.png)

Now that we're logged in if we visit http://ctf-fsi.fe.up.pt:5001/wp-admin/edit.php we get a page with a private post containing sensitive information.

![](https://i.imgur.com/dNGqwaS.png)

![](https://i.imgur.com/BmmCEbt.png)
