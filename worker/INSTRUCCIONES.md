# Activar el asistente con IA para apoderados

La página `apoderados.html` ya funciona en **modo demostración**: responde las consultas frecuentes con respuestas aprobadas. Para que también responda preguntas abiertas con IA, hay que poner en línea el intermediario (`worker.js`). Este guarda la clave de la IA y descarta cualquier respuesta que no se apoye en la guía oficial.

```
apoderados.html (GitHub Pages)  →  worker.js (Cloudflare)  →  API de IA (Gemini o Claude)
```

Antes de activarlo con apoderados reales, se necesita la autorización del liceo o de la SIP. El responsable del tratamiento de datos debe ser la institución.

## 1. Crear la clave de la IA (elegir una)

**Gemini (Google AI Studio)**
1. Entrar a aistudio.google.com con una cuenta de Google y crear una clave de API ("Get API key").
2. Activar la facturación del proyecto. En el nivel gratuito, Google puede usar las consultas para mejorar sus productos; en el nivel pagado, no.
3. Fijar un presupuesto o alerta de gasto en la consola de Google Cloud del proyecto.

**Claude (Anthropic)**
1. Entrar a console.anthropic.com, crear una organización y una clave de API.
2. Fijar un límite de gasto mensual en la sección de límites o facturación.

Revisa en las condiciones de cada proveedor el uso de los datos y las restricciones de edad. Por las condiciones de Gemini, el asistente queda dirigido a **apoderados (adultos)**; los estudiantes usan la ruta sin IA.

## 2. Publicar el intermediario en Cloudflare

1. Crear una cuenta gratuita en dash.cloudflare.com.
2. Ir a **Workers & Pages**, luego **Create**, luego **Create Worker**. Ponerle de nombre `asistente-italia` y presionar **Deploy**.
3. Presionar **Edit code**, borrar el código de ejemplo, pegar todo el contenido de `worker/worker.js` y presionar **Deploy**.
4. En el Worker, ir a **Settings**, luego **Variables and Secrets**, y agregar:

| Nombre | Tipo | Valor |
|---|---|---|
| `PROVIDER` | Text | `gemini` o `anthropic` |
| `GEMINI_API_KEY` o `ANTHROPIC_API_KEY` | **Secret** | la clave del paso 1 |
| `ALLOWED_ORIGIN` | Text | `https://raullizama-profe.github.io` |
| `MODEL` (opcional) | Text | por defecto `gemini-3.5-flash-lite` o `claude-haiku-5-5` |

5. Copiar la dirección del Worker. Tiene la forma `https://asistente-italia.<tu-cuenta>.workers.dev`.

Los nombres de los menús de Cloudflare pueden variar un poco.

## 3. Conectar la página

En `apoderados.html`, completar la línea:

```js
const WORKER_URL = "";
```

con la dirección del Worker, y subir el cambio a GitHub (o pedírselo a Claude).

## 4. Probar antes de mostrar

Hacer las consultas de `worker/pruebas.md` y anotar las que fallen. Si una respuesta inventa algo, el intermediario debería reemplazarla por el mensaje "Esa información no está en la guía oficial...".

## Qué controla el intermediario

- **Fuente única:** la IA recibe solo la guía oficial y debe responder en formato JSON.
- **Cita obligatoria:** cada respuesta incluye una frase textual de la guía. Si esa frase no existe en la guía, la respuesta se descarta.
- **Datos verificados:** si la respuesta contiene horas, correos, números de ley o decreto, o porcentajes que no aparecen en la guía, se descarta.
- **Situaciones de riesgo:** acoso, maltrato y similares reciben un mensaje fijo que deriva al liceo, sin pasar por la IA.
- **Límites:** máximo 400 caracteres por consulta y 8 consultas por minuto por conexión. Solo acepta pedidos desde la página del liceo.
- **Privacidad:** no registra el contenido de las consultas. La página bloquea mensajes con RUT, teléfonos o correos.

## Actualizar la guía

Si cambia un dato del liceo, hay que actualizar el texto de la guía dentro de `worker.js` (constante `GUIDE`) y las respuestas fijas en `apoderados.html` (lista `FIXED`), y volver a pegar el Worker en Cloudflare.
