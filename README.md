# Sistema de automatización de videos cortos — Facebook y YouTube

Workflow de **n8n** que crea y publica, sin intervención manual, videos cortos verticales (Shorts/Reels) con datos curiosos. A partir de una idea generada con IA arma el guion, la narración, las imágenes y el video final, y lo sube a **YouTube** y a una **página de Facebook**.

## Cómo funciona

```
Schedule Trigger
   └─► OpenAI (gpt-4o-mini) ─ genera título, tema, guion, descripción y hashtags en JSON
        └─► Code ─ parsea la respuesta y crea un jobId y las rutas de trabajo
             └─► HTTP Request ─ OpenAI TTS: convierte el guion en audio (.mp3)
                  └─► Code ─ guarda el audio en disco
                       └─► HTTP Request ─ Pexels: busca imágenes según el tema
                            └─► Code ─ prepara las URLs de las imágenes
                                 └─► HTTP Request / Code ─ descarga las imágenes
                                      └─► Code ─ FFmpeg: une imágenes + audio en un video vertical
                                           └─► Read File ─ lee el video generado
                                                └─► YouTube ─ sube el video como #Shorts
                                                     └─► Code ─ publica el video en la página de Facebook (Graph API)
```

1. **Idea y guion**: GPT-4o-mini propone un dato curioso poco conocido (ciencia, historia, naturaleza, psicología, tecnología, astronomía, biología o economía) y devuelve un guion de máximo 8 líneas cortas.
2. **Narración**: el guion se convierte en voz con la API de texto a voz de OpenAI.
3. **Imágenes**: se buscan y descargan imágenes relacionadas desde Pexels.
4. **Montaje**: FFmpeg combina las imágenes y el audio en un video.
5. **Publicación**: el video se sube a YouTube (público, con título, descripción y hashtags) y a la página de Facebook.

## Requisitos

- **n8n** autoalojado (los nodos Code usan `fs`, `https` y `child_process`)
- **FFmpeg** instalado y disponible en el `PATH`
- Cuentas y claves de:
  - **OpenAI** (generación de texto y TTS)
  - **Pexels** (API de imágenes)
  - **YouTube** (credencial OAuth2 en n8n)
  - **Facebook**: página y *Page Access Token* con permiso para publicar videos

Para que los nodos Code puedan usar módulos de Node.js y ejecutar FFmpeg, n8n debe iniciarse con estas variables de entorno:

```
N8N_ALLOW_EXEC=true
NODE_FUNCTION_ALLOW_BUILTIN=fs,path,child_process,https,http
```

En Windows basta con ejecutar **`iniciar-n8n.bat`** (incluido en este repositorio), que define las variables y arranca n8n:

```bat
set N8N_ALLOW_EXEC=true && set NODE_FUNCTION_ALLOW_BUILTIN=fs,path,child_process,https,http && n8n start
```

Después abre `http://localhost:5678`.

## Instalación

1. En n8n: **Workflows → Import from File** y selecciona `workflow.json`.
2. Crea o asigna las credenciales en cada nodo:
   - `OpenAi account` → nodos **Message a model** y **HTTP Request** (TTS)
   - `Header Auth account 2` → **HTTP Request1** (Pexels, cabecera `Authorization: <tu clave>`)
   - `YouTube account` → **Upload a video**
3. Sustituye los marcadores en los nodos Code:

   | Marcador | Nodo | Valor |
   |---|---|---|
   | `TU_OPENAI_API_KEY` | Code in JavaScript2 | Clave de API de OpenAI |
   | `TU_FACEBOOK_PAGE_ID` | Code in JavaScript5 | ID de tu página de Facebook |
   | `TU_FACEBOOK_PAGE_ACCESS_TOKEN` | Code in JavaScript5 | Token de acceso de la página |

4. Ajusta las rutas de trabajo si es necesario. Por defecto son `C:\Users\Public\n8n-videos\images` y `C:\Users\Public\n8n-videos\audio`.
5. Configura el intervalo del **Schedule Trigger** y activa el workflow.

> ⚠️ No subas tus claves reales al repositorio. Este `workflow.json` se publica con marcadores en lugar de los valores secretos.

## Autor

**Santiago Bautista Vizcaíno**
