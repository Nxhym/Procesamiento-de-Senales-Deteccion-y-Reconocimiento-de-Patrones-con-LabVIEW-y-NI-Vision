# 🔬 Procesamiento de Señales: Detección y Reconocimiento de Patrones con LabVIEW y NI Vision

Proyecto y práctica de laboratorio desarrollada en el entorno de instrumentación virtual y programación gráfica **National Instruments LabVIEW** para el procesamiento digital de imágenes, adquisición de video en tiempo real, extracción de planos de color, correlación de plantillas (*Pattern Matching*) y reconocimiento invariante a escala y rotación (*Geometric Matching*).

---

## 📑 Tabla de Contenidos
1. [Visión General del Proyecto](#-visión-general-del-proyecto)
2. [Estructura y Ejercicios Prácticos](#-estructura-y-ejercicios-prácticos)
3. [Arquitectura del Diagrama de Bloques (Block Diagram)](#-arquitectura-del-diagrama-de-bloques-block-diagram)
4. [Requerimientos de Software y Controladores (Drivers)](#-requerimientos-de-software-y-controladores-drivers)
5. [Guía de Configuración Paso a Paso](#-guía-de-configuración-paso-a-paso)
6. [Resultados y Comparativa de Algoritmos](#-resultados-y-comparativa-de-algoritmos)

---

## 🎯 Visión General del Proyecto

El objetivo de este proyecto es implementar un sistema de **visión artificial en tiempo real** capaz de identificar y localizar patrones u objetos de interés (logotipos de referencia) a partir de un flujo de video continuo proveniente de una cámara conectada al equipo.

Se analizan dos métodos de coincidencia de imágenes:
1. **Coincidencia por Escala de Grises / Correlación Estándar (*Normalized Cross-Correlation*)**: Procedimiento rápido para patrones fijos en orientación y dimensiones similares a la plantilla.
2. **Coincidencia Geométrica (*Geometric Matching / Edge-based Matching*)**: Algoritmo avanzado que extrae contornos y curvas vectoriales, garantizando la detección del objeto aun cuando este sufra rotación en el plano 2D, variaciones en el zoom/distancia de la cámara o cambios de escala considerables.

---

## 🛠️ Estructura y Ejercicios Prácticos

### Ejercicio 1: Calibración y Prueba de Cámara en NI MAX
- Detección e inspección de dispositivos de video (cámaras integradas, cámaras web USB DirectShow y cámaras de red) mediante el explorador **NI MAX (Measurement & Automation Explorer)**.
- Verificación del modo de video ($1280 \times 720$ a 10 FPS), espacio de color (`BGRA 8 Packed` / `YUY2`), tiempo de espera (*Timeout*) y área de captura (*Region of Interest*).
- Comprobación de transmisión continua en vivo usando la función **Grab**.

### Ejercicio 2: Detección de Logotipo con Escala de Grises y Plantilla
- Inicialización del búfer de memoria gráfica con el SubVI `IMAQ Create`.
- Captura continua mediante `Vision Acquisition` dentro de una estructura `While Loop`.
- Extracción del plano de color (*Color Plane Extraction*) hacia escala de grises de 8 bits.
- Definición de una plantilla (*Template*) correspondiente al logotipo de referencia y ejecución de *Pattern Matching*.
- Despliegue en el *Front Panel* de las coordenadas espaciales $(X, Y)$, ángulo de inclinación, escala y puntaje de coincidencia (*Score*).

### Ejercicio 3: Invarianza Geométrica ante Rotación y Escala (*Geometric Matching*)
- Detección de figuras cuando varían de tamaño o presentan un giro físico frente al sensor óptico.
- Configuración de extracción de curvas en el *Vision Assistant*:
  - **Umbral de borde (*Edge Threshold*)**: `50`.
  - **Tamaño de filtro de bordes (*Edge Filter Size*)**: `Normal`.
  - **Longitud mínima de curva (*Minimum Length*)**: `25 px`.
- Definición de parámetros de búsqueda tolerantes:
  - **Búsqueda con rotación (*Rotated*)**: rango de búsqueda habilitado (hasta $360^\circ$).
  - **Búsqueda con escala (*Scaled*)**: rango dinámico admitido ($50\%$ a $110\%$).
  - **Tolerancia de oclusión (*Occluded*)**: $0\%$ a $25\%$.
  - **Puntaje mínimo de coincidencia (*Minimum Match Score*)**: $\ge 600$.

---

## 🏗️ Arquitectura del Diagrama de Bloques (Block Diagram)

El sistema opera dentro de un bucle `While Loop` temporizado por la captura de fotogramas y finalizado mediante un botón booleano `Stop`:

```text
[IMAQ Create] (Asignación de Buffer en Memoria)
      │
      ▼
+------------------- While Loop (Adquisición Continua) ---------------+
|                                                                     |
|  [Vision Acquisition] ──► [Color Plane Extraction] (Gris 8-bit)    |
|         │                                │                          |
|         │                                ▼                          |
|         │                       [Vision Assistant]                  |
|         │                       (Geometric Matching)                |
|         │                                │                          |
|         │             ┌──────────────────┴──────────────────┐       |
|         ▼             ▼                                     ▼       |
|  [Image Display]  [Matches Array (X, Y, Angle, Score)]  [Match Count]|
|                                                                     |
+---------------------------------------------------------------------+
      │
      ▼
[IMAQ Dispose] (Liberación de Memoria y Recursos de Cámara)
