# 🛡️ Laboratorio de Seguridad en Red Corporativa & Monitoreo SIEM

Despliegue y hardening de un entorno de red corporativa segmentada a nivel perimetral, con enfoque en análisis de tráfico, auditoría de sistemas y detección de amenazas en tiempo real mediante un SOC doméstico (Wazuh SIEM/XDR).

---

## 🎯 Objetivo del repositorio

Documentar de forma real, práctica y progresiva el desarrollo de habilidades críticas en:

- **Networking y Defensa Perimetral:** Reglas de firewall e inspección de tráfico entre segmentos.
- **Administración de Sistemas Linux:** Hardening de servicios, gestión de daemons y análisis forense de archivos de log.
- **Detección de Amenazas (SOC):** Ingesta, parseo y centralización de eventos sin agentes mediante Syslog.
- **Pentesting Controlado:** Simulación de vectores de ataque para validación de reglas de detección.

Todo el contenido está basado en ejecución real en laboratorio propio, con evidencia técnica auditable.

---

## 🧭 Ruta de Aprendizaje & Estado del Proyecto

`Security+` → `SOC / Blue Team` → `PNPT` → `CPTS` → `OSCP`

### 📈 Progreso Actual:
- **Fase 1 — Fundamentos de Infraestructura**
  - [x] Configuración e interconexión del laboratorio perimetral (VMware + pfSense)
  - [x] Administración básica de sistemas Linux (Auditoría de servicios y logs vía CLI)
  - [x] Despliegue de Core SIEM/XDR (Wazuh Manager, Indexer y Dashboard)
- **Fase 2 — Operaciones de Seguridad (SOC) 🚀 (En Curso)**
  - [x] Ingesta remota de eventos sin agente vía Syslog (UDP/514)
  - [ ] Análisis de protocolos y detección de escaneos de red (Nmap)
  - [ ] Creación de reglas y decodificadores personalizados en Wazuh
  - [x] Ingesta remota de eventos sin agente vía Syslog (UDP/514) -> [Ver Laboratorio Detallado e Imágenes](./screenshots/alerta-de-logs-wazuh/README.md)
---

## 🧪 Arquitectura del Laboratorio

El laboratorio utiliza **pfSense** como núcleo para segmentar el tráfico entre zonas de seguridad independientes, aislando los activos críticos del SOC de las superficies de ataque:

| Zona   | Subred            | Interfaz VM  | Función               |
| :----- | :---------------- | :----------- | :-------------------- |
| WAN    | `192.168.80.X`    | VMnet8 (NAT) | Salida a Internet     |
| LAN    | `192.168.10.0/24` | VMnet4       | Usuarios internos     |
| DMZ    | `192.168.20.0/24` | VMnet3       | Servidores públicos   |
| ATTACK | `10.20.20.0/24`   | VMnet6       | Máquinas de auditoría |
| SOC    | `10.30.30.0/24`   | VMnet7       | Monitoreo y SIEM      |




                INTERNET
                    │
              [ pfSense FW ]
      ┌─────────────┼─────────────┬─────────────┐
      │             │             │             │
    [LAN]         [DMZ]       [ATTACK]       [SOC]
  192.168.10.0  192.168.20.0   10.20.20.0    10.30.30.0

---

## 🛠️ Tecnologías y Herramientas

| Categoría      | Herramientas                            |
| :------------- | :-------------------------------------- |
| Virtualización | VMware Workstation Pro 17 |
| Sistemas       | Debian, Kali Linux, Metasploitable |
| Firewall / Net | pfSense, nmap, tcpdump, Wireshark |
| Defensa / SIEM | Wazuh (Manager, Indexer, Dashboard)     |

---

## ⚙️ Desafíos Técnicos Resueltos (Hardening de Infraestructura)

Durante el pipeline de integración entre los componentes de la red, se identificaron y solucionaron los siguientes problemas de comunicación:

### 1. Filtrado Estricto de Hosts en pfSense
* **Problema:** Los eventos Syslog (`UDP/514`) procedentes de zonas externas (ATTACK/DMZ) eran descartados por las políticas por defecto del firewall.
* **Solución:** Se implementó una regla de pase explícita en pfSense restringiendo el destino mediante una directiva de host único (`Single Host`), mapeando directamente la IP estática del Wazuh Manager (`10.30.30.10`) y evitando configuraciones de red inseguras.

### 2. Apertura del Socket Remoto en Wazuh Manager
* **Problema:** El daemon de Wazuh no procesaba paquetes externos debido a una configuración restrictiva local en el bloque `<remote>`.
* **Solución:** Se realizó el hardening del archivo `/var/ossec/etc/ossec.conf` modificando el parámetro de escucha `<allowed-ips>` a un rango de subred amplio (`0.0.0.0/0`), habilitando el socket de escucha UDP de manera exitosa.

### 3. Debugging Forense mediante Logs Históricos
* **Problema:** Los eventos que no matcheaban con firmas de seguridad preexistentes eran descartados por el motor sin dejar registro indexado.
* **Solución:** Se forzó la directiva `<logall>yes</logall>` para activar el almacenamiento histórico crudo en `/var/ossec/logs/archives/archives.log`. Esto permitió validar mediante CLI (`tail -n 20`) que las tramas Syslog inyectadas desde hosts remotos impactaban correctamente en el backend del servidor.

---

## 🔬 Caso de Uso: Simulación e Indexación de Ataques SSH

Para comprobar el correcto funcionamiento de la arquitectura (Host Atacante -> Firewall -> SIEM -> Dashboard), se realizó una inyección remota simulando un ataque de fuerza bruta por SSH hacia una cuenta crítica:

```bash
echo "<13> May 23 23:58:00 kali-soc sshd[12345]: Failed password for invalid user root from 192.168.20

---


📬 Contacto
 LinkedIn: https://www.linkedin.com/in/victor-enrique-molina-b534ba3a6/

 Email: molina.victor.segurity@gmail.com