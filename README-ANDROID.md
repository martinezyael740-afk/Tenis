# Matchpoint Analytics → Android (Capacitor)

Este proyecto ya está organizado para convertirse en una app Android real con
[Capacitor](https://capacitorjs.com/). La funcionalidad de `www/index.html`
**no se tocó** al empaquetarla — es exactamente la misma app web, ahora
envuelta en un proyecto nativo.

```
project/
├── package.json              ← dependencias de Capacitor
├── capacitor.config.json     ← configuración (appId, nombre, carpeta www)
├── .gitignore
├── www/                      ← tu app (esto es lo que se copia al APK)
│   ├── index.html            ← Matchpoint Analytics
│   ├── manifest.json         ← metadatos de app instalable
│   ├── sw.js                 ← cache offline
│   └── icons/
│       ├── icon-192.png
│       └── icon-512.png
└── android/                  ← NO existe todavía — Capacitor la genera (paso 3)
```

No pude ejecutar los comandos de Capacitor aquí porque este entorno no
tiene acceso a internet (Capacitor necesita descargar paquetes de npm y
plantillas de Android la primera vez). Tú sí podrás hacerlo en tu propia
computadora en unos minutos. Pasos exactos:

## 0. Requisitos previos (una sola vez)

1. **Node.js** 18 o superior — https://nodejs.org
2. **Android Studio** (incluye el Android SDK) — https://developer.android.com/studio
3. Java JDK 17 (Android Studio ya trae uno propio, normalmente no necesitas instalar otro).

## 1. Instalar las dependencias del proyecto

Abre una terminal **dentro de la carpeta `project/`** (donde está `package.json`) y ejecuta:

```bash
npm install
```

Esto descarga `@capacitor/core`, `@capacitor/android` y `@capacitor/cli`.

## 2. Añadir la plataforma Android

```bash
npx cap add android
```

Esto crea la carpeta `android/` con un proyecto nativo completo (Gradle,
`AndroidManifest.xml`, etc.), usando `capacitor.config.json` para el nombre
de la app, el `appId` (`com.matchpoint.analytics`) y la carpeta `www/`.

## 3. Copiar la app web al proyecto nativo

Cada vez que edites algo dentro de `www/index.html`, vuelve a ejecutar:

```bash
npx cap sync android
```

(`sync` = copia los archivos de `www/` + instala/actualiza plugins nativos.
Si solo cambiaste HTML/CSS/JS sin tocar plugins, basta con `npx cap copy android`).

## 4. Abrir el proyecto en Android Studio

```bash
npx cap open android
```

Esto abre Android Studio con el proyecto ya configurado. Espera a que
termine el "Gradle sync" inicial (puede tardar varios minutos la primera vez).

## 5A. Generar un APK de prueba (debug, sin firmar)

Dentro de Android Studio: **Build → Build Bundle(s) / APK(s) → Build APK(s)**.

Cuando termine, aparecerá un enlace "locate" — el archivo queda en:

```
android/app/build/outputs/apk/debug/app-debug.apk
```

Este APK ya se puede instalar en un teléfono Android para probar
(activa "Orígenes desconocidos" / "Instalar apps desconocidas" en el
teléfono). **No sirve para publicarlo en Google Play** porque no está
firmado con una clave de release.

También puedes generarlo por línea de comandos, sin abrir Android Studio:

```bash
cd android
./gradlew assembleDebug
```

## 5B. Generar un APK firmado (release, para distribuir)

1. En Android Studio: **Build → Generate Signed Bundle / APK…**
2. Elige **APK**.
3. Si es tu primera vez, haz clic en **Create new…** para generar un
   *keystore* (guarda ese archivo `.jks` y las contraseñas en un lugar
   seguro — sin él no podrás actualizar la app más adelante).
4. Elige el *build variant* **release**.
5. Al terminar, el APK firmado queda en:
   ```
   android/app/release/app-release.apk
   ```

## 6. Ícono y nombre de la app

Ya incluí íconos básicos en `www/icons/`. Para reemplazarlos por un ícono
propio con las medidas exactas que exige Android (adaptativo, distintas
densidades), usa el generador de íconos de Android Studio:
**clic derecho en `android/app/src/main/res` → New → Image Asset**, y
sube tu imagen ahí — Android Studio genera todas las resoluciones
automáticamente y actualiza el `AndroidManifest.xml` por ti.

El nombre visible de la app ("Matchpoint Analytics") se controla desde
`capacitor.config.json` (`appName`) — si lo cambias, vuelve a correr
`npx cap sync android`.

## Notas importantes

- La app sigue siendo **100% offline**: no hace ninguna llamada de red (el
  buscador de Wikipedia se quitó a propósito). El `sw.js` solo cachea los
  archivos estáticos para que abra más rápido; no es imprescindible dentro
  de la app nativa (Capacitor ya sirve los archivos locales), pero no
  estorba y ayuda si en algún momento la vuelves a abrir como web.
- El almacenamiento (historial de partidos, apuestas de banca, partido en
  curso) usa `localStorage`, que dentro del WebView de Capacitor persiste
  igual que en un navegador normal — no necesitas ningún plugin adicional
  para eso.
- Si más adelante quieres cambiar el `appId` (`com.matchpoint.analytics`),
  hazlo ANTES de correr `npx cap add android` la primera vez — cambiarlo
  después requiere editar manualmente varios archivos dentro de `android/`.
