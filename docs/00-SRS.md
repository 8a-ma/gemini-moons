# 1. Introducción y Alcance
## In-Scope
- Combate táctico 1v1
- Simulación 2D de las unidades y del mapa
- Generación procedural del escenario táctico (Estrellas, planetas, satelites, Meteoroide/Meteoros, Sistemas planetarios)
- Física Newtoniana incorporada (Gravedad, Choque, etc...)
- Formación simple mediante una ley de control entre unidades en un rango de N metros
- Las unidades pueden chocar entre si u con objetos estelares.
- Las unidades no ejercen gravedad entre si
- Control de las unidades por mouse (Selección, Movimiento, Cancelación de orden)
- Control de la cámara por teclado
- El daño recibido por las unidades sigue una degradación teoríca similar a Renewal Theory y Regenerative Processes

## Out-Scope
- Editor de mapas
- Audio
- Sprites para cada unidad

## Definiciones y Acrónimos
- Tick:
- IPC: POR DEFINIR

# 2. Descripción general del sistema
## Contexto
C# contempla la simulación, el motor del juego y la ejecución de la IA. Mientras que Python es la capa de presentación (Interfaz gráfica) para el jugador.

## Restricciones de diseño e implementación
- El cliente de Python tiene prohibido ejecutar lógica de física o estado de juego; toda mutación de estado ocurre exclusivamente en C#
- La IA no tiene una mayor fuente de información que la del jugador humano

## Supuestos y dependencias
- Hardware:
    - CPU x86-64 con soporte AVX2
    - mínimo 2 núcleos físicos
- Software:
    - .NET 10
    - Python 3.12+

# 3. Requisitios funcionales
## Gestión de flotas y movimiento (RF-MOV)
- RF-MOV-01: El sistema debe modelar unidades agrupadas en flotas con vectores de posición, velocidad y aceleración en un plano bidimensional continuo.
- RF-MOV-02: El movimiento debe resolverse en pasos de tiempo fijos (fixed timestep) deterministas.
- RF-MOV-03: El sistema debe de mover las unidades seleccionadas una por una o en un área seleccionada con el mouse.
- RF-MOV-04: Las unidades siempre deben de seguir una ley de control para formación de flotas en enjambre basada en distancia euclidiana.
- RF-MOV-05: Las unidades siempre son afectadas por la gravedad de los cuerpos celestes.

## Detección, Radar y Guerra electrónica (RF-SEN)
- RF-SEN-01: Cada flota posee un radio de emisión electromagnética y un radio de detección pasiva/activa.
- RF-SEN-02: El sistema debe ocultar la posición exacta de flotas enemigas fuera del rango de detección, entregando únicamente lecturas ruidosas o estimaciones probabilísticas (niebla de guerra).

## Resolución de combate (RF-CMB)
- RF-CMB-01: El daño se calcula discretamente por tick en función de la distancia, blindaje de clase y cadencia de fuego mediante un proceso de degeneración (opuesto a un proceso de renovación / Regenerative Processes).
- RF-CMB-02: Las unidades (propias o enemigas) pueden chocar con otra unidad objetivo (propias o enemigas) teniendo una probabilidad de destruir a la unidad objetivo

## Inteligencia Artificial (RF-IA)
- RF-IA-01: El agente autónomo debe de actualizar su matriz de creencias (belief state) sobre la ubicación enemiga utilizando un filtr de kalman extendido ante cada nueva lectura de radar.
- RF-IA-02: La selección de maniobras tácticas (flanqueo, retirada, concentración de fuego, exploración) debe evaluarse mediante un modelo estocástico sin acceso a la memoria del estado real del jugador (prohibido hacer cheating leyendo variables ocultas).

# 4. Requisitos de Interfaces Externas (Fronteras del Sistema)
Todo aquello que no es lógica pura del juego ni métricas internas de velocidad vive aquí (RIE-XX).

## Interfaz de software (RIE-SW)
- RIE-SW-01: Definición del protocolo de transporte (ej. Memoria Compartida / Pipes) y esquema de serialización binaria (ej. Protobuf, FlatBuffers o structs planos).
- RIE-SW-02: Dirección del flujo de datos: C# publica Snapshots de estado de solo lectura; Python envía Commands de intención del usuario (ej. MoveFleetCommand).

## Interfaz de usuario (RIE-UI)
- RIE-UI-01: Renderizado táctico 2D (estilo mapa vectorial/radar) a una tasa independiente de la simulación.
- RIE-UI-02: Panel de telemetría en tiempo real que grafique los tiempos de ejecución de C# y las distribuciones de probabilidad de la IA.

# 5. Requisitos No Funcionales (Presupuesto de Rendimiento y Calidad)
## Rendimiento y Latencia (RNF-PERF)
- RNF-PERF-01: El bucle principal de simulación en C# debe ejecutarse a 30 Hz (8.33 ms por tick), consumiendo un máximo de 4 ms de tiempo de CPU para simular hasta 5,000 entidades activas.
- RNF-PERF-02: La serialización y transferencia de un Snapshot de estado hacia Python debe completar en menos de 2 ms

## Gestión de Memoria y Simpatía Mecánica (RNF-MEM)
- RNF-MEM-01: Cero asignaciones de memoria en el Heap (0 bytes allocated) dentro del bucle crítico de simulación tras la inicialización de la partida (garantizando 0 ejecuciones del Garbage Collector durante el combate).
- RNF-MEM-02: Los datos de entidades deben almacenarse en memoria contigua (Structure of Arrays - SoA) para maximizar el aprovechamiento de líneas de caché L1/L2.

## Concurrencia y Determinismo (RNF-CON)
- RNF-CON-01: El procesamiento de físicas y detección debe paralelizarse a nivel de datos sin bloqueos (lock-free) sobre los hilos trabajadores disponibles.

## Observabilidad y Mantenibilidad (RNF-OBS)
- RNF-OBS-01: El motor en C# debe poder ejecutarse en modo Headless (sin cliente Python conectado) mediante pruebas automatizadas de estrés y benchmarks (ej. BenchmarkDotNet).

# 6. Apéndices y Modelos Matemáticos
En proyectos con fuerte carga algorítmica, se anexa la especificación formal para no ensuciar los requisitos funcionales:
- Apéndice A: Formulación matemática del modelo estocástico (ecuaciones de transición de estado, matriz de covarianza del filtro).
- Apéndice B: Diagrama de secuencia del ciclo de vida de un Tick (Input Python → Cola de Comandos C# → Física/IA Paralela → Publicación de Snapshot → Render Python).

