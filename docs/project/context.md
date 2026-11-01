# Context

Fork de contribución (`macCesar`) a [`tidev/titanium-sdk`](https://github.com/tidev/titanium-sdk), no un producto propio de César. El trabajo son fixes puntuales enviados como PRs, más SDKs locales combinados para usar esos fixes en apps propias antes de que TiDev los mergee.

## Mapa de documentación

- **`README.md`** — presentación del proyecto, para quien llega de fuera.
- **`CONTRIBUTING.md`** — cómo contribuir: reportar issues, compilar desde código fuente, estándares de código, cómo llega un PR de un fork a `main`. Leer antes de tocar el flujo de build o de PR.
- **`AGENTS.md`** — estructura del repo, comandos de build/test/lint, convenciones de commit y PR, importado desde `CLAUDE.md`.
- **`maintainer-docs/releasing-the-sdk.md`** — pasos para cortar un release del SDK. Sólo relevante si algún día César tiene permisos de maintainer.
- **`maintainer-docs/ci-setup.md`** — cómo están armados los workflows de GitHub Actions bajo `.github/workflows/`.
- **`docs/plans/2026-02-22-kroll-locking-modernization-design.md`** — propuesta de reemplazar los locks legacy (`pthread_mutex_t`, `NSLock`, `OSAtomic`, etc.) de la capa Kroll/TitaniumKit de iOS por GCD dispatch queues, para iOS 15+. Fechada 2026-02-22; no verificado si se implementó o sigue como propuesta — revisar antes de asumir su estado.
- **`docs/superpowers/plans/2026-10-28-android-keystore-quoted-path.md`** — plan y checklist del fix, con las adaptaciones de alcance y resultados; fuera del PR.
- **`docs/project/status.md`** — punto de entrada al retomar: estado comprobado, SDK instalado y siguiente paso; leer bajo demanda.
- **`docs/project/requirements.md`** — qué debe cumplir cada fix en curso.
- **`docs/project/decisions.md`** — causa raíz y por qué de cada fix ya enviado, y de cada SDK combinado armado.

## Pruebas oficiales y artefactos locales

`tests/Resources/*.test.js` y sus fixtures pertenecen a la suite oficial del SDK, que se ejecuta en una app de pruebas. `cli/tests/` y `android/cli/tests/` contienen pruebas de CLI con Mocha; el script oficial `npm run test:cli` recoge `**/cli/tests/test-*.js`. Las pruebas de regresión que añadimos para un fix se incluyen junto con el código en el PR, y pasan a upstream cuando se integra. No se borran por haber terminado la comprobación local.

`android/cli/tests/test-keystore-path.js` contiene las 20 pruebas propuestas en el PR #14629. Está en la rama del PR y en la rama local combinada; no estaba en el `main` descargado al publicarlo.

Las carpetas `.gradle/` son cachés generadas. Las notas `docs/project/`, planes de trabajo y logs son documentación o artefactos locales, separados de las pruebas que se contribuyen. Mantenerlos fuera de los PRs de código.

## Convenciones de este repo (no las del resto de proyectos de César)

- **Nunca cambios manuales en Xcode ni en archivos del SDK ya compilado.** Todo cambio pasa por los archivos fuente de este repo y se recompila.
- **PRs y commits en inglés, primera persona, sin atribución de IA.** Detalle completo y el porqué en `decisions.md` (2026-09-14 y 2026-09-17). Antes de publicar cualquier comentario o PR: correr el pase de revisión de `/humanizer` sobre el texto ya terminado, `grep -inE 'claude|session'` debe salir vacío, y mostrárselo a César antes de publicar.
- **`--version-tag` de `node ./build/scons cleanbuild` reemplaza el nombre completo de la carpeta instalada, no lo sufija.** Pasar siempre el nombre completo con prefijo de versión (`13.4.1.fixes`, no `fixes`) — ver `decisions.md` 2026-09-21 para el incidente que esto evitó.
- **Antes de creer un resumen de "exit code 0" de una tarea en segundo plano, leer el marcador de exit code real dentro del propio log.** El wrapper que redirige a un archivo puede reportar éxito aunque el comando real haya fallado.

## Issues/problemas conocidos del propio SDK

- `TitaniumKit.xcframework`, slice Mac Catalyst, usa la estructura versionada de framework de macOS (`Versions/A/`) con symlinks frágiles — `ti sdk install` (extracción ZIP) los pierde. Ver `decisions.md` 2026-02 (PR #14391).
- `TitaniumKit` aparece SIEMPRE en rojo en el navegador de proyecto de Xcode (de larga data, relacionado con la resolución de referencias de framework) — cosmético, no bloquea el build.
- El directorio compartido `~/Library/Application Support/Titanium/modules/` (módulos empaquetados por defecto: facebook, ti.map, hyperloop, etc.) puede quedar en estado inconsistente si un build/install se interrumpe a medio copiar. Ver `decisions.md` 2026-09-21 (build de `14.0.0.fixes`).

## Qué asistente hizo qué

| Cuándo | Asistente · modelo | Qué produjo |
| --- | --- | --- |
| 2026-02 a 2026-09-17 (varias sesiones) | Claude Code · modelo no registrado en las notas de entonces | PRs #14391 y #14417 (Mac Catalyst), colaboración en #14413 (V8, cerrado sin merge), convenciones de PR y no-atribución, PR #14581 inicial |
| 2026-09-21 | Claude Code · Sonnet 5 (`claude-sonnet-5`) | PRs de Android #14551/#14550/#14549 llevados a revisión, diagnóstico del check de CI roto en #14549, SDKs combinados `13.4.1.fixes` y `14.0.0.fixes`, reparación del módulo Facebook compartido, limpieza de SDKs viejos, instalación de esta convención de notas |
| 2026-10-28 | Codex · GPT-6 | Fix de keystore entre comillas, 20 pruebas, aceptación aislada hasta keytool y build/install de Android e iOS como `14.0.0.fixes`; pruebas de César pendientes |
| 2026-11-01 | Codex · GPT-6 | Commit y PR #14629 del keystore; actualización de main y recompilación/install de `14.0.0.fixes` con las seis novedades de Michael y el fix; notas consolidadas y distinción entre suites oficiales y artefactos locales |

Los transcripts de sesiones anteriores a esta documentación no están indexados aquí; si hace falta retomar el detalle de una de ellas, `~/.claude/projects/-Users-cesar-Developer-titanium-sdk/` los tiene mientras la carpeta del proyecto no se haya movido o renombrado.
