# Especificación del artículo (fuente: propuesta IEEE / Scopus)

Requisitos del artículo tal como fueron entregados (2026-10-28), con las
ecuaciones corregidas a LaTeX.

> **Precedencia**: esta propuesta es orientativa. La fuente de verdad del sistema
> es `specs/001-energy-monitoring-system/` (y la constitución). Cuando esta
> propuesta difiere de las especificaciones, el artículo sigue las
> especificaciones; cada diferencia queda resuelta en
> [README.md › Decisiones](README.md#decisions) y no se edita aquí.

## Identificación

- **Tema original**: Sistema de Monitoreo y Análisis de Consumo Eléctrico en Tiempo Real.
- **Título**: *Design, Implementation, and Experimental Evaluation of an IoT-Based
  System for Real-Time Electrical Energy Consumption Monitoring and Analysis*.
- **Áreas**: IoT, Smart Energy, Smart Grids, Sistemas Embebidos, Eficiencia
  Energética, Análisis de Datos.
- **Destino**: congresos IEEE / revistas indexadas en Scopus. Formato IEEE.
- **Longitud de referencia**: seis páginas.
- **Nivel objetivo**: intermedio (monitoreo IoT, análisis de patrones y validación
  frente a instrumento de referencia). Anomalías/predicción solo si hay datos
  suficientes, preferentemente en una versión posterior.
- **Riesgo editorial principal**: presentar solo "un medidor conectado a internet".
  Debe demostrarse exactitud, confiabilidad y utilidad analítica con resultados
  reproducibles.

## Diferenciación de conceptos

Medición (obtener V, I, P, E) · Monitoreo (registrar y visualizar continuamente) ·
Análisis (tendencias, picos, anomalías) · Predicción (modelos estadísticos/ML) ·
Optimización (acciones de reducción). El título exige monitoreo y análisis; **no**
exige IA ni optimización.

## Problema, pregunta, objetivo

- **Problema**: lecturas periódicas o facturación acumulada dificultan identificar
  oportunamente variaciones de demanda, picos y patrones. La utilidad de un sistema
  conectado depende de exactitud, calidad de datos, latencia e indicadores
  interpretables.
- **Pregunta**: ¿Qué desempeño presenta un sistema IoT de monitoreo y análisis del
  consumo eléctrico en términos de exactitud de medición, latencia de
  actualización, confiabilidad de transmisión e identificación de patrones de
  consumo?
- **Objetivo general**: diseñar, implementar y evaluar experimentalmente un sistema
  IoT para el monitoreo y análisis del consumo eléctrico en tiempo real, mediante
  adquisición, transmisión y procesamiento de variables eléctricas, para
  caracterizar patrones de consumo y evaluar su desempeño frente a instrumentos de
  referencia.
- **Contribuciones**:
  1. Arquitectura IoT para adquisición, transmisión y almacenamiento.
  2. Módulo de análisis de patrones de consumo.
  3. Evaluación de exactitud de medición y desempeño de comunicación.
  4. Comparación experimental con instrumento de referencia.
  5. Análisis de limitaciones en diferentes condiciones de carga.
- No se requiere hipótesis formal (desarrollo tecnológico y validación experimental).

## Estructura (4 secciones) y distribución

| Sección | Peso | ≈ páginas (de ~5.5 de cuerpo) |
|---|---|---|
| I. Introducción | 25 % | 1.4 |
| II. Metodología | 35 % | 1.9 |
| III. Resultados y discusión | 32 % | 1.8 |
| IV. Conclusiones | 8 % | 0.4 |

### I. Introducción
- **A. Contexto**: gestión eficiente de energía; necesidad de información
  detallada; limitaciones de la facturación acumulada para patrones horarios;
  oportunidades IoT; aplicaciones (viviendas, edificios, laboratorios, pequeña
  industria). Incluir estadísticas verificables del ámbito local.
- **B. Antecedentes**: 10–15 trabajos, preferentemente 2021–2026: IoT-based Energy
  Monitoring Systems, Smart Energy Meters, Real-Time Electricity Consumption
  Monitoring, Energy Data Analytics, Low-Cost Power Measurement Systems,
  Electrical Load Monitoring, Energy Consumption Anomaly Detection. Comparar
  sensores, microcontroladores, variables, métodos de calibración, indicadores de
  error y procedimientos experimentales.
- **C. Limitaciones previas** (solo si la literatura lo respalda): validación
  insuficiente frente a instrumentos calibrados; cargas simples; falta de análisis
  de factor de potencia; dependencia de conectividad continua; escasa evaluación de
  latencia; ausencia de análisis de incertidumbre; limitaciones para comportamientos
  anómalos.
- **D. Brecha**: integrar monitoreo, análisis de patrones y validación metrológica
  en un sistema IoT de bajo costo, evaluando a la vez exactitud, latencia y
  confiabilidad. La revisión debe confirmar que es diferenciada.
- **E. Objetivo y contribuciones** al cierre.

### II. Metodología
- **A. Diseño**: investigación aplicada y desarrollo tecnológico; cuantitativo;
  evaluación experimental; unidad de análisis = mediciones en intervalos
  definidos; escenario de baja tensión; variables: V, I, P, E, latencia,
  exactitud, confiabilidad.
- **B. Arquitectura** (Fig. 1): carga → medición (V, I, P, E, PF) →
  microcontrolador (adquisición, validación, procesamiento, transmisión) →
  almacenamiento (BD, histórico) → panel (consumo, tendencias, picos, alertas).
- **C. Prototipo** (Fig. 2): componentes y justificación. Alternativas sugeridas:
  PZEM-004T u otro módulo, ESP32 u otro MCU, Wi-Fi/MQTT/HTTPS, servidor y BD,
  aplicación web, instrumento de referencia calibrado. Incluir nota de seguridad:
  conexiones a red diseñadas y verificadas por personal competente.
- **D. Modelo de adquisición**: variables $V_{\mathrm{RMS}}$ [V],
  $I_{\mathrm{RMS}}$ [A], $P$ [W], $E$ [kWh], $PF$ [–], $f$ [Hz].
  $P = V_{\mathrm{RMS}} I_{\mathrm{RMS}} PF$;
  $E = \frac{1}{3\,600\,000}\int_{t_0}^{t_1} P(t)\,dt$ (P en W, t en s, E en kWh).
  Precisar si P se mide directamente o se estima.
- **E. Algoritmo** (Fig. 3): adquisición periódica; validación y datos faltantes;
  almacenamiento con marca de tiempo; consumo acumulado por intervalo; períodos de
  alta demanda; variaciones anómalas (umbral o desviación respecto a referencia);
  reportes y alertas. IA no necesaria.
- **F. Diseño experimental**: cargas resistivas (PF≈1), inductivas (PF≠1),
  electrónicas (no lineales), variación de potencia, operación prolongada,
  interrupción de red. Repeticiones justificadas con pilotos y criterio
  estadístico.
- **G. Métricas**: error absoluto (V, A, W, kWh); error relativo
  $\mathrm{Error}(\%) = \frac{|X_m - X_r|}{|X_r|}\times 100$ (solo con $X_r$
  suficientemente alejado de cero); precisión (variabilidad de repeticiones);
  latencia (s); registros entregados (%); exactitud de detección de eventos (%);
  tiempo de procesamiento (ms); disponibilidad (%).

### III. Resultados y discusión
- **A. Implementación** (Fig. 4): prototipo funcionando, captura, transmisión,
  almacenamiento, visualización, indicadores y alertas.
- **B. Exactitud**: tabla referencia vs. sistema vs. error para V, I, P, E, PF;
  Fig. 5.
- **C. Patrones**: consumo por hora/día, máxima demanda, variaciones de potencia,
  comportamiento por carga, eventos anómalos; Fig. 6.
- **D. Comunicación y rendimiento**: latencia media y p95, disponibilidad,
  recuperación tras desconexión, tiempo de procesamiento. "Tiempo real" respaldado
  por frecuencia y límite de latencia definidos (tiempo real flexible).
- **E. Comparación con trabajos previos**: exactitud, variables, frecuencia,
  latencia, costo, confiabilidad, capacidad analítica; reconocer diferencias de
  protocolo.
- **F. Limitaciones**: exactitud de sensores, incertidumbre, conectividad, cargas no
  lineales, variabilidad de muestreo, escalabilidad, escenario.

### IV. Conclusiones
Implementación tecnológica; desempeño experimental; contribución científica;
trabajo futuro (predicción, anomalías avanzadas, análisis por circuito, gestión
energética). **No afirmar ahorros energéticos ni económicos** sin intervención
evaluada.

## Figuras y tablas prioritarias

| Elemento | Contenido |
|---|---|
| Fig. 1 | Arquitectura general |
| Fig. 2 | Prototipo físico y conexiones |
| Fig. 3 | Algoritmo de adquisición y análisis |
| Fig. 4 | Interfaz de monitoreo |
| Fig. 5 | Comparación de mediciones |
| Fig. 6 | Patrones de consumo |
| Tabla | Comparación con trabajos relacionados |
| Tabla | Exactitud experimental |
| Tabla | Rendimiento del sistema |

Para seis páginas, seleccionar las de mayor evidencia y evitar redundancias.
(La numeración de tablas difiere entre §3 y §4 de la propuesta original; se fija
por orden de aparición en el texto, como exige IEEE.)
