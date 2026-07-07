<div align="center">

# 🍯 SOCEmpire — T-Pot Honeypot Deployment & Real Attack Analysis

### Despliegue de un honeypot multi-servicio en Google Cloud y análisis forense de más de 1 millón de ataques reales

**Proyecto Final · KeepCoding Bootcamp Ciberseguridad Ed.11**

<br>

![Status](https://img.shields.io/badge/Estado-Completado-success?style=for-the-badge)
![Duración](https://img.shields.io/badge/Captura-16_días-blue?style=for-the-badge)
![Ataques](https://img.shields.io/badge/Ataques-1.000.000+-red?style=for-the-badge)

<br>

![T-Pot](https://img.shields.io/badge/T--Pot-24.04-00D4FF?style=flat-square&logo=docker&logoColor=white)
![GCP](https://img.shields.io/badge/Google_Cloud-Compute_Engine-4285F4?style=flat-square&logo=googlecloud&logoColor=white)
![Debian](https://img.shields.io/badge/Debian-12_Bookworm-A81D33?style=flat-square&logo=debian&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-35_contenedores-2496ED?style=flat-square&logo=docker&logoColor=white)
![Elastic](https://img.shields.io/badge/Elastic-Kibana-005571?style=flat-square&logo=elasticsearch&logoColor=white)
![Suricata](https://img.shields.io/badge/Suricata-IDS-E4002B?style=flat-square)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-000000?style=flat-square)

<br>

*«Puse una trampa en Internet. En menos de 5 minutos ya estaba siendo atacada.  
En 16 días, más de un millón de veces.»*

</div>

---

## 📋 Tabla de Contenidos

- [Resumen del Proyecto](#-resumen-del-proyecto)
- [El Equipo](#-el-equipo)
- [¿Qué es un Honeypot y qué es T-Pot?](#-qué-es-un-honeypot-y-qué-es-t-pot)
- [Arquitectura del Despliegue](#-arquitectura-del-despliegue)
- [Infraestructura Técnica](#-infraestructura-técnica)
- [Proceso de Instalación](#-proceso-de-instalación)
- [Problemas Encontrados y Soluciones](#-problemas-encontrados-y-soluciones)
- [Resultados Globales](#-resultados-globales)
- [Análisis por Honeypot](#-análisis-por-honeypot)
- [Análisis Forense del Malware Redtail](#-análisis-forense-del-malware-redtail)
- [Mapeo MITRE ATT&CK](#-mapeo-mitre-attck)
- [Investigación OSINT de los Atacantes](#-investigación-osint-de-los-atacantes)
- [Conclusiones](#-conclusiones)
- [Próximos Pasos](#-próximos-pasos)
- [Contenido del Repositorio](#-contenido-del-repositorio)
- [Glosario](#-glosario)

---

## 🎯 Resumen del Proyecto

Este repositorio documenta el proyecto final del **Bootcamp de Ciberseguridad Ed.11 de KeepCoding**: el despliegue, monitorización y análisis forense de un **honeypot T-Pot** desplegado en infraestructura cloud pública y expuesto a Internet durante **16 días** (del 12 al 28 de junio de 2026).

Un honeypot es un sistema deliberadamente vulnerable cuyo único propósito es **atraer atacantes** para estudiar cómo operan. A diferencia de un entorno de producción, aquí **todo el tráfico es malicioso por definición**: no hay usuarios legítimos, no hay falsos positivos. Cada conexión es un ataque.

Los resultados hablan por sí solos:

| Métrica | Valor |
|:--|:--|
| 🔴 **Ataques totales capturados** | **+1.000.000** |
| 🌍 **IPs únicas atacantes** | 13.659 |
| 🔓 **Logins SSH exitosos** | 85 |
| 🐛 **CVEs detectados en tiempo real** | 10+ |
| 🦠 **Familias de malware capturadas** | Múltiples (ohshit, redtail/XMRig) |
| ⏱️ **Tiempo hasta el primer ataque** | < 5 minutos |

Durante el proyecto se capturó y analizó malware real (el cryptominer **Redtail/XMRig**), se detectaron vulnerabilidades siendo explotadas activamente (incluida una de **hace 20 años** detectada 74.440 veces), se identificó tráfico de la herramienta de Red Team **Cobalt Strike** asociada a grupos APT, y se investigaron las principales IPs atacantes mediante OSINT.

---

## 👥 El Equipo

**SOC con Gofio**

| Autor | Rol |
|:--|:--|
| **Nicolás Navares Sánchez** | Despliegue cloud, análisis de datos, análisis forense de malware |
| **Daniel Landeira** | Análisis de honeypots, documentación, investigación OSINT |

---

## 🍯 ¿Qué es un Honeypot y qué es T-Pot?

### El concepto

Un **honeypot** (tarro de miel) es un señuelo. Se despliega un sistema que *parece* vulnerable y valioso, se expone a Internet, y se registra absolutamente todo lo que los atacantes intentan hacer contra él. Su gran ventaja frente a un SIEM tradicional es la **ausencia de ruido**: en un entorno normal, el 90% del tráfico es legítimo y hay que buscar la aguja en el pajar. En un honeypot, si alguien se conecta, está atacando.

### T-Pot

[**T-Pot**](https://github.com/telekom-security/tpotce) es un framework *open source* desarrollado por **Deutsche Telekom Security** que integra más de 20 honeypots diferentes bajo una plataforma unificada, cada uno aislado en su propio contenedor Docker, con un stack **ELK** (Elasticsearch, Logstash, Kibana) completo para la visualización en tiempo real.

**Honeypots desplegados en este proyecto:**

| Honeypot | Servicio simulado | Función |
|:--|:--|:--|
| **Cowrie** | SSH / Telnet | Captura credenciales, comandos y malware descargado |
| **Dionaea** | SMB, MySQL, MSSQL, MongoDB | Trampa de malware multi-protocolo |
| **Heralding** | SSH, FTP, VNC, PostgreSQL, SOCKS5 | Captura de credenciales multi-protocolo |
| **Honeytrap** | Multi-puerto | Captura conexiones en puertos no cubiertos |
| **H0neytr4p** | Web / HTTP | Honeypot HTTP avanzado |
| **Mailoney** | SMTP (puerto 25) | Honeypot de correo electrónico |
| **Suricata** | IDS de red | Inspección profunda de paquetes, detección de CVEs |
| **Fatt / P0f** | Fingerprinting | Identificación de clientes y sistemas operativos |

---

## 🏗️ Arquitectura del Despliegue

```
                            🌍 INTERNET (atacantes de todo el mundo)
                                          │
                                          │  Puertos 1–64000 (TCP/UDP) abiertos
                                          ▼
        ┌─────────────────────────────────────────────────────────────────┐
        │              GOOGLE CLOUD PLATFORM (europe-southwest1-a)          │
        │  ┌───────────────────────────────────────────────────────────┐  │
        │  │        VM: tpot-honeypot  ·  e2-standard-4  ·  Debian 12   │  │
        │  │                    IP pública: 34.175.139.110              │  │
        │  │                                                            │  │
        │  │   ┌──────────────────── DOCKER (35 contenedores) ──────┐   │  │
        │  │   │                                                    │   │  │
        │  │   │   HONEYPOTS            IDS          ANÁLISIS        │   │  │
        │  │   │   ┌─────────┐      ┌──────────┐    ┌────────────┐   │   │  │
        │  │   │   │ Cowrie  │      │ Suricata │    │Elasticsearch│  │   │  │
        │  │   │   │ Dionaea │─────▶│          │───▶│  Logstash  │   │   │  │
        │  │   │   │Heralding│      └──────────┘    │   Kibana   │   │   │  │
        │  │   │   │ ...     │                      └────────────┘   │   │  │
        │  │   │   └─────────┘                            │          │   │  │
        │  │   └──────────────────────────────────────────┼─────────┘   │  │
        │  │                                              │             │  │
        │  └──────────────────────────────────────────────┼─────────────┘  │
        │                                                  │                │
        │   Puertos de gestión (firewall restringido):     ▼                │
        │   · 64295 → SSH real de administración      Web UI / Kibana       │
        │   · 64297 → Interfaz web (Attack Map, dashboards)                 │
        └─────────────────────────────────────────────────────────────────┘
                                          ▲
                                          │  Solo administradores (SOCEmpire)
                                     👤 Equipo SOC con Gofio
```

**Principio de diseño clave:** dos reglas de firewall opuestas. Los puertos de los honeypots (1–64000) están **completamente abiertos** para atraer atacantes, mientras que los puertos de gestión (64295, 64297) están **restringidos**. Cada honeypot corre aislado en su contenedor: si un atacante compromete uno, queda atrapado sin poder saltar al host ni a los demás.

---

## 💻 Infraestructura Técnica

### Selección del proveedor cloud: Oracle → GCP

El plan original era **Oracle Cloud Free Tier** (instancias ARM gratuitas permanentes). Sin embargo, la región de Madrid presentaba un error persistente de `Out of capacity` que impedía crear instancias. Tras varios días de intentos, se tomó la decisión estratégica de **migrar a Google Cloud Platform**, que ofrecía 258 € en créditos gratuitos.

> 💡 **Lección aprendida:** la capacidad de adaptarse ante fallos de infraestructura y tener un plan B es tan importante como las habilidades técnicas. *Sin plan B, no hay proyecto.*

### Especificaciones de la máquina virtual

| Parámetro | Valor |
|:--|:--|
| **Nombre** | `tpot-honeypot` |
| **Tipo** | e2-standard-4 (4 vCPU, 16 GB RAM) |
| **Sistema operativo** | Debian 12 (Bookworm) |
| **Disco** | 128 GB SSD persistente |
| **Zona** | europe-southwest1-a (Madrid) |
| **IP pública** | 34.175.139.110 |

> ℹ️ T-Pot requiere un mínimo de 8 GB de RAM (solo Elasticsearch consume 4–6 GB bajo carga). Los 16 GB y 4 vCPUs dan margen para ejecutar más de 30 contenedores sin degradación. El disco de 128 GB es obligatorio: si se llena, Elasticsearch entra en modo solo lectura y deja de registrar.

---

## ⚙️ Proceso de Instalación

```bash
# 1. Acceso por SSH (desde el navegador de GCP, sin configurar claves)

# 2. Clonar el repositorio oficial de T-Pot
git clone https://github.com/telekom-security/tpotce.git
cd tpotce

# 3. Ejecutar el instalador (tipo HIVE: instalación completa con Kibana)
sudo ./install.sh
#    → instala Docker + Docker Compose
#    → mueve el SSH de administración del puerto 22 al 64295
#      (libera el 22 para el honeypot Cowrie)
#    → descarga 34 imágenes Docker
#    → genera certificados SSL para la Web UI

# 4. Reinicio obligatorio para aplicar los cambios
sudo reboot

# 5. Verificar el estado de los contenedores
dps    # atajo de T-Pot: muestra los 35 contenedores y su salud
```

Tras la instalación, la interfaz web queda accesible en `https://34.175.139.110:64297` con acceso al **Attack Map** en tiempo real, **Kibana**, **CyberChef** y **SpiderFoot**.

---

## 🐛 Problemas Encontrados y Soluciones

> Esta es la parte que no aparece en ningún tutorial: los problemas reales del despliegue.

### Bug 1 — El servicio systemd autodestruía los datos 🔥

**Síntoma:** tras cada reinicio de la máquina, **todos los datos capturados desaparecían**. Kibana arrancaba vacío.

**Causa:** el servicio `tpot.service` ejecutaba `docker compose down -v` al arrancar. La flag `-v` **elimina los volúmenes de Docker**, que es donde Elasticsearch almacena todos los datos. El propio sistema se autodestruía en cada arranque.

**Solución:**
```bash
sudo systemctl disable tpot.service    # desactivar el servicio problemático
# Gestión manual de los contenedores a partir de aquí:
docker compose up -d
```

### Bug 2 — exim4 bloqueaba el puerto 25

**Síntoma:** el honeypot Mailoney no podía arrancar.

**Causa:** Debian 12 trae preinstalado el servidor de correo `exim4`, que ocupa el puerto 25 — justo el que necesita Mailoney.

**Solución:**
```bash
sudo systemctl disable exim4 && sudo systemctl stop exim4
```

### Bug 3 — Kibana sin dashboards

**Síntoma:** al entrar en Kibana, aparecía vacío ("crea tu primer dashboard"), consecuencia de los `down -v` previos que habían borrado los objetos guardados.

**Solución — recuperación manual:**
1. Crear el **data view** en Kibana → `Stack Management` → `Data Views` → patrón `logstash-*` con campo temporal `@timestamp`.
2. Localizar el fichero de dashboards de fábrica en el repositorio:  
   `~/tpotce/.../etc/objects/kibana_export.ndjson`
3. Importarlo en Kibana → `Stack Management` → `Saved Objects` → `Import` → sobrescribir todo.

Tras esto, aparecieron todos los dashboards (general, Cowrie, Dionaea, Suricata...) con los datos ya indexados en Elasticsearch.

---

## 📊 Resultados Globales

Tras **16 días** de captura continua (12–28 junio 2026):

| Honeypot | Ataques | Detalle |
|:--|--:|:--|
| **Cowrie** (SSH/Telnet) | 465.000+ | 2.746 IPs únicas · 85 logins exitosos |
| **Honeytrap** | 457.000+ | Conexiones multi-puerto |
| **Dionaea** | 123.000+ | 6.578 ataques SMB (patrón EternalBlue) |
| **Heralding** | 75.000+ | 28 IPs · 97% del VNC desde una sola IP |
| **H0neytr4p** | 12.000+ | Honeypot web |
| **Suricata** (IDS) | 2.400.000+ | 13.659 IPs · 10+ CVEs detectados |

> ⚡ **El dato más impactante:** esto **no es** el servidor de un banco ni de una empresa importante. Es una máquina anónima, recién creada, sin ningún dato de valor. Y aun así recibió más de un millón de ataques en poco más de dos semanas. **Ninguna IP pública está segura ni un solo minuto.**

### Distribución geográfica

Los ataques procedieron de **más de 30 países**: Estados Unidos, Países Bajos, China, Rusia, Taiwán, Alemania, Francia, Corea del Sur...

> ⚠️ **Concepto clave — el problema de la atribución:** el país de origen **NO** es el país del atacante. Es solo la ubicación del servidor desde el que sale el tráfico. Los atacantes usan VPS alquilados, máquinas comprometidas de víctimas inocentes y cadenas de proxies. EEUU y Países Bajos lideran no por tener más hackers, sino por ser los países con más centros de datos baratos.

---

## 🔍 Análisis por Honeypot

### 🔐 Cowrie — El honeypot SSH/Telnet

SSH es la llave de entrada a cualquier servidor Linux, por eso es el **objetivo número uno**. Cowrie simula un servidor vulnerable, deja entrar a los atacantes y graba todo lo que hacen.

**Top credenciales intentadas:**

| Usuario | Intentos | | Contraseña | Intentos |
|:--|--:|:--|:--|--:|
| `root` | 1.031 | | `123456` | 97 |
| `admin` | 98 | | `3245gs5662d34` | 37 |
| `345gs5662d34` ⚠️ | 35 | | `1234` | 25 |
| `ubuntu` | 23 | | `password` | 24 |

> 🤖 La credencial `345gs5662d34` / `3245gs5662d34` es la **firma de la botnet Mirai** — viene incrustada en el código del malware. Verla confirma que hay bots de Mirai atacando.

**Los 85 logins exitosos** (credenciales que *funcionaron*):

| Usuario | Contraseña | Origen | Análisis |
|:--|:--|:--|:--|
| `root` | `h3c.com!` | 🇨🇳 China (222.170.175.95) | Contraseña por defecto de routers H3C |
| `viva` | `viva` | 🇺🇸 Azure (20.153.204.5) | Usuario = contraseña |
| `amsterdam` | `amsterdam123` | 🇮🇳 India | Patrón ciudad+números de filtraciones |
| `corpmail` | `corp` | 🇺🇸 Azure (20.153.204.5) | Usuario corporativo con contraseña débil |

**Los comandos ejecutados dentro** revelaron un patrón de reconocimiento **idéntico repetido 38 veces**:

```bash
uname -a              # sistema operativo y arquitectura
cat /proc/cpuinfo     # modelo y potencia de la CPU
free -m               # memoria RAM disponible
crontab -l            # tareas programadas (persistencia)
```

> 🧠 38 ejecuciones idénticas en el mismo orden exacto **no las hace un humano**. Es un bot con un playbook predefinido que mide si el servidor merece la pena para minar criptomonedas. Una cadena de montaje criminal automatizada.

**Malware descargado:** familia `ohshit` (compilada para 5 arquitecturas: x86, x86_64, MIPS, MIPSel, i686), un `sshd` falso (backdoor), `clean.sh` (borra rastros) y `redtail.arm7/arm8` (el cryptominer — el objetivo final).

---

### 🎭 Heralding — Credenciales multi-protocolo

75.000 ataques de **solo 28 IPs únicas**. El hallazgo estrella:

> 🇳🇱 Una sola IP holandesa (**5.83.143.40**, AS Joel Krause) fue responsable de **9.476 de los 9.773 ataques VNC** — el **97%** de todo el tráfico VNC. Un bot dedicado en exclusiva a buscar servidores VNC sin autenticación.

El histograma reveló una **campaña coordinada** con un pico sostenido del 8 al 22 de junio (~5.000 ataques diarios) seguido de una caída brusca: alguien encendió una botnet VNC durante dos semanas y la apagó.

---

### 🦠 Dionaea — La trampa para malware

123.000 ataques. El dato clave: **6.578 ataques al puerto 445 (SMB)**.

> 💣 El puerto 445 es el vector del ransomware **WannaCry** (2017), que explotaba la vulnerabilidad **EternalBlue** (MS17-010). Casi 10 años después y mil parches más tarde, los atacantes **siguen** buscando sistemas Windows con SMBv1 vulnerable. Rusia lidera estos ataques.

---

### 🛡️ Suricata — IDS de red

A diferencia de los honeypots (que atraen), un **IDS inspecciona todo el tráfico** y lo compara contra firmas de ataques conocidos. Procesó **2,4 millones de eventos** de 13.659 IPs.

**CVEs detectados en tiempo real:**

| CVE | Detecciones | Descripción |
|:--|--:|:--|
| **CVE-2006-2369** | **74.440** | VNC auth bypass — ¡vulnerabilidad de **hace 20 años**! |
| CVE-2019-11500 | 111 | Dovecot IMAP/POP3 buffer overflow |
| CVE-2023-46604 | 90 | Apache ActiveMQ RCE — vector de ransomware |
| CVE-2025-55182 | 59 | Exploit de 2025 — atacantes al día |

> 🧟 **CVE-2006-2369 detectado 74.440 veces destruye el mito de que solo los zero-days importan.** Los atacantes no necesitan vulnerabilidades nuevas cuando una cantidad enorme de sistemas jamás se actualiza. Pero también vimos exploits de 2025: hay que parchear lo viejo *y* vigilar lo nuevo.

### 🚨 Hallazgo avanzado: Cobalt Strike

Suricata detectó **10 eventos en el puerto 50050**, el puerto por defecto del *Team Server* de **Cobalt Strike** — una herramienta comercial de Red Team (5.000+ $/año) cuyas versiones pirateadas son usadas por los grupos de amenaza más peligrosos del mundo:

`APT29 (Cozy Bear, Rusia)` · `APT41 (China)` · `Lazarus Group (Corea del Norte)` · `LockBit` · `BlackCat` · `Cl0p`

> Detectar tráfico de Cobalt Strike es **alerta de máxima prioridad** en cualquier SOC. Confirma que ahí fuera no solo hay bots tontos, sino actores con recursos avanzados.

---

## 🧬 Análisis Forense del Malware Redtail

Se realizó análisis estático y dinámico del binario `redtail.arm7.elf`.

### VirusTotal (análisis estático)

| Resultado | |
|:--|:--|
| **Veredicto** | 🔴 **MALICIOUS 76/100** |
| **Clasificación** | CoinMiner / XMRig |
| **Formato** | ELF para arquitectura ARM |
| **Ofuscación** | Empaquetado con UPX |

### Joe Sandbox (análisis dinámico)

El *Behavior Graph* reveló la cadena de ataque completa en **4 fases**:

```
  ┌─────────────┐    ┌──────────────┐    ┌─────────────┐    ┌──────────────┐
  │ 1. EJECUCIÓN│───▶│2. PERSISTENCIA│──▶│ 3. EVASIÓN  │───▶│  4. IMPACTO  │
  │  redtail.elf│    │crontab @reboot│    │  iptables   │    │ Minado Monero│
  │   arranca   │    │ (sobrevive    │    │ (abre C2)   │    │  (CPU víctima)│
  │             │    │  reinicios)   │    │             │    │              │
  └─────────────┘    └──────────────┘    └─────────────┘    └──────────────┘
```

**Regla YARA** `JoeSecurity_Xmrig` detectó el motor XMRig en la memoria RAM del proceso.

> 💰 **¿Por qué mina Monero y no Bitcoin?** Monero es completamente anónimo (Bitcoin es trazable), resistente a la regulación y eficiente de minar con CPU. Un servidor genera solo 2–3 €/mes... pero multiplicado por miles de servidores infectados —con la electricidad y el hardware pagados por las víctimas— el negocio es enorme y sin apenas riesgo. Esto se llama **cryptojacking**.

---

## 🎯 Mapeo MITRE ATT&CK

El ataque completo se mapeó al framework **MITRE ATT&CK**, el lenguaje universal para describir técnicas de atacantes:

| Táctica | Técnica | Evidencia en el honeypot |
|:--|:--|:--|
| Initial Access | **T1078** — Valid Accounts | Credenciales por defecto (`root/h3c.com!`) |
| Execution | **T1059** — Command Scripting | Scripts automatizados de reconocimiento |
| Persistence | **T1053** — Scheduled Task | `crontab @reboot /tmp/redtail` |
| Defense Evasion | **T1027** — Obfuscated Files | Empaquetado UPX |
| Discovery | **T1082** — System Discovery | `uname`, `cpuinfo`, `free` |
| Impact | **T1496** — Resource Hijacking | Minado de Monero |

---

## 🕵️ Investigación OSINT de los Atacantes

Se investigaron las principales IPs mediante **AbuseIPDB** y **VirusTotal**:

| IP | Origen | Reportes AbuseIPDB | Actividad |
|:--|:--|:--|:--|
| **5.83.143.40** | 🇳🇱 Joel Krause | 1.956 (100% abuse) | 73.479 ataques VNC |
| **62.148.227.117** | 🇷🇺 Feo Prest / Rostelecom | 1.131 | 71.952 ataques SMB |
| **45.205.1.5** | 🇧🇷 Vpsvault.host | 9.500 (100% abuse) | Hosting de botnets |

> 🔎 **Reflexión final:** todas estas IPs están perfectamente documentadas y reportadas públicamente. Cualquiera puede consultarlas. Y sin embargo, **siguen operando** día tras día. Conocer al atacante no es lo mismo que poder detenerlo — por eso la defensa (parchear, contraseñas fuertes, monitorizar) sigue siendo nuestra mejor arma.

---

## ✅ Conclusiones

1. **⚡ Internet ataca desde el segundo cero.** En menos de 5 minutos llegaron los primeros ataques. Ninguna IP nueva está segura.
2. **🤖 El 99% de los ataques son automatizados.** Mismos comandos, mismo orden, mismos diccionarios. Los bots no duermen.
3. **🔑 Las credenciales por defecto son *el* problema.** `root/root`, `admin/1234`, `postgres/postgres`: la puerta más explotada del mundo.
4. **🧟 Los exploits viejos siguen activos.** Un CVE de 2006 detectado 74.440 veces en 2026. Parchear es la defensa más básica y la más descuidada.
5. **💰 El objetivo final es económico.** Cryptomining, ransomware, VNC sin autenticación: todo se monetiza.
6. **☁️ La resiliencia cloud es una habilidad profesional.** Oracle falló; migrar a GCP enseñó más que cualquier tutorial.

---

## 🚀 Próximos Pasos

- [ ] **Integración con Splunk SOC** — Universal Forwarder para correlación en tiempo real
- [ ] **Ingeniería inversa con Ghidra** — extracción de IOCs del malware en un entorno REMnux
- [ ] **Threat Intelligence con MISP** — compartir los IOCs capturados con la comunidad
- [ ] **Alertas automáticas en Kibana** — reglas para CVEs críticos, puerto 50050 y logins SSH exitosos
- [ ] **Despliegue multi-región** — sensores T-Pot en EEUU y Asia para comparar vectores por geografía

---

## 📁 Contenido del Repositorio

```
📦 proyecto-final-keepcoding
├── 📄 README.md                              ← este archivo
├── 📊 SOCEmpire_Presentacion_Final.pptx      ← presentación (36 diapositivas + guión)
├── 📑 SOCEmpire_Informe_Final.docx           ← informe ejecutivo completo (34 páginas)
├── 📁 data/
│   ├── cowrie.json                           ← eventos de Cowrie (SSH/Telnet)
│   ├── dionaea.json                          ← eventos de Dionaea (malware)
│   ├── heralding.json                        ← eventos de Heralding (credenciales)
│   └── suricata.json                         ← eventos de Suricata (IDS)
└── 📁 docs/
    ├── SOCEmpire_Guion_Slides.pdf            ← guión de la presentación por diapositiva
    └── SOCEmpire_Chuleta_Kibana_Discover.pdf ← chuleta de consultas KQL
```

> ℹ️ **Sobre los ficheros JSON:** son exportaciones de los eventos capturados por cada honeypot desde Elasticsearch. Contienen los datos en crudo que sustentan todo el análisis de este informe.

---

## 📖 Glosario

| Sigla | Significado | Qué es |
|:--|:--|:--|
| **APT** | Advanced Persistent Threat | Grupo de atacantes avanzado, normalmente patrocinado por un estado |
| **ASN** | Autonomous System Number | Identificador de una red grande en Internet (un ISP o proveedor de hosting) |
| **C2** | Command and Control | Servidor desde el que un atacante controla el malware que ha instalado |
| **CVE** | Common Vulnerabilities and Exposures | Identificador estándar de una vulnerabilidad pública conocida |
| **ELK** | Elasticsearch + Logstash + Kibana | Stack de almacenamiento, procesado y visualización de logs |
| **IDS** | Intrusion Detection System | Sistema que inspecciona el tráfico de red buscando ataques |
| **IOC** | Indicator of Compromise | Evidencia de un ataque: una IP, un hash, un dominio malicioso |
| **KQL** | Kibana Query Language | Lenguaje para filtrar datos en Kibana (`campo : valor`) |
| **RCE** | Remote Code Execution | Vulnerabilidad que permite ejecutar código en un sistema remoto |
| **SIEM** | Security Information and Event Management | Plataforma que centraliza y correlaciona logs de seguridad |
| **SMB** | Server Message Block | Protocolo de compartición de archivos de Windows (puerto 445) |
| **SOC** | Security Operations Center | Equipo que monitoriza y responde a incidentes de seguridad |
| **SSH** | Secure Shell | Protocolo para administrar servidores de forma remota (puerto 22) |
| **TTP** | Tactics, Techniques and Procedures | Cómo opera un atacante (su comportamiento, más difícil de cambiar que un IOC) |
| **VNC** | Virtual Network Computing | Protocolo de escritorio remoto (control visual de una máquina) |

---

<div align="center">

### 🎓 Proyecto Final · KeepCoding Bootcamp Ciberseguridad Ed.11

**Equipo SOC con Gofio** — Nicolás Navares Sánchez & Daniel Landeira

*Julio 2026*

<br>

*Desplegado en la nube. Atacado por el mundo. Analizado por nosotros.*

⭐ *Si este proyecto te ha parecido interesante, deja una estrella al repositorio.*

</div>
