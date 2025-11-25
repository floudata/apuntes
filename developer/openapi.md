# De Script de Laboratorio a Infraestructura Científica: Guía DevSecOps

**Autor:** Gemini (AI Assistant)
**Perfil:** Físico Programador
**Objetivo:** Transformar código de simulación en una API RESTful estandarizada, segura, contenerizada y auditada automáticamente.

-----

## 1. El Concepto: OpenAPI (El Contrato)

Antes de escribir código, definimos las reglas. **OpenAPI** (anteriormente Swagger) es la especificación que describe tu API. En el desarrollo moderno, existen dos filosofías principales: "Contract First" (escribir el YAML de OpenAPI primero) y "Code First" (escribir código y generar el OpenAPI). Nosotros usaremos **Code First** con FastAPI por su agilidad para prototipado científico, pero el resultado final es el mismo: un contrato estricto.

*   **Analogía:** Es el manual del equipo experimental. Define qué inputs (voltaje, temperatura) acepta la máquina y qué outputs entrega, sin revelar la circuitería interna.
*   **Ventaja:** Permite generar clientes automáticos (SDKs) para Python, Matlab o C++, y documentación interactiva (Swagger UI, ReDoc) que permite probar la API desde el navegador.

### Ejemplo 1: Del Modelo al Esquema
Cuando defines una clase en Python con Pydantic, se traduce automáticamente a un esquema JSON estandarizado.

**Python (Pydantic):**
```python
class Particula(BaseModel):
    nombre: str
    spin: float
```

**OpenAPI (JSON Schema generado):**
```json
{
  "Particula": {
    "properties": {
      "nombre": { "type": "string", "title": "Nombre" },
      "spin": { "type": "number", "title": "Spin" }
    },
    "type": "object",
    "required": ["nombre", "spin"],
    "title": "Particula"
  }
}
```

### Ejemplo 2: Interacción Cliente vs. SDK
Sin OpenAPI, un colega tendría que adivinar la URL y los parámetros. Con OpenAPI, puede generar un cliente.

**Sin SDK (Manual - `requests`):**
```python
import requests
# El usuario debe saber la URL exacta y la estructura del JSON
resp = requests.post("http://api/particula", json={"nombre": "electron", "spin": 0.5})
```

**Con SDK Generado (Automático):**
```python
import fisica_client
# El IDE autocompleta los métodos y valida tipos antes de enviar
api = fisica_client.DefaultApi()
particula = fisica_client.Particula(nombre="electron", spin=0.5)
api.crear_particula(particula)
```

-----

## 2. El Código: FastAPI y Tipado Fuerte

Usamos **FastAPI** porque automatiza la creación del OpenAPI basándose en las "Type Hints" de Python. Implementamos validación física (ej. masa > 0) usando **Pydantic** y gestión de dependencias para recursos compartidos.

### `main.py` (El Núcleo)

```python
import io
import time
import uuid
import numpy as np
from fastapi import FastAPI, UploadFile, File, BackgroundTasks, Security, HTTPException, status, Depends
from fastapi.responses import Response
from fastapi.security import APIKeyHeader
from pydantic import BaseModel, Field, validator

app = FastAPI(title="Laboratorio de Física API", version="1.0.0")

# --- SEGURIDAD ---
API_KEY_NAME = "X-API-Key"
API_KEY_SECRET = "fisica-2025" # En prod: os.getenv("API_KEY")

api_key_header = APIKeyHeader(name=API_KEY_NAME, auto_error=True)

def validar_api_key(api_key: str = Security(api_key_header)):
    if api_key != API_KEY_SECRET:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Credenciales inválidas"
        )
    return api_key

# --- MODELOS DE DATOS ---
class InputCinematica(BaseModel):
    masa: float = Field(..., gt=0, description="Masa en kg")
    velocidad: float = Field(..., description="Velocidad en m/s")

    # Ejemplo 1: Validación de consistencia física
    @validator('velocidad')
    def velocidad_no_superaluminal(cls, v):
        c = 299792458
        if abs(v) >= c:
            raise ValueError('La velocidad no puede superar la velocidad de la luz')
        return v

# --- DEPENDENCIAS ---
# Ejemplo 2: Recurso compartido (Singleton)
class ModeloCientifico:
    def __init__(self):
        print("Cargando modelo pesado de ML...")
        self.factor_calibracion = 1.05

modelo_global = ModeloCientifico()

def get_modelo():
    return modelo_global

# --- ENDPOINT 1: CÁLCULO SIMPLE (JSON) ---
@app.post("/cinematica/energia", dependencies=[Security(validar_api_key)])
def calcular_energia(datos: InputCinematica, modelo: ModeloCientifico = Depends(get_modelo)):
    """Calcula Ek = 1/2 m v^2 ajustado por calibración"""
    ek = 0.5 * datos.masa * (datos.velocidad ** 2) * modelo.factor_calibracion
    return {"energia_joules": ek}

# --- ENDPOINT 2: HEAVY DATA (Numpy Binary) ---
@app.post("/procesar/fft", dependencies=[Security(validar_api_key)])
async def calcular_fft(archivo: UploadFile = File(...)):
    """Recibe binario .npy -> Calcula FFT -> Devuelve binario .npy"""
    contenido = await archivo.read()
    matriz = np.load(io.BytesIO(contenido)) # Carga en memoria
    
    # Procesamiento (FFT)
    resultado = np.real(np.fft.fft2(matriz))
    
    # Serialización eficiente
    buffer = io.BytesIO()
    np.save(buffer, resultado)
    return Response(content=buffer.getvalue(), media_type="application/octet-stream")

# --- ENDPOINT 3: SIMULACIÓN ASÍNCRONA (Background Tasks) ---
job_db = {}

def simulacion_larga(job_id: str):
    time.sleep(10) # Simula Monte Carlo pesado
    job_db[job_id] = "Completado: -13.6 eV"

@app.post("/simulacion/start", status_code=202, dependencies=[Security(validar_api_key)])
async def iniciar_simulacion(background_tasks: BackgroundTasks):
    job_id = str(uuid.uuid4())
    background_tasks.add_task(simulacion_larga, job_id)
    return {"job_id": job_id, "status": "queued"}
```

-----

## 3. La Infraestructura: Docker y SSL

Para garantizar la **reproducibilidad** (que funcione igual en tu laptop que en AWS) y la **seguridad en tránsito** (HTTPS), empaquetamos todo. Usamos "Multi-stage builds" para mantener la imagen ligera y segura.

### Generación de Certificados (Local)

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=localhost"
```

### `Dockerfile` (El Plano)

Usamos una estrategia de **Multi-stage build**. Primero una imagen "builder" compila y prepara, y luego copiamos solo lo necesario a una imagen "runner" final más pequeña y segura.

**Ejemplo 1: Dockerfile Optimizado**
```dockerfile
# Etapa 1: Builder
FROM python:3.9-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Etapa 2: Runner (Imagen final)
FROM python:3.9-slim
WORKDIR /app

# Copiamos solo las librerías instaladas del builder
COPY --from=builder /root/.local /root/.local
COPY . .

# Aseguramos que los scripts de python estén en el PATH
ENV PATH=/root/.local/bin:$PATH

# Escuchamos en puerto 443 (HTTPS standard)
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "443", "--ssl-keyfile", "/app/certs/key.pem", "--ssl-certfile", "/app/certs/cert.pem"]
```

### `docker-compose.yaml` (La Orquestación)

Define cómo se ejecuta el contenedor. Aquí añadimos límites de recursos y chequeos de salud (Healthchecks) para que el orquestador sepa si el servicio está vivo.

**Ejemplo 2: Compose con Healthcheck y Límites**
```yaml
version: '3.8'
services:
  api-fisica:
    build: .
    ports:
      - "8443:443"
    volumes:
      - ./key.pem:/app/certs/key.pem:ro
      - ./cert.pem:/app/certs/cert.pem:ro
    # Límite de recursos para evitar colgar el servidor host
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
    # Verificación de salud
    healthcheck:
      test: ["CMD", "curl", "-k", "-f", "https://localhost:443/docs"]
      interval: 30s
      timeout: 10s
      retries: 3
```

-----

## 4. DevSecOps: Automatización y Auditoría

No confiamos en la memoria humana. Implementamos un pipeline en **GitHub Actions** que se activa con cada `git push`. Este pipeline actúa como un "filtro de seguridad" antes de construir el software, siguiendo la filosofía **"Shift Left"** (encontrar errores lo antes posible).

**Archivo:** `.github/workflows/devsecops.yaml`

### Las 3 Puertas de Seguridad (Security Gates)

1.  **SAST (Bandit):** Static Application Security Testing. Analiza tu código Python buscando patrones inseguros.
2.  **SCA (Safety):** Software Composition Analysis. Analiza tus dependencias (`requirements.txt`) contra bases de datos de vulnerabilidades conocidas.
3.  **Container Scanning (Trivy):** Analiza la imagen Docker final buscando fallos en el sistema operativo base.

### Ejemplo 1: Lo que detecta Bandit (SAST)
Si escribes código inseguro, Bandit lo bloqueará.

**Código Inseguro:**
```python
import subprocess
# Bandit detecta: [B404] Import of subprocess.
# Bandit detecta: [B602] subprocess call with shell=True identified, security issue.
subprocess.call("echo 'hola'", shell=True)
```

**Código Seguro (Corrección):**
```python
import subprocess
# Usar shell=False y pasar argumentos como lista
subprocess.call(["echo", "hola"], shell=False)
```

### Ejemplo 2: Reporte de Vulnerabilidad (SCA/Container)
Un reporte típico de fallo en el pipeline se ve así:

```text
+==============================================================================+
|                                                                              |
|                               /$$$$$$            /$$                         |
|                              /$$__  $$          | $$                         |
|           /$$$$$$$  /$$$$$$ | $$  \__//$$$$$$  /$$$$$$   /$$   /$$           |
|          /$$_____/ |____  $$| $$$$   /$$__  $$|_  $$_/  | $$  | $$           |
|         |  $$$$$$   /$$$$$$$| $$_/  | $$$$$$$$  | $$    | $$  | $$           |
|          \____  $$ /$$__  $$| $$    | $$_____/  | $$ /$$| $$  | $$           |
|          /$$$$$$$/|  $$$$$$$| $$    |  $$$$$$$  |  $$$$/|  $$$$$$$           |
|         |_______/  \_______/|__/     \_______/   \___/   \____  $$           |
|                                                          /$$  | $$           |
|                                                         |  $$$$$$/           |
|  by pyup.io                                              \______/            |
|                                                                              |
+==============================================================================+
REPORT
  > Django (3.2.0) found in requirements.txt
  > ID: 45678
  > ADVISORY: Django 3.2.0 is vulnerable to XSS...
  > REMEDIATION: Upgrade to Django >= 3.2.10
```

```yaml
name: Pipeline DevSecOps
on: [push]

jobs:
  auditoria-y-construccion:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'

      - name: Instalar Herramientas
        run: |
          pip install -r requirements.txt
          pip install bandit safety

      # --- GATE 1: Código ---
      - name: Run Bandit (SAST)
        run: bandit -r . -lll

      # --- GATE 2: Dependencias ---
      - name: Run Safety (SCA)
        run: safety check -r requirements.txt
        continue-on-error: false

      # --- CONSTRUCCIÓN ---
      - name: Build Docker
        run: docker build -t api-fisica:${{ github.sha }} .

      # --- GATE 3: Infraestructura ---
      - name: Run Trivy (Container Security)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'api-fisica:${{ github.sha }}'
          format: 'table'
          exit-code: '1' # Rompe el build si hay fallos críticos
          severity: 'CRITICAL,HIGH'
```

-----

## 5. Resumen de Flujo de Trabajo

Si sigues esta guía, este será tu día a día como **Físico DevSecOps**. El ciclo se basa en un **Feedback Loop** rápido: codificar, probar, auditar, corregir.

1.  **Desarrollo:** Modificas tu función de cálculo en `main.py`.
2.  **Prueba Local:** Levantas Docker con `docker-compose up`. Pruebas con HTTPS.
3.  **Commit:** Haces `git push`.
4.  **Auditoría Automática:** GitHub Actions despierta.
      * Revisa tu código (Bandit).
      * Revisa tus librerías (Safety).
      * Construye el Docker.
      * Escanea el Docker (Trivy).
5.  **Resultado:**
      * ✅ **Verde:** Tu código es seguro y reproducible.
      * ❌ **Rojo:** El sistema te avisa que una librería tiene un agujero de seguridad. Corriges y repites.

### Ejemplo 1: El "Broken Build" (Fallo)
Imagina que subes un cambio rápido el viernes a las 5 PM. Olvidaste quitar una clave API hardcodeada para probar.
*   **Acción:** `git push`
*   **Resultado:** A los 2 minutos, recibes un email: "Pipeline Failed".
*   **Causa:** Bandit detectó `Possible hardcoded password`.
*   **Solución:** Borras la clave, usas variables de entorno, y haces push de nuevo. El sistema te salvó de exponer un secreto.

### Ejemplo 2: Despliegue Continuo (Éxito)
Tu simulación está lista.
*   **Acción:** Creas un tag: `git tag v1.0.0 && git push --tags`.
*   **Pipeline:** Al detectar un tag, se activa un paso extra: `docker push mi-registry/api-fisica:v1.0.0`.
*   **Resultado:** En 5 minutos, la nueva versión está disponible en el servidor de producción, lista para que tus colegas la usen.

### Siguiente Paso Lógico

Una vez que el pipeline es verde, el paso final (Despliegue Continuo o CD) sería que GitHub empuje esa imagen Docker automáticamente a un servidor en la nube (AWS, Google Cloud o Azure) para que esté disponible para el mundo.