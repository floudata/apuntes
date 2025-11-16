# Laboratorios de Redes con ContainerLab

## Descripción General

Este directorio contiene laboratorios prácticos de redes utilizando **ContainerLab** con routers **Nokia SR Linux**. Cada laboratorio está diseñado para enseñar conceptos específicos de networking de forma progresiva, desde conectividad básica hasta configuraciones avanzadas.

---

## Estructura de Laboratorios

```
labs/
├── README.md                    # Este archivo
├── basic/                       # Laboratorio básico (2 routers)
│   ├── README.md               # Documentación del lab
│   └── lab1-basic.yml          # Archivo de topología
└── r1-r2-r3/                   # Laboratorio intermedio (3 routers)
    ├── README.md               # Documentación del lab
    ├── basic_r1-r2-r3.yml     # Archivo de topología
    └── configs/                # Configuraciones de inicio
        ├── r1.cfg
        ├── r2.cfg
        └── r3.cfg
```

---

## Laboratorios Disponibles

### 📁 [basic/](basic/) - Laboratorio Básico
**Nivel:** Principiante
**Duración estimada:** 30-45 minutos
**Objetivo:** Aprender los fundamentos de ContainerLab y SR Linux

#### Topología
```mermaid
graph LR
    R1["R1<br/>e1-1: 10.0.0.1/30<br/>Mgmt: 172.25.25.11"]
    R2["R2<br/>e1-1: 10.0.0.2/30<br/>Mgmt: 172.25.25.12"]

    R1 ---|"e1-1 ↔ e1-1<br/>10.0.0.0/30"| R2

    style R1 fill:#e1f5ff,stroke:#333,stroke-width:2px
    style R2 fill:#e1f5ff,stroke:#333,stroke-width:2px
```

#### Características
- **2 routers** Nokia SR Linux (tipo ixrd2)
- **1 enlace** punto a punto
- **Red de management:** 172.25.25.0/24
- **Configuración manual** vía CLI

#### Conceptos que aprenderás
- ✅ Desplegar topologías con ContainerLab
- ✅ Conectarse a routers SR Linux vía SSH
- ✅ Los 3 niveles de `admin-state` (concepto clave de SR Linux)
- ✅ Configurar interfaces físicas y subinterfaces
- ✅ Asignar direcciones IPv4
- ✅ Asociar interfaces a network-instances (VRFs)
- ✅ Verificar conectividad con ping
- ✅ Troubleshooting básico

#### Comandos clave
```bash
# Desplegar
sudo containerlab deploy -t lab1-basic.yml

# Conectarse
ssh admin@clab-lab1-basic-r1

# Destruir
sudo containerlab destroy -t lab1-basic.yml
```

#### Requisitos previos
- Conocimientos básicos de TCP/IP
- Entender conceptos de subnetting
- Familiaridad con línea de comandos

---

### 📁 [r1-r2-r3/](r1-r2-r3/) - Topología de 3 Routers
**Nivel:** Intermedio
**Duración estimada:** 1-2 horas
**Objetivo:** Configuración automatizada con archivos JSON y rutas estáticas

#### Topología
```mermaid
graph LR
    R1["R1<br/>Lo0: 1.1.1.1/32<br/>e1-1: 10.0.0.1/30"]
    R2["R2<br/>Lo0: 2.2.2.2/32<br/>e1-1: 10.0.0.2/30<br/>e1-2: 10.0.0.5/30"]
    R3["R3<br/>Lo0: 3.3.3.3/32<br/>e1-1: 10.0.0.6/30"]

    R1 ---|"e1-1 ↔ e1-1<br/>10.0.0.0/30"| R2
    R2 ---|"e1-2 ↔ e1-1<br/>10.0.0.4/30"| R3

    style R1 fill:#e1f5ff,stroke:#333,stroke-width:2px
    style R2 fill:#ffe1f5,stroke:#333,stroke-width:2px
    style R3 fill:#e1f5ff,stroke:#333,stroke-width:2px
```

#### Características
- **3 routers** Nokia SR Linux en serie
- **2 enlaces** punto a punto
- **Loopback interfaces** en cada router
- **Startup configs** en formato JSON
- **Configuración automática** al desplegar

#### Conceptos que aprenderás
- ✅ Startup configs con ContainerLab
- ✅ Formato JSON para configuraciones SR Linux
- ✅ Configuración de interfaces loopback
- ✅ Esquemas de direccionamiento /30
- ✅ Routing estático (implícito con rutas directamente conectadas)
- ✅ Modelo de datos YANG de SR Linux
- ✅ Network instances avanzadas

#### Configuraciones incluidas
Cada router tiene su archivo de configuración pre-construido:
- **r1.cfg**: Interfaz e1-1 + loopback lo0
- **r2.cfg**: Interfaces e1-1, e1-2 + loopback lo0
- **r3.cfg**: Interfaz e1-1 + loopback lo0

#### Comandos clave
```bash
# Desplegar con configs automáticas
sudo containerlab deploy -t basic_r1-r2-r3.yml

# Inspeccionar
sudo containerlab inspect -t basic_r1-r2-r3.yml

# Ver configuración aplicada
ssh admin@clab-lab1-exercise-r1
A:r1# show interface brief
A:r1# show network-instance default route-table

# Destruir
sudo containerlab destroy -t basic_r1-r2-r3.yml
```

#### Ejercicios propuestos
1. Agregar rutas estáticas para alcanzar todos los loopbacks
2. Configurar OSPF entre los 3 routers
3. Implementar BGP como protocolo de enrutamiento
4. Agregar un 4º router a la topología

---

## Comparación de Laboratorios

| Característica | basic/ | r1-r2-r3/ |
|----------------|--------|-----------|
| **Routers** | 2 | 3 |
| **Complejidad** | Básico | Intermedio |
| **Config manual** | ✅ Sí | ❌ No (automatizada) |
| **Startup configs** | ❌ No | ✅ Sí (JSON) |
| **Loopbacks** | ❌ No | ✅ Sí |
| **Objetivo** | Aprender CLI | Aprender automatización |
| **Tiempo** | 30-45 min | 1-2 horas |
| **Nivel** | Principiante | Intermedio |

### Progresión Visual de Labs

```mermaid
graph LR
    subgraph "Lab 1: basic/"
        B1[R1] --- B2[R2]
    end

    subgraph "Lab 2: r1-r2-r3/"
        R1[R1<br/>+ Loopback] --- R2[R2<br/>+ Loopback]
        R2 --- R3[R3<br/>+ Loopback]
    end

    subgraph "Lab 3: OSPF (próximo)"
        O1[R1<br/>Area 0] --- O2[R2<br/>ABR]
        O2 --- O3[R3<br/>Area 1]
        O2 --- O4[R4<br/>Area 1]
    end

    subgraph "Lab 4: MPLS (futuro)"
        M1[CPE-A] --- M2[PE1<br/>VRF-A]
        M2 --- M3[P<br/>Core]
        M3 --- M4[PE2<br/>VRF-A]
        M4 --- M5[CPE-A]
    end

    basic --> r1-r2-r3
    r1-r2-r3 --> OSPF
    OSPF --> MPLS

    style B1 fill:#e1f5ff
    style B2 fill:#e1f5ff
    style R1 fill:#c8e6c9
    style R2 fill:#c8e6c9
    style R3 fill:#c8e6c9
    style O1 fill:#fff9c4
    style O2 fill:#fff9c4
    style O3 fill:#fff9c4
    style O4 fill:#fff9c4
    style M1 fill:#ffccbc
    style M2 fill:#ffccbc
    style M3 fill:#ffccbc
    style M4 fill:#ffccbc
    style M5 fill:#ffccbc
```

---

## Arquitectura de ContainerLab

```mermaid
graph TB
    subgraph "Host System"
        subgraph "Docker Engine"
            subgraph "Management Network<br/>172.25.25.0/24"
                MGMT[Docker Bridge<br/>clab]
            end

            subgraph "Lab Containers"
                R1[R1 Container<br/>Nokia SR Linux]
                R2[R2 Container<br/>Nokia SR Linux]
                R3[R3 Container<br/>Nokia SR Linux]
            end
        end

        CL[ContainerLab CLI<br/>containerlab]
        YML[Topology File<br/>*.yml]
        CFG[Startup Configs<br/>*.cfg]
    end

    HOST[Your PC/Terminal]

    HOST --> CL
    CL --> YML
    CL --> Docker
    YML -.defines.-> R1
    YML -.defines.-> R2
    YML -.defines.-> R3
    CFG -.applied to.-> R1
    CFG -.applied to.-> R2
    CFG -.applied to.-> R3

    MGMT --- R1
    MGMT --- R2
    MGMT --- R3

    R1 <-->|Data Plane| R2
    R2 <-->|Data Plane| R3

    HOST -.SSH.-> MGMT

    style R1 fill:#4a90e2,color:#fff
    style R2 fill:#4a90e2,color:#fff
    style R3 fill:#4a90e2,color:#fff
    style MGMT fill:#f5a623,color:#000
    style CL fill:#7ed321,color:#000
    style YML fill:#f5f5f5
    style CFG fill:#f5f5f5
```

---

## Guía de Inicio Rápido

### Requisitos Previos

#### 1. Software necesario
```bash
# Verificar Docker
docker --version
# Requerido: Docker 20.10+

# Verificar ContainerLab
containerlab version
# Requerido: ContainerLab 0.40+

# SSH client
ssh -V
```

#### 2. Instalación de ContainerLab
```bash
# Linux (Ubuntu/Debian)
bash -c "$(curl -sL https://get.containerlab.dev)"

# macOS
brew install containerlab
```

#### 3. Pull de imágenes Docker
```bash
# Descargar imagen de Nokia SR Linux
docker pull ghcr.io/nokia/srlinux:latest
```

---

### Flujo de Trabajo Típico

```mermaid
sequenceDiagram
    participant User as Usuario
    participant CL as ContainerLab
    participant Docker as Docker Engine
    participant R1 as Router R1
    participant R2 as Router R2

    User->>CL: containerlab deploy -t lab.yml
    CL->>Docker: Crear network clab
    Docker-->>CL: Network creada
    CL->>Docker: Crear container R1
    Docker->>R1: Iniciar SR Linux
    R1->>R1: Aplicar startup-config
    CL->>Docker: Crear container R2
    Docker->>R2: Iniciar SR Linux
    R2->>R2: Aplicar startup-config
    CL->>Docker: Crear veth links
    Docker-->>CL: Links creados
    CL-->>User: Lab desplegado ✅

    User->>R1: SSH admin@R1
    R1-->>User: CLI prompt
    User->>R1: Configurar interfaces
    User->>R1: commit now
    R1-->>User: Config aplicada

    User->>R1: ping R2
    R1->>R2: ICMP Request
    R2->>R1: ICMP Reply
    R1-->>User: Ping exitoso ✅

    User->>CL: containerlab destroy -t lab.yml
    CL->>Docker: Detener containers
    CL->>Docker: Eliminar containers
    CL-->>User: Lab destruido ✅
```

#### 1. Desplegar laboratorio
```bash
cd /path/to/lab
sudo containerlab deploy -t <archivo.yml>
```

**Salida esperada:**
```
INFO[0000] Containerlab v0.xx.x started
INFO[0000] Creating lab directory: /root/clab-<nombre>
INFO[0000] Creating container: "r1"
INFO[0000] Creating container: "r2"
+---+--------------------+--------------+
| # |        Name        | IPv4 Address |
+---+--------------------+--------------+
| 1 | clab-<lab>-r1     | 172.25.25.11 |
| 2 | clab-<lab>-r2     | 172.25.25.12 |
+---+--------------------+--------------+
```

#### 2. Conectarse a un router
```bash
# Opción 1: SSH (recomendado)
ssh admin@clab-<lab>-r1
# Password: NokiaSrl1!

# Opción 2: Docker exec
docker exec -it clab-<lab>-r1 sr_cli
```

#### 3. Trabajar en el router
```bash
# Prompt inicial
--{ running }--[  ]--
A:r1#

# Ver interfaces
show interface brief

# Entrar a modo configuración
enter candidate

# Hacer cambios
set interface ethernet-1/1 admin-state enable

# Ver cambios sin aplicar
diff

# Aplicar configuración
commit now

# Salir
quit
```

#### 4. Verificar conectividad
```bash
# Desde R1 hacia R2
ping 10.0.0.2 network-instance default

# Ver tabla de rutas
show network-instance default route-table

# Ver tabla ARP
show arpnd arp-entries
```

#### 5. Destruir laboratorio
```bash
sudo containerlab destroy -t <archivo.yml>
```

---

## Conceptos Clave de SR Linux

### 🔑 Los 3 Niveles de Admin-State
**Este es el concepto MÁS IMPORTANTE para trabajar con SR Linux.**

Para que una interfaz funcione, debes habilitar 3 niveles:

```mermaid
graph TD
    A[1. Interface Física] --> B[admin-state enable]
    B --> C[2. Subinterfaz Lógica]
    C --> D[subinterface X admin-state enable]
    D --> E[3. Stack de Protocolo]
    E --> F[ipv4 admin-state enable]
    F --> G[✅ Interfaz Operacional]

    style G fill:#ccffcc
```

#### Nivel 1: Interfaz física
```bash
set interface ethernet-1/1 admin-state enable
```
Enciende el puerto físico (Layer 1).

#### Nivel 2: Subinterfaz
```bash
set interface ethernet-1/1 subinterface 0 admin-state enable
```
Habilita el procesamiento de frames (Layer 2).

#### Nivel 3: Protocolo IPv4
```bash
set interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
```
Activa el stack IPv4 (Layer 3, ARP, routing).

**Sin TODOS estos niveles habilitados, la interfaz NO funcionará.**

---

### 🔑 Network Instances (VRFs)

En SR Linux, una **network-instance** es equivalente a un **VRF**.

```mermaid
graph TB
    subgraph Router["Nokia SR Linux Router"]
        subgraph NI_MGMT["Network Instance: mgmt"]
            MGMT_IF[mgmt0<br/>172.25.25.11/24]
            MGMT_RT[Routing Table]
            MGMT_ARP[ARP Table]
        end

        subgraph NI_DEFAULT["Network Instance: default"]
            DATA_IF1[ethernet-1/1.0<br/>10.0.0.1/30]
            DATA_IF2[ethernet-1/2.0<br/>10.0.0.5/30]
            DATA_RT[Routing Table]
            DATA_ARP[ARP Table]
            DATA_OSPF[OSPF Process]
        end

        subgraph NI_HOST["Network Instance: host"]
            HOST_SYS[Sistema Linux Interno]
        end
    end

    MGMT_IF -.-> MGMT_RT
    MGMT_IF -.-> MGMT_ARP
    DATA_IF1 -.-> DATA_RT
    DATA_IF2 -.-> DATA_RT
    DATA_IF1 -.-> DATA_ARP
    DATA_IF2 -.-> DATA_ARP
    DATA_OSPF -.-> DATA_RT

    style NI_MGMT fill:#fff4e6,stroke:#f39c12,stroke-width:3px
    style NI_DEFAULT fill:#e1f5ff,stroke:#3498db,stroke-width:3px
    style NI_HOST fill:#f0f0f0,stroke:#95a5a6,stroke-width:2px
    style MGMT_IF fill:#ffe6e6
    style DATA_IF1 fill:#e6ffe6
    style DATA_IF2 fill:#e6ffe6
```

**Asociar interfaz a network-instance:**
```bash
set network-instance default interface ethernet-1/1.0
```

**Network instances por defecto:**
- `mgmt`: Management del router (SSH, APIs) - Aislada
- `default`: Tráfico de datos / producción - Principal
- `host`: Comunicación interna del OS - Sistema

**Concepto clave:** Las interfaces DEBEN estar asociadas a una network-instance para poder enrutar tráfico. Cada network-instance tiene su propia tabla de routing, ARP y protocolos completamente aislados.

---

## Comandos de Verificación SR Linux

### Ver información de interfaces
```bash
# Todas las interfaces
show interface

# Interfaz específica
show interface ethernet-1/1

# Solo resumen
show interface brief

# Estadísticas de tráfico
show interface ethernet-1/1 statistics
```

### Ver configuración
```bash
# Configuración completa
info

# Config de interfaz específica
info interface ethernet-1/1

# Formato "set" (comandos)
info flat interface ethernet-1/1
```

### Ver routing
```bash
# Tabla de rutas
show network-instance default route-table

# Protocolos de routing
show network-instance default protocols

# Rutas específicas
show network-instance default route-table ipv4-unicast prefix 10.0.0.0/30
```

### Ver ARP
```bash
# Todas las entradas ARP
show arpnd arp-entries

# ARP de una network-instance
show network-instance default protocols linux arp
```

### Troubleshooting
```bash
# Estado operacional detallado
show interface ethernet-1/1 detail

# Razón de por qué está down
show network-instance default interfaces ethernet-1/1.0

# Logs del sistema
show system logging buffer

# Comparar config candidata vs running
diff
```

---

## Troubleshooting Común

```mermaid
graph TD
    START[Interfaz no funciona] --> CHECK1{Interface física<br/>admin-state enabled?}
    CHECK1 -->|NO| FIX1[set interface eth-X/Y<br/>admin-state enable]
    CHECK1 -->|SÍ| CHECK2{Subinterface<br/>admin-state enabled?}

    CHECK2 -->|NO| FIX2[set interface eth-X/Y<br/>subinterface 0<br/>admin-state enable]
    CHECK2 -->|SÍ| CHECK3{IPv4<br/>admin-state enabled?}

    CHECK3 -->|NO| FIX3[set interface eth-X/Y<br/>subinterface 0 ipv4<br/>admin-state enable]
    CHECK3 -->|SÍ| CHECK4{IP address<br/>configurada?}

    CHECK4 -->|NO| FIX4[set interface eth-X/Y<br/>subinterface 0 ipv4<br/>address X.X.X.X/YY]
    CHECK4 -->|SÍ| CHECK5{Interface asociada<br/>a network-instance?}

    CHECK5 -->|NO| FIX5[set network-instance default<br/>interface eth-X/Y.0]
    CHECK5 -->|SÍ| CHECK6{Peer configurado<br/>correctamente?}

    CHECK6 -->|NO| FIX6[Verificar configuración<br/>del otro router]
    CHECK6 -->|SÍ| SUCCESS[✅ Interfaz OK<br/>Verificar routing]

    FIX1 --> COMMIT[commit now]
    FIX2 --> COMMIT
    FIX3 --> COMMIT
    FIX4 --> COMMIT
    FIX5 --> COMMIT
    FIX6 --> COMMIT
    COMMIT --> VERIFY[Verificar con:<br/>show interface<br/>ping network-instance default]

    style START fill:#ffcccc,stroke:#cc0000,stroke-width:2px
    style SUCCESS fill:#ccffcc,stroke:#00cc00,stroke-width:2px
    style FIX1 fill:#fff4cc,stroke:#ffaa00,stroke-width:2px
    style FIX2 fill:#fff4cc,stroke:#ffaa00,stroke-width:2px
    style FIX3 fill:#fff4cc,stroke:#ffaa00,stroke-width:2px
    style FIX4 fill:#fff4cc,stroke:#ffaa00,stroke-width:2px
    style FIX5 fill:#fff4cc,stroke:#ffaa00,stroke-width:2px
    style FIX6 fill:#fff4cc,stroke:#ffaa00,stroke-width:2px
    style COMMIT fill:#cce5ff,stroke:#0066cc,stroke-width:2px
    style VERIFY fill:#e6ccff,stroke:#6600cc,stroke-width:2px
```

### Problema 1: "no-ip-config"
**Síntoma:** Interfaz down con razón `no-ip-config`

**Causa:** Falta habilitar IPv4 admin-state

**Solución:**
```bash
set interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
commit now
```

---

### Problema 2: "subif-down"
**Síntoma:** Interfaz down con razón `subif-down`

**Causa:** Subinterfaz no está habilitada

**Solución:**
```bash
set interface ethernet-1/1 subinterface 0 admin-state enable
commit now
```

---

### Problema 3: "Network is unreachable"
**Síntoma:** Error al hacer ping

**Causa:** Interfaz no asociada a network-instance

**Solución:**
```bash
set network-instance default interface ethernet-1/1.0
commit now
```

---

### Problema 4: Ping a network-instance incorrecta
**Síntoma:**
```bash
ping 10.0.0.2 network-instance mgmt
From 172.25.25.1 icmp_seq=1 Packet filtered
```

**Causa:** La network-instance `mgmt` solo conoce la red de management

**Solución:**
```bash
# Usar la network-instance correcta
ping 10.0.0.2 network-instance default
```

---

## Checklist de Configuración

Para verificar que una interfaz está completamente configurada:

- [ ] ✅ Interface física habilitada: `admin-state enable`
- [ ] ✅ Subinterface habilitada: `subinterface 0 admin-state enable`
- [ ] ✅ IPv4 habilitado: `ipv4 admin-state enable`
- [ ] ✅ IP configurada: `ipv4 address X.X.X.X/YY`
- [ ] ✅ Asociada a network-instance: `network-instance default interface ethernet-X/Y.Z`
- [ ] ✅ El peer (otro router) configurado correctamente
- [ ] ✅ Ping funciona: `ping <IP> network-instance default`

---

## Gestión de Laboratorios

### Ver estado de laboratorios
```bash
# Todos los labs corriendo
sudo containerlab inspect --all

# Lab específico
sudo containerlab inspect -t lab.yml
```

### Logs y debugging
```bash
# Ver logs de un contenedor
docker logs clab-<lab>-r1

# Ver logs en tiempo real
docker logs -f clab-<lab>-r1

# Entrar al bash del contenedor
docker exec -it clab-<lab>-r1 bash
```

### Limpieza
```bash
# Destruir lab específico
sudo containerlab destroy -t lab.yml

# Destruir TODOS los labs
sudo containerlab destroy --all

# Limpiar imágenes Docker no usadas
docker system prune -a
```

---

## Próximos Laboratorios (Planeados)

### Lab Avanzado: OSPF Multi-Area
- 5+ routers
- Múltiples áreas OSPF
- ABRs (Area Border Routers)
- Route summarization

### Lab Avanzado: BGP eBGP/iBGP
- 4 routers en 2 AS diferentes
- eBGP entre AS
- iBGP dentro de cada AS
- Route reflectors

### Lab Avanzado: MPLS L3VPN
- Topología PE-P-PE
- Múltiples VRFs
- MP-BGP VPNv4
- Clientes con IPs superpuestas

### Lab Automatización
- Ansible playbooks
- Python con gNMI
- CI/CD para configuraciones
- Telemetría streaming

---

## Recursos de Aprendizaje

### Documentación Oficial
- [Nokia SR Linux Documentation](https://documentation.nokia.com/srlinux/)
- [SR Linux CLI Reference](https://documentation.nokia.com/srlinux/SR_Linux_HTML_R24-3/cli-reference/index.html)
- [ContainerLab SR Linux Docs](https://containerlab.dev/manual/kinds/srl/)

### Tutoriales
- [SR Linux Learn](https://learn.srlinux.dev/)
- [ContainerLab Examples](https://github.com/srl-labs/containerlab-examples)
- [SR Labs Tutorials](https://github.com/srl-labs)

### Comunidad
- [Nokia SR Linux Discord](https://learn.srlinux.dev/community/)
- [ContainerLab GitHub Discussions](https://github.com/srl-labs/containerlab/discussions)

---

## Tips y Mejores Prácticas

### 💡 Tip 1: Usa `diff` antes de `commit`
Siempre verifica cambios antes de aplicarlos:
```bash
enter candidate
set <configuración>
diff              # ← Ver cambios
commit now
```

### 💡 Tip 2: Guarda configuraciones
```bash
# Guardar config actual
info flat > /tmp/backup.cfg

# Desde el host
docker cp clab-<lab>-r1:/tmp/backup.cfg ./
```

### 💡 Tip 3: Usa aliases de SSH
Agrega a `~/.ssh/config`:
```
Host r1
  HostName clab-lab1-basic-r1
  User admin
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

Luego: `ssh r1` (sin password ni nombre largo)

### 💡 Tip 4: Automatiza con scripts
```bash
#!/bin/bash
# deploy-and-test.sh
sudo containerlab deploy -t lab.yml
sleep 10
ssh admin@clab-lab-r1 "show interface brief"
```

---

## FAQ

### ¿Por qué usar ContainerLab en lugar de GNS3/EVE-NG?
- ✅ **Más rápido:** Deploy en 10 segundos vs 5 minutos
- ✅ **Menos recursos:** 10x menos RAM que VMs
- ✅ **Reproducible:** Topologías como código (YAML)
- ✅ **APIs nativas:** Telemetría moderna, gNMI, gRPC
- ✅ **CI/CD friendly:** Automatización fácil

### ¿Necesito licencias de Nokia?
No. Nokia SR Linux es gratuito para laboratorios.

### ¿Puedo usar otros vendors?
Sí. ContainerLab soporta:
- Arista cEOS
- Cisco XRv9000, Nexus 9000v
- Juniper cRPD, vMX
- MikroTik RouterOS
- FRRouting
- Cumulus VX

### ¿Cómo exporto configuraciones?
```bash
# Desde SR Linux
info flat > /tmp/config.txt

# Copiar al host
docker cp clab-<lab>-r1:/tmp/config.txt ./
```

### ¿Los cambios persisten?
**Durante el lab:** Sí, mientras no destruyas el lab.

**Después de destroy:** No, a menos que uses `startup-config`.

---

## Contribuir con Nuevos Labs

Si quieres agregar un nuevo laboratorio:

1. Crea un directorio con nombre descriptivo
2. Incluye:
   - `README.md` con documentación completa
   - Archivo YAML de topología
   - Configs de inicio (si aplica)
   - Diagramas mermaid
3. Prueba el lab completamente
4. Documenta los objetivos de aprendizaje
5. Incluye ejercicios propuestos

---

## Licencia

Estos laboratorios son material educativo de uso libre para aprendizaje personal y académico.

---

## Soporte

**Para problemas técnicos:**
1. Revisa la sección de Troubleshooting
2. Consulta los logs: `docker logs <contenedor>`
3. Verifica los requisitos de sistema
4. Busca en ContainerLab GitHub Issues

**Para preguntas de networking:**
1. Consulta la documentación de SR Linux
2. Revisa el tutorial de ContainerLab en `/networks/containerlab.md`
3. Únete a las comunidades de SR Linux

---

**Última actualización:** 2025-11-15
**Versión:** 1.0

---

**¡Disfruta aprendiendo networking con ContainerLab! 🚀**
