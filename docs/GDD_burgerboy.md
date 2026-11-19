# GAME DESIGN DOCUMENT (GDD) — BURGERBOY

**Versión del Documento:** 1.0 (MVP / Demo)  
**Motor de Desarrollo:** Godot Engine 4.7.2 (GDScript)  
**Género:** Gestión de Tiempo / Cocina Arcade (Single Player)  
**Plataforma Objetivo:** PC (Windows / Linux / macOS)  
**Plazo Estimado de Desarrollo:** 1 a 4 semanas  

---

## 1. VISIÓN GENERAL DEL PROYECTO

### 1.1. Concepto Central (High Concept)
*BurgerBoy* es un juego de gestión de tiempo y cocina arcade para un solo jugador. El jugador asume el rol del único empleado operativo en un local de comida rápida con servicio de salón y ventanilla *Drive-Thru*. El núcleo de la experiencia reside en la toma de decisiones bajo presión, la optimización de rutas dentro de la cocina y el manejo simultáneo de dos colas de atención independientes durante las horas pico (*Rush Hours*).

### 1.2. Pilares de Diseño
1. **Multitasking Estratégico y Tensión Creciente:** La dificultad escala mediante la concurrencia de pedidos y el control de tiempos de cocción, generando un ritmo frenético pero justo.
2. **Flujo de Atención Dual (Serie vs. Paralelo):** El jugador debe alternar entre la atención estrictamente secuencial (FIFO) del mostrador interior y la atención paralela de la ventanilla *Drive-Thru*.
3. **Lectura Visual Inmediata:** Toda la información crítica (estado de cocción, paciencia de clientes, comandas activas y nivel de estrés) se comunica mediante barras de progreso, iconos y animaciones sin saturar la pantalla con texto.

### 1.3. Alcance de la Demo (MVP)
La versión MVP contempla el desarrollo completo del núcleo jugable estructurado en **dos jornadas**:
* **Día 1 (Jornada de Inducción / Tutorial):** Horario reducido de 10:00 a 16:00 hs con flujo limitado de clientes y acompañamiento de un personaje guía (compañera de trabajo).
* **Día 2 (Jornada Completa):** Horario completo de 10:00 a 22:00 hs con curvas dinámicas de tráfico, dos franjas de *Rush Hour* y operación completa del *Drive-Thru*.

---

## 2. DIRECCIÓN DE ARTE, CÁMARA Y INTERFAZ (UI/UX)

### 2.1. Perspectiva y Configuración Gráfica
* **Perspectiva:** Vista Top-Down inclinada (3/4), utilizando ordenamiento de profundidad dinámico (`YSort`) para personajes y elementos móviles.
* **Estilo Visual:** Ilustración 2D cartoon limpia con contornos definidos y paleta de colores saturados de alto contraste.
* **Resolución Base:** `1280x720` (Relación de aspecto 16:9), configurada en Godot con `stretch_mode = canvas_items` y `stretch_aspect = keep` para preservar la nitidez de los trazos vectoriales en cualquier resolución de pantalla.

### 2.2. Distribución Espacial del Escenario
El mapa se divide en cuatro zonas funcionales claramente diferenciadas:
1. **Cocina (Zona Central-Izquierda):** Espacio cerrado donde se desplaza el jugador, equipado con mesadas perimetrales, isla central de armado y tacho de basura.
2. **Comedor / Lobby (Zona Derecha):** Área de acceso peatonal con dos filas delimitadas: fila superior (toma de pedido) y fila inferior (espera y entrega).
3. **Vereda Peatonal (Zona Inferior):** Vía de tránsito horizontal bidireccional por donde circulan los transeúntes.
4. **Carril Drive-Thru (Zona Izquierda):** Calle privada vertical de sentido único (arriba hacia abajo) con punto de detención frente a la ventanilla izquierda de la cocina.

`[PONER AQUÍ CAPTURA DEL MOCKUP GENERAL DE FIGMA O LAYOUT DEL ESCENARIO CON REFERENCIAS DE ZONAS]`

### 2.3. Elementos de HUD y Pantallas de Estado
* **Barra Superior de Comandas:** Despliega de izquierda a derecha las tarjetas de pedidos activos. Cada tarjeta incluye:
  * Icono identificador de origen (Peatón o Automóvil).
  * Iconos de los productos solicitados (Hamburguesa con sus modificadores, Papas Fritas, Bebida por color).
* **Indicador de Tiempo y Dinero (Esquina Superior Derecha):** Reloj digital (10:00 AM - 22:00 PM), indicador de día actual y contador acumulado de dinero (puntaje).
* **Medidor de Estabilidad / Estrés:** Representado por un corazón anatómico animado en pantalla. Su frecuencia de latido y agitación visual aumentan a medida que disminuye la estabilidad del jugador.
* **Pantalla de Victoria (Fin de Jornada):** Muestra la animación de celebración del protagonista y el desglose de recaudación del día.
* **Pantalla de Derrota (Colapso por Estrés):** Oscurecimiento progresivo de pantalla acompañado de la animación de derrota cuando la estabilidad llega a cero.

`[PONER AQUÍ CAPTURA DE FIGMA DEL HUD COMPLETO EN PARTIDA]`

`[PONER AQUÍ CAPTURA DE FIGMA DE LA PANTALLA DE VICTORIA]`

`[PONER AQUÍ CAPTURA DE FIGMA DE LA PANTALLA DE DERROTA]`

---

## 3. CORE GAMEPLAY LOOP Y REGLAS DE JUEGO

### 3.1. Ciclo de Jornada y Curva de Demanda
* **Duración Temporal:** El reloj inicia a las `10:00` y finaliza a las `22:00` (o `16:00` en el Día 1). Escala base recomendada: `1 hora de juego = 20 segundos reales` (4 minutos reales por jornada completa).
* **Modificadores de Tráfico (Día 2):**
  * **10:00 - 11:00 (Apertura):** Tráfico bajo.
  * **11:00 - 13:00 (Rush Hour Almuerzo):** Frecuencia alta de aparición de peatones y vehículos; mayor probabilidad de entrada al local.
  * **13:00 - 17:00 (Tarde):** Tráfico moderado.
  * **17:00 - 19:00 (Rush Hour Cena):** Frecuencia máxima de clientes y pedidos compuestos.
  * **19:00 - 22:00 (Cierre):** Tráfico descendente hasta el cierre de puertas.

### 3.2. Condiciones de Victoria y Derrota
* **Victoria:** Alcanzar la hora de cierre (`22:00` o `16:00` según el día) manteniendo el nivel de estabilidad por encima de `0%`.
* **Derrota:** Cada cliente (peatón o vehículo) que abandona la fila debido a que su barra de paciencia llegó a `0%` provoca una penalización de estabilidad (ej. `-25%`). Si la estabilidad alcanza `0%`, la partida termina en derrota inmediata.

### 3.3. Sistema Económico (Puntuación)
El dinero actúa exclusivamente como métrica de rendimiento y puntuación final (sin sistema de tienda en la demo):
* **Tarifario Base:**
  * Hamburguesa simple (Pan + Carne cocida): `$5`
  * Extra Lechuga: `+$1`
  * Extra Tomate: `+$1`
  * Porción de Papas Fritas: `$3`
  * Bebida (cualquier sabor): `$2`
* **Mecánica de Cobro:** El cliente abona el importe exacto al momento de registrar su pedido en caja. Al recibir el pedido correcto, si su paciencia restante supera el `50%`, otorga una **propina** adicional limitada por su atributo individual de dinero disponible.

---

## 4. MECÁNICAS DETALLADAS

### 4.1. Control del Jugador e Inventario
* **Desplazamiento:** Movimiento omnidireccional (8 direcciones) mediante teclas `WASD` o Flechas direccionales.
* **Botón de Acción:** Tecla `E` para interactuar con estaciones, tomar órdenes, entregar pedidos y operar menús radiales.
* **Capacidad de Carga:** El jugador posee **1 único slot de mano** (`null` o instancia de ítem).
  * Al transportar bebidas, el personaje utiliza sus spritesheets específicos sosteniendo el vaso correspondiente.
  * Al transportar ingredientes, hamburguesas o bolsas de pedido, el ítem se acopla dinámicamente a un nodo `Marker2D` (`PuntoAgarre`) sincronizado con las manos en cada fotograma de animación.

### 4.2. Menú Radial por Teclado
Utilizado en estaciones con múltiples opciones de selección (Heladera, Máquina de Bebidas y Estación de Bandeja con ítems):
1. El jugador mantiene presionada la tecla `E` frente a la estación. El desplazamiento del personaje queda bloqueado mientras el menú está abierto, pero el tiempo de la jornada y los temporizadores de cocción continúan corriendo normalmente.
2. El jugador presiona una dirección (`W`, `A`, `S`, `D` o Flechas) para resaltar el cuadrante deseado; las opciones no seleccionadas reducen su opacidad.
3. Al soltar `E`, tras un margen de confirmación de `0.35s` (para evitar selecciones accidentales al soltar teclas simultáneamente), se ejecuta la acción seleccionada.

`[PONER AQUÍ CAPTURA DE FIGMA DEL DISEÑO DEL MENÚ RADIAL]`

### 4.3. Estaciones de Trabajo de la Cocina
Todas las estaciones detectan proximidad mediante un área frontal del jugador y muestran un contorno visual amarillo cuando están seleccionadas como objetivo activo.

| Estación | Cantidad | Slots | Funcionamiento e Interacción |
| :--- | :---: | :---: | :--- |
| **Máquina de Bebidas** | 1 | - | Con manos vacías, abre **Menú Radial (4 opciones)**: Bebida Roja, Verde, Amarilla y Celeste. Entrega el vaso lleno en la mano del jugador. |
| **Heladera** | 1 | - | Con manos vacías, abre **Menú Radial (4 opciones)**: Carne Cruda, Pan, Lechuga (ya cortada) y Tomate (ya cortado). |
| **Caja de Papas** | 1 | - | Interacción directa con manos vacías para obtener una porción de Papas Crudas. |
| **Plancha / Hornalla** | 2 | 1 c/u | Cada plancha opera de forma independiente. Acepta Carne Cruda e inicia temporizador: `Cruda` $\rightarrow$ `Cocida` $\rightarrow$ `Quemada`. Muestra barra de progreso superior. Con manos vacías retira la carne en estado Cocido o Quemado. |
| **Freidora** | 2 | 1 c/u | Cada freidora opera de forma independiente. Acepta Papas Crudas e inicia temporizador: `Crudas` $\rightarrow$ `Fritas` $\rightarrow$ `Quemadas`. Muestra barra de progreso superior. Con manos vacías retira las papas en estado Frito o Quemado. |
| **Mesa de Armado** | 1 | 3 útiles | Ocupa los primeros 3 espacios de la isla central (ver sección 4.4). |
| **Estación de Bandeja** | 1 | 1 útil | Ocupa el 4to espacio (inferior derecho) de la isla central, operando como estación independiente (ver sección 4.5). |
| **Tacho de Basura** | 1 | - | Con cualquier ítem o comida quemada en la mano, lo descarta y limpia el inventario del jugador. |

### 4.4. Lógica de la Mesa de Armado (3 Slots Inteligentes)
La mesa central permite apoyar ingredientes sueltos y ensamblar hamburguesas sin necesidad de abrir un menú radial, resolviendo la interacción mediante un algoritmo de prioridad en dos pasos:
1. **Prioridad de Combinación:** Si el jugador sostiene un ingrediente compatible con una hamburguesa en construcción en alguno de los 3 slots (ej. sostiene Carne Cocida y existe un Pan en el slot 2), el juego deposita y combina el ingrediente automáticamente en ese slot.
2. **Prioridad de Proximidad:** Si no existe combinación válida, interactúa con el slot más cercano al jugador (deposita en un slot vacío si lleva un ítem, o recoge el contenido si tiene las manos vacías).

**Reglas de Ensamblaje de Hamburguesa:**
* Cualquier ingrediente válido (`Pan`, `Carne Cocida`, `Lechuga`, `Tomate`) puede apoyarse individualmente en un slot vacío.
* Para apilar ingredientes, la combinación debe incluir `Pan` y `Carne Cocida` (pueden unirse en cualquier orden) y admite `Lechuga` y `Tomate` sin duplicados.
* Intentar agregar un ingrediente repetido o inválido (ej. Carne Cruda o Quemada) reproduce un sonido de error y bloquea la acción.
* Una vez combinados, los ingredientes no pueden separarse. Si una hamburguesa se arma de forma incorrecta, puede dejarse reservada en otro slot vacío para un futuro cliente o arrojarse al tacho de basura.

### 4.5. Lógica de la Estación de Bandeja y Embolsado
Ubicada en el cuarto cuadrante de la mesa central, funciona exclusivamente para consolidar pedidos finales:
* **Capacidad:** Admite máximo **1 Hamburguesa con Pan**, **1 Porción de Papas Fritas** y **1 Bebida**.
* **Depositar Producto:** Al interactuar sosteniendo un producto terminado cuya categoría aún no esté presente en la bandeja, este se añade a la misma. Si la categoría ya está ocupada, emite sonido de error.
* **Menú Radial de Bandeja (Manos Vacías):** Si la bandeja contiene al menos un producto, al mantener `E` con las manos vacías se abre el menú radial mostrando los productos apoyados (para retirar alguno individualmente) junto con la opción **Embolsar Pedido (Icono de Bolsa de Papel)**.
* **Generación de Bolsa (`BolsaPedido`):** Al elegir la opción de embolsado, todos los ítems de la bandeja se empaquetan en una única bolsa de papel en la mano del jugador.
* **Desempaquetado:** Si el jugador sostiene una `BolsaPedido` y la bandeja está vacía, interactuar con ella vuelve a desplegar los productos sobre la bandeja para permitir correcciones antes de entregar.

---

## 5. SISTEMA DE CLIENTES, FILAS Y TUTORIAL

### 5.1. Generación y Atributos de NPCs
Los peatones circulan por la vereda inferior (de izquierda a derecha o viceversa) y al salir de pantalla retornan a un `PoolInactivo` con un tiempo de enfriamiento individual antes de reaparecer. Cada instancia posee:
* `hambre` (Valor numérico que determina si entra al local y el tamaño de su pedido).
* `umbral_entrada` (Valor mínimo de hambre requerido para ingresar).
* `dinero` (Presupuesto máximo que limita qué productos puede ordenar y su capacidad de propina).
* `paciencia_max` (Tiempo total antes de abandonar el local).

Al cruzar el área de entrada del comedor, el NPC evalúa: `hambre >= umbral_entrada`, disponibilidad de espacio en la fila superior y un factor de probabilidad ajustado por la hora del día.

### 5.2. Flujo de Atención en Salón (Cola en Serie / FIFO)
1. **Fila Superior (Toma de Orden):** El cliente camina hasta su posición en la fila superior. Al detenerse en su punto de destino, se activa su barra de paciencia. Cuando el jugador interactúa con `E` frente al mostrador, el cliente paga, genera su comanda en el HUD, recupera un porcentaje de paciencia (ej. `+35%`) y se desplaza hacia la fila inferior.
2. **Fila Inferior (Espera y Entrega):** Los clientes se ordenan de derecha a izquierda según su orden de pedido. **La entrega es estrictamente FIFO (First-In, First-Out):** el jugador solo puede entregar la `BolsaPedido` al primer cliente de la fila inferior.
3. **Validación:** Si el contenido de la bolsa coincide exactamente con la comanda del primer cliente, este se retira satisfecho. Si el pedido no coincide, se reproduce un sonido de error y el cliente sufre una penalización de paciencia.

### 5.3. Flujo de Atención Drive-Thru (Cola en Paralelo)
* Los vehículos ingresan por el carril superior izquierdo y se detienen frente a la ventanilla lateral.
* Al tomar la orden con `E`, su comanda se añade a la derecha de las existentes en la barra superior identificada con el icono de **Automóvil**.
* Al tratarse de un punto de entrega físico independiente del salón, el jugador puede entregar el pedido del vehículo **en cualquier momento**, sin necesidad de haber despachado antes las comandas peatonales previas.

### 5.4. Secuencia de Tutorial (NPC Compañera — Día 1)
Durante el Día 1, un personaje secundario (compañera de trabajo) guía al jugador mediante una máquina de estados finitos (FSM):
1. **Estado `Guiando_Pedido`:** Acompaña visualmente la llegada del primer cliente, señalando mediante globos de diálogo breves cada paso del flujo: tomar orden, cocinar carne/papas, armar en mesa, embolsar en bandeja y entregar.
2. **Estado `Trabajando_Idle`:** Una vez completado el primer pedido, informa que realizará tareas de ordenamiento y entra en un ciclo autónomo desplazándose entre puntos decorativos de la cocina sin obstruir el paso del jugador.
3. **Estado `Cliente_Gag`:** A mitad de la jornada del Día 1, abandona la cocina, se incorpora al flujo exterior como transeúnte e ingresa al salón para realizar un pedido como un cliente regular.

`[PONER AQUÍ CAPTURA DE FIGMA DEL TUTORIAL O DIÁLOGOS DE LA COMPAÑERA]`

---

## 6. DISEÑO SONORO (REQUERIMIENTOS DE AUDIO)

| Categoría | Evento / Acción | Descripción del Efecto (SFX) |
| :--- | :--- | :--- |
| **Interfaz / Control** | Abrir Menú Radial | Pop suave de apertura. |
| **Interfaz / Control** | Cambiar Opción Radial | Clic mecánico ligero (`tick`). |
| **Interfaz / Control** | Confirmar Selección | Campana corta aguda (`chime`). |
| **Interfaz / Control** | Acción Inválida / Error | Zumbido grave corto (`buzzer`). |
| **Cocina** | Plancha Activa | Chisporroteo continuo mientras hay carne cocinándose. |
| **Cocina** | Freidora Activa | Burbujeo de aceite mientras hay papas friéndose. |
| **Cocina** | Cocción Lista | Timbre de campana de cocina (`ding`). |
| **Cocina** | Alerta de Quemado | Pitido intermitente rápido cuando supera el tiempo óptimo. |
| **Cocina** | Embolsar Pedido | Crujido de papel al cerrar la bolsa. |
| **Cocina** | Descartar en Basura | Golpe seco / caída en contenedor. |
| **Clientes / Salón** | Entrada de Cliente | Campanilla de puerta al ingresar al salón. |
| **Clientes / Salón** | Llegada Drive-Thru | Bocina corta suave al frenar en ventanilla. |
| **Clientes / Salón** | Cobro en Caja | Sonido de caja registradora al tomar el pedido. |
| **Clientes / Salón** | Abandono Enojado | Suspiro de frustración / queja del NPC. |
| **Ambiente / HUD** | Estrés Crítico (< 30%) | Latido de corazón que incrementa velocidad y volumen. |

---

## 7. ARQUITECTURA TÉCNICA (GODOT 4.7.2)

### 7.1. Estructura de Directorios (`res://`)
```text
res://
├── assets/
│   ├── sprites/          # Spritesheets de jugador, NPCs, estaciones, UI y escenario
│   ├── audio/            # Efectos de sonido (sfx) y música de fondo
│   └── fonts/            # Tipografías del HUD y comandas
├── data/                 # Recursos serializados (.tres)
│   ├── ingredientes/     # Definiciones de pan, carne, vegetales, papas y bebidas
│   └── dias/             # Configuración de horarios y curvas de spawn (Día 1 y Día 2)
├── autoloads/            # Singletons globales
│   ├── GameManager.gd    # Control de estado de partida, dinero, estabilidad y fin de día
│   ├── TimeManager.gd    # Reloj de jornada (10:00-22:00) y señales de franjas horarias
│   ├── OrderManager.gd   # Gestión de comandas activas y validación de entregas
│   └── AudioManager.gd   # Reproductor centralizado de SFX y bucle de latidos
├── scenes/
│   ├── main/             # Escena principal, HUD, ComandaUI y RadialMenu
│   ├── player/           # Escena y controlador del protagonista
│   ├── estaciones/       # EstacionBase y escenas heredadas de cada mueble
│   └── npcs/             # Escenas de Peatón, Automóvil, Compañera y Spawner
└── scripts/
    └── resources/        # Clases base custom Resource (IngredienteData, PedidoData, ConfigDia)
```

### 7.2. Patrones de Diseño Aplicados
* **Herencia de Estaciones (`EstacionBase`):** Todas las estaciones heredan de una escena base con detección de interacción (`Area2D`), resaltado visual y el método virtual `interactuar(player)`. Esto desacopla completamente al jugador de la lógica interna de cada mueble.
* **Desacoplamiento mediante Señales:** Los cambios en el dinero, la estabilidad (corazón), el reloj y la cola de comandas emiten señales desde los `Autoloads` hacia el HUD, evitando dependencias circulares entre la UI y la lógica de juego.
* **Comparación Normalizada de Pedidos:** Tanto la `Comanda` como la `BolsaPedido` almacenan los componentes de la hamburguesa como arreglos de identificadores ordenados alfabéticamente antes de compararse, garantizando que el orden en que el jugador apiló lechuga y tomate no afecte la validez del pedido.
* **Object Pooling de Peatones:** Los transeúntes se instancian al inicio de la escena y se reciclan mediante activación/desactivación y temporizadores de enfriamiento, evitando picos de consumo de memoria durante las horas pico.

---

## 8. PLAN DE PRODUCCIÓN (TAREAS PARA CLICKUP)

### Sprint 1: Configuración Base, Escenario y Controlador del Jugador
* **Configuración del Proyecto Godot 4.7.2:** Establecer resolución `1280x720` (`canvas_items` / `keep`), capas de colisión (`Mundo`, `Player`, `Estaciones`, `NPCs`), mapa de entradas (`WASD`, Flechas, `E`) y estructura de carpetas.
* **Montaje de Escenario y Colisiones:** Integrar fondo del mapa, configurar nodos `YSort` para profundidad y delimitar colisiones en paredes, mesadas y carriles.
* **Controlador del Jugador y Animaciones:** Implementar movimiento en 8 direcciones, máquina de estados de animación, `Marker2D` (`PuntoAgarre`) animado por frame y cambio de spritesheet al portar bebidas.
* **Sistema de Detección de Estaciones:** Configurar `Area2D` frontal en el jugador para identificar la estación más cercana y activar su contorno amarillo de selección.
* **Componente UI — Menú Radial por Teclado:** Desarrollar escena reutilizable que inmoviliza al jugador al mantener `E`, selecciona cuadrantes con `WASD` y confirma al soltar con buffer de `0.35s`.

### Sprint 2: Estaciones de Cocina y Lógica de Armado
* **Modelo de Datos (`IngredienteData` y `PedidoData`):** Crear clases de recursos y lógica de validación de combinaciones.
* **Heladera y Máquina de Bebidas:** Conectar ambas estaciones con el Menú Radial de 4 opciones para entregar ingredientes y vasos al jugador.
* **Estaciones de Cocción (Planchas x2 y Freidoras x2):** Implementar instancias independientes con máquina de estados (`Crudo` $\rightarrow$ `Cocido` $\rightarrow$ `Quemado`), temporizadores y barras de progreso visuales.
* **Tacho de Basura y Caja de Papas:** Implementar obtención rápida de papas crudas y descarte de ítems en mano.
* **Mesa de Armado (3 Slots Inteligentes):** Programar algoritmo de prioridad (combinación compatible > proximidad de slot) y apilado visual de sprites de hamburguesa.
* **Estación de Bandeja (4to Slot Independiente):** Programar recepción de productos terminados, menú radial de extracción/embolsado (`BolsaPedido`) y desempaquetado en bandeja vacía.

### Sprint 3: Clientes, Filas, Drive-Thru y Comandas
* **Generador Procedural de Pedidos:** Desarrollar algoritmo que construye pedidos aleatorios filtrados por el presupuesto (`dinero`) y nivel de `hambre` del cliente.
* **Pool de Peatones y Navegación:** Implementar spawner reciclable en vereda inferior, trigger de ingreso por umbral de hambre y desplazamiento hacia nodos de fila.
* **Gestión de Filas en Salón (FIFO):** Implementar avance de Fila Superior (toma de orden + recuperación de paciencia) a Fila Inferior (espera), bloqueando entregas fuera de orden.
* **Sistema Drive-Thru (Paralelo):** Implementar spawner de automóviles en carril izquierdo, detención en ventanilla y entrega independiente en paralelo.
* **UI de Comandas Superiores:** Desarrollar contenedor dinámico que instancia tarjetas de pedido (origen + iconos) en orden de llegada y las elimina al completar la entrega.

### Sprint 4: Ciclo de Jornada, Economía, Estrés y Tutorial
* **Sistema de Tiempo y Rush Hours:** Implementar `TimeManager` (10:00 - 22:00) con multiplicadores de frecuencia de spawn en franjas `11:00-13:00` y `17:00-19:00`.
* **Sistema de Estrés (Corazón Animado) y Game Over:** Conectar penalizaciones por abandono con el sprite animado del corazón en HUD y disparar pantalla de derrota al llegar a `0%`.
* **Sistema de Dinero y Propinas:** Implementar cobro en caja, cálculo de propina por paciencia restante y transición a pantalla de victoria al cierre de jornada.
* **NPC Compañera (Tutorial Día 1):** Programar máquina de estados con diálogos guiados para el primer cliente, transición a tareas decorativas y secuencia final saliendo como compradora.

### Sprint 5: Audio, Balanceo y Pulido (QA)
* **Integración de Efectos de Sonido (SFX):** Conseguir e integrar todos los efectos listados en la Sección 6 dentro de `AudioManager`.
* **Balanceo de Jugabilidad:** Ajustar tiempos de cocción, velocidad de caída de paciencia y tasas de spawn en *Rush Hour* mediante pruebas de juego.
* **[Opcional / Post-MVP] Inspector de NPCs:** Implementar selección por clic sobre NPCs en pantalla con seguimiento de cámara, zoom y panel de depuración con sus atributos (`hambre`, `dinero`, `paciencia`).