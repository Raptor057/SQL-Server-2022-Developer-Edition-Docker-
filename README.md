# SQL Server 2022 Standard Edition (Docker + Compose)

Repositorio para levantar **SQL Server 2022 Standard** en Docker usando **docker-compose** con **.env** y **persistencia local** en la carpeta del proyecto.

> Aviso de licencia: la edicion **Standard** requiere **licencia valida** para su uso.

---

## Requisitos

- Docker Engine + Docker Compose (v2 o superior)
- Puerto **1433** libre en el host
- RAM disponible (recomendado: 8 GB o mas para uso productivo)

---

## Version de SQL Server utilizada

Este proyecto utiliza la imagen oficial de Microsoft:

- **SQL Server 2022 Standard Edition**
- **Cumulative Update:** CU21
- **Sistema base:** Ubuntu 22.04
- **Tag exacto:**

```

mcr.microsoft.com/mssql/server:2022-CU21-ubuntu-22.04

```

> Nota: originalmente se usaba `2022-latest`, pero se cambió a un **tag de CU fijo** para evitar correr versiones RTM sin parches acumulativos.

Fuente oficial de tags:
https://hub.docker.com/r/microsoft/mssql-server

---

## Archivos del proyecto

- `docker-compose.yml` - definicion del servicio SQL Server
- `.env` - variables de entorno locales (no compartir en publico)
- `.env.example` - plantilla con valores de ejemplo
- `sql-data/` - carpeta creada automaticamente para persistir datos
- `Visual Studio Code Connect.png` - captura de ejemplo de conexion

---

## Variables de entorno (.env)

Archivo `.env` en la misma carpeta del `docker-compose.yml`:

```env
ACCEPT_EULA=Y
MSSQL_PID_KEY=Standard
SA_PASSWORD=P@ssw0rd_Str0ng!

```

`MSSQL_PID_KEY` puede ser:

-   `Standard`
    
-   o una **clave de producto valida** (si aplica)
    

Reglas para `SA_PASSWORD`:

-   Minimo 8 caracteres
    
-   Debe incluir mayusculas, minusculas, digitos y simbolos
    
-   Si no cumple, el contenedor se apaga
    

----------

## Puesta en marcha

```bash
# 1) (Opcional) copiar plantilla
# cp .env.example .env

# 2) levantar en segundo plano
docker compose up -d

# 3) ver estado
docker compose ps

# 4) ver logs (hasta ver "Server is listening on ... 1433")
docker logs -f mssqlserver

```

----------

## Persistencia de datos

Los datos se guardan en `./sql-data` (relativo al `docker-compose.yml`) y se mapean a `/var/opt/mssql` dentro del contenedor.

En hosts Linux, es necesario ajustar permisos:

```bash
sudo mkdir -p sql-data
sudo chown -R 10001:0 sql-data
sudo chmod -R 770 sql-data

```

> `docker compose down` **no elimina los datos** mientras la carpeta `sql-data` exista.

----------

## Limites de recursos (Docker)

Para evitar que SQL Server consuma todos los recursos del host, se aplican limites a nivel Docker:

```yaml
mem_limit: 6g
cpus: 2

```

Estos valores estan pensados para un host con:

-   16 GB de RAM
    
-   CPU de 4 cores
    

Ajustar segun el hardware disponible.

----------

## Limite de memoria dentro de SQL Server (recomendado)

Adicionalmente al limite de Docker, se recomienda limitar la memoria interna del motor SQL Server.

Ejecutar una sola vez dentro de SQL Server:

```sql
EXEC sys.sp_configure 'show advanced options', 1;
RECONFIGURE;

EXEC sys.sp_configure 'max server memory (MB)', 5120;
RECONFIGURE;

```

Esto deja:

-   Docker: 6 GB max
    
-   SQL Server: 5 GB max
    
-   Margen para buffers internos y SO
    

### Verificar configuracion aplicada

```sql
SELECT
  name,
  value,
  value_in_use
FROM sys.configurations
WHERE name IN ('show advanced options', 'max server memory (MB)');

```

----------

## Conexion

Puedes conectarte desde **Azure Data Studio** o **SSMS** con:

-   Servidor: `localhost` (o `127.0.0.1`)
    
-   Puerto: `1433`
    
-   Autenticacion: SQL Login
    
-   Usuario: `sa`
    
-   Password: el valor de `SA_PASSWORD`
    
-   Encrypt: Optional / Trust server certificate
    

### Cadenas de conexion (ejemplos)

**ADO.NET (.NET)**

```
Server=localhost,1433;Database=master;User Id=sa;Password=P@ssw0rd_Str0ng!;TrustServerCertificate=True;

```

**ODBC**

```
Driver={ODBC Driver 18 for SQL Server};Server=localhost,1433;Database=master;Uid=sa;Pwd=P@ssw0rd_Str0ng!;TrustServerCertificate=Yes;

```

**JDBC**

```
jdbc:sqlserver://localhost:1433;databaseName=master;user=sa;password=P@ssw0rd_Str0ng!;trustServerCertificate=true;

```

----------

## Comandos utiles

### Ejecutar T-SQL dentro del contenedor

```bash
docker exec -it mssqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U sa -P 'P@ssw0rd_Str0ng!' -Q "SELECT @@VERSION;"

```

### Ver edicion y CU

```sql
SELECT
  SERVERPROPERTY('Edition') AS Edition,
  SERVERPROPERTY('ProductVersion') AS ProductVersion,
  SERVERPROPERTY('ProductUpdateLevel') AS ProductUpdateLevel;

```

----------

## Ciclo de vida

```bash
docker compose stop
docker compose start
docker compose down

```

----------

## Actualizar version de SQL Server

1.  Respaldar bases de datos
    
2.  Cambiar el tag de la imagen en `docker-compose.yml`
    
3.  Recrear el contenedor
    

```bash
docker compose down
docker compose pull
docker compose up -d

```

----------

## Seguridad

Este ejemplo expone `sa` y su contrasena en texto claro para fines didacticos.

Recomendaciones:

-   No compartas tu `.env`
    
-   Cambia la contrasena despues del primer arranque
    
-   Restringe el puerto 1433 por firewall
    
-   Usa usuarios de aplicacion en lugar de `sa`
    

----------

## Creditos

-   Imagen oficial de Microsoft SQL Server
    
-   Docker Hub: [https://hub.docker.com/r/microsoft/mssql-server](https://hub.docker.com/r/microsoft/mssql-server)
    
-   Edicion: **SQL Server 2022 Standard**
    
-   Cumulative Update: **CU21**
    
