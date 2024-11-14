# H4ckGiver

# Enumeración

Empezamos con un escaneo de puertos.  

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 10.70.10.129 -oN H4ckGiver`  

- `-p-` : Todos los puertos
- `--open` : Todos los puertos abiertos
- `-sC` : Escaneo de todos los scripts
- `-sS` : Escaneo de todos los servicios
- `-sV` : Escaneo de todas las versiones de los servicios
- `--min-rate 5000` : Establece mínimo envío de paquetes mínimos 5000/s
- `-n` : No aplica resolución DNS
- `-Pn` : Omisión de detección de hosts
- `-vvv` : Verbose - Muestra toda la información

# Resultado escaneo  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_1.jpg) 

Tenemos tres puertos abiertos: el 22, el 80 y el 3306.  
Echemos un vistazo al puerto 80 para ver qué encontramos.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_2.jpg)   

Nos encontramos con esta página, que a primera vista no contiene nada interesante, ni siquiera en el código fuente.  
Así que realizaremos un poco de fuzzing para ver si conseguimos descubrir algún recurso adicional.  

`sudo gobuster dir -u http://10.70.10.129 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x bak,htm,html,php,txt,zip`  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_3.jpg)    

¡Perfecto!   
Encontramos un archivo llamado **personal.zip**.  
Entonces lo descargamos y vemos qué hay dentro.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_4.jpg)     

Al intentar descomprimirlo, vemos que solicita una contraseña.  
Lo que podemos hacer es utilizar la herramienta **zip2john** para extraer el hash y, posteriormente, utilizar la herramienta **John the Ripper** para intentar descifrar la contraseña.  
Ejecutaremos entonces el siguiente comando:  

`sudo zip2john personal.zip > hash`

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_5.jpg)     

Ahora intentaremos descifrar la contraseña ingresando el siguiente comando:  

`sudo john -w=/usr/share/wordlists/rockyou.txt hash`  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_6.jpg)       

¡Perfecto!  
Después de descomprimir la carpeta, leeremos el contenido del archivo **mission.txt** y podremos obtener un nombre de usuario, **jdalton**.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_7.jpg)     

Intentamos realizar un ataque de fuerza bruta con **hydra** contra SSH, pero vemos que no obtenemos resultados.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_8.jpg)  

Entonces intentaremos realizar el mismo tipo de ataque, pero esta vez contra MySQL, dado que sabemos que el puerto 3306 está abierto.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_9.jpg)  

¡Excelente!  
¡Hemos obtenido credenciales válidas!

Ahora nos conectaremos a MySQL con el siguiente comando:  

`sudo mysql -h 10.70.10.129 -u jdalton -p12345 --skip-ssl`    

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_10.jpg)    

Una vez dentro, buscaremos bases de datos que puedan contener información interesante.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_11.jpg)      

¡Estupendo!  
Hemos encontrado la contraseña del usuario *jdalton*, intentemos conectarnos por SSH para ver si funciona.  

`ssh jdalton@10.70.10.129`  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_12.jpg)     

¡Estamos dentro!  

# Escalada de privilegios  

Lo primero que haremos es ver si hay algún archivo interesante en nuestra carpeta *home*.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_13.jpg)     

Abrimos este archivo oculto llamado **nota.txt**.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_14.jpg)    

Vemos que con mucha probabilidad esto podría ser una contraseña.    
Comprobamos con el comando `cat /etc/passwd` si hay otros usuarios en el sistema.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_15.jpg)     

¡Perfecto!  
Intentemos iniciar sesión como *murdoc* con el comando `su murdoc` e ingresando la contraseña que acabamos de descubrir.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_16.jpg)    

Después de revisar la carpeta *home* del usuario *murdoc* en busca de algún archivo interesante, y al no encontrar nada, lo que haremos será ejecutar el comando `sudo -l`.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_17.jpg)  

Vemos que podemos ejecutar el binario **find** con permisos de sudo.  
Consultamos la web de GTFOBins [https://gtfobins.github.io/](https://gtfobins.github.io/) para ver cómo podemos abusar de este binario.  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_18.jpg)    

Entonces, lo único que nos queda es ejecutar el siguiente comando:  

`sudo find . -exec /bin/sh \; -quit`  

![H](https://github.com/giustiand/CTF-Writeups/blob/main/F%C3%A1cil/images/H4ckGiver/H_19.jpg)      

¡Y listo!   
¡Somos root!  



