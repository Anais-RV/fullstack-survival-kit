# Variables de entorno

> **Gestiona configuración sensible y específica del entorno sin hardcodear valores**

---

## ¿Qué son variables de entorno?

**Variables de entorno** son configuraciones del sistema operativo que:
- Se definen fuera del código
- Son específicas del entorno (dev/test/prod)
- Almacenan datos sensibles (secrets, API keys)
- Permiten configurar sin recompilar

**Ventajas:**
- ✅ Secrets fuera del código fuente
- ✅ Diferentes configs por entorno
- ✅ No se commitean al repositorio
- ✅ Fácil cambiar sin modificar código

---

## Sin variables de entorno (hardcoded)

**config_hardcoded.py**

```python
"""
Configuración hardcodeada (MAL)
"""

# ❌ Secrets en el código
DATABASE_URL = 'postgresql://admin:password123@localhost:5432/mydb'
SECRET_KEY = 'clave-super-secreta-que-no-deberia-estar-aqui'
API_KEY = 'sk_live_51abc123def456ghi789'

# ❌ Mismo valor para todos los entornos
DEBUG = True
PORT = 8000

# ❌ Email credentials en el código
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_USER = 'mi-email@gmail.com'
EMAIL_PASSWORD = 'mi-password-real'
```

**Problemas:**
- ❌ Secrets en Git (peligro de seguridad)
- ❌ Mismo config para dev/prod
- ❌ Difícil cambiar sin recompilar
- ❌ Colaboradores ven tus secrets

---

## Con variables de entorno

**Archivo .env (NO se commitea)**

```bash
# .env - Configuración local (NO COMMITEAR)

# Entorno
ENVIRONMENT=development

# Base de datos
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
DATABASE_POOL_SIZE=10

# Secrets
SECRET_KEY=clave-secreta-super-larga-y-aleatoria-abc123
JWT_SECRET=otra-clave-para-jwt-xyz789

# API externas
STRIPE_API_KEY=sk_test_abc123
SENDGRID_API_KEY=SG.abc123def456

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=mi-email@gmail.com
EMAIL_PASSWORD=mi-password

# Servidor
PORT=8000
DEBUG=true
```

**Archivo .env.example (SÍ se commitea)**

```bash
# .env.example - Plantilla para nuevos desarrolladores

# Entorno
ENVIRONMENT=development

# Base de datos
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
DATABASE_POOL_SIZE=10

# Secrets (valores de ejemplo)
SECRET_KEY=cambiar-por-clave-real
JWT_SECRET=cambiar-por-clave-real

# API externas
STRIPE_API_KEY=sk_test_tu_clave_aqui
SENDGRID_API_KEY=SG.tu_clave_aqui

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=tu-email@gmail.com
EMAIL_PASSWORD=tu-password

# Servidor
PORT=8000
DEBUG=true
```

**Archivo .gitignore**

```
# No commitear variables de entorno
.env
.env.local
.env.*.local

# Sí commitear ejemplos
!.env.example
```

---

## Leer variables de entorno

**config/env.py**

```python
"""
Lectura de variables de entorno
"""

import os
from typing import Optional

def get_env(key: str, default: Optional[str] = None, required: bool = False) -> str:
    """
    Obtiene variable de entorno
    
    Args:
        key: Nombre de la variable
        default: Valor por defecto si no existe
        required: Si es True, lanza error si no existe
    
    Returns:
        Valor de la variable
    
    Raises:
        ValueError: Si required=True y no existe
    """
    value = os.getenv(key, default)
    
    if required and value is None:
        raise ValueError(f'Variable de entorno requerida: {key}')
    
    return value

def get_env_int(key: str, default: int = 0) -> int:
    """Obtiene variable como entero"""
    value = os.getenv(key)
    if value is None:
        return default
    try:
        return int(value)
    except ValueError:
        return default

def get_env_bool(key: str, default: bool = False) -> bool:
    """Obtiene variable como booleano"""
    value = os.getenv(key, '').lower()
    if value in ('true', '1', 'yes', 'on'):
        return True
    if value in ('false', '0', 'no', 'off'):
        return False
    return default

def get_env_list(key: str, separator: str = ',', default: list = None) -> list:
    """Obtiene variable como lista"""
    value = os.getenv(key)
    if not value:
        return default or []
    return [item.strip() for item in value.split(separator)]

# Ejemplos de uso
DATABASE_URL = get_env('DATABASE_URL', required=True)
SECRET_KEY = get_env('SECRET_KEY', required=True)
PORT = get_env_int('PORT', default=8000)
DEBUG = get_env_bool('DEBUG', default=False)
ALLOWED_HOSTS = get_env_list('ALLOWED_HOSTS', default=['localhost'])
```

---

## Cargar desde archivo .env

**config/load_env.py**

```python
"""
Carga variables de entorno desde archivo .env
"""

import os
from pathlib import Path

def load_env(env_file: str = '.env'):
    """
    Carga variables desde archivo .env
    
    Args:
        env_file: Ruta al archivo .env
    """
    env_path = Path(env_file)
    
    if not env_path.exists():
        print(f'⚠️  Archivo {env_file} no encontrado')
        return
    
    with open(env_path, 'r', encoding='utf-8') as f:
        for line in f:
            line = line.strip()
            
            # Ignorar comentarios y líneas vacías
            if not line or line.startswith('#'):
                continue
            
            # Parsear KEY=VALUE
            if '=' in line:
                key, value = line.split('=', 1)
                key = key.strip()
                value = value.strip()
                
                # Remover comillas
                if value.startswith('"') and value.endswith('"'):
                    value = value[1:-1]
                elif value.startswith("'") and value.endswith("'"):
                    value = value[1:-1]
                
                # Setear variable de entorno
                os.environ[key] = value
    
    print(f'✅ Variables cargadas desde {env_file}')

# Cargar al importar el módulo
if __name__ != '__main__':
    load_env()
```

**Con python-dotenv (librería)**

```python
"""
Usar python-dotenv para cargar .env
"""

from dotenv import load_dotenv
import os

# Cargar variables de entorno
load_dotenv()

# Ahora están disponibles
DATABASE_URL = os.getenv('DATABASE_URL')
SECRET_KEY = os.getenv('SECRET_KEY')
PORT = int(os.getenv('PORT', 8000))
DEBUG = os.getenv('DEBUG', 'false').lower() == 'true'
```

---

## Clase de configuración

**config/settings.py**

```python
"""
Clase de configuración con variables de entorno
"""

import os
from dataclasses import dataclass
from typing import List

@dataclass
class Settings:
    """Configuración de la aplicación"""
    
    # Entorno
    environment: str
    debug: bool
    
    # Servidor
    host: str
    port: int
    
    # Base de datos
    database_url: str
    database_pool_size: int
    
    # Secrets
    secret_key: str
    jwt_secret: str
    jwt_expiration_minutes: int
    
    # Email
    email_host: str
    email_port: int
    email_user: str
    email_password: str
    
    # APIs externas
    stripe_api_key: str
    sendgrid_api_key: str
    
    # Otros
    allowed_hosts: List[str]
    
    @classmethod
    def from_env(cls) -> 'Settings':
        """Crea configuración desde variables de entorno"""
        return cls(
            # Entorno
            environment=os.getenv('ENVIRONMENT', 'development'),
            debug=os.getenv('DEBUG', 'false').lower() == 'true',
            
            # Servidor
            host=os.getenv('HOST', 'localhost'),
            port=int(os.getenv('PORT', 8000)),
            
            # Base de datos
            database_url=os.getenv('DATABASE_URL', 'sqlite:///local.db'),
            database_pool_size=int(os.getenv('DATABASE_POOL_SIZE', 10)),
            
            # Secrets
            secret_key=os.getenv('SECRET_KEY', 'dev-secret-key'),
            jwt_secret=os.getenv('JWT_SECRET', 'dev-jwt-secret'),
            jwt_expiration_minutes=int(os.getenv('JWT_EXPIRATION_MINUTES', 15)),
            
            # Email
            email_host=os.getenv('EMAIL_HOST', 'smtp.gmail.com'),
            email_port=int(os.getenv('EMAIL_PORT', 587)),
            email_user=os.getenv('EMAIL_USER', ''),
            email_password=os.getenv('EMAIL_PASSWORD', ''),
            
            # APIs
            stripe_api_key=os.getenv('STRIPE_API_KEY', ''),
            sendgrid_api_key=os.getenv('SENDGRID_API_KEY', ''),
            
            # Otros
            allowed_hosts=os.getenv('ALLOWED_HOSTS', 'localhost').split(',')
        )
    
    def es_produccion(self) -> bool:
        """Verifica si está en producción"""
        return self.environment == 'production'
    
    def es_desarrollo(self) -> bool:
        """Verifica si está en desarrollo"""
        return self.environment == 'development'
    
    def validar(self):
        """Valida configuración"""
        if self.es_produccion():
            if self.secret_key == 'dev-secret-key':
                raise ValueError('SECRET_KEY debe cambiarse en producción')
            if self.debug:
                raise ValueError('DEBUG debe ser False en producción')
            if not self.database_url.startswith('postgresql'):
                print('⚠️  Se recomienda PostgreSQL en producción')

# Instancia global
settings = Settings.from_env()
settings.validar()
```

---

## Configuración por entorno

**config/environments.py**

```python
"""
Configuraciones específicas por entorno
"""

from dataclasses import dataclass

@dataclass
class BaseConfig:
    """Configuración base"""
    APP_NAME = 'Mi Aplicación'
    VERSION = '1.0.0'

@dataclass
class DevelopmentConfig(BaseConfig):
    """Configuración de desarrollo"""
    DEBUG = True
    TESTING = False
    DATABASE_URL = 'sqlite:///dev.db'
    LOG_LEVEL = 'DEBUG'

@dataclass
class TestingConfig(BaseConfig):
    """Configuración de testing"""
    DEBUG = False
    TESTING = True
    DATABASE_URL = 'sqlite:///test.db'
    LOG_LEVEL = 'WARNING'

@dataclass
class ProductionConfig(BaseConfig):
    """Configuración de producción"""
    DEBUG = False
    TESTING = False
    DATABASE_URL = None  # Debe venir de ENV
    LOG_LEVEL = 'ERROR'

def get_config(environment: str = None):
    """
    Obtiene configuración según entorno
    
    Args:
        environment: 'development', 'testing', 'production'
    
    Returns:
        Clase de configuración apropiada
    """
    import os
    
    if environment is None:
        environment = os.getenv('ENVIRONMENT', 'development')
    
    configs = {
        'development': DevelopmentConfig,
        'testing': TestingConfig,
        'production': ProductionConfig
    }
    
    return configs.get(environment, DevelopmentConfig)
```

---

## Uso en la aplicación

**app_env.py**

```python
"""
Aplicación usando variables de entorno
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
import json

# Cargar variables de entorno
from config.load_env import load_env
load_env()

# Importar configuración
from config.settings import settings

class EnvHandler(BaseHTTPRequestHandler):
    """Handler que usa configuración de entorno"""
    
    def do_GET(self):
        if self.path == '/':
            self.home()
        elif self.path == '/health':
            self.health()
        elif self.path == '/config':
            self.config()
        else:
            self.send_response(404)
            self.end_headers()
    
    def home(self):
        """GET /"""
        html = f"""
        <!DOCTYPE html>
        <html>
        <head>
            <title>Variables de Entorno</title>
        </head>
        <body>
            <h1>🔧 Variables de Entorno</h1>
            <p><strong>Entorno:</strong> {settings.environment}</p>
            <p><strong>Debug:</strong> {settings.debug}</p>
            <p><strong>Puerto:</strong> {settings.port}</p>
            
            <h2>Endpoints:</h2>
            <ul>
                <li><a href="/health">/health</a> - Health check</li>
                <li><a href="/config">/config</a> - Ver configuración</li>
            </ul>
            
            <h2>Variables cargadas desde .env:</h2>
            <ul>
                <li>✅ ENVIRONMENT</li>
                <li>✅ DEBUG</li>
                <li>✅ PORT</li>
                <li>✅ DATABASE_URL</li>
                <li>✅ SECRET_KEY (oculto)</li>
                <li>✅ Y más...</li>
            </ul>
        </body>
        </html>
        """
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def health(self):
        """GET /health - Health check"""
        data = {
            'status': 'healthy',
            'environment': settings.environment,
            'debug': settings.debug
        }
        
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(data, indent=2).encode())
    
    def config(self):
        """GET /config - Muestra config (sin secrets)"""
        data = {
            'environment': settings.environment,
            'debug': settings.debug,
            'host': settings.host,
            'port': settings.port,
            'database_url': self._ocultar_password(settings.database_url),
            'database_pool_size': settings.database_pool_size,
            'jwt_expiration_minutes': settings.jwt_expiration_minutes,
            'email_host': settings.email_host,
            'email_port': settings.email_port,
            'email_user': settings.email_user,
            'allowed_hosts': settings.allowed_hosts,
            # Secrets ocultos
            'secret_key': '***OCULTO***',
            'jwt_secret': '***OCULTO***',
            'email_password': '***OCULTO***',
            'stripe_api_key': '***OCULTO***',
            'sendgrid_api_key': '***OCULTO***'
        }
        
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(data, indent=2).encode())
    
    @staticmethod
    def _ocultar_password(url: str) -> str:
        """Oculta password en URL de base de datos"""
        if '@' in url:
            partes = url.split('@')
            if ':' in partes[0]:
                protocolo, credenciales = partes[0].rsplit(':', 1)
                return f'{protocolo}:***@{partes[1]}'
        return url
    
    def log_message(self, format, *args):
        if settings.debug:
            print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    servidor = HTTPServer((settings.host, settings.port), EnvHandler)
    
    print('='*60)
    print('🔧 Aplicación con Variables de Entorno')
    print('='*60)
    print(f'\n🚀 Servidor: http://{settings.host}:{settings.port}')
    print(f'🌍 Entorno: {settings.environment}')
    print(f'🐛 Debug: {settings.debug}')
    print(f'💾 Base de datos: {settings.database_url}')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Diferentes entornos

**Desarrollo (.env.development)**

```bash
ENVIRONMENT=development
DEBUG=true
PORT=8000
DATABASE_URL=sqlite:///dev.db
SECRET_KEY=dev-secret-key-not-secure
LOG_LEVEL=DEBUG
```

**Testing (.env.test)**

```bash
ENVIRONMENT=testing
DEBUG=false
PORT=8001
DATABASE_URL=sqlite:///test.db
SECRET_KEY=test-secret-key
LOG_LEVEL=WARNING
```

**Producción (.env.production)**

```bash
ENVIRONMENT=production
DEBUG=false
PORT=80
DATABASE_URL=postgresql://user:pass@prod-db:5432/mydb
SECRET_KEY=clave-super-larga-y-aleatoria-generada-con-secrets
LOG_LEVEL=ERROR
```

**Usar diferentes archivos:**

```bash
# Desarrollo (usa .env por defecto)
python app_env.py

# Testing
ENV_FILE=.env.test python app_env.py

# Producción
ENV_FILE=.env.production python app_env.py
```

---

## Secrets en producción

**En servidores:**

```bash
# Setear variables en el shell
export DATABASE_URL="postgresql://..."
export SECRET_KEY="..."
python app.py

# O en un script de inicio
#!/bin/bash
source /etc/secrets/app.env
python app.py
```

**En Docker:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    environment:
      - ENVIRONMENT=production
      - DEBUG=false
      - DATABASE_URL=${DATABASE_URL}
      - SECRET_KEY=${SECRET_KEY}
    env_file:
      - .env.production
```

**En servicios cloud:**

- **Heroku:** `heroku config:set SECRET_KEY=...`
- **AWS:** AWS Systems Manager Parameter Store
- **GCP:** Secret Manager
- **Azure:** Azure Key Vault

---

## Validación de variables

**config/validator.py**

```python
"""
Validador de variables de entorno
"""

import os
import sys

REQUIRED_VARS = [
    'DATABASE_URL',
    'SECRET_KEY',
    'JWT_SECRET'
]

OPTIONAL_VARS = {
    'PORT': '8000',
    'DEBUG': 'false',
    'ENVIRONMENT': 'development'
}

def validar_variables():
    """Valida que todas las variables requeridas existan"""
    faltantes = []
    
    for var in REQUIRED_VARS:
        if not os.getenv(var):
            faltantes.append(var)
    
    if faltantes:
        print('❌ Variables de entorno faltantes:')
        for var in faltantes:
            print(f'   - {var}')
        print('\n💡 Crea un archivo .env basado en .env.example')
        sys.exit(1)
    
    print('✅ Todas las variables de entorno están configuradas')

def mostrar_variables():
    """Muestra variables configuradas (sin valores)"""
    print('\n📋 Variables de entorno configuradas:')
    
    for var in REQUIRED_VARS:
        if os.getenv(var):
            print(f'   ✅ {var}')
    
    for var, default in OPTIONAL_VARS.items():
        valor = os.getenv(var, default)
        print(f'   ✅ {var}={valor}')
    
    print()

if __name__ == '__main__':
    validar_variables()
    mostrar_variables()
```

---

## Resumen

Has aprendido:

✅ Usar variables de entorno  
✅ Archivo .env para desarrollo  
✅ .env.example para el equipo  
✅ Clase Settings centralizada  
✅ Diferentes configs por entorno  
✅ Secrets fuera del código

**Siguiente:** Sistema de configuración completo

---

## Recursos

- **[python-dotenv](https://pypi.org/project/python-dotenv/)** - Cargar .env
- **[12 Factor App](https://12factor.net/config)** - Config en entorno

Configuración segura y flexible. 🔧
