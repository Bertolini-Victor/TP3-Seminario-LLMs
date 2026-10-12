# Sistema Backend de Triaje y Resolución de Reclamos para E-Commerce

**Trabajo Práctico – Módulo 3: Ingeniería de Prompts e Interacción Avanzada**  
*Especialización / Maestría en Inteligencia Artificial / LLMs*

---

## 📌 Descripción del Proyecto

Este proyecto implementa y evalúa un **motor inteligente de triaje y resolución de reclamos** para una plataforma de comercio electrónico. Su objetivo es procesar mensajes de clientes, clasificar el nivel de urgencia y categoría, razonar internamente bajo políticas corporativas estrictas mediante *Chain-of-Thought* (CoT) y emitir respuestas en un formato **JSON estructurado** listo para su inserción en bases de datos empresariales.

---

## 🏗️ Arquitectura del Pipeline en Dos Etapas

En lugar de utilizar un único mega-prompt que sobrecargue el contexto del modelo, el sistema se divide en **dos sesiones/etapas independientes del LLM**, orquestadas secuencialmente por una función de pipeline en Python:

1. **Etapa 1: Triaje y Clasificación (Prompt 1)**
   - **Técnica:** *Few-Shot In-Context Learning* con 10 ejemplos detallados.
   - **Clasificación:** Asignación de *bucket* (`HIGH`, `LOW`, `AMBIGUOUS`, `DANGEROUS`), nivel de urgencia y categoría del problema (`Shipping/Logistics`, `Billing`, `Defective Product`).
   - **Seguridad:** Detección de intentos de *Prompt Injection* y derivación a revisiones de seguridad.

2. **Etapa 2: Resolución Asistida y Razonamiento Guiado (Prompt 2)**
   - **Técnica:** *Chain-of-Thought* (CoT) estructurado en tres pasos obligatorios:
     1. Identificación del problema real basado en hechos.
     2. Evaluación de políticas de la compañía aplicables (P1 a P13).
     3. Selección de la mejor acción resolutiva y redacción del mensaje al cliente.
   - **Validación Humana:** Determinación automática de si el caso requiere intervención de un operador humano (`needs_human_operator`).

3. **Ensamblado y Validación Estricta en JSON**
   - El pipeline unifica las salidas validadas de ambas etapas en un esquema JSON estándar:
   ```json
   {
     "metadatos_clasificacion": {
       "categoria": "Facturación",
       "nivel_urgencia": "Alta",
       "necesita_operador_humano": true
     },
     "analisis_interno": {
       "cadena_de_pensamiento": "1) Problema real... 2) Política... 3) Mejor acción...",
       "politica_aplicada": "P4 - Duplicate or unauthorized charges"
     },
     "accion_y_respuesta": {
       "codigo_accion_sistema": "ACC_REEMBOLSO",
       "mensaje_al_cliente": "Hola, lamentamos el inconveniente..."
     }
   }
   ```

---

## 📂 Estructura del Repositorio

- `TP3.md`: Enunciado formal y rúbrica de evaluación del Trabajo Práctico 3.
- `Prompt 1.md`: Definición del prompt de triaje, clasificación de urgencia/categoría y ejemplos few-shot.
- `Prompt 2.md`: Definición del prompt de resolución, políticas comerciales (P1-P13), códigos de acción y reglas de tono.
- `.env.example`: Plantilla de variables de entorno para configuración de credenciales del LLM.
- `README.md`: Documentación general del repositorio y de la arquitectura del proyecto.

---

## 🚀 Requisitos y Configuración

### 1. Clonar el repositorio y crear entorno virtual
```bash
python -m venv venv
# En Windows:
venv\Scripts\activate
# En Linux / macOS:
source venv/bin/activate
```

### 2. Instalar dependencias
```bash
pip install -r requirements.txt  # o pip install openai python-dotenv jupyter
```

### 3. Configurar variables de entorno
Copiar el archivo de plantilla y definir la clave de API:
```bash
cp .env.example .env
```
Editar `.env` con tu clave de API correspondiente.
