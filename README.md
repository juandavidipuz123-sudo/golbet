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
