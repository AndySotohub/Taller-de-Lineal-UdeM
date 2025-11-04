# PROYECTO DE ÁLGEBRA LINEAL
## Rutas Óptimas en Redes de Telecomunicaciones mediante Programación Lineal y Álgebra de Matrices

---

## PORTADA

**UNIVERSIDAD DE MEDELLÍN**  
Facultad de Ingeniería  

**PROYECTO DE ÁLGEBRA LINEAL - ENTREGA #2**  

**Título:** Rutas Óptimas en Redes de Telecomunicaciones mediante Programación Lineal y Álgebra de Matrices  

**Estudiantes:**  
- Dubin Andres Soto Parodi  
- Juan David Idarraga Porras  
- David Santiago Muñoz Espinel

**Profesor:** Juan Pablo Fernández Gutiérrez  
**Fecha de Entrega:** 22 de octubre de 2025  
**Semestre:** 2025-2

---

## INTRODUCCIÓN

En la era digital, las redes de telecomunicaciones constituyen la columna vertebral de la comunicación global. Cada día, millones de datos circulan a través de complejas topologías de red, donde la eficiencia en el enrutamiento del tráfico es fundamental para mantener la calidad del servicio. El presente proyecto demuestra cómo conceptos fundamentales de álgebra lineal pueden aplicarse de manera práctica para resolver problemas reales de optimización en redes.

El desafío central que abordamos es: **¿Cómo encontrar automáticamente la ruta de menor latencia en una red de telecomunicaciones, respetando simultáneamente las restricciones de capacidad de cada enlace?**

Este problema, aunque aparentemente simple, involucra conceptos matemáticos profundos: la teoría de grafos proporciona el marco conceptual, el álgebra lineal permite la representación matricial de la red, y la programación lineal automatiza la búsqueda de la solución óptima. El proyecto integra estas tres disciplinas en una solución práctica y escalable.

---

## PROPÓSITO Y OBJETIVOS

### Propósito General

Demostrar la aplicabilidad del álgebra lineal en la resolución de problemas de optimización en redes de telecomunicaciones mediante la implementación de un modelo de programación lineal que identifique automáticamente la ruta óptima de menor latencia.

### Objetivos Específicos

1. **Modelar una red de telecomunicaciones** como un grafo ponderado utilizando conceptos de teoría de grafos, identificando nodos (routers), aristas (enlaces), y pesos (latencia y capacidad).

2. **Representar la red mediante matrices de álgebra lineal**, incluyendo:
   - Matriz de adyacencia (conectividad)
   - Matriz de costos (latencias en milisegundos)
   - Matriz de capacidades (ancho de banda en Gbps)

3. **Formular un problema de programación lineal** que minimice la latencia total del tráfico desde el nodo origen (A) al destino (F), respetando:
   - Conservación de flujo en nodos intermedios
   - Restricciones de capacidad máxima en cada enlace
   - Restricción de no-negatividad del tráfico

4. **Resolver el problema de optimización** utilizando el algoritmo de programación lineal (método de puntos interiores HiGHS en SciPy).

5. **Validar la solución óptima** verificando el cumplimiento de todas las restricciones y comparándola con rutas alternativas.

6. **Analizar la escalabilidad y extensibilidad** del método para aplicaciones prácticas en operadores de telecomunicaciones, centros de datos e infraestructura de Internet.

---

## MARCO TEÓRICO

### 1. Teoría de Grafos

#### 1.1 Definición Formal

Un **grafo** es una estructura matemática G = (V, E) donde:
- V = conjunto de vértices (nodos)
- E = conjunto de aristas (enlaces)

**En nuestro proyecto:**
- V = {A, B, C, D, E, F} (6 nodos que representan routers)
- E = {(A,B), (A,C), (B,D), (B,E), (C,D), (D,E), (D,F), (E,F)} (8 enlaces bidireccionales)

#### 1.2 Características del Grafo

- **No Dirigido:** Las aristas funcionan en ambas direcciones (enlaces bidireccionales)
- **Ponderado:** Cada arista tiene múltiples pesos asociados (latencia en ms, capacidad en Gbps)
- **Conexo:** Existe un camino entre cualquier par de nodos, garantizando disponibilidad de rutas alternativas

#### 1.3 Tabla de Enlaces con sus Características

| Enlace | Latencia (ms) | Capacidad (Gbps) | Descripción |
|--------|---------------|------------------|-------------|
| A-B | 10 | 100 | Enlace de alta capacidad, baja latencia |
| A-C | 15 | 80 | Enlace alterno de origen |
| B-D | 5 | 50 | Enlace muy rápido pero limitado en capacidad (cuello de botella potencial) |
| B-E | 8 | 120 | Enlace de alta velocidad y buena capacidad |
| C-D | 12 | 90 | Enlace intermedio |
| D-E | 4 | 110 | Enlace más rápido de la red |
| D-F | 20 | 60 | Enlace de alta latencia |
| E-F | 10 | 150 | Enlace de excelente capacidad |

#### 1.4 Rutas Posibles de A a F

| Ruta | Latencia Total | Número de Saltos | Estado |
|------|---|---|---|
| A → B → E → F | 10 + 8 + 10 = **28 ms** | 3 | **ÓPTIMA** |
| A → B → D → F | 10 + 5 + 20 = 35 ms | 3 | Alternativa 1 |
| A → C → D → F | 15 + 12 + 20 = 47 ms | 3 | Alternativa 2 |
| A → C → D → E → F | 15 + 12 + 4 + 10 = 41 ms | 4 | Alternativa 3 |
| A → B → D → E → F | 10 + 5 + 4 + 10 = 29 ms | 4 | Alternativa 4 |

---

### 2. Álgebra Lineal: Representación Matricial

#### 2.1 Matriz de Adyacencia A

Describe la **conectividad entre nodos** (1 = conectado, 0 = no conectado):

|   | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| **A** | 0 | 1 | 1 | 0 | 0 | 0 |
| **B** | 1 | 0 | 0 | 1 | 1 | 0 |
| **C** | 1 | 0 | 0 | 1 | 0 | 0 |
| **D** | 0 | 1 | 1 | 0 | 1 | 1 |
| **E** | 0 | 1 | 0 | 1 | 0 | 1 |
| **F** | 0 | 0 | 0 | 1 | 1 | 0 |

**Propiedades:**
- Matriz simétrica: A[i,j] = A[j,i] (grafo no dirigido)
- Diagonal principal: todos ceros (sin autolazos)
- Rango: proporciona información sobre conectividad de la red

#### 2.2 Matriz de Costos (Latencias) C

Almacena la **latencia en milisegundos** de cada posible enlace:

|   | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| **A** | 0 | 10 | 15 | ∞ | ∞ | ∞ |
| **B** | 10 | 0 | ∞ | 5 | 8 | ∞ |
| **C** | 15 | ∞ | 0 | 12 | ∞ | ∞ |
| **D** | ∞ | 5 | 12 | 0 | 4 | 20 |
| **E** | ∞ | 8 | ∞ | 4 | 0 | 10 |
| **F** | ∞ | ∞ | ∞ | 20 | 10 | 0 |

**Interpretación:**
- C[i,i] = 0: latencia nula de un nodo a sí mismo
- C[i,j] = ∞: no existe conexión directa
- C[D,E] = 4 ms: enlace más rápido
- C[D,F] = 20 ms: enlace más lento

#### 2.3 Matriz de Capacidades Cap

Almacena el **ancho de banda máximo en Gbps** de cada enlace:

|   | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| **A** | 0 | 100 | 80 | ∞ | ∞ | ∞ |
| **B** | 100 | 0 | ∞ | 50 | 120 | ∞ |
| **C** | 80 | ∞ | 0 | 90 | ∞ | ∞ |
| **D** | ∞ | 50 | 90 | 0 | 110 | 60 |
| **E** | ∞ | 120 | ∞ | 110 | 0 | 150 |
| **F** | ∞ | ∞ | ∞ | 60 | 150 | 0 |

**Interpretación crítica:**
- Enlace A-B: 100 Gbps (excelente capacidad)
- Enlace B-D: 50 Gbps (cuello de botella - capacidad limitada)
- Enlace E-F: 150 Gbps (mejor capacidad disponible)

---

### 3. Programación Lineal

#### 3.1 Conceptos Fundamentales

La **programación lineal** es una técnica de optimización matemática que:
- Optimiza (minimiza o maximiza) una **función objetivo lineal**
- Respeta un conjunto de **restricciones lineales**
- **Garantiza** encontrar la solución óptima (si existe)

Estructura general de un problema de programación lineal:

**Minimizar:** Z = c₁x₁ + c₂x₂ + ... + cₙxₙ

**Sujeto a:**
- a_{i1}x₁ + a_{i2}x₂ + ... + a_{in}xₙ ≤ b_i (para todo i)
- x_j ≥ 0 (para todo j)

---

## FORMULACIÓN MATEMÁTICA DEL PROBLEMA

### 3.2 Variables de Decisión

Para cada **enlace dirigido** (i, j), definimos:

**x_{ij} = cantidad de tráfico (Gbps) que fluye del nodo i al nodo j**

**En nuestro proyecto:** 16 variables (8 enlaces × 2 direcciones)
- x_AB, x_BA, x_AC, x_CA, x_BD, x_DB, x_BE, x_EB
- x_CD, x_DC, x_DE, x_ED, x_DF, x_FD, x_EF, x_FE

---

### 3.3 Función Objetivo

**Minimizar la latencia total ponderada:**

**Z = suma de (latencia_{ij} × x_{ij}) para todos los enlaces**

Desarrollado completamente:

**Z = 10×x_AB + 10×x_BA + 15×x_AC + 15×x_CA + 5×x_BD + 5×x_DB + 8×x_BE + 8×x_EB + 12×x_CD + 12×x_DC + 4×x_DE + 4×x_ED + 20×x_DF + 20×x_FD + 10×x_EF + 10×x_FE**

---

### 3.4 Restricciones de Conservación de Flujo

La **ley de conservación de flujo** establece que en nodos intermedios, el flujo que entra debe igualar el flujo que sale. En el nodo origen, el flujo debe salir del sistema, y en el nodo destino, el flujo debe entrar.

#### Nodo A (Origen):

**x_AB + x_AC = 40**

El nodo origen envía exactamente 40 Gbps por los dos enlaces salientes.

#### Nodo F (Destino):

**x_DF + x_EF = 40**

El nodo destino recibe exactamente 40 Gbps de los dos enlaces entrantes.

#### Nodo B (Intermedio):

**x_AB + x_DB + x_EB = x_BA + x_BD + x_BE**

El flujo entrante a B desde A, D y E debe igualar el flujo saliente hacia A, D y E.

#### Nodo C (Intermedio):

**x_AC + x_DC = x_CA + x_CD**

El flujo entrante a C desde A y D debe igualar el flujo saliente hacia A y D.

#### Nodo D (Intermedio):

**x_BD + x_CD + x_ED = x_DB + x_DC + x_DE + x_DF**

El flujo entrante a D desde B, C y E debe igualar el flujo saliente hacia B, C, E y F.

#### Nodo E (Intermedio):

**x_BE + x_DE + x_FE = x_EB + x_ED + x_EF**

El flujo entrante a E desde B, D y F debe igualar el flujo saliente hacia B, D y F.

---

### 3.5 Restricciones de Capacidad

Cada enlace no puede transportar más tráfico que su capacidad máxima.

**Restricción general:** x_ij ≤ capacidad_ij (para todos los enlaces)

Específicamente (enlaces en ambas direcciones):

**Desde nodo A:**
- x_AB ≤ 100 Gbps
- x_AC ≤ 80 Gbps

**Desde nodo B:**
- x_BD ≤ 50 Gbps
- x_BE ≤ 120 Gbps
- x_BA ≤ 100 Gbps (inverso de A-B)

**Desde nodo C:**
- x_CD ≤ 90 Gbps
- x_CA ≤ 80 Gbps (inverso de A-C)

**Desde nodo D:**
- x_DE ≤ 110 Gbps
- x_DF ≤ 60 Gbps
- x_DB ≤ 50 Gbps (inverso de B-D)
- x_DC ≤ 90 Gbps (inverso de C-D)

**Desde nodo E:**
- x_EF ≤ 150 Gbps
- x_ED ≤ 110 Gbps (inverso de D-E)
- x_EB ≤ 120 Gbps (inverso de B-E)

**Desde nodo F:**
- x_FD ≤ 60 Gbps (inverso de D-F)
- x_FE ≤ 150 Gbps (inverso de E-F)

---

### 3.6 Restricción de No-Negatividad

Todo tráfico debe ser no-negativo:

**x_ij ≥ 0 (para todos los enlaces)**

---

## DESARROLLO DEL TRABAJO

### 1. Definición de la Topología de Red

Se modeló una red de telecomunicaciones con 6 nodos (A, B, C, D, E, F) y 8 enlaces bidireccionales. Cada enlace cuenta con dos características principales:

- **Latencia:** tiempo de propagación en milisegundos
- **Capacidad:** ancho de banda máximo en Gigabits por segundo

### 2. Implementación en Código Python

El proyecto se implementó utilizando las siguientes librerías:

```python
import numpy as np                    # Operaciones matriciales
from scipy.optimize import linprog    # Programación lineal
import networkx as nx                 # Modelado de grafos
import matplotlib.pyplot as plt       # Visualización
import pandas as pd                    # Análisis de datos
```

#### 2.1 Definición de Datos

```python
# Topología de la red
nodos = ['A', 'B', 'C', 'D', 'E', 'F']

enlaces = [
    ('A', 'B', 10),    # Latencia 10ms
    ('A', 'C', 15),
    ('B', 'D', 5),
    ('B', 'E', 8),
    ('C', 'D', 12),
    ('D', 'E', 4),
    ('D', 'F', 20),
    ('E', 'F', 10),
]

# Capacidades en Gbps
capacidades = {
    ('A', 'B'): 100,   ('A', 'C'): 80,
    ('B', 'D'): 50,    ('B', 'E'): 120,
    ('C', 'D'): 90,    ('D', 'E'): 110,
    ('D', 'F'): 60,    ('E', 'F'): 150,
}

# Parámetros del problema
origen = 'A'
destino = 'F'
tráfico = 40  # Gbps a enrutar
```

#### 2.2 Construcción de Matrices

Se construyeron las matrices de álgebra lineal para representar algebraicamente la red:

- **Matriz de Costos (C):** Almacena latencias
- **Matriz de Adyacencia (A):** Indica conectividad
- **Matriz de Capacidades (Cap):** Restricciones superiores

#### 2.3 Formulación del Problema de Optimización

Se estructura el problema para scipy.optimize.linprog:

1. **Vector de coeficientes (c):** Latencias asociadas a cada variable
2. **Matriz de restricciones de igualdad (A_eq):** Conservación de flujo
3. **Vector de igualdades (b_eq):** Balance neto en cada nodo
4. **Matriz de restricciones de desigualdad (A_ub):** Capacidades máximas
5. **Vector de desigualdades (b_ub):** Valores de capacidad
6. **Bounds:** Restricción de no-negatividad

#### 2.4 Resolución del Problema

```python
resultado = linprog(
    c=c,
    A_ub=A_ub,
    b_ub=b_ub,
    A_eq=A_eq,
    b_eq=b_eq,
    bounds=bounds,
    method='highs'  # Algoritmo de puntos interiores
)
```

### 3. Visualización de Resultados

Se generó una visualización gráfica de la topología de red utilizando NetworkX y Matplotlib, mostrando:
- Nodos como círculos azul claro
- Enlaces como líneas conectoras
- Etiquetas indicando latencia (ms) y capacidad (Gbps) de cada enlace

### 4. Análisis Comparativo

Se evaluaron 5 rutas posibles desde A hasta F, calculando para cada una:
- Latencia total (suma de latencias en cada salto)
- Número de saltos
- Cuello de botella (enlace con menor capacidad)
- Factibilidad respecto al requisito de 40 Gbps

---

## SOLUCIÓN ÓPTIMA

### Ruta Óptima Encontrada

**A → B → E → F**

### Flujo de Tráfico Óptimo

- **Enlace A→B:** 40 Gbps
- **Enlace B→E:** 40 Gbps
- **Enlace E→F:** 40 Gbps
- **Todos los demás enlaces:** 0 Gbps (no transportan tráfico)

### Cálculo de Latencia Total

**Latencia total = Latencia(A→B) + Latencia(B→E) + Latencia(E→F)**

**Latencia total = 10 ms + 8 ms + 10 ms = 28 milisegundos**

### Valor de la Función Objetivo

**Z = 10×40 + 8×40 + 10×40 = 400 + 320 + 400 = 1120 ms·Gbps**

---

## VALIDACIÓN DE RESTRICCIONES

### 1. Validación de Conservación de Flujo

| Nodo | Flujo Entrante (Gbps) | Flujo Saliente (Gbps) | Balance | Status |
|------|---|---|---|---|
| **A** (Origen) | 0 | 40 | +40 | CUMPLE |
| **B** (Intermedio) | 40 | 40 | 0 | CUMPLE |
| **C** (Intermedio) | 0 | 0 | 0 | CUMPLE |
| **D** (Intermedio) | 0 | 0 | 0 | CUMPLE |
| **E** (Intermedio) | 40 | 40 | 0 | CUMPLE |
| **F** (Destino) | 40 | 0 | -40 | CUMPLE |

**Conclusión:** Todos los nodos cumplen la ley de conservación de flujo. El tráfico enviado en cada nodo intermedio iguala el tráfico recibido, y los nodos origen/destino envían/reciben la cantidad exacta de tráfico requerida.

### 2. Validación de Restricciones de Capacidad

| Enlace | Tráfico (Gbps) | Capacidad (Gbps) | Cumple Restricción |
|--------|---|---|---|
| A→B | 40 | 100 | SÍ (40 ≤ 100) |
| B→E | 40 | 120 | SÍ (40 ≤ 120) |
| E→F | 40 | 150 | SÍ (40 ≤ 150) |
| Otros enlaces | 0 | Varios | SÍ (0 ≤ Capacidad) |

**Conclusión:** El tráfico en cada enlace utilizado no excede su capacidad máxima. La solución es **factible**.

### 3. Validación de No-Negatividad

Todas las variables de flujo son no-negativas:
- Variables en rutas utilizadas: 40, 40, 40 (positivas) ✓
- Variables en rutas no utilizadas: 0 ✓

**Conclusión:** Se respeta la restricción de no-negatividad.

---

## ANÁLISIS COMPARATIVO

### Comparación con Rutas Alternativas

| Ruta | Latencia (ms) | Saltos | Cuello Botella | Cap. Mín (Gbps) | Cumple 40G | Ventaja vs Óptima |
|------|---|---|---|---|---|---|
| **A→B→E→F** | **28** | 3 | A-B | 100 | SÍ | **Base (óptima)** |
| A→B→D→F | 35 | 3 | B-D | 50 | SÍ | -7 ms (-20%) |
| A→C→D→F | 47 | 3 | D-F | 60 | SÍ | -19 ms (-40%) |
| A→C→D→E→F | 41 | 4 | A-C | 80 | SÍ | -13 ms (-32%) |
| A→B→D→E→F | 29 | 4 | B-D | 50 | SÍ | -1 ms (-3%) |

### Observaciones Clave

1. **Optimalidad Confirmada:** La ruta encontrada por programación lineal (28 ms) es efectivamente **la más rápida** entre todas las alternativas viables.

2. **Mejora Significativa:** 
   - Es 7 ms (20%) más rápida que la segunda mejor opción
   - Es 19 ms (40%) más rápida que la peor ruta

3. **Factibilidad Universal:** Todas las rutas pueden transportar los 40 Gbps requeridos, aunque con diferentes cuellos de botella.

4. **Eficiencia del Algoritmo:** Sin enumerar manualmente, el algoritmo de programación lineal identifica automáticamente la mejor solución.

---

## CONCLUSIONES

### 1. Integración de Conceptos Matemáticos

Este proyecto demuestra cómo **tres disciplinas matemáticas** se integran para resolver un problema de ingeniería:

- **Teoría de Grafos:** Proporciona el modelo conceptual de la red (nodos, aristas, ponderación)
- **Álgebra Lineal:** Permite la representación algebraica mediante matrices de adyacencia, costos y capacidades
- **Programación Lineal:** Automatiza la búsqueda de la solución óptima mediante algoritmos de optimización

### 2. Resultados Logrados

✓ **Ruta óptima identificada:** A → B → E → F  
✓ **Latencia total minimizada:** 28 milisegundos  
✓ **Tráfico enrutado:** 40 Gbps  
✓ **Todas las restricciones satisfechas:** Conservación de flujo y capacidades  
✓ **Optimalidad matemáticamente garantizada:** Verificada mediante análisis comparativo

### 3. Ventajas del Método de Programación Lineal

| Ventaja | Descripción |
|---------|-------------|
| **Automatización** | Busca la solución óptima sin enumerar manualmente todas las posibilidades |
| **Optimalidad Garantizada** | Encuentra la mejor solución posible, no una aproximación |
| **Escalabilidad** | Funciona eficientemente tanto con redes pequeñas como con miles de nodos |
| **Manejo de Restricciones Múltiples** | Gestiona simultáneamente latencias, capacidades y conservación de flujo |
| **Extensibilidad** | Permite agregar fácilmente restricciones adicionales (QoS, redundancia, etc.) |

### 4. Aplicabilidad Práctica

El método propuesto se utiliza actualmente en:

- **Operadores de telecomunicaciones:** Para optimizar el enrutamiento de tráfico de voz y datos
- **Centros de datos:** Para distribuir la carga de manera eficiente entre servidores
- **Internet backbone:** En protocolos de enrutamiento como OSPF y BGP
- **Logística y transporte:** Para optimización de rutas de distribución
- **Redes privadas virtuales (VPN):** Para selección de túneles óptimos

### 5. Reflexión Final

El álgebra lineal no es simplemente teoría abstracta desconectada de la realidad. Como demuestra este proyecto, conceptos como **matrices**, **grafos** y **optimización lineal** son **herramientas fundamentales** para resolver problemas reales en ingeniería de telecomunicaciones. La capacidad de formular matemáticamente un problema complejo y resolverlo algorítmicamente es **esencial** en la ingeniería moderna, permitiendo automatización, escalabilidad y garantía de optimalidad en la toma de decisiones.

---

## BIBLIOGRAFÍA

American Psychological Association. (2024). Publication manual of the American Psychological Association (7th ed.). American Psychological Association.

Bertsekas, D. P. (1998). Network optimization: Continuous and discrete models. Athena Scientific.

Boyd, S., & Vandenberghe, L. (2004). Convex optimization. Cambridge University Press.

Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). Introduction to algorithms (3rd ed.). MIT Press.

Gross, J. L., & Yellen, J. (2005). Graph theory and its applications (2nd ed.). Chapman and Hall/CRC.

Hillier, F. S., & Lieberman, G. J. (2010). Introduction to operations research (9th ed.). McGraw-Hill.

International Telecommunication Union. (2019). Recommendation ITU-R M.2083-0: IMT Vision – Framework and overall objectives of the future development of IMT for 2020 and beyond. ITU Publications.

Kurose, J. F., & Ross, K. W. (2020). Computer networking (8th ed.). Pearson.

NetworkX Development Team. (2024). NetworkX: Network analysis in Python. https://networkx.org/

Peterson, L. L., & Davie, B. S. (2012). Computer networks (5th ed.). Morgan Kaufmann Publishers.

SciPy Developers. (2024). SciPy documentation: scipy.optimize.linprog. Retrieved from https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.linprog.html

Tanenbaum, A. S., & Wetherall, D. J. (2011). Computer networks (5th ed.). Pearson Education.

Vanderbilt University. (2022). Introduction to network optimization. Engineering Course Materials.

---

**Documento compilado:** 22 de octubre de 2025  
**Versión:** 4.0 FINAL CORREGIDA  
**Estado:** Completamente revisado, todas las fórmulas corregidas y optimizado para Markdown
