# Sistema Backend de Triaje y Resolución de Reclamos para E-Commerce

**Trabajo Práctico – Módulo 3: Ingeniería de Prompts e Interacción Avanzada**

---

## 📌 Descripción del Proyecto

Este proyecto implementa y evalúa un **motor inteligente de triaje y resolución de reclamos** para una plataforma de comercio electrónico. Su objetivo es procesar mensajes de clientes, clasificar el nivel de urgencia y categoría, razonar internamente bajo políticas corporativas estrictas mediante *Chain-of-Thought* (CoT) y emitir respuestas en un formato **JSON estructurado** listo para su inserción directa en bases de datos empresariales.

---

## 🏗️ Arquitectura del Pipeline en Dos Etapas Desacopladas

En lugar de utilizar un único mega-prompt que sobrecargue el contexto del modelo y mezcle tareas, el sistema desacopla la ejecución en **dos sesiones independientes del LLM**, orquestadas secuencialmente por la función de Python `procesar_mensaje`:

1. **Etapa 1: Triaje y Clasificación (`Prompt 1.md`)**
   - **Técnica:** *Few-Shot In-Context Learning* con 10 ejemplos representativos.
   - **Clasificación:** Asignación de *bucket* (`HIGH`, `LOW`, `AMBIGUOUS`, `DANGEROUS`), nivel de urgencia (`HIGH`, `LOW`) y categoría del problema (`Shipping/Logistics`, `Billing`, `Defective Product`).
   - **Seguridad:** Aislamiento de datos con delimitadores XML `<reports>` y detección de anomalías o intentos de *Prompt Injection*.

2. **Etapa 2: Resolución Asistida y Razonamiento Guiado (`Prompt 2.md`)**
   - **Técnica:** *Chain-of-Thought* (CoT) estructurado en tres pasos obligatorios:
     1. Identificación del problema real basado estrictamente en hechos.
     2. Evaluación y aplicación de políticas corporativas (P1 a P13).
     3. Selección de la mejor acción del sistema y redacción del mensaje al cliente.
   - **Operador Humano:** Determinación automática de si el caso requiere intervención de un operador humano (`needs_human_operator = True`).

3. **Ensamblado y Validación Estricta en JSON Nativo**
   - El pipeline unifica las salidas validadas de ambas etapas en el esquema JSON estipulado por el TP3 (validado puramente en Python con `json.loads` sin dependencias de Pydantic):
   ```json
   {
     "metadatos_clasificacion": {
       "categoria": "Facturación",
       "nivel_urgencia": "Alta",
       "necesita_operador_humano": true
     },
     "analisis_interno": {
       "cadena_de_pensamiento": "1) Real problem: ... 2) Policy: ... 3) Best action: ...",
       "politica_aplicada": "P4 - Duplicate or unauthorized charges; P11 - Escalations and legal threats"
     },
     "accion_y_respuesta": {
       "codigo_accion_sistema": "ACC_REEMBOLSO",
       "mensaje_al_cliente": "Hola, lamentamos el inconveniente..."
     }
   }
   ```

---

## 📂 Estructura del Repositorio

- `TP3_Resolucion.ipynb`: **Entregable principal**. Jupyter Notebook ejecutable estructurado en 5 secciones con salidas persistidas.
- `TP3.md`: Enunciado formal y rúbrica de evaluación del Trabajo Práctico 3.
- `Prompt 1.md`: Definición del prompt de triaje, reglas de clasificación y 10 ejemplos *few-shot*.
- `Prompt 2.md`: Definición del prompt de resolución, políticas comerciales (P1-P13), códigos de acción y reglas de tono.
- `requirements.txt`: Dependencias del entorno de ejecución (`openai`, `python-dotenv`, `jupyter`, `ipykernel`).
- `.env.example`: Plantilla de variables de entorno para configuración de credenciales del LLM.
- `README.md`: Documentación general del repositorio y de la arquitectura del proyecto.

---

## 🚀 Requisitos y Ejecución

### 1. Clonar el repositorio y preparar entorno virtual
```bash
python -m venv venv
# En Windows:
venv\Scripts\activate
# En Linux / macOS:
source venv/bin/activate
```

### 2. Instalar dependencias
```bash
pip install -r requirements.txt
```

### 3. Configurar variables de entorno
Copiar el archivo de plantilla y definir la clave de API:
```bash
cp .env.example .env
```
Editar `.env` con las credenciales correspondientes (`OPENAI_API_KEY`). Si no se configura una clave, el notebook cuenta con un simulador determinista de respaldo para validar la ejecución local offline sin caídas.

### 4. Ejecutar el Notebook
```bash
jupyter notebook TP3_Resolucion.ipynb
```
El cuaderno contiene las 5 secciones requeridas:
1. Configuración del Entorno y Cliente API.
2. Definición Modular de Prompts (Prompt 1 y Prompt 2).
3. Dataset de Casos de Prueba (6 casos complejos multirriesgo y de seguridad).
4. Función de Pipeline `procesar_mensaje` y Validación JSON.
5. Análisis de Temperatura (0.0 vs 1.0 por triplicado) y Conclusiones Críticas.
