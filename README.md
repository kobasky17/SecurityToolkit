# SecurityToolkit
Triage de seguridad para Windows (procesos, red, tareas programadas, persistencia, Windows Defender) con veredicto en lenguaje simple. 100% portable, open-source friendly, creada por @kobasky para su comunidad.


**¿Tu PC anda raro y no sabés si es algo serio o paranoia?** SecurityToolkit la escanea en
un par de minutos y te dice, en lenguaje simple, si hay algo para revisar — sin que tengas
que ser analista de seguridad para entender el resultado.

Creado por **@kobasky** para su comunidad de ciberseguridad — contacto: wa.me/@KobaskyOfficall.
Si tenés una copia de esta herramienta sin esta atribución o con otro nombre, no es la
versión original: escribile a @kobasky para confirmarlo.

## ¿Qué hace?

Recolecta en un solo pase todo lo que un analista revisaría a mano en un triage inicial:
software instalado, conexiones de red activas por proceso, tareas programadas, puntos de
persistencia (arranque automático, servicios), configuración y logs de Windows Defender, y
procesos corriendo (hash, firma digital, árbol padre/hijo, consumo de CPU/memoria). Después
de escanear, la GUI muestra un veredicto directo (🟢 todo bien / 🟡 revisar / 🔴 importante)
en vez de un volcado técnico ilegible — y si hace falta, un botón para pedir asistencia.

## ¿Por qué existe?

La mayoría de la gente no tiene forma de saber si su PC está comprometida más allá de "se
puso lento" o "me aparece algo raro", y las herramientas profesionales de verdad (EDR, XDR)
son caras, complejas o pensadas para empresas. SecurityToolkit achica esa brecha para la
comunidad: es gratuita, corre local (nada se manda a ningún servidor), y cada hallazgo viene
con el motivo exacto por el que se marcó — nada de caja negra ni "score" sin explicación.

## Principales características

- **Triage completo en un escaneo**: software, red, tareas programadas, persistencia,
  Windows Defender y procesos — con hallazgos explicables (severidad + motivo).
- **Resultado en lenguaje simple**: veredicto visual en la GUI, no solo logs técnicos.
- **100% portable**: un `.exe` que corre en cualquier Windows 10/11 sin instalar Python ni
  nada — ver [Empaquetado portable](#empaquetado-portable-todo-incluido-sin-instalar-nada-aparte).
- **Reportes para humanos y para máquinas**: `.txt` resumido y organizado, `.csv` para abrir
  como tabla, y `.json` estructurado para integraciones futuras.
- **Verificable**: el `.exe` incluye un hash SHA256 comprobable desde la propia app, para
  confirmar que tu copia no fue alterada por un tercero.

Cada escaneo genera su propia carpeta en `reports/<host>_<fecha>/` con **un .txt por
colector** (no todo mezclado en un solo archivo gigante), un **.csv por colector** (para
abrir como tabla ordenable/filtrable), un resumen de hallazgos sospechosos y un `.json`
estructurado con todo junto:

```
reports/DESKTOP-XXX_2026-09-20T10-54-06/
  00_hallazgos_sospechosos.txt   <- empezar por acá
  software.txt        software.csv
  network.txt          network.csv
  scheduled_tasks.txt  scheduled_tasks.csv
  persistence.txt      persistence.csv
  defender_logs.txt    defender_logs.csv
  processes.txt        processes.csv
  full_report.json                # todo junto, estructurado
  for_claude.json                  # solo si se pasó --for-claude
```

### Para revisar los reportes más rápido: klogg

Los `.txt` ya vienen organizados y resumidos, pero para un análisis más cómodo se
recomienda [klogg](https://klogg.filimonov.dev/) (visor de logs open source, portable,
sin instalar) — abre cualquiera de los `.txt` directo (sin reformatear nada) y permite
resaltar/filtrar en vivo con regex, por ejemplo `HIGH|MEDIUM` para ver solo lo sospechoso,
o el nombre de un proceso para seguirle el rastro. Los `.csv` están pensados para abrirse
en klogg igual, o en Excel/LibreOffice Calc para ordenar por columna.

Se evaluó también [Timeline Explorer](https://ericzimmerman.github.io/) (el estándar de
facto en DFIR para revisar CSV) pero es gratuito y **no** de código abierto, así que no se
adoptó como dependencia recomendada por defecto.

## Requisitos

```
pip install -r requirements.txt
```

Recomendado correr **como Administrador**: varias secciones (logs de Defender, ver todas las
conexiones/PIDs, algunas claves de registro) quedan incompletas sin elevación. La herramienta
no falla si no hay admin, solo anota el error en esa sección.

## Uso — CLI

```
python -m security_toolkit.cli                       # corre todos los colectores
python -m security_toolkit.cli --only processes,network
python -m security_toolkit.cli --output C:\ruta\salida
python -m security_toolkit.cli --for-claude           # además genera un JSON condensado
```

Colectores disponibles: `software`, `network`, `scheduled_tasks`, `persistence`,
`defender_logs`, `processes`.

## Uso — GUI

```
python -m security_toolkit.gui
```

Ventana con checkboxes por colector, log en vivo (corre en un hilo aparte, no se congela),
y botones para abrir el reporte/carpeta al terminar, más **"Abrir con klogg"**: la primera
vez pide ubicar `klogg.exe` (si no está instalado, hay que descargarlo de
[klogg.filimonov.dev](https://klogg.filimonov.dev/)) y después recuerda la ruta en
`%LOCALAPPDATA%\SecurityToolkit\config.json` para las próximas veces.

## Integración futura con Claude Code

`security_toolkit/core/claude_bridge.py` genera un JSON condensado (`--for-claude` en el CLI):
en vez de mandar el detalle completo de todo (cientos de programas, miles de eventos), incluye
el detalle completo **solo** de los hallazgos sospechosos y conteos para el resto — así, cuando
se conecte con Claude Code, no se gastan tokens en listas sin interés. La invocación automática
del CLI de Claude queda para una siguiente iteración.

## Empaquetado portable (todo incluido, sin instalar nada aparte)

Desde la raíz del proyecto:

```
powershell -File build/build_all.ps1
```

Esto: (1) descarga la build portable oficial de klogg si no está ya en `assets/klogg/`,
(2) compila `dist/SecurityToolkit.exe` (~15 MB, onefile, pide elevación UAC automáticamente
al abrirlo — no necesita Python instalado en la PC destino), (3) copia `klogg/` al lado
del `.exe` en `dist/`, y (4) calcula el SHA256 del `.exe` final y lo deja en
`dist/SHA256SUMS.txt`. El resultado final es la carpeta `dist/` completa (exe + `klogg/` +
`SHA256SUMS.txt`): se copia tal cual a la PC destino y el botón "Abrir con klogg" de la GUI
funciona sin que el usuario tenga que instalar nada por separado.

**Verificación de integridad:** después de cada build, publicá el hash que imprime la
consola (o el contenido de `SHA256SUMS.txt`) en tu canal oficial. Cualquiera puede
confirmar que su copia no fue modificada abriendo "Acerca de" dentro de la propia app
(muestra el SHA256 del `.exe` que está corriendo) y comparándolo con el publicado.

Si solo querés recompilar el `.exe` sin tocar klogg: `pyinstaller build/security_toolkit.spec
--distpath dist --workpath build/work`. klogg no se versiona en git (pesa ~43 MB); para
volver a bajarlo solo: `powershell -File build/download_klogg.ps1`.

**Nota sobre 32/64-bit:** un .exe de PyInstaller solo corre en la arquitectura del Python usado
para compilarlo. El Python de este entorno es 64-bit, así que ese build cubre Windows 10/11
64-bit. Para cubrir también Windows 32-bit hay que repetir el mismo comando con un intérprete
Python 32-bit instalado aparte; el código fuente no tiene ramas específicas por bitness más
allá del manejo ya incluido de `WOW6432Node` en el registro, así que no requiere cambios.

## Terceros incluidos

`dist/klogg/` es una copia sin modificar de la build portable oficial de
[klogg](https://github.com/variar/klogg) (GPL), incluyendo su `COPYING` y `NOTICE`
originales — se redistribuye tal cual la publica su autor, no se cambió nada.


