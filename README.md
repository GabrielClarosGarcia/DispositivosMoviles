# Osden360

Osden360 es un proyecto académico de aplicación móvil para la búsqueda y venta de terrenos en Cochabamba. La interfaz está basada en el prototipo de Figma entregado para el proyecto y la organización del código sigue los conceptos usados en el material de la materia: Kotlin Multiplatform, Compose Multiplatform, ViewModel, flujo MVI, Koin y una capa preparada para Ktor.

## Alcance de esta entrega

El proyecto contiene un MVP funcional que puede abrirse en Android Studio y que reproduce el flujo principal del prototipo:

1. Splash de Osden360.
2. Inicio de sesión.
3. Registro de usuario.
4. Pantalla de inicio.
5. Catálogo de terrenos.
6. Detalle de terreno.
7. Filtros por zona, precio y superficie.
8. Favoritos.
9. Formulario para contactar a la inmobiliaria.
10. Perfil y cierre de sesión.

Los datos de terrenos, usuarios y consultas funcionan localmente para que el proyecto sea demostrable sin depender de un backend externo. También se incluye una clase `KtorPropertyApiService` que deja preparada la capa de red para conectar una API REST posteriormente.

## Tecnologías

- Kotlin Multiplatform.
- Compose Multiplatform.
- Material 3.
- AndroidX Lifecycle ViewModel.
- StateFlow y SharedFlow.
- Koin 4.2.2.
- Ktor 3.4.1.
- Kotlin Serialization.
- Coroutines.

Las versiones principales se mantienen alineadas con el ejemplo de clase revisado en la rama `feature/userinformation` del repositorio del docente.

## Estructura del proyecto

```text
Osden360_KMP/
├── androidApp/
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── kotlin/org/ucb/osden360/
│       │   ├── MainActivity.kt
│       │   └── MainApplication.kt
│       └── res/
├── shared/
│   └── src/
│       ├── commonMain/kotlin/org/ucb/osden360/
│       │   ├── data/
│       │   ├── di/
│       │   ├── domain/
│       │   ├── presentation/
│       │   ├── ui/
│       │   └── App.kt
│       ├── androidMain/
│       └── iosMain/
├── gradle/
│   └── libs.versions.toml
├── build.gradle.kts
├── settings.gradle.kts
└── README.md
```

## Arquitectura

El proyecto usa una separación por capas similar a la vista en clase.

### Data

Contiene implementaciones concretas y la preparación del consumo de servicios.

- `InMemoryLandRepository`: mantiene los terrenos, favoritos y consultas durante la ejecución de la aplicación.
- `InMemorySessionRepository`: mantiene la sesión local.
- `PropertyDto`: representación de datos para una API.
- `PropertyMapper`: convierte un DTO al modelo de dominio.
- `PropertyRemoteDataSource`: contrato del servicio remoto.
- `KtorPropertyApiService`: implementación de ejemplo con Ktor.

El repositorio local es el que está registrado en Koin para que el proyecto funcione inmediatamente sin servidor.

### Domain

Contiene reglas y contratos independientes de la interfaz.

Modelos:

- `Property`
- `User`
- `Inquiry`
- `PropertyFilter`

Repositorios:

- `LandRepository`
- `SessionRepository`

Casos de uso:

- `AuthenticateUseCase`
- `RegisterUseCase`
- `GetPropertiesUseCase`
- `ToggleFavoriteUseCase`
- `SendInquiryUseCase`

### Presentation

La capa de presentación usa ViewModel y un flujo de estilo MVI.

Cada interacción importante se representa mediante eventos. El ViewModel procesa esos eventos, modifica un `StateFlow` y emite efectos mediante `SharedFlow` cuando corresponde.

Ejemplo del catálogo:

```text
Pantalla
   ↓
PropertyEvent
   ↓
PropertyViewModel
   ↓
UseCase / Repository
   ↓
PropertyState
   ↓
Compose recompone la interfaz
```

El flujo de autenticación funciona de la misma manera con `AuthEvent`, `AuthState` y `AuthEffect`.

## Inyección de dependencias con Koin

La configuración está dividida en tres módulos:

```text
DataModule
DomainModule
PresentationModule
```

`DataModule` registra los repositorios.

`DomainModule` registra los casos de uso.

`PresentationModule` registra los ViewModel mediante `viewModelOf`.

Android inicializa Koin desde `MainApplication`. La inicialización iOS está preparada en `KoinModule.ios.kt`.

## Navegación

Para mantener el proyecto pequeño y concentrado en los temas de la materia no se agregó una librería externa de navegación. `App.kt` controla la ruta actual y conserva el terreno seleccionado.

Las rutas implementadas son:

```text
Splash
Login
Register
Home
Catalog
Favorites
Profile
Detail
Filters
Contact
```

La navegación inferior mantiene las cuatro secciones principales del Figma:

```text
Inicio | Catálogo | Favoritos | Perfil
```

## Pantallas

### Splash

Usa el fondo azul oscuro de la identidad visual, el acento dorado y el nombre Osden360.

### Login

Incluye correo, contraseña, recuperación visual de contraseña, inicio de sesión y acceso al registro.

La cuenta prellenada es:

```text
Correo: carlos@correo.com
Contraseña: 123456
```

También se puede iniciar sesión con otro correo siempre que correo y contraseña no estén vacíos.

### Registro

Solicita:

- Nombre completo.
- Correo electrónico.
- Número de teléfono.
- Contraseña.

La contraseña debe tener al menos seis caracteres.

### Inicio

Muestra:

- Nombre del usuario.
- Barra de búsqueda.
- Terrenos destacados.
- Zonas populares.
- Tarjetas de terrenos.

### Catálogo

Muestra todos los terrenos que cumplen la búsqueda y los filtros activos. Desde cada tarjeta se puede abrir el detalle o alternar el estado de favorito.

### Detalle

Muestra:

- Nombre.
- Zona.
- Ciudad.
- Precio.
- Superficie.
- Descripción.
- Coordenadas.
- Representación visual de mapa.
- Botón para contactar a la inmobiliaria.

El mapa es referencial. No se incluyó un SDK de Google Maps porque el prototipo y el material proporcionado no definen una clave de mapas ni un proveedor concreto.

### Filtros

Permite seleccionar una o varias zonas y ajustar rangos de precio y superficie.

Zonas disponibles:

- Tiquipaya.
- Sacaba.
- Quillacollo.
- Valle Bajo.
- Zona Sur.

### Favoritos

Los favoritos se guardan dentro del repositorio local durante la ejecución de la aplicación. Se incluyen dos favoritos iniciales para reflejar el prototipo.

### Contactar inmobiliaria

El formulario se inicializa con:

- Terreno seleccionado.
- Nombre del usuario.
- Correo.
- Teléfono.
- Mensaje sugerido.

Al enviar se crea una consulta local y se muestra un Snackbar.

### Perfil

Muestra los datos de sesión, la cantidad de favoritos, las consultas registradas y la opción de cerrar sesión.

## Datos incluidos

El proyecto contiene terrenos de demostración para:

- Tiquipaya.
- Sacaba.
- Valle Bajo.
- Quillacollo.
- Zona Sur.

Los precios y superficies están inspirados en el prototipo para que las pantallas se vean completas desde el primer inicio.

## Cómo conectar una API real

La estructura ya contiene:

```text
data/dto/PropertyDto.kt
data/mapper/PropertyMapper.kt
data/datasource/PropertyRemoteDataSource.kt
data/service/KtorPropertyApiService.kt
```

`KtorPropertyApiService` espera un endpoint:

```text
GET {BASE_URL}/properties
```

El servidor debería devolver una lista JSON compatible con esta forma:

```json
[
  {
    "id": "1",
    "title": "Terreno Residencial Tiquipaya",
    "zone": "Tiquipaya",
    "city": "Cochabamba",
    "price": 185000,
    "area": 450,
    "description": "Descripción del terreno",
    "latitude": -17.3301,
    "longitude": -66.2177,
    "featured": true
  }
]
```

Cuando exista una API real se puede crear `RemoteLandRepository` y cambiar el registro de Koin de:

```kotlin
single<LandRepository> { InMemoryLandRepository() }
```

por una implementación que use `PropertyRemoteDataSource`.

## Relación con el ejemplo de GitHub de la materia

La organización de red preparada para Osden360 sigue la misma idea del ejemplo de información de usuario visto en clase:

```text
DTO
 ↓
Mapper
 ↓
RemoteDataSource
 ↓
Ktor Service
 ↓
Repository
 ↓
UseCase
 ↓
ViewModel
 ↓
Screen
```

Para la demostración actual se usa un repositorio en memoria en lugar de una API porque el proyecto de Osden360 todavía no tiene una URL de backend definida.

## Relación con MVI

Los ViewModel principales siguen un flujo unidireccional.

Ejemplo al agregar un favorito:

```text
Usuario toca corazón
        ↓
PropertyEvent.ToggleFavorite
        ↓
PropertyViewModel
        ↓
ToggleFavoriteUseCase
        ↓
LandRepository
        ↓
StateFlow de terrenos cambia
        ↓
Compose actualiza Home, Catálogo y Favoritos
```

## Diseño

La paleta principal está definida en `OsdenTheme.kt`.

```text
Azul oscuro: #071B33
Dorado: #D8AD24
Fondo: #F5F6F8
```

El diseño implementa el estilo general del Figma con tarjetas blancas, bordes redondeados, navegación inferior, formularios limpios y acentos dorados.

No se incluyeron las fotografías exactas del Figma porque el PDF no proporciona los archivos originales de cada fotografía como recursos independientes. En su lugar se usa `PropertyVisual`, que genera una representación visual local por zona y evita dependencias externas de imágenes.

## Android

Abrir la carpeta raíz en Android Studio y esperar la sincronización de Gradle.

Configuración usada:

```text
compileSdk = 36
minSdk = 24
targetSdk = 36
Java 11
```

La clase de inicio es:

```text
androidApp/src/main/kotlin/org/ucb/osden360/MainActivity.kt
```

Koin se inicializa desde:

```text
androidApp/src/main/kotlin/org/ucb/osden360/MainApplication.kt
```

## iOS

El módulo `shared` contiene target para:

```text
iosArm64
iosSimulatorArm64
```

También se incluye `MainViewController.kt` e inicialización de Koin para iOS. Esta entrega no incluye un proyecto Xcode completo porque el objetivo principal es el proyecto Android/KMP de la materia. Si se requiere ejecución nativa en iOS, se puede crear el módulo `iosApp` con el asistente de Kotlin Multiplatform y utilizar `MainViewController` como vista raíz.

## Qué es local y qué es real

Actualmente funcionan localmente:

- Inicio de sesión.
- Registro.
- Terrenos.
- Búsqueda.
- Filtros.
- Favoritos.
- Consultas.
- Perfil.

Está preparado pero necesita un backend para ser real:

- Persistencia permanente de usuarios.
- Autenticación con servidor.
- Catálogo remoto.
- Publicación y edición de terrenos.
- Envío real de consultas.
- Recuperación de contraseña.

También necesitan servicios externos si se desean implementar completamente:

- Google Maps u otro proveedor de mapas.
- Carga remota de fotografías.
- Notificaciones push.

## Siguiente evolución recomendada

La siguiente etapa natural es crear una API REST para terrenos y reemplazar `InMemoryLandRepository`. Después se puede agregar autenticación real y persistencia de favoritos.

## Código

Los archivos de código no contienen comentarios. La explicación técnica y funcional se concentra en este README, tal como se solicitó.
