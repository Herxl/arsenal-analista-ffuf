# 🔍 FFUF - Arsenal del Analista de Vulnerabilidades Web

> **Máster en Ciberseguridad — Módulo 6: Vulnerabilidades**
> Recurso técnico profesional orientado a portfolio

---

## 📋 Índice

- [**¿Qué es FFUF?**](#-qué-es-ffuf)
- [**¿Por qué FFUF?**](#️-por-qué-ffuf)
- [**Instalación**](#️-instalación)
- [**Estructura del repositorio**](#-estructura-del-repositorio)
- [**Casos de uso principales**](#-casos-de-uso-principales)
- [**Entorno de práctica seguro**](#-entorno-de-práctica-seguro)
- [**Recursos adicionales**](#-recursos-adicionales)

---

## 🧠 ¿Qué es FFUF?

FFUF (Fuzz Faster U Fool) es una herramienta de fuzzing web de código abierto escrita en Go.
Permite descubrir recursos ocultos en aplicaciones web mediante el reemplazo de la palabra clave `FUZZ` en cualquier parte de una petición HTTP:

- Rutas y directorios ocultos
- Parámetros GET y POST
- Subdominios y Virtual Hosts
- Cabeceras HTTP personalizadas
- Valores de cookies

Se diferencia de otras herramientas similares (como Gobuster o Dirb) por su velocidad superior, filtrado avanzado y flexibilidad total sobre la petición HTTP.

---

## ⚖️ ¿Por qué FFUF?

| Herramienta | Velocidad | Flexibilidad | Dificultad | GUI |
|-------------|-----------|--------------|------------|-----|
| FFUF | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Baja | ❌ |
| Gobuster | ⭐⭐⭐⭐ | ⭐⭐⭐ | Baja | ❌ |
| Dirb | ⭐⭐ | ⭐⭐ | Muy baja | ❌ |
| OWASP ZAP | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Media | ✅ |
| Burp Suite | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Alta | ✅ |

> FFUF se elige cuando: necesitas velocidad y control total sobre la petición en fases de reconocimiento activo.

---

## ⚙️ Instalación

### En Kali Linux (recomendado)
```bash
sudo apt update && sudo apt install ffuf -y

# Verificar instalación
ffuf -V
```

### Instalar SecLists (wordlists esenciales)
```bash
sudo apt install seclists -y

# Las wordlists quedan en:
ls /usr/share/seclists/
```

### Desde código fuente (Go)
```bash
go install github.com/ffuf/ffuf/v2@latest
```

---

## 📁 Estructura del repositorio
```
arsenal-analista-ffuf/
│
├── README.md                          ← Este archivo
├── cheatsheet/
│   └── ffuf-cheatsheet.md             ← Referencia rápida de comandos
├── workflow/
│   └── flujo-analisis-web.md          ← Diagrama del proceso completo
├── guia-operacion/
│   └── guia-rapida.md                 ← Guía para nuevos analistas
├── casos-practicos/
│   ├── 01-directorios-ocultos.md      ← Caso práctico 1
│   ├── 02-fuzzing-parametros.md       ← Caso práctico 2
│   └── 03-vhost-discovery.md          ← Caso práctico 3
└── screenshots/
    └── (capturas del laboratorio)
```

---

## 🎯 Casos de uso principales

### 1. Descubrimiento de directorios ocultos
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://TARGET/FUZZ \
     -c -fc 404
```

### 2. Fuzzing de parámetros GET
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u http://TARGET/page.php?FUZZ=test \
     -c -fc 404
```

### 3. Fuerza bruta en formulario de login (POST)
```bash
ffuf -w /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt \
     -X POST \
     -d "username=admin&password=FUZZ" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://TARGET/login.php \
     -fc 401
```

### 4. Descubrimiento de Virtual Hosts
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -u http://TARGET \
     -H "Host: FUZZ.target.com" \
     -c -fs 0
```

---

## 🧪 Entorno de práctica seguro

> ⚠️ **IMPORTANTE:** Usar FFUF contra sistemas sin autorización es ilegal. Practica **solo** en entornos controlados.

### Opción recomendada — ffuf.me (online)
```
http://ffuf.me  →  disponible online para practicar directamente
```

### Opción local — Docker
```bash
docker pull ghcr.io/ffuf/ffuf.me
docker run -p 80:80 ghcr.io/ffuf/ffuf.me
# Acceder en: http://localhost
```

### Opción local — DVWA en VirtualBox (Kali)
```bash
sudo apt install dvwa -y
sudo dvwa-start
# Acceder en: http://127.0.0.1/dvwa
```

---

## 📚 Recursos adicionales

| Recurso | Enlace |
|---------|--------|
| Repositorio oficial | [ffuf/ffuf](https://github.com/ffuf/ffuf) |
| Documentación oficial | [ffuf/wiki](https://github.com/ffuf/ffuf/wiki) |
| SecLists | [danielmiessler/SecLists](https://github.com/danielmiessler/SecLists) |
| Tutorial YouTube | Buscar: *"Fuzzing & Directory Brute-Force With ffuf HackerSploit"* |
| Entorno de práctica | [ffuf.me](http://ffuf.me) |

---

## ⚖️ Aviso legal

Este repositorio tiene fines exclusivamente educativos. El uso de estas técnicas y herramientas contra sistemas sin autorización expresa es ilegal. El autor no se responsabiliza del uso indebido de la información aquí contenida.

---

> Recurso elaborado como parte del **Máster en Ciberseguridad — Módulo 6: Vulnerabilidades**
