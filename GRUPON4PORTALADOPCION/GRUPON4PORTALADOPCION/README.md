# Portal de Adopción — Grupo N.º 4

**Integrantes:** Catalina Giordano y Francisco Ledesma  
**Materia:** Práctica Profesionalizante · ICES · 2026

---

## 1. Documentación

El proyecto cuenta con documentación funcional, técnica y visual para comprender el alcance del sistema, sus reglas de negocio y los flujos de interacción.

| Recurso | Enlace |
|---|---|
| Documentación general | [Abrir documento](https://docs.google.com/document/d/1w4Xe_tpaFmimHWNv-FOr7oU1mljxRcA6uU23xCQ0NzE/edit?tab=t.0) |
| Mockups (Excalidraw) | [Ver mockups](https://excalidraw.com/#room=8fd5edbd3dc8d019f2b7,l1CTB_eTqamJcjB-0FAZAQ) |
| Casos de uso | [Abrir documento](https://docs.google.com/document/d/18lrj1Eu7lIVF9J3BuKHSHggcuzbWSIxXGOIXdsvKoTc/edit?tab=t.0) |
| Actores | [Abrir documento](https://docs.google.com/document/d/1XNMC8vHmskA5OjkIgs9R0YtFNa8JBHv-QOai8UG1BJg/edit?tab=t.0#heading=h.9m6ryyrg4dgd) |
| Casos de uso detallados | Carpeta `CASOS DE USO/` |
| Modelo de estados y reglas de negocio | `COSAS/Modelo-de-Estados-y-Reglas-de-Negocio.md` |
| Matriz de trazabilidad | `Informes/Trazabilidad_CU.md` |
| PRESENTACION | [Abrir canva](https://canva.link/cg4hzzjtt40amks) | 


---

## 2. Alcance del proyecto

### Descripción

El Portal de Adopción es un sistema de gestión para una organización no gubernamental (ONG) dedicada a la adopción de mascotas.

Su objetivo es centralizar la administración de usuarios, mascotas, solicitudes de adopción, seguimiento clínico, entrevistas, auditoría y reportes.

El sistema permite registrar animales, consultar su disponibilidad, iniciar procesos de adopción y realizar el seguimiento de cada solicitud hasta su resolución.

Cada solicitud comienza en estado `Pendiente` y atraviesa distintas etapas de evaluación. Durante el proceso se aplican controles de permisos, validaciones y reglas de negocio para asegurar la integridad de la información.

### Perfiles de usuario

| Perfil | Responsabilidades principales |
|---|---|
| Administrador | Gestionar usuarios, roles, mascotas, listas negras, auditoría y reportes. |
| Voluntario | Administrar mascotas y participar en la gestión y seguimiento de las adopciones. |
| Veterinario | Registrar historiales clínicos, eventos médicos y validaciones veterinarias. |
| Adoptante | Registrarse, iniciar sesión, consultar mascotas y gestionar sus solicitudes de adopción. |

### Casos de uso demostrados en la defensa

| Caso de uso | Operación | Endpoint principal | Resultado esperado |
|---|---|---|---|
| CU-06 | Registrar usuario | `POST /api/auth/registro` | `201 Created` |
| CU-07 | Iniciar sesión | `POST /api/auth/login` | `200 OK` |
| CU-09 | Registrar mascota | `POST /api/mascotas` | `201 Created` |
| CU-02 / CU-05 | Crear solicitud y proceso de adopción | `POST /api/solicitudes` | `201 Created` |
| CU-04 | Evaluar solicitud | `PATCH /api/solicitudes/{id}/evaluacion` | `200 OK` |

### Reglas principales de la consigna

1. No se puede crear una solicitud de adopción para una mascota que no esté en estado `Disponible`.
2. Un adoptante no puede tener más de un proceso de adopción activo simultáneamente.
3. Las solicitudes deben respetar las etapas y transiciones permitidas.
4. Las operaciones deben respetar los permisos del perfil autenticado.
5. Los cambios importantes deben quedar registrados para mantener la trazabilidad.

### Matriz de complejidad

| Nivel | Funcionalidad | Descripción | Estado en el código |
|---|---|---|---|
| 1 | Máquina de estados | Flujo controlado desde la solicitud hasta la finalización, pasando por entrevista y período de prueba. | Implementado. Las transiciones se validan en `PortalService`. |
| 2 | Abandono automático | Los procesos en entrevista sin actividad durante más de 7 días pasan a `Abandonada`. | Implementado mediante `AbandonoJob` y `AbandonAsync`. |
| 3 | Lista negra | Una cancelación durante el período de prueba genera una sanción de un año. | Implementado en `CancelAsync` y en la gestión de blacklist. |
| 4 | Transferencia clínica | Al finalizar una adopción, el historial clínico se transfiere al adoptante conservando la referencia de origen. | Implementado en `FinalizeAsync`. |

---

## 3. Arquitectura

El proyecto utiliza una arquitectura en capas para separar las responsabilidades y evitar que los controladores, las reglas de negocio y el acceso a datos dependan directamente entre sí.

### Estructura de carpetas

```text
Grupo-N-4-PortalAdopcion/
│
├── PortalAdopcion.slnx
│
├── API/
│   ├── Controllers/
│   │   └── PortalControllers.cs
│   ├── Middleware/
│   │   ├── ExceptionHandlingMiddleware.cs
│   │   └── ValidationErrorTranslator.cs
│   ├── Properties/
│   │   └── launchSettings.json
│   ├── wwwroot/
│   ├── Program.cs
│   ├── TokenIssuer.cs
│   ├── DemoSeed.cs
│   ├── AbandonoJob.cs
│   ├── CorreoJob.cs
│   ├── ReportFiles.cs
│   └── API.csproj
│
├── BusinessLogic/
│   ├── Abstractions/
│   │   └── IPortalService.cs
│   ├── Mapping/
│   │   └── PortalMapper.cs
│   ├── Services/
│   │   └── PortalService.cs
│   └── BusinessLogic.csproj
│
├── DataAccess/
│   ├── Entities/
│   ├── Migrations/
│   ├── Repositories/
│   │   ├── IPortalRepository.cs
│   │   └── PortalRepository.cs
│   ├── PortalAdopcionDbContext.cs
│   └── DataAccess.csproj
│
├── Shared/
│   ├── Constants/
│   │   └── BusinessRules.cs
│   ├── DTOs/
│   │   ├── PortalContracts.cs
│   │   └── PortalResponses.cs
│   ├── Enums/
│   ├── Exceptions/
│   ├── Security/
│   └── Shared.csproj
│
├── Tests/
│   └── Portal.Tests/
│       ├── BusinessRulesUnitTests.cs
│       ├── Verification.cs
│       ├── ValidationHandlingTests.cs
│       └── Portal.Tests.csproj
│
├── Bruno/
│   ├── 0 - Setup/
│   ├── 1 - Autenticacion/
│   ├── 2 - Usuarios y Blacklist/
│   ├── 3 - Mascotas y Clinica/
│   ├── 4 - Proceso de Adopcion/
│   ├── 5 - Administracion y Reportes/
│   └── environments/
│
├── CASOS DE USO/
├── COSAS/
├── Informes/
└── README.md
```

### Flujo de una operación

El recorrido general de una petición es:

```text
Cliente HTTP (Bruno / Frontend)
              │
              ▼
       API / Controller
              │
              ▼
       IPortalService
              │
              ▼
        PortalService
              │
              ▼
      IPortalRepository
              │
              ▼
       PortalRepository
              │
              ▼
    Entity Framework Core
              │
              ▼
          SQLite
              │
              ▼
       DTO de respuesta
              │
              ▼
        Respuesta HTTP
```

### Responsabilidades por capa

| Capa | Componentes | Responsabilidad |
|---|---|---|
| API | Controllers, Middleware, `Program.cs`, `TokenIssuer` | Recibir solicitudes HTTP, autenticar, autorizar, aplicar middleware y devolver respuestas. |
| BusinessLogic | `IPortalService`, `PortalService` | Ejecutar reglas de negocio, validar operaciones y coordinar los cambios. |
| DataAccess | `IPortalRepository`, `PortalRepository`, `PortalAdopcionDbContext` | Consultar y persistir entidades mediante Entity Framework Core. |
| Shared | DTOs, enums, excepciones, constantes y seguridad | Compartir contratos, reglas y tipos utilizados por las distintas capas. |
| Tests | xUnit, Moq y `WebApplicationFactory` | Verificar reglas de negocio e integración HTTP. |

**Criterios de separación:**

- El controlador no implementa las reglas de negocio.
- El servicio no ejecuta sentencias SQL directamente.
- El repositorio no decide permisos ni reglas funcionales.
- Los cambios relacionados se guardan mediante una operación de persistencia que incluye el evento de auditoría.
- Se utiliza control de concurrencia optimista mediante `Version`.

---

## 4. Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| C# | Lenguaje principal. |
| .NET 10 | Plataforma de desarrollo. |
| ASP.NET Core | Implementación de la API REST. |
| Entity Framework Core 10 | ORM y persistencia. |
| SQLite | Base de datos. |
| JWT | Autenticación mediante tokens. |
| xUnit | Pruebas automatizadas. |
| Moq | Simulación de dependencias en pruebas unitarias. |
| `WebApplicationFactory` | Pruebas de integración HTTP. |
| Bruno | Ejecución y validación manual de endpoints. |
| OpenAPI | Descripción de la API. |
| Scalar | Documentación interactiva de endpoints. |
| HTML, CSS y JavaScript | Interfaz web incluida en `API/wwwroot/`. |

---

## 5. Entidades

Las entidades principales se encuentran en `DataAccess/Entities/`.

| Entidad | Descripción |
|---|---|
| `Usuario` | Almacena identidad, email, DNI, contraseña cifrada mediante hash, rol, estado de cuenta y versión. |
| `Adoptante` | Contiene nombre completo, email, DNI, teléfono, dirección, edad, estado civil y datos laborales. |
| `Mascota` | Registra nombre, especie, raza, edad, sexo, foto, chip y estado. |
| `SolicitudDeAdopcion` | Representa el proceso de adopción, incluyendo adoptante, mascota, estado, motivo, código de confirmación, evaluador y fechas. |
| `SancionBlacklist` | Registra sanciones aplicadas a adoptantes, con motivo, período y estado de revocación. |
| `HistorialClinico` | Contiene peso, estado general, vacunas, antecedentes y esterilización. Puede asociarse a una mascota o a un adoptante. |
| `EventoMedico` | Registra tratamientos, descripciones y eventos médicos asociados a un historial. |
| `Entrevista` | Almacena fecha programada, voluntario, resultado y observaciones. |
| `Seguimiento` | Registra ánimo, alimentación, tareas, medicación, limpieza y alertas. |
| `Incidente` | Registra incidentes asociados a una solicitud. |
| `EventoAuditoria` | Guarda usuario responsable, fecha, acción, entidad afectada y motivo. |
| `Notificacion` | Almacena mensajes y sus destinatarios. |

### Estados de las mascotas

El enum `EstadoMascotaEnum` contempla:

- `Disponible`
- `Reservada`
- `EnEntrevista`
- `EnPeriodoDePrueba`
- `Adoptada`
- `EnTratamiento`
- `Inactiva`

La especie se representa mediante `EspecieEnum` (`Perro`, `Gato`) y el sexo mediante `SexoEnum` (`Macho`, `Hembra`).

### Estados de las solicitudes

El enum `EstadoSolicitudEnum` contempla:

- `Pendiente`
- `EnObservacion`
- `Aprobada`
- `EnEntrevista`
- `EnPeriodoDePrueba`
- `Adoptada`
- `Rechazada`
- `Cancelada`
- `Abandonada`

Los estados considerados activos por las reglas del servicio son `Pendiente`, `EnObservacion`, `Aprobada`, `EnEntrevista` y `EnPeriodoDePrueba`.

### Integridad y concurrencia

La base de datos implementa:

- Índices únicos para evitar duplicaciones de email y DNI.
- Índices únicos filtrados para impedir más de un proceso activo por mascota o adoptante.
- Índice único para el chip de la mascota.
- Índice único para el historial clínico asociado a una mascota.
- Índice único para evitar más de un seguimiento por día y solicitud.
- Control de concurrencia optimista mediante la propiedad `Version`.

---

## 6. Módulos principales

### 6.1. Autenticación y usuarios

El módulo permite:

- Registrar nuevos adoptantes.
- Iniciar sesión.
- Consultar la sesión actual.
- Administrar usuarios internos.
- Modificar roles y estados de cuenta.

El registro público crea usuarios con rol `Adoptante`. El DTO `RegistroUsuarioDTO` no permite seleccionar un rol.

La autenticación utiliza JWT con una duración de dos horas. El token contiene identificador, nombre, rol y versión del usuario. Para los adoptantes también incorpora su identificador de perfil.

La API vuelve a comprobar el estado y la versión de la cuenta al validar el token. De esta forma, una cuenta dada de baja o una sesión cuya versión dejó de coincidir puede quedar invalidada.

### 6.2. Mascotas

Permite:

- Registrar mascotas.
- Consultar el catálogo.
- Consultar una mascota por identificador.
- Modificar información.
- Cambiar su estado.
- Dar de baja lógicamente una mascota.

El catálogo público permite filtrar por estado y tiene una política de caché de 20 segundos.

Los perfiles `Administrador` y `Voluntario` pueden administrar el catálogo.

### 6.3. Adopciones

Permite:

- Crear solicitudes de adopción.
- Consultar solicitudes.
- Evaluar solicitudes.
- Cancelar procesos.
- Asignar voluntarios.
- Gestionar entrevistas.
- Registrar seguimientos.
- Validar condiciones veterinarias.
- Finalizar adopciones.

Al crear una solicitud, la mascota pasa a `Reservada` y se genera un código de confirmación. La operación se guarda junto con su auditoría.

### 6.4. Clínica

El módulo veterinario permite:

- Registrar el historial clínico inicial.
- Consultar el historial de una mascota.
- Registrar eventos médicos.
- Asociar tratamientos y observaciones.
- Validar las condiciones clínicas antes de finalizar una adopción.

Al finalizar el proceso, el historial clínico se transfiere al adoptante conservando la referencia de origen.

### 6.5. Entrevistas y seguimientos

Permite programar entrevistas, registrar sus resultados y documentar el período de prueba.

Los seguimientos incluyen:

- Ánimo.
- Alimentación.
- Tareas.
- Medicación.
- Limpieza.
- Alertas.

El sistema evita registrar más de un seguimiento para la misma solicitud y fecha.

También cuenta con un proceso automático de abandono para solicitudes en entrevista que superan el período de inactividad establecido.

### 6.6. Administración y reportes

Incluye:

- Gestión de usuarios y roles.
- Gestión de listas negras.
- Consulta de auditoría.
- Consulta de notificaciones.
- Registro y consulta de incidentes.
- Generación de reportes en CSV y PDF.

La consulta de auditoría y la generación de reportes están restringidas al perfil `Administrador`.

---

## 7. Reglas de negocio

Las reglas se centralizan principalmente en `BusinessLogic/Services/PortalService.cs` y `Shared/Constants/BusinessRules.cs`.

1. El registro público crea únicamente usuarios con rol `Adoptante`.
2. Los emails y DNI deben ser únicos.
3. Los usuarios internos se crean mediante operaciones autorizadas.
4. Una mascota debe estar `Disponible` para iniciar una solicitud.
5. Un adoptante no puede tener más de un proceso activo.
6. Una mascota no puede tener más de un proceso activo.
7. Una solicitud nueva comienza en estado `Pendiente`.
8. Al crear una solicitud, la mascota pasa a `Reservada`.
9. La evaluación solo se permite desde `Pendiente` o `EnObservacion`.
10. Los estados admitidos para la evaluación son `Aprobada`, `Rechazada` y `EnObservacion`.
11. Para aprobar una solicitud deben haber transcurrido al menos 15 días desde su creación.
12. Al rechazar una solicitud se libera la mascota.
13. Una solicitud aprobada puede avanzar a entrevista.
14. El período de prueba requiere seguimiento y validación veterinaria antes de la finalización.
15. La finalización requiere al menos 15 días de período de prueba y un historial clínico.
16. La cancelación durante el período de prueba genera una sanción de un año.
17. Los procesos en entrevista sin actividad durante más de 7 días pueden pasar a `Abandonada`.
18. Las operaciones relevantes registran eventos de auditoría.
19. Las modificaciones concurrentes se controlan mediante `Version`.

### Regla → HTTP

| Situación | Código HTTP | Comportamiento |
|---|---:|---|
| Operación realizada correctamente | `200` | Devuelve el resultado de la operación. |
| Recurso creado | `201` | Devuelve el recurso creado. |
| Operación sin contenido de respuesta | `204` | Confirma la operación. |
| Datos inválidos | `400` | Devuelve mensajes de validación. |
| Falta autenticación | `401` | Requiere iniciar sesión. |
| Rol o permiso insuficiente | `403` | Rechaza la operación. |
| Recurso inexistente | `404` | Indica que no se encontró el recurso. |
| Conflicto con reglas de negocio | `409` | Informa que la operación no puede realizarse en el estado actual. |



