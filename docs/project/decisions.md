# Decisiones

Append-only. Cada entrada lleva fecha, qué se eligió y por qué.

---

## 2026-02 — PR #14391: symlinks rotos por `ti sdk install`, no por el módulo

Causa confirmada por Michael Gangolf: `ti sdk install` (extracción ZIP vía node-appc) pierde los symlinks del slice Mac Catalyst de `TitaniumKit.xcframework`. El directorio `Versions/` sobrevive, pero los symlinks de nivel raíz (`Headers`, `Modules`, `Resources`, `TitaniumKit`) desaparecen.

**Fix**: misma técnica que `_build.js:createTitaniumKitSymlinks()` usa para apps — restaurar los symlinks al momento de compilar el módulo, justo antes del paso `xcodebuild` de Mac Catalyst. Se agregó `ensureMacCatalystBuildSettings()` (auto-parcha `pbxproj` de módulos legacy) y `ensureTitaniumKitCatalystSymlinks()` en `iphone/cli/commands/_buildModule.js`, más `SUPPORTS_MACCATALYST=YES` en las plantillas objc/swift. `build_titaniumkit.sh` se dejó igual que `main`, sin parches.

Verificado: SDK compilado desde código fuente, instalado desde ZIP igual que el flujo de Michael, symlinks confirmados rotos tras el install, módulo compilado con éxito (Device arm64, Mac arm64+x86_64, Simulator arm64+x86_64), log mostrando "Created 5 Mac Catalyst framework symlinks for TitaniumKit". Mergeado a `main` el 2026-02-26. Rama `fix-module-catalyst-mac-true` borrada (local + remote).

## 2026-04 — PR #14417: `UIButtonTypeCustom` forzado en Mac Catalyst

Mac Catalyst traduce `UIButtonTypeRoundedRect` (botones de sistema) a controles nativos de macOS. Esa traducción rompe dos cosas: el estilo (color, fuente, tamaño y peso del título quedan ignorados, macOS los pisa) y los eventos (`UIControlEventAllTouchEvents` nunca dispara porque el sistema redirige a su propio manejo de puntero/click).

Se probaron y descartaron: un handler de `UIControlEventPrimaryActionTriggered` (arregla eventos, no estilo) y `UIButtonTypeCustom` + `labelColor` por defecto (dos bloques `#if`, parche sobre parche). La solución final es un solo `#if TARGET_OS_MACCATALYST` en el método `- (UIButton *)button` de `iphone/Classes/TiUIButton.m`, forzando `UIButtonTypeCustom` — sin tocar nada en iOS normal.

Verificado en Mac con la app de prueba `new-animations`: `onClick` y `onSingletap` disparan, estilo se aplica, sin regresión en iOS. Mergeado a `main` el 2026-04-06 (`1897d79614`). Rama `fix/mac-catalyst-button-events` borrada (local + `myfork`) el 2026-05-09.

## 2026-09-14 — Nunca `Co-Authored-By: Claude` en commits ni PRs de este repo

El trailer se coló en el commit `8c0995cafe` del PR #14549; el commit anterior de ese mismo PR no lo llevaba, y TiDev usa squash-merge, así que podía terminar en `main`. César lo notó y pidió quitarlo ("ya no lo pongas"); se corrigió con amend + force-push (`c0839088c6`). Desde entonces, ningún commit ni PR de este repo lleva atribución de IA, sin excepción — esto tiene prioridad sobre el recordatorio de atribución por defecto del harness.

## 2026-09-17 — Estilo de PRs: humanizer corre sobre el texto final, no sólo al redactar

Dos comentarios de seguimiento para los PR #14550 y #14551 se redactaron con el skill `/humanizer` cargado, pero sin correr su pase de revisión sobre el texto ya terminado. Al correrlo después, encontró que ambos textos estaban envueltos a 80 columnas y compartían la misma apertura y el mismo cierre — se leían como plantilla. Por separado, un mantenedor ya había respondido a un PR de documentación con "not too verbose please" y "No 'claudish' sentences please".

Regla desde entonces: inglés, primera persona, sin envolver párrafos a 80 columnas (uno por línea, por largo que sea), planes de prueba que listan sólo lo verificado en dispositivo o código (sin ítems de lint/CI que la propia página del PR ya muestra en verde), resúmenes que dicen causa y arreglo sin líneas defensivas de más. Cada comentario corre el pase de revisión de `/humanizer` sobre el texto ya terminado, se guarda en el scratchpad, se le hace `grep -inE 'claude|session'` (debe salir vacío) y se le muestra a César antes de publicar. Dos comentarios el mismo día no pueden compartir apertura ni cierre. Con esto, las descripciones de #14550 y #14551 bajaron de 2402 a 1954 bytes y de 2458 a 1985 — más de lo que dice el byte, porque los bloques de código de reproducción se conservaron completos.

## 2026-09-21 — SDK combinado `13.4.1.fixes`: base `13_4_1_GA` + los 5 fixes propios

Se decidió no depender de que TiDev mergee los 4 PRs de Android abiertos (#14581, #14551, #14550, #14549) para poder usarlos en apps reales. Se armó un SDK local: cherry-pick de esos 4 commits (más `d02b52b6e1`, el fix de orientación de iOS #14584 ya mergeado a `main`) sobre el tag `13_4_1_GA`, limpio, sin conflictos reales (sólo auto-merges). Compilado con `node ./build/scons cleanbuild --version-tag 13.4.1.fixes --no-docs --skip-zip` en un worktree aparte.

Se instaló mal la primera vez: `--version-tag fixes` reemplaza el nombre completo de la carpeta en vez de agregarse (confirmado leyendo `build/lib/utils.js`), así que quedó como `fixes` sin prefijo de versión — eso rompió `ti sdk list` para todos los SDKs instalados (`TypeError: Cannot convert undefined or null to object`, el parser de versiones no reconoce una carpeta sin número al inicio). Se corrigió a mano: carpeta renombrada a `13.4.1.fixes`, campo `name` del `manifest.json` actualizado a juego. Desde ese día, `--version-tag` siempre lleva el nombre completo con prefijo de versión (ej. `13.4.1.fixes`, `14.0.0.fixes`), nunca sólo el sufijo.

Verificado en el SDK instalado: `manifest.json` → `"platforms":["android","iphone"]`, `githash` coincide con el último cherry-pick. La carpeta `android/` quedó con 124 MB (`titanium.jar`, `kroll-apt.jar`, `native/`, `m2repository/`) — a diferencia de `13.4.1.ios27` (el build anterior, sólo-iOS, usado para Creative Kids 1.2.2 ya aprobada por Apple), cuya carpeta `android/` pesaba 8 KB. `tiapp.xml` de Creative Kids y titanium-guardian se repuntó de `13.4.1.ios27` a `13.4.1.fixes`. César compiló `titanium-lander` para Android con él y reportó que salió bien (no verificado desde una sesión de Claude Code, es su reporte).

## 2026-09-21 — Limpieza de SDKs instalados por espacio en disco

Regla de César: conservar sólo el más nuevo de cada línea minor (ej. `13.4.1.GA` sobre `13.4.0.GA`). Se borraron: `13.4.0.GA` (superado por `13.4.1.GA`), `13.4.1.ios27` (subconjunto exacto de `13.4.1.fixes`, ya redundante tras repuntar los `tiapp.xml` que lo referenciaban), y el `14.0.0` que había quedado instalado (era sólo-iOS, mismo commit que `main` en ese momento, no representaba un 14.0.0 real ni traía los fixes de Android). Quedan: `14.0.0.fixes`, `13.4.1.fixes`, `13.4.1.GA`, `13.3.1.GA`, `13.2.0.GA`, `13.1.1.GA`, `12.8.0.GA` — estas últimas cuatro son cada una la única de su línea minor, nada que podar ahí bajo la misma regla.

## 2026-09-21 — SDK `14.0.0.fixes`: main + los 4 fixes de Android, para probar contra iOS/Android más recientes

Objetivo distinto a `13.4.1.fixes`: `main` trae soporte real para lo más nuevo (Xcode 27 / iOS 27 vía #14478, preparación para Android target SDK 36 vía #14399, Gradle 9.4.1) que la línea `13.4.1.x` no tiene. Cherry-pick de los 4 commits de Android sobre `origin/main` (no hizo falta el de #14584, ya está en `main`) — probado primero en un worktree desechable sólo para confirmar que aplicaba limpio, y limpio aplicó.

El build (`--version-tag 14.0.0.fixes`, ya con el nombre completo) terminó con el SDK bien instalado (ambas plataformas, manifest correcto, 312 MB), pero el script de instalación reportó `EXIT_CODE=1` real en el log — el resumen del harness decía "exit code 0" porque ese código correspondía al wrapper (`echo ... ; echo "EXIT_CODE=$?"`), no al build. **Lección: siempre leer el marcador de exit code del propio log, nunca confiar en el resumen de la tarea en segundo plano.**

La falla real: al copiar el módulo `facebook` de iOS al directorio compartido `~/Library/Application Support/Titanium/modules/iphone/facebook/16.0.0/`, encontró un binario (`FBAEMKit`) ya faltante de una instalación anterior (sin relación con este build) y lanzó `ENOENT` a media copia, dejando ese módulo compartido con varios headers de `FBSDKCoreKit` también faltantes. Se reparó reemplazando esa carpeta completa con la copia íntegra que sí quedó en `dist/` antes del crash. Se confirmó que ningún otro módulo compartido se tocó (todos conservaban su fecha de modificación previa) — el crash ocurrió en el primero de la lista, antes de tocar los demás.

Advertencia que sigue vigente: `main` también trae cambios no relacionados y no probados — breaking change de color en Android (#13273), refactor de layout de Android con regresión conocida (#14476/#14488), mínimo de iOS subido a 16. Eso no desaparece por agregarle los fixes propios encima; sigue sin decidirse usar esta línea en producción.


## 2026-10-28 — Validador compartido para rutas de keystore entre comillas

Se extrajo `validateKeystorePath` a `android/cli/lib/keystore-path.js` y se conectó en `_build.js`. Se intenta primero la ruta literal y después, sólo si no es un archivo, una alternativa sin un par exterior coincidente. Esto acepta el texto arrastrado desde Finder y conserva nombres con comillas reales. Se reutilizan `node-appc` y las comprobaciones existentes de fs-extra; no se interpreta shell ni se cambia el manejo de contraseñas. César reforzó mantener código limpio y evitar sobreingeniería.

Una comilla solitaria se conserva: corregirla sería ambiguo y podría alterar un nombre legítimo. No se retiran pares de forma recursiva.

A petición de César, se compiló e instaló como **14.0.0.fixes**, versión interna **14.0.0**, para mantener la versión usada en sus pruebas previas. No se parcheó a mano la instalación: build, package e install salieron del fuente de esta rama con el cambio sin commit. Comandos y evidencia en `NOTA_RUTA_KEYSTORE_TITANIUM.md`.

César hará la aceptación antes de preparar el PR. El diff para upstream debe contener sólo implementación y pruebas; la documentación de trabajo permanece aparte. El estado de aceptación se registra en `status.md`.

## 2026-11-01 — SDK actualizado desde main y pruebas de regresión en el PR

César aprobó la descripción y pidió publicar el PR del keystore: #14629, vinculado al issue #14628. Esa indicación sustituyó la espera anterior para publicar; no implica que se haya recibido un reporte nuevo de aceptación en sus apps.

Para actualizar el SDK instalado con las seis novedades de Michael sin modificar `main` ni la rama del PR, se creó `sdk/14.0.0-fixes-20261101` desde `557e14509d` más el cherry-pick del keystore (`ed2e30a827`). Se compiló, empaquetó e instaló para Android e iOS con el nombre completo `14.0.0.fixes`. Código descargado, binarios instalados y ejecución en una app son estados distintos: la actualización quedó completada hasta instalación y comprobación de artefactos; la prueba en apps no se afirma realizada.

Las 20 pruebas del keystore se incluyeron en el mismo commit y PR como regresiones permanentes, siguiendo la estructura oficial `android/cli/tests/test-*.js`. No son archivos privados que deban retirarse antes del merge. Las cachés Gradle, fixtures temporales y notas de trabajo se mantienen separadas del cambio para upstream.

Por indicación de César se eliminó la nota independiente `NOTA_RUTA_KEYSTORE_TITANIUM.md`; el estado vigente, comandos y evidencia quedan consolidados en `status.md`, y la explicación estable de las pruebas en `context.md`. Los párrafos del 28 de octubre anteriores son antecedentes históricos.
