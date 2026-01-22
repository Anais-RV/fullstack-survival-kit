# Sistema de combate del juego

> **Implementa mecánicas de combate por turnos**

---

## Clase Combate

El sistema de combate gestiona peleas por turnos entre personajes.

**modelos/combate.py**

```python
"""
Sistema de combate del juego RPG
"""

from typing import List, Optional
from enum import Enum
from modelos.personaje import Personaje

class EstadoCombate(Enum):
    """Estados posibles del combate"""
    INICIANDO = 'iniciando'
    EN_CURSO = 'en_curso'
    FINALIZADO = 'finalizado'

class AccionCombate(Enum):
    """Acciones disponibles en combate"""
    ATACAR = 'atacar'
    HABILIDAD = 'habilidad'
    USAR_ITEM = 'usar_item'
    HUIR = 'huir'

class Combate:
    """Gestiona un combate por turnos"""
    
    def __init__(
        self,
        id: int,
        personaje1: Personaje,
        personaje2: Personaje
    ):
        self.id = id
        self.personaje1 = personaje1
        self.personaje2 = personaje2
        self.estado = EstadoCombate.INICIANDO
        self.turno_actual = 1
        self.turno_de = personaje1  # Quien tiene el turno
        self.log: List[dict] = []
        self.ganador: Optional[Personaje] = None
        
        # Determinar orden de turnos por agilidad
        if personaje2.agilidad > personaje1.agilidad:
            self.turno_de = personaje2
        
        self._registrar_log({
            'tipo': 'inicio',
            'mensaje': f"¡Combate iniciado! {personaje1.nombre} vs {personaje2.nombre}",
            'turno': 0
        })
    
    def _registrar_log(self, entrada: dict):
        """Registra una entrada en el log del combate"""
        entrada['turno'] = self.turno_actual
        self.log.append(entrada)
    
    def _cambiar_turno(self):
        """Cambia el turno al otro personaje"""
        if self.turno_de == self.personaje1:
            self.turno_de = self.personaje2
        else:
            self.turno_de = self.personaje1
            self.turno_actual += 1  # Nuevo turno cuando vuelve a personaje1
    
    def _verificar_fin_combate(self) -> bool:
        """Verifica si el combate ha terminado"""
        if not self.personaje1.vivo:
            self.estado = EstadoCombate.FINALIZADO
            self.ganador = self.personaje2
            self._registrar_log({
                'tipo': 'fin',
                'mensaje': f"¡{self.personaje2.nombre} ha ganado el combate!",
                'ganador': self.personaje2.nombre
            })
            return True
        
        if not self.personaje2.vivo:
            self.estado = EstadoCombate.FINALIZADO
            self.ganador = self.personaje1
            self._registrar_log({
                'tipo': 'fin',
                'mensaje': f"¡{self.personaje1.nombre} ha ganado el combate!",
                'ganador': self.personaje1.nombre
            })
            return True
        
        return False
    
    def _obtener_oponente(self, personaje: Personaje) -> Personaje:
        """Obtiene el oponente de un personaje"""
        return self.personaje2 if personaje == self.personaje1 else self.personaje1
    
    def iniciar(self):
        """Inicia el combate"""
        if self.estado != EstadoCombate.INICIANDO:
            return {'exito': False, 'mensaje': 'El combate ya está en curso'}
        
        self.estado = EstadoCombate.EN_CURSO
        
        mensaje = f"Turno de {self.turno_de.nombre}"
        self._registrar_log({
            'tipo': 'turno',
            'mensaje': mensaje,
            'personaje': self.turno_de.nombre
        })
        
        return {
            'exito': True,
            'mensaje': mensaje,
            'turno_de': self.turno_de.nombre
        }
    
    def realizar_accion(
        self,
        personaje: Personaje,
        accion: AccionCombate,
        objetivo: Optional[int] = None
    ) -> dict:
        """
        Realiza una acción en combate
        
        Args:
            personaje: Personaje que realiza la acción
            accion: Tipo de acción
            objetivo: ID del item (para usar_item)
        
        Returns:
            Resultado de la acción
        """
        # Validaciones
        if self.estado != EstadoCombate.EN_CURSO:
            return {'exito': False, 'mensaje': 'El combate no está en curso'}
        
        if personaje != self.turno_de:
            return {'exito': False, 'mensaje': 'No es tu turno'}
        
        if not personaje.vivo:
            return {'exito': False, 'mensaje': 'Estás muerto'}
        
        oponente = self._obtener_oponente(personaje)
        resultado = {}
        
        # Ejecutar acción
        if accion == AccionCombate.ATACAR:
            resultado = personaje.atacar(oponente)
        
        elif accion == AccionCombate.HABILIDAD:
            resultado = personaje.usar_habilidad(oponente)
        
        elif accion == AccionCombate.USAR_ITEM:
            if objetivo is None:
                return {'exito': False, 'mensaje': 'Especifica el índice del item'}
            resultado = personaje.usar_item(objetivo)
        
        elif accion == AccionCombate.HUIR:
            # 50% de probabilidad de huir
            import random
            if random.random() < 0.5:
                self.estado = EstadoCombate.FINALIZADO
                resultado = {
                    'exito': True,
                    'mensaje': f'{personaje.nombre} huyó del combate',
                    'huida': True
                }
            else:
                resultado = {
                    'exito': False,
                    'mensaje': f'{personaje.nombre} no pudo huir'
                }
        
        # Registrar en log
        if resultado.get('exito'):
            self._registrar_log({
                'tipo': 'accion',
                'accion': accion.value,
                'personaje': personaje.nombre,
                'resultado': resultado
            })
        
        # Verificar fin del combate
        combate_terminado = self._verificar_fin_combate()
        
        if combate_terminado:
            resultado['combate_terminado'] = True
            resultado['ganador'] = self.ganador.nombre if self.ganador else None
            
            # Otorgar XP al ganador
            if self.ganador:
                xp_ganada = self._calcular_xp_ganada()
                subio_nivel = self.ganador.ganar_experiencia(xp_ganada)
                resultado['xp_ganada'] = xp_ganada
                resultado['subio_nivel'] = subio_nivel
        else:
            # Cambiar turno
            self._cambiar_turno()
            resultado['siguiente_turno'] = self.turno_de.nombre
        
        return resultado
    
    def _calcular_xp_ganada(self) -> int:
        """Calcula XP ganada por victoria"""
        if not self.ganador:
            return 0
        
        perdedor = self._obtener_oponente(self.ganador)
        
        # XP base según nivel del perdedor
        xp_base = 50
        xp = xp_base * perdedor.nivel
        
        # Bonus por diferencia de nivel
        diferencia_nivel = perdedor.nivel - self.ganador.nivel
        if diferencia_nivel > 0:
            xp += (xp * 0.2 * diferencia_nivel)  # +20% por cada nivel de diferencia
        
        return int(xp)
    
    def obtener_estado(self) -> dict:
        """Obtiene el estado actual del combate"""
        return {
            'id': self.id,
            'estado': self.estado.value,
            'turno_actual': self.turno_actual,
            'turno_de': self.turno_de.nombre if self.turno_de else None,
            'personaje1': {
                'nombre': self.personaje1.nombre,
                'vida': self.personaje1.vida,
                'vida_maxima': self.personaje1.vida_maxima,
                'mana': self.personaje1.mana,
                'vivo': self.personaje1.vivo
            },
            'personaje2': {
                'nombre': self.personaje2.nombre,
                'vida': self.personaje2.vida,
                'vida_maxima': self.personaje2.vida_maxima,
                'mana': self.personaje2.mana,
                'vivo': self.personaje2.vivo
            },
            'ganador': self.ganador.nombre if self.ganador else None,
            'log': self.log[-5:]  # Últimas 5 entradas
        }
    
    def to_dict(self) -> dict:
        """Serializa el combate completo"""
        return {
            'id': self.id,
            'estado': self.estado.value,
            'turno_actual': self.turno_actual,
            'personaje1': self.personaje1.to_dict(),
            'personaje2': self.personaje2.to_dict(),
            'ganador': self.ganador.nombre if self.ganador else None,
            'log': self.log
        }
```

---

## Gestor de combates

**servicios/gestor_combates.py**

```python
"""
Servicio para gestionar combates
"""

from typing import Dict, Optional
from modelos.combate import Combate, AccionCombate
from modelos.personaje import Personaje

class GestorCombates:
    """Gestiona todos los combates activos"""
    
    def __init__(self):
        self.combates: Dict[int, Combate] = {}
        self.siguiente_id = 1
    
    def crear_combate(self, personaje1: Personaje, personaje2: Personaje) -> Combate:
        """
        Crea un nuevo combate
        
        Args:
            personaje1: Primer combatiente
            personaje2: Segundo combatiente
        
        Returns:
            Combate creado
        """
        combate = Combate(self.siguiente_id, personaje1, personaje2)
        self.combates[self.siguiente_id] = combate
        self.siguiente_id += 1
        
        return combate
    
    def obtener_combate(self, combate_id: int) -> Optional[Combate]:
        """Obtiene un combate por ID"""
        return self.combates.get(combate_id)
    
    def listar_combates_activos(self) -> list:
        """Lista combates que están en curso"""
        from modelos.combate import EstadoCombate
        return [
            c for c in self.combates.values()
            if c.estado == EstadoCombate.EN_CURSO
        ]
    
    def eliminar_combate(self, combate_id: int) -> bool:
        """Elimina un combate"""
        if combate_id in self.combates:
            del self.combates[combate_id]
            return True
        return False
```

---

## Tests del sistema de combate

**test_combate.py**

```python
"""
Tests del sistema de combate
"""

from modelos.personaje import PersonajeFactory
from modelos.combate import AccionCombate
from servicios.gestor_combates import GestorCombates

def test_combate_completo():
    """Test de un combate completo"""
    print("=== Test: Combate Completo ===\n")
    
    # Crear personajes
    guerrero = PersonajeFactory.crear_personaje("Conan", "guerrero", nivel=5)
    mago = PersonajeFactory.crear_personaje("Gandalf", "mago", nivel=5)
    
    print(f"{guerrero.nombre}: {guerrero.vida} HP, {guerrero.fuerza} Fuerza")
    print(f"{mago.nombre}: {mago.vida} HP, {mago.inteligencia} Inteligencia\n")
    
    # Crear gestor y combate
    gestor = GestorCombates()
    combate = gestor.crear_combate(guerrero, mago)
    
    # Iniciar combate
    resultado = combate.iniciar()
    print(resultado['mensaje'])
    print()
    
    # Simular combate
    turno = 1
    while combate.estado.value == 'en_curso' and turno <= 20:
        print(f"--- Turno {turno} ---")
        
        personaje_activo = combate.turno_de
        oponente = mago if personaje_activo == guerrero else guerrero
        
        print(f"Turno de: {personaje_activo.nombre}")
        print(f"  HP: {personaje_activo.vida}/{personaje_activo.vida_maxima}")
        print(f"  Mana: {personaje_activo.mana}/{personaje_activo.mana_maxima}")
        
        # Decidir acción (simple: usar habilidad si hay mana, sino atacar)
        if personaje_activo.mana >= 20:
            accion = AccionCombate.HABILIDAD
        else:
            accion = AccionCombate.ATACAR
        
        # Realizar acción
        resultado = combate.realizar_accion(personaje_activo, accion)
        
        if resultado['exito']:
            print(f"  {resultado['mensaje']}")
            print(f"  {oponente.nombre} HP: {oponente.vida}/{oponente.vida_maxima}")
        else:
            print(f"  Error: {resultado['mensaje']}")
        
        if resultado.get('combate_terminado'):
            print(f"\n🎉 ¡Combate terminado!")
            print(f"Ganador: {resultado['ganador']}")
            print(f"XP ganada: {resultado['xp_ganada']}")
            if resultado['subio_nivel']:
                print(f"¡{resultado['ganador']} subió de nivel!")
            break
        
        print()
        turno += 1
    
    print("\n--- Log del combate ---")
    for entrada in combate.log:
        print(f"Turno {entrada['turno']}: {entrada['mensaje']}")

def test_usar_items_en_combate():
    """Test usar items durante combate"""
    print("\n=== Test: Usar Items en Combate ===\n")
    
    from modelos.item import obtener_item
    
    # Crear personajes
    guerrero = PersonajeFactory.crear_personaje("Arthur", "guerrero", nivel=3)
    enemigo = PersonajeFactory.crear_personaje("Orco", "guerrero", nivel=3)
    
    # Dar pociones al guerrero
    pocion = obtener_item(20)  # Poción de Vida Pequeña
    guerrero.agregar_item(pocion)
    
    print(f"{guerrero.nombre}: {guerrero.vida} HP")
    print(f"Inventario: {len(guerrero.inventario)} items\n")
    
    # Crear combate
    gestor = GestorCombates()
    combate = gestor.crear_combate(guerrero, enemigo)
    combate.iniciar()
    
    # Guerrero recibe daño primero
    guerrero.recibir_daño(50)
    print(f"{guerrero.nombre} recibe 50 de daño: {guerrero.vida} HP\n")
    
    # Usar poción
    resultado = combate.realizar_accion(guerrero, AccionCombate.USAR_ITEM, objetivo=0)
    print(resultado['mensaje'])
    print(f"{guerrero.nombre} HP: {guerrero.vida}/{guerrero.vida_maxima}\n")

def test_huir_combate():
    """Test huir del combate"""
    print("\n=== Test: Huir del Combate ===\n")
    
    guerrero = PersonajeFactory.crear_personaje("Cobarde", "guerrero")
    enemigo = PersonajeFactory.crear_personaje("Dragon", "guerrero", nivel=10)
    
    gestor = GestorCombates()
    combate = gestor.crear_combate(guerrero, enemigo)
    combate.iniciar()
    
    print(f"{guerrero.nombre} (nivel {guerrero.nivel}) vs {enemigo.nombre} (nivel {enemigo.nivel})")
    print(f"{guerrero.nombre} intenta huir...\n")
    
    # Intentar huir varias veces
    intentos = 0
    while combate.estado.value == 'en_curso' and intentos < 5:
        resultado = combate.realizar_accion(guerrero, AccionCombate.HUIR)
        print(resultado['mensaje'])
        
        if resultado.get('huida'):
            print("¡Huyó exitosamente!")
            break
        
        # Enemigo ataca si el guerrero no huyó
        if combate.estado.value == 'en_curso':
            resultado = combate.realizar_accion(enemigo, AccionCombate.ATACAR)
            print(resultado['mensaje'])
            print(f"{guerrero.nombre} HP: {guerrero.vida}/{guerrero.vida_maxima}\n")
        
        intentos += 1

def test_estado_combate():
    """Test obtener estado del combate"""
    print("\n=== Test: Estado del Combate ===\n")
    
    guerrero = PersonajeFactory.crear_personaje("Héroe", "guerrero")
    mago = PersonajeFactory.crear_personaje("Villano", "mago")
    
    gestor = GestorCombates()
    combate = gestor.crear_combate(guerrero, mago)
    combate.iniciar()
    
    # Realizar algunas acciones
    combate.realizar_accion(guerrero, AccionCombate.ATACAR)
    combate.realizar_accion(mago, AccionCombate.HABILIDAD)
    
    # Obtener estado
    estado = combate.obtener_estado()
    
    print(f"Estado del combate #{estado['id']}:")
    print(f"  Estado: {estado['estado']}")
    print(f"  Turno: {estado['turno_actual']}")
    print(f"  Turno de: {estado['turno_de']}")
    print(f"\n  {estado['personaje1']['nombre']}: {estado['personaje1']['vida']}/{estado['personaje1']['vida_maxima']} HP")
    print(f"  {estado['personaje2']['nombre']}: {estado['personaje2']['vida']}/{estado['personaje2']['vida_maxima']} HP")
    print(f"\n  Últimas acciones:")
    for entrada in estado['log']:
        if 'mensaje' in entrada:
            print(f"    - {entrada['mensaje']}")

if __name__ == '__main__':
    PersonajeFactory.reiniciar_contador()
    
    test_combate_completo()
    test_usar_items_en_combate()
    test_huir_combate()
    test_estado_combate()
    
    print("\n✅ Todos los tests de combate completados")
```

**Ejecutar tests:**

```powershell
python test_combate.py
```

---

## Sistema de torneos

Bonus: Sistema para gestionar múltiples combates

**servicios/gestor_torneos.py**

```python
"""
Sistema de torneos
"""

from typing import List
from modelos.personaje import Personaje
from modelos.combate import AccionCombate
from servicios.gestor_combates import GestorCombates
import random

class Torneo:
    """Gestiona un torneo entre múltiples personajes"""
    
    def __init__(self, nombre: str, participantes: List[Personaje]):
        self.nombre = nombre
        self.participantes = participantes
        self.rondas = []
        self.ganador = None
        self.gestor_combates = GestorCombates()
    
    def iniciar(self):
        """Inicia el torneo"""
        print(f"\n{'='*50}")
        print(f"🏆 TORNEO: {self.nombre}")
        print(f"{'='*50}")
        print(f"\nParticipantes:")
        for p in self.participantes:
            print(f"  - {p.nombre} (Nivel {p.nivel} {p.clase.title()})")
        print()
        
        competidores = self.participantes.copy()
        ronda_num = 1
        
        while len(competidores) > 1:
            print(f"\n--- Ronda {ronda_num} ---")
            ganadores = []
            
            # Emparejar competidores
            random.shuffle(competidores)
            while len(competidores) >= 2:
                p1 = competidores.pop(0)
                p2 = competidores.pop(0)
                
                print(f"\n⚔️ {p1.nombre} vs {p2.nombre}")
                ganador = self._simular_combate(p1, p2)
                print(f"  Ganador: {ganador.nombre}")
                
                # Curar al ganador para siguiente ronda
                ganador.vida = ganador.vida_maxima
                ganador.mana = ganador.mana_maxima
                ganadores.append(ganador)
            
            # Si hay impar, pasa automáticamente
            if competidores:
                print(f"\n{competidores[0].nombre} pasa automáticamente")
                competidores[0].vida = competidores[0].vida_maxima
                ganadores.append(competidores[0])
            
            competidores = ganadores
            ronda_num += 1
        
        self.ganador = competidores[0]
        print(f"\n{'='*50}")
        print(f"🏆 GANADOR DEL TORNEO: {self.ganador.nombre}")
        print(f"{'='*50}\n")
    
    def _simular_combate(self, p1: Personaje, p2: Personaje) -> Personaje:
        """Simula un combate y devuelve el ganador"""
        combate = self.gestor_combates.crear_combate(p1, p2)
        combate.iniciar()
        
        turnos = 0
        max_turnos = 50  # Límite de seguridad
        
        while combate.estado.value == 'en_curso' and turnos < max_turnos:
            personaje = combate.turno_de
            
            # IA simple: usar habilidad si tiene mana suficiente
            if personaje.mana >= 20 and random.random() < 0.6:
                accion = AccionCombate.HABILIDAD
            else:
                accion = AccionCombate.ATACAR
            
            combate.realizar_accion(personaje, accion)
            turnos += 1
        
        return combate.ganador if combate.ganador else p1
```

**test_torneo.py**

```python
"""
Test del sistema de torneos
"""

from modelos.personaje import PersonajeFactory
from servicios.gestor_torneos import Torneo

def test_torneo():
    """Test de torneo completo"""
    PersonajeFactory.reiniciar_contador()
    
    # Crear participantes
    participantes = [
        PersonajeFactory.crear_personaje("Conan", "guerrero", nivel=5),
        PersonajeFactory.crear_personaje("Gandalf", "mago", nivel=5),
        PersonajeFactory.crear_personaje("Legolas", "arquero", nivel=5),
        PersonajeFactory.crear_personaje("Thorin", "guerrero", nivel=5),
        PersonajeFactory.crear_personaje("Merlin", "mago", nivel=5),
        PersonajeFactory.crear_personaje("Robin", "arquero", nivel=5),
    ]
    
    # Crear y ejecutar torneo
    torneo = Torneo("Copa de Campeones", participantes)
    torneo.iniciar()

if __name__ == '__main__':
    test_torneo()
```

---

## Conclusión

Has implementado:

- ✅ Sistema de combate por turnos completo
- ✅ Clase `Combate` con estados y gestión de turnos
- ✅ Acciones de combate: atacar, habilidad, usar item, huir
- ✅ Sistema de log de combate
- ✅ Cálculo de XP y recompensas
- ✅ `GestorCombates` para gestionar múltiples combates
- ✅ Sistema de torneos (bonus)
- ✅ Tests completos del sistema

**Siguiente:** [API REST vanilla](./22-api-rest-juego.md)

---

## Recursos adicionales

- **[Turn-Based Combat](https://gamedevelopment.tutsplus.com/)** - Sistemas de combate por turnos
- **[Game AI](https://www.gamedev.net/articles/programming/artificial-intelligence/)** - Inteligencia artificial para juegos

El sistema de combate está completo y listo para la API. ⚔️
