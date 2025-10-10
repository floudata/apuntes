# Ejemplos Avanzados de Troubleshooting Linux

Basado en técnicas de administración con systemd, módulos del kernel y diagnóstico profundo.

---

## 1. Análisis Profundo de Rendimiento del Arranque

### Caso: Sistema que tarda más de 2 minutos en iniciar

**Diagnóstico completo:**

```bash
# Generar gráfico SVG del arranque completo
systemd-analyze plot > /tmp/boot-analysis.svg

# Ver los servicios más lentos
systemd-analyze blame

# Analizar la cadena crítica de dependencias
systemd-analyze critical-chain

# Obtener estadísticas detalladas
systemd-analyze
```

**Análisis e interpretación:**

```bash
# Si ves algo como esto en critical-chain:
# graphical.target @45.231s
# └─multi-user.target @45.230s
#   └─getty@tty1.service @40.123s +5.107s
#     └─system-getty@.slice @40.121s
#       └─system.slice @0.0s

# Significa que getty está esperando 5 segundos en algo.
# Investiga con:
sudo systemctl status getty@tty1.service
journalctl -u getty@tty1.service -n 50
```

**Solución de ejemplo - desactivar servicios innecesarios:**

```bash
# Ver todos los servicios habilitados
systemctl list-unit-files --state=enabled

# Desactivar servicios que no necesitas (ej: Bluetooth)
sudo systemctl disable bluetooth.service
sudo systemctl mask bluetooth.service  # Impide reactivación accidental

# Daemon reload y reiniciar para verificar
sudo systemctl daemon-reload
sudo systemctl reboot
```

---

## 2. Debugging de Servicios Personalizados que Fallan

### Caso: Tu servicio mi-app.service no inicia correctamente

**Script de servicio problemático:**

```ini
# /etc/systemd/system/mi-app.service
[Unit]
Description=Mi Aplicación Crítica
After=network.target

[Service]
Type=simple
User=appuser
ExecStart=/opt/mi-app/bin/start.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**Debugging paso a paso:**

```bash
# 1. Verificar estado básico
systemctl status mi-app.service

# 2. Ver los logs más recientes
journalctl -u mi-app.service -n 100 --no-pager

# 3. Seguir logs en tiempo real mientras reintentas
journalctl -u mi-app.service -f

# 4. Ver logs del kernel también (a veces los errores están ahí)
journalctl -k -b 0 | grep "mi-app\|start.sh"

# 5. Ejecutar el comando manualmente para ver errores directos
sudo -u appuser /opt/mi-app/bin/start.sh

# 6. Ver el PID del proceso si está ejecutándose
systemctl show mi-app.service -p MainPID

# 7. Si el PID está activo, rastrear sus llamadas al sistema
sudo strace -p <PID> -e trace=open,openat,read,write
```

**Mejora del servicio con mejor manejo de errores:**

```bash
sudo systemctl edit mi-app.service
```

Añade esto en `[Service]`:

```ini
[Service]
# Tiempo máximo antes de considerar el servicio fallido
TimeoutStartSec=30

# Reintentos mejorados
Restart=always
RestartSec=5s
StartLimitInterval=300
StartLimitBurst=5

# Captura mejor de salida
StandardOutput=journal
StandardError=journal
SyslogIdentifier=mi-app

# Aislar el servicio para mejor diagnóstico
PrivateTmp=yes
NoNewPrivileges=true

# Límites de recursos
LimitNOFILE=65536
LimitNPROC=4096

# Esperar a que la red esté completamente lista
After=network-online.target
Wants=network-online.target
```

Luego:

```bash
sudo systemctl daemon-reload
sudo systemctl restart mi-app.service
journalctl -u mi-app.service -f
```

---

## 3. Troubleshooting Avanzado de Módulos del Kernel

### Caso: Driver de GPU no funciona después de actualizar kernel

**Diagnóstico:**

```bash
# Ver información del hardware
lspci -vv | grep -A10 "VGA compatible"

# Ver qué módulos están cargados para GPU
lsmod | grep -E "nvidia|nouveau|amdgpu"

# Ver en tiempo real si hay conflictos de módulos
sudo dmesg | tail -50 | grep -i "gpu\|nouveau\|nvidia"

# Ver parámetros del módulo actual
modinfo nvidia | grep parm
```

**Problema típico: Nouveau vs Nvidia propietario**

```bash
# Verificar si nouveau está interfiriendo
lsmod | grep nouveau

# Ver los logs de carga del módulo
journalctl -b 0 | grep -i "nouveau\|nvidia" | tail -20

# Solución: crear blacklist
sudo nano /etc/modprobe.d/blacklist-nouveau.conf
```

Contenido:

```
# Deshabilitar driver de código abierto para usar NVIDIA propietario
blacklist nouveau
blacklist nvidiafb
options nouveau modeset=0
```

Luego:

```bash
# Regenerar initramfs para que los cambios apliquen al arranque
sudo update-initramfs -u -k all

# O en sistemas Red Hat:
sudo dracut --force

# Verificar que está en el initramfs
lsinitramfs /boot/initrd.img-$(uname -r) | grep blacklist

# Reiniciar
sudo reboot
```

**Verificar después del reinicio:**

```bash
# Confirmar que nouveau está deshabilitado
lsmod | grep nouveau  # No debe devolver nada

# Ver que nvidia está cargado
lsmod | grep nvidia

# Ver información detallada
nvidia-smi
```

### Caso: Touchpad no responde correctamente

**Diagnosis:**

```bash
# Ver información del dispositivo de entrada
xinput list
xinput list-props "Synaptics TouchPad"

# Ver si el módulo psmouse está cargado
lsmod | grep psmouse

# Ver parámetros disponibles del módulo
modinfo psmouse | grep parm

# Ver logs del módulo
dmesg | grep -i "psmouse\|trackpad" | tail -20
```

**Configurar parámetros permanentes:**

```bash
sudo nano /etc/modprobe.d/psmouse.conf
```

Ejemplo de configuración:

```
# Habilitar scrolling vertical y horizontal
options psmouse proto=imps

# O con más opciones avanzadas:
options psmouse proto=imps synaptics_intertouch=1
options psmouse elantech_smbus=1
options psmouse focaltech_detect=1
```

Luego:

```bash
# Método 1: Reconstruir initramfs y reiniciar (más seguro)
sudo update-initramfs -u
sudo reboot

# Método 2: Descargar y recargar módulo en caliente (si el módulo lo permite)
sudo modprobe -r psmouse
sudo modprobe psmouse
```

---

## 4. Análisis Avanzado de Logs con Journalctl

### Caso: Errores misteriosos ocurren aleatoriamente

**Búsqueda de patrones:**

```bash
# Ver todos los errores de los últimos 7 días ordenados por frecuencia
journalctl --since "7 days ago" -p err -o short-monotonic | \
  awk '{print $3, $4, $5, $6}' | sort | uniq -c | sort -rn

# Seguir errores en tiempo real
journalctl -p err -f

# Ver errores de un rango de tiempo específico
journalctl --since "2025-10-01 10:00:00" --until "2025-10-01 11:00:00" -p err

# Exportar logs para análisis en otra máquina
journalctl --since "30 days ago" > /tmp/system-logs.txt
```

### Caso: Servicio falla intermitentemente

**Investigación detallada:**

```bash
# Ver todos los inicios y paradas del servicio
journalctl -u nginx.service --no-pager | grep -E "Started|Stopped|Failed"

# Ver cambios de estado con timestamps
journalctl -u nginx.service -o verbose | grep -E "systemd|Unit nginx"

# Si el servicio reinicia constantemente, ver por qué falla
journalctl -u nginx.service -p err -f

# Correlacionar con otros eventos del sistema
journalctl -u nginx.service -u systemd-logind -u dbus --merge -f

# Ver el tiempo exacto de cada fallo
journalctl -u nginx.service --failures | head -20
```

**Script para monitorear fallos:**

```bash
#!/bin/bash
# monitor-service-failures.sh

SERVICE="nginx.service"
CHECK_INTERVAL=60  # segundos

while true; do
    STATUS=$(systemctl is-active $SERVICE)
    TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
    
    if [ "$STATUS" != "active" ]; then
        echo "[$TIMESTAMP] $SERVICE no está activo. Estado: $STATUS"
        
        # Capturar los últimos 30 logs
        echo "Últimos logs:"
        journalctl -u $SERVICE -n 30 --no-pager
        
        # Intentar reiniciar
        sudo systemctl restart $SERVICE
        
        # Esperar a que se estabilice
        sleep 5
        
        # Enviar alerta (configurable)
        # echo "Alerta: $SERVICE falló a las $TIMESTAMP" | \
        #   mail -s "Alerta de servicio" admin@example.com
    fi
    
    sleep $CHECK_INTERVAL
done
```

---

## 5. Troubleshooting de Red Avanzado

### Caso: Conexión de red intermitente

**Diagnóstico:**

```bash
# Ver estado de interfaces de red
ip link show
ip addr show

# Ver tabla de rutas
ip route show

# Ver estadísticas de paquetes por interfaz
ip -s link show

# Resolver problemas de DNS
systemctl status systemd-resolved
resolvectl status

# Ver logs de conectividad
journalctl -u systemd-networkd -f
journalctl -u NetworkManager -f

# Ping continuo con estadísticas
ping -c 100 8.8.8.8 | tail -4

# Rastrear ruta hacia un servidor
traceroute -m 20 example.com

# Ver conexiones activas y estado
sudo ss -tunapl | grep ESTABLISHED
sudo ss -tunapl | grep TIME_WAIT  # Conexiones en espera
```

**Análisis de paquetes en tiempo real:**

```bash
# Capturar paquetes de una interfaz específica
sudo tcpdump -i eth0 -n 'tcp port 80'

# Capturar a un archivo para análisis posterior
sudo tcpdump -i eth0 -w /tmp/traffic.pcap

# Ver estadísticas de interfaces
sudo ethtool -S eth0
sudo ethtool eth0  # Ver velocidad y duplex
```

**Problema: RF-Kill bloqueando Wi-Fi**

```bash
# Ver estado de bloqueos
rfkill list

# Ejemplo de salida:
# 0: phy0: Wireless LAN
#     Soft blocked: yes
#     Hard blocked: no

# Desbloquear
sudo rfkill unblock all

# Desbloquear solo Wi-Fi
sudo rfkill unblock wifi
```

---

## 6. Debugging de Procesos en Tiempo Real

### Caso: Un proceso consume demasiada CPU/memoria sin razón aparente

**Identificación:**

```bash
# Ver procesos ordenados por CPU
top -o %CPU -b -n 1 | head -20

# Ver procesos ordenados por memoria
top -o %MEM -b -n 1 | head -20

# Ver información detallada de un proceso específico
ps aux | grep nombre_proceso
ps -eLf | grep nombre_proceso  # Incluir threads

# Ver el árbol de procesos
pstree -p nombre_proceso
```

**Rastreo detallado:**

```bash
# Obtener PID del proceso problemático
PID=$(pgrep -f nombre_proceso)

# Ver llamadas al sistema en tiempo real
sudo strace -p $PID -o /tmp/strace.log

# Ver solo llamadas de entrada/salida de archivos
sudo strace -p $PID -e trace=open,openat,read,write,close

# Ver llamadas de red
sudo strace -p $PID -e trace=network

# Ver llamadas de memoria
sudo strace -p $PID -e trace=mmap,brk,mprotect

# Capturar todo con salida resumida
sudo strace -p $PID -c 2>&1 | tee /tmp/syscall-summary.txt
```

**Monitoreo de recursos específico:**

```bash
# Memoria del proceso en detalle
cat /proc/$PID/status | grep -E "VmRSS|VmSize|VmPeak"

# Ver mapeo de memoria del proceso
cat /proc/$PID/maps

# Ver descriptores de archivo abiertos
ls -lh /proc/$PID/fd/

# Ver estadísticas de I/O del proceso
cat /proc/$PID/io

# Monitoreo continuo con iotop
sudo iotop -o -b -n 5 -d 2  # 5 iteraciones, 2 segundos de intervalo
```

---

## 7. Troubleshooting de Initramfs

### Caso: No puedes arrancar después de cambios en módulos

**Verificación:**

```bash
# Ver qué hay en el initramfs actual
lsinitramfs /boot/initrd.img-$(uname -r) | head -50

# Buscar un módulo específico
lsinitramfs /boot/initrd.img-$(uname -r) | grep nombre_modulo

# Ver archivos de configuración
lsinitramfs /boot/initrd.img-$(uname -r) | grep "\.conf$"

# Extraer initramfs para inspeccionar manualmente
mkdir /tmp/initramfs
cd /tmp/initramfs
sudo cpio -id < /boot/initrd.img-$(uname -r)
```

**Regeneración correcta:**

```bash
# En Debian/Ubuntu
sudo update-initramfs -u -k all

# En Red Hat/CentOS/Fedora
sudo dracut --force

# Verificar que se regeneró correctamente
ls -lh /boot/initrd.img*

# Si algo falla, ver los logs de dracut
sudo dracut --verbose --force 2>&1 | tee /tmp/dracut.log
```

---

## 8. Automatización de Troubleshooting

### Script de diagnóstico automático

```bash
#!/bin/bash
# diagnose-system.sh - Script de diagnóstico integral

LOGFILE="/tmp/system-diagnosis-$(date +%Y%m%d-%H%M%S).log"

echo "=== Diagnóstico del Sistema Linux ===" | tee $LOGFILE
echo "Generado: $(date)" | tee -a $LOGFILE
echo "" | tee -a $LOGFILE

# Información del sistema
echo "=== Información del Sistema ===" | tee -a $LOGFILE
uname -a | tee -a $LOGFILE
lsb_release -a 2>/dev/null | tee -a $LOGFILE

# Análisis de arranque
echo -e "\n=== Análisis de Arranque ===" | tee -a $LOGFILE
systemd-analyze | tee -a $LOGFILE
echo "Top 10 servicios lentos:" | tee -a $LOGFILE
systemd-analyze blame | head -10 | tee -a $LOGFILE

# Estado de servicios críticos
echo -e "\n=== Estado de Servicios Críticos ===" | tee -a $LOGFILE
for service in ssh networking NetworkManager docker; do
    echo "--- $service ---" | tee -a $LOGFILE
    systemctl status $service 2>&1 | head -5 | tee -a $LOGFILE
done

# Errores recientes en logs
echo -e "\n=== Errores Recientes ===" | tee -a $LOGFILE
journalctl -p err --since "24 hours ago" | tail -20 | tee -a $LOGFILE

# Hardware
echo -e "\n=== Hardware ===" | tee -a $LOGFILE
echo "CPU:" | tee -a $LOGFILE
nproc | tee -a $LOGFILE
echo "Memoria:" | tee -a $LOGFILE
free -h | tee -a $LOGFILE
echo "Disco:" | tee -a $LOGFILE
df -h / | tee -a $LOGFILE

# Módulos cargados
echo -e "\n=== Módulos Problemáticos ===" | tee -a $LOGFILE
lsmod | wc -l | xargs echo "Total módulos cargados:" | tee -a $LOGFILE

# Red
echo -e "\n=== Red ===" | tee -a $LOGFILE
ip link show | tee -a $LOGFILE
ip route show | tee -a $LOGFILE

echo -e "\n=== Diagnóstico completado ===" | tee -a $LOGFILE
echo "Archivo de log: $LOGFILE"
```

---

## 9. Referencia Rápida de Comandos Avanzados

```bash
# === SYSTEMD ===
systemd-analyze blame              # Servicios lentos
systemd-analyze critical-chain     # Cadena de dependencias
systemd-analyze plot > boot.svg    # Gráfico de arranque
systemctl list-dependencies nginx  # Ver dependencias
systemctl show -p AllowIsolate nginx.service  # Ver propiedades

# === JOURNALCTL ===
journalctl -p err -f               # Errores en tiempo real
journalctl --since "1 hour ago" -u servicio
journalctl -u servicio -n 100      # Últimas 100 líneas
journalctl /usr/bin/comando        # Por ejecutable
journalctl _UID=1000               # Por usuario UID
journalctl --vacuum-size=100M      # Limpiar logs viejos

# === MÓDULOS ===
modinfo nombre                     # Info del módulo
lsmod | grep nombre                # Verificar carga
modprobe -r nombre                 # Descargar
modprobe nombre parámetro=valor    # Cargar con parámetros
depmod -a                          # Reconstruir dependencias

# === NETWORK ===
ip link show                       # Interfaces
ip -s link show                    # Estadísticas
ss -tunapl                         # Conexiones activas
sudo tcpdump -i eth0 -n 'port 80' # Capturar paquetes
netstat -rn                        # Tabla de rutas

# === PROCESOS ===
ps aux | grep nombre               # Ver proceso
pstree -p nombre                   # Árbol de procesos
top -o %CPU                        # Ordenar por CPU
cat /proc/PID/io                   # I/O del proceso
lsof -p PID                        # Archivos abiertos

# === STRACE ===
strace -p PID -e trace=open        # Rastrear llamadas
strace -p PID -c                   # Resumen de llamadas
strace -e trace=network cmd        # Trazar red
strace -o /tmp/trace.log cmd       # A archivo
```

---

## 10. Checklist de Troubleshooting Completo

- [ ] Verificar logs con `journalctl -f`
- [ ] Ver estado del servicio: `systemctl status`
- [ ] Revisar dependencias: `systemctl list-dependencies`
- [ ] Analizar arranque: `systemd-analyze blame`
- [ ] Rastrear llamadas: `strace` o `ltrace`
- [ ] Ver recursos: `top`, `free`, `df`
- [ ] Verificar red: `ip link`, `ss`, `ping`
- [ ] Revisar módulos: `lsmod`, `modinfo`
- [ ] Regenerar initramfs si cambió algo crítico
- [ ] Crear ticket con logs extraídos: `journalctl > logs.txt`