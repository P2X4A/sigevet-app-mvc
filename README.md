# SIGEVET

Aplicación de consulta para el proyecto de Bases de Datos 1. Usa Java 21,
Spring Boot 3.5.14, Thymeleaf y MySQL. Escucha en el puerto 5000 y solo lee datos.

## Contexto y créditos

SIGEVET es el proyecto final de **Bases de Datos 1** (Universidad El Bosque, Ingeniería de Sistemas), hecho en equipo por
Juan David Polo Herrera, Sebastián Castellanos Parra, Juan Miguel Ramos Sevilla y Sergio Alejandro Parra Rivera.
El repositorio completo del equipo (base de datos, documentos, máquina virtual, API REST y frontend Angular) está en
[Estarlan/proyecto-bd1-sigevet](https://github.com/Estarlan/proyecto-bd1-sigevet).

Este repositorio conserva una **iteración anterior de la aplicación**, con las páginas generadas en el servidor con
Thymeleaf. En la versión final del equipo esa interfaz se reemplazó por una API REST y un frontend en Angular.

## Compilar y ejecutar

Desde la raíz del repositorio, con un JDK 21 instalado:

```sh
./mvnw package
java -jar target/sigevet.jar
```

Si el wrapper no tiene permiso de ejecución, usa `sh ./mvnw package`.
Abre `http://localhost:5000`. El jar ya incluye el servidor web. Antes hay que tener la base `sigevet` cargada
(los scripts SQL están en la carpeta [`database`](https://github.com/Estarlan/proyecto-bd1-sigevet/tree/main/database) del repositorio del equipo).

## Variables de entorno

| Variable | Valor predeterminado |
| --- | --- |
| `SIGEVET_DB_URL` | `jdbc:mysql://localhost:3306/sigevet` |
| `SIGEVET_DB_USER` | `vet_app` |
| `SIGEVET_DB_PASSWORD` | Cadena vacía |
| `SIGEVET_ALGEBRA` | `../database/03_varios/01_consultas_algebra_relacional.sql` |

Exporta estas variables antes de ejecutar el jar. La clave de la base va solo en
`SIGEVET_DB_PASSWORD`; no se escribe en ningún archivo. `vet_app` debe tener solo
permiso de lectura. Este repositorio no incluye la carpeta `database`: descarga `database/03_varios/01_consultas_algebra_relacional.sql` del repositorio del equipo y apunta `SIGEVET_ALGEBRA` a ese archivo.
Para correr el jar desde otra carpeta, configura una ruta absoluta en
`SIGEVET_ALGEBRA`. El archivo se lee línea por línea cuando se abre álgebra.

## Estructura

Todo el código está en `src/main/java/co/edu/unbosque/sigevet`:

- `SigevetApplication`: inicia Spring Boot.
- `connection`: `ConexionBD` lee las propiedades y abre conexiones con `DriverManager`.
- `model`: objetos con atributos privados, constructor vacío, getters y setters (y los enums de estados).
- `dao`: consultas SQL escritas a mano, parámetros con `PreparedStatement` y lectura de `ResultSet`.
- `service`: clases `...Service`; reúnen los datos de varios DAO y aplican la lógica de cada página.
- `controller`: clases `...Controller`; reciben las rutas, piden los datos al servicio y los entregan al `Model`.
- `dto`: clases `...DTO`; agrupan los datos que necesita una página (por ejemplo `ClienteDetalleDTO`).
- `exception`: `RecursoNoEncontradoException`, que Spring convierte en un 404.
- `util`: `TextoUtil`, utilidades de texto para los filtros de los formularios.
- `algebra`: lector de los 55 ejercicios agrupados del archivo SQL.

El flujo de una petición es: navegador → `controller` → `service` → `dao` → MySQL, y de vuelta hasta la plantilla
Thymeleaf (la vista).

Las vistas están en `src/main/resources/templates`. El menú y el pie son
fragmentos compartidos. El CSS original está en `src/main/resources/static`.

## JDBC sin ORM

No se usa ORM porque el objetivo de la materia es entender las tablas, las
uniones, los filtros y los cálculos SQL. Cada DAO abre su conexión, prepara la
consulta, crea los objetos con setters y cierra los recursos con
`try-with-resources`. Si MySQL falla, imprime el error y lanza una excepción
con un mensaje en español. Los totales y descuentos se calculan en SQL.

## Herencia y polimorfismo

`Persona` es abstracta. Tiene los datos comunes, `getNombreCompleto()` y el
método abstracto `getDescripcion()`. `Cliente` y `Empleado` heredan de ella.
`Veterinario` y `Auxiliar` heredan de `Empleado`. Cada clase concreta sobrescribe
`getDescripcion()` con `@Override`.

En `PersonalDAO`, el campo `tipo_empleado` decide si se crea un `Veterinario`,
un `Auxiliar` o un `Empleado`. Todos se guardan en un `ArrayList<Empleado>`.
La vista `personal.html` llama a `fila.getDescripcion()`: Java ejecuta la
versión del objeto concreto aunque la lista sea de empleados. Esa llamada
muestra el polimorfismo. Las especialidades se cargan con una segunda consulta.

## Páginas

- `/`: conteos de las 22 tablas, ingresos pagados, citas y stock bajo.
- `/clientes?buscar=` y `/clientes/{id}`: búsqueda de hasta 100 clientes y detalle.
- `/mascotas/{id}`: mascota, historia clínica, consultas y prescripciones.
- `/agenda?sede=&estado=`, `/inventario?sede=` y `/personal?sede=`.
- `/facturas?sede=&estado=` y `/facturas/{id}`: facturación y líneas.
- `/algebra` y `/algebra/{numero}`: enunciado, SQL y resultado genérico.

Un registro inexistente devuelve 404. Los fallos de consulta muestran una página
500 sencilla sin publicar credenciales ni detalles internos.

## Instalación en la VM

Los scripts de la máquina virtual (`vm/`) no están en este repositorio; están en el
[repositorio del equipo](https://github.com/Estarlan/proyecto-bd1-sigevet).
