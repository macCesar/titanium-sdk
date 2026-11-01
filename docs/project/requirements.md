# Requirements

Este repo es una contribución externa a `tidev/titanium-sdk` (fork `macCesar`), no un producto propio. El "contrato" aquí es: qué debe cumplir cada fix que César tiene en curso, y cómo se sabe que lo cumple.

## PR #14581 — keep the resumed activity on top of the activity stack

`TiApplication.getCurrentActivity()` debe devolver la activity que está realmente en pantalla después de que un cambio de dark/light mode recrea las activities. Criterio: tras recrear, abrir una ventana nueva no debe heredar `exitOnClose=true` del root oculto, y un `AlertDialog` debe adjuntarse a la activity visible, no a la raíz oculta. Repro y detalle: https://github.com/tidev/titanium-sdk/pull/14581

## PR #14551 — restore ActionBar up navigation after recreation

Tras una recreación nativa de Activity, el ActionBar debe conservar `displayHomeAsUp` y seguir invocando `onHomeIconItemSelected`. Criterio: el repro del PR (cambiar `overrideUserInterfaceStyle` con la ventana abierta) debe dejar la flecha visible y funcional después de recrear. https://github.com/tidev/titanium-sdk/pull/14551

## PR #14550 — make TabGroup opening lifecycle-safe

Abrir un `TabGroup` no debe quedarse colgado en el splash cuando su Activity dueña se recrea mientras `open()` está pendiente. Criterio: la promesa de `open()` debe resolverse con una activity utilizable de la misma ventana Titanium, o rechazarse si la original en verdad se está cerrando — nunca quedar pendiente para siempre. https://github.com/tidev/titanium-sdk/pull/14550

## PR #14549 — avoid needless activity recreation when night mode is unchanged

Asignar `Ti.UI.overrideUserInterfaceStyle` no debe recrear la activity si el tema resuelto en pantalla no cambia. Criterio: con el valor por defecto (`USER_INTERFACE_STYLE_UNSPECIFIED`) al arrancar, no debe verse el flash del splash; un cambio real de claro/oscuro sí debe seguir recreando. Medido en el propio PR con conteos de recreación antes/después. https://github.com/tidev/titanium-sdk/pull/14549

## Xcode: recommended build settings en el generador de proyecto

`iphone/cli/commands/_build.js` (~línea 3645+, sección `XCBuildConfiguration`) debe dejar de disparar las advertencias de "recommended settings" que Xcode muestra en todo proyecto generado por Titanium.

Seguro de aplicar:
- Enable Recommended Warning (Quoted Include In Framework Header)
- Whole Module Optimization (Swift Compilation Mode, sólo Release)
- String Catalog Symbol Generation (localización)
- Enable Parallelization in Command Line Builds Using '-target'

Necesita investigación antes de aplicarse (puede romper el build):
- Reset Symbol Stripping (`STRIP_INSTALLED_PRODUCT`, `STRIP_STYLE`, `STRIP_SWIFT_SYMBOLS`) — puede chocar con el stripping propio de Titanium
- Enable User Script Sandboxing (`ENABLE_USER_SCRIPT_SANDBOXING`) — alto riesgo, los scripts de build de Titanium acceden a archivos fuera del sandbox

Referencia visual: captura guardada como `10-.png` en la app de prueba `new-animations` (fuera de este repo).


## Android: rutas de keystore entre comillas

El mismo validador debe aceptar rutas del prompt y de `--keystore`. Resolver primero la entrada literal; si no es un archivo y tiene un par exterior coincidente de comillas simples o dobles, intentar una sola vez sin ese par. Un archivo existente con comillas reales en el nombre conserva prioridad. No eliminar una comilla suelta ni pares repetidamente, recortar espacios, interpretar shell o convertir barras. Reutilizar `node-appc` y las comprobaciones de archivo existentes; mantener mensajes, callback único y contraseñas intactos.

Criterios de aceptación: rutas absolutas, relativas y con `~`, espacios y apóstrofes interiores; ambas envolturas; prioridad literal; rechazo de entradas vacías, inexistentes, directorios y comillas malformadas. La ruta absoluta debe llegar a `cli.argv.keystore` y a keytool. Distinguir un rechazo de contenido ficticio de un fallo de ruta.

Para la aceptación de César, usar **14.0.0.fixes** en su flujo Android real: probar el prompt con ruta normal y entre comillas simples/dobles, y `--keystore` con la envoltura recibida literalmente. Si sólo llega una comilla por accidente, la entrada no se limpia. No registrar contraseñas ni contenido de keystores en estas notas. Windows sólo puede declararse verificado tras ejecutarlo allí.

Mantener el cambio acotado a código y pruebas, sin dependencias nuevas, TiNy, otros prompts ni API pública. Nota, plan y documentos de sesión quedan fuera del PR. César aprobó la descripción e indicó publicar el 2026-11-01: PR #14629 abierto contra `main`, enlazado al issue #14628. No hay reporte nuevo de aceptación en su flujo real.
