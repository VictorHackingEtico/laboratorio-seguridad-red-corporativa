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

### 📈 Progreso Actual

- [x] **Fase 1 — Fundamentos de Infraestructura**
  - [x] Configuración e interconexión del laboratorio perimetral (VMware + pfSense)
  - [x] Administración básica de sistemas Linux (auditoría de servicios y logs vía CLI)
  - [x] Despliegue de Core SIEM/XDR (Wazuh Manager, Indexer y Dashboard)
- [x] **Fase 2 — Operaciones de Seguridad (SOC)** 🚀 *(En Curso)*
  - [x] Ingesta remota de eventos sin agente vía Syslog (UDP/514) → [Ver laboratorio detallado](./screenshots/alerta-de-logs-wazuh/README.md)
  - [ ] Análisis de protocolos y detección de escaneos de red (Nmap)
  - [ ] Creación de reglas y decodificadores personalizados en Wazuh
- [ ] **Fase 3 — Pentesting Controlado** *(Próximamente)*
  - [ ] Explotación de servicios vulnerables en red interna (Metasploitable)
  - [ ] Validación end-to-end: ataque simulado → alerta en SIEM → análisis forense

---

## 🧪 Arquitectura del Laboratorio

El laboratorio utiliza **pfSense** como núcleo para segmentar el tráfico entre zonas de seguridad independientes, aislando los activos críticos del SOC de las superficies de ataque:

| Zona   | Subred            | Interfaz VM  | Función               |
|--------|-------------------|--------------|----------------------|
| WAN    | `192.168.80.X`    | VMnet8 (NAT) | Salida a Internet    |
| LAN    | `192.168.10.0/24` | VMnet4       | Usuarios internos    |
| DMZ    | `192.168.20.0/24` | VMnet3       | Servidores públicos  |
| ATTACK | `10.20.20.0/24`   | VMnet6       | Máquinas de auditoría|
| SOC    | `10.30.30.0/24`   | VMnet7       | Monitoreo y SIEM     |

```
              INTERNET
                  │
            [ pfSense FW ]
   ┌──────────────┼──────────────┬──────────────┐
   │              │              │              │
 [LAN]          [DMZ]        [ATTACK]         [SOC]
192.168.10.0  192.168.20.0  10.20.20.0     10.30.30.0
```

---

## 🛠️ Tecnologías y Herramientas

| Categoría       | Herramientas                        |
|-----------------|-------------------------------------|
| Virtualización  | VMware Workstation Pro 17           |
| Sistemas        | Debian, Kali Linux, Metasploitable  |
| Firewall / Red  | pfSense, nmap, tcpdump, Wireshark   |
| Defensa / SIEM  | Wazuh (Manager, Indexer, Dashboard) |

---

## ⚙️ Desafíos Técnicos Resueltos (Hardening de Infraestructura)

Durante la integración entre los componentes de la red, se identificaron y solucionaron los siguientes problemas:

### 1. Filtrado Estricto de Hosts en pfSense

- **Problema:** Los eventos Syslog (`UDP/514`) procedentes de zonas externas (ATTACK/DMZ) eran descartados por las políticas por defecto del firewall.
- **Solución:** Se implementó una regla de pase explícita en pfSense restringiendo el destino mediante una directiva de host único (`Single Host`), mapeando directamente la IP estática del Wazuh Manager (`10.30.30.10`).

### 2. Apertura del Socket Remoto en Wazuh Manager

- **Problema:** El daemon de Wazuh no procesaba paquetes externos debido a una configuración restrictiva en el bloque `<remote>`.
- **Solución:** Se editó `/var/ossec/etc/ossec.conf` modificando el parámetro `<allowed-ips>` al rango `0.0.0.0/0`, habilitando el socket de escucha UDP correctamente.

### 3. Debugging Forense mediante Logs Históricos

- **Problema:** Los eventos que no coincidían con firmas preexistentes eran descartados por el motor sin dejar registro indexado.
- **Solución:** Se activó la directiva `<logall>yes</logall>` para almacenar todos los eventos en crudo en `/var/ossec/logs/archives/archives.log`, permitiendo validar via CLI (`tail -f`) que las tramas Syslog de hosts remotos llegaban correctamente al backend.

---

## 🔬 Caso de Uso: Simulación e Indexación de Ataque SSH

Para verificar el flujo completo (Host Atacante → pfSense → Wazuh SIEM → Dashboard), se ejecutó una inyección remota simulando un ataque de fuerza bruta SSH:

```bash
# Inyección de trama Syslog desde host atacante (Kali Linux)
echo "<13> May 23 23:58:00 kali-soc sshd[12345]: Failed password for invalid user root from 10.20.20.50 port 4444 ssh2" | nc -u 10.30.30.10 514
```

**Resultado:** El evento fue recibido, procesado y visualizado en el Dashboard de Wazuh en tiempo real, confirmando el correcto funcionamiento de toda la cadena de detección.

📸 [Ver evidencia con capturas de pantalla](./screenshots/alerta-de-logs-wazuh/README.md)

---

## 📁 Estructura del Repositorio

```
laboratorio-seguridad-red-corporativa/
├── fase-1-fundamentos/         # Configuración inicial de infraestructura
├── laboratorio/                # Archivos de configuración y scripts del lab
├── screenshots/
│   └── alerta-de-logs-wazuh/  # Evidencia técnica: ingesta Syslog en Wazuh
└── README.md
```

---

## 📬 Contacto

- 🔗 **LinkedIn:** [Victor Enrique Molina](https://www.linkedin.com/in/victor-enrique-molina-b534ba3a6/)
- 📧 **Email:** molinavictor_03@yahoo.com.ar