# Proyecto: Juego RPG con POO

> **Construye un sistema completo aplicando POO**

---

## Introducción al proyecto

Vas a construir un **juego de rol (RPG)** que integra todos los conceptos de POO aprendidos:

- ✅ Clases y objetos
- ✅ Herencia
- ✅ Composición
- ✅ Modelado de dominio
- ✅ Patrones de diseño
- ✅ Principios SOLID

**Objetivo:** Crear un sistema backend para un juego RPG con API REST.

---

## Visión general del juego

### Concepto

**Fantasy Battle Arena** - Un juego RPG donde:

1. **Jugadores** crean **personajes** de diferentes clases
2. Los personajes tienen **atributos** y **habilidades**
3. Pueden **combatir** entre ellos o contra **enemigos**
4. Ganan **experiencia** y **suben de nivel**
5. Obtienen **items** y **equipamiento**
6. Completan **misiones**

### Tecnologías

- **Backend:** Python vanilla + http.server
- **Arquitectura:** POO con SOLID
- **API:** REST JSON
- **Persistencia:** En memoria (luego migraremos a BD)

---

## Arquitectura del sistema

```
┌─────────────────────────────────────────┐
│      API REST (Python vanilla)          │
│  /api/personajes  /api/combates  etc.  │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────┴──────────────────────┐
│         Servicios de Dominio            │
│  GestorPersonajes, GestorCombates      │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────┴──────────────────────┐
│         Modelo de Dominio               │
│  Personaje, Guerrero, Mago, Item       │
└─────────────────────────────────────────┘
```

---

## Entidades del dominio

### Personaje (clase base)

**Atributos:**
- id
- nombre
- clase (guerrero, mago, arquero)
- nivel
- experiencia
- vida (HP)
- vida_maxima
- mana
- mana_maxima
- fuerza
- inteligencia
- agilidad
- defensa
- inventario

**Comportamientos:**
- atacar()
- recibir_daño()
- usar_habilidad()
- subir_nivel()
- equipar_item()

### Clases de personajes

1. **Guerrero**
   - Alta vida y fuerza
   - Baja en mana e inteligencia
   - Habilidad: Golpe Poderoso

2. **Mago**
   - Alta inteligencia y mana
   - Baja vida y fuerza
   - Habilidad: Bola de Fuego

3. **Arquero**
   - Alta agilidad
   - Media en todo
   - Habilidad: Disparo Múltiple

### Item

**Tipos:**
- Arma (aumenta ataque)
- Armadura (aumenta defensa)
- Poción (restaura vida/mana)

### Combate

- Estado del combate
- Turno actual
- Log de acciones

---

## Módulos del proyecto

Este proyecto se divide en 4 módulos:

### Módulo 19 (este): Introducción y setup
- Visión del proyecto
- Arquitectura
- Setup inicial

### [Módulo 20](./20-personajes-entidades.md): Personajes y entidades
- Clase Personaje base
- Clases especializadas (Guerrero, Mago, Arquero)
- Sistema de items
- Inventario

### [Módulo 21](./21-sistema-combate.md): Sistema de combate
- Mecánicas de combate
- Sistema de turnos
- Cálculo de daño
- Experiencia y niveles

### [Módulo 22](./22-api-rest-juego.md): API REST vanilla
- Endpoints para personajes
- Endpoints para combates
- Gestión de estado
- Integración completa

---

## Setup inicial del proyecto

### Estructura de archivos

```
03-backend/
└── proyecto-rpg/
    ├── app.py              # Servidor HTTP vanilla
    ├── config.py           # Configuración
    ├── modelos/
    │   ├── __init__.py
    │   ├── personaje.py    # Clases de personajes
    │   ├── item.py         # Items y equipamiento
    │   └── combate.py      # Sistema de combate
    ├── servicios/
    │   ├── __init__.py
    │   ├── gestor_personajes.py
    │   └── gestor_combates.py
    ├── repositorios/
    │   ├── __init__.py
    │   └── repositorio_memoria.py
    └── utils/
        ├── __init__.py
        └── calculos.py     # Funciones auxiliares
```

### Crear el proyecto

```powershell
# Crear estructura de directorios
mkdir proyecto-rpg
cd proyecto-rpg

mkdir modelos
mkdir servicios
mkdir repositorios
mkdir utils

# Crear archivos __init__.py
echo "" > modelos/__init__.py
echo "" > servicios/__init__.py
echo "" > repositorios/__init__.py
echo "" > utils/__init__.py
```

### Archivo de configuración

**config.py**

```python
"""
Configuración del juego RPG
"""

class Config:
    """Configuración base del juego"""
    
    # Configuración general
    DEBUG = True
    NOMBRE_JUEGO = "Fantasy Battle Arena"
    VERSION = "1.0.0"
    
    # Configuración de personajes
    NIVEL_INICIAL = 1
    VIDA_BASE = 100
    MANA_BASE = 50
    EXPERIENCIA_BASE = 100  # XP necesaria para nivel 2
    MULTIPLICADOR_XP = 1.5  # XP aumenta 50% por nivel
    
    # Configuración de clases
    CLASES_DISPONIBLES = ['guerrero', 'mago', 'arquero']
    
    # Bonificadores por clase
    BONIFICADORES_CLASE = {
        'guerrero': {
            'vida': 1.5,
            'fuerza': 1.8,
            'inteligencia': 0.5,
            'agilidad': 0.7,
            'defensa': 1.3
        },
        'mago': {
            'vida': 0.7,
            'fuerza': 0.5,
            'inteligencia': 2.0,
            'agilidad': 0.8,
            'defensa': 0.6
        },
        'arquero': {
            'vida': 1.0,
            'fuerza': 1.0,
            'inteligencia': 0.8,
            'agilidad': 1.8,
            'defensa': 0.8
        }
    }
    
    # Configuración de combate
    CRITICO_PROBABILIDAD = 0.15  # 15% de crítico
    CRITICO_MULTIPLICADOR = 2.0   # Daño x2
    
    # Configuración de items
    SLOTS_INVENTARIO = 20

class ConfigDesarrollo(Config):
    """Configuración para desarrollo"""
    DEBUG = True
    VIDA_BASE = 200  # Más vida para testing

class ConfigProduccion(Config):
    """Configuración para producción"""
    DEBUG = False

# Configuración activa
config = ConfigDesarrollo()
```

---

## Utilidades básicas

**utils/calculos.py**

```python
"""
Funciones auxiliares para cálculos del juego
"""

import random
from typing import Tuple

def calcular_xp_requerida(nivel: int) -> int:
    """
    Calcula XP necesaria para subir de nivel
    
    Args:
        nivel: Nivel actual
    
    Returns:
        Cantidad de XP necesaria
    """
    from config import config
    base = config.EXPERIENCIA_BASE
    multiplicador = config.MULTIPLICADOR_XP
    
    return int(base * (multiplicador ** (nivel - 1)))

def calcular_daño_base(atacante_fuerza: int, defensor_defensa: int) -> int:
    """
    Calcula daño base de un ataque físico
    
    Args:
        atacante_fuerza: Fuerza del atacante
        defensor_defensa: Defensa del defensor
    
    Returns:
        Daño calculado
    """
    # Fórmula: (Fuerza * 2) - (Defensa / 2)
    daño = (atacante_fuerza * 2) - (defensor_defensa / 2)
    return max(1, int(daño))  # Mínimo 1 de daño

def calcular_daño_magico(atacante_int: int, defensor_defensa: int) -> int:
    """
    Calcula daño mágico
    
    Args:
        atacante_int: Inteligencia del atacante
        defensor_defensa: Defensa del defensor
    
    Returns:
        Daño mágico calculado
    """
    # La defensa reduce menos el daño mágico
    daño = (atacante_int * 2.5) - (defensor_defensa / 4)
    return max(1, int(daño))

def calcular_probabilidad_critico(agilidad: int) -> float:
    """
    Calcula probabilidad de crítico basada en agilidad
    
    Args:
        agilidad: Agilidad del personaje
    
    Returns:
        Probabilidad de crítico (0-1)
    """
    from config import config
    base = config.CRITICO_PROBABILIDAD
    bonus = agilidad / 1000  # +0.1% por punto de agilidad
    return min(0.5, base + bonus)  # Máximo 50%

def es_critico(agilidad: int) -> bool:
    """
    Determina si un ataque es crítico
    
    Args:
        agilidad: Agilidad del personaje
    
    Returns:
        True si es crítico
    """
    probabilidad = calcular_probabilidad_critico(agilidad)
    return random.random() < probabilidad

def aplicar_critico(daño: int) -> int:
    """
    Aplica multiplicador de crítico
    
    Args:
        daño: Daño base
    
    Returns:
        Daño con crítico aplicado
    """
    from config import config
    return int(daño * config.CRITICO_MULTIPLICADOR)

def calcular_atributos_por_nivel(nivel: int, atributo_base: int, bonificador: float) -> int:
    """
    Calcula valor de atributo según nivel
    
    Args:
        nivel: Nivel del personaje
        atributo_base: Valor base del atributo
        bonificador: Multiplicador de clase
    
    Returns:
        Valor del atributo
    """
    # Aumenta un 10% por nivel
    incremento_por_nivel = atributo_base * 0.1 * (nivel - 1)
    valor = (atributo_base + incremento_por_nivel) * bonificador
    return int(valor)

def generar_nombre_aleatorio(clase: str) -> str:
    """
    Genera un nombre aleatorio según la clase
    
    Args:
        clase: Clase del personaje
    
    Returns:
        Nombre aleatorio
    """
    nombres = {
        'guerrero': ['Thorin', 'Ragnar', 'Conan', 'Beowulf', 'Grom'],
        'mago': ['Gandalf', 'Merlin', 'Raistlin', 'Medivh', 'Jaina'],
        'arquero': ['Legolas', 'Artemis', 'Robin', 'Hawkeye', 'Katniss']
    }
    
    lista_nombres = nombres.get(clase, ['Aventurero'])
    return random.choice(lista_nombres)
```

---

## Tests básicos de utilidades

**test_calculos.py** (opcional)

```python
"""
Tests para funciones de cálculo
"""

from utils.calculos import (
    calcular_xp_requerida,
    calcular_daño_base,
    calcular_daño_magico,
    es_critico,
    aplicar_critico
)

def test_xp_requerida():
    """Test cálculo de XP"""
    xp_nivel_1 = calcular_xp_requerida(1)
    xp_nivel_2 = calcular_xp_requerida(2)
    xp_nivel_3 = calcular_xp_requerida(3)
    
    print(f"Nivel 1: {xp_nivel_1} XP")
    print(f"Nivel 2: {xp_nivel_2} XP")
    print(f"Nivel 3: {xp_nivel_3} XP")
    
    assert xp_nivel_2 > xp_nivel_1
    assert xp_nivel_3 > xp_nivel_2

def test_daño():
    """Test cálculo de daño"""
    # Guerrero fuerte (100 fuerza) vs defensor débil (20 defensa)
    daño = calcular_daño_base(100, 20)
    print(f"Daño físico: {daño}")
    assert daño > 0
    
    # Mago inteligente (100 int) vs defensor débil (20 defensa)
    daño_magico = calcular_daño_magico(100, 20)
    print(f"Daño mágico: {daño_magico}")
    assert daño_magico > 0

def test_critico():
    """Test sistema de críticos"""
    agilidad_baja = 10
    agilidad_alta = 200
    
    criticos_bajo = sum(1 for _ in range(100) if es_critico(agilidad_baja))
    criticos_alto = sum(1 for _ in range(100) if es_critico(agilidad_alta))
    
    print(f"Críticos con agilidad baja: {criticos_bajo}/100")
    print(f"Críticos con agilidad alta: {criticos_alto}/100")
    
    assert criticos_alto >= criticos_bajo

def test_aplicar_critico():
    """Test multiplicador de crítico"""
    daño_normal = 50
    daño_critico = aplicar_critico(daño_normal)
    
    print(f"Daño normal: {daño_normal}")
    print(f"Daño crítico: {daño_critico}")
    
    assert daño_critico == 100  # x2

if __name__ == '__main__':
    print("=== Tests de Cálculos ===\n")
    test_xp_requerida()
    print()
    test_daño()
    print()
    test_critico()
    print()
    test_aplicar_critico()
    print("\n✅ Todos los tests pasaron")
```

---

## Servidor HTTP básico

**app.py**

```python
"""
Servidor HTTP vanilla para el juego RPG
"""

from http.server import BaseHTTPRequestHandler, HTTPServer
import json
from config import config

class RPGHandler(BaseHTTPRequestHandler):
    """Handler HTTP para el juego RPG"""
    
    def do_GET(self):
        """Maneja peticiones GET"""
        if self.path == '/':
            self.send_response(200)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            
            response = {
                'nombre': config.NOMBRE_JUEGO,
                'version': config.VERSION,
                'mensaje': 'Bienvenido a Fantasy Battle Arena API'
            }
            self.wfile.write(json.dumps(response).encode())
            
        elif self.path == '/api/info':
            self.send_response(200)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            
            response = {
                'clases_disponibles': config.CLASES_DISPONIBLES,
                'nivel_inicial': config.NIVEL_INICIAL,
                'vida_base': config.VIDA_BASE,
                'slots_inventario': config.SLOTS_INVENTARIO
            }
            self.wfile.write(json.dumps(response).encode())
            
        else:
            self.send_response(404)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            error = {'error': 'Ruta no encontrada'}
            self.wfile.write(json.dumps(error).encode())

if __name__ == '__main__':
    puerto = 8000
    servidor = HTTPServer(('localhost', puerto), RPGHandler)
    print(f"🎮 Iniciando {config.NOMBRE_JUEGO} v{config.VERSION}")
    print(f"🚀 Servidor en http://localhost:{puerto}")
    servidor.serve_forever()
```

---

## Ejecutar el proyecto

```powershell
# Ejecutar servidor
python app.py
```

**Probar la API:**

```powershell
# Información del juego
curl http://localhost:5000/

# Configuración
curl http://localhost:5000/api/info
```

**Respuesta esperada:**

```json
{
  "nombre": "Fantasy Battle Arena",
  "version": "1.0.0",
  "mensaje": "Bienvenido a Fantasy Battle Arena API"
}
```

---

## Diseño del modelo de dominio

### Diagrama de clases (simplificado)

```
┌─────────────────────┐
│    Personaje        │ (abstracta)
├─────────────────────┤
│ - id                │
│ - nombre            │
│ - nivel             │
│ - experiencia       │
│ - vida              │
│ - atributos         │
│ - inventario        │
├─────────────────────┤
│ + atacar()          │
│ + recibir_daño()    │
│ + usar_habilidad()  │
│ + subir_nivel()     │
└──────────┬──────────┘
           │
    ┌──────┴───────┬─────────┐
    │              │         │
┌───▼────┐   ┌────▼───┐ ┌──▼────┐
│Guerrero│   │  Mago  │ │Arquero│
└────────┘   └────────┘ └───────┘
```

### Relaciones

- Personaje **tiene** Inventario (composición)
- Inventario **contiene** Items (agregación)
- Combate **usa** dos Personajes
- GestorPersonajes **gestiona** Personajes (Repository)
- GestorCombates **gestiona** Combates (Service)

---

## Principios aplicados

### SOLID

✅ **SRP:** Cada clase tiene una responsabilidad clara
- `Personaje`: Representar un personaje
- `GestorPersonajes`: Gestionar personajes
- `Combate`: Gestionar un combate

✅ **OCP:** Extensible sin modificar
- Nuevas clases de personajes sin tocar código existente
- Nuevos tipos de items sin modificar inventario

✅ **LSP:** Subclases sustituibles
- `Guerrero`, `Mago`, `Arquero` son intercambiables

✅ **ISP:** Interfaces específicas
- `Atacante`, `Defendible`, `Equipable`

✅ **DIP:** Dependencias de abstracciones
- Servicios dependen de interfaces, no implementaciones

### Patrones de diseño

✅ **Factory:** Crear personajes según clase
✅ **Repository:** Separar persistencia
✅ **Strategy:** Diferentes estrategias de combate
✅ **Observer:** Notificar eventos del juego

---

## Próximos pasos

En el siguiente módulo crearemos:

1. Clase `Personaje` base
2. Clases especializadas: `Guerrero`, `Mago`, `Arquero`
3. Sistema de `Item` y `Equipamiento`
4. `Inventario` con gestión de items
5. Sistema de atributos y estadísticas

**Siguiente:** [Personajes y entidades](./20-personajes-entidades.md)

---

## Conclusión

Has aprendido:

- ✅ Visión del proyecto RPG
- ✅ Arquitectura del sistema
- ✅ Entidades del dominio
- ✅ Setup inicial del proyecto
- ✅ Configuración y utilidades
- ✅ Aplicación Flask básica
- ✅ Principios y patrones aplicados

¡Ahora construiremos el sistema de personajes! 🎮

---

## Recursos adicionales

- **[RPG Game Design](https://www.gamasutra.com/)** - Diseño de juegos
- **[Game Programming Patterns](https://gameprogrammingpatterns.com/)** - Patrones para juegos

El proyecto RPG aplica todos los conceptos de POO. 🚀
