# SQL Server 2022 Developer Edition (Docker + Compose)

Repositorio para levantar **SQL Server 2022 Developer** en Docker usando **docker-compose** con **.env** y **persistencia local** en la carpeta del proyecto.

> Aviso de licencia: la edicion **Developer** es gratuita **solo para desarrollo, pruebas y demos**. **No** usar en **produccion**.

---

## Requisitos

- Docker Engine + Docker Compose (v2 o superior)
- Puerto **1433** libre en el host
- RAM disponible (recomendado: 2 GB libres o mas)

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

```
ACCEPT_EULA=Y
MSSQL_PID=Developer
SA_PASSWORD=P@ssw0rd_Str0ng!
```

Reglas para `SA_PASSWORD`:
- Minimo 8 caracteres
- Debe incluir mayusculas, minusculas, digitos y simbolos
- Si no cumple, el contenedor se apaga

---

## Puesta en marcha

```bash
# 1) (Opcional) copiar plantilla
# cp .env.example .env

# 2) levantar en segundo plano
docker compose up -d

# 3) ver estado
docker compose ps

# 4) ver logs (hasta ver "Server is listening on ... 1433")
docker logs -f mssqlserver-developer-edition
```

---

## Persistencia de datos

Los datos se guardan en `./sql-data` (relativo al `docker-compose.yml`) y se mapean a `/var/opt/mssql` dentro del contenedor.

- Para borrar TODO (contenedor + datos):

```bash
docker compose down
# eliminar carpeta de datos (cuidado)
# rmdir /s /q sql-data
```

> `docker compose down` **no** elimina los datos si la carpeta `sql-data` permanece.

---

## Conexion

Puedes conectarte desde **Azure Data Studio** o **SSMS** con:

- Servidor: `localhost` (o `127.0.0.1`)
- Puerto: `1433`
- Autenticacion: SQL Login
- Usuario: `sa`
- Password: el valor de `SA_PASSWORD`
- Encrypt: Optional / Trust server certificate

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

---

## Comandos utiles

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

### Cambiar la contrasena de `sa`

```sql
ALTER LOGIN sa WITH PASSWORD = 'TuNuev@Clave2025!';
```

---

## Ciclo de vida

```bash
# detener
docker compose stop

# iniciar nuevamente
docker compose start

# apagar y quitar contenedor (datos permanecen)
docker compose down
```

---

## Actualizar imagen

1) Respaldar tus bases (buena practica)
2) Traer la ultima imagen
3) Recrear el contenedor

```bash
docker pull mcr.microsoft.com/mssql/server:2022-latest
docker compose down
docker compose up -d
```

---

## Solucion de problemas

- **El contenedor se apaga al iniciar**: revisa `docker logs mssqlserver-developer-edition`. Casi siempre es contrasena invalida.
- **No conecta desde el host**: revisa que el puerto 1433 no este ocupado y que el firewall permita la entrada.
- **Permisos en archivos**: asegurate de que Docker tenga permisos de lectura/escritura en `./sql-data`.
  - En Linux, si el contenedor corre como usuario `mssql` (UID 10001) y ves errores de permisos:
    ```bash
    sudo mkdir -p sql-data
    sudo chown -R 10001:0 sql-data
    sudo chmod -R g+rwx sql-data
    ```

---

## Seguridad

Este ejemplo expone `sa` y su contrasena en texto claro para fines didacticos.

Recomendaciones:
- No compartas tu `.env`
- Cambia la contrasena despues del primer arranque
- Restringe el puerto segun el entorno (firewall/redes de Docker)

---

## Captura de ejemplo (Visual Studio Code)

Usando la extension de [SQL Server (mssql)](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql)

![Visual Studio Code connect](./Visual%20Studio%20Code%20Connect.png)

---

## Creditos

- Imagen oficial: `mcr.microsoft.com/mssql/server:2022-latest`
- Edicion: **Developer** (solo dev/test/demos)
- Extension [SQL Server (mssql)](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql)
