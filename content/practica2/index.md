+++
date = '2026-02-20T21:23:54-08:00'
draft = false
type = 'posts'
title = 'Practica2: El paradigma orientado a objetos'
+++
# Reporte de Práctica 02: Simulador de Estacionamiento

**Materia:** Paradigmas de la Programación (40032)  
**Institución:** Universidad Autónoma de Baja California - FIAD  
**Docente:** M.I. José Carlos Gallegos Mariscal  
**Alumno:** Palomares Ceseña Alejandro <br>  Hernandez Melchor Daniel Alexandro <br> Nuñez Garcia Diego Nahum <br> Arango Gomez Mauro Daniel <br>
**Matrícula:** 379489 379618 379595 379620 <br>
**Grupo:** 941
---
El código fuente del proyecto está disponible en el repositorio de GitHub: [https://github.com/nahuumm/estacionamiento_poo](https://github.com/nahuumm/estacionamiento_poo)

La página estática del proyecto puede consultarse en: [https://nahuumm.github.io/estacionamiento_poo](https://nahuumm.github.io/estacionamiento_poo)

## 1. Introducción
Este reporte detalla el diseño e implementación de un sistema de gestión de estacionamiento (Parking Lot). El objetivo principal es aplicar los pilares de la Programación Orientada a Objetos (POO) para administrar vehículos, lugares (spots) y cobros, integrando finalmente la lógica de negocio en una interfaz web utilizando el micro-framework Flask bajo un patrón MVC simple.

## 2. Conceptos Fundamentales de POO
A continuación, se explican los conceptos aplicados en este proyecto con ejemplos basados en el dominio del estacionamiento:

### 2.1. Clase y Objeto
* **Clase:** Es la plantilla o "plano" que define los atributos y comportamientos de una entidad. 
  * *Ejemplo:* La clase `Vehicle` define que todo vehículo debe tener placas y un tipo.
* **Objeto:** Es una instancia específica de una clase con datos reales. 
  * *Ejemplo:* Un objeto con placas `ABC-123` de tipo `Car`.

### 2.2. Encapsulamiento
Consiste en ocultar el estado interno de un objeto y protegerlo de modificaciones directas desde el exterior, permitiendo el acceso solo a través de métodos que validan invariantes.
* *Ejemplo:* La clase `ParkingLot` mantiene la lista de lugares (`_spots`) como privada; para ocupar uno, se debe usar el método `enter()`, el cual valida si hay espacio disponible antes de realizar el cambio.

### 2.3. Abstracción
Permite representar las características esenciales de un objeto sin incluir los detalles de implementación complejos.
* *Ejemplo:* La interfaz `RatePolicy` define qué debe hacer una política de cobro (calcular un costo), pero no cómo lo hace cada una (por hora, tarifa plana, etc.).

### 2.4. Herencia
Es el mecanismo por el cual una clase (hija) adquiere las propiedades y métodos de otra (padre), permitiendo la reutilización de código.
* *Ejemplo:* `Car` y `Motorcycle` heredan de la clase base `Vehicle`.

### 2.5. Polimorfismo
Permite que diferentes clases respondan al mismo mensaje de formas distintas. Se logra mediante el uso de una interfaz común.
* *Ejemplo:* Al invocar el método `calculate()` en una `RatePolicy`, el sistema puede ejecutar la lógica de `HourlyRatePolicy` o `FlatRatePolicy` de forma transparente para el controlador.

## 3. Modelo del Dominio
El sistema se basa en la interacción de las siguientes entidades principales:

| Clase | Responsabilidad |
|-------|-----------------|
| **Vehicle** | Abstracción base para vehículos (placas y tipo). |
| **ParkingSpot** | Administra el estado de un lugar individual y su compatibilidad. |
| **Ticket** | Registra la entrada, salida, tiempo transcurrido y estado del servicio. |
| **ParkingLot** | Orquestador principal que gestiona la colección de spots y tickets activos. |
| **RatePolicy** | Define el contrato para el cálculo de tarifas. |

## 4. Evidencia de Conceptos POO (Snippets)

### 4.1. Encapsulamiento y Composición
En este fragmento, `ParkingLot` administra una lista de `ParkingSpot` (composición) y protege el estado mediante validaciones.

```python
class ParkingLot:
    def __init__(self, spots: list[ParkingSpot]):
        self._spots = spots  # Encapsulamiento (atributo privado)
        self._active_tickets = {}

    def enter(self, vehicle: Vehicle):
        # Validación de invariantes
        spot = self._find_available_spot(vehicle)
        if not spot:
            raise ValueError("Estacionamiento lleno")
        # Lógica de asignación...
```

### 4.2. Herencia y Polimorfismo
Se utiliza una interfaz (`Protocol`) para permitir múltiples políticas de cobro intercambiables.

```python
class RatePolicy(Protocol):
    def calculate(self, hours: float, v: Vehicle) -> float:
        ...

class HourlyRatePolicy:
    def calculate(self, hours: float, v: Vehicle) -> float:
        rate = 20.0 if v.type == "Car" else 10.0
        return hours * rate  # Polimorfismo por tipo de vehículo
```

## 5. Implementación MVC con Flask
El proyecto se organizó siguiendo el patrón Modelo-Vista-Controlador para separar la lógica de negocio de la interfaz de usuario:

* **Model:** Contiene las clases del dominio (`Vehicle`, `Ticket`, `ParkingLot`) donde reside toda la lógica de validación y cálculo.
* **View:** Plantillas HTML (Jinja2) que muestran el estado del sistema (Dashboard, Formularios de entrada/salida).
* **Controller:** Rutas definidas en `app.py` que reciben las peticiones del usuario, interactúan con el modelo y devuelven la vista correspondiente.

**Rutas principales:**
* **`GET /`**: Dashboard con ocupación y tickets activos.
* **`POST /entry`**: Procesa el registro de entrada de un vehículo.
* **`POST /exit`**: Procesa la salida, calcula el cobro y libera el lugar.
* ![alt text](<WhatsApp Image 2026-04-03 at 11.07.37 PM.jpeg>)
* ![alt text](<WhatsApp Image 2026-04-03 at 11.15.28 PM.jpeg>)

## 6. Conclusiones
La implementación de este simulador bajo el paradigma orientado a objetos ofrece ventajas significativas en comparación con la programación estructurada:

* **Mantenibilidad:** Gracias a la abstracción, si las reglas de cobro cambian, solo es necesario modificar o agregar una nueva clase `RatePolicy` sin afectar al resto del sistema.
* **Escalabilidad:** La herencia permite añadir nuevos tipos de vehículos (ej. camiones o bicicletas) de forma sencilla.
* **Robustez:** El encapsulamiento garantiza que los datos críticos (como el total recaudado o la ocupación) no sean alterados de forma inválida, manteniendo la integridad del sistema en todo momento.

## 7. Referencias
1. Pallets Projects. (2026). *Welcome to Flask — Flask Documentation (3.1.x)*. https://flask.palletsprojects.com/ 
2. Python Software Foundation. (2026). *dataclasses — Data Classes*. https://docs.python.org/3/library/dataclasses.html 
3. M. Fowler. (2004). *Inversion of Control Containers and the Dependency Injection pattern*. https://martinfowler.com/articles/injection.html 
4. Python Typing Team. (2026). *Protocols — typing specification*. https://typing.python.org/en/latest/spec/protocol.html
```
