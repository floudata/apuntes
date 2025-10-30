# Notas sobre Topologías de Red

Este directorio contiene una colección de notas detalladas y laboratorios prácticos sobre diversas topologías de red, diseñados para niveles de certificación CCNA y CCNP. Cada documento se enfoca en un concepto específico, proporcionando análisis teóricos, diagramas y configuraciones prácticas.

## Contenido

### 📄 [layer_2_3_v1.md](./layer_2_3_v1.md)

- **Tema:** Router Inter-VLAN con Segmentación Departamental.
- **Descripción:** Implementa un diseño básico con dos segmentos LAN (Ventas e Ingeniería) conectados a través de un router de Capa 3. Es ideal para entender el routing inter-red fundamental.
- **Nivel:** CCNA y CCNP.
- **Laboratorios:** Configuración básica de IP, pruebas de conectividad, ACLs, NAT, OSPF, HSRP, BGP, MPLS y QoS.

### 📄 [layer_2_3_v2.md](./layer_2_3_v2.md)

- **Tema:** Límite entre Capa 2 (Local) y Capa 3 (Inter-Red).
- **Descripción:** Ilustra cómo dos dominios de Capa 2 (por ejemplo, dos pisos de un edificio) son interconectados por un dispositivo de Capa 3. Explora el flujo de tráfico intra-VLAN e inter-VLAN y los dominios de broadcast.
- **Nivel:** CCNA y CCNP.
- **Laboratorios:** Router-on-a-Stick, SVIs en switches Capa 3, ACLs, Private VLANs y VRF-Lite.

### 📄 [layer_3_v1.md](./layer_3_v1.md)

- **Tema:** Conectividad Básica de Interfaz (Capa 3).
- **Descripción:** Cubre el escenario más fundamental: la conexión punto a punto entre dos routers. Es la base para entender direccionamiento IP, ARP y rutas conectadas directamente.
- **Nivel:** CCNA y CCNP.
- **Laboratorios:** Configuración de IP, troubleshooting de interfaces, análisis de encapsulación y MTU, seguridad básica de interfaz, IP Accounting y QoS.

### 📄 [mpls_vpn_v1.md](./mpls_vpn_v1.md)

- **Tema:** MPLS L3VPN con VRF (Virtual Routing and Forwarding).
- **Descripción:** Detalla una arquitectura de Service Provider para ofrecer servicios de VPN a múltiples clientes, permitiendo el uso de espacios de direccionamiento IP superpuestos gracias al aislamiento de VRF.
- **Nivel:** CCNA y CCNP.
- **Laboratorios:** Configuración de VRF, MPLS en el core, OSPF, MP-BGP para VPNv4, redistribución de rutas y troubleshooting de MPLS.
