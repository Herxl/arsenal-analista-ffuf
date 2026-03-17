# 🧭 Guía Rápida de Operación con FFUF

> Para analistas que se incorporan al equipo  
> Máster en Ciberseguridad — Módulo 6

---

## ✅ ¿Cuándo usar FFUF?

Usa FFUF cuando necesites:

- **Descubrir directorios y archivos ocultos** en un servidor web durante la fase de reconocimiento activo
- **Enumerar parámetros GET/POST** desconocidos en páginas dinámicas
- **Identificar Virtual Hosts o subdominios** que no están listados públicamente
- **Hacer fuerza bruta de credenciales** en formularios de autenticación
- **Validar superficies de ataque** antes de un pentest más profundo

---

## ❌ ¿Cuándo NO usar FFUF?

| Situación | Herramienta más adecuada |
|-----------|--------------------------|
| Escaneo automático de vulnerabilidades | OWASP ZAP o Nikto |
| Análisis de tráfico interceptado | Burp Suite |
| Escaneo de puertos y servicios | Nmap |
| Análisis de código estático | SonarQube |
| Fuzzing de binarios (no web) | AFL++ |

> ⚠️ FFUF no analiza el contenido de las respuestas — solo informa de que un recurso existe. El análisis manual posterior es imprescindible.

---

## 🚦 Antes de empezar — Lista de verificación

```
[ ] ¿Tengo autorización por escrito para el target?
[ ] ¿Tengo SecLists instalado? (sudo apt install seclists)
[ ] ¿Conozco la tecnología del servidor? (PHP, ASP, Node...)
[ ] ¿He definido el scope correctamente?
[ ] ¿Tengo un directorio de salida para guardar los resultados?
```

---

## 🏃 Flujo mínimo para empezar en 3 comandos

```bash
# 1. Reconocimiento inicial rápido
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://TARGET/FUZZ -c -fc 404

# 2. Buscar archivos PHP y de backup
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt \
     -u http://TARGET/FUZZ \
     -e .php,.bak,.old,.txt \
     -c -fc 404

# 3. Guardar resultados para analizar
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://TARGET/FUZZ \
     -c -fc 404 \
     -o resultados_$(date +%Y%m%d).json -of json
```

---

## 🔧 Configuración recomendada según entorno

### Entorno de laboratorio (sin restricciones)

```bash
ffuf -w wordlist.txt -u http://TARGET/FUZZ \
     -t 100 -c -fc 404
```

### Entorno de producción (cuidado con WAF/IDS)

```bash
ffuf -w wordlist.txt -u http://TARGET/FUZZ \
     -t 10 \              # Pocos threads para no alertar
     -rate 20 \           # Máx 20 peticiones/segundo
     -c -fc 404
```

### Con proxy Burp para análisis paralelo

```bash
ffuf -w wordlist.txt -u http://TARGET/FUZZ \
     -x http://127.0.0.1:8080 \
     -c -fc 404
```

---

## 🧰 Combinaciones de herramientas más útiles

### FFUF + Nmap (reconocimiento completo)

```bash
# Primero, identificar puertos web con Nmap
nmap -sV -p 80,443,8080,8443 TARGET

# Luego, fuzzear cada servicio web encontrado
ffuf -w common.txt -u http://TARGET:8080/FUZZ -c -fc 404
```

### FFUF + Burp Suite (análisis profundo)

```bash
# Activar Burp proxy y redirigir FFUF por él
ffuf -w wordlist.txt \
     -u http://TARGET/FUZZ \
     -x http://127.0.0.1:8080 \
     -c -fc 404

# En Burp: Target > Site map verás todas las peticiones
```

### FFUF + OWASP ZAP (pipeline completo)

```
FFUF descubre rutas → ZAP escanea esas rutas → Informe final combinado
```

---

## 📊 Interpretación de resultados

| Código HTTP | Significado | Acción recomendada |
|-------------|-------------|-------------------|
| **200** | Recurso accesible | Analizar contenido manualmente |
| **301/302** | Redirección | Seguir la redirección |
| **403** | Prohibido (existe pero restringido) | Intentar bypass de 403 |
| **401** | Requiere autenticación | Anotar para fuerza bruta posterior |
| **500** | Error del servidor | Puede indicar vulnerabilidad |
| **404** | No encontrado | Filtrar con `-fc 404` |

> 💡 **Truco profesional:** Los códigos **403** son muy valiosos — significan que el recurso existe pero está protegido. Son candidatos para técnicas de bypass de control de acceso.

---

## 📝 Plantilla de informe rápido

```markdown
## Hallazgos FFUF — [Fecha] — [Target]

**Scope:** http://target.com
**Herramienta:** FFUF v2.x
**Wordlist:** common.txt (4613 entradas)
**Duración:** X minutos

### Recursos descubiertos

| URL | Código | Tamaño | Observaciones |
|-----|--------|--------|---------------|
| /admin | 200 | 2341 bytes | Panel de administración expuesto |
| /backup | 403 | 512 bytes | Directorio protegido — revisar bypass |
| /config.php.bak | 200 | 1024 bytes | ⚠️ Archivo de backup accesible |

### Próximos pasos
- [ ] Analizar /admin manualmente con Burp
- [ ] Intentar bypass en /backup
- [ ] Revisar contenido de /config.php.bak
```

---

*Guía elaborada como parte del Máster en Ciberseguridad — Módulo 6: Vulnerabilidades*
