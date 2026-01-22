# Personajes y entidades del juego

> **Construye el modelo de dominio del RPG**

---

## Clase Personaje (base)

La clase `Personaje` es la base de todos los personajes del juego.

**modelos/personaje.py**

```python
"""
Modelo de dominio: Personaje
"""

from abc import ABC, abstractmethod
from typing import Optional, List
from config import config
from utils.calculos import calcular_atributos_por_nivel, calcular_xp_requerida

class Personaje(ABC):
    """Clase base abstracta para personajes del juego"""
    
    def __init__(
        self,
        id: int,
        nombre: str,
        clase: str,
        nivel: int = 1
    ):
        self.id = id
        self.nombre = nombre
        self.clase = clase
        self.nivel = nivel
        self.experiencia = 0
        self.experiencia_siguiente_nivel = calcular_xp_requerida(nivel)
        
        # Atributos base (antes de bonificadores)
        self._fuerza_base = 10
        self._inteligencia_base = 10
        self._agilidad_base = 10
        self._defensa_base = 10
        
        # Calcular atributos con bonificadores
        self._calcular_atributos()
        
        # Vida y mana
        self.vida_maxima = self._calcular_vida_maxima()
        self.vida = self.vida_maxima
        self.mana_maxima = self._calcular_mana_maxima()
        self.mana = self.mana_maxima
        
        # Inventario
        self.inventario: List['Item'] = []
        self.arma_equipada: Optional['Item'] = None
        self.armadura_equipada: Optional['Item'] = None
        
        # Estado
        self.vivo = True
    
    def _calcular_atributos(self):
        """Calcula atributos con bonificadores de clase"""
        bonificadores = config.BONIFICADORES_CLASE[self.clase]
        
        self.fuerza = calcular_atributos_por_nivel(
            self.nivel, self._fuerza_base, bonificadores['fuerza']
        )
        self.inteligencia = calcular_atributos_por_nivel(
            self.nivel, self._inteligencia_base, bonificadores['inteligencia']
        )
        self.agilidad = calcular_atributos_por_nivel(
            self.nivel, self._agilidad_base, bonificadores['agilidad']
        )
        self.defensa = calcular_atributos_por_nivel(
            self.nivel, self._defensa_base, bonificadores['defensa']
        )
    
    def _calcular_vida_maxima(self) -> int:
        """Calcula vida máxima según nivel y bonificadores"""
        bonificador = config.BONIFICADORES_CLASE[self.clase]['vida']
        vida_base = config.VIDA_BASE
        return int(vida_base * bonificador * (1 + (self.nivel - 1) * 0.1))
    
    def _calcular_mana_maxima(self) -> int:
        """Calcula mana máximo según inteligencia"""
        return int(config.MANA_BASE + (self.inteligencia * 2))
    
    @abstractmethod
    def usar_habilidad(self, objetivo: 'Personaje') -> dict:
        """
        Usa habilidad especial de la clase
        
        Args:
            objetivo: Personaje objetivo
        
        Returns:
            Resultado de la habilidad
        """
        pass
    
    @abstractmethod
    def obtener_descripcion_habilidad(self) -> str:
        """Devuelve la descripción de la habilidad especial"""
        pass
    
    def atacar(self, objetivo: 'Personaje') -> dict:
        """
        Ataque básico físico
        
        Args:
            objetivo: Personaje a atacar
        
        Returns:
            Resultado del ataque
        """
        from utils.calculos import calcular_daño_base, es_critico, aplicar_critico
        
        if not self.vivo:
            return {'exito': False, 'mensaje': f'{self.nombre} está muerto'}
        
        if not objetivo.vivo:
            return {'exito': False, 'mensaje': f'{objetivo.nombre} ya está muerto'}
        
        # Calcular daño
        daño = calcular_daño_base(self.fuerza, objetivo.defensa)
        
        # Añadir daño del arma equipada
        if self.arma_equipada:
            daño += self.arma_equipada.valor
        
        # Verificar crítico
        critico = es_critico(self.agilidad)
        if critico:
            daño = aplicar_critico(daño)
        
        # Aplicar daño
        daño_real = objetivo.recibir_daño(daño)
        
        return {
            'exito': True,
            'atacante': self.nombre,
            'objetivo': objetivo.nombre,
            'daño': daño_real,
            'critico': critico,
            'objetivo_muerto': not objetivo.vivo,
            'mensaje': self._generar_mensaje_ataque(objetivo, daño_real, critico)
        }
    
    def _generar_mensaje_ataque(self, objetivo: 'Personaje', daño: int, critico: bool) -> str:
        """Genera mensaje de ataque"""
        mensaje = f"{self.nombre} ataca a {objetivo.nombre}"
        if critico:
            mensaje += " ¡CRÍTICO!"
        mensaje += f" causando {daño} de daño"
        if not objetivo.vivo:
            mensaje += f". {objetivo.nombre} ha sido derrotado!"
        return mensaje
    
    def recibir_daño(self, daño: int) -> int:
        """
        Recibe daño y actualiza estado
        
        Args:
            daño: Cantidad de daño a recibir
        
        Returns:
            Daño real recibido (después de defensa)
        """
        if not self.vivo:
            return 0
        
        # Reducir daño por armadura equipada
        if self.armadura_equipada:
            daño = max(1, daño - self.armadura_equipada.valor)
        
        # Aplicar daño
        self.vida -= daño
        
        # Verificar muerte
        if self.vida <= 0:
            self.vida = 0
            self.vivo = False
        
        return daño
    
    def curar(self, cantidad: int):
        """Restaura vida"""
        if self.vivo:
            self.vida = min(self.vida_maxima, self.vida + cantidad)
    
    def restaurar_mana(self, cantidad: int):
        """Restaura mana"""
        self.mana = min(self.mana_maxima, self.mana + cantidad)
    
    def ganar_experiencia(self, xp: int) -> bool:
        """
        Gana experiencia y verifica si sube de nivel
        
        Args:
            xp: Cantidad de experiencia ganada
        
        Returns:
            True si subió de nivel
        """
        self.experiencia += xp
        
        if self.experiencia >= self.experiencia_siguiente_nivel:
            self.subir_nivel()
            return True
        
        return False
    
    def subir_nivel(self):
        """Sube de nivel y mejora atributos"""
        self.nivel += 1
        self.experiencia = 0
        self.experiencia_siguiente_nivel = calcular_xp_requerida(self.nivel)
        
        # Recalcular atributos
        self._calcular_atributos()
        
        # Actualizar vida y mana máximos
        vida_anterior = self.vida_maxima
        mana_anterior = self.mana_maxima
        
        self.vida_maxima = self._calcular_vida_maxima()
        self.mana_maxima = self._calcular_mana_maxima()
        
        # Curar según el aumento
        self.vida += (self.vida_maxima - vida_anterior)
        self.mana += (self.mana_maxima - mana_anterior)
        
        print(f"🎉 {self.nombre} subió al nivel {self.nivel}!")
    
    def agregar_item(self, item: 'Item') -> bool:
        """
        Añade item al inventario
        
        Args:
            item: Item a añadir
        
        Returns:
            True si se añadió exitosamente
        """
        if len(self.inventario) >= config.SLOTS_INVENTARIO:
            return False
        
        self.inventario.append(item)
        return True
    
    def usar_item(self, indice: int) -> dict:
        """
        Usa un item del inventario
        
        Args:
            indice: Índice del item en el inventario
        
        Returns:
            Resultado del uso
        """
        if indice < 0 or indice >= len(self.inventario):
            return {'exito': False, 'mensaje': 'Item no encontrado'}
        
        item = self.inventario[indice]
        
        if item.tipo == 'pocion_vida':
            self.curar(item.valor)
            self.inventario.pop(indice)
            return {
                'exito': True,
                'mensaje': f'{self.nombre} usó {item.nombre} y recuperó {item.valor} HP'
            }
        
        elif item.tipo == 'pocion_mana':
            self.restaurar_mana(item.valor)
            self.inventario.pop(indice)
            return {
                'exito': True,
                'mensaje': f'{self.nombre} usó {item.nombre} y recuperó {item.valor} MP'
            }
        
        else:
            return {'exito': False, 'mensaje': 'Este item no se puede usar directamente'}
    
    def equipar_item(self, indice: int) -> dict:
        """
        Equipa un item (arma o armadura)
        
        Args:
            indice: Índice del item en el inventario
        
        Returns:
            Resultado de equipar
        """
        if indice < 0 or indice >= len(self.inventario):
            return {'exito': False, 'mensaje': 'Item no encontrado'}
        
        item = self.inventario[indice]
        
        if item.tipo == 'arma':
            # Desequipar arma anterior
            if self.arma_equipada:
                self.inventario.append(self.arma_equipada)
            
            self.arma_equipada = item
            self.inventario.pop(indice)
            return {
                'exito': True,
                'mensaje': f'{self.nombre} equipó {item.nombre} (+{item.valor} ataque)'
            }
        
        elif item.tipo == 'armadura':
            # Desequipar armadura anterior
            if self.armadura_equipada:
                self.inventario.append(self.armadura_equipada)
            
            self.armadura_equipada = item
            self.inventario.pop(indice)
            return {
                'exito': True,
                'mensaje': f'{self.nombre} equipó {item.nombre} (+{item.valor} defensa)'
            }
        
        else:
            return {'exito': False, 'mensaje': 'Este item no se puede equipar'}
    
    def to_dict(self) -> dict:
        """Serializa personaje a diccionario"""
        return {
            'id': self.id,
            'nombre': self.nombre,
            'clase': self.clase,
            'nivel': self.nivel,
            'experiencia': self.experiencia,
            'experiencia_siguiente_nivel': self.experiencia_siguiente_nivel,
            'vida': self.vida,
            'vida_maxima': self.vida_maxima,
            'mana': self.mana,
            'mana_maxima': self.mana_maxima,
            'fuerza': self.fuerza,
            'inteligencia': self.inteligencia,
            'agilidad': self.agilidad,
            'defensa': self.defensa,
            'vivo': self.vivo,
            'arma_equipada': self.arma_equipada.to_dict() if self.arma_equipada else None,
            'armadura_equipada': self.armadura_equipada.to_dict() if self.armadura_equipada else None,
            'inventario': [item.to_dict() for item in self.inventario],
            'habilidad': self.obtener_descripcion_habilidad()
        }
    
    def __str__(self):
        return f"{self.nombre} (Nivel {self.nivel} {self.clase.title()})"
```

---

## Clase Guerrero

**modelos/personaje.py** (continuación)

```python
class Guerrero(Personaje):
    """Guerrero - Alta fuerza y vida, baja inteligencia"""
    
    def __init__(self, id: int, nombre: str, nivel: int = 1):
        super().__init__(id, nombre, 'guerrero', nivel)
    
    def usar_habilidad(self, objetivo: 'Personaje') -> dict:
        """
        Golpe Poderoso - Ataque devastador con 50% más daño
        Costo: 20 mana
        """
        costo_mana = 20
        
        if not self.vivo:
            return {'exito': False, 'mensaje': f'{self.nombre} está muerto'}
        
        if self.mana < costo_mana:
            return {'exito': False, 'mensaje': 'Mana insuficiente'}
        
        if not objetivo.vivo:
            return {'exito': False, 'mensaje': f'{objetivo.nombre} ya está muerto'}
        
        # Consumir mana
        self.mana -= costo_mana
        
        # Calcular daño (150% del ataque normal)
        from utils.calculos import calcular_daño_base
        daño = int(calcular_daño_base(self.fuerza, objetivo.defensa) * 1.5)
        
        if self.arma_equipada:
            daño += self.arma_equipada.valor
        
        # Aplicar daño
        daño_real = objetivo.recibir_daño(daño)
        
        return {
            'exito': True,
            'atacante': self.nombre,
            'objetivo': objetivo.nombre,
            'daño': daño_real,
            'habilidad': 'Golpe Poderoso',
            'objetivo_muerto': not objetivo.vivo,
            'mensaje': f"⚔️ {self.nombre} usa GOLPE PODEROSO contra {objetivo.nombre} "
                      f"causando {daño_real} de daño!"
        }
    
    def obtener_descripcion_habilidad(self) -> str:
        return "Golpe Poderoso: Ataque devastador con 50% más daño (20 mana)"
```

---

## Clase Mago

```python
class Mago(Personaje):
    """Mago - Alta inteligencia y mana, baja vida"""
    
    def __init__(self, id: int, nombre: str, nivel: int = 1):
        super().__init__(id, nombre, 'mago', nivel)
    
    def usar_habilidad(self, objetivo: 'Personaje') -> dict:
        """
        Bola de Fuego - Ataque mágico poderoso
        Costo: 30 mana
        """
        costo_mana = 30
        
        if not self.vivo:
            return {'exito': False, 'mensaje': f'{self.nombre} está muerto'}
        
        if self.mana < costo_mana:
            return {'exito': False, 'mensaje': 'Mana insuficiente'}
        
        if not objetivo.vivo:
            return {'exito': False, 'mensaje': f'{objetivo.nombre} ya está muerto'}
        
        # Consumir mana
        self.mana -= costo_mana
        
        # Calcular daño mágico
        from utils.calculos import calcular_daño_magico
        daño = calcular_daño_magico(self.inteligencia, objetivo.defensa)
        
        # El arma suma menos al daño mágico
        if self.arma_equipada:
            daño += int(self.arma_equipada.valor * 0.5)
        
        # Aplicar daño
        daño_real = objetivo.recibir_daño(daño)
        
        return {
            'exito': True,
            'atacante': self.nombre,
            'objetivo': objetivo.nombre,
            'daño': daño_real,
            'habilidad': 'Bola de Fuego',
            'objetivo_muerto': not objetivo.vivo,
            'mensaje': f"🔥 {self.nombre} lanza BOLA DE FUEGO contra {objetivo.nombre} "
                      f"causando {daño_real} de daño mágico!"
        }
    
    def obtener_descripcion_habilidad(self) -> str:
        return "Bola de Fuego: Ataque mágico devastador (30 mana)"
```

---

## Clase Arquero

```python
class Arquero(Personaje):
    """Arquero - Alta agilidad, balanceado"""
    
    def __init__(self, id: int, nombre: str, nivel: int = 1):
        super().__init__(id, nombre, 'arquero', nivel)
    
    def usar_habilidad(self, objetivo: 'Personaje') -> dict:
        """
        Disparo Múltiple - 3 ataques rápidos
        Costo: 25 mana
        """
        costo_mana = 25
        
        if not self.vivo:
            return {'exito': False, 'mensaje': f'{self.nombre} está muerto'}
        
        if self.mana < costo_mana:
            return {'exito': False, 'mensaje': 'Mana insuficiente'}
        
        if not objetivo.vivo:
            return {'exito': False, 'mensaje': f'{objetivo.nombre} ya está muerto'}
        
        # Consumir mana
        self.mana -= costo_mana
        
        # 3 disparos de 60% daño cada uno
        from utils.calculos import calcular_daño_base, es_critico, aplicar_critico
        
        daño_total = 0
        disparos = []
        
        for i in range(3):
            if not objetivo.vivo:
                break
            
            daño = int(calcular_daño_base(self.fuerza, objetivo.defensa) * 0.6)
            
            if self.arma_equipada:
                daño += int(self.arma_equipada.valor * 0.6)
            
            # Cada disparo puede ser crítico
            critico = es_critico(self.agilidad)
            if critico:
                daño = aplicar_critico(daño)
            
            daño_real = objetivo.recibir_daño(daño)
            daño_total += daño_real
            disparos.append({'daño': daño_real, 'critico': critico})
        
        return {
            'exito': True,
            'atacante': self.nombre,
            'objetivo': objetivo.nombre,
            'daño': daño_total,
            'disparos': disparos,
            'habilidad': 'Disparo Múltiple',
            'objetivo_muerto': not objetivo.vivo,
            'mensaje': f"🏹 {self.nombre} usa DISPARO MÚLTIPLE contra {objetivo.nombre} "
                      f"causando {daño_total} de daño total!"
        }
    
    def obtener_descripcion_habilidad(self) -> str:
        return "Disparo Múltiple: 3 ataques rápidos (25 mana)"
```

---

## Sistema de Items

**modelos/item.py**

```python
"""
Sistema de items del juego
"""

class Item:
    """Representa un item del juego"""
    
    def __init__(
        self,
        id: int,
        nombre: str,
        tipo: str,
        valor: int,
        descripcion: str = ""
    ):
        """
        Args:
            id: ID único del item
            nombre: Nombre del item
            tipo: arma, armadura, pocion_vida, pocion_mana
            valor: Valor del item (daño, defensa, curación)
            descripcion: Descripción del item
        """
        self.id = id
        self.nombre = nombre
        self.tipo = tipo
        self.valor = valor
        self.descripcion = descripcion
    
    def to_dict(self) -> dict:
        """Serializa a diccionario"""
        return {
            'id': self.id,
            'nombre': self.nombre,
            'tipo': self.tipo,
            'valor': self.valor,
            'descripcion': self.descripcion
        }
    
    def __str__(self):
        return f"{self.nombre} ({self.tipo}, +{self.valor})"

# Items predefinidos
ITEMS_DISPONIBLES = {
    # Armas
    1: Item(1, 'Espada de Hierro', 'arma', 15, 'Una espada básica pero confiable'),
    2: Item(2, 'Espada de Acero', 'arma', 25, 'Espada de acero bien forjada'),
    3: Item(3, 'Báculo Mágico', 'arma', 20, 'Báculo que potencia la magia'),
    4: Item(4, 'Arco Largo', 'arma', 18, 'Arco de largo alcance'),
    5: Item(5, 'Espada Legendaria', 'arma', 50, 'Espada de poder legendario'),
    
    # Armaduras
    10: Item(10, 'Armadura de Cuero', 'armadura', 10, 'Protección ligera'),
    11: Item(11, 'Armadura de Malla', 'armadura', 20, 'Armadura de malla metálica'),
    12: Item(12, 'Armadura de Placas', 'armadura', 35, 'Armadura pesada de placas'),
    13: Item(13, 'Túnica Mágica', 'armadura', 15, 'Túnica con protección mágica'),
    
    # Pociones
    20: Item(20, 'Poción de Vida Pequeña', 'pocion_vida', 30, 'Restaura 30 HP'),
    21: Item(21, 'Poción de Vida Grande', 'pocion_vida', 100, 'Restaura 100 HP'),
    22: Item(22, 'Poción de Mana Pequeña', 'pocion_mana', 20, 'Restaura 20 MP'),
    23: Item(23, 'Poción de Mana Grande', 'pocion_mana', 50, 'Restaura 50 MP'),
}

def obtener_item(item_id: int) -> Item:
    """
    Obtiene una copia de un item por ID
    
    Args:
        item_id: ID del item
    
    Returns:
        Copia del item
    """
    if item_id not in ITEMS_DISPONIBLES:
        raise ValueError(f"Item {item_id} no existe")
    
    item_original = ITEMS_DISPONIBLES[item_id]
    return Item(
        item_original.id,
        item_original.nombre,
        item_original.tipo,
        item_original.valor,
        item_original.descripcion
    )
```

---

## Factory de personajes

**modelos/personaje.py** (añadir al final)

```python
class PersonajeFactory:
    """Factory para crear personajes"""
    
    _contador_id = 0
    
    @classmethod
    def crear_personaje(cls, nombre: str, clase: str, nivel: int = 1) -> Personaje:
        """
        Crea un personaje según la clase
        
        Args:
            nombre: Nombre del personaje
            clase: guerrero, mago, arquero
            nivel: Nivel inicial
        
        Returns:
            Instancia de la clase correspondiente
        """
        cls._contador_id += 1
        
        if clase == 'guerrero':
            return Guerrero(cls._contador_id, nombre, nivel)
        elif clase == 'mago':
            return Mago(cls._contador_id, nombre, nivel)
        elif clase == 'arquero':
            return Arquero(cls._contador_id, nombre, nivel)
        else:
            raise ValueError(f"Clase '{clase}' no válida. Usa: guerrero, mago, arquero")
    
    @classmethod
    def reiniciar_contador(cls):
        """Reinicia el contador de IDs (para testing)"""
        cls._contador_id = 0
```

---

## Pruebas de los personajes

**test_personajes.py**

```python
"""
Tests del sistema de personajes
"""

from modelos.personaje import PersonajeFactory
from modelos.item import obtener_item

def test_crear_personajes():
    """Test creación de personajes"""
    print("=== Test: Crear Personajes ===\n")
    
    guerrero = PersonajeFactory.crear_personaje("Conan", "guerrero")
    mago = PersonajeFactory.crear_personaje("Gandalf", "mago")
    arquero = PersonajeFactory.crear_personaje("Legolas", "arquero")
    
    print(f"Guerrero: {guerrero}")
    print(f"  Vida: {guerrero.vida}/{guerrero.vida_maxima}")
    print(f"  Fuerza: {guerrero.fuerza}, Defensa: {guerrero.defensa}")
    print()
    
    print(f"Mago: {mago}")
    print(f"  Vida: {mago.vida}/{mago.vida_maxima}")
    print(f"  Inteligencia: {mago.inteligencia}, Mana: {mago.mana}/{mago.mana_maxima}")
    print()
    
    print(f"Arquero: {arquero}")
    print(f"  Vida: {arquero.vida}/{arquero.vida_maxima}")
    print(f"  Agilidad: {arquero.agilidad}")
    print()

def test_combate_basico():
    """Test combate básico"""
    print("=== Test: Combate Básico ===\n")
    
    guerrero = PersonajeFactory.crear_personaje("Thorin", "guerrero")
    mago = PersonajeFactory.crear_personaje("Merlin", "mago")
    
    print(f"{guerrero.nombre}: {guerrero.vida} HP")
    print(f"{mago.nombre}: {mago.vida} HP\n")
    
    # Guerrero ataca
    resultado = guerrero.atacar(mago)
    print(resultado['mensaje'])
    print(f"{mago.nombre} tiene {mago.vida} HP restantes\n")
    
    # Mago contraataca
    resultado = mago.atacar(guerrero)
    print(resultado['mensaje'])
    print(f"{guerrero.nombre} tiene {guerrero.vida} HP restantes\n")

def test_habilidades():
    """Test habilidades especiales"""
    print("=== Test: Habilidades Especiales ===\n")
    
    guerrero = PersonajeFactory.crear_personaje("Ragnar", "guerrero", nivel=5)
    mago = PersonajeFactory.crear_personaje("Raistlin", "mago", nivel=5)
    arquero = PersonajeFactory.crear_personaje("Robin", "arquero", nivel=5)
    
    enemigo = PersonajeFactory.crear_personaje("Orco", "guerrero", nivel=3)
    
    print(f"Enemigo: {enemigo.vida} HP\n")
    
    # Golpe Poderoso
    resultado = guerrero.usar_habilidad(enemigo)
    print(resultado['mensaje'])
    print(f"Enemigo: {enemigo.vida} HP\n")
    
    # Bola de Fuego
    resultado = mago.usar_habilidad(enemigo)
    print(resultado['mensaje'])
    print(f"Enemigo: {enemigo.vida} HP\n")
    
    # Disparo Múltiple
    enemigo2 = PersonajeFactory.crear_personaje("Goblin", "arquero", nivel=3)
    resultado = arquero.usar_habilidad(enemigo2)
    print(resultado['mensaje'])
    print(f"  Disparos: {len(resultado['disparos'])}")
    print(f"Enemigo: {enemigo2.vida} HP\n")

def test_items():
    """Test sistema de items"""
    print("=== Test: Sistema de Items ===\n")
    
    guerrero = PersonajeFactory.crear_personaje("Arthur", "guerrero")
    
    print(f"{guerrero.nombre} - Inventario inicial: {len(guerrero.inventario)} items\n")
    
    # Añadir items
    espada = obtener_item(2)  # Espada de Acero
    armadura = obtener_item(11)  # Armadura de Malla
    pocion = obtener_item(20)  # Poción de Vida Pequeña
    
    guerrero.agregar_item(espada)
    guerrero.agregar_item(armadura)
    guerrero.agregar_item(pocion)
    
    print(f"Items añadidos: {len(guerrero.inventario)}")
    for i, item in enumerate(guerrero.inventario):
        print(f"  {i}: {item}")
    print()
    
    # Equipar arma
    resultado = guerrero.equipar_item(0)
    print(resultado['mensaje'])
    print(f"Arma equipada: {guerrero.arma_equipada}\n")
    
    # Equipar armadura
    resultado = guerrero.equipar_item(0)  # Ahora la armadura está en índice 0
    print(resultado['mensaje'])
    print(f"Armadura equipada: {guerrero.armadura_equipada}\n")
    
    # Recibir daño
    guerrero.recibir_daño(50)
    print(f"HP después de daño: {guerrero.vida}/{guerrero.vida_maxima}\n")
    
    # Usar poción
    resultado = guerrero.usar_item(0)  # Poción está en índice 0
    print(resultado['mensaje'])
    print(f"HP después de poción: {guerrero.vida}/{guerrero.vida_maxima}\n")

def test_experiencia():
    """Test sistema de experiencia"""
    print("=== Test: Sistema de Experiencia ===\n")
    
    personaje = PersonajeFactory.crear_personaje("Héroe", "guerrero")
    
    print(f"{personaje.nombre} - Nivel {personaje.nivel}")
    print(f"XP: {personaje.experiencia}/{personaje.experiencia_siguiente_nivel}")
    print(f"Fuerza: {personaje.fuerza}, Vida: {personaje.vida_maxima}\n")
    
    # Ganar experiencia
    xp_ganada = 100
    subio = personaje.ganar_experiencia(xp_ganada)
    
    if subio:
        print(f"XP: {personaje.experiencia}/{personaje.experiencia_siguiente_nivel}")
        print(f"Fuerza: {personaje.fuerza}, Vida: {personaje.vida_maxima}\n")

if __name__ == '__main__':
    PersonajeFactory.reiniciar_contador()
    
    test_crear_personajes()
    test_combate_basico()
    test_habilidades()
    test_items()
    test_experiencia()
    
    print("✅ Todos los tests completados")
```

**Ejecutar tests:**

```powershell
python test_personajes.py
```

---

## Conclusión

Has creado:

- ✅ Clase `Personaje` base con todos los atributos y comportamientos
- ✅ Clases especializadas: `Guerrero`, `Mago`, `Arquero`
- ✅ Sistema de habilidades únicas por clase
- ✅ Sistema de items (armas, armaduras, pociones)
- ✅ Sistema de inventario y equipamiento
- ✅ Sistema de experiencia y niveles
- ✅ Factory para crear personajes
- ✅ Tests completos del sistema

**Siguiente:** [Sistema de combate](./21-sistema-combate.md)

---

## Recursos adicionales

- **[RPG Stat Systems](https://gamedevelopment.tutsplus.com/)** - Sistemas de estadísticas
- **[Game Balance](https://www.gamasutra.com/blogs/IanSchreiber/20090720/84766/Game_Balance_Concepts.php)** - Balance de juegos

El modelo de dominio está completo y funcional. 🎮
