# Tutorial Avanzado de Administración de Linux: systemd, Módulos y Diagnóstico (Versión Extendida)

Este tutorial se enfoca en técnicas avanzadas para gestionar servicios con `systemd` y manipular módulos del kernel con `modprobe`, culminando en un escenario de diagnóstico práctico. Cada sección ha sido ampliada con ejemplos adicionales.

---

## 1. Maestría en `systemd`: Más Allá de `start` y `stop`

Gestionar `systemd` eficientemente significa ir más allá de los comandos básicos. Aquí veremos cómo analizar el arranque, personalizar unidades y usar el journal como un profesional.

### Analizando el Proceso de Arranque

Si tu sistema tarda en arrancar, `systemd` te da las herramientas para descubrir al culpable.

* **Ejemplo 1 (Original): Identificar servicios lentos.**
    Usa `blame` para ver una lista ordenada de los servicios más lentos.
    ```bash
    systemd-analyze blame | head -n 10
    ```
    *Salida de ejemplo:*
    ```
    10.123s networking.service
    5.432s plymouth-quit-wait.service
    ...
    ```

* **Ejemplo 2 (Adicional): Visualizar la cadena de dependencias críticas.**
    A veces, un servicio no es lento por sí mismo, sino porque está esperando a otro. `critical-chain` te muestra esta dependencia.
    ```bash
    systemd-analyze critical-chain
    ```
    *Salida de ejemplo:*
    ```
    graphical.target @21.300s
    └─multi-user.target @21.299s
      └─docker.service @18.123s +3.175s
        └─network-online.target @18.121s
          └─networking.service @7.998s +10.123s
    ```

    **Análisis:** En este caso, `docker.service` tardó 3.1 segundos en iniciar, pero no pudo empezar hasta que `network-online.target` estuvo listo. Este a su vez tuvo que esperar a `networking.service`, que fue el verdadero cuello de botella con 10.1 segundos.

* **Ejemplo 3 (Nuevo): Graficar el arranque completo.**
    Para una visualización más detallada, genera un gráfico SVG del proceso de arranque:
    ```bash
    systemd-analyze plot > boot-analysis.svg
    ```
    Abre el archivo con tu navegador para ver una línea de tiempo completa con todas las dependencias.

### Personalizando Unidades de Servicio Correctamente

**Nunca edites los archivos de unidad directamente en `/lib/systemd/system`**. Usa `systemctl edit` para crear "drop-in snippets" de configuración.

* **Caso práctico 1 (Original): Aumentar el límite de archivos abiertos para Nginx.**
    ```bash
    sudo systemctl edit nginx.service
    ```
    Y en el editor, añade:
    ```ini
    [Service]
    LimitNOFILE=65536
    ```

* **Caso práctico 2 (Adicional): Hacer que un servicio se reinicie automáticamente si falla.**
    Imagina que tienes una aplicación crítica (`my-app.service`) que a veces se cierra inesperadamente. Quieres que `systemd` la reinicie automáticamente.
    ```bash
    sudo systemctl edit my-app.service
    ```
    Añade las siguientes directivas:
    ```ini
    # Contenido de /etc/systemd/system/my-app.service.d/override.conf
    [Service]
    Restart=on-failure
    RestartSec=5s
    ```
    **Explicación:**
    -   `Restart=on-failure`: Le dice a `systemd` que reinicie el servicio si termina con un código de error (es decir, si no se detiene limpiamente).
    -   `RestartSec=5s`: Especifica que `systemd` debe esperar 5 segundos antes de intentar reiniciar el servicio.

    No olvides recargar la configuración después de guardar: `sudo systemctl daemon-reload`.

* **Caso práctico 3 (Nuevo): Crear un servicio personalizado desde cero.**
    Supongamos que tienes un script de Python que necesita ejecutarse como servicio.
    ```bash
    sudo systemctl edit --force --full mi-script.service
    ```
    Añade la configuración completa:
    ```ini
    [Unit]
    Description=Mi Script de Python
    After=network.target

    [Service]
    Type=simple
    User=usuario
    WorkingDirectory=/home/usuario/mi-app
    ExecStart=/usr/bin/python3 /home/usuario/mi-app/script.py
    Restart=always
    RestartSec=10

    [Install]
    WantedBy=multi-user.target
    ```
    Luego habilita e inicia el servicio:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable mi-script.service
    sudo systemctl start mi-script.service
    ```

### Filtrado Avanzado con `journalctl`

El journal es una mina de oro si sabes cómo buscar.

* **Ejemplo 1 (Original): Filtrar por tiempo y prioridad.**
    ```bash
    # Muestra solo errores y advertencias de las últimas 6 horas
    journalctl -p warning..err --since "6 hours ago"
    ```

* **Ejemplo 2 (Adicional): Seguir el rastro de un ejecutable específico.**
    Supongamos que quieres ver todos los logs generados por cualquier proceso que venga del binario de `sshd`.
    ```bash
    journalctl /usr/sbin/sshd --since "yesterday"
    ```

* **Ejemplo 3 (Adicional): Rastrear un script por su PID.**
    Cuando ejecutas un script, puedes obtener su PID y luego usar `journalctl` para ver todo lo que ese proceso (y sus hijos) han registrado en el journal.
    ```bash
    # Inicia un script en segundo plano y obtiene su PID
    ./mi_script_largo.sh &
    MIPID=$!

    # Sigue en tiempo real todo lo relacionado con ese PID
    journalctl -f _PID=$MIPID
    ```

* **Ejemplo 4 (Nuevo): Filtrar por unidad y exportar a archivo.**
    Para analizar problemas específicos de un servicio:
    ```bash
    journalctl -u apache2.service --since "2025-10-01" --until "2025-10-10" > apache-logs.txt
    ```

* **Ejemplo 5 (Nuevo): Ver solo los mensajes de arranque más recientes.**
    ```bash
    journalctl -b -1  # Arranque anterior
    journalctl -b 0   # Arranque actual
    ```

### Targets: Controlando el Estado del Sistema

Los targets son análogos a los antiguos "runlevels" pero más flexibles.

* **Cambiar temporalmente de target:**
    ```bash
    sudo systemctl isolate multi-user.target  # Modo texto
    sudo systemctl isolate graphical.target   # Modo gráfico
    ```

* **Establecer el target predeterminado:**
    ```bash
    sudo systemctl set-default multi-user.target
    ```

* **Ver el target actual:**
    ```bash
    systemctl get-default
    ```

---

## 2. Gestión Avanzada de Módulos del Kernel

Aquí nos enfocaremos en cómo configurar y solucionar problemas con los módulos.

### Configurando Módulos con Parámetros

Muchos módulos aceptan parámetros para modificar su comportamiento.

* **Caso práctico 1 (Original): Activar el ahorro de energía en un driver Wi-Fi.**
    Descubre los parámetros con `modinfo iwlwifi | grep parm:`, y luego crea `/etc/modprobe.d/iwlwifi.conf` con:
    ```
    options iwlwifi power_save=1
    ```

* **Caso práctico 2 (Adicional): Solucionar un problema de audio en una laptop.**
    **Problema:** En una laptop Dell, los altavoces no suenan, solo los auriculares. El driver genérico `snd_hda_intel` no está seleccionando el "modelo" correcto para el hardware.
    1.  **Buscar modelos disponibles:** La documentación del kernel o foros en línea sugieren probar diferentes modelos.
    2.  **Crear archivo de configuración:**
        ```bash
        sudo nano /etc/modprobe.d/alsa-base.conf
        ```
    3.  **Añadir la opción:** Añade la línea para forzar el modelo correcto. El valor `dell-vostro` es un ejemplo; el valor real dependerá del hardware.
        ```
        options snd_hda_intel model=dell-vostro
        ```
    4.  Reinicia el sistema para que el módulo de sonido se cargue con este parámetro desde el principio.

* **Caso práctico 3 (Nuevo): Ajustar el comportamiento del touchpad.**
    Para deshabilitar el touchpad mientras escribes:
    ```bash
    sudo nano /etc/modprobe.d/psmouse.conf
    ```
    Añade:
    ```
    options psmouse proto=imps
    ```

### Solucionando Conflictos con `blacklist`

La directiva `blacklist` es crucial cuando dos módulos compiten por el mismo dispositivo.

* **Caso práctico 1 (Original): Driver `nouveau` vs. `nvidia`.**
    Crea `/etc/modprobe.d/blacklist-nouveau.conf` para deshabilitar el driver de código abierto y permitir que el propietario funcione correctamente.
    ```
    blacklist nouveau
    options nouveau modeset=0
    ```

* **Caso práctico 2 (Adicional): Drivers de Wi-Fi Broadcom en conflicto.**
    **Problema:** Algunos chips Wi-Fi de Broadcom son soportados por el driver `b43` (incluido en el kernel) y por el driver propietario `wl`. A veces, el sistema intenta cargar `b43` por defecto, que puede no funcionar tan bien como `wl`.
    1.  **Instala el driver propietario `wl`.**
    2.  **Pon en lista negra los drivers conflictivos:** Crea el archivo `/etc/modprobe.d/blacklist-broadcom.conf`.
        ```
        blacklist b43
        blacklist b43legacy
        blacklist ssb
        blacklist bcma
        ```
    3.  **Asegúrate de que el driver correcto se cargue:** Puedes forzar la carga de `wl` al regenerar el `initramfs` o simplemente reiniciar. Después de reiniciar, `lsmod | grep wl` debería mostrar que el driver correcto está en uso.

### Cargando y Descargando Módulos Manualmente

A veces necesitas experimentar con módulos antes de hacer cambios permanentes.

* **Cargar un módulo temporalmente:**
    ```bash
    sudo modprobe nombre_modulo
    ```

* **Descargar un módulo:**
    ```bash
    sudo modprobe -r nombre_modulo
    ```

* **Ver información detallada de un módulo:**
    ```bash
    modinfo nombre_modulo
    ```

* **Cargar módulos automáticamente al inicio:**
    Añade el nombre del módulo a `/etc/modules`:
    ```bash
    echo "nombre_modulo" | sudo tee -a /etc/modules
    ```

---

## 3. Escenario de Diagnóstico: "Mi Wi-Fi dejó de funcionar"

Pongamos todo en práctica. Has actualizado el kernel y ahora tu Wi-Fi no se conecta.

**Paso 1: Identificar el hardware y el driver esperado.**
```bash
lspci -knn | grep -iA3 network
```

Esto te dirá qué driver debería estar en uso (ej: `iwlwifi`).

**Paso 2: Comprobar si el módulo está cargado.**

```bash
lsmod | grep iwlwifi
```

Si no hay salida, el módulo no está cargado.

**Paso 3: Revisar los logs del kernel para buscar la causa raíz.**

```bash
journalctl -k -b 0 | grep iwlwifi
```

  * **Posible Error A (Firmware):** `iwlwifi 0000:02:00.0: firmware: failed to load iwl-debug-yoyo.bin (-2)`

      * **Diagnóstico:** El driver no encuentra el archivo de firmware que necesita.
      * **Solución:** Reinstalar el paquete `linux-firmware` (o el específico de tu distribución) y reiniciar. `sudo apt install --reinstall linux-firmware`.

  * **Posible Error B (RF-Kill):** `iwlwifi: RF_KILL detected. Restarting.`

      * **Diagnóstico:** El hardware está deshabilitado, ya sea por un interruptor físico en la laptop o por software.
      * **Solución:**
        1.  Verifica los interruptores físicos/teclas de función (Fn + Tecla de Wi-Fi).
        2.  Usa la herramienta `rfkill` para comprobar el estado.
            ```bash
            rfkill list
            ```
            *Salida de ejemplo:*
            ```
            0: phy0: Wireless LAN
                Soft blocked: yes 
                Hard blocked: no 
            ```
        3.  Si está "Soft blocked", desbloquéalo:
            ```bash
            sudo rfkill unblock all
            ```

**Paso 4: Verificar que no esté en una lista negra.**
Si los logs no muestran nada, asegúrate de que no hayas deshabilitado el módulo por error en el pasado.

```bash
grep -r iwlwifi /etc/modprobe.d/
```

Si encuentras una línea como `blacklist iwlwifi`, elimínala y reconstruye el initramfs (`sudo update-initramfs -u`) antes de reiniciar.

**Paso 5: Comprobar la configuración de red.**
Verifica que NetworkManager o el gestor de red esté funcionando:
```bash
systemctl status NetworkManager
```

**Paso 6: Reintentar la conexión manualmente.**
```bash
sudo ip link set wlan0 up
sudo iwlist wlan0 scan | grep ESSID
```

---

## 4. Técnicas Avanzadas de Troubleshooting

### Usando `strace` para Depurar Procesos

Cuando un programa falla de forma misteriosa, `strace` muestra todas las llamadas al sistema que realiza.

**Ejemplo: Depurar por qué un comando falla.**
```bash
strace -e openat ls /directorio/inexistente
```

Esto mostrará exactamente qué archivos intenta abrir `ls` y dónde falla.

### Monitoreo de Recursos en Tiempo Real

* **Uso de CPU por proceso:**
    ```bash
    top -o %CPU
    ```

* **Uso de memoria detallado:**
    ```bash
    free -h
    cat /proc/meminfo
    ```

* **Monitoreo de I/O de disco:**
    ```bash
    sudo iotop
    ```

* **Analizar conexiones de red activas:**
    ```bash
    sudo netstat -tunapl
    # o con ss (más moderno)
    sudo ss -tunapl
    ```

### Gestionando el initramfs

El initramfs es crucial para el arranque. Si modificas módulos o configuraciones importantes, necesitas regenerarlo.

**Regenerar initramfs (Debian/Ubuntu):**
```bash
sudo update-initramfs -u
```

**Regenerar initramfs (Red Hat/CentOS/Fedora):**
```bash
sudo dracut --force
```

---

## 5. Automatización y Scripts de Mantenimiento

### Script de Limpieza Automática

Crea un servicio que limpie logs antiguos semanalmente:

```bash
sudo nano /usr/local/bin/cleanup-logs.sh
```

Contenido del script:
```bash
#!/bin/bash
journalctl --vacuum-time=30d
find /var/log -name "*.log" -type f -mtime +30 -delete
apt-get autoremove -y
apt-get autoclean
```

Dale permisos de ejecución:
```bash
sudo chmod +x /usr/local/bin/cleanup-logs.sh
```

Crea un timer de systemd:
```bash
sudo systemctl edit --force --full cleanup-logs.timer
```

```ini
[Unit]
Description=Limpieza semanal de logs

[Timer]
OnCalendar=weekly
Persistent=true

[Install]
WantedBy=timers.target
```

Y el servicio correspondiente:
```bash
sudo systemctl edit --force --full cleanup-logs.service
```

```ini
[Unit]
Description=Ejecutar limpieza de logs

[Service]
Type=oneshot
ExecStart=/usr/local/bin/cleanup-logs.sh
```

Habilita el timer:
```bash
sudo systemctl enable cleanup-logs.timer
sudo systemctl start cleanup-logs.timer
```

---

## 6. Conclusión y Recursos Adicionales

Este tutorial cubre aspectos avanzados de administración de Linux que te permitirán diagnosticar y resolver problemas complejos del sistema. Las habilidades con `systemd`, gestión de módulos y análisis de logs son fundamentales para cualquier administrador de sistemas.

### Recursos Recomendados

* **Documentación oficial de systemd:** https://www.freedesktop.org/wiki/Software/systemd/
* **Linux Kernel Module Programming Guide**
* **ArchWiki:** Una de las mejores fuentes de documentación técnica de Linux
* **man pages:** No olvides `man systemd.unit`, `man journalctl`, `man modprobe.d`

### Comandos de Referencia Rápida

```bash
# systemd
systemctl status servicio.service
systemctl enable --now servicio.service
systemctl daemon-reload
systemd-analyze blame
journalctl -u servicio.service -f

# Módulos
lsmod
modinfo nombre_modulo
sudo modprobe nombre_modulo
sudo modprobe -r nombre_modulo

# Diagnóstico
dmesg | tail
journalctl -k -b 0
lspci -k
rfkill list
ip link show
```

¡Practica estos comandos en un entorno seguro y verás cómo tu dominio de Linux alcanza un nuevo nivel!