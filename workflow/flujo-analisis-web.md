# 🔄 Flujo de Trabajo con FFUF — Del Reconocimiento al Hallazgo

> Guía visual del proceso completo de análisis usando FFUF  
> Máster en Ciberseguridad — Módulo 6

---

## 📊 Diagrama del flujo completo

```
┌─────────────────────────────────────────────────────────┐
│                  INICIO DEL ANÁLISIS                     │
│              (Scope definido y autorizado)               │
└─────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  FASE 1 — RECONOCIMIENTO PASIVO                          │
│  • ¿Qué tecnología usa el servidor? (Wappalyzer, curl)   │
│  • ¿Qué puertos están abiertos? (Nmap)                  │
│  • ¿Hay información pública? (Google Dorks, Shodan)      │
└─────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  FASE 2 — SELECCIÓN DE WORDLIST                          │
│                                                          │
│  ¿PHP/ASP/CGI?  →  usar extensiones específicas         │
│  ¿API REST?     →  usar api-endpoints.txt               │
│  ¿Genérico?     →  usar common.txt                      │
│  ¿Login?        →  usar password lists                  │
└─────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  FASE 3 — PRIMER ESCANEO (Directorios base)              │
│                                                          │
│  ffuf -w common.txt -u http://TARGET/FUZZ -c -fc 404    │
│                                                          │
│  → Anotar todos los códigos 200, 301, 302, 403           │
└─────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  FASE 4 — PROFUNDIZAR (Recursión y extensiones)          │
│                                                          │
│  • Añadir -recursion -recursion-depth 2                  │
│  • Añadir -e .php,.bak,.txt,.html                        │
│  • Explorar rutas 403 con herramientas de bypass         │
└─────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  FASE 5 — FUZZING DE PARÁMETROS                          │
│                                                          │
│  En rutas interesantes encontradas:                      │
│  • Fuzzear parámetros GET                                │
│  • Fuzzear parámetros POST                               │
│  • Probar cabeceras personalizadas                       │
└─────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  FASE 6 — ANÁLISIS MANUAL (Burp Suite)                   │
│                                                          │
│  Redirigir tráfico FFUF por proxy Burp:                  │
│  ffuf [...] -x http://127.0.0.1:8080                     │
│                                                          │
│  Inspeccionar respuestas interesantes manualmente        │
└─────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  FASE 7 — DOCUMENTACIÓN Y REPORTE                        │
│                                                          │
│  ffuf [...] -o resultados.json -of json                  │
│                                                          │
│  • Clasificar hallazgos por severidad                    │
│  • Tomar capturas de pantalla                            │
│  • Redactar informe técnico                              │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 Árbol de decisión: ¿Qué tipo de fuzzing aplicar?

```
¿Qué quiero descubrir?
│
├─► Rutas y archivos ocultos
│       └─► ffuf -u http://TARGET/FUZZ -w common.txt
│
├─► Parámetros GET desconocidos
│       └─► ffuf -u http://TARGET/page.php?FUZZ=valor -w params.txt
│
├─► Valores de parámetros (LFI, SQLi, etc.)
│       └─► ffuf -u http://TARGET/page.php?id=FUZZ -w payloads.txt
│
├─► Credenciales de login
│       └─► ffuf -X POST -d "user=admin&pass=FUZZ" -w passwords.txt
│
└─► Subdominios o VirtualHosts
        └─► ffuf -H "Host: FUZZ.target.com" -u http://TARGET -w subdomains.txt
```

---

## ⏱️ Estimación de tiempos por fase

| Fase | Tiempo estimado | Herramientas |
|------|-----------------|-------------|
| Reconocimiento pasivo | 10-20 min | Nmap, curl, Wappalyzer |
| Selección de wordlist | 5 min | SecLists |
| Primer escaneo FFUF | 5-15 min | FFUF |
| Recursión y extensiones | 10-30 min | FFUF |
| Fuzzing de parámetros | 10-20 min | FFUF |
| Análisis manual Burp | Variable | Burp Suite |
| Documentación | 15-30 min | Markdown/PDF |

---

*Flujo elaborado como parte del Máster en Ciberseguridad — Módulo 6: Vulnerabilidades*
