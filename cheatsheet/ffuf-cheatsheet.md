# 📄 FFUF — CheatSheet Profesional

> Referencia rápida de comandos para análisis de vulnerabilidades web  
> Actualizado: 2025 | Kali Linux

---

## ⚡ Sintaxis base

```
ffuf -w <wordlist> -u <URL con FUZZ> [opciones]
```

La clave es la palabra `FUZZ` — se puede colocar en **cualquier parte** de la URL o la petición.

---

## 🗂️ 1. Descubrimiento de directorios y archivos

```bash
# Básico — directorios ocultos
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://TARGET/FUZZ

# Con colores y filtrar 404
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://TARGET/FUZZ \
     -c -fc 404

# Buscar archivos con extensión específica
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt \
     -u http://TARGET/FUZZ.php \
     -c -fc 404

# Múltiples extensiones a la vez
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt \
     -u http://TARGET/FUZZ \
     -e .php,.html,.txt,.bak \
     -c -fc 404
```

---

## 🔎 2. Fuzzing de parámetros GET

```bash
# Descubrir parámetros ocultos en URL
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u http://TARGET/page.php?FUZZ=test \
     -c -fc 404

# Fuzzear el valor de un parámetro conocido
ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt \
     -u "http://TARGET/page.php?file=FUZZ" \
     -c -fc 404
```

---

## 📬 3. Fuzzing de peticiones POST

```bash
# Fuerza bruta de login — password
ffuf -w /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt \
     -X POST \
     -d "username=admin&password=FUZZ" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://TARGET/login.php \
     -fc 401

# Fuzzear usuario y contraseña a la vez (FUZZ y W2)
ffuf -w users.txt:FUZZ \
     -w passwords.txt:W2 \
     -X POST \
     -d "username=FUZZ&password=W2" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://TARGET/login.php \
     -fc 401
```

---

## 🌐 4. Virtual Host / Subdomain Discovery

```bash
# Descubrir Virtual Hosts por cabecera Host
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -u http://TARGET \
     -H "Host: FUZZ.target.com" \
     -c -fs 0

# Subdominios DNS (modo recursión)
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -u http://FUZZ.target.com \
     -c -fc 404
```

---

## 🎛️ 5. Opciones y flags más importantes

### Opciones de entrada

| Flag | Descripción | Ejemplo |
|------|-------------|---------|
| `-w` | Wordlist (ruta:KEYWORD) | `-w list.txt` |
| `-u` | URL objetivo | `-u http://target/FUZZ` |
| `-X` | Método HTTP | `-X POST` |
| `-d` | Datos POST | `-d "user=FUZZ"` |
| `-H` | Cabecera HTTP | `-H "Authorization: Bearer TOKEN"` |
| `-b` | Cookie | `-b "session=abc123"` |
| `-x` | Proxy | `-x http://127.0.0.1:8080` |

### Opciones de rendimiento

| Flag | Descripción | Valor recomendado |
|------|-------------|-------------------|
| `-t` | Threads | `-t 50` (cuidado con WAF) |
| `-rate` | Peticiones por segundo | `-rate 100` |
| `-timeout` | Timeout por petición | `-timeout 10` |

### Filtros (MUY IMPORTANTES)

| Flag | Filtra por | Ejemplo |
|------|-----------|---------|
| `-fc` | Código de estado (excluir) | `-fc 404,403` |
| `-mc` | Código de estado (incluir) | `-mc 200,301` |
| `-fs` | Tamaño de respuesta (excluir) | `-fs 1234` |
| `-ms` | Tamaño de respuesta (incluir) | `-ms 200` |
| `-fw` | Número de palabras (excluir) | `-fw 10` |
| `-fl` | Número de líneas (excluir) | `-fl 5` |
| `-fr` | Regex en respuesta (excluir) | `-fr "Not Found"` |
| `-mr` | Regex en respuesta (incluir) | `-mr "Welcome"` |

### Salida y resultados

| Flag | Descripción | Ejemplo |
|------|-------------|---------|
| `-o` | Guardar resultados | `-o resultados.json` |
| `-of` | Formato de salida | `-of json` o `-of csv` |
| `-c` | Colores en consola | `-c` |
| `-v` | Verbose (muestra URL completa) | `-v` |
| `-s` | Silent (solo resultados) | `-s` |

---

## 🧩 6. Casos de uso avanzados

### Integración con Burp Suite (proxy)

```bash
# Redirigir tráfico por Burp para inspección
ffuf -w wordlist.txt \
     -u http://TARGET/FUZZ \
     -x http://127.0.0.1:8080 \
     -c -fc 404
```

### Recursión automática en directorios

```bash
# Explorar subdirectorios automáticamente
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://TARGET/FUZZ \
     -recursion \
     -recursion-depth 2 \
     -c -fc 404
```

### Fuzzing con cabecera de autenticación

```bash
# Bearer Token
ffuf -w wordlist.txt \
     -u http://TARGET/api/FUZZ \
     -H "Authorization: Bearer <TOKEN>" \
     -H "Content-Type: application/json" \
     -c -fc 401,404
```

### Guardar solo los positivos en JSON y analizar

```bash
ffuf -w wordlist.txt \
     -u http://TARGET/FUZZ \
     -c -fc 404 \
     -o resultados.json -of json

# Ver solo las URLs encontradas
cat resultados.json | python3 -m json.tool | grep url
```

---

## 📦 7. Wordlists recomendadas (SecLists)

| Caso de uso | Ruta en Kali |
|-------------|-------------|
| Directorios comunes | `/usr/share/seclists/Discovery/Web-Content/common.txt` |
| Directorios grandes | `/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt` |
| Nombres de parámetros | `/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt` |
| Subdominios | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` |
| Contraseñas comunes | `/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt` |
| LFI | `/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt` |
| APIs | `/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt` |

---

## ⚠️ 8. Qué evitar

| ❌ Error común | ✅ Solución |
|----------------|------------|
| No filtrar resultados | Siempre usar `-fc 404` como mínimo |
| Threads muy altos | Usar `-t 50` máximo; WAFs bloquean a partir de cierto umbral |
| No guardar resultados | Usar siempre `-o output.json -of json` |
| Atacar sistemas reales | Solo en entornos autorizados o laboratorios como ffuf.me |
| Ignorar el tamaño de respuesta | Si `-fc` no funciona bien, probar con `-fs` para filtrar por bytes |

---

## 🔗 9. Combinaciones recomendadas con otras herramientas

```
Nmap → detecta puertos y servicios web abiertos
   ↓
FFUF → fuzzing de directorios, parámetros, vhosts
   ↓
Burp Suite → análisis manual de los endpoints descubiertos
   ↓
Nikto / OWASP ZAP → escaneo automático de vulnerabilidades en rutas halladas
```

---

*CheatSheet elaborado como parte del Máster en Ciberseguridad — Módulo 6: Vulnerabilidades*
