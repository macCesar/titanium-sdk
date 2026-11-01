# Android keystore quoted paths — Implementation Plan

> **For agentic workers:** Use `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox syntax for tracking. Este documento prepara el trabajo; no autoriza por sí solo su implementación ni un push.

**Goal:** Aceptar una ruta de keystore con un par exterior de comillas simples o dobles, como la que puede insertar Finder en el prompt de Android.

**Architecture:** Extraer el validador de rutas de keystore a una función pequeña de CLI y mantener el contrato de callback actual. Resolver primero la entrada literal para conservar nombres válidos que contengan comillas; si no corresponde a un archivo y tiene un par exterior coincidente, intentar una sola vez sin ese par. El prompt y la comprobación previa a keytool seguirán usando el mismo validador.

**Tech Stack:** JavaScript ESM, Node >=22.19.0, node-appc, fs-extra, Mocha y Chai existentes.

**Spec:** [Requisitos del proyecto](../../project/requirements.md), apartado de rutas de keystore. La nota de diagnóstico original se eliminó por indicación de César; las decisiones y verificaciones están en `docs/project/`.

## Global Constraints

- Rama: `fix/android-keystore-quoted-path`, creada desde `main` en `9a6e101f58`.
- Conservar espacios, apóstrofes interiores, barras invertidas y la expansión actual de rutas relativas y `~`.
- Retirar como máximo un par exterior coincidente, sin interpretar escapes ni sintaxis de shell.
- Mantener los mensajes actuales para entrada ausente y archivo inválido.
- No cambiar contraseñas, otros prompts, TiNy ni dependencias. Después de implementar, César solicitó compilar e instalar `14.0.0.fixes`; completado para Android e iOS.
- Usar archivos temporales ficticios; no leer keystores reales ni pedir sus contraseñas.
- Validación proporcional: pruebas del validador, regresión directa de CLI Android, lint de archivos tocados y diff; ampliar sólo ante un fallo o riesgo concreto.

## Evidencia comprobada el 28 de octubre de 2026

- `android/cli/commands/_build.js:642–674`: `keystore.validate` llama directamente a `afs.resolvePath`, `existsSync` y `statSync().isFile()`.
- El prompt enlaza ese callback en la línea 658; `store-password.validate` vuelve a invocarlo en la línea 722. No hace falta modificar el manejo de contraseñas.
- Una prueba desechable con el node-appc instalado y un archivo ficticio llamado `Cesar's signing key.keystore` dio: ruta normal existente; rutas entre comillas simples y dobles inexistentes. Confirma la resolución defectuosa, sin probar aún toda la interacción del terminal.
- `android/cli/tests/README.md` sitúa las pruebas unitarias de `android/cli/lib` en `android/cli/tests/test-*.js`. El test actual de AndroidManifest usa Mocha y Chai.

## Review Focus

- Un nombre real con comillas exteriores conserva prioridad frente a su alternativa sin comillas.
- Apóstrofes, espacios y caracteres de shell interiores siguen siendo texto literal.
- Comillas desparejadas o anidadas no se eliminan de forma repetida.
- Archivo inexistente y directorio siguen siendo inválidos, incluso entre comillas.
- Las rutas nativas de Windows conservan barras y semántica; una prueba POSIX no certifica su comportamiento en Windows.

### Task 1: Corregir y verificar el validador compartido

**Files:**
- Create: `android/cli/lib/keystore-path.js`.
- Modify: `android/cli/commands/_build.js` (import y propiedad `keystore.validate`).
- Test: `android/cli/tests/test-keystore-path.js`.

**Interfaces:**
- Produce `validateKeystorePath(keystoreFile, callback)` como export nombrado ESM; entrada string o ausente, callback `(Error | null, resolvedPath?)`, invocado exactamente una vez.
- Consume `appc.fs.resolvePath` y las comprobaciones existentes de fs-extra. La validación comprueba ruta/tipo; keytool continúa comprobando el contenido después.
- `_build.js` asignará esa función a `keystore.validate`; conservará el enlace del prompt y la llamada desde `store-password.validate`.

- [x] **1. Crear pruebas de regresión con archivos temporales.**
  Usar `mkdtemp` y limpieza en `afterEach`; ningún archivo necesita ser un keystore válido. El callback debe devolver la ruta resuelta exacta o el error esperado, una sola vez. Cubrir:
  - Archivo absoluto normal y el mismo entre comillas simples/dobles.
  - Archivo con espacios y apóstrofe interior; conservar los caracteres tras quitar sólo la envoltura.
  - Ruta relativa y `~` con ambas envolturas y sin ellas; crear una fixture temporal bajo el home para el caso `~`, y eliminarla al terminar.
  - Nombre POSIX cuyo primer/último carácter sea una comilla real: crear tanto ese archivo como el nombre sin comillas y exigir que gane el literal. Saltar sólo este caso donde el sistema no permita ese nombre.
  - Comillas desparejadas y dos pares exteriores: no corregir repetidamente una entrada hasta encontrar un archivo.
  - Inexistente y directorio, con y sin envolturas: `Invalid keystore file`.
  - Entrada ausente/cadena vacía: `Please specify the path to your keystore file`; `''` y `""` literales: inválidos.
  - Nombres con `$`, paréntesis y backticks, permitidos por el sistema: tratarlos como caracteres de archivo.
  Las fixtures absolutas usarán `path.join`, de modo que estos mismos casos ejerciten rutas nativas al ejecutarse en Windows.

- [x] **2. Ejecutar la prueba antes de implementar.**
  `./node_modules/.bin/mocha android/cli/tests/test-keystore-path.js`
  Debe fallar por faltar el módulo; registrar ese resultado. Tras extraer el validador sin corregirlo, comprobar que las pruebas de rutas envueltas fallen por archivo inválido.

- [x] **3. Implementar y conectar el validador.**
  Leer los archivos antes de editarlos. Extraer el comportamiento existente y añadir el intento alternativo sin un único par coincidente de comillas simples/dobles. Dar prioridad al archivo literal existente; conservar `isFile()`, mensajes y resolución de node-appc. No añadir `trim`, `eval`, procesos de shell ni conversión de barras. Conectar el export a `keystore.validate`.

- [x] **4. Ejecutar validación acotada (alcance corregido por la instrucción de ejecución).**
  - `./node_modules/.bin/mocha android/cli/tests/test-keystore-path.js` → 20 pasan. No ejecutar AndroidManifest ni suites de dispositivos: el usuario los excluyó expresamente.
  - Oxlint 1.77.0 sobre los tres archivos → cero diagnósticos; se usó el ejecutable existente en caché npm porque faltaba en `node_modules/.bin`, sin instalar dependencias.
  - `git diff --check` → sin errores.
  - Revisar el diff: el prompt y la comprobación previa a keytool deben compartir el validador; contraseñas y demás opciones sin cambios.

- [x] **5. Comprobar la interacción real con una fixture desechable.**
  Verificado con un harness desechable que importó la configuración del SDK modificado y usó `CLI.validate()`, el prompt real en PTY y keytool real, aislando la firma de la detección Android. No se compiló una app en esta aceptación; la prueba del flujo real por César queda pendiente. En el prompt pegar la ruta de una fixture entre comillas simples y dobles; debe superar la comprobación de ruta. Después keytool podrá rechazar el contenido ficticio: distinguir ese error del rechazo inicial. Repetir pasando la ruta con `--keystore`, tanto con comillas consumidas por el shell como con comillas que lleguen literalmente al proceso. No iniciar publicación en Play Store. Registrar SDK, plataforma y límites de la prueba; verificar Windows en un entorno Windows antes de afirmar soporte comprobado allí.

- [x] **6. Revisar y preparar el cambio para entrega.**
  Actualizar la nota de diagnóstico con los resultados y cualquier limitación. Mantener la nota y este plan fuera del diff propuesto para upstream. Cuando la implementación esté autorizada y verificada, preparar el commit de código/pruebas con `fix(android): accept quoted keystore paths`; no incluir cambios de otras ramas. Push y PR se decidirán al entregar el resultado.

## Estado de este plan

Implementación, pruebas y revisión completadas el 2026-10-28. Reproducción: 7 fallos por archivo inválido con el comportamiento original; después del fix, 20 pruebas pasan. Lint de tres archivos sin diagnósticos y diff sin errores. Revisión independiente sin hallazgos.

`14.0.0.fixes` compilado e instalado para Android e iOS por solicitud posterior de César, con código instalado idéntico al fuente y validación de ambas envolturas comprobada. Build, package e install terminaron con código 0. Ver nota para comandos y límites.

Paso 6 completado el 2026-11-01 por indicación de César: commit `3f820557c4`, rama publicada y PR #14629 abierto, vinculado al issue #14628. Después se recompiló e instaló `14.0.0.fixes` desde `main` actualizado más el fix, hash `ed2e30a827`, con las seis novedades de Michael. No hay reporte nuevo del flujo real de César; Windows no se verificó.

Estado actual: `docs/project/status.md`. Este plan y las demás notas no deben incluirse en upstream.
