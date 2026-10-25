# TomarMedis 💊⭐ (español)

**TomarMedis** viene de «tomar medicinas». En la versión en inglés se llama **TakeMeds**.

Rutina de medicinas para peques con sus personajes favoritos. El peque elige compañero, toca cada medicina cuando se la toma (el personaje le anima con una voz natural) y, al completar la toma, gana un **vídeo oficial** corto (de 2 a 5 minutos) de esa serie.

Pensada para un iPad en la cocina, pero funciona en cualquier navegador moderno. [English version](README.md).

**Pruébala:** https://tomarmedis.com (elige «Continuar sin código»).

**Cómo se hizo:** en una tarde, con Gemini en [Google Antigravity](https://antigravity.google/) y desplegada en Google Cloud Run. Los prompts para repetirlo paso a paso están en [PROMPTS.es.md](PROMPTS.es.md) ([in English](PROMPTS.md)).

![TomarMedis en un iPad](docs/screenshot.png)

## Qué incluye

- **6 mundos**: La Patrulla Canina, SuperThings, Frozen, Toy Story, El Rey León y Wish, cada uno con sus colores, tipografía, fondo animado, sonidos, personajes y frases (español e inglés).
- **Compañeros con voz propia** (Gemini-TTS de Google Cloud) y la voz del dispositivo como alternativa.
- **Cualquier pauta**: de 1 a 10 medicinas (nombre, dosis/tamaño, cómo se toma, color y en qué tomas) y cualquier horario: mañana y noche, una vez al día, cada 12/8/6 horas o personalizado. Las tomas de madrugada cuentan para el día anterior.
- **Vídeos de premio** solo de los canales oficiales de YouTube (lista verificada + búsqueda opcional), de 2 a 5 minutos, con botón de «otro vídeo» y límite por toma configurable.
- **Calendario semanal** con progreso, rachas y la hora de cada toma.
- **Zona de padres** con control para mayores: ajustes, copia de seguridad (también importa las copias de la versión de AI Studio) y enlace para configurar otro dispositivo.
- **Privacidad**: los registros se guardan solo en el dispositivo. El servidor no guarda nada.
- **Instalable** en el iPad («Añadir a pantalla de inicio»).

## Arrancar en local

```bash
npm install
cp .env.example .env   # opcional
npm run dev            # http://localhost:3000
```

## Desplegar en Cloud Run

Necesitas un proyecto de Google Cloud con facturación y la CLI `gcloud`:

```bash
gcloud auth login
PROJECT_ID=tu-proyecto ./deploy/deploy-cloud-run.sh
```

El script activa las APIs, crea una cuenta de servicio, guarda el **código familiar** en Secret Manager (lo genera si no pasas `ACCESS_CODE`), guarda opcionalmente la clave de YouTube (`YOUTUBE_API_KEY=...` o `CREATE_YOUTUBE_KEY=true`) y despliega con Cloud Build. Cada familia usa su propio proyecto y sus propias claves: el repositorio no contiene ninguna.

## En el iPad

1. Abre la URL en Safari e introduce el código familiar una vez.
2. Pulsa **Compartir → Añadir a pantalla de inicio**.
3. En la Zona de padres (rueda dentada) configura nombre, tema, medicinas y horario.

## Personajes, imágenes y vídeos

El proyecto **no incluye material gráfico oficial**. Cada personaje muestra un avatar con emoji y puedes añadir imágenes por dispositivo (URL o foto, por ejemplo de sus juguetes) o por despliegue con `THEME_OVERRIDES_JSON`. Los vídeos se reproducen con el reproductor oficial de YouTube desde los canales de los propietarios.

## Aviso

TomarMedis es una herramienta de recordatorio y motivación, no un producto sanitario. Sigue siempre las indicaciones de tu pediatra.

Proyecto no oficial, sin relación con Disney, Pixar, Spin Master, Nickelodeon ni Magic Box. Todas las marcas, personajes y vídeos pertenecen a sus propietarios.

Licencia: [Apache 2.0](LICENSE).
