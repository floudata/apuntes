# Informe Exhaustivo sobre Calidad de Servicio (QoS) en Redes Cisco
## De los Fundamentos a la Implementación Avanzada

---

## 1. Introducción

La Calidad de Servicio (QoS) en redes Cisco representa un conjunto de tecnologías y prácticas diseñadas para garantizar que las aplicaciones críticas reciban el ancho de banda y la prioridad necesarios en una red. Este informe proporciona una guía completa desde conceptos fundamentales hasta estrategias avanzadas de implementación.

---

## 2. Fundamentos de QoS

### 2.1 ¿Qué es QoS?

QoS es la capacidad de una red para proporcionar diferentes niveles de servicio a diferentes tipos de tráfico. Los objetivos principales incluyen:

- Asegurar el ancho de banda mínimo para aplicaciones críticas
- Limitar el uso de ancho de banda para tráfico no esencial
- Reducir la latencia y jitter en aplicaciones sensibles al tiempo
- Evitar la congestión de la red mediante priorización inteligente

### 2.2 Parámetros Clave de QoS

**Ancho de Banda (Bandwidth)**: Cantidad de datos que puede transmitirse en un período determinado, medida en bits por segundo (bps).

**Latencia**: Retardo de propagación de datos desde el origen hasta el destino. Las aplicaciones como VoIP requieren latencia inferior a 150 ms.

**Jitter**: Variación en la latencia entre paquetes consecutivos. Es crítico en aplicaciones de tiempo real.

**Pérdida de Paquetes**: Porcentaje de paquetes que no alcanzan su destino. Diferentes aplicaciones toleran distintos niveles de pérdida.

**Confiabilidad**: Capacidad de entregar datos de manera consistente y sin corrupción.

---

## 3. Modelos de QoS

### 3.1 Best Effort (Mejor Esfuerzo)

El modelo más simple donde la red no garantiza ningún nivel de servicio específico. Todos los paquetes se tratan por igual.

### 3.2 Integrated Services (IntServ)

Modelo que reserva recursos de red para flujos específicos mediante el protocolo RSVP (Resource Reservation Protocol). Proporciona garantías explícitas de QoS pero requiere mantenimiento de estado en cada router.

### 3.3 Differentiated Services (DiffServ)

Modelo escalable que clasifica el tráfico en diferentes categorías y aplica diferentes políticas de forwarding. Utiliza el campo DSCP en el encabezado IP.

---

## 4. Clasificación y Marcado de Tráfico

### 4.1 Métodos de Clasificación

**Basada en Puertos**: Identifica aplicaciones por número de puerto TCP/UDP.

**Basada en Protocolos**: Reconoce el tipo de protocolo (IP, MPLS, etc.).

**Basada en ACL**: Utiliza listas de control de acceso para clasificar tráfico según criterios específicos.

**Basada en DSCP**: Examina el valor DSCP en el encabezado del paquete IP.

**Basada en Comportamiento**: Inspecciona el contenido del paquete para identificar aplicaciones.

### 4.2 Campos de Marcado

**ToS (Type of Service)**: Campo de 8 bits en el encabezado IPv4 que incluye 3 bits para precedencia y 4 bits para ToS.

**DSCP (Differentiated Services Code Point)**: 6 bits del campo ToS que permiten 64 niveles diferentes de prioridad.

**CoS (Class of Service)**: Campo de 3 bits en etiquetas VLAN (802.1Q) que proporciona 8 niveles de prioridad.

**EXP (Experimental Bits)**: 3 bits en la etiqueta MPLS para llevar información de QoS.

### 4.3 Políticas de Marcado

Las políticas definen cómo se marcan los paquetes basándose en su clasificación:

```
class-map match-any VIDEO
 match protocol rtp
 match port tcp 554 1755

policy-map MARCA-TRAFICO
 class VIDEO
  set dscp af41
 class default
  set dscp default
```

---

## 5. Técnicas de Control de Tráfico

### 5.1 Policing (Vigilancia)

Limita la velocidad de tráfico permitido en una interfaz. Los paquetes que exceden el límite se descartan o remarcan.

**Algoritmo Token Bucket**: Los tokens se generan a una velocidad fija. Cada paquete consume tokens; si no hay suficientes, se descarta o remarcar.

**Implementación**: `police rate 1000000 burst 10000`

### 5.2 Shaping (Modelado)

Similar al policing pero bufferiza paquetes en exceso en lugar de descartarlos. Proporciona una salida más suave.

**Beneficio**: Reduce picos de tráfico evitando descartes.

**Ubicación**: Generalmente en interfaces de salida donde el tráfico se transmite.

### 5.3 Queuing (Encolamiento)

Define cómo se manejan los paquetes cuando hay congestión.

**FIFO (First In, First Out)**: El orden de llegada determina el orden de salida. No diferencia entre tipos de tráfico.

**Priority Queuing (PQ)**: Establece colas de prioridad alta, media, normal y baja. El tráfico de alta prioridad se procesa primero.

**Fair Queuing (FQ)**: Asigna igual ancho de banda a cada conversación/flujo.

**Weighted Fair Queuing (WFQ)**: Asigna ancho de banda basado en pesos definidos, útil para aplicaciones con diferentes requisitos.

**Class-Based Weighted Fair Queuing (CBWFQ)**: Combina clasificación con WFQ, permitiendo garantías de ancho de banda por clase.

---

## 6. Implementación de QoS en Redes Cisco

### 6.1 Arquitectura de QoS Recomendada

La arquitectura típica consta de tres componentes integrados:

**1. Clasificación y Marcado**: En el punto de entrada a la red, usualmente en acceso o distribución.

**2. Control y Conformidad**: Políticas de policing y shaping en puntos estratégicos.

**3. Encolamiento y Descarte**: En interfaces de salida donde hay cuellos de botella potenciales.

### 6.2 Configuración Básica en IOS

```
! Crear una clase de mapa para identificar tráfico
class-map match-any DATOS-CRITICOS
 match protocol http
 match protocol https
 match access-group 101

! Crear una política de mapa
policy-map QOS-ENTRADA
 class DATOS-CRITICOS
  set dscp af21
  police 5000000 10000 exceed-action drop
 class default
  set dscp default

! Aplicar la política a una interfaz
interface GigabitEthernet0/1
 service-policy input QOS-ENTRADA
```

### 6.3 Configuración en IOS XE

```
! Definir una política más avanzada
policy-map AVANZADA-QOS
 class DATOS-CRITICOS
  priority percent 30
  police 5000000 10000
 class VIDEO
  bandwidth percent 40
 class VOVP
  priority percent 20
 class default
  bandwidth percent 10

! Aplicar tanto en entrada como salida
interface GigabitEthernet0/0/1
 service-policy input AVANZADA-QOS
 service-policy output AVANZADA-QOS
```

---

## 7. Implementación Avanzada

### 7.1 MPLS Traffic Engineering (MPLS-TE)

Para redes de gran escala, MPLS-TE permite crear túneles explícitos con garantías de ancho de banda:

```
! Habilitar MPLS TE
mpls traffic-eng tunnels

! Crear un tunnel con ancho de banda reservado
interface Tunnel100
 ip address 192.168.100.1 255.255.255.0
 tunnel mode mpls traffic-eng
 tunnel destination 192.168.200.1
 tunnel mpls traffic-eng bandwidth 10000
 tunnel mpls traffic-eng priority 3 3
 tunnel mpls traffic-eng fast-reroute
```

### 7.2 QoS de Extremo a Extremo

Implementar QoS consistente en toda la red requiere coordinación entre múltiples dispositivos:

**En el Acceso**: Clasificar y marcar según requisitos de aplicación.

**En la Distribución**: Aplicar políticas de conformidad y encolamiento.

**En el Núcleo**: Usar MPLS-TE para garantías explícitas de ancho de banda.

**En la Salida**: Aplicar encolamiento para optimizar la entrega.

### 7.3 Hierarchical QoS (H-QoS)

Permite aplicar políticas QoS multinivel, útil en interfaces con múltiples servicios:

```
policy-map PARENT-POLICY
 class CUSTOMER-A
  shape average 100000000
  service-policy CHILD-POLICY-A
 class CUSTOMER-B
  shape average 50000000
  service-policy CHILD-POLICY-B

policy-map CHILD-POLICY-A
 class CRITICO-A
  priority percent 50
 class default
  fair-queue
```

### 7.4 Queue Depth y Tail Drop

Configurar el tamaño máximo de cola para evitar que se agoten los buffers:

```
policy-map QOS-AVANZADA
 class DATOS-CRITICOS
  bandwidth percent 30
  queue-limit 100 packets
 class VIDEO
  bandwidth percent 40
  queue-limit 200 packets
 class default
  bandwidth percent 30
  queue-limit 50 packets
```

### 7.5 Random Early Detection (RED)

Descarta selectivamente paquetes antes de que se llenen los buffers, mejorando el flujo general:

```
policy-map RED-POLICY
 class DATOS
  bandwidth percent 50
  random-detect
```

---

## 8. Consideraciones para Aplicaciones Específicas

### 8.1 VoIP (Voice over IP)

**Requisitos**: Latencia <150 ms, jitter <30 ms, pérdida <1%

**Configuración**:
- Prioridad absoluta (strict priority queuing)
- DSCP EF (Expedited Forwarding - 46)
- Ancho de banda garantizado: 80-128 kbps por llamada

### 8.2 Video Streaming

**Requisitos**: Ancho de banda suficiente, latencia tolerante, pérdida <5%

**Configuración**:
- DSCP AF41 para video adaptativo
- Ancho de banda garantizado basado en resolución
- Buffer adecuado en colas

### 8.3 Datos Interactivos

**Requisitos**: Latencia baja (<100 ms), pérdida mínima

**Configuración**:
- DSCP AF21 para datos normales
- Encolamiento justo (fair queuing)
- Policing para limitar uso excesivo

---

## 9. Monitoreo y Análisis de QoS

### 9.1 Comandos de Verificación

```
! Ver la política aplicada
show service-policy interface GigabitEthernet0/1

! Ver estadísticas de clase
show policy-map interface GigabitEthernet0/1

! Ver información de queues
show interface GigabitEthernet0/1 | include queue

! Análisis de NetFlow para QoS
show flow exporter

! Ver estadísticas DSCP
show queueing interface GigabitEthernet0/1
```

### 9.2 Herramientas de Monitoreo

**Cisco Prime Infrastructure**: Proporciona visibilidad centralizada de QoS en la red.

**NETCONF/YANG**: Permite automatización y monitoreo de configuraciones QoS.

**Telemetría**: Recopila datos en tiempo real para análisis avanzado.

---

## 10. Mejores Prácticas

**1. Clasificación Consistente**: Definir y mantener esquemas de clasificación uniformes en toda la red.

**2. Sobredimensionamiento**: Aunque se use QoS, un ancho de banda suficiente reduce problemas.

**3. Monitoreo Continuo**: Validar que las políticas QoS funcionan como se espera.

**4. Documentación**: Mantener registros detallados de políticas y cambios.

**5. Testing Gradual**: Implementar cambios de QoS de forma incremental en la red.

**6. Prioridades Realistas**: No asignar prioridad crítica a demasiadas aplicaciones.

**7. Revisión Periódica**: Ajustar políticas según cambios en requisitos de negocio.

---

## 11. Conclusión

QoS es esencial para garantizar que aplicaciones críticas reciban los recursos necesarios en redes congestionadas. Desde clasificación básica hasta arquitecturas complejas con MPLS-TE, Cisco proporciona herramientas robustas para implementar QoS en cualquier escala. Una implementación exitosa requiere planificación cuidadosa, monitoreo continuo y ajustes periódicos según las necesidades de la organización.

---

**Documento Preparado**: Octubre 2025
**Versión**: 1.0
**Clasificación**: Referencia Técnica