+++
date = '2026-02-20T21:22:59-08:00'
draft = false
type = 'posts'
title = 'Practica1: Elementos basicos de los lenguajes de programacion'
+++
**Materia:** 40032 - Paradigmas de la Programación <br>
**Docente:** M.I. José Carlos Gallegos Mariscal <br>
**Alumno:** Palomares Ceseña Alejandro <br>  Hernandez Melchor Daniel Alexandro <br> Nuñez Garcia Diego Nahum <br> Arango Gomez Mauro Daniel <br>
**Matrícula:** 379489 379618 379595 379620 <br>
**Grupo:** 941

---

## 1. Introducción
En esta práctica se desarrolló un simulador de cola de impresión en lenguaje C. El objetivo del proyecto fue implementar una estructura FIFO (First-In, First-Out) para gestionar trabajos de impresión. La solución evolucionó desde el uso de un arreglo estático con capacidad fija, hasta una lista enlazada dinámica (utilizando `malloc` y `free`). Finalmente, se implementó una simulación visual en consola con gestión de retrasos y mejoras como el sistema de prioridades y estadísticas.

## 2. Diseño
### Definición de PrintJob_t
La estructura `PrintJob_t` contiene la información fundamental de cada trabajo:
* **id:** Identificador numérico único y autoincremental.
* **usuario y documento:** Arreglos de caracteres para identificar el origen y el archivo.
* **paginas_total, paginas_restantes y copias:** Variables para el control numérico de la impresión y la simulación.
* **prioridad:** Enumerador (`Prioridad_t`) que indica si el trabajo es `NORMAL` (0) o `URGENTE` (1).
* **estado:** Enumerador (`Estado_t`) que transiciona entre `EN_COLA`, `IMPRIMIENDO`, `COMPLETADO` o `CANCELADO`.
* **ms_por_pagina:** Entero para definir el retraso en la simulación.

### Modelos de la Cola
* **Cola Dinámica (`QueueDynamic_t`):** Implementada mediante nodos (`Node_t`) enlazados. Posee punteros `head` (frente de la cola) y `tail` (final de la cola). Además del tamaño (`size`), se incluyeron variables de estadísticas en la propia estructura (`total_completados` y `total_paginas_impresas`).

## 3. Implementación
* **qd_enqueue:** Modificada para soportar prioridades. Si el trabajo es `URGENTE`, la función recorre la lista e inserta el nuevo nodo justo después del último trabajo urgente, pasando por encima de los trabajos normales. Validando siempre que `malloc` no devuelva `NULL`.
* **qd_peek:** Consulta el trabajo apuntado por `head` sin extraerlo de la lista.
* **qd_dequeue:** Remueve el nodo apuntado por `head`, transfiere sus datos, reasigna la nueva cabeza y libera la memoria del nodo usando `free`.
* **Decisiones relevantes:** Se omitió el uso de `scanf` debido a sus vulnerabilidades. En su lugar, se creó la función `valid_num` que utiliza `fgets` y valida que la entrada contenga exclusivamente dígitos numéricos mediante la función `isdigit`.

## 4. Demostración de Conceptos
### 4.1. Alcance y duración
En el código se evidencian claramente distintos tipos de alcance y duración de variables:
1. **Global / Static:** La variable `static int generador_id = 1;`. Tiene alcance restringido al archivo actual (gracias a `static`) pero duración de almacenamiento estática, lo que permite que mantenga y aumente su valor a lo largo de toda la ejecución del programa sin reiniciarse.
2. **Local (Automática):** La variable `int opcion` en la función `main`. Su alcance está limitado a la función principal y su duración termina cuando el programa finaliza.
3. **Memoria Dinámica:** La variable `Node_t* nuevo` en `qd_enqueue`. Aunque el puntero en sí es local, el espacio en memoria reservado por `malloc` vive en el *heap* y su duración persiste hasta que se llama explícitamente a `free` en la función `qd_dequeue`.

### 4.2. Contratos de funciones
El lenguaje C permite definir contratos explícitos. Por ejemplo:
* `int qd_enqueue(QueueDynamic_t* q, PrintJob_t job)` recibe un puntero mutable, pues su contrato requiere modificar los enlaces de la cola.
* `int qd_peek(const QueueDynamic_t* q, PrintJob_t* out)` recibe un puntero constante (`const`), garantizando a nivel compilador que esta función solo consultará la estructura sin alterar su estado.

## 5. Respuestas a Preguntas Guía
1. **¿Dónde guardaste el contador de id y por qué?** Se guardó como una variable global de archivo usando `static int generador_id = 1;`. Se hizo de esta forma para que la variable persista y conserve su valor entre distintas llamadas a la opción de agregar trabajos, garantizando que el ID sea único y autoincremental sin exponer la variable al resto del proyecto.

2. **En tu versión dinámica: ¿qué función es responsable de liberar memoria? ¿cómo lo verificas?** La memoria de cada nodo individual se libera en la función `qd_dequeue` una vez que el trabajo es extraído. Además, la función `qd_destroy` contiene un ciclo que llama a `dequeue` hasta vaciar la cola, garantizando que no haya fugas de memoria al salir del programa. Se verifica asegurando que cada `malloc` invocado en `enqueue` tenga un paso lógico obligatorio por el `free` en `dequeue`.

3. **¿Qué invariantes mantiene tu cola?** Mantiene la invariante de que el campo `size` representa la cantidad exacta de nodos activos en memoria. Además, si la cola está vacía (`head == NULL`), por diseño lógico `tail` también se mantiene como `NULL` y el tamaño es exactamente 0. En cuanto a prioridad, mantiene la invariante de que un trabajo URGENTE jamás estará posicionado detrás de un trabajo NORMAL.

4. **¿Por qué peek no debe modificar la cola?** Porque su único propósito, a nivel de diseño de estructuras de datos, es inspeccionar el frente (saber quién sigue). Modificar la estructura sin extraer formalmente el elemento corrompería el flujo lógico y los punteros `head`/`tail`.

5. **Si el programa falla al agregar trabajos, ¿cómo distingues entre “cola llena” y “entrada inválida”?** Se distinguen en etapas distintas. Las entradas inválidas son atrapadas inmediatamente por la función de apoyo `valid_num`, la cual no permite avanzar hasta que el dato ingresado sea correcto. Por otro lado, la falla al agregar ("cola llena" o fallo de memoria RAM) ocurre cuando la función `qd_enqueue` retorna `0` si `malloc` devuelve `NULL`.

## 6. Mejoras Implementadas
Se implementaron exitosamente tres de las mejoras sugeridas:
* **Prioridad:** Se añadió el enum `Prioridad_t`. Al insertar en la cola, si el trabajo es `URGENTE`, la lista enlazada se recorre para colocar este nodo justo delante de todos los trabajos normales, respetando el FIFO entre los urgentes.
* **Robustez de entrada:** Se eliminó por completo `scanf`. Se desarrolló la función `valid_num` que lee todo como cadena con `fgets`, limpia el salto de línea y corrobora con un ciclo `for` y la función `isdigit` que el usuario no haya escrito letras por accidente antes de hacer el casteo.
* **Estadísticas:** La propia estructura de la cola (`QueueDynamic_t`) fue modificada para rastrear `total_completados` y `total_paginas_impresas`. Al finalizar la simulación de toda la cola (Opción 4), se imprime un reporte estadístico final del lote de impresiones.

## 7. Análisis Comparativo

La evolución del sistema desde una implementación basada en memoria estática hacia una fundamentada en memoria dinámica representó un cambio de paradigma fundamental durante el desarrollo de la práctica. Este rediseño estructural no solo modificó la sintaxis del código, sino que transformó profundamente la eficiencia algorítmica y la flexibilidad del simulador de impresión.

**Análisis de la Memoria Estática (Arreglos Fijos)**
El modelo inicial, apoyado en un arreglo predefinido (por ejemplo, con capacidad para `MAX_JOBS`), destacó por su enorme simplicidad lógica y seguridad. Al estar la memoria reservada desde el tiempo de compilación, se elimina por completo el riesgo de fugas de memoria y el acceso a la estructura general es más controlable a nivel local.
Sin embargo, esta estructura presenta deficiencias críticas para un sistema de colas en un entorno de producción:
1. **Ineficiencia en la extracción (`dequeue`):** Al requerir que el frente de la cola se mantenga siempre en el índice `0`, retirar el primer trabajo de impresión obliga a desplazar todos los elementos restantes un índice hacia la izquierda. Este corrimiento de datos genera una complejidad temporal lineal de $O(n)$, lo cual representa un costo computacional redundante que empeora progresivamente a medida que la cola crece.
2. **Rigidez espacial:** El límite de trabajos está estrictamente dictado por una constante. Esto genera dos escenarios ineficientes: si hay pocos trabajos encolados, se desperdicia espacio en memoria RAM inútilmente; si el flujo de impresiones es alto, el sistema se ve obligado a rechazar trabajos legítimos (creando un cuello de botella artificial) a pesar de que la computadora tenga recursos de sobra.

**Análisis de la Memoria Dinámica (Listas Enlazadas)**
La migración hacia una cola enlazada con nodos dinámicos administrados mediante punteros (`head` y `tail`) resolvió de tajo las limitantes del modelo estático, introduciendo a su vez nuevas responsabilidades de bajo nivel.
1. **Eficiencia Algorítmica:** La operación de remover el primer elemento (`dequeue`) redujo drásticamente su complejidad, pasando de $O(n)$ a un $O(1)$ constante. En lugar de mover bloques enteros de datos, el sistema simplemente reasigna el puntero `head` al nodo adyacente (`head = head->next`) y descarta el elemento obsoleto. Asimismo, la inserción de impresiones con categoría `URGENTE` resultó mucho más eficiente, ya que colocar un nodo en medio de la cola solo requiere actualizar dos punteros, sin realizar desplazamientos masivos.
2. **Escalabilidad a Demanda:** El límite máximo de la cola dejó de ser una restricción programada para convertirse en un límite dictado exclusivamente por el hardware del sistema. La memoria se solicita al sistema operativo (utilizando `malloc`) justo en el instante en que el usuario envía una impresión, garantizando que la aplicación tenga una huella de memoria estrictamente proporcional a la carga de trabajo actual.
3. **Gestión de Riesgos:** El principal desafío asumido en este nuevo paradigma es la administración manual del ciclo de vida de los datos. Un solo puntero mal manejado, o la pérdida de la referencia `head`, produciría fugas de memoria (*memory leaks*) irreversibles durante la ejecución. Para mitigar esta vulnerabilidad, el diseño arquitectónico requirió la estricta implementación de funciones emparejadas: cada `malloc` en `qd_enqueue` tiene su ineludible `free` en `qd_dequeue`, y se implementó rigurosamente la rutina `qd_destroy` para purgar el *heap* en caso de interrupciones o finalización del programa.

**Conclusión**
El costo de abandonar la seguridad automática de la memoria estática se ve ampliamente justificado por las ganancias en rendimiento y escalabilidad. La memoria dinámica demostró ser el paradigma indispensable para modelar colas de impresión modernas, exigiendo a cambio una disciplina superior en la ingeniería de software y el manejo de apuntadores.

## 8. Evidencia de Ejecución

* **Sesión 1 y 2 (Funcionalidad):**
  > ![alt text](<Captura de pantalla 2026-03-16 193613-1.png>)![alt text](<Captura de pantalla 2026-03-16 194320-2.png>) ![alt text](<Captura de pantalla 2026-03-16 194308-1.png>)

* **Sesión 3 (Simulación y Robustez):**
  > ![alt text](<Captura de pantalla 2026-03-16 194633-1.png>)
  > ![alt text](<Captura de pantalla 2026-03-16 194113-1.png>) ![alt text](<Captura de pantalla 2026-03-16 200623-1.png>)

## 9. Referencias
* Documentación de la Práctica 01: Cola de impresión en lenguaje C, 40032 - Paradigmas de la Programación