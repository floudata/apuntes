# DevSecOps Avanzado: De Script Científico a Infraestructura de Producción

**Versión Expandida y Profundizada**  
**Autor Original:** Gemini (AI Assistant)  
**Expandido por:** Claude  
**Perfil:** Físico Programador / DevSecOps Engineer  
**Objetivo:** Guía completa para transformar código de simulación en una API RESTful de nivel enterprise

---

## Tabla de Contenidos

1. [OpenAPI: El Contrato Detallado](#1-openapi-el-contrato-detallado)
2. [FastAPI Avanzado](#2-fastapi-avanzado)
3. [Seguridad en Profundidad](#3-seguridad-en-profundidad)
4. [Infraestructura Docker Avanzada](#4-infraestructura-docker-avanzada)
5. [Testing Completo](#5-testing-completo)
6. [DevSecOps Pipeline Expandido](#6-devsecops-pipeline-expandido)
7. [Monitoreo y Observabilidad](#7-monitoreo-y-observabilidad)
8. [Despliegue Continuo (CD)](#8-despliegue-continuo-cd)
9. [Kubernetes y Escalabilidad](#9-kubernetes-y-escalabilidad)
10. [Mejores Prácticas de Producción](#10-mejores-prácticas-de-producción)

---

## 1. OpenAPI: El Contrato Detallado

### 1.1 Contract-First vs Code-First: Análisis Profundo

#### Contract-First (YAML primero)

**Ventajas:**
- Diseño independiente de la implementación
- Stakeholders no técnicos pueden revisar el contrato
- Generación de mocks para testing paralelo

**Ejemplo de Especificación OpenAPI 3.0:**

```yaml
openapi: 3.0.3
info:
  title: Laboratorio de Física API
  description: |
    API para simulaciones científicas con:
    - Validación física de parámetros
    - Procesamiento de datos en formato binario
    - Simulaciones asíncronas de larga duración
  version: 1.0.0
  contact:
    name: Equipo de Física Computacional
    email: fisica@lab.com
  license:
    name: MIT

servers:
  - url: https://api.fisica.com/v1
    description: Producción
  - url: https://staging.api.fisica.com/v1
    description: Staging
  - url: http://localhost:8000
    description: Desarrollo local

security:
  - ApiKeyAuth: []
  - BearerAuth: []

paths:
  /cinematica/energia:
    post:
      summary: Calcular energía cinética
      description: |
        Calcula la energía cinética ajustada con factor de calibración.
        Formula: Ek = 0.5 * m * v² * factor_calibración
      operationId: calcularEnergia
      tags:
        - Cinemática
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/InputCinematica'
            examples:
              electron:
                summary: Electrón a velocidad moderada
                value:
                  masa: 9.10938e-31
                  velocidad: 1000000
              proton:
                summary: Protón
                value:
                  masa: 1.67262e-27
                  velocidad: 500000
      responses:
        '200':
          description: Cálculo exitoso
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OutputEnergia'
        '400':
          description: Parámetros inválidos
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '403':
          description: API Key inválida
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '422':
          description: Error de validación física
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ValidationError'

components:
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
      description: API Key para autenticación básica
    
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: Token JWT para autenticación avanzada

  schemas:
    InputCinematica:
      type: object
      required:
        - masa
        - velocidad
      properties:
        masa:
          type: number
          format: double
          minimum: 0
          exclusiveMinimum: true
          description: Masa en kilogramos (kg)
          example: 9.10938e-31
        velocidad:
          type: number
          format: double
          minimum: -299792458
          maximum: 299792458
          description: Velocidad en metros por segundo (m/s)
          example: 1000000
      description: Parámetros de entrada para cálculo cinético

    OutputEnergia:
      type: object
      properties:
        energia_joules:
          type: number
          format: double
          description: Energía cinética en Joules
          example: 4.54969e-19
        timestamp:
          type: string
          format: date-time
          description: Momento del cálculo
        metadata:
          type: object
          properties:
            factor_calibracion:
              type: number
            version_modelo:
              type: string

    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: string
          example: INVALID_API_KEY
        message:
          type: string
          example: La API Key proporcionada no es válida
        details:
          type: object

    ValidationError:
      type: object
      properties:
        detail:
          type: array
          items:
            type: object
            properties:
              loc:
                type: array
                items:
                  type: string
              msg:
                type: string
              type:
                type: string
```

#### Code-First con FastAPI: Profundizado

**Ventajas:**
- Desarrollo ágil
- Type safety con Python typing
- Validación automática
- Documentación sincronizada con el código

**Estructura de Proyecto Profesional:**

```
proyecto/
├── app/
│   ├── __init__.py
│   ├── main.py              # Entry point
│   ├── config.py            # Configuración centralizada
│   ├── dependencies.py      # Inyección de dependencias
│   ├── api/
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── endpoints/
│   │   │   │   ├── cinematica.py
│   │   │   │   ├── simulacion.py
│   │   │   │   └── procesamiento.py
│   │   │   └── router.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── security.py      # Autenticación/Autorización
│   │   ├── errors.py        # Manejo de errores
│   │   └── logging.py       # Configuración de logs
│   ├── models/
│   │   ├── __init__.py
│   │   ├── schemas.py       # Pydantic models
│   │   └── domain.py        # Lógica de dominio
│   ├── services/
│   │   ├── __init__.py
│   │   ├── calculo.py       # Servicios de cálculo
│   │   └── simulacion.py    # Servicios de simulación
│   └── utils/
│       ├── __init__.py
│       └── validators.py    # Validadores custom
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── migrations/              # Si usas base de datos
├── docs/                    # Documentación adicional
├── scripts/                 # Scripts de utilidad
├── requirements/
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── .env.example
├── docker-compose.yml
├── Dockerfile
└── pyproject.toml
```

### 1.2 Generación de Clientes SDK

**Instalación de OpenAPI Generator:**

```bash
# Via npm
npm install @openapitools/openapi-generator-cli -g

# Via Docker
docker pull openapitools/openapi-generator-cli
```

**Generar Cliente Python:**

```bash
# Primero, exporta el OpenAPI schema de tu API
curl https://localhost:8443/openapi.json > openapi.json

# Genera el cliente
openapi-generator-cli generate \
  -i openapi.json \
  -g python \
  -o ./client-python \
  --additional-properties=packageName=fisica_client,projectName=fisica-client
```

**Usar el Cliente Generado:**

```python
# cliente_ejemplo.py
import fisica_client
from fisica_client.rest import ApiException

# Configuración
configuration = fisica_client.Configuration()
configuration.host = "https://api.fisica.com/v1"
configuration.api_key['X-API-Key'] = 'fisica-2025'

# Crear instancia del cliente
with fisica_client.ApiClient(configuration) as api_client:
    api = fisica_client.DefaultApi(api_client)
    
    try:
        # Llamada type-safe con autocompletado en IDE
        input_data = fisica_client.InputCinematica(
            masa=9.10938e-31,
            velocidad=1000000
        )
        
        resultado = api.calcular_energia(input_data)
        print(f"Energía: {resultado.energia_joules} J")
        
    except ApiException as e:
        print(f"Error al llamar a la API: {e}")
```

### 1.3 Versionado de API

**Estrategias de Versionado:**

1. **URL Path Versioning** (Recomendado para APIs públicas)
```python
# main.py
from fastapi import FastAPI

app = FastAPI()

# Versión 1
@app.post("/v1/cinematica/energia")
async def calcular_energia_v1(...):
    pass

# Versión 2 con breaking changes
@app.post("/v2/cinematica/energia")
async def calcular_energia_v2(...):
    pass
```

2. **Header Versioning**
```python
from fastapi import Header, HTTPException

@app.post("/cinematica/energia")
async def calcular_energia(
    api_version: str = Header(..., alias="API-Version")
):
    if api_version == "1.0":
        return calcular_v1()
    elif api_version == "2.0":
        return calcular_v2()
    else:
        raise HTTPException(400, "Versión no soportada")
```

3. **Query Parameter Versioning**
```python
@app.post("/cinematica/energia")
async def calcular_energia(version: str = "1.0"):
    if version == "1.0":
        return calcular_v1()
    # ...
```

**Estrategia Recomendada para Transición:**

```python
# api/v1/endpoints/cinematica.py
from fastapi import APIRouter, status
from typing import Union

router = APIRouter()

@router.post("/energia", deprecated=True)
async def calcular_energia_v1(...):
    """
    ⚠️ DEPRECATED: Esta versión será removida en 2026-01-01.
    Use /v2/cinematica/energia
    """
    pass

# api/v2/endpoints/cinematica.py
router = APIRouter()

@router.post("/energia")
async def calcular_energia_v2(...):
    """Nueva versión con soporte para relatividad especial"""
    pass
```

---

## 2. FastAPI Avanzado

### 2.1 Validación Física Avanzada con Pydantic

```python
# models/schemas.py
from pydantic import BaseModel, Field, validator, root_validator
from typing import Optional, List
from enum import Enum
import numpy as np

class TipoParticula(str, Enum):
    ELECTRON = "electron"
    PROTON = "proton"
    NEUTRON = "neutron"
    PHOTON = "photon"

class ConstantesFisicas:
    """Constantes fundamentales"""
    C = 299792458  # m/s
    H = 6.62607015e-34  # J⋅s (Planck)
    ME = 9.1093837015e-31  # kg (masa electrón)
    MP = 1.67262192369e-27  # kg (masa protón)

class InputCinematicaAvanzada(BaseModel):
    """Modelo con validaciones físicas complejas"""
    
    tipo_particula: TipoParticula
    velocidad: float = Field(
        ..., 
        description="Velocidad en m/s",
        ge=-ConstantesFisicas.C,
        le=ConstantesFisicas.C
    )
    energia_inicial: Optional[float] = Field(
        None,
        description="Energía inicial en Joules",
        gt=0
    )
    campo_magnetico: Optional[float] = Field(
        None,
        description="Campo magnético en Tesla"
    )
    
    # Propiedades calculadas
    masa: Optional[float] = None
    gamma: Optional[float] = None  # Factor de Lorentz
    
    class Config:
        use_enum_values = True
        schema_extra = {
            "example": {
                "tipo_particula": "electron",
                "velocidad": 100000000,
                "energia_inicial": 1.6e-19,
                "campo_magnetico": 0.5
            }
        }
    
    @validator('velocidad')
    def validar_velocidad_subluminal(cls, v):
        """Validar que v < c"""
        if abs(v) >= ConstantesFisicas.C:
            raise ValueError(
                f"La velocidad ({abs(v)} m/s) no puede igualar o "
                f"superar la velocidad de la luz ({ConstantesFisicas.C} m/s)"
            )
        return v
    
    @validator('masa', always=True)
    def calcular_masa(cls, v, values):
        """Asignar masa según tipo de partícula"""
        if 'tipo_particula' not in values:
            return v
        
        masas = {
            TipoParticula.ELECTRON: ConstantesFisicas.ME,
            TipoParticula.PROTON: ConstantesFisicas.MP,
            TipoParticula.NEUTRON: 1.67492749804e-27,
            TipoParticula.PHOTON: 0  # Sin masa en reposo
        }
        return masas.get(values['tipo_particula'], v)
    
    @validator('gamma', always=True)
    def calcular_factor_lorentz(cls, v, values):
        """Calcular factor relativista γ = 1/√(1 - v²/c²)"""
        if 'velocidad' not in values:
            return None
        
        velocidad = values['velocidad']
        beta = velocidad / ConstantesFisicas.C
        
        # Para v << c, γ ≈ 1
        if abs(beta) < 0.01:
            return 1.0
        
        return 1 / np.sqrt(1 - beta**2)
    
    @root_validator
    def validar_consistencia_fisica(cls, values):
        """Validaciones que requieren múltiples campos"""
        tipo = values.get('tipo_particula')
        velocidad = values.get('velocidad')
        energia = values.get('energia_inicial')
        
        # Los fotones siempre viajan a c
        if tipo == TipoParticula.PHOTON and abs(velocidad) != ConstantesFisicas.C:
            raise ValueError(
                "Los fotones deben viajar a la velocidad de la luz"
            )
        
        # Si se proporciona energía, debe ser consistente
        if energia is not None and values.get('masa', 0) > 0:
            # E = γmc² (energía total relativista)
            gamma = values.get('gamma', 1)
            masa = values.get('masa')
            energia_calculada = gamma * masa * ConstantesFisicas.C**2
            
            # Tolerancia del 1%
            if abs(energia - energia_calculada) / energia_calculada > 0.01:
                raise ValueError(
                    f"Energía inconsistente: proporcionada={energia}, "
                    f"calculada={energia_calculada}"
                )
        
        return values

class OutputCinematicaAvanzada(BaseModel):
    """Output con información completa"""
    energia_cinetica: float = Field(..., description="Energía cinética en J")
    energia_total: float = Field(..., description="Energía total (incluyendo masa en reposo)")
    momento: float = Field(..., description="Momento lineal en kg⋅m/s")
    longitud_de_onda: Optional[float] = Field(None, description="Longitud de onda de De Broglie en m")
    radio_ciclotrón: Optional[float] = Field(None, description="Radio de curvatura en campo magnético")
    
    metadata: dict = Field(
        default_factory=dict,
        description="Información adicional del cálculo"
    )
    
    class Config:
        schema_extra = {
            "example": {
                "energia_cinetica": 4.55e-21,
                "energia_total": 8.19e-14,
                "momento": 9.11e-23,
                "longitud_de_onda": 7.27e-12,
                "radio_ciclotrón": 1.14e-6,
                "metadata": {
                    "regimen": "no-relativista",
                    "gamma": 1.00005,
                    "timestamp": "2025-01-15T10:30:00Z"
                }
            }
        }
```

### 2.2 Dependency Injection Avanzada

```python
# dependencies.py
from typing import Annotated, Generator
from fastapi import Depends, HTTPException, status
from sqlalchemy.orm import Session
import redis
from functools import lru_cache

# --- DATABASE CONNECTION ---
class DatabaseManager:
    def __init__(self, connection_string: str):
        from sqlalchemy import create_engine
        from sqlalchemy.orm import sessionmaker
        
        self.engine = create_engine(
            connection_string,
            pool_pre_ping=True,  # Verificar conexiones antes de usar
            pool_size=10,
            max_overflow=20
        )
        self.SessionLocal = sessionmaker(
            autocommit=False,
            autoflush=False,
            bind=self.engine
        )
    
    def get_session(self) -> Generator[Session, None, None]:
        db = self.SessionLocal()
        try:
            yield db
        finally:
            db.close()

@lru_cache()
def get_db_manager() -> DatabaseManager:
    """Singleton de DatabaseManager"""
    from app.config import settings
    return DatabaseManager(settings.DATABASE_URL)

def get_db() -> Generator[Session, None, None]:
    """Dependencia para obtener sesión de DB"""
    manager = get_db_manager()
    yield from manager.get_session()

# --- REDIS CACHE ---
class CacheManager:
    def __init__(self):
        from app.config import settings
        self.redis_client = redis.Redis(
            host=settings.REDIS_HOST,
            port=settings.REDIS_PORT,
            db=0,
            decode_responses=True
        )
    
    def get(self, key: str):
        return self.redis_client.get(key)
    
    def set(self, key: str, value: str, ex: int = 3600):
        """ex = expiration time en segundos"""
        return self.redis_client.set(key, value, ex=ex)
    
    def delete(self, key: str):
        return self.redis_client.delete(key)

@lru_cache()
def get_cache_manager() -> CacheManager:
    return CacheManager()

# --- MODELO CIENTÍFICO SINGLETON ---
class ModeloCientificoComplejo:
    """Modelo pesado que se carga una vez"""
    
    def __init__(self):
        import time
        print("Cargando modelo ML pesado...")
        time.sleep(2)  # Simula carga
        
        # Cargar modelo de ML, calibraciones, etc.
        self.factor_calibracion = 1.05
        self.version = "2.1.0"
        self.coeficientes = np.random.rand(1000)  # Ejemplo
        
        print(f"Modelo v{self.version} cargado exitosamente")
    
    def predecir(self, features: np.ndarray) -> float:
        """Predicción con el modelo"""
        return float(np.dot(self.coeficientes[:len(features)], features))

_modelo_singleton = None

def get_modelo_cientifico() -> ModeloCientificoComplejo:
    """Dependency que retorna el singleton del modelo"""
    global _modelo_singleton
    if _modelo_singleton is None:
        _modelo_singleton = ModeloCientificoComplejo()
    return _modelo_singleton

# --- RATE LIMITING ---
from datetime import datetime, timedelta
from collections import defaultdict

class RateLimiter:
    """Rate limiter simple en memoria"""
    
    def __init__(self, max_requests: int = 100, window_seconds: int = 60):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = defaultdict(list)
    
    def check_limit(self, client_id: str) -> bool:
        """Retorna True si el cliente puede hacer request"""
        now = datetime.now()
        cutoff = now - timedelta(seconds=self.window_seconds)
        
        # Limpiar requests antiguos
        self.requests[client_id] = [
            req_time for req_time in self.requests[client_id]
            if req_time > cutoff
        ]
        
        # Verificar límite
        if len(self.requests[client_id]) >= self.max_requests:
            return False
        
        # Agregar nuevo request
        self.requests[client_id].append(now)
        return True

_rate_limiter = RateLimiter(max_requests=100, window_seconds=60)

async def check_rate_limit(
    client_id: str = Depends(lambda: "default")  # En prod: obtener de API key
):
    """Dependency para verificar rate limit"""
    if not _rate_limiter.check_limit(client_id):
        raise HTTPException(
            status_code=status.HTTP_429_TOO_MANY_REQUESTS,
            detail="Límite de requests excedido. Intente en 1 minuto."
        )

# --- COMPOSICIÓN DE DEPENDENCIES ---
# Tipo annotations para dependencies comunes
DatabaseDep = Annotated[Session, Depends(get_db)]
CacheDep = Annotated[CacheManager, Depends(get_cache_manager)]
ModeloDep = Annotated[ModeloCientificoComplejo, Depends(get_modelo_cientifico)]
```

**Uso en Endpoints:**

```python
# api/v1/endpoints/cinematica.py
from fastapi import APIRouter, Depends
from app.dependencies import DatabaseDep, CacheDep, ModeloDep, check_rate_limit
from app.models.schemas import InputCinematicaAvanzada, OutputCinematicaAvanzada

router = APIRouter()

@router.post(
    "/energia/avanzada",
    response_model=OutputCinematicaAvanzada,
    dependencies=[Depends(check_rate_limit)]  # Rate limiting global
)
async def calcular_energia_avanzada(
    datos: InputCinematicaAvanzada,
    db: DatabaseDep,  # Sesión de base de datos
    cache: CacheDep,  # Cliente de Redis
    modelo: ModeloDep  # Modelo ML
):
    """
    Cálculo avanzado con:
    - Efectos relativistas
    - Predicción ML
    - Caching
    - Persistencia en DB
    """
    # 1. Verificar cache
    cache_key = f"energia:{datos.tipo_particula}:{datos.velocidad}"
    cached = cache.get(cache_key)
    if cached:
        return OutputCinematicaAvanzada.parse_raw(cached)
    
    # 2. Cálculos físicos
    masa = datos.masa
    gamma = datos.gamma
    velocidad = datos.velocidad
    
    # Energía cinética: Ek = (γ - 1)mc²
    energia_cinetica = (gamma - 1) * masa * ConstantesFisicas.C**2
    
    # Energía total: E = γmc²
    energia_total = gamma * masa * ConstantesFisicas.C**2
    
    # Momento relativista: p = γmv
    momento = gamma * masa * velocidad
    
    # Longitud de onda de De Broglie: λ = h/p
    longitud_onda = None
    if momento != 0:
        longitud_onda = ConstantesFisicas.H / abs(momento)
    
    # Radio de ciclotrón si hay campo magnético
    radio_ciclotron = None
    if datos.campo_magnetico is not None and datos.campo_magnetico != 0:
        # r = mv/(qB) para partículas cargadas
        if datos.tipo_particula in [TipoParticula.ELECTRON, TipoParticula.PROTON]:
            q = 1.602176634e-19  # Carga elemental
            radio_ciclotron = (masa * velocidad) / (q * datos.campo_magnetico)
    
    # 3. Predicción ML (ejemplo: factor de corrección)
    features = np.array([velocidad, masa, gamma])
    correccion_ml = modelo.predecir(features)
    energia_cinetica *= (1 + correccion_ml * 0.01)  # Ajuste pequeño
    
    # 4. Construir respuesta
    resultado = OutputCinematicaAvanzada(
        energia_cinetica=energia_cinetica,
        energia_total=energia_total,
        momento=momento,
        longitud_de_onda=longitud_onda,
        radio_ciclotrón=radio_ciclotron,
        metadata={
            "regimen": "relativista" if gamma > 1.1 else "no-relativista",
            "gamma": gamma,
            "modelo_version": modelo.version,
            "correccion_ml": correccion_ml,
            "timestamp": datetime.utcnow().isoformat()
        }
    )
    
    # 5. Guardar en cache (1 hora)
    cache.set(cache_key, resultado.json(), ex=3600)
    
    # 6. Guardar en DB (opcional)
    # db_record = CalculoEnergia(**resultado.dict())
    # db.add(db_record)
    # db.commit()
    
    return resultado
```

### 2.3 Background Tasks y Celery

**Para Tareas Ligeras: Background Tasks de FastAPI**

```python
from fastapi import BackgroundTasks
import logging

logger = logging.getLogger(__name__)

def guardar_resultado_async(job_id: str, resultado: dict):
    """Tarea que se ejecuta en background"""
    logger.info(f"Guardando resultado de job {job_id}")
    # Guardar en DB, enviar email, etc.
    time.sleep(2)
    logger.info(f"Job {job_id} guardado exitosamente")

@router.post("/simulacion/rapida")
async def simulacion_rapida(
    parametros: dict,
    background_tasks: BackgroundTasks
):
    # Procesamiento rápido
    resultado = {"status": "completado"}
    
    # Agregar tarea post-procesamiento
    background_tasks.add_task(guardar_resultado_async, "job-123", resultado)
    
    return {"message": "Procesado", "data": resultado}
```

**Para Tareas Pesadas: Celery con Redis**

```python
# celery_app.py
from celery import Celery
from app.config import settings

celery_app = Celery(
    "fisica_tasks",
    broker=settings.CELERY_BROKER_URL,  # redis://localhost:6379/0
    backend=settings.CELERY_RESULT_BACKEND
)

celery_app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='UTC',
    enable_utc=True,
    task_track_started=True,
    task_time_limit=3600,  # 1 hora máximo
    worker_prefetch_multiplier=1,
    worker_max_tasks_per_child=1000,
)

# tasks.py
from celery_app import celery_app
import numpy as np
import time

@celery_app.task(bind=True, name="simulacion.monte_carlo")
def simulacion_monte_carlo(self, n_iterations: int, parametros: dict):
    """
    Simulación Monte Carlo pesada
    self.update_state permite reportar progreso
    """
    resultados = []
    
    for i in range(n_iterations):
        # Simulación pesada
        resultado = np.random.rand() * parametros.get('factor', 1.0)
        resultados.append(resultado)
        
        # Reportar progreso cada 10%
        if i % (n_iterations // 10) == 0:
            self.update_state(
                state='PROGRESS',
                meta={
                    'current': i,
                    'total': n_iterations,
                    'percent': int((i / n_iterations) * 100)
                }
            )
        
        time.sleep(0.001)  # Simula trabajo
    
    return {
        'mean': float(np.mean(resultados)),
        'std': float(np.std(resultados)),
        'iterations': n_iterations
    }

# endpoint usando Celery
from celery.result import AsyncResult

@router.post("/simulacion/monte-carlo", status_code=202)
async def iniciar_monte_carlo(n_iterations: int = 10000):
    """Iniciar simulación asíncrona con Celery"""
    task = simulacion_monte_carlo.delay(n_iterations, {"factor": 2.0})
    return {
        "task_id": task.id,
        "status": "queued",
        "check_url": f"/simulacion/status/{task.id}"
    }

@router.get("/simulacion/status/{task_id}")
async def check_status(task_id: str):
    """Verificar estado de la tarea"""
    task_result = AsyncResult(task_id, app=celery_app)
    
    if task_result.state == 'PENDING':
        response = {
            'state': task_result.state,
            'status': 'Esperando en cola...'
        }
    elif task_result.state == 'PROGRESS':
        response = {
            'state': task_result.state,
            'status': 'Procesando...',
            'progress': task_result.info
        }
    elif task_result.state == 'SUCCESS':
        response = {
            'state': task_result.state,
            'status': 'Completado',
            'result': task_result.result
        }
    else:  # FAILURE, RETRY, etc.
        response = {
            'state': task_result.state,
            'status': str(task_result.info)
        }
    
    return response
```

**docker-compose.yml con Celery:**

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8443:443"
    depends_on:
      - redis
      - postgres
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/fisica
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0

  celery_worker:
    build: .
    command: celery -A celery_app worker --loglevel=info --concurrency=4
    depends_on:
      - redis
      - postgres
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/fisica
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0

  celery_beat:  # Para tareas programadas
    build: .
    command: celery -A celery_app beat --loglevel=info
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=fisica
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 2.4 Streaming y Server-Sent Events (SSE)

Para simulaciones que necesitan reportar progreso en tiempo real:

```python
from fastapi.responses import StreamingResponse
import asyncio
import json

@router.get("/simulacion/stream/{task_id}")
async def stream_progress(task_id: str):
    """Stream de progreso en tiempo real usando SSE"""
    
    async def event_generator():
        """Generador de eventos"""
        task_result = AsyncResult(task_id, app=celery_app)
        
        while True:
            if task_result.state == 'PENDING':
                data = {'status': 'waiting', 'progress': 0}
            elif task_result.state == 'PROGRESS':
                data = {'status': 'running', **task_result.info}
            elif task_result.state == 'SUCCESS':
                data = {'status': 'completed', 'result': task_result.result}
                yield f"data: {json.dumps(data)}\n\n"
                break  # Terminar stream
            elif task_result.state == 'FAILURE':
                data = {'status': 'failed', 'error': str(task_result.info)}
                yield f"data: {json.dumps(data)}\n\n"
                break
            else:
                data = {'status': task_result.state}
            
            yield f"data: {json.dumps(data)}\n\n"
            await asyncio.sleep(1)  # Actualizar cada segundo
    
    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream"
    )
```

**Cliente JavaScript para SSE:**

```javascript
// Consumir el stream desde el navegador
const eventSource = new EventSource('/simulacion/stream/task-123');

eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Progreso:', data);
    
    if (data.status === 'completed' || data.status === 'failed') {
        eventSource.close();
    }
    
    // Actualizar UI
    document.getElementById('progress').innerText = 
        `${data.progress?.percent || 0}%`;
};

eventSource.onerror = (error) => {
    console.error('Error en stream:', error);
    eventSource.close();
};
```

---

## 3. Seguridad en Profundidad

### 3.1 Múltiples Estrategias de Autenticación

```python
# core/security.py
from fastapi import Security, HTTPException, status
from fastapi.security import (
    APIKeyHeader,
    HTTPBearer,
    HTTPAuthorizationCredentials
)
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
from typing import Optional
import secrets

# --- CONFIGURACIÓN ---
SECRET_KEY = "your-secret-key-change-in-production"  # Desde env var
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# --- API KEY AUTH ---
api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)

VALID_API_KEYS = {
    "fisica-2025-dev": {"client": "desarrollo", "tier": "free"},
    "fisica-2025-prod": {"client": "produccion", "tier": "premium"}
}

async def get_api_key(api_key: str = Security(api_key_header)):
    """Validar API Key"""
    if api_key not in VALID_API_KEYS:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="API Key inválida o expirada"
        )
    return VALID_API_KEYS[api_key]

# --- JWT TOKEN AUTH ---
security_bearer = HTTPBearer()

class TokenData:
    def __init__(self, username: str, scopes: list[str] = None):
        self.username = username
        self.scopes = scopes or []

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    """Crear JWT token"""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Security(security_bearer)
):
    """Validar JWT token y extraer usuario"""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="No se pudo validar las credenciales",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    try:
        token = credentials.credentials
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        scopes: list = payload.get("scopes", [])
        
        if username is None:
            raise credentials_exception
        
        token_data = TokenData(username=username, scopes=scopes)
        
    except JWTError:
        raise credentials_exception
    
    # Aquí irías a DB a buscar el usuario
    # user = get_user_from_db(username=token_data.username)
    # if user is None:
    #     raise credentials_exception
    
    return token_data

# --- OAUTH2 SCOPES (Permisos granulares) ---
from fastapi.security import OAuth2PasswordBearer, SecurityScopes

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={
        "read:simulations": "Leer simulaciones",
        "write:simulations": "Crear simulaciones",
        "admin": "Acceso administrativo completo"
    }
)

async def get_current_active_user(
    security_scopes: SecurityScopes,
    token: str = Depends(oauth2_scheme)
):
    """Validar token y verificar scopes"""
    if security_scopes.scopes:
        authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
    else:
        authenticate_value = "Bearer"
    
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="No se pudo validar las credenciales",
        headers={"WWW-Authenticate": authenticate_value},
    )
    
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        token_scopes = payload.get("scopes", [])
        
        if username is None:
            raise credentials_exception
        
        token_data = TokenData(username=username, scopes=token_scopes)
        
    except JWTError:
        raise credentials_exception
    
    # Verificar que el token tiene los scopes necesarios
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Permisos insuficientes",
                headers={"WWW-Authenticate": authenticate_value},
            )
    
    return token_data

# --- PASSWORD HASHING ---
def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Verificar password contra hash"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """Hashear password"""
    return pwd_context.hash(password)
```

**Endpoints de Autenticación:**

```python
# api/v1/endpoints/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from pydantic import BaseModel
from datetime import timedelta

router = APIRouter(tags=["autenticación"])

class Token(BaseModel):
    access_token: str
    token_type: str
    expires_in: int

class User(BaseModel):
    username: str
    email: str
    disabled: bool = False
    scopes: list[str] = []

# Simulación de DB de usuarios
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "email": "john@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",  # "secret"
        "disabled": False,
        "scopes": ["read:simulations", "write:simulations"]
    },
    "admin": {
        "username": "admin",
        "email": "admin@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",  # "secret"
        "disabled": False,
        "scopes": ["admin"]
    }
}

def authenticate_user(username: str, password: str):
    """Autenticar usuario contra DB"""
    user = fake_users_db.get(username)
    if not user:
        return False
    if not verify_password(password, user["hashed_password"]):
        return False
    return user

@router.post("/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    """Endpoint de login - retorna JWT token"""
    user = authenticate_user(form_data.username, form_data.password)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Usuario o password incorrectos",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    # Crear token con scopes del usuario
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={
            "sub": user["username"],
            "scopes": form_data.scopes or user["scopes"]
        },
        expires_delta=access_token_expires
    )
    
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "expires_in": ACCESS_TOKEN_EXPIRE_MINUTES * 60
    }

@router.post("/register")
async def register_user(username: str, email: str, password: str):
    """Registrar nuevo usuario"""
    if username in fake_users_db:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Usuario ya existe"
        )
    
    hashed_password = get_password_hash(password)
    
    fake_users_db[username] = {
        "username": username,
        "email": email,
        "hashed_password": hashed_password,
        "disabled": False,
        "scopes": ["read:simulations"]  # Scopes por defecto
    }
    
    return {"message": "Usuario registrado exitosamente"}

@router.get("/users/me", response_model=User)
async def read_users_me(current_user: TokenData = Depends(get_current_user)):
    """Obtener información del usuario actual"""
    user = fake_users_db.get(current_user.username)
    return User(**user)
```

**Usar Scopes en Endpoints:**

```python
from fastapi import Security

@router.post(
    "/simulacion/create",
    dependencies=[Security(get_current_active_user, scopes=["write:simulations"])]
)
async def crear_simulacion(datos: dict):
    """Solo usuarios con scope 'write:simulations' pueden acceder"""
    return {"message": "Simulación creada"}

@router.delete(
    "/simulacion/{sim_id}",
    dependencies=[Security(get_current_active_user, scopes=["admin"])]
)
async def eliminar_simulacion(sim_id: str):
    """Solo administradores pueden eliminar"""
    return {"message": f"Simulación {sim_id} eliminada"}
```

### 3.2 Rate Limiting Avanzado con Slowapi

```bash
pip install slowapi
```

```python
# main.py
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

# En endpoints
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@router.post("/simulacion/rapida")
@limiter.limit("5/minute")  # 5 requests por minuto
async def simulacion_rapida(request: Request):
    return {"message": "OK"}

@router.post("/simulacion/pesada")
@limiter.limit("1/minute")  # Solo 1 request por minuto
async def simulacion_pesada(request: Request):
    return {"message": "OK"}

# Rate limits por API tier
def get_api_key_limit(request: Request):
    """Límite dinámico basado en API key"""
    api_key = request.headers.get("X-API-Key")
    tier = VALID_API_KEYS.get(api_key, {}).get("tier", "free")
    
    limits = {
        "free": "10/minute",
        "premium": "100/minute",
        "enterprise": "1000/minute"
    }
    return limits.get(tier, "10/minute")

@router.post("/data/process")
@limiter.limit(get_api_key_limit)
async def process_data(request: Request):
    return {"message": "Procesado"}
```

### 3.3 CORS Configuración Segura

```python
# main.py
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Configuración de CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://physics-lab.com",
        "https://app.physics-lab.com"
    ],  # En dev: ["*"]
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],  # Métodos específicos
    allow_headers=["*"],
    expose_headers=["X-Total-Count", "X-Page-Size"],
    max_age=3600,  # Cache preflight por 1 hora
)
```

### 3.4 Input Sanitization y Prevención de Ataques

```python
from pydantic import validator, BaseModel
import re
import bleach

class UserInput(BaseModel):
    nombre: str
    descripcion: str
    
    @validator('nombre')
    def sanitize_nombre(cls, v):
        """Prevenir SQL injection y XSS"""
        # Remover caracteres peligrosos
        v = re.sub(r'[^\w\s-]', '', v)
        # Limitar longitud
        if len(v) > 100:
            raise ValueError('Nombre demasiado largo')
        return v.strip()
    
    @validator('descripcion')
    def sanitize_descripcion(cls, v):
        """Limpiar HTML potencialmente malicioso"""
        # Permitir solo tags seguros
        allowed_tags = ['p', 'br', 'strong', 'em']
        cleaned = bleach.clean(v, tags=allowed_tags, strip=True)
        return cleaned

# Path parameters seguros
from fastapi import Path

@router.get("/archivo/{filename}")
async def obtener_archivo(
    filename: str = Path(
        ...,
        regex="^[a-zA-Z0-9_-]+\.(txt|json|csv)$"  # Solo ciertos nombres y extensiones
    )
):
    """Prevenir path traversal attacks"""
    import os
    from pathlib import Path as PathLib
    
    # Validar que no haya path traversal
    safe_path = PathLib(f"/data/{filename}").resolve()
    if not str(safe_path).startswith("/data/"):
        raise HTTPException(400, "Ruta inválida")
    
    if not safe_path.exists():
        raise HTTPException(404, "Archivo no encontrado")
    
    return {"path": str(safe_path)}
```

---

## 4. Infraestructura Docker Avanzada

### 4.1 Multi-stage Build Optimizado

```dockerfile
# ====================
# STAGE 1: Builder
# ====================
FROM python:3.11-slim as builder

# Variables de build
ARG BUILD_DATE
ARG VCS_REF
ARG VERSION

LABEL org.opencontainers.image.created=$BUILD_DATE \
      org.opencontainers.image.revision=$VCS_REF \
      org.opencontainers.image.version=$VERSION \
      org.opencontainers.image.title="Physics Lab API" \
      org.opencontainers.image.description="Scientific simulation API" \
      org.opencontainers.image.authors="fisica@lab.com"

# Instalar dependencias del sistema necesarias para compilar
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    g++ \
    gfortran \
    libopenblas-dev \
    liblapack-dev \
    && rm -rf /var/lib/apt/lists/*

# Crear usuario no-root
RUN useradd -m -u 1000 appuser

WORKDIR /build

# Copiar solo requirements primero (cache layer)
COPY requirements.txt .

# Instalar dependencias en ubicación custom
RUN pip install --user --no-cache-dir --no-warn-script-location \
    -r requirements.txt

# ====================
# STAGE 2: Runner (imagen final)
# ====================
FROM python:3.11-slim

# Instalar solo runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libopenblas0 \
    libgomp1 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Crear usuario no-root
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copiar librerías Python del builder
COPY --from=builder --chown=appuser:appuser /root/.local /home/appuser/.local

# Copiar código de la aplicación
COPY --chown=appuser:appuser . .

# Asegurar que scripts estén en PATH
ENV PATH=/home/appuser/.local/bin:$PATH

# Cambiar a usuario no-root
USER appuser

# Healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f -k https://localhost:443/health || exit 1

# Exponer puerto
EXPOSE 443

# Comando por defecto
CMD ["uvicorn", "app.main:app", \
     "--host", "0.0.0.0", \
     "--port", "443", \
     "--ssl-keyfile", "/app/certs/key.pem", \
     "--ssl-certfile", "/app/certs/cert.pem", \
     "--workers", "4", \
     "--log-level", "info"]
```

### 4.2 Docker Compose para Desarrollo y Producción

**docker-compose.dev.yml:**

```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        BUILD_DATE: ${BUILD_DATE}
        VCS_REF: ${VCS_REF:-dev}
        VERSION: ${VERSION:-dev}
    ports:
      - "8443:443"
      - "5678:5678"  # Puerto para debugger
    volumes:
      - ./app:/app/app:ro  # Código en modo read-only
      - ./certs:/app/certs:ro
      - ./logs:/app/logs
    environment:
      - ENV=development
      - DEBUG=true
      - DATABASE_URL=postgresql://fisica:password@postgres:5432/fisica_dev
      - REDIS_URL=redis://redis:6379/0
      - LOG_LEVEL=DEBUG
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    command: >
      uvicorn app.main:app
      --host 0.0.0.0
      --port 443
      --ssl-keyfile /app/certs/key.pem
      --ssl-certfile /app/certs/cert.pem
      --reload
      --log-level debug

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: fisica
      POSTGRES_PASSWORD: password
      POSTGRES_DB: fisica_dev
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U fisica"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_dev_data:/data
    command: redis-server --appendonly yes

  celery_worker:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./app:/app/app:ro
    environment:
      - ENV=development
      - DATABASE_URL=postgresql://fisica:password@postgres:5432/fisica_dev
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - redis
      - postgres
    command: celery -A app.celery_app worker --loglevel=debug --concurrency=2

  flower:  # Celery monitoring UI
    image: mher/flower:0.9.7
    ports:
      - "5555:5555"
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - FLOWER_PORT=5555
    depends_on:
      - redis

  adminer:  # DB admin interface
    image: adminer:latest
    ports:
      - "8080:8080"
    depends_on:
      - postgres

volumes:
  postgres_dev_data:
  redis_dev_data:
```

**docker-compose.prod.yml:**

```yaml
version: '3.8'

services:
  api:
    image: ${DOCKER_REGISTRY}/fisica-api:${VERSION}
    restart: always
    ports:
      - "443:443"
    volumes:
      - ./certs:/app/certs:ro
      - api_logs:/app/logs
    environment:
      - ENV=production
      - DEBUG=false
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - LOG_LEVEL=INFO
      - SENTRY_DSN=${SENTRY_DSN}
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
    healthcheck:
      test: ["CMD", "curl", "-f", "-k", "https://localhost:443/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  celery_worker:
    image: ${DOCKER_REGISTRY}/fisica-api:${VERSION}
    restart: always
    command: celery -A app.celery_app worker --loglevel=info --concurrency=8
    environment:
      - ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  nginx:  # Reverse proxy con load balancing
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - api
    deploy:
      replicas: 2

volumes:
  api_logs:
```

### 4.3 Optimización de Capas Docker

```dockerfile
# ❌ MAL: Crea muchas capas
RUN pip install numpy
RUN pip install scipy
RUN pip install pandas
RUN pip install scikit-learn

# ✅ BIEN: Una sola capa
RUN pip install --no-cache-dir \
    numpy==1.24.0 \
    scipy==1.10.0 \
    pandas==2.0.0 \
    scikit-learn==1.2.0

# ❌ MAL: Invalida cache fácilmente
COPY . /app

# ✅ BIEN: Copiar dependencias primero (se cachean)
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY . /app

# ❌ MAL: Imagen grande
FROM python:3.11

# ✅ BIEN: Imagen slim
FROM python:3.11-slim

# ✅ MEJOR: Alpine (más pequeña)
FROM python:3.11-alpine
# Nota: Alpine puede tener problemas con algunas libs científicas

# Reducir tamaño eliminando archivos innecesarios
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
    /tmp/* \
    /var/tmp/* \
    /usr/share/doc/* \
    /usr/share/man/*

# Usar .dockerignore
# Crear archivo .dockerignore:
# __pycache__
# *.pyc
# *.pyo
# *.pyd
# .git
# .github
# tests/
# docs/
# *.md
# .env
```

### 4.4 Security Scanning en Build

```bash
# Build con security scan
docker build -t fisica-api:latest .

# Scan con Trivy
trivy image fisica-api:latest

# Scan con Snyk
snyk container test fisica-api:latest

# Scan con Grype
grype fisica-api:latest
```

---

## 5. Testing Completo

### 5.1 Estructura de Testing

```
tests/
├── __init__.py
├── conftest.py              # Fixtures compartidos
├── unit/
│   ├── __init__.py
│   ├── test_models.py       # Tests de Pydantic models
│   ├── test_services.py     # Tests de lógica de negocio
│   └── test_utils.py        # Tests de utilidades
├── integration/
│   ├── __init__.py
│   ├── test_api_endpoints.py
│   ├── test_database.py
│   └── test_celery_tasks.py
├── e2e/
│   ├── __init__.py
│   └── test_user_workflows.py
└── performance/
    ├── __init__.py
    └── test_load.py
```

### 5.2 Unit Tests con Pytest

**conftest.py:**

```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.dependencies import get_db
from app.core.security import get_current_user, TokenData

# Database de testing (en memoria)
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False}
)
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Override dependencies
def override_get_db():
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()

def override_get_current_user():
    return TokenData(username="testuser", scopes=["read:simulations", "write:simulations"])

app.dependency_overrides[get_db] = override_get_db
app.dependency_overrides[get_current_user] = override_get_current_user

@pytest.fixture
def client():
    """Cliente de testing"""
    return TestClient(app)

@pytest.fixture
def auth_headers():
    """Headers con autenticación"""
    return {"X-API-Key": "fisica-2025-test"}

@pytest.fixture
def sample_cinematica_data():
    """Datos de ejemplo válidos"""
    return {
        "tipo_particula": "electron",
        "velocidad": 1000000,
        "campo_magnetico": 0.5
    }
```

**tests/unit/test_models.py:**

```python
import pytest
from pydantic import ValidationError
from app.models.schemas import InputCinematicaAvanzada, TipoParticula

def test_validacion_velocidad_subluminal():
    """Validar que velocidad > c lanza error"""
    with pytest.raises(ValidationError) as exc_info:
        InputCinematicaAvanzada(
            tipo_particula=TipoParticula.ELECTRON,
            velocidad=300000000  # Mayor que c
        )
    
    assert "velocidad de la luz" in str(exc_info.value).lower()

def test_calculo_automatico_masa():
    """Verificar que la masa se asigna automáticamente"""
    datos = InputCinematicaAvanzada(
        tipo_particula=TipoParticula.ELECTRON,
        velocidad=1000000
    )
    
    assert datos.masa == 9.1093837015e-31

def test_factor_lorentz_bajo_regimen_no_relativista():
    """Γ ≈ 1 para v << c"""
    datos = InputCinematicaAvanzada(
        tipo_particula=TipoParticula.PROTON,
        velocidad=100  # Muy lento
    )
    
    assert pytest.approx(datos.gamma, abs=0.001) == 1.0

def test_factor_lorentz_regimen_relativista():
    """Γ > 1 para v cercana a c"""
    c = 299792458
    datos = InputCinematicaAvanzada(
        tipo_particula=TipoParticula.ELECTRON,
        velocidad=0.9 * c  # 90% de c
    )
    
    assert datos.gamma > 2.0

def test_foton_debe_viajar_a_c():
    """Los fotones solo pueden viajar a c"""
    c = 299792458
    
    # Debe funcionar
    datos = InputCinematicaAvanzada(
        tipo_particula=TipoParticula.PHOTON,
        velocidad=c
    )
    assert datos.velocidad == c
    
    # Debe fallar
    with pytest.raises(ValidationError):
        InputCinematicaAvanzada(
            tipo_particula=TipoParticula.PHOTON,
            velocidad=0.5 * c
        )

@pytest.mark.parametrize("velocidad,esperado_regimen", [
    (100, "no-relativista"),
    (1000000, "no-relativista"),
    (0.3 * 299792458, "relativista"),
    (0.9 * 299792458, "relativista"),
])
def test_clasificacion_regimen(velocidad, esperado_regimen):
    """Test parametrizado para múltiples velocidades"""
    datos = InputCinematicaAvanzada(
        tipo_particula=TipoParticula.ELECTRON,
        velocidad=velocidad
    )
    
    regimen = "relativista" if datos.gamma > 1.1 else "no-relativista"
    assert regimen == esperado_regimen
```

**tests/integration/test_api_endpoints.py:**

```python
import pytest
from fastapi import status

def test_calcular_energia_success(client, auth_headers, sample_cinematica_data):
    """Test de endpoint exitoso"""
    response = client.post(
        "/v1/cinematica/energia/avanzada",
        json=sample_cinematica_data,
        headers=auth_headers
    )
    
    assert response.status_code == status.HTTP_200_OK
    data = response.json()
    
    # Verificar estructura de respuesta
    assert "energia_cinetica" in data
    assert "energia_total" in data
    assert "momento" in data
    assert "metadata" in data
    
    # Verificar valores físicos razonables
    assert data["energia_cinetica"] > 0
    assert data["energia_total"] > data["energia_cinetica"]

def test_sin_autenticacion_retorna_403(client, sample_cinematica_data):
    """Test sin API key"""
    response = client.post(
        "/v1/cinematica/energia/avanzada",
        json=sample_cinematica_data
    )
    
    assert response.status_code == status.HTTP_403_FORBIDDEN

def test_velocidad_supraluminal_retorna_422(client, auth_headers):
    """Test con velocidad inválida"""
    invalid_data = {
        "tipo_particula": "electron",
        "velocidad": 400000000  # > c
    }
    
    response = client.post(
        "/v1/cinematica/energia/avanzada",
        json=invalid_data,
        headers=auth_headers
    )
    
    assert response.status_code == status.HTTP_422_UNPROCESSABLE_ENTITY
    assert "velocidad de la luz" in response.text.lower()

def test_cache_funciona(client, auth_headers, sample_cinematica_data, mocker):
    """Verificar que el cache evita recalcular"""
    # Mock del servicio de cálculo
    mock_calculo = mocker.patch("app.services.calculo.calcular_energia_detallada")
    mock_calculo.return_value = {"energia": 100}
    
    # Primera llamada
    response1 = client.post(
        "/v1/cinematica/energia/avanzada",
        json=sample_cinematica_data,
        headers=auth_headers
    )
    
    # Segunda llamada idéntica
    response2 = client.post(
        "/v1/cinematica/energia/avanzada",
        json=sample_cinematica_data,
        headers=auth_headers
    )
    
    # La función debe llamarse solo 1 vez (cache en la segunda)
    assert mock_calculo.call_count == 1
    assert response1.json() == response2.json()

@pytest.mark.asyncio
async def test_simulacion_asincrona(client, auth_headers):
    """Test de tarea asíncrona"""
    # Iniciar simulación
    response = client.post(
        "/v1/simulacion/monte-carlo?n_iterations=100",
        headers=auth_headers
    )
    
    assert response.status_code == status.HTTP_202_ACCEPTED
    data = response.json()
    assert "task_id" in data
    
    task_id = data["task_id"]
    
    # Verificar estado
    status_response = client.get(
        f"/v1/simulacion/status/{task_id}",
        headers=auth_headers
    )
    
    assert status_response.status_code == status.HTTP_200_OK
    status_data = status_response.json()
    assert status_data["state"] in ["PENDING", "PROGRESS", "SUCCESS"]
```

### 5.3 Coverage y Reporting

```bash
# Instalar coverage
pip install pytest-cov

# Ejecutar tests con coverage
pytest --cov=app --cov-report=html --cov-report=term

# Ver reporte en navegador
open htmlcov/index.html

# Generar reporte XML para CI
pytest --cov=app --cov-report=xml

# Fallar si coverage < 80%
pytest --cov=app --cov-fail-under=80
```

**pytest.ini:**

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts =
    --strict-markers
    --cov=app
    --cov-report=html
    --cov-report=term-missing
    --cov-branch
    --maxfail=1
    -v
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: marks tests as integration tests
    unit: marks tests as unit tests
    e2e: marks tests as end-to-end tests
```

### 5.4 Performance Testing con Locust

```python
# tests/performance/locustfile.py
from locust import HttpUser, task, between
import json

class FisicaAPIUser(HttpUser):
    wait_time = between(1, 3)  # Espera entre 1-3 segundos entre requests
    host = "https://localhost:8443"
    
    def on_start(self):
        """Ejecutado una vez al inicio por usuario virtual"""
        self.headers = {
            "X-API-Key": "fisica-2025-test",
            "Content-Type": "application/json"
        }
    
    @task(3)  # Peso 3: se ejecuta 3 veces más que otras tasks
    def calcular_energia(self):
        """Simular cálculo de energía"""
        payload = {
            "tipo_particula": "electron",
            "velocidad": 1000000,
            "campo_magnetico": 0.5
        }
        
        with self.client.post(
            "/v1/cinematica/energia/avanzada",
            json=payload,
            headers=self.headers,
            verify=False,  # Para SSL self-signed en testing
            catch_response=True
        ) as response:
            if response.status_code != 200:
                response.failure(f"Error: {response.status_code}")
            elif response.elapsed.total_seconds() > 2:
                response.failure("Request tardó más de 2 segundos")
    
    @task(1)
    def iniciar_simulacion(self):
        """Simular inicio de simulación pesada"""
        with self.client.post(
            "/v1/simulacion/monte-carlo?n_iterations=1000",
            headers=self.headers,
            verify=False,
            catch_response=True
        ) as response:
            if response.status_code != 202:
                response.failure(f"Error: {response.status_code}")
            else:
                # Extraer task_id y verificar status
                task_id = response.json().get("task_id")
                self.verificar_status(task_id)
    
    def verificar_status(self, task_id):
        """Verificar estado de tarea"""
        with self.client.get(
            f"/v1/simulacion/status/{task_id}",
            headers=self.headers,
            verify=False,
            name="/v1/simulacion/status/[id]"  # Agrupar en stats
        ) as response:
            pass

# Ejecutar:
# locust -f tests/performance/locustfile.py
# Abrir http://localhost:8089
```

---

## 6. DevSecOps Pipeline Expandido

### 6.1 GitHub Actions Completo

**.github/workflows/ci-cd.yml:**

```yaml
name: CI/CD Pipeline Completo

on:
  push:
    branches: [main, develop]
    tags:
      - 'v*'
  pull_request:
    branches: [main]

env:
  PYTHON_VERSION: '3.11'
  DOCKER_REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ====================
  # JOB 1: LINTING Y FORMATEO
  # ====================
  lint:
    name: Lint y Format Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Cache pip
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
      
      - name: Install linting tools
        run: |
          pip install black flake8 isort mypy pylint
      
      - name: Check formatting with Black
        run: black --check app/ tests/
      
      - name: Check imports with isort
        run: isort --check-only app/ tests/
      
      - name: Lint with flake8
        run: |
          flake8 app/ tests/ \
            --max-line-length=100 \
            --ignore=E203,W503 \
            --exclude=.git,__pycache__,venv
      
      - name: Type check with mypy
        run: mypy app/ --ignore-missing-imports
      
      - name: Lint with pylint
        run: pylint app/ --disable=C0111,R0903

  # ====================
  # JOB 2: SECURITY SCANNING
  # ====================
  security:
    name: Security Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install bandit safety pip-audit
      
      # SAST: Análisis estático de código
      - name: Run Bandit (SAST)
        run: |
          bandit -r app/ \
            -ll \
            -f json \
            -o bandit-report.json
        continue-on-error: true
      
      - name: Upload Bandit results
        uses: actions/upload-artifact@v3
        with:
          name: bandit-report
          path: bandit-report.json
      
      # SCA: Análisis de dependencias
      - name: Run Safety Check (SCA)
        run: safety check --json > safety-report.json
        continue-on-error: true
      
      - name: Run pip-audit
        run: pip-audit --desc -f json > pip-audit-report.json
        continue-on-error: true
      
      - name: Upload dependency scan results
        uses: actions/upload-artifact@v3
        with:
          name: dependency-reports
          path: |
            safety-report.json
            pip-audit-report.json
      
      # Secret scanning
      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      # SAST avanzado con Semgrep
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/python
            p/jwt

  # ====================
  # JOB 3: TESTS
  # ====================
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: [lint]
    
    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Cache pip
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
      
      - name: Run unit tests with coverage
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379/0
        run: |
          pytest tests/unit/ \
            --cov=app \
            --cov-report=xml \
            --cov-report=html \
            --cov-report=term-missing \
            --junitxml=junit.xml \
            -v
      
      - name: Run integration tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379/0
        run: |
          pytest tests/integration/ \
            --cov=app \
            --cov-append \
            --cov-report=xml \
            -v
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.xml
          flags: unittests
          name: codecov-umbrella
      
      - name: Publish test results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: junit.xml

  # ====================
  # JOB 4: BUILD DOCKER
  # ====================
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [security, test]
    outputs:
      image_tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.DOCKER_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            BUILD_DATE=${{ github.event.head_commit.timestamp }}
            VCS_REF=${{ github.sha }}
            VERSION=${{ github.ref_name }}

  # ====================
  # JOB 5: CONTAINER SECURITY SCAN
  # ====================
  scan-image:
    name: Scan Docker Image
    runs-on: ubuntu-latest
    needs: [build]
    
    steps:
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ needs.build.outputs.image_tag }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Run Snyk Container scan
        uses: snyk/actions/docker@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          image: ${{ needs.build.outputs.image_tag }}
          args: --severity-threshold=high

  # ====================
  # JOB 6: DEPLOY TO STAGING
  # ====================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [scan-image]
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.api.fisica.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to staging server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /opt/fisica-api
            docker-compose -f docker-compose.staging.yml pull
            docker-compose -f docker-compose.staging.yml up -d
            docker-compose -f docker-compose.staging.yml exec -T api alembic upgrade head
      
      - name: Run smoke tests
        run: |
          curl -f https://staging.api.fisica.com/health || exit 1
          curl -f https://staging.api.fisica.com/docs || exit 1

  # ====================
  # JOB 7: DEPLOY TO PRODUCTION
  # ====================
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [scan-image]
    if: startsWith(github.ref, 'refs/tags/v')
    environment:
      name: production
      url: https://api.fisica.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to production with blue-green
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /opt/fisica-api
            
            # Blue-green deployment
            docker-compose -f docker-compose.prod.yml pull
            
            # Start green environment
            docker-compose -f docker-compose.prod.yml up -d --scale api=6
            
            # Wait for health checks
            sleep 30
            
            # Stop blue (old) containers
            docker-compose -f docker-compose.prod.yml up -d --scale api=3
      
      - name: Run production smoke tests
        run: |
          curl -f https://api.fisica.com/health || exit 1
      
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment to production completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()

  # ====================
  # JOB 8: PERFORMANCE TESTING
  # ====================
  performance:
    name: Performance Tests
    runs-on: ubuntu-latest
    needs: [deploy-staging]
    if: github.ref == 'refs/heads/develop'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install Locust
        run: pip install locust
      
      - name: Run load tests
        run: |
          locust -f tests/performance/locustfile.py \
            --host=https://staging.api.fisica.com \
            --users=100 \
            --spawn-rate=10 \
            --run-time=5m \
            --headless \
            --html=locust-report.html \
            --csv=locust-stats
      
      - name: Upload performance report
        uses: actions/upload-artifact@v3
        with:
          name: performance-report
          path: |
            locust-report.html
            locust-stats*.csv
```

### 6.2 Pre-commit Hooks

**.pre-commit-config.yaml:**

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ['--maxkb=1000']
      - id: check-json
      - id: check-merge-conflict
      - id: detect-private-key

  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort
        args: ["--profile", "black"]

  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
        args: ["--max-line-length=100", "--ignore=E203,W503"]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.3.0
    hooks:
      - id: mypy
        additional_dependencies: [types-all]

  - repo: https://github.com/pycqa/bandit
    rev: 1.7.5
    hooks:
      - id: bandit
        args: ["-ll", "-r", "app/"]

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets

# Instalar: pip install pre-commit
# Activar: pre-commit install
# Ejecutar manualmente: pre-commit run --all-files
```

---

## 7. Monitoreo y Observabilidad

### 7.1 Logging Estructurado

```python
# core/logging.py
import logging
import sys
from typing import Any
import json
from datetime import datetime
from pythonjsonlogger import jsonlogger

class CustomJsonFormatter(jsonlogger.JsonFormatter):
    """Formatter JSON custom con campos adicionales"""
    
    def add_fields(self, log_record: dict, record: logging.LogRecord, message_dict: dict):
        super().add_fields(log_record, record, message_dict)
        
        # Agregar timestamp ISO 8601
        log_record['timestamp'] = datetime.utcnow().isoformat()
        
        # Agregar nivel como string
        log_record['level'] = record.levelname
        
        # Agregar información del módulo
        log_record['module'] = record.module
        log_record['function'] = record.funcName
        log_record['line'] = record.lineno
        
        # Agregar trace_id si existe (para correlacionar requests)
        if hasattr(record, 'trace_id'):
            log_record['trace_id'] = record.trace_id

def setup_logging(log_level: str = "INFO", service_name: str = "fisica-api"):
    """Configurar logging para la aplicación"""
    
    # Crear handler para stdout
    handler = logging.StreamHandler(sys.stdout)
    
    # Usar formato JSON
    formatter = CustomJsonFormatter(
        '%(timestamp)s %(level)s %(name)s %(message)s'
    )
    handler.setFormatter(formatter)
    
    # Configurar logger root
    root_logger = logging.getLogger()
    root_logger.addHandler(handler)
    root_logger.setLevel(log_level)
    
    # Silenciar loggers ruidosos
    logging.getLogger("uvicorn.access").setLevel(logging.WARNING)
    logging.getLogger("fastapi").setLevel(logging.INFO)
    
    # Logger de la aplicación
    app_logger = logging.getLogger(service_name)
    app_logger.setLevel(log_level)
    
    return app_logger

# Uso en la aplicación
logger = setup_logging(service_name="fisica-api")
```

**Middleware para Request Logging:**

```python
# middleware/logging.py
import time
import uuid
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware
import logging

logger = logging.getLogger("fisica-api")

class RequestLoggingMiddleware(BaseHTTPMiddleware):
    """Middleware para logging de requests"""
    
    async def dispatch(self, request: Request, call_next):
        # Generar trace_id único
        trace_id = str(uuid.uuid4())
        request.state.trace_id = trace_id
        
        # Timestamp de inicio
        start_time = time.time()
        
        # Log del request
        logger.info(
            "Request started",
            extra={
                "trace_id": trace_id,
                "method": request.method,
                "url": str(request.url),
                "client_ip": request.client.host,
                "user_agent": request.headers.get("user-agent"),
            }
        )
        
        # Procesar request
        try:
            response = await call_next(request)
            duration = time.time() - start_time
            
            # Log del response
            logger.info(
                "Request completed",
                extra={
                    "trace_id": trace_id,
                    "method": request.method,
                    "url": str(request.url),
                    "status_code": response.status_code,
                    "duration_ms": int(duration * 1000),
                }
            )
            
            # Agregar headers de tracing
            response.headers["X-Trace-ID"] = trace_id
            response.headers["X-Process-Time"] = f"{duration:.3f}"
            
            return response
            
        except Exception as exc:
            duration = time.time() - start_time
            
            logger.error(
                "Request failed",
                extra={
                    "trace_id": trace_id,
                    "method": request.method,
                    "url": str(request.url),
                    "duration_ms": int(duration * 1000),
                    "error": str(exc),
                },
                exc_info=True
            )
            raise

# Agregar a la app
from app.middleware.logging import RequestLoggingMiddleware
app.add_middleware(RequestLoggingMiddleware)
```

### 7.2 Métricas con Prometheus

```bash
pip install prometheus-fastapi-instrumentator
```

```python
# main.py
from prometheus_fastapi_instrumentator import Instrumentator

app = FastAPI()

# Instrumentar la aplicación
Instrumentator().instrument(app).expose(app)

# Métricas personalizadas
from prometheus_client import Counter, Histogram, Gauge
import time

# Counter: cuenta eventos (solo sube)
simulaciones_totales = Counter(
    'simulaciones_totales',
    'Número total de simulaciones ejecutadas',
    ['tipo', 'estado']
)

# Histogram: distribución de valores
duracion_simulaciones = Histogram(
    'duracion_simulaciones_segundos',
    'Duración de las simulaciones',
    ['tipo'],
    buckets=[0.1, 0.5, 1, 2, 5, 10, 30, 60, 120]
)

# Gauge: valor que puede subir y bajar
simulaciones_activas = Gauge(
    'simulaciones_activas',
    'Número de simulaciones actualmente en ejecución'
)

# Uso en endpoint
@router.post("/simulacion/monte-carlo")
async def simulacion_monte_carlo(n_iter: int):
    simulaciones_activas.inc()  # Incrementar
    start = time.time()
    
    try:
        resultado = ejecutar_simulacion(n_iter)
        
        # Registrar éxito
        simulaciones_totales.labels(tipo='monte-carlo', estado='exitoso').inc()
        duracion_simulaciones.labels(tipo='monte-carlo').observe(time.time() - start)
        
        return resultado
        
    except Exception as e:
        # Registrar fallo
        simulaciones_totales.labels(tipo='monte-carlo', estado='fallido').inc()
        raise
    
    finally:
        simulaciones_activas.dec()  # Decrementar
```

**Dashboards Grafana (JSON):**

```json
{
  "dashboard": {
    "title": "Física API Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{handler}}"
          }
        ]
      },
      {
        "title": "Response Time (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ]
      },
      {
        "title": "Active Simulations",
        "targets": [
          {
            "expr": "simulaciones_activas"
          }
        ]
      }
    ]
  }
}
```

### 7.3 Tracing Distribuido con OpenTelemetry

```bash
pip install opentelemetry-api \
    opentelemetry-sdk \
    opentelemetry-instrumentation-fastapi \
    opentelemetry-instrumentation-sqlalchemy \
    opentelemetry-exporter-jaeger
```

```python
# core/tracing.py
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor

def setup_tracing(app, service_name: str = "fisica-api"):
    """Configurar OpenTelemetry tracing"""
    
    # Configurar provider
    provider = TracerProvider()
    trace.set_tracer_provider(provider)
    
    # Configurar exporter (Jaeger)
    jaeger_exporter = JaegerExporter(
        agent_host_name="localhost",
        agent_port=6831,
    )
    
    # Agregar processor
    provider.add_span_processor(BatchSpanProcessor(jaeger_exporter))
    
    # Instrumentar FastAPI
    FastAPIInstrumentor.instrument_app(app)
    
    # Instrumentar SQLAlchemy
    SQLAlchemyInstrumentor().instrument()
    
    return trace.get_tracer(service_name)

# En main.py
tracer = setup_tracing(app)

# Usar en funciones
@router.post("/simulacion/compleja")
async def simulacion_compleja(datos: dict):
    with tracer.start_as_current_span("simulacion_compleja") as span:
        span.set_attribute("tipo", datos.get("tipo"))
        span.set_attribute("n_particulas", datos.get("n"))
        
        # Sub-span para operación específica
        with tracer.start_as_current_span("preparar_datos"):
            datos_preparados = preparar(datos)
        
        with tracer.start_as_current_span("ejecutar_calculo"):
            resultado = calcular(datos_preparados)
        
        span.set_attribute("tiempo_total", resultado["tiempo"])
        return resultado
```

### 7.4 Health Checks Avanzados

```python
# api/v1/endpoints/health.py
from fastapi import APIRouter, status, Depends
from pydantic import BaseModel
from typing import Dict
import psutil
import redis
from sqlalchemy.orm import Session

router = APIRouter(tags=["health"])

class HealthResponse(BaseModel):
    status: str
    version: str
    checks: Dict[str, str]

class DetailedHealthResponse(HealthResponse):
    system: Dict[str, any]

@router.get("/health", response_model=HealthResponse)
async def health_check():
    """Health check básico para load balancers"""
    return {
        "status": "healthy",
        "version": "1.0.0",
        "checks": {
            "api": "ok"
        }
    }

@router.get("/health/ready", response_model=HealthResponse)
async def readiness_check(db: Session = Depends(get_db)):
    """Readiness check - verifica dependencias críticas"""
    checks = {}
    all_healthy = True
    
    # Check database
    try:
        db.execute("SELECT 1")
        checks["database"] = "ok"
    except Exception as e:
        checks["database"] = f"error: {str(e)}"
        all_healthy = False
    
    # Check Redis
    try:
        cache = get_cache_manager()
        cache.redis_client.ping()
        checks["redis"] = "ok"
    except Exception as e:
        checks["redis"] = f"error: {str(e)}"
        all_healthy = False
    
    # Check Celery workers
    try:
        from app.celery_app import celery_app
        stats = celery_app.control.inspect().stats()
        if stats:
            checks["celery"] = "ok"
        else:
            checks["celery"] = "no workers"
            all_healthy = False
    except Exception as e:
        checks["celery"] = f"error: {str(e)}"
        all_healthy = False
    
    status_code = status.HTTP_200_OK if all_healthy else status.HTTP_503_SERVICE_UNAVAILABLE
    
    return {
        "status": "healthy" if all_healthy else "unhealthy",
        "version": "1.0.0",
        "checks": checks
    }

@router.get("/health/live")
async def liveness_check():
    """Liveness check - solo verifica que el proceso esté vivo"""
    return {"status": "alive"}

@router.get("/health/detailed", response_model=DetailedHealthResponse)
async def detailed_health_check(db: Session = Depends(get_db)):
    """Health check detallado con métricas del sistema"""
    
    # Checks básicos
    checks = {}
    
    try:
        db.execute("SELECT 1")
        checks["database"] = "ok"
    except Exception as e:
        checks["database"] = f"error: {str(e)}"
    
    try:
        cache = get_cache_manager()
        cache.redis_client.ping()
        checks["redis"] = "ok"
    except Exception as e:
        checks["redis"] = f"error: {str(e)}"
    
    # Métricas del sistema
    system_info = {
        "cpu_percent": psutil.cpu_percent(interval=1),
        "memory": {
            "total_gb": round(psutil.virtual_memory().total / (1024**3), 2),
            "available_gb": round(psutil.virtual_memory().available / (1024**3), 2),
            "percent_used": psutil.virtual_memory().percent
        },
        "disk": {
            "total_gb": round(psutil.disk_usage('/').total / (1024**3), 2),
            "free_gb": round(psutil.disk_usage('/').free / (1024**3), 2),
            "percent_used": psutil.disk_usage('/').percent
        }
    }
    
    return {
        "status": "healthy",
        "version": "1.0.0",
        "checks": checks,
        "system": system_info
    }
```

---

## 8. Despliegue Continuo (CD)

### 8.1 Blue-Green Deployment

```bash
# deploy-blue-green.sh
#!/bin/bash

set -e

# Variables
NEW_VERSION=$1
CURRENT_COLOR=$(cat /tmp/current_color.txt)
NEW_COLOR=$([ "$CURRENT_COLOR" == "blue" ] && echo "green" || echo "blue")

echo "Current environment: $CURRENT_COLOR"
echo "Deploying to: $NEW_COLOR"

# Pull nueva imagen
docker pull ghcr.io/myorg/fisica-api:$NEW_VERSION

# Deploy a nuevo color
docker-compose -f docker-compose.$NEW_COLOR.yml up -d

# Esperar health checks
echo "Waiting for health checks..."
sleep 30

# Verificar salud del nuevo deployment
HEALTH_URL="https://api-$NEW_COLOR.fisica.com/health"
if curl -f -k $HEALTH_URL; then
    echo "Health check passed!"
    
    # Cambiar tráfico al nuevo color
    echo "Switching traffic to $NEW_COLOR"
    # Actualizar NGINX config o load balancer
    sed -i "s/api-$CURRENT_COLOR/api-$NEW_COLOR/g" /etc/nginx/sites-enabled/api.conf
    nginx -s reload
    
    # Guardar nuevo color actual
    echo $NEW_COLOR > /tmp/current_color.txt
    
    # Esperar drenaje de conexiones del color antiguo
    sleep 60
    
    # Detener color antiguo
    docker-compose -f docker-compose.$CURRENT_COLOR.yml down
    
    echo "Deployment successful!"
else
    echo "Health check failed! Rolling back..."
    docker-compose -f docker-compose.$NEW_COLOR.yml down
    exit 1
fi
```

### 8.2 Canary Deployment con Traefik

**docker-compose.canary.yml:**

```yaml
version: '3.8'

services:
  api-stable:
    image: ghcr.io/myorg/fisica-api:v1.0.0
    deploy:
      replicas: 9
      labels:
        - "traefik.enable=true"
        - "traefik.http.services.api-stable.loadbalancer.server.port=443"
        - "traefik.http.routers.api.rule=Host(`api.fisica.com`)"
        - "traefik.http.routers.api.service=api-weighted"
        - "traefik.http.services.api-weighted.weighted.services[0].name=api-stable"
        - "traefik.http.services.api-weighted.weighted.services[0].weight=90"

  api-canary:
    image: ghcr.io/myorg/fisica-api:v1.1.0-canary
    deploy:
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.http.services.api-canary.loadbalancer.server.port=443"
        - "traefik.http.routers.api.service=api-weighted"
        - "traefik.http.services.api-weighted.weighted.services[1].name=api-canary"
        - "traefik.http.services.api-weighted.weighted.services[1].weight=10"

  traefik:
    image: traefik:v2.10
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.swarmMode=true"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
```

### 8.3 Rollback Automático

```python
# scripts/auto-rollback.py
import requests
import time
import subprocess

def check_error_rate(api_url: str, threshold: float = 0.05) -> bool:
    """
    Verificar tasa de errores de la API
    Retorna True si la tasa de errores > threshold
    """
    # Query a Prometheus
    query = f'rate(http_requests_total{{status=~"5.."}}[5m]) / rate(http_requests_total[5m])'
    
    response = requests.get(
        'http://prometheus:9090/api/v1/query',
        params={'query': query}
    )
    
    data = response.json()
    
    if data['data']['result']:
        error_rate = float(data['data']['result'][0]['value'][1])
        print(f"Current error rate: {error_rate:.2%}")
        return error_rate > threshold
    
    return False

def rollback():
    """Ejecutar rollback al deployment anterior"""
    print("Initiating rollback...")
    subprocess.run([
        'kubectl', 'rollout', 'undo', 'deployment/fisica-api'
    ], check=True)
    print("Rollback completed")

def main():
    api_url = "https://api.fisica.com"
    check_interval = 60  # segundos
    max_checks = 10
    
    print(f"Monitoring deployment for {max_checks * check_interval} seconds...")
    
    for i in range(max_checks):
        if check_error_rate(api_url, threshold=0.05):
            print("ERROR: Error rate exceeded threshold!")
            rollback()
            return 1
        
        print(f"Check {i+1}/{max_checks} passed")
        time.sleep(check_interval)
    
    print("Deployment stable - monitoring complete")
    return 0

if __name__ == "__main__":
    exit(main())
```

---

## 9. Kubernetes y Escalabilidad

### 9.1 Deployments y Services

**deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fisica-api
  namespace: production
  labels:
    app: fisica-api
    version: v1.0.0
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: fisica-api
  template:
    metadata:
      labels:
        app: fisica-api
        version: v1.0.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8000"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: api
        image: ghcr.io/myorg/fisica-api:v1.0.0
        imagePullPolicy: Always
        ports:
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8000
          name: metrics
          protocol: TCP
        
        env:
        - name: ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: redis-url
        - name: SENTRY_DSN
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: sentry-dsn
        
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        
        livenessProbe:
          httpGet:
            path: /health/live
            port: 443
            scheme: HTTPS
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 443
            scheme: HTTPS
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        volumeMounts:
        - name: certs
          mountPath: /app/certs
          readOnly: true
        - name: config
          mountPath: /app/config
          readOnly: true
      
      volumes:
      - name: certs
        secret:
          secretName: tls-certs
      - name: config
        configMap:
          name: api-config
      
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - fisica-api
              topologyKey: kubernetes.io/hostname
---
apiVersion: v1
kind: Service
metadata:
  name: fisica-api-service
  namespace: production
spec:
  type: LoadBalancer
  selector:
    app: fisica-api
  ports:
  - name: https
    port: 443
    targetPort: 443
    protocol: TCP
  - name: metrics
    port: 8000
    targetPort: 8000
    protocol: TCP
```

### 9.2 Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: fisica-api-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: fisica-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
      - type: Pods
        value: 4
        periodSeconds: 30
      selectPolicy: Max
```

### 9.3 Ingress con TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fisica-api-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/limit-rps: "50"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.fisica.com
    secretName: api-tls-cert
  rules:
  - host: api.fisica.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: fisica-api-service
            port:
              number: 443
```

---

## 10. Mejores Prácticas de Producción

### 10.1 Gestión de Secretos

**Nunca hardcodear secretos:**

```python
# ❌ MAL
API_KEY = "fisica-2025-secret-key"
DATABASE_URL = "postgresql://user:password@localhost/db"

# ✅ BIEN
import os
from pydantic import BaseSettings

class Settings(BaseSettings):
    api_key: str
    database_url: str
    redis_url: str
    sentry_dsn: str
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

settings = Settings()
```

**Usar gestores de secretos:**

```python
# Con AWS Secrets Manager
import boto3
import json

def get_secret(secret_name: str):
    client = boto3.client('secretsmanager', region_name='us-east-1')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

# Con HashiCorp Vault
import hvac

def get_vault_secret(path: str):
    client = hvac.Client(url='http://vault:8200', token=os.getenv('VAULT_TOKEN'))
    secret = client.secrets.kv.v2.read_secret_version(path=path)
    return secret['data']['data']
```

### 10.2 Configuración por Entorno

**config.py:**

```python
from pydantic import BaseSettings, PostgresDsn, RedisDsn
from typing import Literal

class Settings(BaseSettings):
    # Environment
    env: Literal["development", "staging", "production"] = "development"
    debug: bool = False
    
    # API
    api_v1_prefix: str = "/v1"
    project_name: str = "Física Lab API"
    version: str = "1.0.0"
    
    # Security
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30
    
    # Database
    database_url: PostgresDsn
    db_pool_size: int = 10
    db_max_overflow: int = 20
    
    # Redis
    redis_url: RedisDsn
    cache_ttl: int = 3600
    
    # Celery
    celery_broker_url: str
    celery_result_backend: str
    
    # Monitoring
    sentry_dsn: str = ""
    log_level: str = "INFO"
    
    # Rate Limiting
    rate_limit_per_minute: int = 100
    
    class Config:
        env_file = ".env"
        case_sensitive = False
    
    @property
    def is_production(self) -> bool:
        return self.env == "production"
    
    @property
    def is_development(self) -> bool:
        return self.env == "development"

@lru_cache()
def get_settings() -> Settings:
    return Settings()

settings = get_settings()
```

**Archivos .env por entorno:**

```bash
# .env.development
ENV=development
DEBUG=true
DATABASE_URL=postgresql://fisica:dev@localhost:5432/fisica_dev
REDIS_URL=redis://localhost:6379/0
LOG_LEVEL=DEBUG

# .env.production
ENV=production
DEBUG=false
DATABASE_URL=postgresql://fisica:SECURE_PASSWORD@prod-db:5432/fisica
REDIS_URL=redis://prod-redis:6379/0
LOG_LEVEL=INFO
SENTRY_DSN=https://your-sentry-dsn
```

### 10.3 Graceful Shutdown

```python
# main.py
import signal
import sys

def handle_shutdown(signum, frame):
    """Manejar señal de shutdown"""
    print("Shutting down gracefully...")
    
    # Cerrar conexiones de DB
    # Terminar tareas de Celery
    # Flush logs
    
    sys.exit(0)

# Registrar handlers
signal.signal(signal.SIGTERM, handle_shutdown)
signal.signal(signal.SIGINT, handle_shutdown)

# Para FastAPI con Uvicorn
@app.on_event("shutdown")
async def shutdown_event():
    """Ejecutar al cerrar la aplicación"""
    print("Application shutting down...")
    
    # Cerrar conexiones
    await database.disconnect()
    cache_manager.redis_client.close()
```

### 10.4 Documentación

**README.md completo:**

# Física Lab API

API RESTful para simulaciones científicas con validación física y procesamiento paralelo.

## Features

- ✅ Validación física de parámetros
- ✅ Cálculos relativistas
- ✅ Simulaciones asíncronas (Celery)
- ✅ Caching con Redis
- ✅ Autenticación JWT + API Keys
- ✅ Rate limiting
- ✅ Monitoreo con Prometheus + Grafana
- ✅ Tracing con OpenTelemetry

## Quick Start

### Desarrollo Local

```bash
# Clonar repositorio
git clone https://github.com/myorg/fisica-api
cd fisica-api

# Crear entorno virtual
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements-dev.txt

# Configurar variables de entorno
cp .env.example .env
# Editar .env con tus valores

# Iniciar servicios con Docker Compose
docker-compose -f docker-compose.dev.yml up -d

# Ejecutar migraciones
alembic upgrade head

# Iniciar servidor de desarrollo
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Acceder a la Documentación

- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- OpenAPI Schema: http://localhost:8000/openapi.json

## Estructura del Proyecto

```
app/
├── api/
│   └── v1/
│       ├── endpoints/
│       │   ├── cinematica.py
│       │   ├── simulacion.py
│       │   └── auth.py
│       └── router.py
├── core/
│   ├── security.py
│   ├── config.py
│   └── logging.py
├── models/
│   └── schemas.py
├── services/
│   ├── calculo.py
│   └── simulacion.py
└── main.py
```

## Testing

```bash
# Tests unitarios
pytest tests/unit/ -v

# Tests de integración
pytest tests/integration/ -v

# Coverage
pytest --cov=app --cov-report=html

# Tests de performance
locust -f tests/performance/locustfile.py
```

## Deployment

Ver [DEPLOYMENT.md](DEPLOYMENT.md) para instrucciones detalladas.

## Contribuir

Ver [CONTRIBUTING.md](CONTRIBUTING.md)

## License

MIT
## Resumen Final y Próximos Pasos

Has llegado al final de esta guía comprehensiva. Ahora tienes:

1. ✅ **OpenAPI completo** - Contract-first o code-first
2. ✅ **FastAPI avanzado** - Validación física, DI, async tasks
3. ✅ **Seguridad en profundidad** - JWT, API keys, scopes, rate limiting
4. ✅ **Docker optimizado** - Multi-stage, security scanning
5. ✅ **Testing completo** - Unit, integration, E2E, performance
6. ✅ **DevSecOps pipeline** - CI/CD automatizado con múltiples gates
7. ✅ **Observabilidad** - Logging, métricas, tracing, health checks
8. ✅ **CD avanzado** - Blue-green, canary, rollback automático
9. ✅ **Kubernetes** - Deployments, HPA, Ingress
10. ✅ **Mejores prácticas** - Secretos, configuración, documentación

### Checklist de Implementación

## Fase 1: Fundamentos (Semana 1-2)
- [ ] Configurar proyecto con estructura profesional
- [ ] Implementar modelos Pydantic con validación
- [ ] Crear endpoints básicos con FastAPI
- [ ] Configurar Docker con multi-stage build
- [ ] Escribir tests unitarios básicos

## Fase 2: Seguridad (Semana 3)
- [ ] Implementar autenticación JWT
- [ ] Agregar API key authentication
- [ ] Configurar CORS
- [ ] Implementar rate limiting
- [ ] Scanner de seguridad (Bandit, Safety)

## Fase 3: Infraestructura (Semana 4)
- [ ] Configurar PostgreSQL
- [ ] Agregar Redis para caching
- [ ] Implementar Celery para tareas async
- [ ] Docker Compose completo
- [ ] Tests de integración

## Fase 4: DevSecOps (Semana 5)
- [ ] GitHub Actions pipeline
- [ ] Trivy container scanning
- [ ] Pre-commit hooks
- [ ] Coverage > 80%
- [ ] Documentation completa

## Fase 5: Observabilidad (Semana 6)
- [ ] Logging estructurado
- [ ] Métricas con Prometheus
- [ ] Dashboards en Grafana
- [ ] Tracing con OpenTelemetry
- [ ] Health checks avanzados

## Fase 6: Producción (Semana 7-8)
- [ ] Deploy en staging
- [ ] Performance testing
- [ ] Deploy en production
- [ ] Monitoring 24/7
- [ ] Documentación de runbooks

### Recursos Adicionales

**Documentación:**
- FastAPI: https://fastapi.tiangolo.com
- Pydantic: https://docs.pydantic.dev
- Docker: https://docs.docker.com
- Kubernetes: https://kubernetes.io/docs
- OpenTelemetry: https://opentelemetry.io

**Herramientas:**
- GitHub Actions: https://docs.github.com/actions
- Prometheus: https://prometheus.io/docs
- Grafana: https://grafana.com/docs
- Trivy: https://aquasecurity.github.io/trivy
- Locust: https://docs.locust.io

### Contacto y Soporte

Si necesitas ayuda o tienes preguntas:
- 📧 Email: devops@fisica-lab.com
- 💬 Slack: #fisica-api
- 📚 Wiki: https://wiki.fisica-lab.com

---

**¡Éxito en tu journey DevSecOps!** 🚀