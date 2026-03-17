# 🎯 Caso Práctico — Reconocimiento Web con FFUF en ffuf.me

> Entorno: Kali Linux en VirtualBox  
> Objetivo: http://ffuf.me (entorno oficial de práctica de FFUF)  
> Nivel: Principiante

---

## 🧪 Contexto del ejercicio

`ffuf.me` es el entorno oficial de práctica creado por los propios desarrolladores de FFUF. Contiene rutas ocultas, parámetros y retos diseñados específicamente para aprender a usar la herramienta de forma segura y legal.

**Objetivo:** Descubrir todos los recursos ocultos del servidor usando las técnicas aprendidas.

---

## 🛠️ Preparación del entorno

```bash
# 1. Verificar que ffuf está instalado
ffuf -V

# 2. Verificar que SecLists está disponible
ls /usr/share/seclists/Discovery/Web-Content/common.txt

# 3. Crear directorio de resultados
mkdir -p ~/ffuf-practica/resultados
cd ~/ffuf-practica
```

---

## 📋 Paso 1 — Reconocimiento inicial

Antes de lanzar FFUF, recopilar información básica del servidor:

```bash
# ¿Qué tecnología usa el servidor?
curl -I http://ffuf.me

# Resultado esperado:
# Server: Apache/2.4.x
# X-Powered-By: PHP/x.x
```

> 💡 Saber que usa PHP nos indica que debemos buscar archivos `.php` además de directorios.

---

## 📋 Paso 2 — Primer escaneo de directorios

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://ffuf.me/cd/basic/FUZZ \
     -c -fc 404 \
     -o resultados/paso2_directorios.json -of json
```

**Resultados esperados:**

```
class                   [Status: 301, Size: 0]
development.log         [Status: 200, Size: 1]
```

![escaneo de directorios](https://raw.githubusercontent.com/Herxl/arsenal-analista-ffuf/main/screenshots/primer_escaneo_de_directorios.png)
---

## 📋 Paso 3 — Búsqueda con extensiones

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt \
     -u http://ffuf.me/cd/ext/logs/FUZZ \
     -e .log,.txt,.bak,.php \
     -c -fc 404 \
     -o resultados/paso3_extensiones.json -of json
```

**Resultados esperados:**

```
users.log               [Status: 200, Size: 162]
```

![búsqueda con extensiones](https://raw.githubusercontent.com/Herxl/arsenal-analista-ffuf/main/screenshots/busqueda_extensiones.png)

---

## 📋 Paso 4 — Fuzzing de parámetros GET

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u "http://ffuf.me/cd/param/data?FUZZ=1" \
     -c -fc 404 \
     -o resultados/paso4_parametros.json -of json
```

**Resultados esperados:**

```
debug                   [Status: 200, Size: 1]
```

![fuzzing de parámetros](https://raw.githubusercontent.com/Herxl/arsenal-analista-ffuf/main/screenshots/fuzzing_parametros.png)

---

## 📋 Paso 5 — Filtrado por tamaño de respuesta

A veces los resultados falsos positivos tienen todos el mismo tamaño. Se puede filtrar así:

```bash
# Primero, identificar el tamaño de las respuestas "vacías"
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://ffuf.me/cd/no404/FUZZ \
     -c

# Luego filtrar ese tamaño específico (ejemplo: 669 bytes)
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://ffuf.me/cd/no404/FUZZ \
     -c -fs 669 \
     -o resultados/paso5_filtrado.json -of json
```

**Resultados esperados:**

```
secret                  [Status: 200, Size: 25]
```

![filtrado por tamaño](https://raw.githubusercontent.com/Herxl/arsenal-analista-ffuf/main/screenshots/filtrado_tama%C3%B1o.png)

---

## 📊 Resumen de hallazgos

| Paso | Técnica | Recurso descubierto | Código |
|------|---------|---------------------|--------|
| 2 | Directorios básicos | `/development.log` | 200 |
| 3 | Extensiones .log | `/users.log` | 200 |
| 4 | Parámetros GET | `?debug=1` | 200 |
| 5 | Filtro por tamaño | `/secret` | 200 |

---

## 💡 Lecciones aprendidas

1. **Siempre hacer reconocimiento previo** — conocer la tecnología ayuda a elegir las extensiones correctas
2. **El filtrado es tan importante como el escaneo** — sin `-fc` y `-fs` los resultados son inmanejables
3. **Los falsos positivos son comunes** — aprender a identificarlos es una habilidad crítica
4. **Guardar siempre los resultados** — `-o output.json -of json` para documentar el hallazgo

---

## 🔗 Recursos para seguir practicando

- **ffuf.me** — http://ffuf.me (entorno oficial)
- **HackTheBox** — máquinas de nivel principiante con servicios web
- **TryHackMe** — sala "Overpass" y similares
- **DVWA** — instalar local en Kali: `sudo apt install dvwa -y`

---

*Caso práctico elaborado como parte del Máster en Ciberseguridad — Módulo 6: Vulnerabilidades*
