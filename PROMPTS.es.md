# Cómo se hizo TomarMedis: los prompts

TomarMedis (TakeMeds en inglés) se hizo en una tarde entre una familia y un agente de programación con IA: **Gemini en [Google Antigravity](https://antigravity.google/)**, desplegado en **Google Cloud**. Aquí están, en orden, los prompts para repetir el mismo proceso desde cero. También puedes partir del código de este repositorio y usar los prompts para adaptarlo a tu familia.

Sustituye los `<MARCADORES>`. Algunos hábitos que funcionaron:

- Trabaja en una carpeta local normal, pide al agente un plan antes de empezar y revisa los cambios que propone.
- Pídele que **compruebe** (typecheck, build, capturas con un navegador automático, `curl` contra la URL publicada) en lugar de fiarte del resumen.
- No pegues claves ni datos de salud reales en los prompts ni en los commits. Los ajustes personales (nombre del peque, medicinas) viven solo en el dispositivo.

---

## 0. Prototipo rápido (opcional, Google AI Studio "Build")

```text
Crea una web divertida para que mi hijo de 4 años se tome sus medicinas por la mañana y por la
noche. Tarjetas grandes y fáciles de tocar para cada medicina (nombre, dosis, dibujo de un vasito
medidor o de una jeringuilla oral), un selector sol/luna, un calendario semanal con estrellas,
un personaje que le anima con voz y un vídeo de premio cuando se toma todas. Interfaz en español,
pensada para iPad.
```

## 1. Convertir el prototipo en un proyecto de verdad

Adjunta el prototipo (zip) y pide:

```text
Esto es un prototipo de una app de rutina de medicinas para niños hecho en AI Studio.
Conviértelo en un proyecto profesional y listo para publicar como código abierto.

Temas: La Patrulla Canina, SuperThings, Frozen, Toy Story, El Rey León y Wish. Cada tema con su
paleta, tipografía, fondo animado, efectos de sonido y de 4 a 8 personajes con frases propias en
español e inglés.

Flujo: el peque elige compañero, toca cada medicina al tomarla (el personaje le anima) y al
completar la toma gana UN vídeo oficial de 2 a 5 minutos (normalmente ~3) de esa serie, con un
botón de "otro vídeo" que prioriza los que aún no ha visto.

Zona de padres con control para mayores: nombre del peque, idioma, tema, de 1 a 10 medicinas
(nombre, dosis/tamaño, cómo se toma, color, en qué tomas), horarios predefinidos (mañana y noche,
una vez al día, cada 12/8/6 horas, horas personalizadas), límites de premios, sonido, copia de
seguridad y enlace para configurar otro dispositivo.

Voces naturales: un endpoint en el servidor con Google Cloud Text-to-Speech (Gemini-TTS) con una
voz y un prompt de estilo por personaje, con caché; si falla, voz del dispositivo.

Stack: React + Vite + TypeScript + Tailwind, un servidor Node/Express pequeño que sirve la app y
las APIs, y un Dockerfile para Cloud Run. Todo configurable con variables de entorno. Nada de
secretos ni datos personales en el repositorio.

Trabaja en una carpeta local nueva. Enséñame primero el plan, después constrúyelo y pasa un
typecheck y una prueba con navegador automático con capturas en iPad horizontal y vertical.
```

## 2. Solo vídeos oficiales

```text
Investiga los canales oficiales de YouTube de cada franquicia (Disney / Disney Junior, Pixar,
La Patrulla Canina, Magic Box / SuperThings) en español de España y en inglés. Verifica cada id
de canal desde su página. Crea una lista de vídeos oficiales que se puedan incrustar, de 120 a 300
segundos, verificados con oEmbed y con la página del vídeo, etiquetados por personaje. No te
inventes ningún id.

Después añade un endpoint que busque SOLO dentro de esos canales con la YouTube Data API
(safeSearch estricto, incrustables, de 90 a 300 s, teniendo en cuenta el país, títulos que sean
claramente del tema, sin tráilers, sin subidas de solo audio, sin "making of"), con caché de 12
horas y la lista verificada como respaldo. En el reproductor, limita la duración máxima y salta
cualquier vídeo que no cargue.
```

## 3. Imágenes de personajes sin publicar material gráfico

```text
No subas ningún material gráfico oficial. Por defecto cada personaje muestra un avatar con emoji.
Añade (1) subir foto o URL por dispositivo desde la zona de padres y (2) una variable
THEME_OVERRIDES_JSON en el servidor con una URL de imagen por personaje, que solo se sirva a los
dispositivos que han metido el código familiar.
```

## 4. Voces que suenen a personaje

```text
Asigna a cada personaje una voz de Gemini-TTS y un prompt de estilo corto (por ejemplo "un
muñeco de nieve gracioso y tierno al que le encantan los abrazos calentitos"). Precarga las
frases que dirá el compañero elegido y los saludos de los demás, guarda el audio en caché en el
servidor y en el dispositivo (Cache Storage) y reprodúcelo con Web Audio para que Safari en el
iPad no lo bloquee.
```

## 5. Desplegar en Google Cloud Run

```text
Escribe deploy/deploy-cloud-run.sh que lea PROJECT_ID y REGION del entorno, active las APIs
necesarias, cree una cuenta de servicio con roles/aiplatform.user (la necesita Gemini-TTS),
guarde en Secret Manager un ACCESS_CODE familiar, una clave opcional de YouTube y los ajustes
de tema, y despliegue con `gcloud run deploy --source` (escala a cero, máximo 2 instancias, URL
pública).

Después despliégalo en el proyecto <PROJECT_ID> en <REGION> y comprueba en la URL publicada:
health check, login con el código familiar, una petición de voz y una búsqueda de vídeos.
```

## 6. Un código familiar que no estorbe

```text
Cuando el servidor tenga código familiar, muestra una pantalla con dos botones del mismo tamaño:
"Entrar con código" y "Continuar sin código". Recuerda la elección en el dispositivo y deja que
los padres metan el código más tarde desde la zona de padres. Cambia el código a <NUEVO_CODIGO>.
```

## 7. Que el peque pueda cambiar de compañero cuando quiera

```text
En la pantalla principal, añade una fila siempre visible con los compañeros del tema (un toque
para cambiar y el nuevo le saluda) y un botón claro de "cambiar compañero" en la cabecera.
```

## 8. Dominio propio

```text
He comprado <DOMINIO> en <REGISTRADOR>. Ayúdame a verificarlo con la cuenta de Google dueña del
proyecto de Cloud Run, conecta <DOMINIO> y www.<DOMINIO> al servicio, dime exactamente qué
registros DNS poner (solo DNS, sin proxy), redirige www al dominio raíz y vigila hasta que se
emitan los certificados.
```

## 9. Nombre, últimos detalles y código abierto

```text
Cambia el nombre de la app a <NOMBRE> (<NOMBRE_EN_INGLÉS> en inglés) en la interfaz, el
manifiesto de la PWA y los README, sin cambiar las claves internas de almacenamiento. Después
prepara el repositorio para GitHub: licencia Apache-2.0, README en inglés y español con capturas,
documentación para arrancar en local y desplegar, aviso de marcas, .gitignore/.gcloudignore que
dejen fuera los secretos, una revisión de claves y datos personales, y un zip para subirlo.
```

---

## Tropiezos que tuvimos (para que tú no los tengas)

- **Gemini-TTS a través de Cloud Text-to-Speech** necesita `aiplatform.endpoints.predict` en la identidad del servicio (`roles/aiplatform.user`); sin ese permiso el servidor usa Chirp 3 HD sin avisar.
- **Safari en iPad**: desbloquea Web Audio con el primer toque, usa `playsinline` en YouTube y "Añadir a pantalla de inicio" para que el navegador no borre los datos.
- **YouTube**: muchos vídeos musicales de Disney en inglés no se reproducen incrustados aunque oEmbed diga que sí; el reproductor tiene que capturar el error y saltar al siguiente.
- **DNS en Cloudflare**: los registros que apuntan a Google tienen que estar en *solo DNS* (nube gris) o el certificado no se emite nunca.
- **Cambiar de dominio**: los datos del dispositivo van por dirección web, así que guarda una copia en la URL antigua y restáurala en la nueva (o usa un enlace de configuración).
- **Firebase Hosting** delante de Cloud Run solo reenvía una cookie llamada `__session`; con el mapeo de dominios de Cloud Run no pasa.
