# Especificación de Requisitos de Software (SRS)

**Nombre del Proyecto:** ResucitóLive 
**Versión:** 0.1  
**Fecha:** 11/02/2026  
**Autor:** Andrés Marín Pérez  
**Estado:** Borrador Inicial  

---

## 1. Introducción

### 1.1 Propósito
El propósito de este documento es definir los requisitos funcionales y no funcionales para el desarrollo de una aplicación de escritorio destinada a músicos del Camino Neocatecumenal. La aplicación facilitará el aprendizaje y la ejecución de cantos litúrgicos mediante el procesamiento del PDF oficial "Resucitó" y el seguimiento en tiempo real (*score following*) de la voz y la guitarra del usuario.

### 1.2 Alcance
El sistema abarcará:
* Ingesta y visualización fiel del PDF "Resucitó".
* Extracción de datos semánticos (letra vs. acordes) del archivo.
* Motor de audio híbrido (Voz + Armonía) para detectar la posición en el canto.
* Interfaz gráfica de usuario (GUI) con desplazamiento automático (*scrolling*) y visualización de acordes.
* Funcionamiento 100% desconectado (Offline).

---

## 2. Descripción General

### 2.1 Perspectiva del Producto
El software actúa como una capa de "Realidad Aumentada Musical" sobre un documento estático. A diferencia de lectores de PDF tradicionales, este sistema "entiende" la música contenida en la página y reacciona a la interpretación del usuario.

### 2.2 Características de los Usuarios
* **Perfil:** Salmistas y guitarristas de comunidades neocatecumenales.
* **Habilidades Técnicas:** Variables. La interfaz debe ser extremadamente simple.
* **Equipamiento:** Ordenador portátil (Windows/Mac/Linux), micrófono integrado o externo, guitarra española (con uso frecuente de cejilla/capo).

---

## 3. Requisitos Funcionales (RF)

### 3.1 Módulo A: Ingesta y Procesamiento (Backend PDF)

| ID         | Requisito                    | Descripción                                                  | Prioridad |
| :--------- | :--------------------------- | :----------------------------------------------------------- | :-------- |
| **RF-A01** | **Carga de Archivo**         | El sistema permitirá cargar el archivo `RESUCITO XX EDICION 2014.pdf`. | Alta      |
| **RF-A02** | **Renderizado de Imagen**    | Conversión de páginas PDF a imágenes de alta resolución (PNG/JPG) para su uso como fondo en la GUI (vía `pdf2image`). | Alta      |
| **RF-A03** | **Mapeo de Coordenadas**     | Extracción de texto y sus coordenadas (`x`, `y`, `width`, `height`) usando `pdfplumber`. | Alta      |
| **RF-A04** | **Clasificación Semántica**  | Algoritmo basado en Regex para distinguir si un token de texto es **Letra** (ej: "Señor") o **Acorde** (ej: "Re-", "Fa7", "Sol#"). | Alta      |
| **RF-A05** | **Normalización de Acordes** | Conversión interna de notación latina (PDF) a notación anglosajona (Sistema).<br>*Ejemplo:* `Re-` → `Dm`; `Sol7` → `G7`. | Media     |

### 3.2 Módulo B: Motor de Audio Híbrido (Core)

| ID         | Requisito                        | Descripción                                                  | Prioridad |
| :--------- | :------------------------------- | :----------------------------------------------------------- | :-------- |
| **RF-B01** | **Captura de Audio**             | Acceso al micrófono del sistema con gestión de *buffers* de baja latencia. | Alta      |
| **RF-B02** | **Reconocimiento de Voz (Vosk)** | Transcripción *offline* de voz a texto y *Fuzzy Matching* (coincidencia aproximada) con la letra extraída del PDF. Ventana de búsqueda: +/- 5 palabras. | Alta      |
| **RF-B03** | **Detección Armónica (Chroma)**  | Cálculo de FFT/Chromagrama en tiempo real para identificar el acorde dominante que está sonando. | Alta      |
| **RF-B04** | **Gestión de Cejilla (Capo)**    | Selector en la UI para indicar el traste de la cejilla (0-9). El sistema debe transponer matemáticamente el acorde esperado antes de compararlo con el audio detectado. | Alta      |
| **RF-B05** | **Fusión de Sensores**           | Lógica de decisión ponderada:<br>- Si `Confianza_Voz` > 60% → Mover cursor.<br>- Si `Confianza_Voz` < 60% Y `Confianza_Guitarra` > 80% → Mover cursor.<br>- En caso contrario → Esperar. | Alta      |

### 3.3 Módulo C: Interfaz de Usuario (Frontend Yousician-style)

| ID         | Requisito                     | Descripción                                                  | Prioridad |
| :--------- | :---------------------------- | :----------------------------------------------------------- | :-------- |
| **RF-C01** | **Vista de Scroll Vertical**  | Las líneas del PDF se recortan y se presentan en una lista continua que se desplaza hacia arriba automáticamente. | Alta      |
| **RF-C02** | **Línea de Actuación**        | Zona fija horizontal (hilo de ejecución) donde el usuario debe leer. El contenido fluye hacia esta línea. | Alta      |
| **RF-C03** | **Panel de Acordes Dinámico** | Panel lateral que muestra la imagen del diagrama de guitarra del *próximo* acorde 2 segundos antes de que llegue a la línea de actuación. | Media     |
| **RF-C04** | **Feedback Visual**           | Resaltado de la palabra/acorde actual:<br>- **Verde:** Coincidencia detectada.<br>- **Neutro:** Esperando input. | Baja      |
| **RF-C05** | **Navegación Manual**         | Posibilidad de hacer clic en cualquier parte del texto para forzar al sistema a saltar a ese punto (recuperación de errores). | Alta      |

### 3.4 Módulo D: Herramientas de Edición (Beat Mapper)

| ID         | Requisito                | Descripción                                                  | Prioridad |
| :--------- | :----------------------- | :----------------------------------------------------------- | :-------- |
| **RF-D01** | **Grabadora de Tiempos** | Modo "Edición" donde el usuario escucha un metrónomo y pulsa `ESPACIO` para marcar el cambio de línea/acorde. | Media     |
| **RF-D02** | **Persistencia JSON**    | Guardado de los tiempos marcados en un archivo `.json` vinculado al hash del PDF de la canción. | Media     |

---

## 4. Requisitos No Funcionales (RNF)

* **RNF-01 Latencia:** El tiempo de respuesta sistema-usuario (desde que se canta hasta que se mueve el cursor) debe ser inferior a **200ms**.
* **RNF-02 Offline First:** La aplicación no debe requerir conexión a internet para ninguna funcionalidad principal (ni carga de PDF, ni reconocimiento de voz).
* **RNF-03 Rendimiento:** El uso de CPU en un procesador Intel i5 de 8ª gen (o equivalente) no debe superar el **30%**.
* **RNF-04 Portabilidad:** El código debe ser compatible con Windows 10/11, macOS y Linux.

---

## 5. Stack Tecnológico

* **Lenguaje:** Python 3.10+
* **Interfaz Gráfica (GUI):** PyQt6 (Qt for Python).
* **Procesamiento de PDF:** `pdfplumber` (texto/datos), `pdf2image` (renderizado).
* **Reconocimiento de Voz:** Vosk API (Modelo `vosk-model-small-es`).
* **Procesamiento de Audio (DSP):** `NumPy`, `SciPy` (FFT rápida) o `Librosa` (análisis armónico).
* **Entrada de Audio:** `SoundDevice` o `PyAudio`.

---

## 6. Diagrama de Arquitectura (Mermaid)

```mermaid
graph TD
    User((Usuario)) -->|Voz + Guitarra| Mic[Micrófono]
    Mic -->|Stream Audio| AudioEngine{Motor de Audio}
    
    subgraph "Procesamiento (Python Backend)"
        AudioEngine -->|Hilo 1| Vosk[Vosk: Voz a Texto]
        AudioEngine -->|Hilo 2| Chroma[DSP: Detector Acordes]
        
        Vosk -->|Texto Detectado| Logic[Lógica de Fusión]
        Chroma -->|Acorde Detectado| Logic
        
        PDF[PDF Resucitó] -->|pdfplumber| Parser[Parser de PDF]
        Parser -->|Estructura JSON| Logic
    end
    
    subgraph "Interfaz (PyQt6)"
        Logic -->|% Posición & Estado| GUI[Vista Principal]
        Parser -->|Imágenes Recortadas| GUI
        GUI -->|Scroll & Diagramas| User

    end
