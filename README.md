# SOC-Wazuh-Snort-Lab-
# 🛡️ SOC & XDR Home Lab: Wazuh SIEM + Snort NIDS Integration

## 📌 Project Overview
Este proyecto documenta el diseño, despliegue y afinación de un entorno de laboratorio de operaciones de seguridad (SOC). Se implementó una arquitectura XDR (Extended Detection and Response) unificando capacidades de HIDS (Host-based Intrusion Detection System) y NIDS (Network Intrusion Detection System), culminando con la configuración de respuestas activas automatizadas ante amenazas en la red.

El objetivo principal fue simular un entorno corporativo para la monitorización de endpoints, análisis de tráfico de red y respuesta a incidentes en tiempo real.

## 🏗️ Architecture & Environment
* **SIEM / Defender Node:** Kali Linux (Wazuh Manager + Snort 3)
* **Attacker / Victim Node:** Metasploitable 2 (Simulación de vulnerabilidades y ataques)
* **Network Segment:** Host-Only Adapter (Entorno aislado y controlado)

---

## 🚀 Key Implementations & Use Cases

### 1. Network Intrusion Detection (Snort 3 to Wazuh via Syslog)
Se desplegó Snort 3 como sensor de red para inspeccionar el tráfico en la interfaz `vboxnet0`.
* **Desarrollo de Reglas:** Creación de Custom Rules (`local.rules`) para detectar patrones de ataque específicos (ej. Inundación ICMP / Nmap Scans).
* **Ingesta de Logs:** Se configuró el pipeline para enviar las alertas de Snort directamente al Kernel de Linux a través de **Syslog**, evitando problemas de decodificación y asegurando la ingesta en tiempo real en el Dashboard de Wazuh.
* **Correlación:** Activación de las reglas 20101 (IDS Event) y 20151 (Multiple IDS Events) en el SIEM.

![Snort IDS Detection](assets/02-wazuh-syslog-ids.jpg)
*(Evidencia: Wazuh decodificando eventos de Snort a través de Syslog).*

### 2. Active Response (Automated Threat Mitigation)
Se configuró el motor de respuesta activa de Wazuh para mitigar ataques de fuerza bruta en progreso.
* **Vector de Ataque:** Ataque de fuerza bruta por SSH utilizando `Hydra`.
* **Detección:** El decodificador nativo de Wazuh identificó múltiples fallos de autenticación (`sshd: authentication failed`), elevando la severidad a Nivel 10.
* **Mitigación Automática:** Se modificó el archivo `ossec.conf` para vincular los eventos de Nivel 10 al script `firewall-drop`, bloqueando automáticamente la IP del atacante mediante *iptables/firewalld* e interrumpiendo el ataque.

![Active Response Config](assets/03-ossec-active-response.png)
*(Evidencia: Configuración del bloque <active-response> en ossec.conf).*

### 3. File Integrity Monitoring (FIM)
Implementación de defensas contra Ransomware y modificaciones no autorizadas.
* **Configuración:** Se desplegó el motor `Syscheck` para monitorizar en tiempo real los directorios que contienen datos sensibles (ej. `/passwords.txt`).
* **Auditoría:** Captura y alerta inmediata ante la alteración del archivo, registrando los cambios en los Hashes criptográficos (MD5, SHA-1, SHA-256) para análisis forense.

---

## 🛠️ Troubleshooting & Skills Demonstrated
Durante el despliegue del laboratorio, se aplicaron técnicas de resolución de problemas avanzadas (Troubleshooting), habilidades esenciales para el día a día en un equipo Blue Team:
* **Log Pipeline Debugging:** Uso de la herramienta `wazuh-logtest` para diagnosticar fallas en las tres fases de decodificación de logs de formato `snort-fast`.
* **File Permissions & Rootcheck:** Resolución de alertas de anomalías en el Host (Nivel 7) generadas por permisos inseguros en archivos de logs.
* **Time Synchronization:** Diagnóstico de pérdida de visibilidad de eventos debido a desincronización de zonas horarias (Timezones) entre el sensor NIDS y el visualizador del SIEM.

## 💻 Herramientas y Tecnologías
`Wazuh 4.x` | `Snort 3` | `Linux CLI` | `Bash Scripting` | `Syslog` | `Nmap` | `Hydra`
