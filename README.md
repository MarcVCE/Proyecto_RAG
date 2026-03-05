# 🚀 RAG Completo con Groq — Gratis y Ultra-Rápido

Sistema de **Retrieval-Augmented Generation (RAG)** construido sobre LangChain + Groq, con soporte multilingüe, ingestión de PDFs y webs, y doble vector store (FAISS en memoria + Chroma persistente).

---

## 📋 Tabla de contenidos

- [¿Qué es RAG?](#qué-es-rag)
- [Arquitectura](#arquitectura)
- [Tecnologías](#tecnologías)
- [Requisitos](#requisitos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Próximos pasos](#próximos-pasos)

---

## ¿Qué es RAG?

RAG (Retrieval-Augmented Generation) es una técnica que permite a un LLM responder preguntas basándose **únicamente en tus propios documentos**, en lugar de depender solo de su conocimiento preentrenado.

```
Tus docs → Chunks → Vectores → Vector DB
                                   ↓
              Pregunta → Recuperar chunks relevantes → LLM → Respuesta
```

---

## Arquitectura

El pipeline sigue 6 pasos:

| Paso | Descripción |
|------|-------------|
| 1️⃣ **Carga** | Lee documentos desde URLs web y PDFs locales |
| 2️⃣ **Limpieza** | Normaliza saltos de línea, elimina HTML residual y artefactos de PDF |
| 3️⃣ **División** | Fragmenta en chunks de 1000 chars con solapamiento de 200 |
| 4️⃣ **Embeddings** | Convierte texto a vectores con un modelo multilingüe de HuggingFace |
| 5️⃣ **Vector Store** | Indexa en FAISS (en memoria) y Chroma (persistente en disco) |
| 6️⃣ **Generación** | Recupera los 4 chunks más relevantes y genera respuesta con Llama 3.3 70B vía Groq |

---

## Tecnologías

| Librería | Rol |
|----------|-----|
| [LangChain](https://python.langchain.com/) | Orquestación del pipeline RAG |
| [Groq](https://console.groq.com/) | Inferencia ultra-rápida del LLM (Llama 3.3 70B) — **gratuito** |
| [FAISS](https://github.com/facebookresearch/faiss) | Vector store en memoria, ideal para prototipos |
| [Chroma](https://www.trychroma.com/) | Vector store persistente en disco |
| [HuggingFace `sentence-transformers`](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2) | Embeddings multilingües (ES + EN en el mismo espacio vectorial) |
| [PyMuPDF](https://pymupdf.readthedocs.io/) | Ingestión y limpieza de PDFs |
| [BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/) | Carga y parseo de páginas web |

---

## Requisitos

- Python 3.10+
- Una API key gratuita de Groq → [console.groq.com/keys](https://console.groq.com/keys)
- *(Opcional)* API key de LangSmith para evaluación → [smith.langchain.com](https://smith.langchain.com)

---

## Instalación

```bash
pip install langchain-classic langchain-groq langchain-chroma faiss-cpu \
            python-dotenv langchain-huggingface beautifulsoup4 \
            sentence-transformers chromadb pinecone-client cohere pymupdf
```

---

## Configuración

Crea un fichero `.env` en la raíz del proyecto:

```env
# Obligatorio
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxx

# Opcional — para evaluación con LangSmith
LANGSMITH_API_KEY=lsv2_xxxxxxxxxxxxxxxxxxxx
LANGSMITH_PROJECT=mi-rag-groq
```

Además, el notebook espera dos ficheros en la misma carpeta:

| Fichero | Descripción |
|---------|-------------|
| `urls.json` | JSON con clave `"news"` y array de URLs a indexar |
| `organic.pdf` | PDF de ejemplo para ingestión (sustituye por el tuyo) |

Formato de `urls.json`:

```json
{
  "news": [
    "https://ejemplo.com/articulo-1",
    "https://ejemplo.com/articulo-2"
  ]
}
```

---

## Uso

Abre y ejecuta el notebook celda a celda:

```bash
jupyter notebook rag_groq.ipynb
```

### Hacer una consulta

```python
consulta = "¿Cuáles son los requisitos de etiquetado para productos orgánicos?"
rag_test(qa_chain_faiss, consulta)
```

El sistema devuelve la respuesta y las fuentes (con preview del chunk) que la respaldan.

### Cambiar el vector store

```python
# FAISS — rápido, en memoria
rag_test(qa_chain_faiss, consulta)

# Chroma — persistente en disco, ideal para producción
rag_test(qa_chain_chroma, consulta)
```

### Guardar y recargar el índice FAISS

```python
# Guardar
vectorstore_faiss.save_local("mi_rag_index")

# Recargar (sin re-embeber documentos)
vectorstore_reloaded = FAISS.load_local(
    "mi_rag_index", embeddings, allow_dangerous_deserialization=True
)
```

> ⚠️ Chroma ya persiste automáticamente en `./chroma_db`. No es necesario guardar/recargar explícitamente.

---

## Estructura del proyecto

```
.
├── rag_groq.ipynb       # Notebook principal
├── urls.json            # URLs a indexar
├── organic.pdf          # PDF de ejemplo
├── .env                 # API keys (no subir a Git)
├── mi_rag_index/        # Índice FAISS persistido
│   ├── index.faiss
│   └── index.pkl
└── chroma_db/           # Base de datos Chroma
```

> 💡 Añade `chroma_db/`, `mi_rag_index/` y `.env` a tu `.gitignore`.

---

## Próximos pasos

| Mejora | Cómo | Beneficio | Estado |
|--------|------|-----------|--------|
| **Streaming** | `qa_chain.stream()` | Respuestas token a token en tiempo real | ⬜ Pendiente |
| **Reranking** | `CohereRerank` | Reordena chunks recuperados → +30% precisión | ⬜ Pendiente |
| **Cloud DB** | Chroma / Pinecone | Persistencia escalable y multi-usuario | ✅ Chroma local |
| **PDFs** | `PyMuPDFLoader` | Ingesta documentos empresariales | ✅ Implementado |
| **Multi-idioma** | `paraphrase-multilingual-MiniLM-L12-v2` | Embeddings nativos en ES y EN | ✅ Implementado |
| **Evaluación** | LangSmith + `evaluate()` | Mide calidad con datasets de Q&A | ⬜ Pendiente |

---

## Licencia

MIT — libre para uso personal y comercial.
