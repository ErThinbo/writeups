
______________
Tags:
___________

# CONTEXTO

Somos un extrabajador de la empresa que nos acaban de echar y tenemos las credenciales de nuestra cuenta pero esta desactivada
Credenciales: samuel::fzghn4lw

# RECONOCIMIENTO



![[Pasted image 20250512115147.png]]


**FUZZING**

Con dirsearch encontramos una ruta /admin/admin.php

![[Pasted image 20250513093658.png]]

En esta ruta podemos ver todos los usuarios registrados y vemos que nuestra cuenta de samuel esta inactiva

![[Pasted image 20250513094232.png]]


Al pinchar en inactive para ver si lo podemos activar nosotros nos manda a esta pagina donde vemos que en el url tienen un parametro status=active. Puede que este enlace nos ayude a activar la cuenta si conseguimos un CSRF

![[Pasted image 20250513095227.png]]

vamos a probar a crear una cuenta y comprobar si se acontece un XSS para hacer una peticion a la url anterior con el parametro status=active

![[Pasted image 20250513095934.png]]

tiene un bloqueo en el boton sign up, pero es un bloqueo del lado del cliente que nos lo podemos saltar borrando el disable="" de este boton

![[Pasted image 20250513100757.png]]

podemos ver que el campo **firstname**  y **lastname** son vulnerables a inyecciones XSS

![[Pasted image 20250513101217.png]]

Ahora probaremos a mandar un script para acontecer el posible CSRF 

Payload para el campo Firstname o Lastname:

```bash
<img src="http://10.10.2.59/admin/admin.php?id=11&status=active">
```

Conseguimos acontecer el CSRF desde un XSS y activamos la cuenta

![[Pasted image 20250513101833.png]]

Ahora nos logueamos en nuestra cuenta activada


Podemos ver un muro donde poder postear comentarios

![[Pasted image 20250513102432.png]]

Nuestra peticion para que nos paguen 750 euros por un viaje de trabajo


![[Pasted image 20250513102443.png]]

Y nuestro perfil donde podemos ver quien es nuestro manager **Manon riviere**

![[Pasted image 20250513102452.png]]

Lo primero aceptaremos el envio de la petición de pago para que le llegue a nuestro manager

Segundo probaremos si el tablón también es vulnerable a XSS como el tipico alert(1) es muy molesto voy a usar otros que molestan menos

(Payloads)[]

Output en la consola del navegador de XSS

```bash
<script>console.log('XSS')</script>
```

El background cambiara a rojo

```bash
<body style="background:red"> 
```

Cambio de título de la página

```bash
<script>document.title = "XSS found"</script>
```




