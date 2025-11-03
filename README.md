# SQL Server 2022 — Developer Edition (Docker)

Este repositorio levanta una instancia de **SQL Server 2022 Developer** en Docker usando un `docker-compose.yml` **simple** (sin `.env`).

> ⚠️ **Aviso de licencia**: La edición **Developer** es gratuita **solo para desarrollo, pruebas y demos**. **No** se debe usar en **producción**.

----------

## Requisitos

-   Docker Engine + Docker Compose (v2 o superior).
    
-   Puerto **1433** libre en tu host.
    
-   RAM disponible (recomendado: ≥ 2 GB libres).
    

----------

## Archivos

**`docker-compose.yml`**

```yaml
services: # Servicios del contenedor
  sqlserver: # Servicio de SQL Server
    image: mcr.microsoft.com/mssql/server:2022-latest # Imagen de SQL Server 2022
    container_name: mssqlserver-developer-edition # Nombre del contenedor
    ports: # Puertos expuestos
      - "1433:1433" # Puerto para SQL Server
    environment: # Variables de entorno
      - ACCEPT_EULA=Y # Acepta condiciones de uso
      - MSSQL_PID=Developer # Usa la edición Developer
      - SA_PASSWORD=P@ssw0rd_Str0ng!  # cambia si quieres
      # Nota: la contraseña de sa debe cumplir complejidad (mín. 8, mayúscula, minúscula, dígito y símbolo).
      # Si usas una que no cumple, el contenedor se apaga.
    volumes: # Volúmenes para persistencia de datos
      - sql-data:/var/opt/mssql # Volumen para datos de SQL Server
    restart: always # Reinicia el contenedor automáticamente

volumes: # Volúmenes para persistencia de datos
  sql-data: # Volumen para datos de SQL Server

```

----------

## Puesta en marcha

```bash
# 1) Levantar en segundo plano
docker compose up -d

# 2) Ver estado
docker compose ps

# 3) Ver logs (hasta ver "Server is listening on ... 1433")
docker logs -f mssqlserver-developer-edition

```

----------

## Conexión

Puedes conectarte desde **Azure Data Studio** o **SSMS** con:

-   **Servidor**: `localhost`
    
-   **Autenticación**: SQL Login
    
-   **Usuario**: `sa`
    
-   **Password**: `P@ssw0rd_Str0ng!` (o la que hayas establecido)
    
-   **Encrypt**: Optional / Trust server certificate ✅
    

**Cadenas de conexión (ejemplos)**

-   **ADO.NET (.NET)**
    
    ```
    Server=localhost,1433;Database=master;User Id=sa;Password=P@ssw0rd_Str0ng!;TrustServerCertificate=True;
    
    ```
    
-   **ODBC**
    
    ```
    Driver={ODBC Driver 18 for SQL Server};Server=localhost,1433;Database=master;Uid=sa;Pwd=P@ssw0rd_Str0ng!;TrustServerCertificate=Yes;
    
    ```
    
-   **JDBC**
    
    ```
    jdbc:sqlserver://localhost:1433;databaseName=master;user=sa;password=P@ssw0rd_Str0ng!;trustServerCertificate=true;
    
    ```
    

> Si estás en Windows y no conecta, revisa el **Firewall** para permitir el puerto 1433 entrante/local.

----------

## Comandos útiles

### Ejecutar T-SQL dentro del contenedor

```bash
docker exec -it mssqlserver-developer-edition /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U sa -P 'P@ssw0rd_Str0ng!' -Q "SELECT @@VERSION;"

```

### Crear una base de datos de ejemplo

```bash
docker exec -it mssqlserver-developer-edition /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U sa -P 'P@ssw0rd_Str0ng!' \
  -Q "IF DB_ID('Demo') IS NULL CREATE DATABASE Demo;"

```

### Cambiar la contraseña de `sa`

```sql
-- Conéctate y ejecuta:
ALTER LOGIN sa WITH PASSWORD = 'TuNuev@Clave2025!';

```

----------

## Persistencia de datos

Los datos se guardan en el volumen **`sql-data`** (mapea `/var/opt/mssql`).  
Para listar, respaldar o eliminar:

```bash
# Ver volúmenes
docker volume ls

# Inspeccionar
docker volume inspect sql-data

# (Precaución) Eliminar volumen y datos
docker compose down
docker volume rm sql-data

```

> Hacer `down` **no** borra el volumen a menos que lo elimines explícitamente.

----------

## Ciclo de vida

```bash
# Detener
docker compose stop

# Iniciar nuevamente
docker compose start

# Apagar y quitar el contenedor (datos permanecen en el volumen)
docker compose down

```

----------

## Actualizar a una nueva imagen

1.  **Respaldar** tus bases (buena práctica).
    
2.  Traer la última imagen:
    
    ```bash
    docker pull mcr.microsoft.com/mssql/server:2022-latest
    
    ```
    
3.  Recrear manteniendo datos:
    
    ```bash
    docker compose down
    docker compose up -d
    
    ```
    

----------

## Solución de problemas

-   **El contenedor se apaga al iniciar**  
    Revisa `docker logs mssqlserver-developer-edition`. Usualmente es **contraseña inválida** (no cumple complejidad).
    
-   **No conecta desde el host**
    
    -   Asegúrate de que el puerto **1433** no esté ocupado.
        
    -   Revisa Firewall.
        
    -   Prueba `localhost` vs `127.0.0.1`.
        
-   **Permisos en archivos**  
    Si cambias el mapeo de volúmenes a una carpeta del host, procura que Docker tenga permisos de lectura/escritura.
    

----------

## Seguridad

Este ejemplo expone `sa` y su contraseña **en texto claro** para fines **didácticos**.  
Para proyectos reales:

-   Usa variables de entorno o secretos (Compose/Swarm/K8s).
    
-   Cambia la contraseña después del primer arranque.
    
-   Limita el puerto al entorno que corresponda (redes de Docker, firewalls, etc.).
    

----------

## Captura de ejemplo (Visual Studio Code)

> Parámetros básicos para conectarte a `localhost` con usuario `sa`.

Usando la extencion de [SQL Server (mssql)](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql)

![Azure Data Studio connect](./Visual%20Studio%20Code%20Connect.png)


----------

## Créditos

-   Imagen oficial: `mcr.microsoft.com/mssql/server:2022-latest`
    
-   Edición: **Developer** (solo dev/test/demos)

-   Extencion [SQL Server (mssql)](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql)
