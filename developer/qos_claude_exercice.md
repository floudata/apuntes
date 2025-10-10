# Guía Exhaustiva de Troubleshooting de QoS en Redes Cisco
## Diagnóstico y Resolución de Problemas Prácticos

---

## Introducción

El troubleshooting de QoS es un arte que requiere tanto comprensión teórica como habilidades prácticas de diagnóstico. A diferencia de los problemas de conectividad básica (donde un ping fallido indica una ruta rota), los problemas de QoS son sutiles. Un usuario puede tener conectividad perfecta pero experimentar una calidad de voz degradada o una aplicación crítica lenta durante los períodos de congestión. Este informe proporciona un conjunto completo de escenarios del mundo real con metodologías de diagnóstico paso a paso, comandos verificables y resoluciones prácticas.

---

## Parte I: Fundamentos del Troubleshooting de QoS

### Enfoque Metodológico

El troubleshooting de QoS sigue una metodología sistemática que se puede dividir en cinco pasos:

1. **Recopilación de síntomas** - ¿Qué reporta el usuario?
2. **Identificación del punto de congestión** - ¿Dónde ocurre realmente el problema?
3. **Análisis de clasificación y marcado** - ¿Se está clasificando correctamente el tráfico?
4. **Evaluación de políticas aplicadas** - ¿Las políticas están correctamente configuradas?
5. **Implementación y validación** - ¿Se resolvió el problema sin introducir otros?

### Herramientas Esenciales de Diagnóstico

Antes de entrar en escenarios específicos, es crítico dominar estos comandos:

```
show policy-map interface [nombre-interfaz]
show policy-map interface [nombre-interfaz] input
show queuing interface [nombre-interfaz]
show traffic-shape interface [nombre-interfaz]
show class-map
show access-lists
show ip nbar protocol-discovery interface [nombre-interfaz]
debug policy-map class
show interface [nombre-interfaz] | include dropped|input queue|output queue
```

---

## Parte II: Escenarios Reales de Troubleshooting

## Escenario 1: Llamadas VoIP de Mala Calidad en Horarios Pico

### Síntomas Reportados por el Usuario

- Las llamadas VoIP suenan "entrecortadas" o robóticas entre las 9 AM y 11 AM
- Los usuarios reportan ocasionalmente palabras o sílabas perdidas
- Fuera de las horas pico, las llamadas funcionan perfectamente
- Otros servicios (navegación web, email) parecen funcionar normalmente

### Análisis Inicial

El usuario experimenta jitter y pérdida de paquetes selectiva, posiblemente combinada con latencia variable. Esto sugiere que el tráfico de voz está siendo afectado por la congestión pero no está siendo priorizado adecuadamente.

### Paso 1: Verificar la Congestión en la Interfaz

Primero, confirmar que existe congestión:

```bash
Router# show interface Serial0/0/0 | include dropped|input queue|output queue
  Output queue: 127/40 (packets), 45692/3000 (bytes)
  5 minute output rate 9850000 bits/sec, 1234 packets/sec
    127 packets input, 3456 bytes, 2 drops
    234 packets output, 7890 bytes, 5 drops
```

**Análisis:** La cola de salida tiene 127 paquetes pendientes contra un límite de 40 (mostrada como 127/40), y hay 5 drops. Esto indica congestión moderada a severa. Sin embargo, esto no es suficiente para entender qué está causando el problema.

### Paso 2: Verificar si Existe una Política de QoS Aplicada

```bash
Router# show policy-map interface Serial0/0/0
  Service-policy output: WAN-EGRESS-POLICY

  Class map: VOICE (match-any)
    Match: ip dscp ef (46)
    Packets matched 15234
    QueueStatistics for queue 0
      Bandwidth requested 30% (3000 kbps) BW Sum 100%
      Bandwidth allocated 30% (3000 kbps)
      Output queue (Packets dropped/Packets output) = 45/154
      Tail drop/Red drops = 0/45
      Minimum Threshold 64, Maximum Threshold 64
      Mark-prob value = 1/10
```

**Hallazgo Crítico:** Hay 45 drops en la cola de voz durante el período de observación. Esto es inaceptable para VoIP. A pesar de que existe una política de QoS, la voz está siendo descartada.

### Paso 3: Verificar Clasificación y Marcado en la Fuente

El siguiente paso es verificar si el tráfico de voz realmente está siendo identificado:

```bash
Router# debug policy-map class
```

Alternativamente, usar NBAR para verificar qué tráfico realmente ingresa:

```bash
Router# show ip nbar protocol-discovery interface Serial0/0/0 | include voice|sip
  Protocol                  Packets       Bytes       % Packets    % Bytes
  rtp                       25641        2856403        45.2%       42.1%
  sip                       3421         456789         6.0%        6.7%
```

**Descubrimiento:** El NBAR muestra que hay tráfico RTP (voz) siendo transportado, pero solo el 6% del tráfico de señalización es SIP. Esto sugiere un posible problema: el tráfico de voz puede estar llegando con diferentes valores DSCP, no solo EF.

### Paso 4: Verificar la Configuración de Clasificación Actual

```bash
Router# show class-map VOICE
Class Map 'VOICE' (match-any)
  Match ip dscp ef
```

**Problema Identificado:** La clase VOICE solo coincide con DSCP EF (46), pero en muchas implementaciones, la señalización VoIP puede venir con CS3 (24), y algunos teléfonos IP o gateways heredados pueden marcar la voz con valores diferentes.

### Paso 5: Verificar desde el Teléfono IP

En este punto, es necesario verificar la configuración en el teléfono IP:

Acceder al teléfono Cisco IP (en este ejemplo, un modelo 7960):
- Presionar Settings > Network Configuration > QoS
- Verificar que Voice DSCP esté configurado como 46 (EF)
- Verificar que Signaling DSCP esté configurado como 24 (CS3)

**Descubrimiento Potencial:** El teléfono está marcando correctamente con DSCP EF, pero la clase-map del router que usa una ACL podría estar bloqueando algunos de estos paquetes por otra razón.

### Paso 6: Verificar la Frontera de Confianza

```bash
Router# show running-config interface Fa0/1
interface FastEthernet0/1
 description Conexión a switch de acceso
 switchport mode access
 switchport access vlan 100
 spanning-tree bpdufilter enable
 mls qos trust device cisco-phone
 mls qos trust dscp
```

**Análisis:** El comando `mls qos trust dscp` está habilitado, lo que significa que el switch de acceso confía en las marcas DSCP de los dispositivos. Esta es una configuración razonable para teléfonos Cisco, pero el problema es que esto ocurre en un switch Layer 2.

### Paso 7: Diagnosticar la Causa Raíz

Haciendo un seguimiento de los paquetes con una captura de tráfico:

```bash
Router# capture VOICE interface Serial0/0/0 match ipv4 any any
Router# capture VOICE interface Serial0/0/0 export packet-number 1 10 pcap bootflash:voice-capture.pcap
```

Analizando el archivo con un analizador (como Wireshark), se descubre:
- Los paquetes RTP salen del teléfono marcados correctamente con DSCP 46
- Pero después de pasar por el switch de acceso, algunos paquetes llegan al router con DSCP 0
- Esto ocurre aleatoriamente, particularmente cuando hay congestión en el switch

**Conclusión:** El problema NO es la configuración del router, sino una configuración incorrecta en el switch de acceso que está "re-marcando" algunos paquetes de voz a DSCP 0.

### Resolución

En el switch de acceso (Catalyst 2960):

```
switch# configure terminal
switch(config)# class-map match-any VOICE-TRAFFIC
switch(config-cmap)# match ip dscp ef
switch(config-cmap)# match ip dscp cs3
switch(config-cmap)# exit

switch(config)# policy-map PRESERVE-VOICE-QOS
switch(config-pmap)# class VOICE-TRAFFIC
switch(config-pmap-c)# set dscp ef
switch(config-pmap-c)# exit

switch(config)# interface range Fa0/1-48
switch(config-if-range)# service-policy input PRESERVE-VOICE-QOS
switch(config-if-range)# exit
```

Alternativamente, si el problema es que el switch está descartando paquetes en lugar de remarcar:

```
switch(config)# mls qos queue-set output 1 buffers 10 10 20 60
switch(config)# mls qos srr-queue output priorities 1 2 3 3
switch(config)# priority-queue out
```

### Validación Post-Resolución

```bash
Router# show policy-map interface Serial0/0/0
  Class map: VOICE (match-any)
    Packets matched 18234
    QueueStatistics for queue 0
      Output queue (Packets dropped/Packets output) = 0/18234
      Tail drop/Red drops = 0/0
```

**Éxito:** Los drops en la cola de voz ahora son cero. Las llamadas deben sonar claras sin cortes durante las horas pico.

### Lecciones Aprendidas

- QoS no es un problema aislado del router; debe considerarse de extremo a extremo
- La frontera de confianza debe ser colocada cuidadosamente
- Los switches de acceso Layer 2 pueden interferir con las marcas DSCP si no se configuran adecuadamente
- El troubleshooting requiere verificar múltiples puntos en la ruta del tráfico

---

## Escenario 2: Aplicación Crítica Lenta Durante la Congestión

### Síntomas Reportados por el Usuario

- Una aplicación de base de datos crítica (SAP/Oracle) se vuelve lenta entre las 11 AM y 1 PM
- Los usuarios reportan tiempos de respuesta de 5-10 segundos para consultas que normalmente toman 200 ms
- Otros servicios como email y navegación web no parecen afectados
- El problema es predecible y ocurre en la misma ventana de tiempo cada día

### Análisis Inicial

Esto sugiere que:
1. La aplicación de base de datos está siendo afectada selectivamente por congestión
2. Existen otros servicios que están consumiendo ancho de banda
3. La aplicación no está siendo priorizada apropiadamente, o existe un problema en la política de QoS actual

### Paso 1: Identificar el Tráfico de Base de Datos

Primero, identificar exactamente qué puertos y protocolos usa la aplicación:

```bash
Router# show ip nbar protocol-discovery interface Serial0/0/0 | include database
  Protocol                  Packets       Bytes       % Packets    % Bytes
  oracle                    45342        23456789     8.2%        34.5%
  tcp 1521                  2341         1234567      0.4%         1.8%
```

NBAR detectó que hay tráfico Oracle en el puerto 1521. Esto es útil, pero podría no ser exhaustivo.

### Paso 2: Verificar la Configuración de ACLs para la Aplicación Crítica

```bash
Router# show access-lists | include DATABASE
Extended IP access list DATABASE_APP
    10 permit tcp host 172.16.10.50 host 10.0.1.200 eq 1521
    15 permit tcp host 172.16.10.50 host 10.0.1.201 eq 1521
    20 permit tcp host 10.0.1.200 eq 1521 host 172.16.10.50
    25 permit tcp host 10.0.1.201 eq 1521 host 172.16.10.50
```

**Hallazgo:** La ACL está configurada y es específica, pero solo cubre direcciones de servidor específicas. Si hay múltiples servidores de base de datos o si la aplicación se ha movido a una dirección IP diferente, el tráfico podría no coincidir.

### Paso 3: Verificar Cómo se Está Clasificando Este Tráfico

```bash
Router# show policy-map interface Serial0/0/0 | include class|packets matched
Class map: VOICE (match-any)
  Packets matched 15234

Class map: CRITICAL-APP (match-any)
  Packets matched 2145

Class map: class-default
  Packets matched 12345678
```

**Descubrimiento:** Solo 2145 paquetes se están clasificando como CRITICAL-APP, pero NBAR mostró 45342 paquetes de Oracle. Esto significa que la mayoría del tráfico de base de datos se está clasificando como clase-default, no como CRITICAL-APP.

### Paso 4: Investigar por Qué No Coinciden

Hay varias razones por las que el tráfico puede no coincidir:

A) La ACL no es tan amplia como se pensaba
B) El tráfico está enrutándose de manera diferente
C) Hay múltiples flujos de tráfico de base de datos desde diferentes orígenes

Para diagnosticar, usar un filtro más amplio:

```bash
Router# configure terminal
Router(config)# access-list 101 permit tcp any any eq 1521
Router(config)# access-list 101 permit tcp any eq 1521 any
Router(config)# exit

Router# show ip nbar protocol-discovery interface Serial0/0/0 stats sorted
```

Luego, aplicar temporalmente esta ACL más amplia a la clase para ver cuánto tráfico coincide realmente:

```bash
Router(config)# class-map match-any CRITICAL-APP-TEST
Router(config-cmap)# match access-group 101
Router(config-cmap)# exit

Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class CRITICAL-APP
Router(config-pmap-c)# bandwidth percent 40
Router(config-pmap-c)# random-detect
Router(config-pmap-c)# random-detect dscp af21 80 120
Router(config-pmap-c)# random-detect dscp af22 60 100
Router(config-pmap-c)# random-detect dscp af23 40 80
Router(config-pmap-c)# exit

Router(config-pmap)# class class-default
Router(config-pmap-c)# random-detect
Router(config-pmap-c)# exit
```

**Explicación:** `random-detect` activa WRED. Los parámetros indican los umbrales mínimo y máximo antes de empezar a descartar paquetes al azar. Esto permite que la cola aproveche mejor el espacio disponible mientras se previene la congestión severa.

**Paso C: Verificar si el Problema es el Ancho de Banda Insuficiente**

Si la interfaz está constantemente al 85-90% de uso, aumentar el ancho de banda es la única solución real a largo plazo:

```
Router# show interface Serial0/0/0 | include bandwidth
  Encapsulation HDLC, loopback not set
  Keepalive set (10 sec)
  Last input 00:00:03, output 00:00:02
  Bandwidth: 10000 kbit/sec (configurable)
```

Cambiar a una velocidad de enlace más rápida (por ejemplo, de 10 Mbps a 20 Mbps o 50 Mbps, según la disponibilidad del ISP).

**Paso D: Implementar Policing en el Tráfico de Menor Prioridad**

Para evitar que aplicaciones de bajo valor (como descargas de torrents o actualizaciones de software) consuman todo el ancho de banda:

```
Router(config)# class-map match-any LOW-PRIORITY-APPS
Router(config-cmap)# match protocol http
Router(config-cmap)# match protocol ftp
Router(config-cmap)# exit

Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class LOW-PRIORITY-APPS
Router(config-pmap-c)# police 2000000 100000 exceed-action drop
Router(config-pmap-c)# exit
```

### Validación Post-Resolución

```bash
Router# clear interface Serial0/0/0
Router# wait 15 minutes during normal operation

Router# show interface Serial0/0/0 | include drops
  Transmitted 32145678 packets, 34567890 bytes, 12 drops  ! Reducido de 567

Router# show policy-map interface Serial0/0/0
  Class map: CRITICAL-APP
    Packets dropped/matched: 2/45234  ! Reducido de 45
    Random early drops: 8  ! ECN está funcionando

Router# iperf test
  Bandwidth measured: 4.8 MB/s  ! Mejorado de 500 KB/s antes
```

Realizar una transferencia FTP de prueba y confirmar que la velocidad ha mejorado significativamente.

---

## Escenario 4: Tráfico de Baja Prioridad Acaparando Ancho de Banda

### Síntomas Reportados por el Usuario

- Un usuario o aplicación desconocida está descargando archivos/contenido de streaming masivamente
- Las aplicaciones empresariales se vuelven lentas
- El administrador quiere controlar el tráfico de bajo valor (P2P, descargas de torrents, YouTube, etc.)
- No se sabe exactamente qué aplicación o usuario es el culpable

### Análisis Inicial

Este es un problema común en redes empresariales donde no existe suficiente control de acceso al ancho de banda. Un usuario descuidado o malintencionado puede saturar completamente un enlace WAN compartido, perjudicando a todos los demás.

### Paso 1: Identificar el Tráfico que Consume Ancho de Banda

Usar NBAR para descubrir protocolos:

```bash
Router# show ip nbar protocol-discovery interface Serial0/0/0 sorted by bytes
  Protocol                    Bytes           % Bytes      Packets
  http                        3456789012      45.2%        234567
  https                       2345678901      30.7%        156789
  bittorrent                  1234567890      16.1%        98765
  ftp                         345678901       4.5%         23456
  youtube                     234567890       3.1%         12345
```

**Descubrimiento:** BitTorrent está consumiendo el 16.1% del ancho de banda total (más de 1.2 GB en el período de observación). Esto es claramente no autorizado en la mayoría de ambientes corporativos.

### Paso 2: Identificar el Origen del Tráfico

```bash
Router# show ip nbar protocol-discovery interface Serial0/0/0 | include bittorrent
  Protocol: BitTorrent
    Client IP: 192.168.1.105, Server IP: various
    Port: 6881-6889
    Bytes: 1234567890
```

**Conclusión:** La IP 192.168.1.105 es la fuente del tráfico BitTorrent. Esta es una máquina en el segmento de usuarios (192.168.1.0/24).

### Paso 3: Verificar si Existe una Política que Bloquee o Restrinja este Tráfico

```bash
Router# show running-config | include class-map|policy-map
  class-map match-any VOICE
  class-map match-any CRITICAL-APP
  policy-map WAN-EGRESS-POLICY
```

**Hallazgo:** No existe una clase específica para tráfico de baja prioridad o P2P. Todo el tráfico desconocido cae en la clase-default, que se está compartiendo con navegación web legítima.

### Paso 4: Verificar si NBAR puede Identificar Automáticamente Aplicaciones

```bash
Router# show ip nbar protocol-discovery | include application
  NBAR 2 enabled
  Protocol Discovery enabled
  Applications discovered: 1247
```

El router tiene NBAR2 habilitado, que puede detectar más de 1000 aplicaciones, incluida la capacidad de clasificar P2P, videostreaming, redes sociales, etc.

### Resolución Completa

**Paso A: Crear Clases para Tráfico No Deseado**

```
Router(config)# class-map match-any P2P-APPLICATIONS
Router(config-cmap)# match protocol bittorrent
Router(config-cmap)# match protocol edonkey
Router(config-cmap)# match protocol gnutella
Router(config-cmap)# match protocol dcpp
Router(config-cmap)# exit

Router(config)# class-map match-any STREAMING-VIDEO
Router(config-cmap)# match protocol youtube
Router(config-cmap)# match protocol netflix
Router(config-cmap)# match protocol hulu
Router(config-cmap)# match protocol twitch
Router(config-cmap)# exit

Router(config)# class-map match-any SOCIAL-MEDIA
Router(config-cmap)# match protocol facebook
Router(config-cmap)# match protocol twitter
Router(config-cmap)# match protocol instagram
Router(config-cmap)# exit
```

**Paso B: Crear una Política Restrictiva**

Existen varias opciones según la política de la empresa:

**Opción 1: Bloqueo Total**

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class P2P-APPLICATIONS
Router(config-pmap-c)# drop
Router(config-pmap-c)# exit

Router(config-pmap)# class STREAMING-VIDEO
Router(config-pmap-c)# police 1000000 50000 exceed-action drop
Router(config-pmap-c)# exit
```

Este enfoque es duro pero efectivo: elimina completamente P2P y limita streaming a 1 Mbps máximo.

**Opción 2: Control de Tasa (Rate Limiting)**

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class P2P-APPLICATIONS
Router(config-pmap-c)# police 500000 25000 exceed-action mark-as-class SCAVENGER
Router(config-pmap-c)# exit

Router(config-pmap)# class STREAMING-VIDEO
Router(config-pmap-c)# police 2000000 100000 exceed-action mark-as-class SCAVENGER
Router(config-pmap-c)# exit

Router(config-pmap)# class SCAVENGER
Router(config-pmap-c)# bandwidth percent 5
Router(config-pmap-c)# exit
```

Este enfoque permite algo de tráfico pero lo restringe y lo marca como "scavenger" (carroñero), de modo que solo consume el 5% del ancho de banda disponible.

**Opción 3: Control Basado en Horarios**

Si P2P es permitido en ciertas horas (por ejemplo, después de las 5 PM):

```
Router(config)# time-range OFFICE-HOURS
Router(config-time-range)# periodic weekdays 09:00 to 17:00
Router(config-time-range)# exit

Router(config)# access-list extended P2P-BLOCK
Router(config-ext-acl)# deny udp any any eq 6881 time-range OFFICE-HOURS
Router(config-ext-acl)# deny udp any any eq 6889 time-range OFFICE-HOURS
Router(config-ext-acl)# permit udp any any
Router(config-ext-acl)# exit

Router(config)# class-map match-any P2P-BLOCK
Router(config-cmap)# match access-group P2P-BLOCK
Router(config-cmap)# exit

Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class P2P-BLOCK
Router(config-pmap-c)# drop
Router(config-pmap-c)# exit
```

**Paso C: Aplicar la Política**

```
Router(config)# interface Serial0/0/0
Router(config-if)# service-policy output WAN-EGRESS-POLICY
Router(config-if)# exit
Router(config)# exit
```

**Paso D: Monitoreo Activo**

Configurar logging para monitorear cuándo se detectan aplicaciones no permitidas:

```
Router(config)# access-list 199 permit tcp any any
Router(config)# ip access-list extended LOG-P2P
Router(config-ext-acl)# permit tcp any any eq 6881 log
Router(config-ext-acl)# permit tcp any any eq 6889 log
Router(config-ext-acl)# exit

Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class P2P-APPLICATIONS
Router(config-pmap-c)# set ip dscp scavenger
Router(config-pmap-c)# police 500000 exceed-action drop
Router(config-pmap-c)# exit
```

### Validación Post-Resolución

```bash
Router# clear ip nbar protocol-discovery
Router# wait 30 minutes

Router# show ip nbar protocol-discovery interface Serial0/0/0 sorted by bytes
  Protocol                    Bytes           % Bytes      Packets
  http                        234567890       54.2%        134567
  https                       145678901       33.6%        87654
  bittorrent                  12345678        2.8%         1234      ! Reducido de 16.1%
  ftp                         34567890        7.9%         2345
  youtube                     5678901         1.3%         234       ! Reducido de 3.1%

Router# show policy-map interface Serial0/0/0
  Class map: P2P-APPLICATIONS
    Packets matched: 1234
    Packets dropped: 1200  ! Bloqueados efectivamente

  Class map: STREAMING-VIDEO
    Packets matched: 234
    Packets dropped: 156  ! Limitados
```

El ancho de banda disponible para aplicaciones críticas ha aumentado significativamente, y el usuario en 192.168.1.105 verá que sus descargas de torrent se ralentizan drásticamente o se bloquean por completo.

---

## Escenario 5: Problemas de Jitter en Videoconferencia

### Síntomas Reportados por el Usuario

- Videollamadas se congelan ocasionalmente durante 100-200 ms
- El video no está sincronizado con el audio
- Algunos usuarios reportan problemas, otros no (inconsistencia geográfica o temporal)
- Aumentar el ancho de banda de la conexión no ayuda

### Análisis Inicial

El jitter es la variación en el tiempo de llegada de paquetes consecutivos. A diferencia de la latencia, que es un problema de retraso absoluto, el jitter es un problema de *consistencia* del retraso. Un enlaces con latencia consistente de 100 ms es mejor que un enlace con latencia variable de 50-150 ms, incluso aunque el promedio sea el mismo.

### Paso 1: Medir la Latencia y el Jitter en Tiempo Real

Usar un comando que muestre la latencia de paquete a paquete:

```bash
Router# ping 10.0.1.1 -c 100 -i 0.01
  ICMP Seq 1: bytes=56 time=45.123 ms
  ICMP Seq 2: bytes=56 time=46.234 ms
  ICMP Seq 3: bytes=56 time=45.876 ms
  ICMP Seq 4: bytes=56 time=48.123 ms
  ICMP Seq 5: bytes=56 time=45.654 ms
  ...
  Round-trip time (ms):
    Min = 45.123, Max = 48.123, Avg = 46.123, StdDev = 0.856
```

**Análisis:** La desviación estándar de 0.856 ms sugiere un jitter bajo. Sin embargo, esto es desde el router. El jitter puede ser más alto en otros puntos de la ruta.

### Paso 2: Medir Jitter en el Switch de Acceso

Si es posible, realizar la misma prueba desde el switch de acceso hacia otro switch de acceso remoto:

```bash
Switch# ping 172.16.1.1 -c 100 -i 0.01
  Round-trip time (ms):
    Min = 15.234, Max = 67.890, Avg = 35.123, StdDev = 12.456
```

**Descubrimiento:** El jitter aquí es mucho más alto (StdDev 12.456 ms). Esto sugiere que el problema podría estar en el switch de acceso o en el enlace entre switches.

### Paso 3: Verificar si la Videoconferencia Está Siendo Correctamente Priorizada

```bash
Router# show policy-map interface Gi0/0/1
  Class map: VOICE (match-any)
    Match: ip dscp ef
    Packets matched: 45234
    QueueStatistics: Dropped = 0/45234

  Class map: VOICE-SIGNALING (match-any)
    Match: ip dscp cs3
    Packets matched: 12345
    QueueStatistics: Dropped = 0/12345

  Class map: CRITICAL-APP (match-any)
    Match: access-group CRITICAL-APP
    Packets matched: 234567
    QueueStatistics: Dropped = 45/234567

  Class map: VIDEO-CONFERENCE (match-any)
    Match: protocol h.323
    Packets matched: 0  ! <-- PROBLEMA: Sin coincidencias
```

**Descubrimiento Crítico:** La clase VIDEO-CONFERENCE tiene 0 paquetes coincidentes, lo que significa que el tráfico de videoconferencia no está siendo identificado o clasificado.

### Paso 4: Investigar por qué no se Clasifica la Videoconferencia

Hay varias razones:

A) H.323 no es el protocolo correcto (puede ser SIP, WebRTC, etc.)
B) El tráfico está codificado o encapsulado de manera que NBAR no puede detectarlo
C) La videoconferencia utiliza un rango de puertos dinámicos no cubierto por las ACLs

Usar NBAR para ver qué protocolos se están detectando:

```bash
Router# show ip nbar protocol-discovery interface Gi0/0/1 | include video|conference|rtp|h323
  Protocol: h.323                    Bytes: 0
  Protocol: rtp                      Bytes: 45678901
  Protocol: rtcp                     Bytes: 1234567
```

**Descubrimiento:** RTP (Real-time Transport Protocol) está siendo detectado con 45 MB de tráfico, pero la clase está buscando específicamente h.323. RTP es el protocolo de transporte que lleva tanto voz como video, pero la clase-map no lo está capturando.

### Paso 5: Verificar la Configuración de la Clase-Map

```bash
Router# show class-map VIDEO-CONFERENCE
Class Map 'VIDEO-CONFERENCE' (match-any)
  Match protocol h.323
  Match protocol skype
```

**Problema:** La clase está buscando protocolos de aplicación (h.323, skype) en lugar del protocolo de transporte (RTP). Además, está buscando puertos específicos, que pueden no ser precisos para videoconferencia moderna basada en web.

### Resolución

**Paso A: Crear Clases Más Inclusivas para Videoconferencia**

```
Router(config)# class-map match-any REALTIME-MEDIA
Router(config-cmap)# match protocol rtp
Router(config-cmap)# match protocol rtcp
Router(config-cmap)# match protocol h.323
Router(config-cmap)# match protocol sip
Router(config-cmap)# exit

Router(config)# class-map match-any VIDEO-STREAMING
Router(config-cmap)# match protocol rtp
Router(config-cmap)# match ip dscp af41
Router(config-cmap)# match ip dscp af42
Router(config-cmap)# exit
```

**Paso B: Asignar Prioridad Apropiada**

La videoconferencia interactiva es sensible al retardo pero no tanto como la voz. Usar una clase intermedia:

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class VOICE
Router(config-pmap-c)# priority percent 25
Router(config-pmap-c)# queue-limit 100
Router(config-pmap-c)# exit

Router(config-pmap)# class REALTIME-MEDIA
Router(config-pmap-c)# bandwidth percent 20
Router(config-pmap-c)# queue-limit 200
Router(config-pmap-c)# random-detect dscp af41 100 150
Router(config-pmap-c)# random-detect dscp af42 80 120
Router(config-pmap-c)# exit

Router(config-pmap)# class CRITICAL-APP
Router(config-pmap-c)# bandwidth percent 35
Router(config-pmap-c)# queue-limit 300
Router(config-pmap-c)# exit

Router(config-pmap)# class class-default
Router(config-pmap-c)# bandwidth percent 20
Router(config-pmap-c)# fair-queue
Router(config-pmap-c)# exit
```

**Explicación:**
- VOICE obtiene 25% de ancho de banda garantizado con prioridad estricta (máxima velocidad, mínima latencia)
- REALTIME-MEDIA (videoconferencia) obtiene 20% de ancho de banda garantizado pero sin prioridad estricta (puede esperar un poco por la voz, pero tiene garantía)
- CRITICAL-APP obtiene 35% (para datos empresariales)
- class-default obtiene el 20% restante

**Paso C: Marcar Correctamente el Tráfico en la Fuente**

Muchas aplicaciones de videoconferencia, como Cisco Webex, Zoom o Teams, pueden marcar su propio tráfico. Verificar la configuración:

En Microsoft Teams:
- Configuración > Dispositivos > Configuración de vídeo/audio
- Buscar opciones de QoS o "Calidad de servicio"
- Configurar DSCP para video como AF41 (34) y audio como EF (46)

En Cisco Webex:
- Configuración del sitio > Seguridad > QoS
- Habilitar marcado de DSCP
- Configurar DSCP para video como AF42 (36)

**Paso D: Implementar una Política Adicional en el Switch de Acceso**

El switch de acceso también debe ser configurado para no interferir con las marcas de DSCP:

```
Switch(config)# interface range Fa0/1-48
Switch(config-if-range)# mls qos trust dscp
Switch(config-if-range)# exit

Switch(config)# class-map match-any REALTIME-MEDIA
Switch(config-cmap)# match ip dscp ef
Switch(config-cmap)# match ip dscp af41
Switch(config-cmap)# match ip dscp af42
Switch(config-cmap)# exit

Switch(config)# policy-map PRESERVE-VIDEO-QOS
Switch(config-pmap)# class REALTIME-MEDIA
Switch(config-pmap-c)# priority queue 1
Switch(config-pmap-c)# exit

Switch(config)# interface range Fa0/1-48
Switch(config-if-range)# service-policy input PRESERVE-VIDEO-QOS
Switch(config-if-range)# exit
```

### Validación Post-Resolución

Realizar una videollamada de prueba durante períodos de congestión:

```bash
Router# show policy-map interface Gi0/0/1
  Class map: REALTIME-MEDIA (match-any)
    Packets matched: 45678901
    QueueStatistics: Packets dropped = 0/45678901
    Average queue depth: 23 packets
    Mean jitter impact: Minimal

Router# show ip nbar protocol-discovery interface Gi0/0/1 | grep rtp
  Protocol: rtp                      Bytes: 45678901  ! Ahora el cálculo de utilization será correcto:

```bash
Router# show interface Serial0/0/0 | include Txload|Rxload
  Txload: 15%, Rxload: 12%  ! Ahora refleja la realidad
```

El problema estaba revelado: la interfaz estaba siendo sobre-subscrita en un factor de 6.5x. Con la configuración correcta de ancho de banda, el administrador puede ver el verdadero estado y ajustar QoS apropiadamente.

### Validación Post-Resolución

```bash
Router# show policy-map interface Serial0/0/0
  Class map: CRITICAL-APP (match-any)
    Output queue: 45/500 (packets)  ! Menos densidad de cola
    Packets dropped: 0  ! Sin descartes
    Random early drops: 0
    Average queue depth: 34 packets

Router# show interface Serial0/0/0 | include dropped
  Output queue: 89/100 (packets), 45678/3000 (bytes)
  1234 packets input, 56789 bytes, 0 drops  ! Drops eliminados
```

---

## Escenario 7: Problemas de QoS entre Dominios (Múltiples Proveedores)

### Síntomas Reportados por el Usuario

- QoS funciona perfectamente dentro de la red corporativa
- Las llamadas VoIP se degradan cuando el tráfico cruza hacia la red del ISP
- La marca DSCP se "pierde" en algún punto entre el router corporativo y el destino remoto
- El usuario remoto reporta problemas, pero el usuario local no

### Análisis Inicial

Este es un problema común en redes WAN donde el tráfico sale de la red corporativa hacia la de un ISP. Muchos ISPs:
1. No respetan marcas DSCP de entrada por razones de seguridad
2. Remarcan todo el tráfico a valores por defecto
3. Tienen sus propias políticas de QoS internas que pueden no alinearse con las corporativas

### Paso 1: Verificar la Marca de DSCP en el Borde Corporativo

```bash
Router-Corporativo# show policy-map interface Serial0/0/0
  Class map: VOICE (match-any)
    Match: ip dscp ef
    Packets matched: 45234
    Set ip dscp ef  ! Verificar que está siendo remarked
```

**Análisis:** El router corporativo está marcando correctamente el tráfico con DSCP EF.

### Paso 2: Capturar Tráfico en Ambos Lados de la Interfaz del ISP

Usar una herramienta de captura como Wireshark o tcpdump en ambos lados del enlace:

En el lado corporativo:
```bash
Router-Corporativo# capture VOICE-CAPTURE interface Serial0/0/0 match ipv4 172.16.0.0 255.255.0.0 172.16.10.0 255.255.255.0
```

En el lado del ISP (si es accesible):
```bash
Router-ISP# capture ISP-CAPTURE interface Serial0/0/0 match ipv4 172.16.0.0 255.255.0.0 172.16.10.0 255.255.255.0
```

Analizando las capturas:

**Lado Corporativo:**
```
Frame 1: 172.16.10.50 > 10.0.1.100
  IP Header: DSCP = 46 (EF)
  Payload: RTP Audio
```

**Lado ISP (entrada):**
```
Frame 1: 172.16.10.50 > 10.0.1.100
  IP Header: DSCP = 0 (BE)  ! <-- HA SIDO REMARKED
  Payload: RTP Audio
```

**Descubrimiento:** El ISP está remarking el tráfico de entrada a DSCP 0 (Best Effort).

### Paso 3: Revisar el Acuerdo de Nivel de Servicio (SLA) con el ISP

Contactar con el ISP y verificar:
1. ¿Respetan las marcas DSCP de entrada?
2. ¿Tienen un servicio de QoS especial?
3. ¿Cuál es su política predeterminada para remarking?
4. ¿Pueden hacer excepciones para tráfico crítico?

**Descubrimiento Posible:** Muchos ISPs no respetan DSCP de entrada por defecto, pero algunos ofrecen un servicio adicional donde acuerdan "confiar" en ciertas marcas.

### Resolución

**Opción 1: Negociar con el ISP**

Si el ISP ofrece un servicio QoS premium, contratarlo y suministrar:
1. Rango de IPs corporativas
2. Clases de tráfico crítico (puertos, protocolos)
3. Valores DSCP esperados

El ISP configurará su borde para confiar en estas marcas.

**Opción 2: Usar Tunneling Encriptado**

Si el ISP no respeta DSCP, pero ofrece un circuito privado o VPN con ciertos atributos, usar eso. Por ejemplo:

```
Router-Corporativo(config)# crypto map ISP-CRYPTO-MAP 10 ipsec-isakmp
Router-Corporativo(config-crypto-map)# set peer 203.0.113.1  ! ISP Edge Router
Router-Corporativo(config-crypto-map)# set transform-set ISP-TRANSFORM
Router-Corporativo(config-crypto-map)# match address VPN-TRAFFIC
Router-Corporativo(config-crypto-map)# exit

Router-Corporativo(config)# interface Serial0/0/0
Router-Corporativo(config-if)# crypto map ISP-CRYPTO-MAP
Router-Corporativo(config-if)# exit
```

En este caso, el encriptado puede servir como un identificador "natural" que el ISP puede usar para identificar el tráfico VPN y aplicarle diferente tratamiento.

**Opción 3: Aplicar QoS Basada en Encapsulación**

Si el ISP no respeta DSCP pero respeta MPLS o 802.1Q tags:

```
Router-Corporativo(config)# class-map match-any VOICE
Router-Corporativo(config-cmap)# match ip dscp ef
Router-Corporativo(config-cmap)# exit

Router-Corporativo(config)# policy-map WAN-EGRESS
Router-Corporativo(config-pmap)# class VOICE
Router-Corporativo(config-pmap-c)# set mpls experimental topmost 5  ! Top priority in MPLS
Router-Corporativo(config-pmap-c)# set ip dscp ef  ! También mantener DSCP para dentro de la red
Router-Corporativo(config-pmap-c)# exit
```

El ISP puede ser configurado para "confiar" en los tags MPLS en lugar de DSCP.

**Opción 4: Redesignar la Arquitectura**

Si el ISP simplemente no coopera, considerar:
1. Cambiar a un ISP diferente con mejor soporte de QoS
2. Usar múltiples ISPs con tráfico crítico en el que tenga más control
3. Usar un MPLS provider (como una conexión Carrier Ethernet) que garantice QoS

### Validación Post-Resolución

Después de resolver el problema:

```bash
Router-Corporativo# capture VOICE-CAPTURE interface Serial0/0/0

Router-Corporativo# export capture VOICE-CAPTURE bootflash:voice-after.pcap

! Analizar con Wireshark en ambos lados para confirmar que las marcas se preservan
```

Realizar una prueba de VoIP desde el usuario corporativo hacia el usuario remoto durante congestión para confirmar que no hay degradación.

---

## Escenario 8: Comportamiento Inesperado de LLQ (Low Latency Queuing)

### Síntomas Reportados por el Usuario

- Configuró LLQ para priorizar voz, pero el ancho de banda no se distribuye correctamente
- La clase de prioridad acapara todo el ancho de banda incluso cuando no hay tráfico de voz
- Otras clases están siendo "starved" (inanidas) de ancho de banda
- El comportamiento es inconsistente o no se alinea con las expectativas

### Análisis Inicial

LLQ es una herramienta poderosa pero fácil de configurar incorrectamente. Los problemas comunes incluyen:
1. No configurar correctamente el policer implícito de la cola de prioridad
2. Asignar demasiado ancho de banda a la cola de prioridad
3. Olvidar que el 25% reservado por defecto necesita consideración
4. Confundir "priority" con "bandwidth percent"

### Paso 1: Examinar la Configuración de LLQ Actual

```bash
Router# show running-config | section policy-map WAN-EGRESS
policy-map WAN-EGRESS-POLICY
  class VOICE
    priority percent 30
  class CRITICAL-APP
    bandwidth percent 40
  class class-default
    fair-queue
```

**Análisis:** Superficialmente, esto se ve correcto. 30% para voz, 40% para aplicaciones críticas. Pero hay un problema oculto.

### Paso 2: Verificar el Comportamiento Real en Tiempo de Ejecución

Crear un escenario de prueba donde:
1. Generar tráfico VOICE a bajo volumen (por ejemplo, 500 Kbps)
2. Generar tráfico CRITICAL-APP a máximo volumen (por ejemplo, 5 Mbps)
3. Observar cómo se distribuye el ancho de banda

```bash
Router# show policy-map interface Serial0/0/0 | include bandwidth|priority
  Class map: VOICE
    Priority Percentage 30% (3000 kbps)
    Output queue: 5/64 (packets)  ! Baja profundidad
    Packets matched: 234
    Bytes sent: 456789

  Class map: CRITICAL-APP
    Bandwidth Percentage 40% (4000 kbps)
    Output queue: 234/200 (packets)  ! Alta profundidad, esperando servicio
    Packets matched: 456789
    Bytes sent: 2345678
```

**Análisis:** La cola de CRITICAL-APP está al 117% de su capacidad (234/200), lo que indica que está descartando paquetes. Mientras tanto, VOICE tiene solo 5 paquetes pendientes y está enviando datos.

Ahora, detener la prueba de CRITICAL-APP y generar tráfico VOICE al máximo (por ejemplo, 3 Mbps):

```bash
Router# wait 60 seconds

Router# show policy-map interface Serial0/0/0
  Class map: VOICE
    Output queue: 120/64 (packets)  ! Ahora desbordado!
    Packets matched: 234567
    Packets dropped: 45678  ! Drops masivos
```

**Descubrimiento:** Cuando solo hay tráfico VOICE, la cola está desbordada y se están descartando paquetes. Esto es contradictorio con la intención de LLQ, que es dar máxima prioridad a la voz.

### Paso 3: Investigar el Tamaño de la Cola

El problema está en el tamaño de la cola de prioridad. El limite de 64 paquetes es demasiado pequeño para manejar ráfagas de VoIP:

```bash
Router# show running-config | section priority
  class VOICE
    priority percent 30
    queue-limit 64 packets
```

64 paquetes × tamaño promedio de 160 bytes (paquete RTP típico) = 10.24 KB. Con una velocidad de 3 Mbps, esto se llena en aproximadamente 27 ms. Si hay cualquier variación en la tasa de salida, se producirán descartes.

### Resolución

**Paso A: Aumentar el Tamaño de la Cola de Prioridad**

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class VOICE
Router(config-pmap-c)# priority percent 30
Router(config-pmap-c)# queue-limit 150  ! Aumentado de 64
Router(config-pmap-c)# exit
```

**Paso B: Verificar el Policer Implícito**

El policer implícito en LLQ debe estar configurado correctamente. Según la documentación de Cisco, el policer implícito tiene una profundidad de cola interna:

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class VOICE
Router(config-pmap-c)# priority percent 30
Router(config-pmap-c)# police 3000000 50000  ! CIR 3 Mbps, Bc 50 KB (burst)
Router(config-pmap-c)# exit
```

**Explicación:**
- CIR (Committed Information Rate): 3 Mbps
- Bc (Committed Burst): 50 KB (aproximadamente 166 ms de tráfico a 3 Mbps)

Si el tráfico de voz intentan exceder 3 Mbps durante más de 166 ms, los paquetes excedentes serán descartados. Esto evita que una aplicación malintencionada marque su tráfico como VOICE y acapare todo el ancho de banda.

**Paso C: Considerar la Asignación General de Ancho de Banda**

Recordar la regla del 75%: Por defecto, Cisco solo permite asignar 75% del ancho de banda:

```
Router# show policy-map interface Serial0/0/0 | include Sum
  Queue Statistics for all classes
    BW Sum: 100%  ! 30% (voice) + 40% (app) + 20% (default) = 90%
```

Verificar si se alcanzó o se superó el 75%:
- 30% (VOICE priority) + 40% (CRITICAL-APP) = 70%

Esto está dentro del límite del 75%, así que está bien.

**Paso D: Verificar que "Fair-Queue" en la Clase Default Está Funcionando**

```bash
Router# show policy-map interface Serial0/0/0 | include fair|class-default
  Class map: class-default
    Queue Mechanism: fair-queue
    Output queue: 456/1000 (packets)
    Number of flows: 234
```

WFQ está funcionando correctamente, distribuyendo 20% del ancho de banda (2 Mbps) entre 234 flujos de manera justa.

### Validación Post-Resolución

Repetir las pruebas:

```bash
! Generar VOICE a máximo (3 Mbps)
! Generar CRITICAL-APP simultáneamente (4 Mbps)

Router# show policy-map interface Serial0/0/0
  Class map: VOICE
    Output queue: 45/150 (packets)  ! Dentro de límite
    Packets matched: 345678
    Packets dropped: 0  ! Sin drops
    Average queue depth: 34 packets
    Average bandwidth utilized: 3000 kbps

  Class map: CRITICAL-APP
    Output queue: 120/200 (packets)
    Packets matched: 456789
    Packets dropped: 0
    Average bandwidth utilized: 4000 kbps

  Class map: class-default
    Output queue: 234/1000 (packets)
    Average bandwidth utilized: 2000 kbps  ! Recibe el ancho de banda restante
```

El comportamiento ahora es predecible y correcto:
- VOICE siempre obtiene 3 Mbps mínimo
- CRITICAL-APP obtiene 4 Mbps mínimo
- class-default obtiene el 3 Mbps restante
- No hay descartes en ninguna clase de prioridad

---

## Parte III: Matriz de Referencia Rápida para Troubleshooting

### Tabla: Síntoma → Causa Probable → Solución

| Síntoma | Causas Probables | Soluciones Iniciales |
|---------|-----------------|---------------------|
| Voz entrecortada/robótica | Jitter alto, Pérdida de paquetes | Verificar `show policy-map interface`; Aumentar priority en LLQ; Verificar frontera de confianza en switch |
| Aplicación lenta sin motivo aparente | Clasificación incorrecta, Ancho de banda insuficiente | Usar NBAR para verificar tráfico; Expandir ACLs; Considerar aumento de ancho de banda |
| Pérdida de paquetes aleatorias | Colas demasiado pequeñas, WRED agresivo | Aumentar `queue-limit`; Ajustar umbrales de WRED; Verificar `bandwidth` configurado vs. real |
| P2P/Torrents acaparando ancho de banda | Falta de clasificación, Policer no configurado | Usar NBAR para detectar P2P; Crear clases restrictivas; Aplicar policing |
| Video congelado, desincronizado | Jitter en RTP, Falta de priorización | Clasificar RTP correctamente; Usar bandwidth garantizado; Implementar WRED |
| QoS no funciona fuera de la red | ISP remarking DSCP | Contactar ISP; Usar VPN/MPLS; Considerar servicio QoS premium de ISP |
| Drops a pesar de bajo utilization | Ancho de banda configurado incorrecto | Usar `show interface bandwidth`; Comparar con `show controllers`; Alinear configuración |
| LLQ desatendido o inanido | Policer implícito no configurado correctamente | Ajustar `queue-limit` en priority class; Configurar policer explícito; Verificar distribución de ancho de banda |

### Tabla: Comandos de Diagnóstico Esenciales

| Comando | Propósito | Información Crítica |
|---------|----------|-------------------|
| `show policy-map interface [ifname]` | Ver estadísticas de QoS en tiempo real | Paquetes matched, dropped, profundidad de cola |
| `show ip nbar protocol-discovery interface [ifname]` | Descubrir protocolos en el enlace | Porcentaje de bytes por protocolo/aplicación |
| `show interface [ifname] \| include Txload\|Rxload` | Ver utilization real | Compara utilization vs. ancho de banda configurado |
| `show class-map` | Verificar definiciones de clases | Criterios de clasificación |
| `show access-lists` | Revisar ACLs para clasificación | Confirma que están correctas |
| `show controllers [ifname]` | Ver parámetros de hardware | Velocidad real, búferes, configuración de controlador |
| `show traffic-shape interface [ifname]` | Verificar shaping activo | CIR, Bc, Bytes sent, Peak rate |
| `ping -c [count] -i [interval]` | Medir latencia y jitter | Min, Max, Avg, StdDev (jitter) |
| `debug policy-map class` | Rastrear clasificación en tiempo real | Qué clase coincide con cada paquete |
| `capture [name] interface [ifname]` | Capturar tráfico para análisis con Wireshark | Valores DSCP, tamaño de paquetes, secuencia |

---

## Parte IV: Mejores Prácticas y Lecciones Aprendidas

### 1. QoS es un Diseño Integral, No un Parche

El mayor error común es intentar "arreglar" QoS en un único dispositivo. QoS requiere:
- Planificación estratégica desde el principio
- Configuración coherente en múltiples dispositivos
- Alineación con objetivos empresariales
- Revisión periódica a medida que cambian las aplicaciones

### 2. Documentar Todo

Mantener registros precisos de:
- Clases de tráfico definidas y su propósito
- Valores DSCP asignados a cada clase
- Ancho de banda garantizado vs. disponible
- Cambios realizados y cuándo
- Resultados de las pruebas

### 3. Validación Continua

No implementar QoS y "olvidarlo". Monitorear:
- Cambios en patrones de tráfico
- Nuevas aplicaciones que requieren priorización
- Desviaciones en rendimiento
- Capacidad y utilization

### 4. La Congestión es tu Amiga

Paradójicamente, la congestión es necesaria para que QoS funcione. Sin congestión, todas las clases obtienen el ancho de banda que necesitan sin priorización. Es durante la congestión cuando QoS elige qué flujos se atienden primero. Si no existe congestión, QoS está ocultando problemas de capacidad subyacentes.

### 5. Escalabilidad vs. Granularidad

Cuanto más específica sea la clasificación (más clases, criterios más estrictos), más granular será el control pero más compleja será la configuración y mayor será la carga de CPU. Encontrar el equilibrio:
- Grandes organizaciones: 5-8 clases (VOICE, VIDEO, CRITICAL-APP, WEB, BULK, SCAVENGER, DEFAULT)
- Medianas: 3-5 clases (VOICE, CRITICAL, BULK, DEFAULT)
- Pequeñas: 2-3 clases (VOICE, BULK, DEFAULT)

### 6. Pruebas en Laboratorio Primero

Simular escenarios de congestión en un laboratorio antes de implementar en producción:
- Usar herramientas como iperf, netperf para generar carga
- Crear diferentes perfiles de tráfico
- Validar que QoS se comporta como se espera
- Documentar los resultados

### 7. Monitoreo y Alertas

Implementar monitoreo continuo con herramientas como:
- SNMP para recolectar estadísticas de interfaces y políticas
- NetFlow/sFlow para análisis de tráfico a nivel de flujo
- Syslog para capturar eventos de error
- Alertas cuando drops excedan umbrales aceptables

---

## Conclusión

El troubleshooting de QoS es un arte que combina comprensión teórica profunda con experiencia práctica. Los siete escenarios presentados en este informe cubren la mayoría de situaciones del mundo real que un administrador de red encontrará. Las claves del éxito son:

1. **Diagnóstico metódico:** No asumir; verificar con herramientas
2. **Pensar de extremo a extremo:** QoS no es solo del router
3. **Validar cambios:** Cada modificación debe ser probada
4. **Documentar:** Mantener registros precisos
5. **Aprender continuamente:** Las redes evolucionan constantemente

Con estas habilidades y herramientas, incluso los problemas más complejos de QoS pueden ser identificados y resueltos de manera sistemática y eficiente. siendo clasificado
```

Repetir el ping test:

```bash
Router# ping 10.0.1.1 -c 100 -i 0.01
  Round-trip time (ms):
    Min = 44.123, Max = 45.876, Avg = 44.987, StdDev = 0.345  ! StdDev reducido significativamente
```

El jitter ha disminuido de 12.456 ms a 0.345 ms, lo que significa que los paquetes ahora llegan de manera más consistente, permitiendo que la videoconferencia funcione sin congelaciones.

---

## Escenario 6: Descartes Persistentes a Pesar de Tener Ancho de Banda Disponible

### Síntomas Reportados por el Usuario

- El router muestra "dropped packets" en las estadísticas de interfaz
- Sin embargo, el utilization de la interfaz es solo del 40-50%
- El problema es intermitente y correlaciona con ráfagas de tráfico
- Los descartes ocurren solo en ciertas clases de tráfico, no en todas

### Análisis Inicial

Este es un problema sutil que generalmente se debe a una de estas causas:

1. Las colas están llenas de tráfico de baja prioridad que no está siendo descartado antes de su tiempo
2. El tamaño de la cola es demasiado pequeño para las ráfagas de tráfico
3. WRED no está configurado apropiadamente
4. Hay una desalineación entre el ancho de banda configurado y el actual disponible

### Paso 1: Verificar Estadísticas de Cola Detalladas

```bash
Router# show policy-map interface Serial0/0/0 | include queue|dropped|threshold
  Class map: VOICE (match-any)
    Output queue: 10/100 (packets)
    Packets dropped: 0
    
  Class map: CRITICAL-APP (match-any)
    Output queue: 145/200 (packets)
    Packets dropped: 12
    Tail drop: 0, Random early drops: 12
    Minimum threshold: 150 packets
    Maximum threshold: 200 packets

  Class map: class-default
    Output queue: 234/1000 (packets)
    Packets dropped: 567
    Tail drop: 456, Random early drops: 111
    Minimum threshold: 700 packets
    Maximum threshold: 1000 packets
```

**Descubrimiento:** La clase CRITICAL-APP está descartando paquetes aunque la cola esté solo al 72.5% de su capacidad (145/200). Esto sugiere que WRED está activado y está descartando paquetes al azar incluso cuando hay espacio.

### Paso 2: Verificar la Configuración de WRED

```bash
Router# show running-config | section policy-map WAN-EGRESS
policy-map WAN-EGRESS-POLICY
  class VOICE
    priority percent 25
    queue-limit 100
  class CRITICAL-APP
    bandwidth percent 40
    queue-limit 200
    random-detect
    random-detect dscp af21 100 200 1/10
```

**Análisis:** WRED está configurado con umbrales mínimo de 100 paquetes y máximo de 200 paquetes. Esto significa que WRED comienza a descartar al azar cuando la cola alcanza 100 paquetes. Con 145 paquetes en la cola, WRED está activo y descartando.

La probabilidad 1/10 significa que WRED está descartando aproximadamente 1 de cada 10 paquetes en este rango de congestión leve.

### Paso 3: Verificar si los Descartes son Aceptables

Para aplicaciones como FTP, SSH, y navegación web que usan TCP, estos descartes causarán retransmisiones, lo que no es deseable. Para aplicaciones como VoIP que usan UDP, los descartes simplemente causan distorsión de audio, que es aceptable en pequeñas cantidades.

```bash
Router# show ip flow top-talkers | include dscp
  Source IP     Dest IP       Protocol  DSCP  Bytes       Packets
  172.16.10.50  10.0.1.200    TCP       AF21  23456789    345678  ! CRITICAL-APP
  192.168.1.100 8.8.8.8       TCP       0     45678901    234567  ! Default traffic
  172.16.10.40  10.0.1.201    UDP       EF    12345678    98765   ! Voice
```

El tráfico de CRITICAL-APP está marcado con AF21 (asurance of delivery), lo que significa que TCP esperará retransmisiones si hay pérdida.

### Resolución

**Paso A: Ajustar los Umbrales de WRED**

Si el objetivo es aceptar algo de congestión sin descartes prematuros, aumentar los umbrales mínimo y máximo:

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class CRITICAL-APP
Router(config-pmap-c)# bandwidth percent 40
Router(config-pmap-c)# queue-limit 400  ! Aumentado de 200
Router(config-pmap-c)# random-detect
Router(config-pmap-c)# random-detect dscp af21 300 400 1/20
Router(config-pmap-c)# random-detect dscp af22 250 350 1/15
Router(config-pmap-c)# random-detect dscp af23 200 300 1/10
Router(config-pmap-c)# exit
```

**Explicación:**
- El tamaño de la cola ahora es 400 en lugar de 200, permitiendo más ráfagas
- El umbral mínimo es 300 (75% de 400), no 100
- El umbral máximo es 400 (100% de 400)
- Esto significa que WRED no comienza a descartar hasta que la cola esté al 75% de capacidad

**Paso B: Alternativamente, Deshabilitar WRED para Ciertas Clases**

Si WRED no es deseado para aplicaciones críticas que requieren cero pérdidas:

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class CRITICAL-APP
Router(config-pmap-c)# bandwidth percent 40
Router(config-pmap-c)# queue-limit 500
Router(config-pmap-c)# no random-detect  ! Deshabilitar WRED
Router(config-pmap-c)# exit

Router(config-pmap)# class class-default
Router(config-pmap-c)# bandwidth percent 35  ! Reducido de 20% para dejar margen
Router(config-pmap-c)# queue-limit 1000
Router(config-pmap-c)# random-detect  ! Mantener WRED aquí
Router(config-pmap-c)# random-detect 500 1000 1/10
Router(config-pmap-c)# exit
```

En este escenario, el tráfico de CRITICAL-APP nunca será descartado a menos que la cola se llene completamente. El tráfico de class-default utilizará WRED para evitar que su propia congestión sea demasiado severa.

**Paso C: Verificar la Velocidad Real vs. Configurada**

A veces, el problema es que la interfaz está configurada para una velocidad diferente a la real:

```bash
Router# show interface Serial0/0/0 | include bandwidth|encapsulation|speed
  Encapsulation HDLC
  Bandwidth: 10000 kbit/sec (configurable)
  Reliability: 255/255
  Txload: 98%, Rxload: 87%

Router# show controllers serial 0/0/0 | include speed|clock
  clock source: line
  clock speed: 1.544 Mbps  ! <-- DESALINEACIÓN
```

**Descubrimiento:** La configuración del router dice 10 Mbps, pero el controlador de hardware informa 1.544 Mbps (una línea T1 antigua). El utilization calculado del 50% en una interfaz de 10 Mbps sería en realidad del 320% en una interfaz T1, ¡lo que causaría descartes masivos!

### Corrección

Alinear la configuración con la realidad:

```
Router(config)# interface Serial0/0/0
Router(config-if)# bandwidth 1544  ! Cambiado de 10000
Router(config-if)# exit
```

Ahora TEST-POLICY
Router(config-pmap)# class CRITICAL-APP-TEST
Router(config-pmap-c)# bandwidth percent 40
Router(config-pmap)# exit
```

Aplicar esta política de prueba a una interfaz de prueba (por ejemplo, mediante un redir de tráfico) y observar:

```bash
Router# show policy-map interface [test-interface]
  Class map: CRITICAL-APP-TEST
    Packets matched 45342  <-- Ahora coincide!
```

**Conclusión:** La ACL original era demasiado restrictiva. No coincidía con todo el tráfico de base de datos porque hay otros servidores, gateways o rutas que no estaban consideradas.

### Paso 5: Verificar el Comportamiento Actual de la Cola

```bash
Router# show policy-map interface Serial0/0/0
  Class map: CRITICAL-APP (match-any)
    Bandwidth requested 40% (4000 kbps)
    Bandwidth allocated 4000 kbps
    Output queue (Packets dropped/Packets output) = 234/45234
```

**Análisis:** Aunque la política solicita 40% de ancho de banda (4 Mbps), hay 234 drops (descarte de paquetes). Esto es un número bajo en porcentaje, pero para una aplicación de base de datos, incluso un pequeño porcentaje de pérdida puede causar retransmisiones TCP significativas que ralentizan drásticamente las transacciones.

Pero el problema más grande es que el tráfico no clasificado (clase-default) está consumiendo la mayoría del ancho de banda:

```bash
Router# show policy-map interface Serial0/0/0
  Class map: class-default
    Queue statistics for queue 2
      Output queue (Packets dropped/Packets output) = 12345/98765432
      Bandwidth available: unallocated
```

El tráfico no clasificado está experimentando 12345 drops, y esto es probablemente el tráfico de navegación web, descarga de archivos, etc.

### Paso 6: Identificar el Tráfico Competidor

¿Qué está causando la congestión a las 11 AM?

```bash
Router# show ip nbar protocol-discovery interface Serial0/0/0 sorted by bytes
  Protocol                  Bytes           % Bytes
  http                      456789123       67.2%
  https                     123456789       18.2%
  smtp                      45678901        6.7%
  oracle                    45342123        6.7%
  ftp                       12345678        1.8%
```

**Descubrimiento:** Entre las 11 AM y 1 PM, HTTP y HTTPS están consumiendo el 85% del ancho de banda disponible. Esto sugiere que alguien está descargando un gran archivo o que hay una transferencia de datos masiva en ese momento.

### Paso 7: Verificar si Existe una Política de Shaping o Policing en Segundo Plano

A veces, el problema no es la QoS del cliente sino una restricción de ancho de banda impuesta por el ISP:

```bash
Router# show traffic-shape interface Serial0/0/0
  Interface Serial0/0/0
  Traffic Shaping enabled
  Target Bit Rate: 10000000
  Byte Limit: 1250000
  Interval: 125 ms
  Current Byte Sent: 1250000
  Peak Rate Bytes: 1250000
```

Esto está bien; el shaping de 10 Mbps es superior a la velocidad de interfaz y no es un factor limitante.

Sin embargo, verificar también si hay un policer externo:

```bash
Router# show mls qos interface serial0/0/0
  QoS Summary [IP]:
    Ingress policies: 0
    Ingress aggregate police interface stats: Enabled
    Policer ID 1
      Rate 9000000 (bps) Burst 1000000 (bytes)
      Conformed bytes: 8956789
      Exceeded bytes: 3445789
```

**Descubrimiento:** Hay un policer de entrada que está limitando el tráfico a 9 Mbps y rechazando (exceeded) aproximadamente 3.4 MB de datos. Este policer puede estar causando o agravando el problema.

### Resolución

La resolución es multifacética:

**Paso A: Expandir la Clasificación de la Aplicación Crítica**

```
Router(config)# no access-list DATABASE_APP
Router(config)# access-list DATABASE_APP remark "All database traffic for critical app"
Router(config)# access-list DATABASE_APP permit tcp any any eq 1521
Router(config)# access-list DATABASE_APP permit tcp any eq 1521 any
Router(config)# access-list DATABASE_APP permit tcp any any eq 3306  ! MySQL if applicable
Router(config)# access-list DATABASE_APP permit tcp any eq 3306 any
Router(config)# exit

Router(config)# class-map match-any CRITICAL-APP
Router(config-cmap)# no match ip dscp af21
Router(config-cmap)# match access-group DATABASE_APP
Router(config-cmap)# match protocol oracle
Router(config-cmap)# exit
```

**Paso B: Asegurar que la Clase CRITICAL-APP Esté Priorizada**

Cambiar la política para usar LLQ (Low Latency Queuing) en lugar de solo CBWFQ:

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class VOICE
Router(config-pmap-c)# priority percent 30
Router(config-pmap-c)# exit

Router(config-pmap)# class CRITICAL-APP
Router(config-pmap-c)# bandwidth percent 35
Router(config-pmap-c)# exit

Router(config-pmap)# class class-default
Router(config-pmap-c)# fair-queue
Router(config-pmap-c)# exit
```

**Paso C: Aplicar un Policer al Tráfico de Menor Prioridad**

Para evitar que el tráfico web consume todo el ancho de banda, aplicar un límite al tráfico no clasificado:

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class class-default
Router(config-pmap-c)# police 3000000 100000 exceed-action drop
Router(config-pmap-c)# exit
```

Esto limita el tráfico no clasificado a 3 Mbps máximo.

**Paso D: Verificar que el Policer de Entrada No Esté Interfiriendo**

Si es posible, contactar con el ISP para aumentar el límite del policer o para confirmar que está alineado con la velocidad contratada. Si el contrato es de 10 Mbps, un policer de 9 Mbps puede ser inapropiado.

### Validación Post-Resolución

Esperar hasta que ocurra el próximo período pico (11 AM a 1 PM) y verificar:

```bash
Router# show policy-map interface Serial0/0/0
  Class map: VOICE (match-any)
    Packets matched 18234
    QueueStatistics: Packets dropped/output = 0/18234

  Class map: CRITICAL-APP (match-any)
    Packets matched 45342
    QueueStatistics: Packets dropped/output = 2/45342  ! Mínimo aceptable

  Class map: class-default
    Packets matched 456789
    QueueStatistics: Packets dropped/output = 12345/456789  ! Paquetes de menor prioridad se descartan
```

Realizar una prueba de aplicación:
- Ejecutar una consulta de base de datos que normalmente toma 200 ms
- Verificar que el tiempo de respuesta siga siendo aproximadamente 200 ms incluso durante la congestión

---

## Escenario 3: Pérdida de Paquetes Aleatoria en TCP

### Síntomas Reportados por el Usuario

- Las transferencias de archivos FTP son muy lentas (500 KB/s en lugar de 5 MB/s esperado)
- El navegador web ocasionalmente se congela durante 2-3 segundos
- Los tiempos de respuesta SSH son inconsistentes
- El problema es aleatorio, no predecible, y ocurre sin un patrón temporal claro

### Análisis Inicial

TCP es sensible a la pérdida de paquetes. Una pequeña tasa de pérdida (incluso del 1%) puede reducir drásticamente el rendimiento de TCP debido al mecanismo de control de congestión de TCP (multiplicación del factor de congestión por 0.5 cada vez que detecta una pérdida). Esto es diferente de UDP (VoIP), donde la pérdida simplemente hace que el audio se distorsione localmente.

### Paso 1: Verificar la Tasa de Pérdida en la Interfaz

```bash
Router# show interface Serial0/0/0 | include drops|error
  Received 45678901 packets, 56789012 bytes, 234 drops
  5 minute output rate 8956789 bits/sec, 1234 packets/sec
  Transmitted 32145678 packets, 34567890 bytes, 567 drops
```

**Hallazgo:** Hay 567 drops de salida (output drops). En un período de 5 minutos con 1234 packets/sec, eso es aproximadamente 367500 paquetes transmitidos, por lo que la tasa de pérdida es 567/367500 = 0.15%.

### Paso 2: Verificar si la Pérdida es Causa de la Política de QoS

Deshabilitar temporalmente la política de QoS en la interfaz:

```bash
Router# configure terminal
Router(config)# interface Serial0/0/0
Router(config-if)# no service-policy output WAN-EGRESS-POLICY
Router(config-if)# exit
```

Monitorear la tasa de pérdida durante 15 minutos en la interfaz:

```bash
Router# clear interface Serial0/0/0
Router# wait 900 seconds (15 minutes of traffic)
Router# show interface Serial0/0/0 | include drops
  Transmitted 32145678 packets, 34567890 bytes, 1230 drops
```

**Análisis:** La tasa de pérdida aumentó a 1230 drops en lugar de 567, lo que significa que la política de QoS, aunque no era óptima, estaba ayudando a reducir las pérdidas de algo peor. Reactivar la política y investigar por qué hay pérdidas incluso con QoS.

### Paso 3: Examinar los Detalles de la Cola

Reactivar la política:

```bash
Router(config)# interface Serial0/0/0
Router(config-if)# service-policy output WAN-EGRESS-POLICY
Router(config-if)# exit
```

Ahora examinar detalladamente las estadísticas de la cola:

```bash
Router# show policy-map interface Serial0/0/0 | include queue|dropped|class
  Class map: VOICE (match-any)
    Output queue: 45/64 (packets), 34567/4000 (bytes)
    Packets dropped/matched: 0/18234

  Class map: CRITICAL-APP (match-any)
    Output queue: 234/200 (packets), 456789/8000 (bytes)
    Packets dropped/matched: 45/45234

  Class map: class-default
    Output queue: 1234/1000 (packets), 6789123/10000 (bytes)
    Packets dropped/matched: 234/456789
```

**Descubrimiento Clave:** La cola de class-default tiene un tamaño máximo de 1000 paquetes, pero actualmente tiene 1234 paquetes pendientes. Esto significa que se está descartando tráfico en esta cola. Los 234 drops de class-default sugieren que la cola se está desbordando.

### Paso 4: Verificar el Tamaño de la Cola y los Umbrales

El tamaño de la cola para class-default probablemente no se ha configurado explícitamente:

```bash
Router# show running-config | include queue-limit
policy-map WAN-EGRESS-POLICY
  class VOICE
    priority percent 30
    queue-limit 64 packets
  class CRITICAL-APP
    bandwidth percent 40
    queue-limit 200 packets
```

**Problema:** El tamaño de la cola para CRITICAL-APP está limitado a 200 paquetes, que es relativamente bajo. Si el tráfico de aplicación crítica llega en ráfagas, la cola podría llenarse rápidamente y descartar paquetes.

### Paso 5: Verificar WRED (Weighted Random Early Detection)

WRED es una técnica de prevención de congestión que descarta paquetes proactivamente antes de que la cola se llene por completo:

```bash
Router# show policy-map interface Serial0/0/0 | include WRED|drop|early
  Class map: CRITICAL-APP
    Exponential Weighting Constant: 9 (1/512)
    Mean queue depth: 145
    Transmitted: 45234 bytes (Ecn marked: 0, Random early dropped: 45)
```

**Descubrimiento:** WRED ha descartado 45 paquetes al azar (random early drops). Este es un mecanismo de prevención de congestión que funciona correctamente, pero indica que la cola ha estado bajo presión.

### Paso 6: Verificar el Tamaño del Búfer de la Interfaz Física

A veces, el problema no es la QoS sino la capacidad del búfer de la interfaz misma:

```bash
Router# show controllers serial 0/0/0 | include buffers|queue
  Interface Serial0/0/0
  hardware is QUICC Serial
  txslotmap 0, rxslotmap 0
  rxring size 384, rxring addr 0x2A045000
```

Para cambiar el tamaño del búfer (esto varía según el hardware):

```
Router(config)# interface Serial0/0/0
Router(config-if)# tx-ring-limit 512
Router(config-if)# exit
```

### Paso 7: Investigar si hay un Problema de Ancho de Banda Insuficiente

Analizar el uso de ancho de banda actual durante los períodos problemáticos:

```bash
Router# show interface Serial0/0/0 load
  5 minute input rate: 8950000 bits/sec, 1234 packets/sec
  5 minute output rate: 8956789 bits/sec, 1200 packets/sec
  Load (1 min) Output: 89%, Input: 88%
```

**Análisis:** El enlace está utilizando el 88-89% de su capacidad durante estos períodos. Con un enlace de 10 Mbps teóricos, esto significa que el tráfico real es de aproximadamente 8.8-8.9 Mbps. En un escenario real, esto es demasiado cercano a la capacidad máxima. El protocolo TCP/IP y otros mecanismos de red necesitan algo de margen de maniobra.

### Resolución

**Paso A: Aumentar el Tamaño de las Colas**

El tamaño de la cola actual es demasiado pequeño para manejar ráfagas de tráfico:

```
Router(config)# policy-map WAN-EGRESS-POLICY
Router(config-pmap)# class VOICE
Router(config-pmap-c)# priority percent 30
Router(config-pmap-c)# queue-limit 128 packets  ! Aumentado de 64 a 128
Router(config-pmap-c)# exit

Router(config-pmap)# class CRITICAL-APP
Router(config-pmap-c)# bandwidth percent 40
Router(config-pmap-c)# queue-limit 500 packets  ! Aumentado de 200 a 500
Router(config-pmap-c)# exit

Router(config-pmap)# class class-default
Router(config-pmap-c)# queue-limit 2000 packets  ! Nuevo límite explícito
Router(config-pmap-c)# fair-queue
Router(config-pmap-c)# exit
```

**Paso B: Implementar WRED de Forma Más Granular**

Usar WRED para marcar paquetes con ECN (Explicit Congestion Notification) en lugar de descartarlos:

```
Router(config)# policy-map