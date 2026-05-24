# 🔬 Práctica SOC: Ingesta de Logs Remotos y Hardening en Wazuh SIEM

Este documento detalla la simulación de un ataque/evento operativo externo desde un host de auditoría hacia el SIEM, atravesando el firewall perimetral pfSense.

## 🛠️ Desafíos de Infraestructura Resueltos

### 1. Filtrado Estricto de Hosts en pfSense
* **Problema:** Los eventos Syslog (`UDP/514`) externos eran descartados por las políticas por defecto del firewall.
* **Solución:** Se implementó una regla de pase en pfSense restringiendo el destino mediante la directiva `Single Host` hacia la IP del Wazuh Manager (`10.30.30.10`).

### 2. Apertura del Socket Remoto en Wazuh Manager
* **Problema:** El daemon de Wazuh no procesaba paquetes externos por restricciones en el bloque `<remote>`.
* **Solución:** Se editó el archivo `/var/ossec/etc/ossec.conf` cambiando la directiva `<allowed-ips>` al rango amplio `0.0.0.0/0`.

### 3. Debugging Forense mediante Logs Históricos
* **Problema:** Los eventos que no matcheaban con firmas de seguridad preexistentes eran descartados por el motor sin indexarse en la web.
* **Solución:** Se forzó la directiva `<logall>yes</logall>` para auditar en tiempo real mediante CLI (`tail -n 20`) el archivo `/var/ossec/logs/archives/archives.log`.

---

## 🚀 Ejecución del Caso de Uso

Se realizó una inyección remota simulando tramas Syslog de alertas operativas:

```bash
echo "<13> May 23 23:55:00 kali-soc sshd[123]: test wazuh :alert en tiempo real solucionado" | nc -u 10.30.30.10 514