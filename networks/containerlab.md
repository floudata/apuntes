# Tutorial Completo de Containerlab
## De Básico a Avanzado - Networking con Contenedores

> **Autor:** Tutorial para automatización de redes con IA  
> **Nivel:** Básico → Intermedio → Avanzado  
> **Objetivo:** Dominar Containerlab para simulación de redes y generación de telemetría para modelos de IA

---

# 📋 Tabla de Contenidos

- [Nivel Básico](#nivel-básico)
  - [1. Introducción y Conceptos](#1-introducción-y-conceptos)
  - [2. Instalación](#2-instalación)
  - [3. Primera Topología](#3-primera-topología)
  - [4. Comandos Esenciales](#4-comandos-esenciales)
  - [5. Configuración por CLI](#5-configuración-por-cli)
- [Nivel Intermedio](#nivel-intermedio)
  - [6. Topologías Complejas](#6-topologías-complejas)
  - [7. Tipos de Nodos](#7-tipos-de-nodos)
  - [8. Startup Configs](#8-startup-configs)
  - [9. Routing Protocols](#9-routing-protocols)
  - [10. Troubleshooting Básico](#10-troubleshooting-básico)
- [Nivel Avanzado](#nivel-avanzado)
  - [11. MPLS L3VPN](#11-mpls-l3vpn)
  - [12. Telemetría y Monitoreo](#12-telemetría-y-monitoreo)
  - [13. Integración con Python](#13-integración-con-python)
  - [14. Automation y APIs](#14-automation-y-apis)
  - [15. Escenarios para Demo de IA](#15-escenarios-para-demo-de-ia)

---

# Nivel Básico

## 1. Introducción y Conceptos

### ¿Qué es Containerlab?

**Containerlab** es una herramienta CLI para orquestar y gestionar laboratorios de redes basados en contenedores. A diferencia de EVE-NG o GNS3 que usan máquinas virtuales pesadas, Containerlab usa contenedores Docker que son:

- ⚡ **Rápidos**: Deploy en segundos
- 💚 **Eficientes**: Consumen 10x menos recursos que VMs
- 🔄 **Reproducibles**: Definidos como código (YAML)
- 🌐 **Multivendor**: Nokia, Arista, Cisco, Juniper, etc.

### ¿Por qué Containerlab para IA/ML en Redes?

1. **Telemetría moderna**: gNMI, gRPC streaming nativo
2. **APIs programáticas**: REST, NETCONF, Python SDK
3. **Datos estructurados**: JSON nativo, no parsing de CLI
4. **Escalabilidad**: 20+ nodos en un laptop
5. **CI/CD friendly**: Perfecto para testing automatizado

### Arquitectura

```
┌─────────────────────────────────────┐
│   Containerlab CLI                  │
│   (orquestación)                    │
└──────────────┬──────────────────────┘
               │
               ↓
┌──────────────────────────────────────┐
│   Docker Engine                      │
│   ├── Contenedor: Router PE1        │
│   ├── Contenedor: Router P1         │
│   ├── Contenedor: Switch SW1        │
│   └── Contenedor: Linux Host        │
└──────────────────────────────────────┘
```

### Conceptos Clave

- **Lab**: Una topología completa definida en YAML
- **Node**: Un dispositivo (router, switch, host, servidor)
- **Link**: Conexión virtual entre nodos
- **Kind**: Tipo de dispositivo (nokia_srlinux, linux, arista_ceos)
- **Type**: Modelo/tamaño del dispositivo (ixrd3, ixrd2, ixrd1)

---

## 2. Instalación

### Requisitos Previos

**Hardware mínimo:**
- CPU: 4 cores
- RAM: 8GB (16GB recomendado)
- Disco: 50GB libres
- Virtualización habilitada (VT-x/AMD-V)

**Software:**
- Sistema operativo: Linux (Ubuntu 22.04/24.04 recomendado)
- Docker Engine
- Permisos de root/sudo

### Instalación en Linux (Ubuntu/Debian)

```bash
# 1. Actualizar sistema
sudo apt update && sudo apt upgrade -y

# 2. Instalar Docker
sudo apt install -y docker.io

# 3. Iniciar Docker
sudo systemctl start docker
sudo systemctl enable docker

# 4. Añadir usuario al grupo docker (opcional, evita usar sudo)
sudo usermod -aG docker $USER
# Cerrar sesión y volver a entrar para aplicar cambios

# 5. Verificar Docker
docker --version
docker ps

# 6. Instalar Containerlab
bash -c "$(curl -sL https://get.containerlab.dev)"

# Output esperado:
# getting containerlab latest version...
# downloading containerlab 0.50.0 from https://github.com/srl-labs/containerlab/releases/download/v0.50.0/containerlab_0.50.0_linux_amd64.tar.gz
# installing containerlab 0.50.0 to /usr/bin/containerlab
# containerlab installed successfully!

# 7. Verificar instalación
containerlab version

# Output esperado:
#                           _                   _       _     
#                 _        (_)                 | |     | |    
#  ____ ___  ____| |_  ____  _ ____   _____  __| | ____| | _  
# / ___) _ \|  _ |  _)/ _  || |  _ \ / ___ |/ _  |/ _  | || \ 
#( (__| |_| | | | | |_( ( | || | | | ( (___( (_| ( (_| | |_) )
# \____)___/|_| |_|\___)_||_||_|_| |_|\____|\____|\____|____/ 
#
# version: 0.50.0
# commit: abc123def
# date: 2024-01-15T10:00:00Z
# source: https://github.com/srl-labs/containerlab
```

### Instalación en Windows (WSL2)

```powershell
# 1. Habilitar WSL2
wsl --install

# 2. Instalar Ubuntu desde Microsoft Store
# Buscar "Ubuntu 22.04 LTS" y instalar

# 3. Abrir Ubuntu WSL2 y ejecutar:
sudo apt update
sudo apt install -y docker.io

# 4. Instalar Containerlab
bash -c "$(curl -sL https://get.containerlab.dev)"

# 5. Iniciar Docker
sudo service docker start
```

### Instalación en macOS

```bash
# 1. Instalar Docker Desktop
# Descargar desde: https://www.docker.com/products/docker-desktop

# 2. Instalar Containerlab
bash -c "$(curl -sL https://get.containerlab.dev)"

# 3. Verificar
containerlab version
```

### Descargar Imágenes Necesarias

```bash
# Nokia SR Linux (gratis, recomendada)
docker pull ghcr.io/nokia/srlinux:latest

# Output esperado:
# latest: Pulling from nokia/srlinux
# 7264a8db6415: Pull complete 
# 5b7c4e2e64e4: Pull complete 
# 3e5b4c8a7d2e: Pull complete 
# Digest: sha256:abc123def456...
# Status: Downloaded newer image for ghcr.io/nokia/srlinux:latest
# ghcr.io/nokia/srlinux:latest

# Alpine Linux (para hosts)
docker pull alpine:latest

# Output esperado:
# latest: Pulling from library/alpine
# 4abcf2066143: Pull complete 
# Digest: sha256:xyz789abc123...
# Status: Downloaded newer image for alpine:latest
# docker.io/library/alpine:latest

# Verificar imágenes descargadas
docker images

# Output esperado:
# REPOSITORY                  TAG       IMAGE ID       CREATED        SIZE
# ghcr.io/nokia/srlinux      latest    a1b2c3d4e5f6   2 weeks ago    2.1GB
# alpine                     latest    9c6f07244728   3 weeks ago    7.34MB
```

---

## 3. Primera Topología

### Topología Simple: 2 Routers

Vamos a crear nuestra primera topología con 2 routers Nokia SR Linux conectados entre sí.

**Crear archivo `lab1-basic.yml`:**

```yaml
name: lab1-basic

topology:
  nodes:
    # Router 1
    r1:
      kind: nokia_srlinux
      type: ixrd2
      image: ghcr.io/nokia/srlinux:latest
    
    # Router 2
    r2:
      kind: nokia_srlinux
      type: ixrd2
      image: ghcr.io/nokia/srlinux:latest
  
  links:
    # Conectar r1:ethernet-1/1 con r2:ethernet-1/1
    - endpoints: ["r1:e1-1", "r2:e1-1"]
```

**Explicación:**
- `name`: Nombre del laboratorio
- `nodes`: Definición de dispositivos
- `kind`: Tipo de plataforma (nokia_srlinux)
- `type`: Modelo del dispositivo (ixrd2 = mediano, ixrd3 = grande, ixrd1 = pequeño)
- `links`: Conexiones entre dispositivos
- `endpoints`: Par de interfaces a conectar (formato: `nodo:interfaz`)

### Desplegar el Laboratorio

```bash
# Deploy
sudo containerlab deploy -t lab1-basic.yml

# Output esperado:
INFO[0000] Containerlab v0.50.0 started                 
INFO[0000] Parsing & checking topology file: lab1-basic.yml 
INFO[0000] Creating lab directory: /root/clab-lab1-basic 
INFO[0000] Creating docker network: Name="clab", IPv4Subnet="172.20.20.0/24", IPv6Subnet="2001:172:20:20::/64", MTU=1500 
INFO[0000] Creating container: "r1"                     
INFO[0001] Creating container: "r2"                     
INFO[0002] Creating virtual wire: r1:e1-1 <--> r2:e1-1 
INFO[0003] Adding containerlab host entries to /etc/hosts file 
INFO[0003] Executed command "ip link set dev eth1 up && ip addr add 172.20.20.11/24 dev eth1" on the node "r1". stdout: 
INFO[0003] Executed command "ip link set dev eth1 up && ip addr add 172.20.20.12/24 dev eth1" on the node "r2". stdout: 
INFO[0004] 🎉 New containerlab version 0.50.0 is available! Release notes: https://containerlab.dev/rn/0.50/
Run 'containerlab version upgrade' to upgrade or go check other installation options at https://containerlab.dev/install/ 
+---+-----------------+--------------+----------------------------+-------+---------+----------------+----------------------+
| # |      Name       | Container ID |           Image            | Kind  |  State  |  IPv4 Address  |     IPv6 Address     |
+---+-----------------+--------------+----------------------------+-------+---------+----------------+----------------------+
| 1 | clab-lab1-r1    | 8f8c8d8d9e0a | ghcr.io/nokia/srlinux      | srl   | running | 172.20.20.11/24| 2001:172:20:20::11/64|
| 2 | clab-lab1-r2    | 7a7b7c7d8e9f | ghcr.io/nokia/srlinux      | srl   | running | 172.20.20.12/24| 2001:172:20:20::12/64|
+---+-----------------+--------------+----------------------------+-------+---------+----------------+----------------------+
```

### Verificar el Estado

```bash
# Ver contenedores activos
sudo docker ps

# Output esperado:
CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS          PORTS     NAMES
8f8c8d8d9e0a   ghcr.io/nokia/srlinux      "/tini -- fixuid -q …"   30 seconds ago   Up 28 seconds             clab-lab1-basic-r1
7a7b7c7d8e9f   ghcr.io/nokia/srlinux      "/tini -- fixuid -q …"   30 seconds ago   Up 28 seconds             clab-lab1-basic-r2

# Inspeccionar el lab
sudo containerlab inspect -t lab1-basic.yml

# Output esperado:
+---+-----------------+--------------+----------------------------+-------+---------+----------------+----------------------+
| # |      Name       | Container ID |           Image            | Kind  |  State  |  IPv4 Address  |     IPv6 Address     |
+---+-----------------+--------------+----------------------------+-------+---------+----------------+----------------------+
| 1 | clab-lab1-r1    | 8f8c8d8d9e0a | ghcr.io/nokia/srlinux      | srl   | running | 172.20.20.11/24| 2001:172:20:20::11/64|
| 2 | clab-lab1-r2    | 7a7b7c7d8e9f | ghcr.io/nokia/srlinux      | srl   | running | 172.20.20.12/24| 2001:172:20:20::12/64|
+---+-----------------+--------------+----------------------------+-------+---------+----------------+----------------------+

# Ver detalles de un nodo específico
sudo docker inspect clab-lab1-basic-r1

# Output esperado (resumido):
[
    {
        "Id": "8f8c8d8d9e0a...",
        "Created": "2024-01-15T10:30:45.123456789Z",
        "Path": "/tini",
        "Args": [...],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 12345,
            "ExitCode": 0,
            ...
        },
        "Image": "sha256:abc123...",
        "Name": "/clab-lab1-basic-r1",
        ...
    }
]
```

### Destruir el Laboratorio

```bash
# Destroy (elimina contenedores y limpia)
sudo containerlab destroy -t lab1-basic.yml

# Output esperado:
INFO[0000] Parsing & checking topology file: lab1-basic.yml 
Are you sure you want to destroy lab "lab1-basic"? [y/N]: y
INFO[0001] Destroying lab: lab1-basic                   
INFO[0001] Removed container: clab-lab1-basic-r1        
INFO[0001] Removed container: clab-lab1-basic-r2        
INFO[0002] Removing containerlab host entries from /etc/hosts file 
INFO[0002] Removing /root/clab-lab1-basic directory...  
INFO[0002] Cleanup complete!
```

---

## 4. Comandos Esenciales

### Gestión de Labs

```bash
# Deploy un lab
sudo containerlab deploy -t <topology.yml>

# Deploy con cleanup previo (destruye y recrea)
sudo containerlab deploy -t <topology.yml> --reconfigure

# Destroy un lab
sudo containerlab destroy -t <topology.yml>

# Destroy todos los labs
sudo containerlab destroy --all

# Listar labs activos
sudo containerlab inspect --all

# Inspeccionar un lab específico
sudo containerlab inspect -t <topology.yml>

# Ver detalles en formato JSON
sudo containerlab inspect -t <topology.yml> --format json
```

### Interacción con Nodos

```bash
# Acceder a CLI de un nodo (SR Linux)
sudo docker exec -it clab-<lab-name>-<node-name> sr_cli

# Ejemplo concreto
sudo docker exec -it clab-lab1-basic-r1 sr_cli

# Ejecutar comando bash en un nodo
sudo docker exec -it clab-lab1-basic-r1 bash

# Ejecutar comando único (sin interactivo)
sudo docker exec clab-lab1-basic-r1 sr_cli "show version"

# Ver logs de un nodo
sudo docker logs clab-lab1-basic-r1

# Logs en tiempo real (follow)
sudo docker logs -f clab-lab1-basic-r1
```

### Gestión de Recursos

```bash
# Ver uso de recursos de contenedores
sudo docker stats

# Output esperado:
CONTAINER ID   NAME                 CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
8f8c8d8d9e0a   clab-lab1-basic-r1   2.45%     856.3MiB / 15.6GiB    5.36%     1.2kB / 856B      15.4MB / 8.19kB   42
7a7b7c7d8e9f   clab-lab1-basic-r2   2.12%     823.1MiB / 15.6GiB    5.15%     856B / 1.2kB      14.8MB / 4.1kB    38

# Parar un nodo específico
sudo docker stop clab-lab1-basic-r1

# Output esperado:
clab-lab1-basic-r1

# Iniciar un nodo parado
sudo docker start clab-lab1-basic-r1

# Output esperado:
clab-lab1-basic-r1

# Reiniciar un nodo
sudo docker restart clab-lab1-basic-r1

# Output esperado:
clab-lab1-basic-r1

# Ver todos los contenedores (incluso parados)
sudo docker ps -a

# Output esperado:
CONTAINER ID   IMAGE                   COMMAND                  CREATED        STATUS                     PORTS     NAMES
8f8c8d8d9e0a   ghcr.io/nokia/srlinux  "/tini -- fixuid -q …"   5 minutes ago  Up 3 minutes                         clab-lab1-basic-r1
7a7b7c7d8e9f   ghcr.io/nokia/srlinux  "/tini -- fixuid -q …"   5 minutes ago  Up 5 minutes                         clab-lab1-basic-r2
```

### Networking

```bash
# Ver redes Docker creadas
docker network ls

# Output esperado:
NETWORK ID     NAME      DRIVER    SCOPE
9876543210ab   bridge    bridge    local
1234567890cd   clab      bridge    local
abcdef123456   host      host      local
fedcba654321   none      null      local

# Inspeccionar red de Containerlab
docker network inspect clab

# Output esperado (resumido):
[
    {
        "Name": "clab",
        "Id": "1234567890cd...",
        "Created": "2024-01-15T10:30:44.123456789Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": true,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.20.0/24",
                    "Gateway": "172.20.20.1"
                },
                {
                    "Subnet": "2001:172:20:20::/64",
                    "Gateway": "2001:172:20:20::1"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "8f8c8d8d9e0a...": {
                "Name": "clab-lab1-basic-r1",
                "EndpointID": "abcd1234...",
                "MacAddress": "02:42:ac:14:14:0b",
                "IPv4Address": "172.20.20.11/24",
                "IPv6Address": "2001:172:20:20::11/64"
            },
            "7a7b7c7d8e9f...": {
                "Name": "clab-lab1-basic-r2",
                "EndpointID": "efgh5678...",
                "MacAddress": "02:42:ac:14:14:0c",
                "IPv4Address": "172.20.20.12/24",
                "IPv6Address": "2001:172:20:20::12/64"
            }
        },
        ...
    }
]

# Ver IPs asignadas a nodos
sudo containerlab inspect -t lab1-basic.yml | grep -A 5 "IPv4"

# Output esperado:
|  IPv4 Address  |     IPv6 Address     |
+----------------+----------------------+
| 172.20.20.11/24| 2001:172:20:20::11/64|
| 172.20.20.12/24| 2001:172:20:20::12/64|
```

---

## 5. Configuración por CLI

### Acceder a la CLI

```bash
# Método 1: Docker exec (recomendado)
sudo docker exec -it clab-lab1-basic-r1 sr_cli

# Método 2: SSH (requiere conocer IP)
ssh admin@172.20.20.2
# Password: NokiaSrl1!
```

### Navegación Básica en SR Linux CLI

```bash
# Al entrar verás:
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.

--{ running }--[  ]--
A:r1#

# Modos disponibles:
# - running: Modo operacional (solo lectura)
# - candidate: Modo configuración (lectura/escritura)
```

### Comandos Show Básicos

```bash
# Ver versión del sistema
A:r1# show version

# Output esperado:
----------------------------------------------------------
Hostname             : r1
Chassis Type         : 7220 IXR-D3
Part Number          : Sim Part No.
Serial Number        : Sim Serial No.
System HW MAC Address: 02:00:00:FF:00:00
Software Version     : v23.10.1
Build Number         : 318-g8c8d8f6f
Architecture         : x86_64
Last Booted          : 2024-01-15T10:30:45.123Z
Total Memory         : 24052875 kB
Free Memory          : 15234567 kB
----------------------------------------------------------

# Ver todas las interfaces
A:r1# show interface

# Output esperado:
=======================================================================
ethernet-1/1 is up, speed 100G, type None
  ethernet-1/1.0 is up
    Network-instances:
      * default
    Encapsulation   : null
    Type            : routed
    IPv4 addr    : 10.0.0.1/30 (static, None)
=======================================================================
mgmt0 is up, speed 1G, type None
  mgmt0.0 is up
    Network-instances:
      * mgmt
    Encapsulation   : null
    Type            : routed
    IPv4 addr    : 172.20.20.11/24 (dhcp, preferred)
    IPv6 addr    : 2001:172:20:20::11/64 (dhcp, preferred)
    IPv6 addr    : fe80::42:acff:fe14:140b/64 (link-layer, preferred)
=======================================================================
system0 is up, speed None, type None
  system0.0 is up
    Network-instances:
      * default
    Encapsulation   : null
    Type            : None
    IPv4 addr    : 1.1.1.1/32 (static, preferred)
=======================================================================

# Ver interfaces resumido
A:r1# show interface brief

# Output esperado:
+---------------+---------------+---------------+---------------+---------------+
|     Port      |  Admin State  |  Oper State   |     Speed     |     Type      |
+===============+===============+===============+===============+===============+
| ethernet-1/1  | enable        | up            | 100G          | None          |
| ethernet-1/2  | disable       | down          | 100G          | None          |
| ethernet-1/3  | disable       | down          | 100G          | None          |
| mgmt0         | enable        | up            | 1G            | None          |
| system0       | enable        | up            | None          | None          |
+---------------+---------------+---------------+---------------+---------------+

# Ver configuración actual
A:r1# info

# Output esperado:
    set / interface ethernet-1/1 admin-state enable
    set / interface ethernet-1/1 description "Link to R2"
    set / interface ethernet-1/1 subinterface 0 admin-state enable
    set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
    set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.1/30
    set / interface system0 admin-state enable
    set / interface system0 subinterface 0 admin-state enable
    set / interface system0 subinterface 0 ipv4 admin-state enable
    set / interface system0 subinterface 0 ipv4 address 1.1.1.1/32
    set / network-instance default type default
    set / network-instance default admin-state enable
    set / network-instance default interface ethernet-1/1.0
    set / network-instance default interface system0.0

# Ver configuración de un objeto específico
A:r1# info interface ethernet-1/1

# Output esperado:
    interface ethernet-1/1 {
        admin-state enable
        description "Link to R2"
        subinterface 0 {
            admin-state enable
            ipv4 {
                admin-state enable
                address 10.0.0.1/30 {
                }
            }
        }
    }

# Ver estado del sistema
A:r1# show system information

# Output esperado:
--------------------------------------------------------------------
Hostname             : r1
Chassis Type         : 7220 IXR-D3
Part Number          : Sim Part No.
Serial Number        : Sim Serial No.
System HW MAC Address: 02:00:00:FF:00:00
Software Version     : v23.10.1
Build Number         : 318-g8c8d8f6f
Architecture         : x86_64
Last Booted          : 2024-01-15T10:30:45.123Z
Current Time         : 2024-01-15T11:45:23.456Z
--------------------------------------------------------------------

# Ver procesos
A:r1# show system processes

# Output esperado (resumido):
PID   PPID  VSZ      RSS      %MEM  STATE  TIME      COMMAND
1     0     234432   12456    0.1   S      0:02      /tini
42    1     1234567  456789   1.8   S      0:45      sr_linux
123   42    987654   234567   0.9   S      0:12      sr_mgmt_server
...

# Ver uso de memoria
A:r1# show system memory

# Output esperado:
--------------------------------------------------------------------
                 Total        Used        Free      Shared
Memory:       24052875    8818308    15234567      123456
Swap:          2097152          0     2097152
--------------------------------------------------------------------

# Ver tabla de routing
A:r1# show network-instance default route-table

# Output esperado:
--------------------------------------------------------------------
IPv4 unicast route table of network instance default
--------------------------------------------------------------------
+---------------+-------+------------+------------+--------+---------+
|    Prefix     | ID    | Route Type | Route Owner| Active | Metric  |
+===============+=======+============+============+========+=========+
| 1.1.1.1/32    | 0     | local      | local_route| True   | 0       |
| 10.0.0.0/30   | 0     | local      | local_route| True   | 0       |
| 10.0.0.1/32   | 0     | host       | local_route| True   | 0       |
+---------------+-------+------------+------------+--------+---------+
--------------------------------------------------------------------
IPv4 routes total                    : 3
IPv4 prefixes with active routes     : 3
IPv4 prefixes with active ECMP routes: 0
--------------------------------------------------------------------

# Ver neighbors ARP
A:r1# show network-instance default arp

# Output esperado:
--------------------------------------------------------------------
ARP entries for network-instance "default"
--------------------------------------------------------------------
+----------------+-------------------+-------------+----------+-------+
|  IP Address    |   MAC Address     |  Interface  |  State   | Timer |
+================+===================+=============+==========+=======+
| 10.0.0.2       | 00:1a:c5:00:00:01 | ethernet-1/1| reachable| 234   |
+----------------+-------------------+-------------+----------+-------+
--------------------------------------------------------------------
Total entries: 1
--------------------------------------------------------------------
```

### Configuración Básica - Paso a Paso

#### Paso 1: Entrar a Modo Configuración

```bash
# Desde modo running
A:r1# enter candidate

# Ahora estás en modo candidate
--{ candidate shared default }--[  ]--
A:r1#
```

#### Paso 2: Configurar Interfaz

```bash
# Navegar a la interfaz
A:r1# interface ethernet-1/1

# Habilitar la interfaz
A:r1# admin-state enable

# Añadir descripción
A:r1# description "Link to R2"

# Configurar subinterface (equivalente a interfaz L3)
A:r1# subinterface 0

# Asignar IP
A:r1# ipv4 address 10.0.0.1/30

# Ver cambios pendientes
A:r1# diff

# Output esperado:
+ interface ethernet-1/1 {
+     admin-state enable
+     description "Link to R2"
+     subinterface 0 {
+         ipv4 {
+             address 10.0.0.1/30 {
+             }
+         }
+     }
+ }
```

#### Paso 3: Aplicar Configuración

```bash
# Commit (aplicar cambios)
A:r1# commit now

# Output esperado:
All changes have been committed. Starting version validation.
Leaving candidate mode.

--{ running }--[  ]--
A:r1# 

# Volver a running mode
A:r1# quit

# Verificar configuración aplicada
A:r1# show interface ethernet-1/1

# Output esperado:
===============================================================================
ethernet-1/1 is up, speed 100G, type None
  ethernet-1/1.0 is up
    Network-instances:
      * default
    Encapsulation   : null
    Type            : routed
    IPv4 addr    : 10.0.0.1/30 (static, preferred)
    IPv6 addr    : None
===============================================================================
```

#### Paso 4: Configurar Loopback

```bash
A:r1# enter candidate

--{ candidate shared default }--[  ]--
A:r1# 

# Configurar interfaz system (loopback en SR Linux)
A:r1# interface system0
A:r1# admin-state enable
A:r1# subinterface 0
A:r1# ipv4 address 1.1.1.1/32

A:r1# commit now

# Output esperado:
All changes have been committed. Starting version validation.
Leaving candidate mode.
```

#### Paso 5: Verificar Conectividad

```bash
# Hacer ping desde R1 a R2
A:r1# ping 10.0.0.2 network-instance default

# Output esperado:
Using network instance default
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.123 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.098 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.115 ms
64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=0.102 ms
64 bytes from 10.0.0.2: icmp_seq=5 ttl=64 time=0.109 ms

--- 10.0.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4095ms
rtt min/avg/max/mdev = 0.098/0.109/0.123/0.008 ms

# Traceroute
A:r1# traceroute 10.0.0.2 network-instance default

# Output esperado:
Using network instance default
traceroute to 10.0.0.2 (10.0.0.2), 30 hops max, 60 byte packets
 1  10.0.0.2 (10.0.0.2)  0.145 ms  0.089 ms  0.078 ms

# Ver estadísticas de interfaz
A:r1# show interface ethernet-1/1 statistics

# Output esperado:
===============================================================================
ethernet-1/1 statistics
===============================================================================
  In packets                 : 1234
  In octets                  : 123456
  In unicast packets         : 1200
  In broadcast packets       : 34
  In multicast packets       : 0
  In discarded packets       : 0
  In error packets           : 0
  In FCS error packets       : 0
  
  Out packets                : 5678
  Out octets                 : 567890
  Out unicast packets        : 5600
  Out broadcast packets      : 78
  Out multicast packets      : 0
  Out discarded packets      : 0
  Out error packets          : 0
===============================================================================
```

### Configuración de R2 (para completar el lab)

```bash
# Acceder a R2
sudo docker exec -it clab-lab1-basic-r2 sr_cli

# Configurar
A:r2# enter candidate

# Loopback
A:r2# interface system0
A:r2# admin-state enable
A:r2# subinterface 0
A:r2# ipv4 address 2.2.2.2/32

# Interfaz hacia R1
A:r2# interface ethernet-1/1
A:r2# admin-state enable
A:r2# description "Link to R1"
A:r2# subinterface 0
A:r2# ipv4 address 10.0.0.2/30

A:r2# commit now

# Verificar conectividad
A:r2# ping 10.0.0.1 network-instance default
A:r2# ping 1.1.1.1 network-instance default
```

### Guardar Configuración

```bash
# Las configs de SR Linux son persistentes por defecto
# Están guardadas en: /etc/opt/srlinux/config.json

# Para backup manual
sudo docker exec clab-lab1-basic-r1 cat /etc/opt/srlinux/config.json > r1-backup.json

# Para copiar archivo del contenedor a host
sudo docker cp clab-lab1-basic-r1:/etc/opt/srlinux/config.json ./r1-config.json
```

### Tips de CLI

```bash
# Autocompletar con TAB
A:r1# int<TAB>
interface

# Ver opciones disponibles con ?
A:r1# show ?

# Historial de comandos (flechas arriba/abajo)
↑ ↓

# Salir de modo configuración sin commit (descartar cambios)
A:r1# quit

# Ver solo cambios desde último commit
A:r1# diff

# Ver configuración en formato tree
A:r1# info from state

# Buscar en output
A:r1# show interface | grep ethernet-1/1
```

---

## Ejercicio Práctico 1: Topología de 3 Routers

**Objetivo:** Crear una topología lineal R1 -- R2 -- R3 y configurar conectividad completa.

### Archivo `lab1-exercise.yml`

```yaml
name: lab1-exercise

topology:
  nodes:
    r1:
      kind: nokia_srlinux
      type: ixrd2
    r2:
      kind: nokia_srlinux
      type: ixrd2
    r3:
      kind: nokia_srlinux
      type: ixrd2
  
  links:
    - endpoints: ["r1:e1-1", "r2:e1-1"]
    - endpoints: ["r2:e1-2", "r3:e1-1"]
```

### Tarea

1. Deploy el laboratorio
2. Configurar R1:
   - Loopback: 1.1.1.1/32
   - e1-1: 10.0.0.1/30
3. Configurar R2:
   - Loopback: 2.2.2.2/32
   - e1-1: 10.0.0.2/30
   - e1-2: 10.0.0.5/30
4. Configurar R3:
   - Loopback: 3.3.3.3/32
   - e1-1: 10.0.0.6/30
5. Verificar:
   - R1 puede hacer ping a R2
   - R2 puede hacer ping a R1 y R3
   - R3 puede hacer ping a R2
   - ¿R1 puede hacer ping a R3? ¿Por qué sí o no?

### Solución

<details>
<summary>Click para ver solución</summary>

```bash
# Deploy
sudo containerlab deploy -t lab1-exercise.yml

# R1 Config
sudo docker exec -it clab-lab1-exercise-r1 sr_cli
A:r1# enter candidate
A:r1# interface system0 admin-state enable subinterface 0 ipv4 address 1.1.1.1/32
A:r1# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 10.0.0.1/30
A:r1# commit now

# R2 Config
sudo docker exec -it clab-lab1-exercise-r2 sr_cli
A:r2# enter candidate
A:r2# interface system0 admin-state enable subinterface 0 ipv4 address 2.2.2.2/32
A:r2# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 10.0.0.2/30
A:r2# interface ethernet-1/2 admin-state enable subinterface 0 ipv4 address 10.0.0.5/30
A:r2# commit now

# R3 Config
sudo docker exec -it clab-lab1-exercise-r3 sr_cli
A:r3# enter candidate
A:r3# interface system0 admin-state enable subinterface 0 ipv4 address 3.3.3.3/32
A:r3# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 10.0.0.6/30
A:r3# commit now

# Verificación
# R1 → R2: ✅ (directamente conectados)
# R2 → R3: ✅ (directamente conectados)
# R1 → R3: ❌ (no hay ruta, necesitas routing protocol o rutas estáticas)
```

**Explicación:** R1 no puede alcanzar R3 porque no tiene una ruta. Necesitas:
- Opción 1: Configurar rutas estáticas
- Opción 2: Configurar protocolo de routing (OSPF/BGP) ← Veremos en nivel intermedio

</details>

---

## Resumen Nivel Básico

✅ **Has aprendido:**
- Qué es Containerlab y por qué es útil
- Instalar Containerlab en tu sistema
- Crear topologías simples en YAML
- Comandos esenciales (deploy, destroy, inspect)
- Acceder y navegar la CLI de SR Linux
- Configurar interfaces y direcciones IP
- Verificar conectividad básica

🎯 **Siguiente paso:** Nivel Intermedio - Topologías complejas y protocolos de routing

---

# Nivel Intermedio

## 6. Topologías Complejas

### Topología Hub-and-Spoke

Simula una red corporativa con sede central y sucursales.

```yaml
name: lab2-hub-spoke

topology:
  nodes:
    # Hub (Central)
    hub:
      kind: nokia_srlinux
      type: ixrd3
    
    # Spokes (Branches)
    spoke1:
      kind: nokia_srlinux
      type: ixrd2
    spoke2:
      kind: nokia_srlinux
      type: ixrd2
    spoke3:
      kind: nokia_srlinux
      type: ixrd2
    
    # Hosts
    server:
      kind: linux
      image: alpine:latest
    pc1:
      kind: linux
      image: alpine:latest
    pc2:
      kind: linux
      image: alpine:latest
  
  links:
    # Hub connections
    - endpoints: ["hub:e1-1", "spoke1:e1-1"]
    - endpoints: ["hub:e1-2", "spoke2:e1-1"]
    - endpoints: ["hub:e1-3", "spoke3:e1-1"]
    
    # Local connections
    - endpoints: ["hub:e1-10", "server:eth1"]
    - endpoints: ["spoke1:e1-10", "pc1:eth1"]
    - endpoints: ["spoke2:e1-10", "pc2:eth1"]
```

### Topología Full Mesh (para BGP)

Útil para practicar iBGP y route reflectors.

```yaml
name: lab2-full-mesh

topology:
  nodes:
    r1:
      kind: nokia_srlinux
      type: ixrd2
    r2:
      kind: nokia_srlinux
      type: ixrd2
    r3:
      kind: nokia_srlinux
      type: ixrd2
    r4:
      kind: nokia_srlinux
      type: ixrd2
  
  links:
    # Full mesh: cada router conectado con todos los demás
    - endpoints: ["r1:e1-1", "r2:e1-1"]
    - endpoints: ["r1:e1-2", "r3:e1-1"]
    - endpoints: ["r1:e1-3", "r4:e1-1"]
    - endpoints: ["r2:e1-2", "r3:e1-2"]
    - endpoints: ["r2:e1-3", "r4:e1-2"]
    - endpoints: ["r3:e1-3", "r4:e1-3"]
```

### Topología Data Center (Leaf-Spine)

Arquitectura moderna para centros de datos.

```yaml
name: lab2-datacenter

topology:
  nodes:
    # Spine layer
    spine1:
      kind: nokia_srlinux
      type: ixrd3
    spine2:
      kind: nokia_srlinux
      type: ixrd3
    
    # Leaf layer
    leaf1:
      kind: nokia_srlinux
      type: ixrd2
    leaf2:
      kind: nokia_srlinux
      type: ixrd2
    leaf3:
      kind: nokia_srlinux
      type: ixrd2
    
    # Servers
    srv1:
      kind: linux
      image: alpine:latest
    srv2:
      kind: linux
      image: alpine:latest
    srv3:
      kind: linux
      image: alpine:latest
  
  links:
    # Leaf1 to Spines
    - endpoints: ["leaf1:e1-1", "spine1:e1-1"]
    - endpoints: ["leaf1:e1-2", "spine2:e1-1"]
    
    # Leaf2 to Spines
    - endpoints: ["leaf2:e1-1", "spine1:e1-2"]
    - endpoints: ["leaf2:e1-2", "spine2:e1-2"]
    
    # Leaf3 to Spines
    - endpoints: ["leaf3:e1-1", "spine1:e1-3"]
    - endpoints: ["leaf3:e1-2", "spine2:e1-3"]
    
    # Servers to Leafs
    - endpoints: ["srv1:eth1", "leaf1:e1-10"]
    - endpoints: ["srv2:eth1", "leaf2:e1-10"]
    - endpoints: ["srv3:eth1", "leaf3:e1-10"]
```

---

## 7. Tipos de Nodos

### Nokia SR Linux

```yaml
nodes:
  router1:
    kind: nokia_srlinux
    type: ixrd3  # Grande: 8 CPU, 16GB RAM
    # type: ixrd2  # Mediano: 4 CPU, 8GB RAM
    # type: ixrd1  # Pequeño: 2 CPU, 4GB RAM
    image: ghcr.io/nokia/srlinux:latest
```

**Uso típico:**
- `ixrd3`: PE routers, core routers
- `ixrd2`: P routers, distribution
- `ixrd1`: CE routers, access layer

### Linux Containers (Hosts)

```yaml
nodes:
  client:
    kind: linux
    image: alpine:latest  # Ligero
    # image: ubuntu:22.04  # Más completo
    # image: nicolaka/netshoot  # Herramientas de networking
```

**Configurar IP en host Linux:**

```bash
# Acceder al contenedor
sudo docker exec -it clab-<lab>-client sh

# Configurar IP manualmente
ip addr add 192.168.1.100/24 dev eth1
ip route add default via 192.168.1.1

# Verificar
ping 192.168.1.1
```

### Arista cEOS (opcional)

```yaml
nodes:
  switch1:
    kind: arista_ceos
    image: ceos:latest
    # Requiere importar imagen desde Arista
```

### FRRouting (Open Source Router)

```yaml
nodes:
  router_frr:
    kind: linux
    image: frrouting/frr:latest
```

---

## 8. Startup Configs

### ¿Por qué usar Startup Configs?

- ⚡ Deploy rápido con configuración pre-cargada
- 🔄 Reproducibilidad (mismo estado cada vez)
- 📝 Documentación (config as code)
- 🧪 Testing automatizado

### Estructura de Directorios

```
my-lab/
├── topology.yml
├── configs/
│   ├── r1.cfg
│   ├── r2.cfg
│   └── r3.cfg
└── README.md
```

### Crear Startup Config

**Método 1: Desde CLI configurada**

```bash
# 1. Configura el router manualmente
sudo docker exec -it clab-lab-r1 sr_cli
# ... configuración ...

# 2. Exporta la config
sudo docker exec clab-lab-r1 cat /etc/opt/srlinux/config.json > configs/r1.cfg
```

**Método 2: Crear manualmente**

**Archivo `configs/r1.cfg`:**

```bash
# Nokia SR Linux startup config for R1

set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 description "Link to R2"
set / interface ethernet-1/1 subinterface 0 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.1/30

set / interface system0 admin-state enable
set / interface system0 subinterface 0 admin-state enable
set / interface system0 subinterface 0 ipv4 admin-state enable
set / interface system0 subinterface 0 ipv4 address 1.1.1.1/32

set / network-instance default admin-state enable
set / network-instance default type default
set / network-instance default interface ethernet-1/1.0
set / network-instance default interface system0.0
```

### Usar Startup Config en Topología

```yaml
name: lab2-with-configs

topology:
  nodes:
    r1:
      kind: nokia_srlinux
      type: ixrd2
      startup-config: configs/r1.cfg
    
    r2:
      kind: nokia_srlinux
      type: ixrd2
      startup-config: configs/r2.cfg
  
  links:
    - endpoints: ["r1:e1-1", "r2:e1-1"]
```

```bash
# Deploy con configs pre-cargadas
sudo containerlab deploy -t lab2-with-configs.yml

# Los routers arrancan ya configurados
sudo docker exec -it clab-lab2-with-configs-r1 sr_cli
A:r1# show interface brief
# Verás las IPs ya configuradas
```

---

## 9. Routing Protocols

### OSPF (Open Shortest Path First)

#### Topología para OSPF

```yaml
name: lab2-ospf

topology:
  nodes:
    r1:
      kind: nokia_srlinux
      type: ixrd2
    r2:
      kind: nokia_srlinux
      type: ixrd2
    r3:
      kind: nokia_srlinux
      type: ixrd2
    r4:
      kind: nokia_srlinux
      type: ixrd2
  
  links:
    # Area 0 (backbone)
    - endpoints: ["r1:e1-1", "r2:e1-1"]
    - endpoints: ["r2:e1-2", "r3:e1-1"]
    
    # Area 1 (stub)
    - endpoints: ["r3:e1-2", "r4:e1-1"]
```

#### Configurar OSPF en R1

```bash
sudo docker exec -it clab-lab2-ospf-r1 sr_cli

A:r1# enter candidate

# Configurar interfaces primero (ya sabes cómo)
# Loopback: 1.1.1.1/32
# e1-1: 10.0.0.1/30

# Configurar OSPF
A:r1# network-instance default
A:r1# protocols ospf
A:r1# instance main
A:r1# admin-state enable
A:r1# version ospf-v2
A:r1# router-id 1.1.1.1

# Area 0 (backbone)
A:r1# area 0.0.0.0

# Añadir interfaces al area
A:r1# interface ethernet-1/1.0
A:r1# interface-type point-to-point

# Añadir loopback (pasivo)
A:r1# interface system0.0
A:r1# passive true

A:r1# commit now
```

#### Configurar OSPF en R2 (ABR - Area Border Router)

```bash
A:r2# enter candidate

# Interfaces:
# Loopback: 2.2.2.2/32
# e1-1: 10.0.0.2/30 (hacia R1, Area 0)
# e1-2: 10.0.0.5/30 (hacia R3, Area 0)

# OSPF
A:r2# network-instance default protocols ospf instance main
A:r2# admin-state enable
A:r2# version ospf-v2
A:r2# router-id 2.2.2.2

# Area 0
A:r2# area 0.0.0.0
A:r2# interface ethernet-1/1.0 interface-type point-to-point
A:r2# interface ethernet-1/2.0 interface-type point-to-point
A:r2# interface system0.0 passive true

A:r2# commit now
```

#### Verificar OSPF

```bash
# Ver neighbors
A:r1# show network-instance default protocols ospf neighbor

# Output esperado:
-------------------------------------------------------------------------------
OSPF Neighbor Information for network-instance "default"
-------------------------------------------------------------------------------
Interface          : ethernet-1/1.0
Neighbor Router ID : 2.2.2.2
Neighbor IP Address: 10.0.0.2
Neighbor State     : full
Neighbor Priority  : 1
DR IP Address      : 0.0.0.0
BDR IP Address     : 0.0.0.0
Area ID            : 0.0.0.0
Dead Timer         : 00:00:34
Hello Timer        : 00:00:08
Retransmit Queue   : 0
Request Queue      : 0
-------------------------------------------------------------------------------
Total Neighbors: 1 (1 Full)
-------------------------------------------------------------------------------

# Ver rutas OSPF en tabla de routing
A:r1# show network-instance default route-table

# Output esperado:
--------------------------------------------------------------------
IPv4 unicast route table of network instance default
--------------------------------------------------------------------
+---------------+-------+------------+------------+--------+---------+
|    Prefix     | ID    | Route Type | Route Owner| Active | Metric  |
+===============+=======+============+============+========+=========+
| 1.1.1.1/32    | 0     | local      | local_route| True   | 0       |
| 2.2.2.2/32    | 0     | ospf       | ospf_mgr   | True   | 10      |
| 3.3.3.3/32    | 0     | ospf       | ospf_mgr   | True   | 20      |
| 10.0.0.0/30   | 0     | local      | local_route| True   | 0       |
| 10.0.0.4/30   | 0     | ospf       | ospf_mgr   | True   | 20      |
+---------------+-------+------------+------------+--------+---------+
--------------------------------------------------------------------
IPv4 routes total                    : 5
IPv4 prefixes with active routes     : 5
IPv4 prefixes with active ECMP routes: 0
--------------------------------------------------------------------

# Ver database OSPF
A:r1# show network-instance default protocols ospf database

# Output esperado:
-------------------------------------------------------------------------------
OSPF Link State Database for network-instance "default"
-------------------------------------------------------------------------------
Type       Link State ID   Advertising Router  Age  Sequence       Checksum
-------------------------------------------------------------------------------
Router     1.1.1.1         1.1.1.1             245  0x80000003     0x1a2b
Router     2.2.2.2         2.2.2.2             198  0x80000002     0x2b3c
Router     3.3.3.3         3.3.3.3             156  0x80000001     0x3c4d
Network    10.0.0.1        1.1.1.1             245  0x80000001     0x4d5e
-------------------------------------------------------------------------------
Total LSAs: 4
-------------------------------------------------------------------------------

# Ver interfaz OSPF específica
A:r1# show network-instance default protocols ospf interface ethernet-1/1.0

# Output esperado:
-------------------------------------------------------------------------------
OSPF Interface Information for network-instance "default"
-------------------------------------------------------------------------------
Interface          : ethernet-1/1.0
State              : point-to-point
Area               : 0.0.0.0
IP Address         : 10.0.0.1/30
Network Type       : point-to-point
Cost               : 10
Priority           : 1
Hello Interval     : 10 seconds
Dead Interval      : 40 seconds
Retransmit Interval: 5 seconds
Transit Delay      : 1 second
Neighbors          : 1
Passive            : False
-------------------------------------------------------------------------------
```

#### Troubleshooting OSPF

```bash
# Verificar que OSPF esté enabled
A:r1# info network-instance default protocols ospf instance main admin-state

# Output esperado:
    network-instance default {
        protocols {
            ospf {
                instance main {
                    admin-state enable
                }
            }
        }
    }

# Verificar router-id
A:r1# info network-instance default protocols ospf instance main router-id

# Output esperado:
    network-instance default {
        protocols {
            ospf {
                instance main {
                    router-id 1.1.1.1
                }
            }
        }
    }

# Ver timers de OSPF
A:r1# show network-instance default protocols ospf interface ethernet-1/1.0 detail

# Output esperado (detallado):
===============================================================================
OSPF Interface Detailed Information
===============================================================================
Interface              : ethernet-1/1.0
Interface Index        : 1
State                  : point-to-point
Area                   : 0.0.0.0 (Backbone)
IP Address/Mask        : 10.0.0.1/30
DR IP Address          : 0.0.0.0
BDR IP Address         : 0.0.0.0
Network Type           : point-to-point
MTU                    : 1500
Cost                   : 10
Priority               : 1
Hello Interval         : 10 seconds
Dead Interval          : 40 seconds
Retransmit Interval    : 5 seconds
Transit Delay          : 1 second
Authentication Type    : none
Passive                : False
Demand Circuit         : False
Number of Neighbors    : 1
Number of Adjacencies  : 1
===============================================================================

# Si no hay neighbors, verificar:
# 1. Interfaces administratively up
A:r1# show interface ethernet-1/1

# Output esperado:
===============================================================================
ethernet-1/1 is up, speed 100G, type None
  ethernet-1/1.0 is up
    ...
===============================================================================

# 2. Ping directo funciona
A:r1# ping 10.0.0.2 network-instance default

# Output esperado (si funciona):
Using network instance default
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.123 ms
...
--- 10.0.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss

# 3. OSPF configurado en ambos lados
# (verificar en el otro router)
```

---

### BGP (Border Gateway Protocol)

#### Topología para iBGP

```yaml
name: lab2-bgp

topology:
  nodes:
    rr1:  # Route Reflector 1
      kind: nokia_srlinux
      type: ixrd3
    rr2:  # Route Reflector 2
      kind: nokia_srlinux
      type: ixrd3
    pe1:  # Provider Edge 1
      kind: nokia_srlinux
      type: ixrd2
    pe2:
      kind: nokia_srlinux
      type: ixrd2
  
  links:
    # Full mesh underlay (OSPF)
    - endpoints: ["rr1:e1-1", "rr2:e1-1"]
    - endpoints: ["rr1:e1-2", "pe1:e1-1"]
    - endpoints: ["rr1:e1-3", "pe2:e1-1"]
    - endpoints: ["rr2:e1-2", "pe1:e1-2"]
    - endpoints: ["rr2:e1-3", "pe2:e1-2"]
```

#### Configurar BGP en RR1 (Route Reflector)

```bash
A:rr1# enter candidate

# Prerequisito: OSPF ya configurado (loopbacks accesibles)

# BGP básico
A:rr1# network-instance default
A:rr1# protocols bgp
A:rr1# admin-state enable
A:rr1# autonomous-system 65000
A:rr1# router-id 1.1.1.1

# Grupo de peers (iBGP)
A:rr1# group ibgp
A:rr1# peer-as 65000
A:rr1# local-as 65000

# Route reflector client config
A:rr1# route-reflector client true

# Neighbors (PE routers)
A:rr1# neighbor 3.3.3.3  # PE1
A:rr1# peer-group ibgp
A:rr1# transport local-address 1.1.1.1

A:rr1# neighbor 4.4.4.4  # PE2
A:rr1# peer-group ibgp
A:rr1# transport local-address 1.1.1.1

# Otro RR (no-client)
A:rr1# neighbor 2.2.2.2  # RR2
A:rr1# peer-group ibgp
A:rr1# transport local-address 1.1.1.1
# No marcar como client a otro RR

A:rr1# commit now
```

#### Configurar BGP en PE1

```bash
A:pe1# enter candidate

# BGP
A:pe1# network-instance default protocols bgp
A:pe1# admin-state enable
A:pe1# autonomous-system 65000
A:pe1# router-id 3.3.3.3

# Grupo para Route Reflectors
A:pe1# group rr
A:pe1# peer-as 65000

# Neighbors (ambos RRs para redundancia)
A:pe1# neighbor 1.1.1.1  # RR1
A:pe1# peer-group rr
A:pe1# transport local-address 3.3.3.3

A:pe1# neighbor 2.2.2.2  # RR2
A:pe1# peer-group rr
A:pe1# transport local-address 3.3.3.3

A:pe1# commit now
```

#### Verificar BGP

```bash
# Ver sesiones BGP
A:rr1# show network-instance default protocols bgp neighbor

# Output esperado:
-------------------------------------------------------------------------------
BGP neighbor information for network-instance "default"
-------------------------------------------------------------------------------
Neighbor          : 3.3.3.3
Peer group        : ibgp
Peer AS           : 65000
State             : established
Uptime            : 0d:00:15:23
Hold time         : 180
Keep alive        : 60
Local Address     : 1.1.1.1
Local Port        : 179
Remote Address    : 3.3.3.3
Remote Port       : 45678
Route Reflector   : client
Import Policy     : None
Export Policy     : None
Received Routes   : 10
Accepted Routes   : 10
Sent Routes       : 15
Active Routes     : 10
-------------------------------------------------------------------------------
Neighbor          : 4.4.4.4
Peer group        : ibgp
Peer AS           : 65000
State             : established
Uptime            : 0d:00:14:45
Hold time         : 180
Keep alive        : 60
Local Address     : 1.1.1.1
Local Port        : 179
Remote Address    : 4.4.4.4
Remote Port       : 45679
Route Reflector   : client
Import Policy     : None
Export Policy     : None
Received Routes   : 8
Accepted Routes   : 8
Sent Routes       : 15
Active Routes     : 8
-------------------------------------------------------------------------------
Total Neighbors: 2, Established: 2
-------------------------------------------------------------------------------

# Ver summary
A:rr1# show network-instance default protocols bgp summary

# Output esperado:
-------------------------------------------------------------------------------
BGP Summary for network-instance "default"
-------------------------------------------------------------------------------
AS Number         : 65000
Router ID         : 1.1.1.1
Local AS          : 65000
-------------------------------------------------------------------------------
Neighbor          Peer AS   State        Uptime      #Received #Accepted
-------------------------------------------------------------------------------
3.3.3.3           65000     established  0d:00:15:23 10        10
4.4.4.4           65000     established  0d:00:14:45 8         8
2.2.2.2           65000     established  0d:00:16:12 10        10
-------------------------------------------------------------------------------
Total Neighbors: 3
Established    : 3
Active         : 0
Idle           : 0
Connect        : 0
OpenSent       : 0
OpenConfirm    : 0
-------------------------------------------------------------------------------

# Ver rutas BGP
A:rr1# show network-instance default protocols bgp routes

# Output esperado:
-------------------------------------------------------------------------------
BGP Routes for network-instance "default"
-------------------------------------------------------------------------------
Status codes: u - used, * - valid, > - best, i - internal, e - external
Origin codes: i - IGP, e - EGP, ? - incomplete
-------------------------------------------------------------------------------
Type|Status| Network          |Next Hop      |Metric|LocPrf|AsPath|Origin
-------------------------------------------------------------------------------
ipv4|u*>i  | 192.168.1.0/24   |3.3.3.3       |0     |100   |      |i
ipv4|u*>i  | 192.168.2.0/24   |4.4.4.4       |0     |100   |      |i
ipv4|u*>i  | 10.0.0.0/30      |0.0.0.0       |0     |100   |      |i
ipv4|u*>i  | 10.0.0.4/30      |2.2.2.2       |0     |100   |      |i
-------------------------------------------------------------------------------
Total routes: 4
Valid routes: 4
Best routes : 4
-------------------------------------------------------------------------------

# Ver detalles de un neighbor específico
A:rr1# show network-instance default protocols bgp neighbor 3.3.3.3 detail

# Output esperado (detallado):
===============================================================================
BGP Neighbor Detailed Information
===============================================================================
Neighbor                 : 3.3.3.3
Description              : None
Peer Group               : ibgp
Peer Type                : internal
Peer AS                  : 65000
Local AS                 : 65000
State                    : established
State Duration           : 0d:00:15:23
Last State Change        : 2024-01-15T11:30:00.123Z
Last Error Code          : None
Last Error Subcode       : None
Last Notification        : None

Connection Information:
  Local Address          : 1.1.1.1
  Local Port             : 179
  Remote Address         : 3.3.3.3
  Remote Port            : 45678
  TCP MD5 Authentication : disabled
  
Timer Information:
  Hold Time              : 180 seconds
  Keep Alive             : 60 seconds
  Connect Retry          : 120 seconds
  Min AS Origination     : 15 seconds
  Min Route Advertisement: 30 seconds

Capability Information:
  4-byte AS              : advertised and received
  Route Refresh          : advertised and received
  Graceful Restart       : not advertised
  Add-Path               : not advertised
  Extended Next Hop      : not advertised

Route Statistics:
  Received Routes        : 10
  Accepted Routes        : 10
  Rejected Routes        : 0
  Sent Routes            : 15
  Active Routes          : 10
  Advertised Routes      : 15

Route Reflector:
  Client                 : True
  Cluster ID             : 1.1.1.1

Policy:
  Import Policy          : None
  Export Policy          : None
  
Address Families:
  IPv4 Unicast           : negotiated
  VPN-IPv4               : negotiated
===============================================================================
```

---

## 10. Troubleshooting Básico

### Metodología General

1. **Physical Layer** (¿Interfaces up?)
2. **Data Link** (¿MAC addresses correctas?)
3. **Network Layer** (¿IPs correctas? ¿Routing funciona?)
4. **Transport/Application** (¿Servicios corriendo?)

### Checklist Paso a Paso

#### 1. Verificar Interfaces

```bash
# Ver todas las interfaces
A:r1# show interface brief

# ¿Están "up"?
# admin-state: enable
# oper-state: up

# Si están down:
A:r1# show interface ethernet-1/1 detail
# Revisar: admin-state, oper-down-reason

# Habilitar si está administratively down
A:r1# enter candidate
A:r1# interface ethernet-1/1 admin-state enable
A:r1# commit now
```

#### 2. Verificar IPs

```bash
# Ver IPs configuradas
A:r1# info interface

# Verificar subnet correcta
A:r1# show interface ethernet-1/1 subinterface 0

# Ping a gateway directo
A:r1# ping 10.0.0.2 network-instance default
```

#### 3. Verificar Routing

```bash
# Ver tabla de routing completa
A:r1# show network-instance default route-table

# Buscar ruta específica
A:r1# show network-instance default route-table ipv4-unicast route 3.3.3.3

# Si no hay ruta:
# - Verificar protocolo de routing (OSPF/BGP)
# - Verificar rutas estáticas
# - Revisar network statements
```

#### 4. Verificar Protocolos de Routing

**OSPF:**
```bash
# ¿Hay neighbors?
A:r1# show network-instance default protocols ospf neighbor

# Si no:
# 1. Verificar que OSPF esté enabled
A:r1# info network-instance default protocols ospf instance main admin-state

# 2. Verificar router-id
A:r1# info network-instance default protocols ospf instance main router-id

# 3. Verificar que interfaces estén en OSPF
A:r1# show network-instance default protocols ospf interface
```

**BGP:**
```bash
# ¿Sesiones established?
A:r1# show network-instance default protocols bgp summary

# Si no:
# 1. Verificar conectividad IP al neighbor
A:r1# ping <neighbor-ip> network-instance default

# 2. Verificar AS numbers coinciden (iBGP) o diferentes (eBGP)
A:r1# info network-instance default protocols bgp

# 3. Ver logs de BGP
A:r1# show network-instance default protocols bgp neighbor <ip> detail
```

#### 5. Captura de Paquetes

```bash
# Tcpdump en una interfaz
sudo docker exec clab-lab-r1 tcpdump -i eth1 -n

# Output esperado:
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
12:34:56.123456 IP 10.0.0.1 > 10.0.0.2: ICMP echo request, id 1234, seq 1, length 64
12:34:56.123789 IP 10.0.0.2 > 10.0.0.1: ICMP echo reply, id 1234, seq 1, length 64
12:34:57.456123 IP 10.0.0.1.179 > 10.0.0.2.45678: Flags [P.], seq 1:89, ack 1, length 88: BGP
12:34:57.456456 IP 10.0.0.2.45678 > 10.0.0.1.179: Flags [.], ack 89, length 0
12:34:58.789012 IP 10.0.0.1 > 224.0.0.5: OSPFv2, Hello, length 48
^C
5 packets captured
5 packets received by filter
0 packets dropped by kernel

# Guardar a archivo
sudo docker exec clab-lab-r1 tcpdump -i eth1 -w /tmp/capture.pcap

# Output:
tcpdump: listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
^C
125 packets captured
125 packets received by filter
0 packets dropped by kernel

# Copiar al host
sudo docker cp clab-lab-r1:/tmp/capture.pcap ./

# Output:
Successfully copied 23.5kB to /home/user/capture.pcap

# Abrir con Wireshark
wireshark capture.pcap

# Wireshark mostrará los paquetes gráficamente donde puedes ver:
# - Paquetes OSPF Hello
# - BGP Keepalives y Updates
# - ICMP pings
# - LDP Label Advertisements
# - etc.
```

### Problemas Comunes y Soluciones

#### Problema 1: "No route to host"

```bash
# Síntoma: ping falla con "no route"
A:r1# ping 3.3.3.3 network-instance default

# Output con error:
Using network instance default
PING 3.3.3.3 (3.3.3.3) 56(84) bytes of data.
ping: sendmsg: Network is unreachable
ping: sendmsg: Network is unreachable
--- 3.3.3.3 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss

# Diagnóstico:
A:r1# show network-instance default route-table ipv4-unicast route 3.3.3.3

# Output cuando falta ruta:
--------------------------------------------------------------------
No matching routes found for prefix 3.3.3.3
--------------------------------------------------------------------

# Solución:
# Opción A: Añadir ruta estática
A:r1# enter candidate
A:r1# network-instance default static-routes route 3.3.3.3/32 next-hop 10.0.0.2
A:r1# commit now

# Output después:
All changes have been committed.

# Verificar:
A:r1# show network-instance default route-table ipv4-unicast route 3.3.3.3

# Output con ruta:
--------------------------------------------------------------------
IPv4 route information for network instance default
--------------------------------------------------------------------
Prefix        : 3.3.3.3/32
Route Type    : static
Route Owner   : static_mgr
Active        : True
Metric        : 1
Next-hop      : 10.0.0.2 via ethernet-1/1.0
--------------------------------------------------------------------

# Opción B: Configurar protocolo de routing (OSPF/BGP)
# (ver secciones anteriores)
```

#### Problema 2: "OSPF neighbors no forman"

```bash
# Diagnóstico:
A:r1# show network-instance default protocols ospf neighbor

# Output cuando NO hay neighbors:
-------------------------------------------------------------------------------
OSPF Neighbor Information for network-instance "default"
-------------------------------------------------------------------------------
No neighbors found
-------------------------------------------------------------------------------

# Checklist:
# 1. ¿Interfaces up?
A:r1# show interface brief | grep ethernet-1/1

# Output esperado (debe estar "up"):
ethernet-1/1  | enable        | up            | 100G          | None

# Si está "down":
ethernet-1/1  | disable       | down          | 100G          | None
# Solución: A:r1# enter candidate; interface ethernet-1/1 admin-state enable; commit now

# 2. ¿Ping directo funciona?
A:r1# ping 10.0.0.2 network-instance default

# Output si funciona:
Using network instance default
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.123 ms
--- 10.0.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss

# Output si NO funciona:
From 10.0.0.1 icmp_seq=1 Destination Host Unreachable
# Problema: Conectividad Layer 2/3

# 3. ¿Interfaces en mismo subnet?
A:r1# show interface ethernet-1/1 subinterface 0

# Output:
ethernet-1/1.0:
  IPv4 addr: 10.0.0.1/30
# Verificar que el neighbor esté en 10.0.0.2 (mismo /30)

# 4. ¿OSPF configurado en ambos lados?
# En R2:
A:r2# show network-instance default protocols ospf interface

# Output si está configurado:
ethernet-1/1.0 | 0.0.0.0 | point-to-point | 10

# Output si NO está configurado:
No OSPF interfaces found
# Solución: Configurar OSPF en R2

# 5. ¿Mismo area?
A:r1# info network-instance default protocols ospf instance main area

# Output:
area 0.0.0.0 {
    interface ethernet-1/1.0 {
        interface-type point-to-point
    }
}

# Verificar que R2 también esté en area 0.0.0.0
```

#### Problema 3: "BGP stuck en Active/Connect"

```bash
# Diagnóstico:
A:r1# show network-instance default protocols bgp neighbor

# Output con problema (State NO es "established"):
-------------------------------------------------------------------------------
Neighbor          : 3.3.3.3
Peer AS           : 65000
State             : active       ← PROBLEMA: debería ser "established"
Uptime            : N/A
Connect Retry     : 00:01:23
Last Error        : Connection refused
-------------------------------------------------------------------------------

# Otro estado problemático:
State             : connect      ← PROBLEMA: intentando conectar
State             : idle         ← PROBLEMA: no está intentando
State             : opensent     ← PROBLEMA: esperando respuesta
State             : openconfirm  ← PROBLEMA: casi listo pero no completo

# Estado correcto:
State             : established  ← ✅ CORRECTO

# Soluciones:
# 1. Verificar conectividad Layer 3
A:r1# ping 3.3.3.3 network-instance default

# Output si funciona:
PING 3.3.3.3 (3.3.3.3) 56(84) bytes of data.
64 bytes from 3.3.3.3: icmp_seq=1 ttl=64 time=0.234 ms
--- 3.3.3.3 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss

# Output si NO funciona:
From 1.1.1.1 icmp_seq=1 Destination Host Unreachable
# Solución: Verificar OSPF/IGP primero

# 2. Verificar AS numbers
A:r1# info network-instance default protocols bgp autonomous-system

# Output:
autonomous-system 65000

A:r1# info network-instance default protocols bgp neighbor 3.3.3.3 peer-as

# Output:
neighbor 3.3.3.3 {
    peer-as 65000
}

# Para iBGP: autonomous-system y peer-as deben ser IGUALES
# Para eBGP: autonomous-system y peer-as deben ser DIFERENTES

# 3. Verificar local/remote IPs
A:r1# info network-instance default protocols bgp neighbor 3.3.3.3

# Output:
neighbor 3.3.3.3 {
    peer-as 65000
    transport {
        local-address 1.1.1.1
    }
}

# Verificar que 1.1.1.1 sea alcanzable desde 3.3.3.3

# 4. Ver logs específicos
A:r1# show network-instance default protocols bgp neighbor 3.3.3.3 detail | grep -A 20 "last-error"

# Output con información del problema:
Last Error Code          : Cease
Last Error Subcode       : Connection Rejected
Last Notification Sent   : 2024-01-15T12:30:45.123Z
Last Notification Error  : Hold Timer Expired

# Soluciones comunes según error:
# - "Connection Refused": Neighbor no tiene BGP configurado o puerto bloqueado
# - "Hold Timer Expired": Paquetes BGP no llegan (firewall? routing?)
# - "Bad Peer AS": peer-as configurado incorrectamente
# - "Authentication Failure": Password MD5 no coincide
```

---

## Ejercicio Práctico 2: OSPF Multi-Area

**Objetivo:** Configurar OSPF con múltiples áreas y verificar conectividad completa.

### Topología

```
        Area 0 (Backbone)
    R1 -------- R2 -------- R3
                |
                | Area 1
                |
                R4
```

### Archivo `lab2-ospf-multiarea.yml`

```yaml
name: lab2-ospf-multiarea

topology:
  nodes:
    r1:
      kind: nokia_srlinux
      type: ixrd2
    r2:
      kind: nokia_srlinux
      type: ixrd2  # ABR (Area Border Router)
    r3:
      kind: nokia_srlinux
      type: ixrd2
    r4:
      kind: nokia_srlinux
      type: ixrd2
  
  links:
    - endpoints: ["r1:e1-1", "r2:e1-1"]  # Area 0
    - endpoints: ["r2:e1-2", "r3:e1-1"]  # Area 0
    - endpoints: ["r2:e1-3", "r4:e1-1"]  # Area 1
```

### Tareas

1. Deploy el laboratorio
2. Configurar IPs:
   - R1: loopback 1.1.1.1/32, e1-1: 10.0.0.1/30
   - R2: loopback 2.2.2.2/32, e1-1: 10.0.0.2/30, e1-2: 10.0.0.5/30, e1-3: 10.0.1.1/30
   - R3: loopback 3.3.3.3/32, e1-1: 10.0.0.6/30
   - R4: loopback 4.4.4.4/32, e1-1: 10.0.1.2/30
3. Configurar OSPF:
   - R1, R2, R3 en Area 0
   - R2-R4 link en Area 1 (R2 es ABR)
4. Verificar:
   - Todos los routers tienen neighbors correctos
   - R1 puede alcanzar 4.4.4.4
   - Ver en R2 que es ABR (tiene rutas inter-area)

### Solución

<details>
<summary>Click para ver solución completa</summary>

```bash
# Deploy
sudo containerlab deploy -t lab2-ospf-multiarea.yml

# === R1 Config ===
sudo docker exec -it clab-lab2-ospf-multiarea-r1 sr_cli
A:r1# enter candidate

# Interfaces
A:r1# interface system0 admin-state enable subinterface 0 ipv4 address 1.1.1.1/32
A:r1# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 10.0.0.1/30
A:r1# network-instance default interface system0.0
A:r1# network-instance default interface ethernet-1/1.0

# OSPF Area 0
A:r1# network-instance default protocols ospf instance main admin-state enable
A:r1# network-instance default protocols ospf instance main version ospf-v2
A:r1# network-instance default protocols ospf instance main router-id 1.1.1.1
A:r1# network-instance default protocols ospf instance main area 0.0.0.0 interface ethernet-1/1.0 interface-type point-to-point
A:r1# network-instance default protocols ospf instance main area 0.0.0.0 interface system0.0 passive true

A:r1# commit now

# === R2 Config (ABR) ===
sudo docker exec -it clab-lab2-ospf-multiarea-r2 sr_cli
A:r2# enter candidate

# Interfaces
A:r2# interface system0 admin-state enable subinterface 0 ipv4 address 2.2.2.2/32
A:r2# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 10.0.0.2/30
A:r2# interface ethernet-1/2 admin-state enable subinterface 0 ipv4 address 10.0.0.5/30
A:r2# interface ethernet-1/3 admin-state enable subinterface 0 ipv4 address 10.0.1.1/30

A:r2# network-instance default interface system0.0
A:r2# network-instance default interface ethernet-1/1.0
A:r2# network-instance default interface ethernet-1/2.0
A:r2# network-instance default interface ethernet-1/3.0

# OSPF Area 0
A:r2# network-instance default protocols ospf instance main admin-state enable
A:r2# network-instance default protocols ospf instance main version ospf-v2
A:r2# network-instance default protocols ospf instance main router-id 2.2.2.2
A:r2# network-instance default protocols ospf instance main area 0.0.0.0 interface ethernet-1/1.0 interface-type point-to-point
A:r2# network-instance default protocols ospf instance main area 0.0.0.0 interface ethernet-1/2.0 interface-type point-to-point
A:r2# network-instance default protocols ospf instance main area 0.0.0.0 interface system0.0 passive true

# OSPF Area 1
A:r2# network-instance default protocols ospf instance main area 0.0.0.1 interface ethernet-1/3.0 interface-type point-to-point

A:r2# commit now

# === R3 Config ===
sudo docker exec -it clab-lab2-ospf-multiarea-r3 sr_cli
A:r3# enter candidate

# Interfaces
A:r3# interface system0 admin-state enable subinterface 0 ipv4 address 3.3.3.3/32
A:r3# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 10.0.0.6/30
A:r3# network-instance default interface system0.0
A:r3# network-instance default interface ethernet-1/1.0

# OSPF Area 0
A:r3# network-instance default protocols ospf instance main admin-state enable
A:r3# network-instance default protocols ospf instance main version ospf-v2
A:r3# network-instance default protocols ospf instance main router-id 3.3.3.3
A:r3# network-instance default protocols ospf instance main area 0.0.0.0 interface ethernet-1/1.0 interface-type point-to-point
A:r3# network-instance default protocols ospf instance main area 0.0.0.0 interface system0.0 passive true

A:r3# commit now

# === R4 Config ===
sudo docker exec -it clab-lab2-ospf-multiarea-r4 sr_cli
A:r4# enter candidate

# Interfaces
A:r4# interface system0 admin-state enable subinterface 0 ipv4 address 4.4.4.4/32
A:r4# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 10.0.1.2/30
A:r4# network-instance default interface system0.0
A:r4# network-instance default interface ethernet-1/1.0

# OSPF Area 1
A:r4# network-instance default protocols ospf instance main admin-state enable
A:r4# network-instance default protocols ospf instance main version ospf-v2
A:r4# network-instance default protocols ospf instance main router-id 4.4.4.4
A:r4# network-instance default protocols ospf instance main area 0.0.0.1 interface ethernet-1/1.0 interface-type point-to-point
A:r4# network-instance default protocols ospf instance main area 0.0.0.1 interface system0.0 passive true

A:r4# commit now

# === Verificación ===

# En R1:
A:r1# show network-instance default protocols ospf neighbor
# Debe ver neighbor 2.2.2.2 (R2)

A:r1# show network-instance default route-table
# Debe ver rutas a 2.2.2.2, 3.3.3.3, 4.4.4.4

A:r1# ping 4.4.4.4 network-instance default
# Debe funcionar

# En R2 (ABR):
A:r2# show network-instance default protocols ospf neighbor
# Debe ver 3 neighbors: R1, R3, R4

A:r2# show network-instance default protocols ospf database
# Verás LSAs de ambas áreas

# En R4:
A:r4# show network-instance default route-table ipv4-unicast route 1.1.1.1
# Verás ruta inter-area
```

</details>

---

## Resumen Nivel Intermedio

✅ **Has aprendido:**
- Crear topologías complejas (hub-spoke, full-mesh, leaf-spine)
- Diferentes tipos de nodos (SR Linux, Linux containers)
- Usar startup configs para deploy rápido
- Configurar OSPF single-area y multi-area
- Configurar BGP con route reflectors
- Troubleshooting metódico de problemas de red

🎯 **Siguiente paso:** Nivel Avanzado - MPLS, telemetría, integración con IA

---

# Nivel Avanzado

## 11. MPLS L3VPN

### ¿Qué es MPLS L3VPN?

MPLS L3VPN permite a un Service Provider ofrecer conectividad IP entre sitios de un mismo cliente a través de una red MPLS compartida, manteniendo aislamiento entre clientes.

**Componentes clave:**
- **PE (Provider Edge):** Routers que conectan con clientes
- **P (Provider):** Routers core del SP (solo MPLS forwarding)
- **CE (Customer Edge):** Routers del cliente
- **VRF (Virtual Routing and Forwarding):** Tabla de routing aislada por cliente
- **RD (Route Distinguisher):** Hace únicas las rutas de cada VRF
- **RT (Route Target):** Controla import/export de rutas entre VRFs

### Topología MPLS L3VPN

```yaml
name: lab3-mpls-l3vpn

topology:
  nodes:
    # Provider Edge routers
    pe1:
      kind: nokia_srlinux
      type: ixrd3
    pe2:
      kind: nokia_srlinux
      type: ixrd3
    
    # Provider core routers
    p1:
      kind: nokia_srlinux
      type: ixrd2
    p2:
      kind: nokia_srlinux
      type: ixrd2
    
    # Customer A sites
    ce1-customerA:
      kind: nokia_srlinux
      type: ixrd1
    ce2-customerA:
      kind: nokia_srlinux
      type: ixrd1
    
    # Customer B sites
    ce1-customerB:
      kind: nokia_srlinux
      type: ixrd1
  
  links:
    # MPLS core (PE-P-PE)
    - endpoints: ["pe1:e1-1", "p1:e1-1"]
    - endpoints: ["p1:e1-2", "p2:e1-1"]
    - endpoints: ["p2:e1-2", "pe2:e1-1"]
    
    # Customer A connections
    - endpoints: ["ce1-customerA:e1-1", "pe1:e1-10"]
    - endpoints: ["ce2-customerA:e1-1", "pe2:e1-10"]
    
    # Customer B connections
    - endpoints: ["ce1-customerB:e1-1", "pe1:e1-11"]
```

### Paso 1: Configurar IGP (OSPF) en Core

**PE1:**
```bash
A:pe1# enter candidate

# Interfaces
A:pe1# interface system0 admin-state enable subinterface 0 ipv4 address 1.1.1.1/32
A:pe1# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 10.0.0.1/30
A:pe1# network-instance default interface system0.0
A:pe1# network-instance default interface ethernet-1/1.0

# OSPF
A:pe1# network-instance default protocols ospf instance core admin-state enable
A:pe1# network-instance default protocols ospf instance core version ospf-v2
A:pe1# network-instance default protocols ospf instance core router-id 1.1.1.1
A:pe1# network-instance default protocols ospf instance core area 0.0.0.0 interface ethernet-1/1.0 interface-type point-to-point
A:pe1# network-instance default protocols ospf instance core area 0.0.0.0 interface system0.0 passive true

A:pe1# commit now
```

Repite para P1, P2, PE2 con sus respectivas IPs y conexiones.

### Paso 2: Configurar LDP (Label Distribution Protocol)

**PE1:**
```bash
A:pe1# enter candidate

# LDP
A:pe1# network-instance default protocols ldp
A:pe1# admin-state enable
A:pe1# router-id 1.1.1.1

# Habilitar LDP en interfaces core
A:pe1# interface ethernet-1/1.0
A:pe1# admin-state enable

A:pe1# commit now
```

Repite en P1, P2, PE2.

**Verificar LDP:**
```bash
# Ver neighbors LDP
A:pe1# show network-instance default protocols ldp neighbor

# Output esperado:
-------------------------------------------------------------------------------
LDP Neighbor Information for network-instance "default"
-------------------------------------------------------------------------------
Neighbor ID       : 2.2.2.2:0
Peer Address      : 10.0.0.2
State             : operational
Uptime            : 0d:00:12:34
Hold Time         : 45 seconds
Keep Alive        : 15 seconds
Local Address     : 10.0.0.1
Transport Address : 1.1.1.1
Sessions          : 1
Messages Sent     : 156
Messages Received : 148
Adjacency Type    : link
-------------------------------------------------------------------------------
Total Neighbors: 1, Operational: 1
-------------------------------------------------------------------------------

# Ver label table
A:pe1# show network-instance default protocols mpls label-table

# Output esperado:
-------------------------------------------------------------------------------
MPLS Label Table for network-instance "default"
-------------------------------------------------------------------------------
Label    Owner         In Use  Prefix/LSP
-------------------------------------------------------------------------------
100001   ldp           yes     2.2.2.2/32
100002   ldp           yes     3.3.3.3/32
100003   ldp           yes     10.0.0.4/30
100004   bgp-vpn       yes     VPN:customerA:192.168.1.0/24
100005   bgp-vpn       yes     VPN:customerA:192.168.2.0/24
-------------------------------------------------------------------------------
Total Labels: 5
In Use      : 5
Free        : 100195
-------------------------------------------------------------------------------
```

### Paso 3: Configurar MP-BGP entre PEs

**PE1:**
```bash
A:pe1# enter candidate

# BGP
A:pe1# network-instance default protocols bgp
A:pe1# admin-state enable
A:pe1# autonomous-system 65000
A:pe1# router-id 1.1.1.1

# Address family VPNv4
A:pe1# afi-safi ipv4-unicast admin-state enable
A:pe1# afi-safi vpn-ipv4 admin-state enable

# Neighbor PE2
A:pe1# neighbor 2.2.2.2
A:pe1# peer-as 65000
A:pe1# transport local-address 1.1.1.1
A:pe1# afi-safi vpn-ipv4 admin-state enable

A:pe1# commit now
```

**PE2:**
```bash
A:pe2# enter candidate

A:pe2# network-instance default protocols bgp
A:pe2# admin-state enable
A:pe2# autonomous-system 65000
A:pe2# router-id 2.2.2.2
A:pe2# afi-safi ipv4-unicast admin-state enable
A:pe2# afi-safi vpn-ipv4 admin-state enable

# Neighbor PE1
A:pe2# neighbor 1.1.1.1
A:pe2# peer-as 65000
A:pe2# transport local-address 2.2.2.2
A:pe2# afi-safi vpn-ipv4 admin-state enable

A:pe2# commit now
```

**Verificar MP-BGP:**
```bash
A:pe1# show network-instance default protocols bgp neighbor

# Output esperado:
-------------------------------------------------------------------------------
BGP Neighbor Information for network-instance "default"
-------------------------------------------------------------------------------
Neighbor          : 2.2.2.2
Peer AS           : 65000
State             : established
Uptime            : 0d:00:18:45
Local Address     : 1.1.1.1
Remote Address    : 2.2.2.2
Address Families:
  ipv4-unicast    : negotiated
  vpn-ipv4        : negotiated
Route Statistics:
  Received Routes : 25 (ipv4: 10, vpn-ipv4: 15)
  Accepted Routes : 25
  Sent Routes     : 30 (ipv4: 15, vpn-ipv4: 15)
  Active Routes   : 25
-------------------------------------------------------------------------------
# IMPORTANTE: State debe ser "established"
# IMPORTANTE: vpn-ipv4 debe estar "negotiated"
```

### Paso 4: Crear VRF para Customer A

**PE1:**
```bash
A:pe1# enter candidate

# Crear network-instance tipo ip-vrf
A:pe1# network-instance customerA
A:pe1# type ip-vrf

# Route distinguisher (formato: AS:number o IP:number)
A:pe1# route-distinguisher 65000:100

# Route targets
A:pe1# protocols bgp-vpn
A:pe1# route-target export target:65000:100
A:pe1# route-target import target:65000:100

# Interface hacia CE1
A:pe1# interface ethernet-1/10.100

# Configurar interfaz PE-CE
A:pe1# interface ethernet-1/10
A:pe1# admin-state enable
A:pe1# description "Link to CE1-CustomerA"
A:pe1# subinterface 100
A:pe1# admin-state enable
A:pe1# ipv4 address 192.168.1.1/30

# Asociar subinterface a VRF
A:pe1# network-instance customerA interface ethernet-1/10.100

A:pe1# commit now
```

**PE2 (similar para Customer A site 2):**
```bash
A:pe2# enter candidate

# VRF Customer A
A:pe2# network-instance customerA type ip-vrf
A:pe2# network-instance customerA route-distinguisher 65000:100
A:pe2# network-instance customerA protocols bgp-vpn route-target export target:65000:100
A:pe2# network-instance customerA protocols bgp-vpn route-target import target:65000:100

# Interface hacia CE2
A:pe2# interface ethernet-1/10 admin-state enable subinterface 100 ipv4 address 192.168.2.1/30
A:pe2# network-instance customerA interface ethernet-1/10.100

A:pe2# commit now
```

### Paso 5: Configurar PE-CE Routing (OSPF o BGP)

**Opción A: PE-CE con OSPF**

**PE1:**
```bash
A:pe1# enter candidate

# OSPF en VRF
A:pe1# network-instance customerA protocols ospf instance pe-ce
A:pe1# admin-state enable
A:pe1# version ospf-v2
A:pe1# router-id 1.1.1.1
A:pe1# area 0.0.0.0 interface ethernet-1/10.100 interface-type point-to-point

A:pe1# commit now
```

**CE1-CustomerA:**
```bash
A:ce1# enter candidate

# Interfaces
A:ce1# interface system0 admin-state enable subinterface 0 ipv4 address 192.168.100.1/32
A:ce1# interface ethernet-1/1 admin-state enable subinterface 0 ipv4 address 192.168.1.2/30
A:ce1# network-instance default interface system0.0
A:ce1# network-instance default interface ethernet-1/1.0

# OSPF
A:ce1# network-instance default protocols ospf instance main admin-state enable
A:ce1# network-instance default protocols ospf instance main version ospf-v2
A:ce1# network-instance default protocols ospf instance main router-id 192.168.100.1
A:ce1# network-instance default protocols ospf instance main area 0.0.0.0 interface ethernet-1/1.0 interface-type point-to-point
A:ce1# network-instance default protocols ospf instance main area 0.0.0.0 interface system0.0 passive true

A:ce1# commit now
```

Repite configuración similar en PE2 y CE2.

### Paso 6: Verificar L3VPN

**En PE1:**
```bash
# Ver rutas en VRF
A:pe1# show network-instance customerA route-table

# Output esperado:
--------------------------------------------------------------------
IPv4 unicast route table of network instance customerA
--------------------------------------------------------------------
+------------------+-------+------------+-------------+--------+---------+
|    Prefix        | ID    | Route Type | Route Owner | Active | Metric  |
+==================+=======+============+=============+========+=========+
| 192.168.1.0/30   | 0     | local      | local_route | True   | 0       |
| 192.168.1.2/32   | 0     | host       | local_route | True   | 0       |
| 192.168.100.1/32 | 0     | ospf       | ospf_mgr    | True   | 10      |
| 192.168.2.0/30   | 0     | bgp-vpn    | bgp_mgr     | True   | 0       |
| 192.168.200.1/32 | 0     | bgp-vpn    | bgp_mgr     | True   | 10      |
+------------------+-------+------------+-------------+--------+---------+
--------------------------------------------------------------------
# IMPORTANTE: Debes ver rutas de ambos sitios del cliente A
# Rutas locales (192.168.1.x) y rutas remotas vía BGP (192.168.200.x)
--------------------------------------------------------------------

# Ver neighbors OSPF en VRF
A:pe1# show network-instance customerA protocols ospf neighbor

# Output esperado:
-------------------------------------------------------------------------------
OSPF Neighbor Information for network-instance "customerA"
-------------------------------------------------------------------------------
Interface          : ethernet-1/10.100
Neighbor Router ID : 192.168.100.1
Neighbor IP Address: 192.168.1.2
Neighbor State     : full
Neighbor Priority  : 1
Area ID            : 0.0.0.0
Dead Timer         : 00:00:36
Hello Timer        : 00:00:08
-------------------------------------------------------------------------------

# Ver VPN routes en BGP
A:pe1# show network-instance default protocols bgp routes vpn-ipv4

# Output esperado:
-------------------------------------------------------------------------------
BGP VPN-IPv4 Routes for network-instance "default"
-------------------------------------------------------------------------------
Status codes: u - used, * - valid, > - best, i - internal
Origin codes: i - IGP, e - EGP, ? - incomplete
-------------------------------------------------------------------------------
VPN RD: 65000:100 (Customer A)
Type|Status| Network            |Next Hop    |Label |LocPrf|Origin
-------------------------------------------------------------------------------
vpn |u*>i  | 192.168.1.0/30     |1.1.1.1     |100004|100   |i
vpn |u*>i  | 192.168.100.1/32   |1.1.1.1     |100005|100   |i
vpn |u*>i  | 192.168.2.0/30     |2.2.2.2     |200004|100   |i
vpn |u*>i  | 192.168.200.1/32   |2.2.2.2     |200005|100   |i
-------------------------------------------------------------------------------
# IMPORTANTE: Debes ver rutas de ambos PEs (1.1.1.1 y 2.2.2.2)
# Con labels MPLS asignados
```

**En CE1:**
```bash
# Ver tabla de routing
A:ce1# show network-instance default route-table

# Output esperado:
--------------------------------------------------------------------
IPv4 unicast route table of network instance default
--------------------------------------------------------------------
+------------------+-------+------------+-------------+--------+---------+
|    Prefix        | ID    | Route Type | Route Owner | Active | Metric  |
+==================+=======+============+=============+========+=========+
| 192.168.1.0/30   | 0     | local      | local_route | True   | 0       |
| 192.168.1.2/32   | 0     | host       | local_route | True   | 0       |
| 192.168.100.1/32 | 0     | local      | local_route | True   | 0       |
| 192.168.2.0/30   | 0     | ospf       | ospf_mgr    | True   | 20      |
| 192.168.200.1/32 | 0     | ospf       | ospf_mgr    | True   | 20      |
+------------------+-------+------------+-------------+--------+---------+
--------------------------------------------------------------------
# IMPORTANTE: Debes ver rutas remotas (192.168.2.x, 192.168.200.1)
# aprendidas vía OSPF (en realidad vienen por MPLS L3VPN)
--------------------------------------------------------------------

# Ping a loopback de CE2
A:ce1# ping 192.168.200.1 network-instance default

# Output esperado:
Using network instance default
PING 192.168.200.1 (192.168.200.1) 56(84) bytes of data.
64 bytes from 192.168.200.1: icmp_seq=1 ttl=62 time=1.234 ms
64 bytes from 192.168.200.1: icmp_seq=2 ttl=62 time=1.156 ms
64 bytes from 192.168.200.1: icmp_seq=3 ttl=62 time=1.198 ms
64 bytes from 192.168.200.1: icmp_seq=4 ttl=62 time=1.167 ms
64 bytes from 192.168.200.1: icmp_seq=5 ttl=62 time=1.203 ms

--- 192.168.200.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4008ms
rtt min/avg/max/mdev = 1.156/1.192/1.234/0.028 ms

# ✅ Si el ping funciona, ¡MPLS L3VPN está completamente operativo!
```

**Si funciona, ¡MPLS L3VPN está operativo!** 🎉

### Crear Customer B (VRF separada)

**PE1:**
```bash
A:pe1# enter candidate

# VRF distinta para Customer B
A:pe1# network-instance customerB type ip-vrf
A:pe1# network-instance customerB route-distinguisher 65000:200
A:pe1# network-instance customerB protocols bgp-vpn route-target export target:65000:200
A:pe1# network-instance customerB protocols bgp-vpn route-target import target:65000:200

# Interface hacia CE1-CustomerB
A:pe1# interface ethernet-1/11 admin-state enable subinterface 200 ipv4 address 192.168.10.1/30
A:pe1# network-instance customerB interface ethernet-1/11.200

A:pe1# commit now
```

**Verificar aislamiento:**
```bash
# Routes de Customer A no deben verse en Customer B
A:pe1# show network-instance customerA route-table

# Output esperado (solo rutas de Customer A):
--------------------------------------------------------------------
IPv4 unicast route table of network instance customerA
--------------------------------------------------------------------
+------------------+-------+------------+-------------+--------+---------+
|    Prefix        | ID    | Route Type | Route Owner | Active | Metric  |
+==================+=======+============+=============+========+=========+
| 192.168.1.0/30   | 0     | local      | local_route | True   | 0       |
| 192.168.100.1/32 | 0     | ospf       | ospf_mgr    | True   | 10      |
| 192.168.2.0/30   | 0     | bgp-vpn    | bgp_mgr     | True   | 0       |
| 192.168.200.1/32 | 0     | bgp-vpn    | bgp_mgr     | True   | 10      |
+------------------+-------+------------+-------------+--------+---------+

A:pe1# show network-instance customerB route-table

# Output esperado (solo rutas de Customer B):
--------------------------------------------------------------------
IPv4 unicast route table of network instance customerB
--------------------------------------------------------------------
+------------------+-------+------------+-------------+--------+---------+
|    Prefix        | ID    | Route Type | Route Owner | Active | Metric  |
+==================+=======+============+=============+========+=========+
| 192.168.10.0/30  | 0     | local      | local_route | True   | 0       |
| 192.168.10.2/32  | 0     | host       | local_route | True   | 0       |
+------------------+-------+------------+-------------+--------+---------+

# ✅ IMPORTANTE: Las tablas están completamente separadas
# Customer A NO puede ver las rutas de Customer B y viceversa
# Esto es el aislamiento de MPLS L3VPN funcionando correctamente
```

---

## 12. Telemetría y Monitoreo

### ¿Por qué Telemetría?

Para tu proyecto de IA necesitas:
1. **Datos en tiempo real** (no cada 5 minutos)
2. **Datos estructurados** (JSON, no texto)
3. **Alto volumen** (múltiples métricas simultáneamente)
4. **APIs modernas** (gRPC, no SNMP legacy)

### Arquitectura de Telemetría

```
┌──────────────────────────────┐
│  Routers (SR Linux)          │
│  ├── gNMI telemetry          │
│  ├── SNMP (backup)           │
│  └── Syslog                  │
└──────────┬───────────────────┘
           │ (streaming data)
           ↓
┌──────────────────────────────┐
│  Telegraf (collector)        │
│  ├── Input: gNMI, SNMP       │
│  └── Output: InfluxDB        │
└──────────┬───────────────────┘
           │
           ↓
┌──────────────────────────────┐
│  InfluxDB (time-series DB)   │
│  └── Métricas históricas     │
└──────────┬───────────────────┘
           │
           ↓
┌──────────────────────────────┐
│  Grafana (visualización)     │
│  └── Dashboards             │
└──────────────────────────────┘
           │
           ↓
┌──────────────────────────────┐
│  Colab / Python              │
│  └── Modelo de IA            │
└──────────────────────────────┘
```

### Setup Completo con Telemetría

#### Topología con Stack de Monitoreo

```yaml
name: lab3-telemetry

topology:
  nodes:
    # Network devices
    pe1:
      kind: nokia_srlinux
      type: ixrd3
    p1:
      kind: nokia_srlinux
      type: ixrd2
    pe2:
      kind: nokia_srlinux
      type: ixrd3
    
    # Monitoring stack
    telegraf:
      kind: linux
      image: telegraf:latest
      binds:
        - telegraf.conf:/etc/telegraf/telegraf.conf:ro
      ports:
        - 9273:9273  # Prometheus metrics
    
    influxdb:
      kind: linux
      image: influxdb:2.7
      ports:
        - 8086:8086
      env:
        DOCKER_INFLUXDB_INIT_MODE: setup
        DOCKER_INFLUXDB_INIT_USERNAME: admin
        DOCKER_INFLUXDB_INIT_PASSWORD: adminpass123
        DOCKER_INFLUXDB_INIT_ORG: myorg
        DOCKER_INFLUXDB_INIT_BUCKET: network_metrics
        DOCKER_INFLUXDB_INIT_ADMIN_TOKEN: mytoken123
    
    grafana:
      kind: linux
      image: grafana/grafana:latest
      ports:
        - 3000:3000
      env:
        GF_SECURITY_ADMIN_PASSWORD: admin
  
  links:
    # Network topology
    - endpoints: ["pe1:e1-1", "p1:e1-1"]
    - endpoints: ["p1:e1-2", "pe2:e1-1"]
    
    # Management network (todos conectados)
    - endpoints: ["pe1:e1-49", "telegraf:eth1"]
    - endpoints: ["telegraf:eth2", "influxdb:eth1"]
    - endpoints: ["influxdb:eth2", "grafana:eth1"]
```

#### Configurar Telegraf

**Archivo `telegraf.conf`:**

```toml
[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000

# Input: gNMI desde routers SR Linux
[[inputs.gnmi]]
  addresses = ["clab-lab3-telemetry-pe1:57400", "clab-lab3-telemetry-p1:57400", "clab-lab3-telemetry-pe2:57400"]
  username = "admin"
  password = "NokiaSrl1!"
  encoding = "proto"
  redial = "10s"

  # Subscriptions para métricas
  [[inputs.gnmi.subscription]]
    name = "interface_stats"
    path = "/interface[name=ethernet-1/*]/statistics"
    subscription_mode = "sample"
    sample_interval = "10s"

  [[inputs.gnmi.subscription]]
    name = "cpu_usage"
    path = "/system/cpu/total"
    subscription_mode = "sample"
    sample_interval = "10s"

  [[inputs.gnmi.subscription]]
    name = "memory_usage"
    path = "/system/memory"
    subscription_mode = "sample"
    sample_interval = "10s"

  [[inputs.gnmi.subscription]]
    name = "bgp_neighbors"
    path = "/network-instance[name=default]/protocols/bgp/neighbor"
    subscription_mode = "sample"
    sample_interval = "30s"

# Output: InfluxDB v2
[[outputs.influxdb_v2]]
  urls = ["http://clab-lab3-telemetry-influxdb:8086"]
  token = "mytoken123"
  organization = "myorg"
  bucket = "network_metrics"
```

#### Habilitar gNMI en SR Linux

**PE1:**
```bash
A:pe1# enter candidate

# Habilitar gNMI server
A:pe1# system gnmi-server
A:pe1# admin-state enable
A:pe1# timeout 7200
A:pe1# rate-limit 60
A:pe1# session-limit 20
A:pe1# network-instance mgmt
A:pe1# admin-state enable

A:pe1# commit now
```

Repite en todos los routers.

#### Verificar Streaming Telemetry

```bash
# Verificar que Telegraf esté recibiendo datos
sudo docker exec clab-lab3-telemetry-telegraf telegraf --test

# Ver logs de Telegraf
sudo docker logs -f clab-lab3-telemetry-telegraf

# Acceder a Grafana
# Abrir navegador: http://localhost:3000
# Usuario: admin
# Password: admin

# Configurar datasource InfluxDB en Grafana:
# - URL: http://clab-lab3-telemetry-influxdb:8086
# - Organization: myorg
# - Token: mytoken123
# - Default bucket: network_metrics
```

#### Crear Dashboard en Grafana

**Panel 1: Interface Traffic**
```sql
from(bucket: "network_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r["_measurement"] == "interface_stats")
  |> filter(fn: (r) => r["_field"] == "in-octets" or r["_field"] == "out-octets")
  |> aggregateWindow(every: 1m, fn: mean, createEmpty: false)
```

**Panel 2: CPU Usage**
```sql
from(bucket: "network_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r["_measurement"] == "cpu_usage")
  |> filter(fn: (r) => r["_field"] == "instant")
  |> aggregateWindow(every: 1m, fn: mean, createEmpty: false)
```

---

## 13. Integración con Python

### Conectar Python con gNMI

**Instalar librerías:**
```bash
pip install pygnmi pandas influxdb-client
```

**Script para leer telemetría:**

```python
from pygnmi.client import gNMIclient
import json

# Conectar a router
host = '172.20.20.2'  # IP de clab-lab3-telemetry-pe1
port = 57400
username = 'admin'
password = 'NokiaSrl1!'

with gNMIclient(target=(host, port), username=username, password=password, insecure=True) as gc:
    # Get interface statistics
    path = ['/interface[name=ethernet-1/1]/statistics']
    result = gc.get(path=path, encoding='json_ietf')
    
    print(json.dumps(result, indent=2))
```

**Output esperado:**
```json
{
  "notification": [
    {
      "update": [
        {
          "path": "interface[name=ethernet-1/1]/statistics",
          "val": {
            "in-octets": 1234567,
            "out-octets": 7654321,
            "in-packets": 12345,
            "out-packets": 54321,
            "in-errors": 0,
            "out-errors": 0
          }
        }
      ]
    }
  ]
}
```

### Suscribirse a Streaming Telemetry

```python
from pygnmi.client import gNMIclient
import time

def telemetry_callback(message):
    """Callback para procesar cada update"""
    print(f"Timestamp: {time.time()}")
    print(f"Data: {message}")
    
    # Aquí podrías:
    # 1. Guardar en InfluxDB
    # 2. Enviar a modelo de IA
    # 3. Generar alertas

host = '172.20.20.2'
port = 57400
username = 'admin'
password = 'NokiaSrl1!'

with gNMIclient(target=(host, port), username=username, password=password, insecure=True) as gc:
    # Subscribe to interface stats (streaming)
    subscription = {
        'subscription': [
            {
                'path': '/interface[name=ethernet-1/1]/statistics',
                'mode': 'sample',
                'sample_interval': 10000000000  # 10 seconds in nanoseconds
            }
        ],
        'mode': 'stream',
        'encoding': 'json_ietf'
    }
    
    # Recibir actualizaciones continuamente
    for update in gc.subscribe(subscribe=subscription):
        telemetry_callback(update)
```

### Leer desde InfluxDB y Entrenar Modelo

```python
import pandas as pd
from influxdb_client import InfluxDBClient
from sklearn.preprocessing import StandardScaler
import numpy as np

# Conectar a InfluxDB
client = InfluxDBClient(
    url="http://localhost:8086",
    token="mytoken123",
    org="myorg"
)

query_api = client.query_api()

# Query: últimas 24 horas de métricas de interfaces
query = '''
from(bucket: "network_metrics")
  |> range(start: -24h)
  |> filter(fn: (r) => r["_measurement"] == "interface_stats")
  |> filter(fn: (r) => r["_field"] == "in-octets" or r["_field"] == "out-octets")
  |> aggregateWindow(every: 1m, fn: mean, createEmpty: false)
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
'''

result = query_api.query_data_frame(query)

# Convertir a DataFrame
df = pd.DataFrame(result)
print(df.head())

# Preparar para modelo de IA
# (Asumiendo que tienes un LSTM Autoencoder ya)

# Feature engineering
df['in_octets_diff'] = df['in-octets'].diff()
df['out_octets_diff'] = df['out-octets'].diff()
df['total_octets'] = df['in-octets'] + df['out-octets']

# Normalizar
scaler = StandardScaler()
features = ['in-octets', 'out-octets', 'in_octets_diff', 'out_octets_diff', 'total_octets']
df[features] = scaler.fit_transform(df[features])

# Ahora puedes entrenar tu modelo LSTM Autoencoder
# (código que ya tienes de tu experiencia previa)
```

---

## 14. Automation y APIs

### NETCONF con Python

```python
from ncclient import manager
import xml.etree.ElementTree as ET

# Conectar via NETCONF
m = manager.connect(
    host='172.20.20.2',
    port=830,
    username='admin',
    password='NokiaSrl1!',
    hostkey_verify=False,
    device_params={'name': 'default'}
)

# Get config
config = m.get_config(source='running').data_xml
print(config)

# Edit config (cambiar descripción de interfaz)
config_change = '''
<config>
  <interface xmlns="urn:nokia.com:srlinux">
    <name>ethernet-1/1</name>
    <description>Automated change via NETCONF</description>
  </interface>
</config>
'''

m.edit_config(target='candidate', config=config_change)
m.commit()
```

### REST API con Requests

```python
import requests
import json

# SR Linux REST API (requiere habilitar gnmi-server)
base_url = "https://172.20.20.2/jsonrpc"
headers = {
    "Content-Type": "application/json"
}
auth = ('admin', 'NokiaSrl1!')

# Get interface info
payload = {
    "jsonrpc": "2.0",
    "id": 1,
    "method": "get",
    "params": {
        "commands": [
            {
                "path": "/interface[name=ethernet-1/1]",
                "datastore": "state"
            }
        ]
    }
}

response = requests.post(base_url, headers=headers, auth=auth, json=payload, verify=False)
print(json.dumps(response.json(), indent=2))
```

### Automation: Configurar 10 Routers con Python

```python
from pygnmi.client import gNMIclient
import yaml

# Leer topología
with open('topology.yml', 'r') as f:
    topology = yaml.safe_load(f)

# Configuración template
def configure_ospf(router_ip, router_id, interfaces):
    with gNMIclient(target=(router_ip, 57400), username='admin', password='NokiaSrl1!', insecure=True) as gc:
        
        # Build config
        config = {
            "network-instance": [
                {
                    "name": "default",
                    "protocols": {
                        "ospf": {
                            "instance": [
                                {
                                    "name": "main",
                                    "admin-state": "enable",
                                    "version": "ospf-v2",
                                    "router-id": router_id,
                                    "area": [
                                        {
                                            "area-id": "0.0.0.0",
                                            "interface": interfaces
                                        }
                                    ]
                                }
                            ]
                        }
                    }
                }
            ]
        }
        
        # Apply config via gNMI
        gc.set(update=[(path, config)])
        print(f"Configured OSPF on {router_ip}")

# Configurar todos los routers
routers = [
    {"ip": "172.20.20.2", "id": "1.1.1.1", "interfaces": ["ethernet-1/1.0", "system0.0"]},
    {"ip": "172.20.20.3", "id": "2.2.2.2", "interfaces": ["ethernet-1/1.0", "ethernet-1/2.0", "system0.0"]},
    # ... más routers
]

for router in routers:
    configure_ospf(router["ip"], router["id"], router["interfaces"])
```

---

## 15. Escenarios para Demo de IA

### Escenario 1: Predicción de Falla de Interfaz

**Objetivo:** Detectar degradación gradual antes de falla completa.

#### Setup

```bash
# Deploy topología MPLS
sudo containerlab deploy -t lab3-mpls-l3vpn.yml

# Configurar todo (MPLS L3VPN completo)
# ... (usa configs de sección 11)

# Iniciar recolección de telemetría
# Telegraf/InfluxDB ya corriendo
```

#### Generar Datos Normales (1 semana simulada)

```python
import time
import random

def generate_normal_traffic(duration_hours=168):  # 1 week
    """Simula tráfico normal durante 1 semana"""
    
    for hour in range(duration_hours):
        # Tráfico con patrón diurno normal
        base_traffic = 1000000  # 1 Mbps baseline
        
        # Patrón diurno (más tráfico durante el día)
        hour_of_day = hour % 24
        if 8 <= hour_of_day <= 18:  # Horario laboral
            traffic_multiplier = random.uniform(1.5, 2.0)
        else:
            traffic_multiplier = random.uniform(0.3, 0.7)
        
        current_traffic = base_traffic * traffic_multiplier
        
        # Pequeñas variaciones aleatorias
        noise = random.uniform(0.95, 1.05)
        final_traffic = current_traffic * noise
        
        print(f"Hour {hour}: {final_traffic:.2f} bytes/sec")
        
        # Aquí enviarías datos reales al router o simularías en DB
        time.sleep(1)  # En realidad serían 3600 segundos (1 hora)

# Ejecutar
generate_normal_traffic()
```

#### Simular Degradación Progresiva

```bash
# Script en el router para simular drops graduales
# (ejecutar desde host)

for i in {1..48}; do
  # Incrementar packet drops artificialmente
  drop_rate=$(echo "scale=2; $i * 0.01" | bc)
  
  # Usar tc (traffic control) para simular pérdida
  sudo docker exec clab-lab3-mpls-pe1 tc qdisc replace dev eth1 root netem loss ${drop_rate}%
  
  echo "Hour $i: Loss rate ${drop_rate}%"
  
  sleep 3600  # 1 hora (o menos para demo)
done
```

#### Entrenar Modelo de Detección

```python
import pandas as pd
import numpy as np
from tensorflow import keras
from sklearn.preprocessing import StandardScaler

# 1. Cargar datos de InfluxDB (ya vimos cómo)
# df = cargar_datos_influxdb()

# 2. Preparar datos
def prepare_sequences(data, sequence_length=60):
    """Crear secuencias para LSTM"""
    X = []
    for i in range(len(data) - sequence_length):
        X.append(data[i:i+sequence_length])
    return np.array(X)

# 3. Definir modelo LSTM Autoencoder
def build_model(sequence_length, n_features):
    model = keras.Sequential([
        # Encoder
        keras.layers.LSTM(128, activation='relu', input_shape=(sequence_length, n_features), return_sequences=True),
        keras.layers.LSTM(64, activation='relu', return_sequences=False),
        keras.layers.RepeatVector(sequence_length),
        
        # Decoder
        keras.layers.LSTM(64, activation='relu', return_sequences=True),
        keras.layers.LSTM(128, activation='relu', return_sequences=True),
        keras.layers.TimeDistributed(keras.layers.Dense(n_features))
    ])
    
    model.compile(optimizer='adam', loss='mse')
    return model

# 4. Entrenar solo con datos "normales" (sin anomalías)
# Asume que df contiene: ['timestamp', 'in-octets', 'out-octets', 'in-errors', 'out-errors']

features = ['in-octets', 'out-octets', 'in-errors', 'out-errors']
scaler = StandardScaler()
df[features] = scaler.fit_transform(df[features])

X_train = prepare_sequences(df[features].values, sequence_length=60)

model = build_model(sequence_length=60, n_features=len(features))
model.fit(X_train, X_train, epochs=50, batch_size=32, validation_split=0.1)

# 5. Guardar modelo
model.save('network_anomaly_detector.h5')

# 6. En tiempo real, detectar anomalías
def detect_anomaly(model, new_data, threshold=0.01):
    """Detecta si nueva secuencia es anómala"""
    reconstruction = model.predict(new_data)
    mse = np.mean(np.power(new_data - reconstruction, 2), axis=1)
    
    if mse > threshold:
        return True, mse  # Anomalía detectada
    return False, mse  # Normal
```

#### Demo en Vivo

```bash
# Terminal 1: Mostrar Grafana dashboard
# http://localhost:3000

# Terminal 2: Ejecutar predictor en Python
python anomaly_detector_realtime.py

# Terminal 3: Simular falla gradual
./simulate_degradation.sh

# En Grafana verás:
# - Gráfica de tráfico normal → degradándose
# - Alerta de modelo de IA: "Anomalía detectada con 85% confianza"
# - Predicción: "Falla inminente en 2 horas"
```

### Escenario 2: Diagnóstico Inteligente

**Objetivo:** Sistema que sugiere troubleshooting basándose en síntomas.

```python
from sklearn.ensemble import RandomForestClassifier
import pandas as pd

# Dataset de casos históricos
# Cada fila: [síntomas] → [causa raíz]

data = {
    'bgp_flapping': [1, 0, 0, 1, 0],
    'high_cpu': [0, 1, 1, 0, 1],
    'packet_loss': [1, 1, 0, 1, 0],
    'ospf_down': [0, 0, 1, 0, 1],
    'causa_raiz': ['BGP_config_error', 'CPU_spike', 'OSPF_neighbor_down', 'BGP_dampening', 'Memory_leak']
}

df = pd.DataFrame(data)

# Entrenar clasificador
X = df[['bgp_flapping', 'high_cpu', 'packet_loss', 'ospf_down']]
y = df['causa_raiz']

clf = RandomForestClassifier(n_estimators=100)
clf.fit(X, y)

# En producción: nuevo incidente
nuevo_incidente = [[1, 0, 1, 0]]  # BGP flapping + packet loss
prediccion = clf.predict(nuevo_incidente)
probabilidad = clf.predict_proba(nuevo_incidente)

print(f"Causa más probable: {prediccion[0]}")
print(f"Confianza: {max(probabilidad[0]) * 100:.2f}%")

# Output:
# Causa más probable: BGP_config_error
# Confianza: 85.00%
```

---

## Resumen Nivel Avanzado

✅ **Has dominado:**
- MPLS L3VPN completo (PE-P-PE con VRFs)
- Streaming telemetry (gNMI, Telegraf, InfluxDB, Grafana)
- Integración con Python (pygnmi, APIs)
- Automation avanzada (NETCONF, REST)
- Escenarios realistas para demostrar IA en redes

🎯 **Proyecto Final:** Demo completa para tus jefes
- Topología MPLS L3VPN multi-cliente
- Telemetría en tiempo real
- Modelo de IA prediciendo fallas
- Dashboard mostrando predicciones
- Documentación profesional

---

# Apéndices

## A. Comandos de Referencia Rápida

### Containerlab CLI

```bash
# Gestión básica
sudo containerlab deploy -t <file.yml>
sudo containerlab destroy -t <file.yml>
sudo containerlab inspect -t <file.yml>
sudo containerlab inspect --all
sudo containerlab save -t <file.yml>

# Con opciones
sudo containerlab deploy -t <file.yml> --reconfigure
sudo containerlab destroy --all --cleanup
```

### Docker Useful Commands

```bash
# Contenedores
docker ps                    # Listar activos
docker ps -a                 # Listar todos
docker stop <container>      # Parar
docker start <container>     # Iniciar
docker restart <container>   # Reiniciar
docker rm <container>        # Eliminar
docker logs <container>      # Ver logs
docker stats                 # Ver recursos

# Redes
docker network ls
docker network inspect clab

# Imágenes
docker images
docker pull <image>
docker rmi <image>

# Limpieza
docker system prune -a       # Limpiar todo
```

### SR Linux CLI Quick Reference

```bash
# Navegación
enter candidate              # Entrar a config mode
quit                        # Salir de config mode
commit now                  # Aplicar cambios
diff                        # Ver cambios pendientes

# Show commands
show interface brief
show network-instance default route-table
show network-instance default protocols ospf neighbor
show network-instance default protocols bgp neighbor
show version
show system cpu
show system memory

# Troubleshooting
ping <ip> network-instance default
traceroute <ip> network-instance default
tcpdump -i ethernet-1/1
```

---

## B. Troubleshooting Common Issues

### Issue: Containerlab deploy fails

**Error:** `cannot create container`

**Solución:**
```bash
# Verificar Docker está corriendo
sudo systemctl status docker
sudo systemctl start docker

# Verificar permisos
sudo usermod -aG docker $USER
# Logout y login

# Limpiar contenedores anteriores
sudo containerlab destroy --all --cleanup
```

### Issue: Cannot access CLI

**Error:** `connection refused`

**Solución:**
```bash
# Verificar contenedor está corriendo
sudo docker ps | grep <node-name>

# Ver logs del contenedor
sudo docker logs clab-<lab>-<node>

# Reiniciar contenedor
sudo docker restart clab-<lab>-<node>
```

### Issue: No connectivity between nodes

**Diagnóstico:**
```bash
# 1. Verificar links creados
sudo docker network inspect clab | grep -A 10 Containers

# 2. Verificar interfaces en nodos
sudo docker exec clab-<lab>-<node> ip link

# 3. Ping directo entre contenedores
sudo docker exec clab-<lab>-node1 ping <ip-node2>
```

---

## C. Recursos y Documentación

### Oficial

- **Containerlab Docs:** https://containerlab.dev/
- **SR Linux Documentation:** https://documentation.nokia.com/srlinux/
- **SR Linux Learn:** https://learn.srlinux.dev/

### GitHub Repos

- **Containerlab:** https://github.com/srl-labs/containerlab
- **SR Linux Labs:** https://github.com/srl-labs

### Comunidad

- **Containerlab Discussions:** https://github.com/srl-labs/containerlab/discussions
- **Network to Code Slack:** https://networktocode.slack.com/

---

## D. Próximos Pasos

1. **Practica con los ejercicios** de cada nivel
2. **Experimenta con diferentes topologías**
3. **Integra con tus proyectos de IA existentes**
4. **Documenta tus casos de uso**
5. **Comparte conocimiento con tu equipo**

---

¡Felicitaciones por completar este tutorial! 🎉

Ahora tienes las habilidades para:
- Simular redes complejas en minutos
- Configurar protocolos avanzados (OSPF, BGP, MPLS)
- Recolectar telemetría en tiempo real
- Integrar con modelos de IA
- Automatizar configuraciones
- Crear demos profesionales

**¿Listo para tu demo?** ¡Adelante! 🚀