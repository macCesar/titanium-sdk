# Estado

**Actualizado:** 2026-11-01
**Sesión por:** Codex · GPT-6
**Fase:** PR del keystore publicado y SDK `14.0.0.fixes` actualizado para Android e iOS.

## Repositorio y contribución

- Base de código: `main`, `557e14509d`, sincronizada con el `origin/main` descargado hoy. Para guardar la documentación por indicación de César, la copia de trabajo quedó en `docs/local-session-20261101`, publicada en `myfork` por indicación posterior de César; no mezclar esta rama en PRs de código.
- Fix original: `3f820557c4`, rama `fix/android-keystore-quoted-path`, publicada en `myfork`.
- PR https://github.com/tidev/titanium-sdk/pull/14629, contra `main`, enlazado al issue https://github.com/tidev/titanium-sdk/issues/14628. Al crearlo estaba abierto y no draft; no se ha comprobado el resultado posterior de CI ni su integración.
- El PR contiene sólo `_build.js`, el nuevo validador `android/cli/lib/keystore-path.js` y `android/cli/tests/test-keystore-path.js`: un commit, tres archivos. Destinado a la línea 14.0.0; no se realizó backport a 13.x.
- Rama local reproducible de compilación: `sdk/14.0.0-fixes-20261101`, basada en `main` más cherry-pick del keystore, HEAD `ed2e30a827`. No se publicó esta rama combinada ni se añadió el fix a `main`.

## SDK instalado

`~/Library/Application Support/Titanium/mobilesdk/osx/14.0.0.fixes`

Manifiesto comprobado: nombre `14.0.0.fixes`, versión interna `14.0.0`, hash `ed2e30a827`, plataformas Android e iOS, timestamp `11/1/2026 20:16`.

Incluye el fix del keystore y las seis novedades de Michael. Se confirmó que estos commits son ancestros de la rama compilada:

| Cambio | Commit |
| --- | --- |
| ListView.itemlongpress | `32df010365` |
| ios.deviceId predeterminado | `3a08b636ca` |
| Animación de rotation | `d749302ea6` |
| Snapping de ListView/TableView | `9092c4a5e6` |
| Padding inicial de ScrollableView Android | `086200b1db` |
| galleryFolder de cámara Android | `0c2285a6c3` |

Se ejecutó desde el fuente combinado:

```sh
node build/scons build --sdk-version 14.0.0
node build/scons package --sdk-version 14.0.0 --version-tag 14.0.0.fixes --skip-zip --no-docs
node build/scons install --sdk-version 14.0.0 --version-tag 14.0.0.fixes
```

Los tres pasos terminaron con código 0, confirmado dentro de cada log. Android: `BUILD SUCCESSFUL`. iOS: `ARCHIVE SUCCEEDED`, Xcode 27.2. Se reemplazó la instalación anterior, sin cambiar la selección global del SDK.

Logs: `/tmp/titanium-14.0.0-fixes-20261101-{build,package,install}.log`. Copia previa: `/var/folders/dx/7wnhkmkn79l22c68s7rk76600000gn/T/titanium-fixes-before-20261101-qctkkcie/14.0.0.fixes`; su ruta está también en `/tmp/titanium-14.0.0-fixes-20261101-backup.txt`. Estos archivos temporales pueden desaparecer.

## Pruebas realizadas y alcance

- 20 pruebas del keystore pasan en la rama del fix y en la rama combinada. Comando específico: `./node_modules/.bin/mocha android/cli/tests/test-keystore-path.js`. El archivo está en ambas ramas, pero no en el `main` descargado; hay que seleccionar una de ellas para ejecutarlo desde el fuente.
- Oxlint 1.77.0 sin diagnósticos en los tres archivos del fix; `git diff --check` limpio.
- Antes del fix, siete casos fallaban por rutas envueltas. El 28 de octubre se verificaron el prompt real y `--keystore` con fixtures hasta la invocación de keytool; fue una prueba aislada de firma, no un build completo de una app.
- Después de instalar hoy: manifiesto correcto; CLI de Android/iOS idéntico al fuente; 32 artefactos nativos idénticos al paquete y 7 binarios TitaniumKit idénticos al build. Validador instalado probado con fixture desechable: ruta normal, comillas simples y dobles aceptadas, callback único.
- No se ejecutaron las suites completas ni pruebas de interfaz en dispositivos para las seis novedades. No se verificó Windows. No hay reporte nuevo de aceptación de César en su flujo real.

## Documentación y archivos locales

Las suites bajo `tests/`, `cli/tests/` y `android/cli/tests/` ya forman parte del repositorio oficial. Nuestro archivo con las 20 pruebas se propone como prueba de regresión permanente dentro del PR, no como un script privado desechable. El comando oficial `test:cli` incluye el patrón `**/cli/tests/test-*.js` que lo recoge cuando está presente.

Son documentación de trabajo local las notas en `docs/project/`, el plan bajo `docs/superpowers/` y el puntero de `AGENTS.md`. César pidió guardarlos en Git: se incluyen explícitamente en `docs/local-session-20261101`, publicada en `myfork` por indicación posterior de César, sin cambios al PR. La regla de `.git/info/exclude` para `docs/project/` se conserva para los archivos nuevos sin seguimiento. Logs, fixtures desechables y cachés `.gradle/` quedan sin versionar; la caché `tests/modules-source/ti.modulesdk1341/android/.gradle/` se añadió a las exclusiones locales. `NOTA_RUTA_KEYSTORE_TITANIUM.md` se eliminó por indicación de César.

## Limpieza de ramas completada

César autorizó eliminar las ramas de PRs ya integrados. Se eliminaron tanto locales como de `myfork`:

- `fix/android-night-mode-unchanged-recreate`: PR #14549 integrado el 2026-10-28.
- `fix/android-tabgroup-stale-activity`: PR #14550 integrado el 2026-10-28.
- `fix/android-exit-on-close-after-recreate`: PR #14581 integrado el 2026-10-02. Antes de eliminarla se conservó su commit adicional de `CLAUDE.md` mediante cherry-pick `9070e5babc` en la rama de documentación, comprobado idéntico y subido al fork.

Verificados en GitHub: PR #14551 de ActionBar y PR #14629 del keystore siguen abiertos. Se conservaron sus ramas, `main`, la rama del SDK instalado, las combinaciones antiguas y las ramas de pruebas. Estas últimas requieren revisión aparte antes de eliminarlas.

## Próximo paso

Comprobar la revisión y CI del PR #14629 cuando se retome su seguimiento. Usar el SDK instalado en las apps de César y registrar resultados concretos del keystore y de las novedades que se prueben. No repetir builds ni ampliar suites sólo por retomar la sesión; validar según el comportamiento afectado.

## Antecedentes no retomados

Las notas de septiembre registran SDKs combinados anteriores, recommended settings de Xcode y un borrador de close-on-recreate. No se revisaron en esta sesión ni son bloqueos de este cierre. César informó que sus PRs anteriores fueron aprobados; el detalle histórico está en `decisions.md` y no sustituye comprobar el estado actual en GitHub.
