+++
date = '2026-02-20T21:24:03-08:00'
draft = false
type = 'posts'
title = 'Practica3: El paradigma funcional'
+++

# Reporte de Práctica 03: Instalación del Entorno Haskell y Aplicación TODO

**Materia:** Paradigmas de la Programación (40032)  
**Institución:** Universidad Autónoma de Baja California - FIAD  
**Docente:** M.I. José Carlos Gallegos Mariscal  
**Alumno:** Palomares Ceseña Alejandro  
**Matrícula:** 379489  
**Grupo:** 941  

---

## 1. Introducción

El **paradigma funcional** es un estilo de programación declarativa en el que los programas se construyen mediante la composición de funciones matemáticas puras, sin estado mutable ni efectos secundarios. A diferencia del paradigma imperativo (C, Python), no se describe *cómo* ejecutar una secuencia de pasos; se declaran relaciones entre valores y el compilador/intérprete se encarga del resto.

**Haskell** es el representante más prominente de este paradigma: es un lenguaje puramente funcional con tipado estático y evaluación perezosa (*lazy evaluation*). Sus características fundamentales son:

| Concepto | Descripción | Ejemplo |
|----------|-------------|---------|
| **Transparencia referencial** | Una función siempre devuelve el mismo resultado para los mismos argumentos | `double x = x * 2` |
| **Inmutabilidad** | Los valores no cambian; cada operación produce una nueva estructura | `editIndex i x xs = take i xs ++ [x] ++ drop (i+1) xs` |
| **Tipos algebraicos** | Valores que pueden existir (`Just`) o no (`Nothing`), sin punteros nulos | `deleteOne 5 [] = Nothing` |
| **Evaluación perezosa** | Las expresiones no se evalúan hasta que su valor es necesario | `take 5 [1..]` → `[1,2,3,4,5]` |

La evaluación del programa en Haskell se basa en **reducción de expresiones** y **coincidencia de patrones**: el compilador (GHC) transforma las expresiones aplicando las definiciones de las funciones hasta alcanzar un valor normal.

---

## 2. Instalación del Entorno de Desarrollo

### 2.1 GHCup como gestor del entorno

**GHCup** es el instalador oficial recomendado por la comunidad de Haskell. Centraliza la gestión de todas las herramientas del ecosistema en un único punto de entrada, de forma similar a como `rustup` gestiona Rust o `nvm` gestiona Node.js.

Para instalar en Windows se sigue la siguiente secuencia de pasos:

1. Abrir **PowerShell** como usuario normal (sin privilegios de administrador).
2. Navegar a [https://www.haskell.org/ghcup/](https://www.haskell.org/ghcup/) y copiar el comando de instalación para Windows.
3. Pegar y ejecutar el siguiente comando en la terminal:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force;
[System.Net.ServicePointManager]::SecurityProtocol =
  [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;
try {
  & ([ScriptBlock]::Create((Invoke-WebRequest
    https://www.haskell.org/ghcup/sh/bootstrap-haskell.ps1
    -UseBasicParsing))) -Interactive -DisableCurl
} catch { Write-Error $_ }
```

4. Seguir el asistente interactivo y aceptar las opciones predeterminadas para instalar GHC, Stack, Cabal y HLS.
5. Al terminar, cerrar y reabrir PowerShell para que las variables de entorno queden activas.

### 2.2 Componentes del ecosistema instalado

El script instala automáticamente los siguientes seis componentes:

| Herramienta | Tipo | Descripción |
|-------------|------|-------------|
| **GHCup** | Gestor del entorno | Instalador y administrador de todas las herramientas del ecosistema Haskell. |
| **GHC** | Compilador | *Glasgow Haskell Compiler*. Compila archivos `.hs` a binarios nativos optimizados. |
| **GHCi** | Intérprete interactivo | REPL incluido con GHC. Permite evaluar expresiones Haskell en tiempo real. |
| **HLS** | Servidor de lenguaje | *Haskell Language Server*. Provee autocompletado y verificación de tipos al editor. GHC y Stack lo utilizan internamente. |
| **Stack** | Manejador de paquetes | Gestiona dependencias y versiones, garantizando builds reproducibles. Similar a `pip` en Python. |
| **Cabal** | Build tool | Usa Stack para descargar dependencias y GHC para compilar en un solo comando. |

### 2.3 Verificación de la instalación

Se verifica que cada herramienta quedó instalada correctamente ejecutando:

```bash
ghc --version         # GHC 9.x.x
stack --version       # Stack 2.x.x
cabal --version       # Cabal 3.x.x
ghci                  # Abre el intérprete interactivo
```

Dentro de **GHCi** se puede probar una expresión básica para confirmar el funcionamiento:

```
GHCi, version 9.x.x: https://www.haskell.org/ghc/
Prelude> 2 + 2
4
Prelude> map (*2) [1..5]
[2,4,6,8,10]
Prelude> :quit
```

Los archivos de código fuente de Haskell utilizan la extensión `.hs`.

---

## 3. Implementación de la Aplicación TODO

La aplicación **TODO** del repositorio `steadylearner/Haskell` es una aplicación de línea de comandos que permite gestionar una lista de tareas interactivamente. A continuación se describen todos los pasos para implementarla desde cero con Stack.

### 3.1 Paso 1 — Crear el proyecto con Stack

Se crea el proyecto ejecutando `stack new`, que descarga una plantilla estándar y genera la estructura de directorios:

```bash
stack new todo
cd todo
```

Stack resuelve las dependencias automáticamente y genera la siguiente estructura:

```
todo/
├── app/
│   └── Main.hs          ← Punto de entrada (función main)
├── src/
│   └── Lib.hs           ← Lógica de la aplicación
├── test/
│   └── Spec.hs          ← Pruebas unitarias
├── package.yaml         ← Dependencias y metadatos
├── stack.yaml           ← Resolver de versiones
└── todo.cabal           ← Generado automáticamente por Stack
```

### 3.2 Paso 2 — Configurar dependencias en `package.yaml`

La aplicación TODO utiliza dos paquetes externos además de la librería base: `dotenv` para leer variables de entorno desde un archivo `.env`, y `open-browser` para abrir una URL en el navegador por defecto. Se edita `package.yaml` agregando estas dependencias:

```yaml
dependencies:
- base >= 4.7 && < 5
- dotenv
- open-browser
```

Stack descargará automáticamente estos paquetes de **Hackage** (el repositorio central de paquetes Haskell) la primera vez que se compile el proyecto.

### 3.3 Paso 3 — Escribir `app/Main.hs`

`Main.hs` es el punto de entrada de la aplicación. Su función `main` imprime el menú de comandos disponibles y delega el control al bucle principal de `Lib.hs` a través de la función `prompt`, iniciando con una lista vacía:

```haskell
module Main where

import Lib (prompt)

main :: IO ()
main = do
    putStrLn "Commands:"
    putStrLn "+ <String> - Add a TODO entry"
    putStrLn "- <Int>    - Delete the numbered entry"
    putStrLn "s <Int>    - Show the numbered entry"
    putStrLn "e <Int>    - Edit the numbered entry"
    putStrLn "l          - List todo"
    putStrLn "r          - Reverse todo"
    putStrLn "c          - Clear todo"
    putStrLn "q          - Quit"
    prompt []   -- inicia con la lista vacía
```

La notación `IO ()` indica que `main` es una acción que realiza entrada/salida y devuelve un resultado unitario (equivalente a `void` en C). La palabra clave `do` encadena acciones de IO en orden secuencial.

### 3.4 Paso 4 — Escribir `src/Lib.hs`

`Lib.hs` contiene toda la lógica de la aplicación. Se divide en dos partes: la interfaz de usuario (funciones `prompt` e `interpret`) y las funciones puras de manipulación de listas.

#### 3.4.1 Módulo, exportaciones y función de presentación

El módulo declara explícitamente qué funciones exporta (visibles desde `Main.hs` y `Spec.hs`):

```haskell
module Lib
  ( prompt,
    editIndex,
  )
where

import Data.List

-- Muestra una tarea con su índice: "0: Comprar leche"
putTodo :: (Int, String) -> IO ()
putTodo (n, todo) = putStrLn (show n ++ ": " ++ todo)
```

#### 3.4.2 Función `prompt` — el bucle principal

`prompt` es el corazón de la aplicación. Es una función **recursiva** que se llama a sí misma con la lista actualizada después de cada operación, manteniendo la aplicación en ejecución indefinidamente hasta que el usuario ingresa `q`:

```haskell
prompt :: [String] -> IO ()
prompt todos = do
  putStrLn ""
  putStrLn "Comandos: +(crear) -(eliminar) s(mostrar) e(editar)"
  putStrLn "          l(listar) r(invertir) c(limpiar) q(salir)"
  command <- getLine
  if "e" `isPrefixOf` command
    then do
      print "Nuevo texto para esa tarea:"
      newTodo <- getLine
      editTodo command todos newTodo
    else interpret command todos
```

El caso del comando `e` (editar) requiere una entrada adicional, por lo que se maneja por separado antes de delegar a `interpret`.

#### 3.4.3 Función `interpret` — despacho por pattern matching

`interpret` usa **coincidencia de patrones** (*pattern matching*) sobre el primer carácter del comando. Cada patrón corresponde a un comando del menú; esta es la forma idiomática en Haskell de implementar un `switch/case`, verificada en tiempo de compilación:

```haskell
interpret :: String -> [String] -> IO ()

-- (+) Agregar: antepone la tarea a la lista
interpret ('+' : ' ' : todo) todos = prompt (todo : todos)

-- (-) Eliminar por índice
interpret ('-' : ' ' : num) todos =
  case deleteOne (read num) todos of
    Nothing     -> do putStrLn "Índice inválido"; prompt todos
    Just todos' -> prompt todos'

-- (s) Mostrar tarea por índice
interpret ('s' : ' ' : num) todos =
  case showOne (read num) todos of
    Nothing   -> do putStrLn "Índice inválido"; prompt todos
    Just todo -> do print (num ++ ". " ++ todo); prompt todos

-- (l) Listar todas las tareas
interpret "l" todos = do
  print $ show (length todos) ++ " tareas en total"
  mapM_ putTodo (zip [0..] todos)
  prompt todos

-- (r) Mostrar en orden inverso
interpret "r" todos = do
  print $ show (length todos) ++ " tareas en total"
  mapM_ putTodo (zip [0..] (reverseTodos todos))
  prompt todos

-- (c) Limpiar lista
interpret "c" _ = do print "Lista limpiada."; prompt []

-- (q) Salir
interpret "q" _ = return ()

-- Comando inválido
interpret cmd todos = do
  putStrLn ("Comando inválido: `" ++ cmd ++ "`")
  prompt todos
```

#### 3.4.4 Funciones puras de manipulación de listas

Estas funciones son **puramente funcionales**: no realizan entrada/salida y siempre devuelven el mismo resultado para los mismos argumentos. Son el núcleo de la lógica de la aplicación:

```haskell
-- deleteOne: elimina elemento en posición n. Devuelve Nothing si n es inválido.
deleteOne :: Int -> [a] -> Maybe [a]
deleteOne 0 (_ : as) = Just as          -- caso base: elimina la cabeza
deleteOne n (a : as) = do               -- caso recursivo
  as' <- deleteOne (n - 1) as
  return (a : as')
deleteOne _ [] = Nothing                -- lista vacía: índice inválido

-- showOne: devuelve el elemento en posición n, o Nothing.
showOne :: Int -> [a] -> Maybe a
showOne n todos
  | n < 0 || n >= length todos = Nothing
  | otherwise                  = Just (todos !! n)

-- editIndex: reemplaza el elemento en posición i con x. Función pura.
editIndex :: Int -> a -> [a] -> [a]
editIndex i x xs = take i xs ++ [x] ++ drop (i + 1) xs

-- editTodo: orquesta la edición de una tarea con manejo de errores.
editTodo :: String -> [String] -> String -> IO ()
editTodo ('e' : ' ' : num) todos newTodo =
  case editOne (read num) todos newTodo of
    Nothing  -> do putStrLn "Índice inválido"; prompt todos
    Just old -> do
      print $ "Anterior: " ++ old
      print $ "Nuevo:    " ++ newTodo
      let newTodos = editIndex (read num :: Int) newTodo todos
      mapM_ putTodo (zip [0..] newTodos)
      prompt newTodos

editOne :: Int -> [a] -> String -> Maybe a
editOne n todos _
  | n < 0 || n >= length todos = Nothing
  | otherwise                  = Just (todos !! n)

-- reverseTodos: invierte la lista manualmente con acumulador (sin usar reverse).
reverseTodos :: [a] -> [a]
reverseTodos xs = go xs []
  where
    go []     ys = ys
    go (x:xs) ys = go xs (x:ys)
```

### 3.5 Paso 5 — Escribir `test/Spec.hs`

El archivo de pruebas verifica que `editIndex` funcione correctamente usando `assert` de `Control.Exception`. Si la condición es falsa lanza una excepción; si es verdadera devuelve el mensaje de éxito:

```haskell
import Control.Exception
import Lib (editIndex)

main :: IO ()
main = do
    putStrLn "Test:"
    let index    = 1
        new_todo = "two"
        todos    = ["Write", "a", "blog", "post"]
        expected = ["Write", "two", "blog", "post"]
        result   = editIndex index new_todo todos == expected
    putStrLn $ assert result "editIndex funcionó correctamente."
```

### 3.6 Paso 6 — Compilar y ejecutar la aplicación

Con los tres archivos escritos, se gestiona el proyecto con los siguientes comandos de Stack:

```bash
stack build        # Descarga dependencias y compila el proyecto
stack run          # Compila (si hay cambios) y ejecuta la aplicación
stack test         # Ejecuta las pruebas unitarias de Spec.hs
stack repl         # Abre GHCi en el contexto del proyecto
```

La primera vez que se ejecuta `stack build`, Stack descarga el snapshot **LTS** (*Long Term Support*) de Stackage y compila todas las dependencias. Este proceso puede tardar varios minutos, pero las compilaciones posteriores son mucho más rápidas al quedar en caché.

### 3.7 Ejecución y resultados

Al ejecutar `stack run`, la aplicación muestra el menú y queda en espera de comandos:

```
Commands:
+ <String> - Add a TODO entry
- <Int>    - Delete the numbered entry
s <Int>    - Show the numbered entry
e <Int>    - Edit the numbered entry
l          - List todo
r          - Reverse todo
c          - Clear todo
q          - Quit

Comandos: +(crear) -(eliminar) s(mostrar) e(editar)
          l(listar) r(invertir) c(limpiar) q(salir)
```

A continuación se muestra una sesión completa de uso:

```
→  + Estudiar para el examen parcial
→  + Entregar práctica de Haskell
→  + Revisar notas de clase
→  l

"3 tareas en total"
0: Revisar notas de clase
1: Entregar práctica de Haskell
2: Estudiar para el examen parcial

→  e 1
Nuevo texto para esa tarea:
→  Entregar práctica de Haskell (URGENTE)

"Anterior: Entregar práctica de Haskell"
"Nuevo:    Entregar práctica de Haskell (URGENTE)"
0: Revisar notas de clase
1: Entregar práctica de Haskell (URGENTE)
2: Estudiar para el examen parcial

→  - 2
→  l

"2 tareas en total"
0: Revisar notas de clase
1: Entregar práctica de Haskell (URGENTE)

→  q
```

Los comandos disponibles se resumen en la siguiente tabla:

| Comando | Descripción |
|---------|-------------|
| `+ <texto>` | Agrega una nueva tarea al inicio de la lista |
| `- <índice>` | Elimina la tarea en la posición indicada |
| `s <índice>` | Muestra el contenido de la tarea indicada |
| `e <índice>` | Edita la tarea indicada; solicita el nuevo texto |
| `l` | Lista todas las tareas con su índice y el total |
| `r` | Muestra las tareas en orden inverso (sin modificar la lista) |
| `c` | Limpia completamente la lista |
| `q` | Termina la ejecución de la aplicación |

### 3.8 Resultado de las pruebas unitarias

Al ejecutar `stack test`, Stack compila el módulo de pruebas y lo ejecuta. La salida confirma que `editIndex` produce el resultado esperado:

```
todo> test (suite: todo-test)

Test:
editIndex funcionó correctamente.

todo> Test suite todo-test passed
Completed 2 action(s).
```

### 3.9 Paso opcional — Generar ejecutable binario

Stack permite compilar el proyecto en un binario ejecutable independiente:

```bash
stack install --local-bin-path .   # Guarda 'todo-exe' en la carpeta actual
.\todo-exe                          # Ejecuta la aplicación directamente en Windows
```

---

## 4. Conceptos del Paradigma Funcional Observados

A través de esta práctica se pudo apreciar cómo el paradigma funcional aborda la resolución de problemas de una manera diferente a los paradigmas imperativo y lógico:

- **Inmutabilidad:** Las listas de tareas nunca se modifican en lugar; cada operación devuelve una nueva lista. `editIndex` usa `take` y `drop` para construir una lista nueva sin tocar la original.

- **Recursión en lugar de bucles:** `prompt` se llama a sí misma con la lista actualizada, implementando el bucle principal sin ninguna variable de control ni ciclo `while`.

- **Tipos algebraicos con `Maybe`:** En lugar de devolver `null` o `-1` para indicar error, `deleteOne` y `showOne` devuelven `Maybe [a]`. El compilador obliga al llamador a manejar ambos casos (`Just` y `Nothing`).

- **Pattern matching:** `interpret` despacha a la función correcta basándose en el patrón del primer carácter del comando, sin condicionales `if-else` encadenados.

- **Funciones de orden superior:** `mapM_` aplica `putTodo` a cada elemento de la lista, combinando la mónada `IO` con el recorrido funcional de listas.

- **Transparencia referencial:** Las funciones puras (`deleteOne`, `editIndex`, `reverseTodos`) siempre devuelven el mismo resultado para los mismos argumentos, lo que las hace fáciles de probar y razonar.

El paradigma funcional resulta especialmente adecuado para problemas de **transformación de datos, verificación formal y procesamiento de listas**, siendo Haskell una herramienta poderosa para explorar sus principios.

---

## 5. Instrucciones de ejecución

### Requisitos

- [GHCup](https://www.haskell.org/ghcup/) instalado (incluye GHC, Stack y Cabal).

### Archivos del proyecto

```
todo/
├── app/Main.hs          # Punto de entrada
├── src/Lib.hs           # Lógica de la aplicación
├── test/Spec.hs         # Pruebas unitarias
└── package.yaml         # Dependencias
```

### Comandos de ejecución

**Opción 1 — Ejecutar directamente con Stack:**

```bash
stack run
```

**Opción 2 — Compilar y ejecutar por separado:**

```bash
stack build
stack exec todo-exe
```

**Opción 3 — Explorar funciones en el intérprete:**

```bash
stack repl
# Dentro de GHCi:
Prelude> :load src/Lib.hs
Prelude> deleteOne 1 ["a","b","c"]
Just ["a","c"]
Prelude> editIndex 0 "nueva" ["vieja","b","c"]
["nueva","b","c"]
Prelude> :quit
```

---

## 6. Conclusiones

A través de esta práctica se pudo apreciar cómo el paradigma funcional aborda la construcción de software de una manera radicalmente diferente al paradigma imperativo:

- **Declaratividad y composición:** El código de la aplicación TODO expresa *qué* hace cada función, no *cómo* ejecutarla paso a paso. Las funciones se componen para construir comportamientos más complejos a partir de piezas simples y verificables.

- **Recursión natural:** `prompt` y `deleteOne` son ejemplos de cómo la recursión en Haskell puede ser tan concisa como la definición matemática del problema. No se necesitan contadores ni condiciones de ruptura artificiales.

- **Seguridad de tipos:** El tipo `Maybe` elimina una clase completa de errores (acceso a índices inválidos) al nivel del sistema de tipos, antes de que el programa se ejecute. El compilador GHC rechaza código que no maneje ambos casos.

- **Stack como gestor de proyectos:** La resolución automática de dependencias mediante snapshots LTS de Stackage garantiza la reproducibilidad de los builds entre diferentes máquinas y versiones del compilador.

El paradigma funcional resulta especialmente adecuado para problemas de **transformación de datos, verificación formal y concurrencia**, siendo Haskell una herramienta que, aunque con una curva de aprendizaje pronunciada, fuerza al programador a construir software más correcto y predecible desde su diseño.

---

## 7. Referencias

- Haskell.org. (2024). *Downloads*. Recuperado de https://www.haskell.org/downloads/
- Haskell.org. (2024). *GHCup*. Recuperado de https://www.haskell.org/ghcup/
- Haskell.org. (2024). *Get Started*. Recuperado de https://www.haskell.org/get-started/
- HaskellWiki. (2024). *Haskell Tutorial for C Programmers*. Recuperado de https://wiki.haskell.org/Haskell_Tutorial_for_C_Programmers
- Steadylearner. (2021). *How to use Haskell to build a todo app with Stack*. DEV Community. Recuperado de https://dev.to/steadylearner/how-to-use-stack-to-build-a-haskell-app-499j
- GitHub — steadylearner/Haskell. *examples/blog/todo*. Recuperado de https://github.com/steadylearner/Haskell/tree/main/examples/blog/todo
- Gallegos Mariscal, J. C. (2024). *Práctica I — El paradigma funcional*. ISyTE — UABC.
