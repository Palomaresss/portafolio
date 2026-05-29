+++
date = '2026-02-20T21:24:11-08:00'
draft = false
type = 'posts'
title = 'Practica4: El paradigma logico'
+++

# Reporte de Práctica 04: Aplicaciones Con Prolog

**Materia:** Paradigmas de la Programación (40032)  
**Institución:** Universidad Autónoma de Baja California - FIAD  
**Docente:** M.I. José Carlos Gallegos Mariscal  
**Alumno:** Palomares Ceseña Alejandro <br> 
**Matrícula:** 379489<br>
**Grupo:** 941
---

## 1. Introducción

El **paradigma lógico** es un estilo de programación declarativa en el que el programador especifica *qué* es verdadero (hechos y reglas), y el motor de inferencia (la máquina Prolog) se encarga de determinar *cómo* llegar a la respuesta. A diferencia del paradigma funcional o imperativo, no se define una secuencia de pasos; se declara conocimiento y se realizan consultas sobre él.

Los tres elementos fundamentales de Prolog son:

| Elemento | Descripción | Ejemplo |
|----------|-------------|---------|
| **Hecho** | Afirmación incondicional sobre el mundo | `cat(tom).` |
| **Regla** | Relación condicional entre hechos o reglas | `happy(X) :- dances(X).` |
| **Consulta** | Pregunta al motor de inferencia | `?- happy(lili).` |

El flujo de ejecución en Prolog se basa en **unificación** y **backtracking**: el motor intenta unificar la consulta con los hechos y cabezas de reglas de la base de conocimientos; si falla, retrocede y prueba otra alternativa.

---

## 2. Aplicaciones con Prolog

### 2.1 Torres de Hanoi

#### 2.1.1 Descripción del problema

Las **Torres de Hanoi** es un puzzle matemático clásico que consta de tres postes y un conjunto de *N* discos de distintos diámetros. Las reglas son:

1. Solo se puede mover **un disco a la vez**.
2. Un disco mayor **nunca puede colocarse sobre uno menor**.
3. Se puede usar cualquier poste como **auxiliar**.

El objetivo es mover toda la pila del poste origen al poste destino usando el mínimo de movimientos posibles. La solución óptima requiere exactamente **2ᴺ − 1** movimientos.

#### 2.1.2 Implementación en Prolog (`hanoi.pl`)

La solución aprovecha la naturaleza **recursiva** del problema: para mover *N* discos de A a C usando B como auxiliar, se hace lo siguiente:

1. Mover los *(N−1)* discos superiores de **A** a **B** (usando C como auxiliar).
2. Mover el disco más grande de **A** a **C**.
3. Mover los *(N−1)* discos de **B** a **C** (usando A como auxiliar).

```prolog
% ============================================================
%  Torres de Hanoi — Paradigma Lógico (Prolog)
%  Práctica IV — Paradigmas de la Programación
% ============================================================

% hanoi(+N): resuelve las Torres de Hanoi con N discos.
hanoi(N) :-
    N > 0,
    write('=== Torres de Hanoi con '), write(N), write(' disco(s) ==='), nl,
    mover(N, izquierda, derecha, centro),
    nl,
    calcular_movimientos(N, Total),
    write('Total de movimientos: '), write(Total), nl.

% Caso base: mover 1 disco directamente.
mover(1, Origen, Destino, _) :-
    write('  Mover disco 1  ['), write(Origen),
    write(']  -->  ['), write(Destino), write(']'), nl.

% Caso recursivo: mover N discos.
mover(N, Origen, Destino, Aux) :-
    N > 1,
    N1 is N - 1,
    mover(N1, Origen, Aux, Destino),
    write('  Mover disco '), write(N),
    write(' ['), write(Origen),
    write(']  -->  ['), write(Destino), write(']'), nl,
    mover(N1, Aux, Destino, Origen).

% Calculo del numero minimo de movimientos (2^N - 1)
calcular_movimientos(N, Total) :-
    Total is 2^N - 1.
```

#### 2.1.3 Ejecución y resultados

Consulta con 3 discos (`?- hanoi(3).`):

```
=== Torres de Hanoi con 3 disco(s) ===
  Mover disco 1  [izquierda]  -->  [derecha]
  Mover disco 2 [izquierda]  -->  [centro]
  Mover disco 1  [derecha]  -->  [centro]
  Mover disco 3 [izquierda]  -->  [derecha]
  Mover disco 1  [centro]  -->  [izquierda]
  Mover disco 2 [centro]  -->  [derecha]
  Mover disco 1  [izquierda]  -->  [derecha]

Total de movimientos: 7
```

Consulta con 4 discos (`?- hanoi(4).`):

```
=== Torres de Hanoi con 4 disco(s) ===
  Mover disco 1  [izquierda]  -->  [centro]
  Mover disco 2 [izquierda]  -->  [derecha]
  Mover disco 1  [centro]  -->  [derecha]
  Mover disco 3 [izquierda]  -->  [centro]
  Mover disco 1  [derecha]  -->  [izquierda]
  Mover disco 2 [derecha]  -->  [centro]
  Mover disco 1  [izquierda]  -->  [centro]
  Mover disco 4 [izquierda]  -->  [derecha]
  Mover disco 1  [centro]  -->  [derecha]
  Mover disco 2 [centro]  -->  [izquierda]
  Mover disco 1  [derecha]  -->  [izquierda]
  Mover disco 3 [centro]  -->  [derecha]
  Mover disco 1  [izquierda]  -->  [centro]
  Mover disco 2 [izquierda]  -->  [derecha]
  Mover disco 1  [centro]  -->  [derecha]

Total de movimientos: 15
```

#### 2.1.4 Análisis

| Discos (N) | Movimientos (2ᴺ−1) |
|:---:|:---:|
| 1 | 1 |
| 2 | 3 |
| 3 | 7 |
| 4 | 15 |
| 10 | 1 023 |
| 20 | 1 048 575 |

La solución en Prolog es notablemente concisa gracias a la recursión. El predicado `mover/4` expresa directamente la definición matemática del problema, y el motor de backtracking de Prolog garantiza que se exploren todas las ramas necesarias.

---

### 2.2 El Mono y el Plátano

#### 2.2.1 Descripción del problema

El problema del **mono y el plátano** es un problema clásico de planificación en Inteligencia Artificial:

- Un **mono** está parado en la **puerta** de un cuarto.
- Un **plátano** cuelga del techo sobre la posición **ventana**.
- Hay una **caja** en el **centro** del cuarto.
- El mono quiere el plátano pero no puede alcanzarlo directamente.
- Debe **empujar la caja** hasta quedar bajo el plátano y **subirse** para alcanzarlo.

#### 2.2.2 Representación del estado

El estado del mundo se representa con el término compuesto:

```
estado(PosMono, NivelMono, PosCaja, TienePlatano)
```

| Argumento | Valores posibles | Significado |
|-----------|------------------|-------------|
| `PosMono` | `puerta`, `centro`, `ventana` | Posición horizontal del mono |
| `NivelMono` | `en_suelo`, `en_caja` | Si el mono está sobre la caja o no |
| `PosCaja` | `puerta`, `centro`, `ventana` | Posición de la caja |
| `TienePlatano` | `no`, `si` | Si el mono ya obtuvo el plátano |

- **Estado inicial:** `estado(puerta, en_suelo, centro, no)`
- **Estado objetivo:** `estado(_, _, _, si)` — cualquier estado donde `TienePlatano = si`

#### 2.2.3 Implementación en Prolog (`mono_platano.pl`)

```prolog
% ============================================================
%  El Mono y el Platano -- Paradigma Logico (Prolog)
%  Practica IV -- Paradigmas de la Programacion
% ============================================================

% --------------- Movimientos (en orden de prioridad) ---------------

% 1. Agarrar el platano
movimiento(
    estado(ventana, en_caja, ventana, no),
    estado(ventana, en_caja, ventana, si),
    'Agarrar el platano (objetivo alcanzado)'
).

% 2. Subir a la caja
movimiento(
    estado(P, en_suelo, P, Tiene),
    estado(P, en_caja,  P, Tiene),
    subir_caja(en_posicion(P))
).

% 3. Empujar la caja hacia otra posicion
movimiento(
    estado(P, en_suelo, P, Tiene),
    estado(Q, en_suelo, Q, Tiene),
    empujar_caja(de(P), a(Q))
) :-
    member(Q, [ventana, centro, puerta]),
    Q \= P.

% 4. Caminar a otra posicion (la caja no se mueve)
movimiento(
    estado(P, en_suelo, Caja, Tiene),
    estado(Q, en_suelo, Caja, Tiene),
    caminar(de(P), a(Q))
) :-
    member(Q, [centro, ventana, puerta]),
    Q \= P.

% --------------- Busqueda con control de ciclos ---------------

puede_obtener(EstadoInicial) :-
    puede_obtener(EstadoInicial, [EstadoInicial], []).

% Caso base: el mono ya tiene el platano
puede_obtener(estado(_, _, _, si), _Visitados, Pasos) :-
    reverse(Pasos, PasosOrdenados),
    imprimir_pasos(PasosOrdenados, 1).

% Caso recursivo: aplicar movimiento a estado no visitado
puede_obtener(Estado, Visitados, Pasos) :-
    movimiento(Estado, Siguiente, Accion),
    \+ member(Siguiente, Visitados),
    puede_obtener(Siguiente, [Siguiente | Visitados], [Accion | Pasos]).

% --------------- Impresion ---------------

imprimir_pasos([], _).
imprimir_pasos([Accion | Resto], N) :-
    format('  Paso ~w: ~w~n', [N, Accion]),
    N1 is N + 1,
    imprimir_pasos(Resto, N1).

% --------------- Entrada principal ---------------

resolver :-
    write('============================================'), nl,
    write('  Problema del Mono y el Platano'), nl,
    write('============================================'), nl, nl,
    EI = estado(puerta, en_suelo, centro, no),
    format('Estado inicial : ~w~n', [EI]),
    format('Objetivo       : estado(_,_,_,si)~n~n', []),
    write('Secuencia de acciones encontrada:'), nl,
    puede_obtener(EI),
    nl,
    write('=> El mono obtuvo el platano exitosamente!'), nl,
    write('============================================'), nl.
```

#### 2.2.4 Estrategia de búsqueda

El predicado `puede_obtener/3` implementa una **búsqueda en profundidad (DFS)** con control de estados visitados para evitar ciclos:

1. Si el estado actual tiene `TienePlatano = si`, se imprime la solución.
2. En caso contrario, se genera un movimiento posible mediante `movimiento/3`.
3. Si el estado resultante no ha sido visitado, se agrega a la lista de visitados y se continúa la búsqueda recursivamente.
4. Si no hay movimientos disponibles, Prolog hace **backtracking** automáticamente y prueba otra alternativa.

#### 2.2.5 Ejecución y resultados

Consulta: `?- resolver.`

```
============================================
  Problema del Mono y el Platano
============================================

Estado inicial : estado(puerta,en_suelo,centro,no)
Objetivo       : estado(_,_,_,si)

Secuencia de acciones encontrada:
  Paso 1: caminar(de(puerta),a(centro))
  Paso 2: empujar_caja(de(centro),a(ventana))
  Paso 3: subir_caja(en_posicion(ventana))
  Paso 4: Agarrar el platano (objetivo alcanzado)

=> El mono obtuvo el platano exitosamente!
============================================
```

#### 2.2.6 Diagrama de estados de la solución

```
estado(puerta, en_suelo, centro, no)
        |
        | caminar(de(puerta), a(centro))
        v
estado(centro, en_suelo, centro, no)
        |
        | empujar_caja(de(centro), a(ventana))
        v
estado(ventana, en_suelo, ventana, no)
        |
        | subir_caja(en_posicion(ventana))
        v
estado(ventana, en_caja, ventana, no)
        |
        | Agarrar el platano
        v
estado(ventana, en_caja, ventana, si)  *** OBJETIVO ***
```

---

## 3. Conclusiones

A través de esta práctica se pudo apreciar cómo el paradigma lógico aborda la resolución de problemas de una manera radicalmente diferente a los paradigmas imperativo y funcional:

- **Declaratividad:** En ambas soluciones, el código expresa *qué* es una solución válida, no *cómo* calcularla paso a paso. Por ejemplo, las reglas de `movimiento/3` definen qué transiciones son legales; el motor de Prolog encuentra la secuencia automáticamente.

- **Recursión natural:** Las Torres de Hanoi son un ejemplo perfecto de cómo la recursión en Prolog puede ser tan concisa como su definición matemática. El predicado `mover/4` tiene exactamente 2 cláusulas: el caso base y el caso recursivo.

- **Backtracking automático:** En el problema del mono y el plátano, el motor de Prolog explora el espacio de estados de forma automática. La lista de estados visitados evita ciclos sin necesidad de implementar explícitamente la lógica de exploración.

- **Representación del conocimiento:** El estado del mundo como término Prolog (`estado/4`) permite manipular y comparar estados de forma sencilla usando unificación.

El paradigma lógico resulta especialmente adecuado para problemas de **planificación, búsqueda y razonamiento simbólico**, siendo Prolog una herramienta poderosa en aplicaciones de Inteligencia Artificial.

---

## 4. Instrucciones de ejecución

### Requisitos
- [SWI-Prolog](https://www.swi-prolog.org/) instalado en el sistema.

### Archivos del proyecto
```
practica4/
├── hanoi.pl          # Torres de Hanoi
├── mono_platano.pl   # El mono y el plátano
└── reporte_practica4.md
```

### Comandos de ejecución

**Opción 1 — Desde la línea de comandos:**
```bash
# Torres de Hanoi (3 discos)
swipl -g "hanoi(3), halt" hanoi.pl

# Mono y platano
swipl -g "resolver, halt" mono_platano.pl
```

**Opción 2 — Desde el REPL interactivo de SWI-Prolog:**
```bash
swipl hanoi.pl
# Dentro del interprete:
?- hanoi(3).
?- hanoi(4).
?- halt.

swipl mono_platano.pl
# Dentro del interprete:
?- resolver.
?- puede_obtener(estado(puerta, en_suelo, centro, no)).
?- halt.
```

---

## 5. Referencias

- Bratko, I. (2001). *Prolog Programming for Artificial Intelligence* (3rd ed.). Addison-Wesley.
- Clocksin, W. F., & Mellish, C. S. (2003). *Programming in Prolog* (5th ed.). Springer.
- SWI-Prolog Reference Manual. Recuperado de: https://www.swi-prolog.org/pldoc/
- Gallegos Mariscal, J. C. (2024). *Práctica IV — El paradigma lógico*. ISyTE — UABC.
