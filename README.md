# JSSP: Sistema Multi-Agente vs Modelo MIP Exacto

Comparación entre un Sistema Multi-Agente descentralizado (MAS) y un modelo exacto de Programación Entera Mixta (MIP) para resolver el **Job Shop Scheduling Problem (JSSP)**.

Desarrollado en el curso *Introducción a Sistemas Inteligentes de Producción* — Ingeniería Civil Industrial, Universidad de Concepción (2025-2).

---

## Problema

El **Job Shop Scheduling Problem (JSSP)** consiste en determinar los tiempos de inicio y término de cada operación para minimizar el **makespan (Cmax)**, sujeto a:

- **Precedencia**: las operaciones de un trabajo deben ejecutarse en orden.
- **Capacidad**: cada máquina procesa una operación a la vez.
- **No preempción**: las operaciones no pueden interrumpirse.

El JSSP es un problema combinatorio **NP-hard**, lo que hace que los métodos exactos escalen mal con el tamaño de la instancia.

---

## Objetivo

Evaluar el desempeño de un enfoque **descentralizado basado en MAS** y compararlo con un **modelo exacto MIP** usado como referencia óptima, analizando la brecha de calidad y el tiempo de cómputo en distintas instancias.

---

## Metodología

### Modelo MIP (baseline exacto)
- Implementado con **PuLP** y solver **CBC**
- Formulación disyuntiva clásica: variables de tiempo (S, C), variables binarias de precedencia (y), función objetivo Min Cmax
- Límite de tiempo: 180 segundos por instancia

### Sistema Multi-Agente (MAS-SMART)
Arquitectura completamente distribuida con dos tipos de agentes:

| Agente | Rol |
|--------|-----|
| **JobAgent** | Representa cada trabajo. Gestiona su secuencia de operaciones y solicita procesamiento a las máquinas. |
| **MachineAgent** | Representa cada recurso. Resuelve conflictos entre solicitudes aplicando la política SMART. |

**Política SMART** — combina tres criterios de priorización:

```
score = α · EST + β · p + γ · Rem
```

- `EST`: Earliest Start Time (inicio más temprano)
- `p`: duración de la operación (SPT)
- `Rem`: trabajo restante del job (remaining work)

La operación con **menor puntaje** es seleccionada. No existe coordinador central — la eficiencia global emerge de decisiones locales.

---

## Instancias evaluadas

Basadas en casos reales de **molinos de acero (steel mill)**, tomadas del paper de referencia:

| Instancia | Tamaño | Cmax óptimo (MIP) |
|-----------|--------|-------------------|
| JSSP1 | 8 trabajos × 6 máquinas | 505 |
| JSSP2 | 6 trabajos × 6 máquinas | 444 |
| JSSP3 | 6 trabajos × 6 máquinas | 379 |
| JSSP1_fast (−20%) | 8×6 escalada | 404 |
| JSSP2_slow (+20%) | 8×6 escalada | 532 |

---

## Resultados

### Instancias base

| Instancia | Cmax MIP | Cmax MAS | Brecha | t MIP (s) | t MAS (s) |
|-----------|----------|----------|--------|-----------|-----------|
| JSSP1 (8×6) | 505 | 522 | +3.4% | 180 | 0.00020 |
| JSSP2 (6×6) | 444 | 468 | +5.4% | 1.11 | 0.00017 |
| JSSP3 (6×6) | 379 | 461 | +21.6% | 0.93 | 0.00031 |

### Instancias escaladas (análisis de sensibilidad)

| Instancia | Cmax MIP | Cmax MAS | Brecha | t MIP (s) | t MAS (s) |
|-----------|----------|----------|--------|-----------|-----------|
| JSSP1_fast (−20%) | 404 | 416 | +3.0% | 180 (límite) | ~0 |
| JSSP2_slow (+20%) | 532 | 561 | +5.5% | 2.38 | ~0 |

**El MAS fue completamente determinista** — desviación estándar = 0 en todas las corridas.

---

## Conclusiones

- El **MIP** obtiene soluciones óptimas, pero su tiempo de cómputo crece rápidamente y en instancias complejas alcanza el límite sin garantizar optimalidad.
- El **MAS** resuelve todas las instancias en tiempos prácticamente nulos (~0.0002 s), con resultados estables y deterministas.
- La política **SMART** funciona bien en instancias de baja y media congestión (brechas de 3–5%), pero pierde desempeño en estructuras con alta densidad de precedencias (JSSP3: +22%).
- Ambos métodos son **complementarios**: exactitud vs. velocidad.

---

## Trabajo futuro

- Incorporar **aprendizaje por refuerzo (Q-learning / Deep RL)** para que los agentes aprendan políticas adaptativas.
- Ajustar pesos α, β, γ de SMART según la estructura de la instancia.
- Agregar coordinación global ligera entre máquinas para evitar cuellos de botella.
- Escalar a instancias industriales más grandes o con perturbaciones dinámicas.
- Explorar un **híbrido MAS + MIP**: usar MIP como solución inicial y MAS para ajuste en tiempo real.

---

## Stack técnico

- **Python 3.11** — Jupyter Notebook
- **PuLP** — modelado y resolución MIP (solver CBC)
- **matplotlib** — diagramas de Gantt

---

## Estructura del repositorio

```
jssp-mas-vs-mip/
│
├── Código_JSSP-MAS.ipynb        # Notebook principal: MIP + MAS + comparación
├── Informe_JSSP-MAS.pdf         # Informe técnico completo
├── Presentación_JSSP-MAS.pdf    # Presentación del proyecto
└── README.md
```

---

## Cómo ejecutar

1. Clonar el repositorio
2. Instalar dependencias:
   ```bash
   pip install pulp matplotlib
   ```
3. Abrir y ejecutar `Código_JSSP-MAS.ipynb` en Jupyter Notebook
4. El código es completamente determinista — no requiere semillas aleatorias

---

## Autores

Proyecto grupal — Grupo 4:
- **Alan Wilson Germany** — implementación MAS, modelo MIP, benchmarking y análisis de resultados
- Javiera Montesinos Abarca
- Maximiliano Pérez Luarte

Universidad de Concepción · Ingeniería Civil Industrial · 2025
