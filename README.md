# NEXO AI HQ

Centro privado de operaciones con 14 agentes, conversaciones reales, tareas y aprobaciones, empresas y proyectos aislados y estudio de imágenes.

## Uso

1. Abre Configuración → Añadir API. Introduce nombre, proveedor, modelo y clave.
2. Guarda la conexión cifrada. Para chat, pulsa Probar API: realiza una inferencia real que puede consumir saldo.
3. Asigna la API a uno o varios agentes. Una misma clave puede registrarse con distintos modelos.
4. Conversa con el equipo. El servidor rechaza el modo demo. Los mensajes de simulaciones anteriores se conservan en los datos históricos y se excluyen de la conversación conectada.
5. Para imágenes, añade una API de uso Imágenes y utiliza Estudio de imágenes. Los archivos se guardan en R2 y la galería se separa por usuario, empresa y proyecto.

Proveedores de chat: NVIDIA NIM, Google Gemini, OpenAI, Groq, OpenRouter, Together, Mistral y DeepSeek. NVIDIA Imágenes usa `black-forest-labs/flux.1-schnell` o `black-forest-labs/flux.1-dev`; OpenAI Imágenes admite modelos GPT Image con salida base64. La disponibilidad y las cuotas dependen de cada cuenta. Los hosts están definidos en el servidor; no se permite enviar claves a URLs arbitrarias.

## Estado de validación

- TypeScript, pruebas de dominio, proveedores, cifrado AES-GCM, autorización y contratos de imágenes.
- La prueba real de NVIDIA SD3 y SDXL con la clave proporcionada respondió HTTP 404: modelo/función no disponible para la cuenta. La ficha oficial de SD3 confirma su retiro desde abril de 2026. Se sustituyó por FLUX.1-schnell como modelo predeterminado. Esto no confirma que la clave tenga permisos de generación de imágenes. La aplicación muestra el fallo real; no genera imágenes simuladas.
- La prueba de FLUX.1-schnell terminó por tiempo de espera; la generación de imágenes con la clave suministrada sigue sin verificarse.
- Las claves NVIDIA de chat preexistentes del hosting se conservan. Los agentes sin clave de su proveedor original usan NVIDIA Bolt como conexión predeterminada (u otra conexión de hosting disponible); una asignación explícita tiene prioridad. Las pruebas sin sus valores utilizan mocks; cada conexión puede probarse desde Configuración.
- No se verificó la interfaz en navegador en este entorno: la herramienta `cua_repl.js` requerida por Sites no está disponible.

## Seguridad

Las claves introducidas en Configuración se cifran con AES-GCM y nonce aleatorio, asociadas al usuario y al registro. D1 solo guarda el texto cifrado. `NEXO_VAULT_KEY` permanece en el gestor de secretos del hosting; no debe rotarse sin un procedimiento de recifrado de todas las conexiones. Los listados nunca devuelven claves.

El acceso depende de la pasarela de identidad de Sites. Todas las operaciones de escritura exigen identidad, correo de propietario y Origin del sitio. No desplegar el Worker directamente en un host que permita falsificar las cabeceras `oai-authenticated-*` sin implementar autenticación equivalente.

No subir claves, datos de producción ni archivos `.env` a GitHub. Revocar las claves compartidas en chat y reemplazarlas desde el proveedor. Las variables de servidor preexistentes son conexiones de solo lectura en la interfaz; se reemplazan en el hosting. Las conexiones nuevas pueden eliminarse y añadirse de nuevo para rotar su clave.

## Variables de hosting

- `NEXO_OWNER_EMAIL`: propietario autorizado, obligatorio para escritura.
- `NEXO_VAULT_KEY`: secreto aleatorio de al menos 32 bytes para cifrado, obligatorio para registrar claves.
- `NVIDIA_OMNI_API_KEY`, `NVIDIA_BOLT_API_KEY`: conexiones de chat preexistentes opcionales.
- `GEMINI_API_KEY`, `GEMINI_MODEL`: conexión Gemini predeterminada opcional.
- `NVIDIA_IMAGE_API_KEY`: conexión NVIDIA Imágenes predeterminada opcional.
- `BRAVE_SEARCH_API_KEY`: búsqueda web opcional.
- `CRON_SECRET`, `INFERENCE_PROBE_HOURS`, `HEALTH_SCHEDULE_ACTIVE`: monitorización opcional. El cron requiere activación y verificación en el hosting.

`.openai/hosting.json` conserva el proyecto Sites existente y declara D1 `DB` y R2 `BUCKET`. Las migraciones Drizzle son aditivas; Sites las aplica al publicar.

## Desarrollo

Node >=22.13, pnpm 10.28.2:

```sh
pnpm install --frozen-lockfile
pnpm exec tsc --noEmit
node --experimental-strip-types --test tests/*.test.ts
python tests/persistence.py
pnpm build
```

`pnpm db:generate` genera migraciones tras modificar `db/schema.ts`.

Para publicar con Sites se debe enviar el commit al repositorio fuente asociado y guardar/desplegar esa versión. Copiar este código a GitHub no cambia automáticamente el repositorio asociado al hosting ni activa despliegue continuo desde GitHub. El repositorio GitHub solicitado es `H1karu01/Agentes-de-IA`; la conexión disponible al iniciar este trabajo solo tenía lectura.

## Alcance

El equipo usa inferencia real y genera borradores. Las tareas, cambios de estado, empresas y proyectos se aplican mediante aprobación. Publicaciones, envíos, gastos y campañas requieren ejecución manual: no hay conectores operativos a esas plataformas. No se ejecutan agentes en segundo plano al cerrar la aplicación. Los adjuntos multimedia requieren NVIDIA Omni; la búsqueda requiere Brave.
