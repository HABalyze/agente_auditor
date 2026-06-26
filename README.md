# Agente Auditor — Auditoría Semántica de Decisiones con IA

Reto de auditoría automática que evalúa las decisiones de un agente de atención al cliente (Agente B) en una aseguradora. El Agente Auditor (Agente A) verifica que cada respuesta respete los controles de negocio definidos, los límites financieros y las alertas críticas.

---

## Estructura

El sistema tiene tres módulos independientes, cada uno con un propósito específico:

**`auditor_llm.ipynb`** — Motor LLM  
Este usa la API de Gemini (`gemini-2.5-flash`) como motor de razonamiento. El modelo lee cada caso completo junto con las reglas de negocio y emite directamente el diagnóstico. Es la versión más directa de ejecutar.

**`auditor.ipynb`** — Motor Semántico  
Este notebook usa `sentence-transformers` (`paraphrase-multilingual-MiniLM-L12-v2`) para convertir los textos en embeddings y comparar semánticamente las respuestas del Agente B contra el contexto RAG. Calcula el Índice de Fidelidad Analítica (IF) y evalúa cada control por tipo y severidad.

**`optimizar_reglas.ipynb`** — Optimizador de Reglas  
Se construyo una herramienta interactiva con las librerias `ipywidgets` y `nltk`. Analiza los casos históricos, sugiere nuevas keywords para los controles y permite al usuario aceptar, editar o rechazar cada sugerencia antes de guardar los cambios en `reglas.json`. Diseñado para mantener el sistema actualizado a nivel de keywords.

---

## Estructura del repositorio

```
agente_auditor/
├── config/
│   └── reglas.json          # Controles de negocio, umbrales y regex
├── data/
│   └── casos.json           # Casos de prueba con contexto RAG y respuestas del Agente B
├── auditor_llm.ipynb        # Motor LLM — ejecutar en Google Colab
├── auditor.ipynb            # Motor semántico — ejecutar localmente
└── optimizar_reglas.ipynb   # Optimizador de reglas — ejecutar localmente
```

---

## Estados de auditoría

El Agente A (Auditor) clasifica cada decisión en uno de tres estados:

| Estado | Significado |
|---|---|
| APROBADO | Agente B actuó correctamente dentro de los parámetros. |
| RECHAZADO | Agente B rompió una regla de negocio o superó un límite. |
| BLOQUEADO | Agente B ignoró una alerta crítica — requiere intervención inmediata. |

---

## Ejecución

### `auditor_llm.ipynb` — Google Colab

El notebook incluye un botón **Open in Colab** el cual permite abrir desde GitHub activandose automaticamente sin una confirguración adicional.

Al ejecutar la celda de importaciones, aparecera un campo para ingresar la `GEMINI_API_KEY` de fomra segura. La key no quedara almacenada en el notebook.

> Este notebook requiere una GEMINI_API_KEY con billing activo en Google AI Studio.

### `auditor.ipynb` y `optimizar_reglas.ipynb` — Local

```bash
# 1. Clonar el repositorio
git clone https://github.com/HABalyze/agente_auditor.git
cd agente_auditor

# 2. Instalar dependencias
pip install sentence-transformers scikit-learn numpy nltk ipywidgets

# 3. Abrir el notebook
jupyter notebook auditor.ipynb
```
> Estos notebooks no requieren API key ni billing. Correran completamente local para ver la interacción entre el
> `optimizar_reglas.ipynb` y las `reglas.json`

---

## Configuración — `reglas.json`

El sistema opera con reglas identificadas y almacenadas. Todos los controles, keywords y umbrales viven en `config/reglas.json` y las keywords podrán actualizarse sin tocar el json a traves del notebook `optimizar_reglas.ipynb`.

Los tres controles activos en la versión actual:

| Control | Tipo | Severidad |
|---|---|---|
| `control_bloqueo_urgente` | Detección de urgencia | CRÍTICA |
| `control_limite_numerico` | Comparación numérica | ALTA |
| `control_derivacion` | Detección de derivación | ALTA |

---

## Salida esperada

```
────────────────────────────────────────────────────────────
Caso [X]: APROBADO
Índice de Fidelidad Analítica: 0.95 (CONFORME)
Diagnóstico/Razón:
  Agente B aprobó/rechazo/ignoro...
```
