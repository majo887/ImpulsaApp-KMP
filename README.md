# Impulsa — Kotlin Multiplatform (Android + Desktop)

App de **gestión de ventas y comisiones** para equipos comerciales. Migrada desde un prototipo Android (Views XML + Fragments)
a **Kotlin Multiplatform + Compose Multiplatform**: una sola base de código para **Android** y **Desktop (Windows)**.

## Tecnologías
| Capa | Tecnología |
|---|---|
| Lenguaje / build | Kotlin 2.0.21, Gradle Kotlin DSL, catálogo de versiones (`gradle/libs.versions.toml`) |
| UI | Compose Multiplatform 1.7.3 + Material 3 (UI 100 % compartida) |
| Arquitectura | MVVM + repositorio, inyección manual (`AppContainer`) |
| Navegación | Propia, en `commonMain` (`Navigator` + `Route`), sin duplicar por plataforma |
| Persistencia | SQLDelight 2.0.2 sobre SQLite (driver Android / driver JDBC en Desktop) |
| Asincronía | Kotlin Coroutines + Flow |

## Estructura
```
composeApp/src
├── commonMain/                      ← ~95 % del código
│   ├── kotlin/com/impulsa/app/
│   │   ├── ImpulsaApp.kt            ← punto de entrada de la UI + host de navegación
│   │   ├── core/                    ← formatos, fechas, textos, AppInfo (créditos)
│   │   ├── domain/                  ← modelos + interfaz ImpulsaRepository
│   │   ├── data/                    ← repositorio SQLDelight, datos semilla, DatabaseDriverFactory (expect)
│   │   ├── di/                      ← AppContainer
│   │   ├── navigation/              ← Route, Navigator, ViewModel base, PlatformBackHandler (expect)
│   │   └── ui/                      ← theme/, components/ y una carpeta por pantalla (Screen + ViewModel)
│   ├── sqldelight/…/Impulsa.sq      ← esquema y consultas SQL
│   └── composeResources/drawable/   ← logo oficial (impulsa.jpg) e ícono de ventana
├── androidMain/                     ← MainActivity, manifiesto, íconos, driver SQLite Android, BackHandler
├── desktopMain/                     ← Main.kt (ventana), driver SQLite JDBC, BackHandler (no-op)
├── commonTest/                      ← pruebas de formatos y navegación
└── desktopTest/                     ← pruebas del repositorio con SQLite real
```

## Pantallas (9)
Login · Inicio (dashboard) · Ventas (lista + búsqueda + filtros) · Desempeño · Comisiones · Perfil ·
Registrar/Editar venta · Detalle de venta · Acerca de Impulsa (descripción y créditos).

## Navegación compartida
`Route` (sealed interface) define los destinos; `Navigator` mantiene la pila (`navegar`, `retroceder`, `reiniciarEn`,
`seleccionarPrincipal`) y el ciclo de vida de cada ViewModel. En pantallas angostas se muestra barra inferior; en ventanas
anchas (≥ 840 dp, Desktop) un riel lateral. El botón "atrás" de Android se conecta con `PlatformBackHandler` (expect/actual).

## Persistencia
SQLite con tablas `AsesorEntity`, `VentaEntity` y `ConfigEntity` (sesión y preferencias). Al primer arranque se cargan los datos
de ejemplo del prototipo. CRUD de ventas: crear, consultar, **editar** y **eliminar** (los acumulados del asesor se ajustan).
- Android: `/data/data/com.impulsa.app/databases/impulsa.db`
- Windows: `C:\Users\<usuario>\.impulsa\impulsa.db` (borra ese archivo para reiniciar los datos)

## Requisitos
JDK 17+ (Android Studio ya lo incluye) y Android Studio Koala o superior. Para Android: SDK 34.

## Cómo ejecutar
1. Abrir la carpeta `ImpulsaApp` en Android Studio y esperar el *Gradle Sync*.
   (El ZIP original no traía `gradlew`; Android Studio usa su Gradle. Para generarlo: `gradle wrapper --gradle-version 8.7`.)
2. **Android**: elegir la configuración `composeApp` y ▶ Run en un emulador/dispositivo (API 24+).
3. **Desktop (Windows)**: en la terminal de Android Studio:
   ```
   gradlew.bat :composeApp:run
   ```
   (o `gradle :composeApp:run`). 

## Comandos útiles
```
gradlew.bat :composeApp:assembleDebug          # APK Android
gradlew.bat :composeApp:run                    # ejecutar Desktop
gradlew.bat :composeApp:desktopTest            # pruebas del repositorio
gradlew.bat :composeApp:allTests               # todas las pruebas
gradlew.bat :composeApp:packageMsi             # instalador de Windows (requiere WiX Toolset)
```

## Usuario de prueba
Cualquier correo válido y contraseña de 6+ caracteres. Si el correo coincide con un asesor (p. ej. `pedro@impulsa.co`) se usa ese
perfil; si no, se usa Ana García (comportamiento del prototipo). **No hay autenticación real ni servicios externos** (no hay Firebase ni API).

## Pendientes / limitaciones conocidas
- Autenticación simulada (igual que el prototipo); "¿Olvidaste tu contraseña?" y "Solicitar liquidación" solo muestran un aviso.
- Los chips Hoy/Semana/Mes del Inicio son visuales (igual que el prototipo).
- Textos "Agosto" fijos (los datos de ejemplo son de agosto).
- Editar en `core/AppInfo.kt` los nombres de los integrantes y datos académicos (hoy son marcadores).
- Textos en código (`Textos.kt`), sin internacionalización.
- El proyecto es un único módulo Gradle con capas por paquetes; separar `data/domain` en un módulo propio es posible si se requiere.
- Ícono `.ico` de Windows para el instalador no incluido (se usa el logo en la ventana).


Elaborado por
Daniel Esteban Arciniegas Barrera
Maria Jose Peña Anacona
Dayerli Katerine Tamayo Solarte
