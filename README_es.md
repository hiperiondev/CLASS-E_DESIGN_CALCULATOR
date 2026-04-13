<div align="center">
<img src="https://github.com/hiperiondev/CLASS-E_DESIGN_CALCULATOR/raw/main/images/logo.png" width="500">

# Calculadora de Diseño de Amplificador RF Clase-E

> **Planilla Excel/LibreOffice Calc para diseño de amplificador de conmutación Clase-E óptimo y cálculo de filtros pasa-bajos de armónicos — basado en las ecuaciones de Sokal/Raab (WA1HQC · QEX Ene/Feb 2001)**

**Autor:** Emiliano Gonzalez LU3VEA — lu3vea@gmail.com  
**Licencia:** GPL v3  
**Archivo:** `ClassE_Amplifier_Calculator.xlsx`

---
</div>

## Tabla de Contenidos

1. [¿Qué es un Amplificador Clase-E?](#1-qué-es-un-amplificador-clase-e)
2. [¿Por Qué Esta Calculadora?](#2-por-qué-esta-calculadora)
3. [Estructura de la Planilla](#3-estructura-de-la-planilla)
4. [Fundamento Teórico](#4-fundamento-teórico)
   - [Principio de Operación Clase-E](#41-principio-de-operación-clase-e)
   - [Condiciones ZVS y ZDVS](#42-condiciones-zvs-y-zdvs)
   - [Ecuaciones de Diseño Sokal/Raab](#43-ecuaciones-de-diseño-sokalraab)
   - [Teoría del Filtro Pasa-Bajos](#44-teoría-del-filtro-pasa-bajos)
5. [Inicio Rápido](#5-inicio-rápido)
6. [Parámetros de Entrada Explicados](#6-parámetros-de-entrada-explicados)
7. [Salidas / Valores Calculados](#7-salidas--valores-calculados)
8. [Hoja 1 — Calculadora Clase-E](#8-hoja-1--calculadora-clase-e)
9. [Hoja 2 — LPF Directo 5 Polos](#9-hoja-2--lpf-directo-5-polos)
10. [Hoja 3 — LPF Directo 7 Polos](#10-hoja-3--lpf-directo-7-polos)
11. [Hoja 4 — Tabla de Referencia de MOSFETs](#11-hoja-4--tabla-de-referencia-de-mosfets)
12. [Cálculo Iterativo: Por Qué y Cómo](#12-cálculo-iterativo-por-qué-y-cómo)
13. [Guía de Selección de Componentes](#13-guía-de-selección-de-componentes)
    - [C1 — Capacitor de Derivación](#131-c1--capacitor-de-derivación)
    - [C2 — Capacitor Resonante Serie](#132-c2--capacitor-resonante-serie)
    - [L2 — Inductor Resonante Serie](#133-l2--inductor-resonante-serie)
    - [RFC — Bobina de RF](#134-rfc--bobina-de-rf)
    - [Componentes del LPF](#135-componentes-del-lpf)
14. [Guía de Selección de MOSFET](#14-guía-de-selección-de-mosfet)
15. [Cumplimiento Regulatorio (Armónicos)](#15-cumplimiento-regulatorio-armónicos)
16. [Flujo de Trabajo de Simulación (LTspice)](#16-flujo-de-trabajo-de-simulación-ltspice)
17. [Limitaciones Conocidas](#17-limitaciones-conocidas)
18. [Referencias](#18-referencias)
19. [Licencia](#19-licencia)

---

## 1. ¿Qué es un Amplificador Clase-E?

Un **amplificador de potencia Clase-E** es una topología de amplificador RF de potencia de **conmutación (no lineal) de extremo simple** y alta eficiencia. Fue inventado por Nathan O. Sokal (WA1HQC) y Alan D. Sokal, descrito por primera vez en su artículo fundamental de 1975 en el *IEEE Journal of Solid-State Circuits* ("Class E — A New Class of High-Efficiency Tuned Single-Ended Switching Power Amplifiers"), y perfeccionado posteriormente con ecuaciones de diseño prácticas en el artículo *QEX* de Sokal de 2001.

A diferencia de los amplificadores lineales (Clase A, AB, B), donde el dispositivo activo opera en su región lineal y disipa potencia significativa, un amplificador Clase-E opera su transistor como un **interruptor duro** — el dispositivo está completamente ENCENDIDO o completamente APAGADO. Bajo condiciones óptimas ideales (ZVS + ZDVS), la tensión del interruptor es cero en el momento de encendido y la pendiente de la tensión también es cero en ese instante, lo que significa que **no se disipa energía almacenada en el capacitor de salida**. Esto hace que la eficiencia de drenador teórica se aproxime al **100%**, con eficiencias prácticas de 80–95% logradas de manera rutinaria en diseños HF QRP.

Características clave:
- Topología de conmutación óptima (ZVS + ZDVS)
- Eficiencia teórica → 100%; práctica: 80–95%
- Tensión de pico de drenador ≈ 3,56 × Vcc (debe estar dentro del límite V_DSS del MOSFET)
- Más adecuado para transmisores CW/WSPR/datos de frecuencia única
- Ampliamente usado en balizas HF QRP de radioaficionados (WSPR, QRSS, CW) desde 136 kHz hasta VHF

---

## 2. ¿Por Qué Esta Calculadora?

Diseñar un amplificador Clase-E desde cero requiere resolver un conjunto de ecuaciones no lineales acopladas que involucran:

- Potencia de salida, tensión de alimentación, frecuencia de operación y Q cargado
- Valores de componentes (C1, C2, L2, RFC) que dependen de la eficiencia η
- Eficiencia η que depende de los valores de componentes y las pérdidas de ESR
- Capacitancia parásita del MOSFET (Coss) que absorbe parte del C1 requerido

Esta planilla automatiza todos esos cálculos y agrega:

- **Convergencia iterativa** de la dependencia circular η ↔ I_dc
- **Sustracción de Coss** para conocer el capacitor *externo* exacto a agregar en el drenador
- **Redondeo a serie E de 5% estándar** para cada valor de componente
- **Advertencias de clasificación de tensión y corriente** para el MOSFET y los capacitores del LPF
- **Verificaciones de auto-resonancia y saturación del RFC**
- **Un LPF Chebyshev asimétrico completo de 5 elementos** diseñado a la impedancia de fuente correcta R_opt (no 50 Ω) — Hoja 2
- **Un LPF Chebyshev asimétrico completo de 7 elementos** para rechazo de armónicos más estricto (≥−65 dBc a 2f) — Hoja 3
- **Una tabla de referencia de MOSFETs** con dispositivos populares HF/VHF/VLF

---

## 3. Estructura de la Planilla

El libro de trabajo contiene **cuatro hojas**:

| Hoja | Propósito |
|---|---|
| **Calculadora Clase-E** | Hoja de diseño principal. Ingrese parámetros aquí; obtenga todos los valores de componentes. |
| **LPF Directo 5 polos** | LPF Chebyshev de 5 elementos diseñado a la impedancia de drenador real R_opt. Vinculado automáticamente a la hoja principal. |
| **LPF Directo 7 polos** | LPF Chebyshev de 7 elementos diseñado a R_opt para ≥14 dB de rechazo adicional en la banda de supresión vs. el diseño de 5 polos. Vinculado automáticamente a la hoja principal. |
| **Tabla de Referencia de MOSFET** | Parámetros de hoja de datos para 16 MOSFETs comunes usados en diseños Clase-E HF. |

---

## 4. Fundamento Teórico

### 4.1 Principio de Operación Clase-E

El amplificador Clase-E clásico (Sokal Fig. 2) consiste en:

```
Vcc ──── RFC ──────────────────────┬──── L2 ──── C2 ──── R (carga)
                                   │ ← Nodo de Drenador
                             ┌─────┤
                            [SW]   C1
                             │     │
                            GND   GND
```

> **Nota:** `SW` (el MOSFET) y `C1` están ambos conectados entre el **Nodo de Drenador** y GND. `RFC` conecta Vcc al Nodo de Drenador en serie únicamente — **no** aparece como componente de derivación.

- **RFC** (bobina de RF): presenta muy alta impedancia a la frecuencia de operación para que la corriente DC de alimentación fluya con mínima ondulación RF
- **C1** (capacitor de derivación, drenador a GND): da forma a la forma de onda de tensión del drenador; absorbe el Coss del MOSFET
- **L2–C2** (red resonante serie): forma una función pasa-banda que pasa solo el fundamental a la carga mientras filtra armónicos
- **R** (resistencia de carga óptima): calculada a partir de Vcc, P_out y η; difiere de la impedancia de antena (50 Ω) — de ahí la necesidad de una red de adaptación o transformación de impedancia del LPF

### 4.2 Condiciones ZVS y ZDVS

Para **Conmutación de Tensión Cero (ZVS)**, el transistor debe encenderse cuando su tensión drenador-fuente es exactamente cero:

```
V_ds(t_on) = 0
```

Para **Conmutación de Derivada de Tensión Cero (ZDVS)** — también escrita ZDS o ZVDS en algunas publicaciones — la pendiente de la tensión de drenador también debe ser cero en el encendido:

```
dV_ds/dt|_(t_on) = 0
```

Cuando ambas condiciones se cumplen simultáneamente (Clase-E óptimo), el transistor no disipa potencia al conmutar, lo que produce la máxima eficiencia. Cualquier desviación del óptimo (valores de componentes incorrectos, error de frecuencia, desacoplamiento de carga) viola ZVS/ZDVS y degrada rápidamente la eficiencia.

### 4.3 Ecuaciones de Diseño Sokal/Raab

La calculadora implementa las **ecuaciones de forma cerrada del AACD 2001 de Sokal** (ajustes polinomiales corregidos por QL a las soluciones numéricas de la Tabla I). Las ecuaciones clave son:

**Resistencia de Carga Óptima (Ec. 6a, polinomio de 3er orden, ±0,01%):**

```
R = (Veff² / P_out) × 0,576801 × (1,0000086 − 0,414395/QL − 0,577501/QL² + 0,205967/QL³)
```

donde `Veff = Vcc − Vsat` (excursión de tensión efectiva).

**C1 — Capacitor de Derivación (Ec. 7, corregido por QL):**

```
C1 = [poli-QL / (5,4466 × ω × R)] + 0,6 / (ω² × L_RFC)
```

donde `poli-QL` es el factor de corrección polinomial dependiente de QL de la Ec. 7 de Sokal. El término `+0,6/(ω²·L_RFC)` es la corrección del RFC — está vinculado dinámicamente al valor del RFC en la hoja, no es un desplazamiento fijo.

**C2 — Capacitor Resonante Serie (Ec. 9, ajuste racional ±0,072%):**

```
C2 × ω × R = [1/(QL − 0,104823)] × [1,00121 + 1,01468/(QL − 1,7879)]
```

Válido para QL ≥ 1,7879. Se aplica la corrección del RFC −0,2/(ω²·L_RFC).

**L2 — Inductor Resonante Serie (Ec. 10):**

```
L2 = QL × R / ω
```

Esta es la bobina física a devanar. `L_ser = L2 − 1/(ω²·C2)` es el exceso inductivo neto de la rama L2-C2 — es **solo informativo** y no es un componente separado.

**Modelo de Eficiencia (Sokal Ec. 2, extendido para pérdidas en componentes reales):**

```
η = R / [R + ESR_L2 + ESR_C2 + 1,365 × Ron + 0,2116 × ESR_C1 + (ESR_RFC × I_dc²)/P_out]
```

η retroalimenta a I_dc = P_out/(Vcc × η), creando la dependencia circular resuelta por cálculo iterativo.

### 4.4 Teoría del Filtro Pasa-Bajos

#### Por qué se necesita un LPF

Un amplificador Clase-E, por naturaleza de su operación de conmutación dura, genera **rico contenido armónico** en el drenador. Sin un LPF, los armónicos radiarían desde la antena, violando la Parte 97 de la FCC (límite de −43 dBc) y las recomendaciones de la IARU (−50 dBc). Un LPF bien diseñado pasa el fundamental con pérdida de inserción mínima (<0,1–0,5 dB) mientras atenúa los armónicos a niveles conformes.

#### Chebyshev vs Butterworth

Un **LPF Chebyshev (Tipo I, rizado equi-ondulado)** es preferible sobre Butterworth para supresión de armónicos porque:

- Logra una caída más empinada para el mismo orden de filtro
- Permite un pequeño rizado de banda de paso controlado (0,1 dB en este diseño) para ganar significativamente más atenuación en la banda de supresión
- Para un diseño de 5 elementos, Chebyshev 0,1 dB proporciona ≈10–15 dB más de rechazo a 2f que el Butterworth del mismo orden

Los valores prototipo g de Chebyshev para 5 elementos con rizado de 0,1 dB son:

```
g1 = 1,1468,  g2 = 1,3712,  g3 = 1,9750,  g4 = 1,3712,  g5 = 1,1468
```

#### Por qué las tablas estándar de LPF de 50 Ω son INCORRECTAS para conexión directa al drenador

Un LPF Chebyshev de libro diseñado para fuente de 50 Ω y carga de 50 Ω funciona correctamente **solo cuando la impedancia de fuente es efectivamente 50 Ω**. En un amplificador Clase-E conectado directamente al drenador, la impedancia de fuente es **R_opt** (típicamente 5–50 Ω), no 50 Ω. El capacitor de derivación del lado de la fuente debe ser:

```
C_fuente = g2 / (R_opt × ωc)          [CORRECTO]
C_fuente = g2 / (50Ω × ωc)            [INCORRECTO — hace C ≈2× muy pequeño, pierde 10–20 dB a 2f]
```

Por esto, la hoja **LPF Directo (R_opt)** calcula una **escalera asimétrica** con Z_fuente = R_opt y Z_carga = 50 Ω, usando la impedancia de referencia media geométrica `Z0_ef = √(R_opt × Z_carga)` para los inductores serie y escalado independiente para cada capacitor de derivación.

La frecuencia de corte se establece en **fc = 1,40 × f** para mantener la pérdida de inserción del fundamental por debajo de 0,1 dB mientras se proporciona rechazo útil de armónicos a 2f.

---

## 5. Inicio Rápido

### Paso 1 — Activar el Cálculo Iterativo

La planilla usa una pequeña referencia circular (η → I_dc → pérdida RFC → η) que requiere cálculo iterativo para converger.

**Excel:**
> Archivo → Opciones → Fórmulas → Activar Cálculo Iterativo  
> Establecer Máximo de Iteraciones: **100**, Cambio Máximo: **0,0001**

**LibreOffice Calc:**
> Herramientas → Opciones → LibreOffice Calc → Calcular  
> Activar: Iteraciones, establecer en **100** iteraciones, Cambio Mínimo: **0,0001**

### Paso 2 — Ingresar sus Parámetros

En la hoja **Calculadora Clase-E**, edite las celdas azules de **PARÁMETROS DE ENTRADA**:

| Celda | Parámetro | Ejemplo |
|---|---|---|
| Vcc | Tensión de alimentación (V) | 5 |
| P_out | Potencia de salida deseada (W) | 0,4 |
| f | Frecuencia de operación (MHz) | 7 |
| Q_L | Factor Q cargado | 5 |
| Vo | Vsat del transistor (0 para MOSFET) | 0 |
| ESR_L2 | ESR del bobinado L2 (Ω) | 0,05 |
| ESR_C2 | ESR serie del capacitor C2 (Ω) | 0,01 |
| ESR_C1 | ESR serie del capacitor C1 (Ω) | 0,01 |
| ESR_RFC | ESR del bobinado RFC (Ω) | 0,1 |
| Z_out | Impedancia de antena/carga (Ω) | 50 |
| Ciss | Ciss del MOSFET de la hoja de datos (pF) | 60 |
| Coss | Coss del MOSFET de la hoja de datos (pF) | 12 |
| Escala Coss | Factor de escala del punto de polarización | 0,4 |

> **Nota sobre ESR_C1 y ESR_C2:** Estos valores de ESR del capacitor alimentan directamente la fórmula de eficiencia de Sokal (Sección 4.3). Para capacitores NP0/C0G, los valores típicos son 0,01–0,05 Ω y suelen ser despreciables. Para dieléctricos con pérdidas (X7R, Z5U), el ESR puede alcanzar 0,1–0,5 Ω y debe medirse. Usar 0 es aceptable para estimaciones iniciales con capacitores de bajo ESR.

### Paso 3 — Converger la Iteración

Presione **F9** repetidamente (F9 recalcula todo el libro; Shift+F9 recalcula solo la hoja activa) hasta que el indicador de convergencia en la celda C6 muestre:

```
✅ Cálculo iterativo CONVERGIDO — valores estables
```

### Paso 4 — Leer los Valores de Componentes

La sección **VALORES DE COMPONENTES CLASE-E** le proporciona:
- C1_ext — capacitor de derivación externo a agregar (después de restar Coss)
- C2 — capacitor resonante serie
- L2 — inductor resonante serie (una sola bobina)
- RFC — inductancia mínima de la bobina de RF
- Valores estándar E12/E24 al 5% más cercanos para cada uno

### Paso 5 — Leer los Valores del LPF

Cambie a la hoja **LPF Directo (R_opt)** (Hoja 2) para el LPF Chebyshev de 5 elementos, o a la hoja **LPF Directo 7 polos** (Hoja 3) para mayor rechazo de armónicos. Cada hoja proporciona:
- L1, L3, L5 (y L7 para el de 7 polos) — inductores serie
- C2, C4 (y C6 para el de 7 polos) — capacitores de derivación
- Estimación de atenuación de armónicos a 2f y 3f

### Paso 6 — Simular

Antes de construir, simule en **LTspice** (gratuito, de Analog Devices). Verifique:
- ZVS logrado (V_drenador → 0 al encender el interruptor)
- Tensión de pico del drenador dentro de V_DSS / factor_de_seguridad
- Potencia de salida coincide con el objetivo
- La atenuación de armónicos cumple los requisitos regulatorios

---

## 6. Parámetros de Entrada Explicados

### Tensión de Alimentación (Vcc)
Tensión DC de alimentación en voltios. Los diseños QRP típicos usan 5–13,8 V. Mayor Vcc reduce I_dc pero aumenta la tensión de pico del drenador (V_pk ≈ 3,56 × Vcc). Asegúrese de que V_DSS del MOSFET exceda V_pk × factor de seguridad (típicamente 3×).

### Potencia de Salida (P_out)
Potencia RF objetivo entregada a la carga, en vatios. Esta es la potencia en el conector de antena después del LPF. Tenga en cuenta que el MOSFET ve mayor disipación cuando η < 1.

### Frecuencia de Operación (f)
Frecuencia de transmisión fundamental en MHz. Para HF QRP, los valores típicos son 1,8, 3,5, 7, 10, 14, 18, 21, 24, 28 MHz (bandas de radioaficionados). Para experimentación VLF/LF/MF: 0,136, 0,472 MHz.

### Factor Q Cargado (Q_L)
El factor Q del resonador serie L2–C2, definido como:
```
Q_L = 2π·f·L2 / R
```
Elegir Q_L implica compromisos:
- **Q_L mayor (10–15):** Mejor rechazo de armónicos del tanque, ancho de banda más estrecho, sintonización más crítica, valores de L2 mayores y C2 menores, más sensible a las tolerancias de componentes
- **Q_L menor (3–5):** Mayor ancho de banda, menos filtrado de armónicos, más fácil de construir, más tolerante a variaciones de frecuencia
- **Recomendado para HF (3–14 MHz):** Q_L = 5–10

### Vsat del Transistor (Vo)
La tensión de saturación del dispositivo de conmutación:
- **MOSFET:** 0 V (interruptor ideal, pérdida por Rdson modelada separadamente en η)
- **BJT:** 0,1–1 V (tensión de saturación colector-emisor)

### ESR_L2
La resistencia serie del bobinado del inductor L2, medida a la frecuencia de operación con un medidor LCR o VNA. Este parámetro entra en la ecuación de eficiencia de Sokal con coeficiente 1,0 — los errores aquí corrompen directamente η, R, C1, C2 e I_dc. Valores típicos:
- Solenoide de núcleo de aire pequeño: 0,03–0,08 Ω
- Toroide devanado (HF): 0,05–0,15 Ω

### ESR_C2 y ESR_C1
La resistencia serie de los capacitores resonante C2 y de derivación C1, respectivamente. Estos valores entran directamente en la ecuación de eficiencia de Sokal (ESR_C2 con coeficiente 1,0; ESR_C1 con coeficiente 0,2116). Para tipos NP0/C0G o mica plateada, el ESR es típicamente 0,01–0,05 Ω y puede dejarse en 0,01 para diseño inicial. Los dieléctricos con pérdidas (X7R, Z5U) pueden tener ESR de 0,1–0,5 Ω — mida con VNA si importa la precisión de la eficiencia.

### ESR_RFC
La resistencia serie del bobinado de la bobina de RF. La pérdida del RFC es aproximadamente `ESR_RFC × I_dc²`. Valores típicos:
- Toroide de núcleo de aire: 0,05–0,1 Ω
- Ferrita devanada (HF): 0,1–0,5 Ω
- Núcleos de ferrita pequeños: hasta 1 Ω

### Factor de Escala de Coss
La capacitancia de salida del MOSFET (Coss) es altamente **dependiente de la polarización** — disminuye bruscamente a mayor Vds. El factor de escala corrige el valor de la hoja de datos (medido a un Vds fijo, típicamente 25 V) al valor efectivo a Vds ≈ 0,5 × Vcc:

| Tipo de Dispositivo | Factor de Escala Recomendado |
|---|---|
| Si planar (BS170, 2N7000) | 0,40 |
| Si vertical DMOS (IRF510, IRFZ44N) | 0,35 |
| RF LDMOS (RD16HVF1, BLF574) | 0,85 (use Coss,er de hoja de datos si disponible) |
| Desconocido / conservador | 0,25 |

Para mayor precisión, mida Coss con un VNA a Vds ≈ 0,5 × Vcc, o use el Coss,er equivalente de energía especificado por el fabricante si se provee.

### Factor de Seguridad V_DSS
Multiplicador aplicado a la tensión de pico nominal del drenador para determinar el V_DSS mínimo requerido del MOSFET:
```
V_DSS_min = V_pk_nominal × factor_de_seguridad
```
Recomendaciones:
- **2,0×** — operación en estado estacionario ajustado óptimamente solamente
- **3,0×** — trabajo en banco HF con alguna tolerancia para desajuste
- **4,0×** — desajuste severo o diseños de alta tensión VLF

Nota: Bajo desajuste severo, V_pk puede alcanzar 5× Vcc. La hoja incluye una verificación en el peor caso para esta condición.

---

## 7. Salidas / Valores Calculados

### Frecuencia Angular ω
```
ω = 2π × f   [rad/s]
```
Usado internamente en todas las fórmulas de componentes.

### Resistencia de Carga Óptima R
La impedancia de carga del drenador Clase-E para la que el circuito logra ZVS + ZCS. Esta **no** es la impedancia de antena (50 Ω) — es la impedancia de fuente correcta para el LPF. Típica: 5–50 Ω para HF QRP.

### Corriente DC de Alimentación I_dc
```
I_dc = P_out / (Vcc × η)   [A]
```
Corriente promedio extraída de la fuente. Depende de la η convergida.

### C1_ext — Capacitor de Derivación Externo
Este es el capacitor que realmente necesita **comprar y soldar** en el drenador:
```
C1_ext = C1_total − Coss_efectivo
```
donde `Coss_efectivo = Coss_datasheet × factor_de_escala`. Si C1_ext es negativo, el propio Coss del MOSFET ya excede el C1 requerido — esto es una situación común con dispositivos DMOS verticales más grandes.

### L2 — Inductor Resonante Serie
Devane **una sola bobina** con este valor de inductancia. La hoja también puede mostrar filas `L_ser` y `L_total` — son solo informativas e iguales a L2. **No** devane bobinas separadas para L_ser.

### RFC — Bobina de RF
La inductancia mínima de la bobina es:
```
RFC_min = 30 × R / ω
```
La hoja usa `50 × R / ω` en la práctica para margen. Verificaciones críticas:
- **La FRA (frecuencia de auto-resonancia) debe ser > 10× la frecuencia de operación** — en HF, un RFC grande puede auto-resonar dentro de la banda y actuar como capacitor
- **Saturación del núcleo** — verifique que el núcleo no se sature a I_dc

---

## 8. Hoja 1 — Calculadora Clase-E

Esta es la hoja de diseño principal. El diseño está dividido en secciones:

| Sección | Descripción |
|---|---|
| **Encabezado** | Indicador de estado de convergencia e instrucciones de cálculo iterativo |
| **MODELO DE MOSFET** | Nombre del dispositivo seleccionado y verificación de conformidad V_DSS / I_D |
| **PARÁMETROS DE ENTRADA** | Celdas editables por el usuario (azul): Vcc, P_out, f, Q_L, valores ESR, datos MOSFET |
| **CÁLCULOS INTERMEDIOS** | ω, R, I_dc, V_ef — valores internos |
| **VALORES DE COMPONENTES CLASE-E** | C1, C2, L2, RFC, L_ser — todos con unidades, valores estándar y notas |
| **RENDIMIENTO / EFICIENCIA** | η, P_disipada, V_pk, I_pk, estimaciones térmicas |
| **VERIFICACIONES DE CLASIFICACIÓN DEL MOSFET** | Conformidad V_DSS, I_D con factores de seguridad |
| **LPF (50 Ω, hoja principal)** | Red L opcional + LPF de 5 elementos para usar cuando no se usa la hoja LPF Directo |

> **Nota:** La hoja principal también proporciona un transformador de impedancia en red L (de R_opt a 50 Ω) seguido de un LPF Chebyshev simétrico de 50 Ω. Esta es una alternativa a la hoja **LPF Directo (R_opt)** — los dos enfoques **no deben** cascadearse en serie. Elija uno u otro.

---

## 9. Hoja 2 — LPF Directo 5 Polos

Esta hoja calcula el LPF Chebyshev **correcto** de 5 elementos cuando el filtro está conectado **directamente al drenador**, sin red L intermedia. Todos los parámetros están automáticamente vinculados desde la hoja principal.

### Decisiones clave de diseño

| Parámetro | Valor | Razón |
|---|---|---|
| Tipo de filtro | Chebyshev Tipo I | Mejor atenuación de banda de supresión por elemento |
| Rizado | 0,1 dB | Pérdida de inserción mínima en banda de paso |
| Orden | 5 elementos | Buen rechazo de armónicos; práctico de construir |
| Corte fc | 1,40 × f | Mantiene fundamental ≤0,1 dB de pérdida de inserción |
| Z de fuente | R_opt | Correcto para conexión directa al drenador |
| Z de carga | 50 Ω | Impedancia de antena estándar |

### Ecuaciones de componentes

```
L1 = g1 × Z0_ef / ωc           [g1 = 1,1468]
C2 = g2 / (R_opt × ωc)          [g2 = 1,3712, usa R_opt — NO 50Ω]
L3 = g3 × Z0_ef / ωc           [g3 = 1,9750]
C4 = g4 / (Z_carga × ωc)        [g4 = 1,3712, usa Z_carga = 50Ω]
L5 = g5 × Z0_ef / ωc           [g5 = 1,1468, por simetría L5 = L1]

Z0_ef = √(R_opt × Z_carga)      [impedancia de referencia media geométrica]
```

Tenga en cuenta que C2 ≠ C4 cuando R_opt ≠ Z_carga (terminaciones asimétricas).

### Estimaciones de rendimiento

La hoja proporciona estimaciones de atenuación de armónicos a 2f y 3f. Estas estimaciones asumen una **fuente resistiva ideal R_opt** — en la práctica, el rechazo real diferirá ligeramente dependiendo de la impedancia de salida del MOSFET a frecuencias armónicas. Verifique con simulación LTspice antes de la construcción final.

Si el LPF de 5 elementos no cumple el requisito regulatorio (FCC −43 dBc, IARU −50 dBc), cambie a la hoja **LPF Directo 7 polos** (Hoja 3), que usa un filtro Chebyshev de 7 elementos para ≥14 dB de rechazo adicional en la banda de supresión, o baje la frecuencia de corte fc.

---

## 10. Hoja 3 — LPF Directo 7 Polos

Esta hoja calcula un **LPF Chebyshev de 7 elementos** acoplado directamente al drenador, usando R_opt como impedancia de fuente. Proporciona aproximadamente 14 dB más de rechazo en la banda de supresión que el diseño de 5 polos, apuntando a −65 dBc o mejor a 2f para cumplimiento estricto de la IARU. Todos los parámetros están automáticamente vinculados desde la hoja principal.

### Decisiones clave de diseño

| Parámetro | Valor | Razón |
|---|---|---|
| Tipo de filtro | Chebyshev Tipo I | Mejor atenuación de banda de supresión por elemento |
| Rizado | 0,1 dB | Pérdida de inserción mínima en banda de paso |
| Orden | 7 elementos | Rechazo superior de armónicos; recomendado para cumplimiento IARU −50 dBc |
| Corte fc | 1,40 × f | Mantiene fundamental ≤0,1 dB de pérdida de inserción |
| Z de fuente | R_opt | Correcto para conexión directa al drenador |
| Z de carga | 50 Ω | Impedancia de antena estándar |

### Ecuaciones de componentes

```
L1 = g1 × Z0_ef / ωc           [g1 = 1,1812]
C2 = g2 / (R_opt × ωc)          [g2 = 1,4228, usa R_opt — NO 50Ω]
L3 = g3 × Z0_ef / ωc           [g3 = 2,0967]
C4 = g4 / (Z0_ef × ωc)          [g4 = 1,5734, usa Z0_ef (media geométrica)]
L5 = g5 × Z0_ef / ωc           [g5 = 2,0967, por simetría L5 = L3]
C6 = g6 / (Z_carga × ωc)        [g6 = 1,4228, usa Z_carga = 50Ω]
L7 = g7 × Z0_ef / ωc           [g7 = 1,1812, por simetría L7 = L1]

Z0_ef = √(R_opt × Z_carga)      [impedancia de referencia media geométrica]
```

Nota: C2 ≠ C4 ≠ C6 debido a terminaciones asimétricas. El capacitor de derivación central C4 usa Z0_ef (media geométrica), a diferencia del diseño de 5 polos donde solo se usan las impedancias de fuente y carga.

### Estimaciones de rendimiento

La hoja proporciona estimaciones de atenuación de armónicos a 2f y 3f. Estas estimaciones asumen una **fuente R_opt ideal** — verifique con simulación LTspice. El diseño de 7 polos apunta a −65 dBc o mejor a 2f y es la elección recomendada cuando se debe cumplir con margen el estricto IARU −50 dBc.

⚠ **Advertencia de filtro asimétrico**: Las fórmulas de atenuación asumen terminación de fuente igual a R_opt. Para relaciones de impedancia n = Z_carga/R_opt > 1,5, la profundidad real de la banda de supresión puede diferir ±3–6 dB de las predicciones de Chebyshev. Verifique con LTspice.

> **Elegir entre 5 polos y 7 polos:** Use los 5 polos (Hoja 2) para la mayoría de los diseños QRP. Use los 7 polos (Hoja 3) cuando la hoja de 5 polos marque incumplimiento 🚨, cuando se requieran márgenes IARU más estrictos, o cuando opere cerca de 2f.

---

## 11. Hoja 4 — Tabla de Referencia de MOSFET

Una tabla de referencia rápida de 16 MOSFETs comúnmente usados en diseños Clase-E HF. Columnas:

| Columna | Descripción |
|---|---|
| N° de Parte | Nombre del dispositivo |
| Encapsulado | Paquete físico |
| V_DSS | Tensión de ruptura drenador-fuente (V) |
| I_D | Corriente de drenador (A) |
| R_ds(on) | Resistencia encendida (Ω) |
| C_iss / C_oss | Capacitancia de entrada/salida (pF) — dependiente de Vds |
| P_D | Disipación de potencia del encapsulado (W) |
| f_T | Frecuencia de ganancia unidad (MHz) |
| P_RF Típica | Potencia de salida RF alcanzable (W) |
| Rango de Freq. RF | Rango de frecuencia de operación adecuado |
| Aplicaciones | Notas sobre casos de uso típicos |
| Escala Coss | Factor de escala del punto de polarización recomendado para este diseño |
| Tipo de Dispositivo | Si-planar / Si-DMOS / LDMOS |

### Aspectos destacados de selección de dispositivos

| Aplicación | Dispositivo Recomendado |
|---|---|
| Baliza WSPR QRP, 20m–80m, <0,5W | BS170 (TO-92), 2N7000 |
| CW QRP, todas las bandas HF, 1–30W | IRF510 (TO-220) — el MOSFET clásico de radioaficionados |
| Potencia media, 40–80m, 10–80W | IRF520, IRF530 |
| Alta potencia, 160m–40m, 50–150W | IRF540, IRFP250 |
| VLF/LF/MF (136 kHz, 472 kHz) | IRFZ44N, IXFN55N50 |
| HF/VHF optimizado para RF, 10m–40m | RD16HVF1 (LDMOS) |
| HF profesional, 100–300W | BLF574, MRF101AN (LDMOS) |

> ⚠ **VN66AF es obsoleto** (descontinuado ~2010). Reemplace con BS170 o 2N7000 en diseños nuevos.

---

## 12. Cálculo Iterativo: Por Qué y Cómo

### La dependencia circular

Tres cantidades forman un lazo de retroalimentación:

```
η (eficiencia) → I_dc = P_out/(Vcc·η) → pérdida_RFC = ESR_RFC × I_dc² → η
```

Esta referencia circular no puede resolverse en un solo paso — requiere iteración. Los otros componentes (R, C1, C2, L2, RFC) dependen solo de entradas fijas y **no** forman parte de este lazo; son válidos desde el primer paso de cálculo.

### Cómo converger

1. Abra la planilla y active el cálculo iterativo (vea Inicio Rápido)
2. Ingrese sus parámetros
3. Presione **F9** (recalcular) una o dos veces
4. Verifique el indicador de convergencia en la celda C6:
   - `✅ Cálculo iterativo CONVERGIDO — valores estables` → listo
   - Aún mostrando el valor anterior o sin converger → continúe presionando F9
5. Para ESR_RFC muy alto o puntos de operación inusuales, la convergencia puede requerir 3–5 presiones de F9

### Por qué converge

El término de pérdida del RFC `ESR_RFC × I_dc²` es típicamente una pequeña fracción de P_out para circuitos HF QRP bien diseñados. La ganancia de retroalimentación es mucho menor que 1, por lo que la iteración siempre converge. Si no converge, verifique que ESR_RFC no sea excesivamente grande.

---

## 13. Guía de Selección de Componentes

### 13.1 C1 — Capacitor de Derivación

- **Tipo:** Cerámica NP0/C0G o mica plateada
- **Clasificación de tensión:** ≥2× V_pk (es decir, ≥ 2 × 3,56 × Vcc); use 100 V mínimo para HF QRP
- **Valor:** Use el valor estándar E12/E24 más cercano a C1_ext. Una ligera desviación del óptimo desplaza el punto ZVS pero no falla catastróficamente el circuito
- **Nota:** El Coss del MOSFET (escalado) absorbe parte del C1 requerido. Instale solo C1_ext como componente externo

### 13.2 C2 — Capacitor Resonante Serie

- **Tipo:** Película de polipropileno (WIMA FKP, MKP) o mica plateada
- **Clasificación de tensión:** ≥2× V_pk — este capacitor ve la excursión completa de tensión del drenador
- **Estabilidad:** Use C0G/NP0 o polipropileno; evite X7R/Z5U que derivan con la temperatura y causan inestabilidad de frecuencia
- **Valor:** Valor de serie E de 5% más cercano; las pequeñas desviaciones pueden absorberlas ajustando ligeramente L2

### 13.3 L2 — Inductor Resonante Serie

- **Núcleo:** T68-6 (amarillo, mezcla 6) o toroide de polvo de hierro T50-6 para 7–30 MHz; T68-2 (rojo) para 1,8–10 MHz
- **Hilo:** Use AWG apropiado para I_rms (vea la hoja del LPF para orientación sobre clasificación de corriente)
- **Devanado:** Devane como un solenoide de vuelta cerrada o sobre toroide; deje un pequeño espacio entre el inicio y el final
- **Verificación de FRA:** La frecuencia de auto-resonancia debe ser >10× la frecuencia de operación
- **Verificación:** Mida la inductancia terminada con medidor LCR o VNA a la frecuencia de operación antes de la instalación; ajuste separando/comprimiendo las vueltas

### 13.4 RFC — Bobina de RF

- **Requisito crítico:** FRA > 10× la frecuencia de operación (p.ej. >70 MHz para un diseño de 7 MHz)
- **Núcleo:** La ferrita mezcla 43 o 61 funciona bien en HF; evite el polvo de hierro para valores RFC grandes (demasiado pérdidas a estas frecuencias)
- **Q de devanado:** Apunte a Q sin carga > 100 a la frecuencia de operación
- **Saturación:** El núcleo no debe saturarse a la corriente de polarización DC I_dc; verifique los valores AL del fabricante
- **Regla práctica:** La reactancia del RFC debe ser ≥ 30× R; esta planilla usa 50× R para margen

### 13.5 Componentes del LPF

- **Inductores (L1, L3, L5 para 5 polos; L1, L3, L5, L7 para 7 polos):** Toroides T50-6 o T68-6; verifique FRA > 10×f; devane al valor calculado en µH
- **Capacitores (C2, C4):** NP0/C0G o mica plateada clasificados ≥ 2× V_pk (≥ 100 V para diseños QRP de 5–15 V)
- **Vestido de leads:** Mantenga los leads cortos; separe físicamente la entrada y la salida para evitar acoplamiento
- **Blindaje:** Para el mejor rechazo de armónicos, aloje el LPF en un gabinete de hojalata

---

## 14. Guía de Selección de MOSFET

### V_DSS Mínimo
```
V_DSS_requerido = V_pk_nominal × factor_de_seguridad = 3,56 × Vcc × 3,0
```
Para Vcc = 5 V: V_DSS_requerido ≈ 53 V → BS170 (60 V) es marginal; IRF510 (100 V) es seguro.

### Requisitos de excitación de puerta
- La capacitancia de puerta (Ciss) debe cargarse/descargarse completamente dentro del tiempo de transición del interruptor
- Para controladores de MOSFET, asegúrese de que la tensión de excitación de puerta exceda V_gs(th) con un margen cómodo (2–3×)
- Use un resistor de amortiguación de puerta (R_puerta = 10–47 Ω) para prevenir oscilación en el lazo de puerta

### fT del MOSFET vs frecuencia de operación
Para conmutación eficiente, la frecuencia de ganancia unidad fT del MOSFET debe ser ≥ 10× la frecuencia de operación. Dispositivos como el IRFZ44N (fT ≈ 30 MHz) no son adecuados por encima de 3 MHz. Los dispositivos LDMOS (fT > 500 MHz) funcionan bien hasta 50 MHz y más allá.

---

## 15. Cumplimiento Regulatorio (Armónicos)

| Regulación | Límite de armónicos |
|---|---|
| FCC Parte 97 (EE.UU.) | −43 dBc para P > 5W; −40 dBc para P ≤ 5W |
| Recomendación IARU | −50 dBc |
| CEPT/UE (radioaficionados) | −43 dBc mínimo; −50 dBc recomendado |

Un LPF Chebyshev de 5 elementos proporciona aproximadamente:
- 2° armónico (2f): −17 a −25 dBc (depende de R_opt, elección de fc)
- 3° armónico (3f): −38 a −55 dBc

Para cumplimiento a alta potencia o cumplimiento estricto de la IARU, considere:
1. Cambiar al **LPF Chebyshev de 7 elementos** (Hoja 3 — LPF Directo 7 polos), que proporciona ≥14 dB de rechazo adicional
2. Bajar fc de 1,40×f a 1,25×f (aumenta ligeramente la pérdida de inserción del fundamental)
3. Agregar una **trampa de armónicos** separada (LC serie a GND sintonizado a 2f o 3f)
4. Verificar con un **analizador de espectro** — siempre mida antes de transmitir

---

## 16. Flujo de Trabajo de Simulación (LTspice)

LTspice (gratuito de Analog Devices: https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html) es la herramienta recomendada para validación previa a la construcción. *(Nota: si la URL redirige, busque "LTspice download Analog Devices" para encontrar la página de descarga actual.)*

### Modelo de simulación básico

1. Cree una fuente de tensión `V1 = PULSE(0 {Vgs_on} 0 {tr} {tf} {Ton} {T})` para la excitación de puerta
2. Use un interruptor ideal (`SW`) o modelo SPICE del MOSFET del fabricante
3. Agregue Coss como un capacitor fijo en paralelo con el drenador-fuente del interruptor
4. Use los valores calculados C1_ext, C2, L2, RFC de la planilla como valores iniciales
5. Ejecute la simulación `.tran` para ≥20 ciclos RF para alcanzar el estado estacionario
6. Grafique V(drenador) e I(drenador) — verifique ZVS (V_drenador → 0 al encender el interruptor)
7. Agregue el LPF y ejecute `.fourier` o `.meas` para verificar los niveles de armónicos

### Ajuste en simulación

- Si V_drenador no llega a cero antes del encendido: aumente C1 ligeramente
- Si V_drenador cae por debajo de cero: disminuya C1 ligeramente
- Ajuste fino L2 ±5–10% para optimizar ZVS y potencia de salida simultáneamente
- Adapte la potencia de salida al objetivo ajustando C2 alrededor del valor calculado

---

## 17. Limitaciones Conocidas

- El **modelo de eficiencia** está basado en el modelo lineal de ESR de la Ec. 2 de Sokal. Los mecanismos de pérdida no lineales (histéresis del núcleo, efecto pelicular en VHF, pérdida de Coss) están parcialmente considerados a través del modelo de pérdida de Coss (`p_coss = 0,5 × Coss_ef × V_pk² × f × k_coss`) pero no completamente modelados
- Las **estimaciones de atenuación de armónicos** en la hoja del LPF asumen una fuente resistiva ideal a R_opt. La impedancia de salida real del MOSFET a frecuencias armónicas es reactiva y diferente de R_opt — siempre verifique con simulación o medición
- El **LPF de 5 elementos** puede no lograr cumplimiento FCC/IARU en todos los puntos de operación. La hoja marca el incumplimiento con advertencias 🚨. Se recomienda un diseño de 7 elementos para cumplimiento estricto
- **Coss depende de Vds**. El enfoque del factor de escala es una aproximación. Para mayor precisión, use Coss,er (equivalente de energía) especificado por el fabricante o mida a Vds ≈ 0,5×Vcc
- La calculadora asume **D = 0,5 (ciclo de trabajo del 50%)**, que es requerido para las ecuaciones de Sokal. Los diseños de ciclo de trabajo no-50% requieren ecuaciones diferentes no implementadas aquí
- La calculadora es válida para **QL ≥ 1,7879** (límite inferior del ajuste racional de la Tabla I de Sokal). QL < 1,79 está fuera del espacio de diseño válido para Clase-E óptimo
- Las **condiciones ZVS/ZDVS** pueden violarse a altas relaciones de impedancia (Z_carga/R_opt > 10). Siempre verifique con simulación LTspice cuando R_opt es muy bajo (< 5 Ω)

---

## 18. Referencias

1. **Sokal, N. O.** — "Class-E RF Power Amplifiers," *QEX Magazine*, No. 204, Ene/Feb 2001, pp. 9–20. American Radio Relay League. *(Referencia primaria para todas las ecuaciones de diseño en esta calculadora)*

2. **Sokal, N. O.** — "Class-E High-Efficiency RF/Microwave Power Amplifiers: Principles of Operation, Design Procedures, and Experimental Verification," en *Analog Circuit Design* (AACD 2001), Kluwer Academic, 2002. Disponible: https://people.eecs.berkeley.edu/~culler/AIIT/papers/radio/Sokal%20AACD5-poweramps.pdf

3. **Sokal, N. O. y Sokal, A. D.** — "Class E — A New Class of High-Efficiency Tuned Single-Ended Switching Power Amplifiers," *IEEE Journal of Solid-State Circuits*, Vol. SC-10, No. 3, pp. 168–176, Junio 1975. *(Publicación original de la topología Clase-E)*

4. **Raab, F. H.** — "Idealized Operation of the Class E Tuned Power Amplifier," *IEEE Transactions on Circuits and Systems*, Vol. CAS-24, No. 12, pp. 725–735, Diciembre 1977.

5. **Williams, A. B. y Taylor, F. J.** — *Electronic Filter Design Handbook*, 4ª ed., McGraw-Hill, 2006. *(Fuente para valores g del LPF Chebyshev asimétrico y escalado de escalera)*

6. **Fajardo, A. y de Sousa, F. R.** — "Design of the Class-E Power Amplifier with Finite DC Feed Inductance under Maximum-Rating Constraints," *Applied Sciences*, Vol. 11, No. 9, 2021. DOI: 10.3390/app11093727. Disponible: https://www.mdpi.com/2076-3417/11/9/3727

7. **Dobbs, G. (G3RJV)** — "A Short Guide to Harmonic Filters for QRP Transmitter Output," GQRP Club. Disponible: https://www.gqrp.com/harmonic_filters.pdf

8. **VK1SV Clase-E para Principiantes** — Tutorial práctico de diseño Clase-E con ejemplos LTspice. Disponible: https://people.physics.anu.edu.au/~dxt103/class-e/

9. **Calculadora Online Clase-E VK2ZAY** — Calculadora web basada en las ecuaciones de Sokal. Disponible: https://people.physics.anu.edu.au/~dxt103/calculators/class-e.php

10. **RF Cafe — Valores de Elementos Prototipo Chebyshev** — Tablas de valores g Chebyshev normalizados. Disponible: https://www.rfcafe.com/references/electrical/cheby-proto-values.htm

11. **Engineering LibreTexts — Aproximación Pasa-Bajos Chebyshev** — Fundamento teórico. Disponible: https://eng.libretexts.org/Bookshelves/Electrical_Engineering/Electronics/Microwave_and_RF_Design_IV:_Modules_(Steer)/02:_Filters/2.05:_The_Chebyshev_Lowpass_Approximation

12. **Kazimierczuk, M. K.** — *RF Power Amplifiers*, 2ª ed., Wiley, 2015. *(Libro de texto exhaustivo que cubre la teoría Clase-E, condiciones ZVS/ZDVS y efectos parásitos del MOSFET a frecuencias RF)*

13. **Kee, S. D., Aoki, I., Hajimiri, A., y Rutledge, D.** — "The Class-E/F Family of ZVS Switching Amplifiers," *IEEE Transactions on Microwave Theory and Techniques*, Vol. 51, No. 6, pp. 1677–1690, Junio 2003. *(Extiende la teoría Clase-E a topologías Clase-E/F; útil como fondo para sintonización de armónicos)*

14. **Kazimierczuk, M. K. y Puczko, K.** — "Exact Analysis of Class E Tuned Power Amplifier at any Q and Switch Duty Cycle," *IEEE Transactions on Circuits and Systems*, Vol. CAS-34, No. 2, pp. 149–159, Febrero 1987. *(Análisis exacto en forma cerrada sin la suposición de Q infinito; útil cuando QL < 5)*

15. **Wetherhold, E. (W3NQN)** — "Second-Harmonic-Optimized Low-Pass Filters," *QST*, Febrero 1999, pp. 44–48. American Radio Relay League. *(Introduce la topología CWAZ Chebyshev con cero; contexto para elegir entre LPF estándar de 50 Ω y conexión directa al drenador)*

---

## 19. Licencia

Esta planilla y la documentación asociada se distribuyen bajo la **Licencia Pública General GNU v3 (GPL v3)**.

```
Copyright (C) LU3VEA — lu3vea@gmail.com

Este programa es software libre: puede redistribuirlo y/o modificarlo
bajo los términos de la Licencia Pública General GNU publicada por
la Free Software Foundation, ya sea la versión 3 de la Licencia, o
(a su elección) cualquier versión posterior.

Este programa se distribuye con la esperanza de que sea útil,
pero SIN NINGUNA GARANTÍA; sin siquiera la garantía implícita de
COMERCIABILIDAD o APTITUD PARA UN PROPÓSITO PARTICULAR. Consulte
la Licencia Pública General GNU para más detalles.
```

Texto completo de la licencia: https://www.gnu.org/licenses/gpl-3.0.html

---

*73 de LU3VEA*
