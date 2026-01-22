# Sistema de configuración completo

> **Arquitectura robusta para gestionar toda la configuración de la aplicación**

---

## Sistema de configuración multicapa

Un sistema de configuración completo integra:
- Variables de entorno
- Archivos de configuración
- Valores por defecto
- Validación
- Configuración por entorno

---

## Estructura del sistema

```
proyecto/
├── config/
│   ├── __init__.py
│   ├── base.py           # Configuración base
│   ├── development.py    # Config desarrollo
│   ├── testing.py        # Config testing
│   ├── production.py     # Config producción
│   ├── settings.py       # Settings principal
│   └── validator.py      # Validador
├── .env                  # Variables locales (no commitear)
├── .env.example          # Plantilla
└── app.py               # Aplicación
```

---

## Configuración base

**config/base.py**

```python
"""
Configuración base compartida por todos los entornos
"""

from dataclasses import dataclass, field
from typing import List
import os

@dataclass
class BaseConfig:
    """Configuración base"""
    
    # Aplicación
    APP_NAME: str = 'Mi Aplicación Backend'
    APP_VERSION: str = '1.0.0'
    
    # Servidor
    HOST: str = 'localhost'
    PORT: int = 8000
    
    # Base de datos
    DATABASE_URL: str = 'sqlite:///app.db'
    DATABASE_POOL_SIZE: int = 10
    DATABASE_ECHO: bool = False
    
    # Secrets
    SECRET_KEY: str = ''
    JWT_SECRET: str = ''
    JWT_ALGORITHM: str = 'HS256'
    JWT_ACCESS_TOKEN_EXPIRE_MINUTES: int = 15
    JWT_REFRESH_TOKEN_EXPIRE_DAYS: int = 7
    
    # CORS
    CORS_ORIGINS: List[str] = field(default_factory=lambda: ['http://localhost:3000'])
    CORS_ALLOW_CREDENTIALS: bool = True
    
    # Rate limiting
    RATE_LIMIT_ENABLED: bool = True
    RATE_LIMIT_MAX_REQUESTS: int = 100
    RATE_LIMIT_WINDOW_SECONDS: int = 60
    
    # Logging
    LOG_LEVEL: str = 'INFO'
    LOG_FORMAT: str = '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    
    # Email
    EMAIL_HOST: str = ''
    EMAIL_PORT: int = 587
    EMAIL_USER: str = ''
    EMAIL_PASSWORD: str = ''
    EMAIL_FROM: str = ''
    EMAIL_USE_TLS: bool = True
    
    # Archivos
    UPLOAD_FOLDER: str = 'uploads'
    MAX_UPLOAD_SIZE_MB: int = 10
    ALLOWED_EXTENSIONS: List[str] = field(
        default_factory=lambda: ['jpg', 'jpeg', 'png', 'pdf']
    )
    
    # Cache
    CACHE_TYPE: str = 'simple'
    CACHE_DEFAULT_TIMEOUT: int = 300
    
    # Sesiones
    SESSION_COOKIE_SECURE: bool = False
    SESSION_COOKIE_HTTPONLY: bool = True
    SESSION_COOKIE_SAMESITE: str = 'Lax'
    
    @classmethod
    def from_env(cls) -> 'BaseConfig':
        """Carga configuración desde variables de entorno"""
        return cls(
            # Aplicación
            APP_NAME=os.getenv('APP_NAME', cls.APP_NAME),
            APP_VERSION=os.getenv('APP_VERSION', cls.APP_VERSION),
            
            # Servidor
            HOST=os.getenv('HOST', cls.HOST),
            PORT=int(os.getenv('PORT', cls.PORT)),
            
            # Base de datos
            DATABASE_URL=os.getenv('DATABASE_URL', cls.DATABASE_URL),
            DATABASE_POOL_SIZE=int(os.getenv('DATABASE_POOL_SIZE', cls.DATABASE_POOL_SIZE)),
            DATABASE_ECHO=os.getenv('DATABASE_ECHO', 'false').lower() == 'true',
            
            # Secrets
            SECRET_KEY=os.getenv('SECRET_KEY', ''),
            JWT_SECRET=os.getenv('JWT_SECRET', ''),
            JWT_ALGORITHM=os.getenv('JWT_ALGORITHM', cls.JWT_ALGORITHM),
            JWT_ACCESS_TOKEN_EXPIRE_MINUTES=int(
                os.getenv('JWT_ACCESS_TOKEN_EXPIRE_MINUTES', cls.JWT_ACCESS_TOKEN_EXPIRE_MINUTES)
            ),
            JWT_REFRESH_TOKEN_EXPIRE_DAYS=int(
                os.getenv('JWT_REFRESH_TOKEN_EXPIRE_DAYS', cls.JWT_REFRESH_TOKEN_EXPIRE_DAYS)
            ),
            
            # CORS
            CORS_ORIGINS=os.getenv('CORS_ORIGINS', ','.join(cls.CORS_ORIGINS)).split(','),
            CORS_ALLOW_CREDENTIALS=os.getenv('CORS_ALLOW_CREDENTIALS', 'true').lower() == 'true',
            
            # Rate limiting
            RATE_LIMIT_ENABLED=os.getenv('RATE_LIMIT_ENABLED', 'true').lower() == 'true',
            RATE_LIMIT_MAX_REQUESTS=int(
                os.getenv('RATE_LIMIT_MAX_REQUESTS', cls.RATE_LIMIT_MAX_REQUESTS)
            ),
            RATE_LIMIT_WINDOW_SECONDS=int(
                os.getenv('RATE_LIMIT_WINDOW_SECONDS', cls.RATE_LIMIT_WINDOW_SECONDS)
            ),
            
            # Logging
            LOG_LEVEL=os.getenv('LOG_LEVEL', cls.LOG_LEVEL),
            LOG_FORMAT=os.getenv('LOG_FORMAT', cls.LOG_FORMAT),
            
            # Email
            EMAIL_HOST=os.getenv('EMAIL_HOST', ''),
            EMAIL_PORT=int(os.getenv('EMAIL_PORT', cls.EMAIL_PORT)),
            EMAIL_USER=os.getenv('EMAIL_USER', ''),
            EMAIL_PASSWORD=os.getenv('EMAIL_PASSWORD', ''),
            EMAIL_FROM=os.getenv('EMAIL_FROM', ''),
            EMAIL_USE_TLS=os.getenv('EMAIL_USE_TLS', 'true').lower() == 'true',
            
            # Archivos
            UPLOAD_FOLDER=os.getenv('UPLOAD_FOLDER', cls.UPLOAD_FOLDER),
            MAX_UPLOAD_SIZE_MB=int(os.getenv('MAX_UPLOAD_SIZE_MB', cls.MAX_UPLOAD_SIZE_MB)),
            ALLOWED_EXTENSIONS=os.getenv(
                'ALLOWED_EXTENSIONS',
                ','.join(cls.ALLOWED_EXTENSIONS)
            ).split(','),
            
            # Cache
            CACHE_TYPE=os.getenv('CACHE_TYPE', cls.CACHE_TYPE),
            CACHE_DEFAULT_TIMEOUT=int(
                os.getenv('CACHE_DEFAULT_TIMEOUT', cls.CACHE_DEFAULT_TIMEOUT)
            ),
            
            # Sesiones
            SESSION_COOKIE_SECURE=os.getenv('SESSION_COOKIE_SECURE', 'false').lower() == 'true',
            SESSION_COOKIE_HTTPONLY=os.getenv('SESSION_COOKIE_HTTPONLY', 'true').lower() == 'true',
            SESSION_COOKIE_SAMESITE=os.getenv('SESSION_COOKIE_SAMESITE', cls.SESSION_COOKIE_SAMESITE)
        )
```

---

## Configuraciones por entorno

**config/development.py**

```python
"""
Configuración de desarrollo
"""

from dataclasses import dataclass
from config.base import BaseConfig

@dataclass
class DevelopmentConfig(BaseConfig):
    """Configuración para desarrollo local"""
    
    # Ambiente
    ENVIRONMENT: str = 'development'
    DEBUG: bool = True
    TESTING: bool = False
    
    # Base de datos
    DATABASE_URL: str = 'sqlite:///dev.db'
    DATABASE_ECHO: bool = True
    
    # Logging
    LOG_LEVEL: str = 'DEBUG'
    
    # CORS (permitir todo en dev)
    CORS_ORIGINS: list = None  # Permite todos
    
    # Rate limiting (más permisivo)
    RATE_LIMIT_MAX_REQUESTS: int = 1000
    
    # Email (modo debug - no envía)
    EMAIL_HOST: str = 'localhost'
    EMAIL_PORT: int = 1025  # MailHog
    
    # Sesiones
    SESSION_COOKIE_SECURE: bool = False  # HTTP ok en dev
```

**config/testing.py**

```python
"""
Configuración de testing
"""

from dataclasses import dataclass
from config.base import BaseConfig

@dataclass
class TestingConfig(BaseConfig):
    """Configuración para tests"""
    
    # Ambiente
    ENVIRONMENT: str = 'testing'
    DEBUG: bool = False
    TESTING: bool = True
    
    # Base de datos
    DATABASE_URL: str = 'sqlite:///:memory:'
    DATABASE_ECHO: bool = False
    
    # Logging
    LOG_LEVEL: str = 'WARNING'
    
    # Rate limiting (desactivado en tests)
    RATE_LIMIT_ENABLED: bool = False
    
    # Email (no enviar en tests)
    EMAIL_HOST: str = ''
    
    # Cache (simple para tests)
    CACHE_TYPE: str = 'simple'
```

**config/production.py**

```python
"""
Configuración de producción
"""

from dataclasses import dataclass
from config.base import BaseConfig

@dataclass
class ProductionConfig(BaseConfig):
    """Configuración para producción"""
    
    # Ambiente
    ENVIRONMENT: str = 'production'
    DEBUG: bool = False
    TESTING: bool = False
    
    # Base de datos
    DATABASE_URL: str = ''  # DEBE venir de ENV
    DATABASE_ECHO: bool = False
    DATABASE_POOL_SIZE: int = 20
    
    # Logging
    LOG_LEVEL: str = 'ERROR'
    
    # CORS (restrictivo)
    CORS_ORIGINS: list = None  # Configurar explícitamente
    
    # Rate limiting (estricto)
    RATE_LIMIT_MAX_REQUESTS: int = 60
    
    # Sesiones
    SESSION_COOKIE_SECURE: bool = True  # Solo HTTPS
    SESSION_COOKIE_SAMESITE: str = 'Strict'
```

---

## Settings principal

**config/settings.py**

```python
"""
Sistema principal de configuración
"""

import os
from typing import Union
from config.base import BaseConfig
from config.development import DevelopmentConfig
from config.testing import TestingConfig
from config.production import ProductionConfig

ConfigType = Union[DevelopmentConfig, TestingConfig, ProductionConfig]

class Settings:
    """Sistema de configuración"""
    
    _instance: ConfigType = None
    
    @classmethod
    def get_config(cls, environment: str = None) -> ConfigType:
        """
        Obtiene configuración según entorno
        
        Args:
            environment: 'development', 'testing', 'production'
        
        Returns:
            Instancia de configuración
        """
        if cls._instance is not None:
            return cls._instance
        
        # Determinar entorno
        if environment is None:
            environment = os.getenv('ENVIRONMENT', 'development')
        
        # Seleccionar configuración
        configs = {
            'development': DevelopmentConfig,
            'testing': TestingConfig,
            'production': ProductionConfig
        }
        
        config_class = configs.get(environment, DevelopmentConfig)
        
        # Crear instancia desde variables de entorno
        cls._instance = config_class.from_env()
        cls._instance.ENVIRONMENT = environment
        
        return cls._instance
    
    @classmethod
    def reset(cls):
        """Resetea configuración (útil para tests)"""
        cls._instance = None

# Crear instancia global
settings = Settings.get_config()
```

---

## Validador

**config/validator.py**

```python
"""
Validador de configuración
"""

from typing import List
from config.settings import ConfigType

class ConfigValidator:
    """Valida configuración"""
    
    @staticmethod
    def validar(config: ConfigType) -> List[str]:
        """
        Valida configuración completa
        
        Returns:
            Lista de errores (vacía si es válida)
        """
        errores = []
        
        # Validar secrets
        errores.extend(ConfigValidator._validar_secrets(config))
        
        # Validar base de datos
        errores.extend(ConfigValidator._validar_database(config))
        
        # Validar email
        errores.extend(ConfigValidator._validar_email(config))
        
        # Validar producción
        if config.ENVIRONMENT == 'production':
            errores.extend(ConfigValidator._validar_produccion(config))
        
        return errores
    
    @staticmethod
    def _validar_secrets(config: ConfigType) -> List[str]:
        """Valida secrets"""
        errores = []
        
        if not config.SECRET_KEY:
            errores.append('SECRET_KEY es requerido')
        elif len(config.SECRET_KEY) < 32:
            errores.append('SECRET_KEY debe tener al menos 32 caracteres')
        
        if not config.JWT_SECRET:
            errores.append('JWT_SECRET es requerido')
        elif len(config.JWT_SECRET) < 32:
            errores.append('JWT_SECRET debe tener al menos 32 caracteres')
        
        # En producción, no usar valores por defecto
        if config.ENVIRONMENT == 'production':
            if 'dev-' in config.SECRET_KEY.lower():
                errores.append('SECRET_KEY no debe contener "dev" en producción')
        
        return errores
    
    @staticmethod
    def _validar_database(config: ConfigType) -> List[str]:
        """Valida configuración de base de datos"""
        errores = []
        
        if not config.DATABASE_URL:
            errores.append('DATABASE_URL es requerido')
        
        if config.ENVIRONMENT == 'production':
            if 'sqlite' in config.DATABASE_URL.lower():
                errores.append('SQLite no recomendado en producción')
        
        return errores
    
    @staticmethod
    def _validar_email(config: ConfigType) -> List[str]:
        """Valida configuración de email"""
        errores = []
        
        if config.EMAIL_HOST and not config.EMAIL_USER:
            errores.append('EMAIL_USER requerido si EMAIL_HOST está configurado')
        
        return errores
    
    @staticmethod
    def _validar_produccion(config: ConfigType) -> List[str]:
        """Validaciones específicas de producción"""
        errores = []
        
        if config.DEBUG:
            errores.append('DEBUG debe ser False en producción')
        
        if not config.SESSION_COOKIE_SECURE:
            errores.append('SESSION_COOKIE_SECURE debe ser True en producción')
        
        if config.LOG_LEVEL == 'DEBUG':
            errores.append('LOG_LEVEL no debe ser DEBUG en producción')
        
        return errores

def validar_configuracion(config: ConfigType):
    """
    Valida configuración y lanza error si es inválida
    
    Raises:
        ValueError: Si la configuración es inválida
    """
    errores = ConfigValidator.validar(config)
    
    if errores:
        mensaje = '❌ Errores de configuración:\n'
        for error in errores:
            mensaje += f'   - {error}\n'
        raise ValueError(mensaje)
    
    print('✅ Configuración válida')
```

---

## Aplicación completa

**app_config.py**

```python
"""
Aplicación con sistema de configuración completo
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
import json

# Cargar variables de entorno
from dotenv import load_dotenv
load_dotenv()

# Importar configuración
from config.settings import settings
from config.validator import validar_configuracion

# Validar configuración al inicio
try:
    validar_configuracion(settings)
except ValueError as e:
    print(e)
    exit(1)

class ConfigHandler(BaseHTTPRequestHandler):
    """Handler con configuración"""
    
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
            <title>{settings.APP_NAME}</title>
        </head>
        <body>
            <h1>⚙️ {settings.APP_NAME}</h1>
            <p><strong>Versión:</strong> {settings.APP_VERSION}</p>
            <p><strong>Entorno:</strong> {settings.ENVIRONMENT}</p>
            <p><strong>Debug:</strong> {settings.DEBUG}</p>
            
            <h2>Endpoints:</h2>
            <ul>
                <li><a href="/health">/health</a> - Health check</li>
                <li><a href="/config">/config</a> - Ver configuración</li>
            </ul>
            
            <h2>Configuración activa:</h2>
            <ul>
                <li>🌍 Entorno: {settings.ENVIRONMENT}</li>
                <li>🐛 Debug: {settings.DEBUG}</li>
                <li>📊 Log Level: {settings.LOG_LEVEL}</li>
                <li>💾 Database: {settings.DATABASE_URL.split('://')[0]}://...</li>
                <li>📧 Email: {'Configurado' if settings.EMAIL_HOST else 'No configurado'}</li>
                <li>🔒 CORS: {len(settings.CORS_ORIGINS) if settings.CORS_ORIGINS else 'Todos'} orígenes</li>
                <li>⚡ Rate Limit: {'Activo' if settings.RATE_LIMIT_ENABLED else 'Desactivado'}</li>
            </ul>
        </body>
        </html>
        """
        
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(html.encode())
    
    def health(self):
        """GET /health"""
        data = {
            'status': 'healthy',
            'app': settings.APP_NAME,
            'version': settings.APP_VERSION,
            'environment': settings.ENVIRONMENT,
            'debug': settings.DEBUG
        }
        
        self._enviar_json(data)
    
    def config(self):
        """GET /config - Configuración (sin secrets)"""
        data = {
            'app': {
                'name': settings.APP_NAME,
                'version': settings.APP_VERSION,
                'environment': settings.ENVIRONMENT,
                'debug': settings.DEBUG,
                'testing': settings.TESTING
            },
            'server': {
                'host': settings.HOST,
                'port': settings.PORT
            },
            'database': {
                'url': self._ocultar_password(settings.DATABASE_URL),
                'pool_size': settings.DATABASE_POOL_SIZE,
                'echo': settings.DATABASE_ECHO
            },
            'jwt': {
                'algorithm': settings.JWT_ALGORITHM,
                'access_token_expire_minutes': settings.JWT_ACCESS_TOKEN_EXPIRE_MINUTES,
                'refresh_token_expire_days': settings.JWT_REFRESH_TOKEN_EXPIRE_DAYS
            },
            'cors': {
                'origins': settings.CORS_ORIGINS or ['*'],
                'allow_credentials': settings.CORS_ALLOW_CREDENTIALS
            },
            'rate_limit': {
                'enabled': settings.RATE_LIMIT_ENABLED,
                'max_requests': settings.RATE_LIMIT_MAX_REQUESTS,
                'window_seconds': settings.RATE_LIMIT_WINDOW_SECONDS
            },
            'logging': {
                'level': settings.LOG_LEVEL,
                'format': settings.LOG_FORMAT
            },
            'email': {
                'host': settings.EMAIL_HOST,
                'port': settings.EMAIL_PORT,
                'user': settings.EMAIL_USER,
                'from': settings.EMAIL_FROM,
                'use_tls': settings.EMAIL_USE_TLS
            },
            'uploads': {
                'folder': settings.UPLOAD_FOLDER,
                'max_size_mb': settings.MAX_UPLOAD_SIZE_MB,
                'allowed_extensions': settings.ALLOWED_EXTENSIONS
            },
            'cache': {
                'type': settings.CACHE_TYPE,
                'default_timeout': settings.CACHE_DEFAULT_TIMEOUT
            },
            'session': {
                'cookie_secure': settings.SESSION_COOKIE_SECURE,
                'cookie_httponly': settings.SESSION_COOKIE_HTTPONLY,
                'cookie_samesite': settings.SESSION_COOKIE_SAMESITE
            },
            'secrets': {
                'secret_key': '***OCULTO***',
                'jwt_secret': '***OCULTO***',
                'email_password': '***OCULTO***'
            }
        }
        
        self._enviar_json(data)
    
    def _enviar_json(self, data: dict, status: int = 200):
        """Envía JSON"""
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(data, indent=2).encode())
    
    @staticmethod
    def _ocultar_password(url: str) -> str:
        """Oculta password en URL"""
        if '@' in url:
            partes = url.split('@')
            if ':' in partes[0]:
                protocolo, _ = partes[0].rsplit(':', 1)
                return f'{protocolo}:***@{partes[1]}'
        return url
    
    def log_message(self, format, *args):
        if settings.DEBUG:
            print(f'📥 {self.command} {self.path}')

if __name__ == '__main__':
    servidor = HTTPServer((settings.HOST, settings.PORT), ConfigHandler)
    
    print('='*60)
    print(f'⚙️  {settings.APP_NAME} v{settings.APP_VERSION}')
    print('='*60)
    print(f'\n🚀 Servidor: http://{settings.HOST}:{settings.PORT}')
    print(f'🌍 Entorno: {settings.ENVIRONMENT}')
    print(f'🐛 Debug: {settings.DEBUG}')
    print(f'📊 Log Level: {settings.LOG_LEVEL}')
    print(f'💾 Database: {settings.DATABASE_URL.split("://")[0]}://...')
    print('\n⛔ Ctrl+C para detener\n')
    
    try:
        servidor.serve_forever()
    except KeyboardInterrupt:
        print('\n\n⛔ Servidor detenido')
        servidor.shutdown()
```

---

## Uso en diferentes entornos

```bash
# Desarrollo (usa .env por defecto)
python app_config.py

# Testing
ENVIRONMENT=testing python app_config.py

# Producción
ENVIRONMENT=production python app_config.py
```

---

## Archivo .env completo

```bash
# .env - Configuración de desarrollo

# Aplicación
APP_NAME=Mi Aplicación Backend
APP_VERSION=1.0.0
ENVIRONMENT=development

# Servidor
HOST=localhost
PORT=8000

# Base de datos
DATABASE_URL=sqlite:///dev.db
DATABASE_POOL_SIZE=10
DATABASE_ECHO=true

# Secrets
SECRET_KEY=dev-secret-key-cambiar-en-produccion-abcdef1234567890
JWT_SECRET=dev-jwt-secret-cambiar-en-produccion-xyz9876543210

# JWT
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7

# CORS
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
CORS_ALLOW_CREDENTIALS=true

# Rate Limiting
RATE_LIMIT_ENABLED=true
RATE_LIMIT_MAX_REQUESTS=100
RATE_LIMIT_WINDOW_SECONDS=60

# Logging
LOG_LEVEL=DEBUG

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=tu-email@gmail.com
EMAIL_PASSWORD=tu-password-o-app-password
EMAIL_FROM=noreply@tuapp.com
EMAIL_USE_TLS=true

# Uploads
UPLOAD_FOLDER=uploads
MAX_UPLOAD_SIZE_MB=10
ALLOWED_EXTENSIONS=jpg,jpeg,png,pdf,doc,docx

# Cache
CACHE_TYPE=simple
CACHE_DEFAULT_TIMEOUT=300

# Sesiones
SESSION_COOKIE_SECURE=false
SESSION_COOKIE_HTTPONLY=true
SESSION_COOKIE_SAMESITE=Lax
```

---

## Resumen

Has aprendido:

✅ Sistema de configuración multicapa  
✅ Configuraciones por entorno  
✅ Validación de configuración  
✅ Variables de entorno  
✅ Secrets management  
✅ Configuración centralizada

**Siguiente:** Bloque 7 - Django

---

## Recursos

- **[12 Factor App - Config](https://12factor.net/config)** - Configuración en entorno
- **[Pydantic Settings](https://pydantic-docs.helpmanual.io/usage/settings/)** - Settings con Pydantic

Sistema de configuración robusto y flexible. ⚙️
