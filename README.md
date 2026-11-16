# gemini-moons

stack: C#, Python (Pygame)

# Propósito del Proyecto
El presente proyecto tiene como objetivo diseñar, implementar y evaluar un simulador de estrategia táctica en tiempo real basado en una reinterpretación mecánicas de *The Day of Sagittarius III*. El sistema constituye un entorno experimental para validar la integración de computación concurrente de bajo nivel con modelos matemáticos de toma de decisiones bajo incertidumbre.

# Objetivos
## De aprendizaje
1. Sistemas multilenguaje (C#, Python): Implementar un puente IPC (memoria compartida/pipes) entre el motor de simulación en C# (.NET) y una capa de visualización en Python, logrando una latencia de serialización menor a 2 ms por tick a 30 Hz sin asignación de memoria dinámica en el bucle crítico.
2. Matemática estocástica: Modelar la toma de decisiones de la IA enemiga bajo niebla de guerra mediante Filtro de Kalman Extendido y Geometría de la información usando la divergencia de Fisher-Rho para estimar vectores de flotas ocultas, demostrando en pruebas unitarias que la IA converge a la posición real en menos de 5 segundos tras detectar un contacto.
3. Concurrencia y simpatía mecánica: Diseñar un motor de combate en C# usando estructuras Data-Oriented procesados en paralelo por múltiples hilos, manteniendo el frame time de simulación por debajo de 4 ms para 5000 entidades simultáneas y cero recolección de basura (GC Gen 0/1/2 = 0) durante el combate activo.

## De proyecto
- Desarrollar un prototipo jugable 1v1 (Jugador vs. IA) en un mapa 2D con 3 tipos de unidades (Acorazados, Cruceros, Exploradores), sistema de niebla de guerra basada en rango de radar y resolución de combate por flotas.