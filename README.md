# GolBet

## 1. Requisitos

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) (`dotnet --version` debe ser 8.x)
- SQL Server en ejecución y accesible desde `localhost`
- Puertos libres `5034` (HTTP) y `7226` (HTTPS)

## 2. Configurar la base de datos

La conexión vive en `GolBet.Web/appsettings.json`:

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost;Database=GolBetDB;User Id=sa;Password=GolBet*2026;TrustServerCertificate=True;MultipleActiveResultSets=true"
}
```

Ajusta `Server`, `User Id` y `Password` a tu entorno. No hay que crear la BD a mano: al arrancar, la app aplica lo necesario y `DbSeeder` inserta equipos y partidos si están vacíos.

## 3. Compilar

Desde la raíz de la solución (`golbet/`, donde está `GolBet.sln`):

```bash
dotnet build
```

Esperado: `0 Error(s)`.

## 4. Ejecutar

```bash
dotnet run --project GolBet.Web
```

Por defecto usa el perfil `https` (`https://localhost:7226;http://localhost:5034`).

Solo HTTP (útil en contenedores sin certificado):

```bash
dotnet run --project GolBet.Web --launch-profile http
```

o:

```bash
dotnet run --project GolBet.Web --urls "http://localhost:5034"
```

## 5. Probar en el navegador

- `/` → inicio
- `/Matches` → cartelera (6 partidos)
- `/Matches?status=Scheduled` → programados
- `/Matches?status=InProgress` → en juego
- `/Matches?status=Finished` → finalizados
- `/Matches/Detail/1` → detalle de un partido
- `/Matches/Detail/999` → 404 esperado

## 6. Error típico: certificado HTTPS

Si al arrancar ves:

```text
Unable to configure HTTPS endpoint ... run 'dotnet dev-certs https'
```

Solución:

```bash
dotnet dev-certs https
dotnet run --project GolBet.Web
```

Si estás en un contenedor y no necesitas HTTPS, usa el perfil `http` del punto 4.

## 7. Gestión y cuentas (módulos 6 y 7)

Al iniciar se aplican las migraciones y se crean los roles `Admin` y `Apostador`.
La cuenta de administración del ejercicio es `admin@golbet.com`, contraseña
`Admin123*`, como indica el módulo 7.

- `/Account/Register`: registra un apostador con 100.000 FutCoins e inicia sesión.
- `/Account/Login`: inicio de sesión; el cierre de sesión usa un formulario POST.
- `/Teams`: crear, editar y desactivar equipos, solo para Admin.
- `/Matches/Create` y `/Matches/Edit/{id}`: crear y editar partidos, solo para Admin.
- El detalle permite al Admin editar o desactivar el partido mediante POST.

Los formularios validan los campos en cliente y servidor. El servicio impide
partidos con equipos iguales o fecha pasada. Las fechas del formulario son de
Colombia y se almacenan en UTC. Desactivar conserva los registros y los retira
del listado. Las cuotas usan campos numéricos con dos decimales.

La migración de Identity supone que `Bets` está vacía, como en el módulo 7:
no asigna usuarios ficticios a apuestas existentes.

## 8. Buscador de partidos (segunda entrega)

La búsqueda es GET y se combina con el estado:

```text
/Matches?search=nacional
/Matches?search=NACIONAL&status=Scheduled
/Matches?search=equipo-inexistente&status=Finished
```

El repositorio filtra por el nombre del equipo local o visitante antes de
materializar la consulta. La collation de `Team.Name` definida en el módulo 2
ignora mayúsculas. El servicio propaga el parámetro y el controller conserva
`CurrentSearch` y `CurrentStatus` en `ViewBag`.

Comprobación manual:

1. Como anónimo, abrir cartelera y detalle; confirmar que no hay botones de gestión.
2. Buscar `nacional` y `NACIONAL`: deben devolver los mismos partidos.
3. Cambiar entre Programados, En juego, Finalizados y Todos: la búsqueda permanece.
4. Buscar con un filtro activo: se conserva el estado. Refrescar o copiar la URL
   debe conservar ambos valores.
5. Buscar un equipo inexistente: aparece el mensaje y el enlace «Volver a la
   cartelera de partidos», que limpia ambos filtros.
6. Ingresar como Admin y crear un partido futuro con Nacional de visitante.
   Buscar `nacional`: deben aparecer partidos con Nacional como local y visitante.
   El seeder original solo lo incluye como local.
7. Crear, editar y desactivar equipos y partidos. Probar equipos iguales, fecha
   pasada y cuota menor o igual a 1: deben mostrarse errores sin guardar.
8. Registrar un apostador: comprobar nombre y 100.000 FutCoins; intentar abrir
   `/Teams` y `/Matches/Create`: debe aparecer Acceso denegado.
9. Cerrar sesión y volver a ingresar: el saldo permanece.

Verificación realizada en desarrollo: compilación .NET 8 sin errores, migración
sobre SQL Server 2022 vacío y reinicio sin duplicar semillas, pruebas HTTP de los
recorridos anteriores y revisión del SQL generado con búsqueda y estado en el
`WHERE`. NuGet reporta la advertencia `NU1903` de AutoMapper 13.0.1, versión que
ya utilizaba el proyecto.

Los tags `modulo-6` y `modulo-7` marcan los cierres de los módulos. Los commits
posteriores incorporan la búsqueda y el ajuste de los campos numéricos.

La entrega también requiere el video personal de 5 a 8 minutos, en una toma
continua, y publicar las URL del repositorio público y del video en el Excel de
Teams. Seguir el orden y las condiciones de `03_Entrega_Buscador_Partidos.md`.
