# 🔍 Resumen de cambios (marzo 2026)

## Sesión 2026-03-15 — Fixes Gemini + RAG básico por keywords

### 1. Fixes críticos Gemini (`server.py`)

**API obsoleta corregida:**
`genai.generate_text()` no existe en `google-generativeai` v0.8.6. Sustituido por:
```python
genai.configure(api_key=effective_key)
gemini_model = genai.GenerativeModel(model)
response = gemini_model.generate_content(prompt_text)
```

**Key de UI ahora funciona:**
`_request_gemini` ignoraba el parámetro `api_key` y usaba solo el global `GEMINI_API_KEY`.
Corregido con `effective_key = api_key or GEMINI_API_KEY`.

**Modelos Gemini actualizados:**
`gemini-1.5-pro` y `gemini-1.5` retirados por Google. Nuevos modelos en dropdown:
- `gemini-2.5-flash` → default (free tier, 20 RPD)
- `gemini-1.5-flash` → alternativa free tier
- `gemini-2.0-flash` → requiere billing (quota 0 en free tier)

### 2. RAG básico: contexto completo + búsqueda por keywords (`server.py`)

**`_build_context_text(context, query="")` rediseñada:**

- **Con vídeo seleccionado:** envía `full_text` completo (~14K chars) en lugar del recorte de 500 chars anterior.
- **Sin vídeo seleccionado y con query:** busca keywords (palabras >3 chars) en los 999 vídeos del índice; envía top-5 por score de relevancia (sum de ocurrencias) con transcripción completa.

Corpus: 999 vídeos · ~18.9M chars · ~4.7M tokens totales · ~14K chars/vídeo media.
Top-5 vídeos × 14K chars ≈ 70K chars / ~17K tokens → dentro del límite de Gemini 2.5 Flash (1M tokens).

**`_query_with_provider()` actualizado:** pasa `query` a `_build_context_text()`.

### 3. Modelos dropdown (`viewer.html`)
Actualizado `LLM_PROVIDER_MAP.gemini.models` con los modelos actuales.

---

## Sesión 2026-03-14 — Multi-proveedor LLM

Se amplía la integración LLM para soportar **Anthropic Claude, OpenAI Chat y Google Gemini**
desde la UI, sin variables de entorno.

### Backend (`server.py`)
- `LLM_PROVIDERS` map con label y modelo por defecto.
- `_query_with_provider()` despachador unificado.
- Helpers: `_request_anthropic`, `_request_openai`, `_request_gemini`.
- `_handle_http_error()` y `_extract_response()` para normalización.
- `configure_gemini()` solicita key interactivamente al arrancar.
- `POST /api/llm-query` acepta `provider`, `model`, `api_key`.

### Frontend (`viewer.html`)
- `LLM_PROVIDER_MAP` con proveedores y modelos.
- Selects de proveedor/modelo + campo password para API key.
- `setupLLMSelectors()` / `updateLLMModelOptions()`.
- `sendLLMQuery()` actualizado para multi-proveedor.

### Documentación
- `README.md`, `LLM_QUERIES.md`, `RESUMEN_CAMBIOS.md`: actualizados.

## Próximos pasos sugeridos
1. Mejorar scoring RAG: TF-IDF o embeddings (ChromaDB) en lugar de conteo simple.
2. Mostrar en UI qué vídeos se han usado como contexto.
3. Sesión de API key en `sessionStorage` para no reintroducirla cada recarga.
4. Limitar tokens de contexto dinámicamente según el modelo seleccionado.
