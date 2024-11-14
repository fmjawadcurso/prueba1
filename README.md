# Documentación del Proyecto de Búsqueda de Películas y Series en PHP usando MVC

**Autor**: [Tu Nombre]  
**Fecha**: [Fecha de hoy]  
**Versión**: 1.0

---
## Introducción al Proyecto

El objetivo de este proyecto es construir una pequeña aplicación en PHP que permita buscar información de películas o series en una base de datos. El usuario puede buscar por distintos criterios, como **título**, **actor/actriz**, **año de estreno**, **género**, y **categoría**. Cuando el usuario hace una consulta, la aplicación muestra los resultados en una tabla y también permite exportarlos en formatos CSV o PDF.

Para organizar mejor el código y hacerlo más fácil de entender y mantener, he usado una arquitectura llamada **MVC (Modelo-Vista-Controlador)**. Esta estructura separa el código en tres partes:

1. **Modelo (Model)**: donde guardo la lógica para acceder a los datos en la base de datos.
2. **Vista (View)**: donde defino el diseño que verá el usuario, o sea, la interfaz.
3. **Controlador (Controller)**: que coordina todo, detecta las solicitudes y decide qué vista y qué datos debe cargar.

---

# Documentación del Proyecto - Pregunta 1

Este documento detalla los pasos realizados para la **importación de una base de datos**, la **creación de una vista** en MySQL Workbench, y la **implementación de una clase en PHP** para conectar y mostrar datos desde la vista en la base de datos.

## Parte 1: Importación de Tablas y Creación de una Vista en MySQL Workbench

### 1.1 Importar las Tablas en MySQL Workbench

1. Abre **MySQL Workbench** y selecciona la conexión a tu base de datos 

2. Asegúrate de que la base de datos llamada `movies` esté creada en tu servidor. Si no existe, puedes crearla ejecutando el siguiente comando en el editor SQL de Workbench:
   ```sql
   CREATE DATABASE movies;
3. Selecciona la base de datos movies en la lista de bases de datos.
4. Haz clic derecho sobre movies y selecciona Table Data Import Wizard.
5. En la ventana emergente, selecciona el archivo .sql que contiene el esquema y los datos de las tablas que deseas importar.
6. Sigue los pasos para completar la importación. Esto creará las tablas y, si el archivo .sql incluye datos, los insertará automáticamente en la base de datos.

## 1.2 Crear una Vista en MySQL Workbench
### Objetivo de la Vista
La vista se utiliza para combinar datos de dos tablas diferentes en una sola estructura de consulta y lo guarda en la base de datos. Esto permite obtener información de ambas tablas en una sola consulta, simplificando el acceso a los datos para el modelo de la aplicación.
### 1.2 Pasos para Crear la Vista
1. Una vez importadas las tablas, puedes crear una vista 
2. Abre un nuevo editor SQL en MySQL Workbench.
3. Ejecuta el siguiente comando para crear una vista que muestre el título de cada película y el nombre del director (ajustando los nombres de tablas y columnas si es necesario):
 ```sql
     CREATE VIEW MovieDirectors AS
    SELECT movies.title, directors.name
    FROM movies
    JOIN directors ON movies.director_id = directors.id;

    CREATE VIEW nombre_de_la_vista AS
   SELECT t1.columna1, t1.columna2, t2.columna3
   FROM tabla1 AS t1
   JOIN tabla2 AS t2 ON t1.id = t2.id;
```
Esta consulta selecciona columnas específicas de cada tabla y las une en función de un id común. Puedes modificar la consulta según los datos de tus tablas.
4. Ejecuta la consulta en MySQL Workbench para crear la vista.
5. Para verificar que la vista MovieDirectors se ha creado correctamente, ejecuta la siguiente consulta:
    ```sql
         SELECT * FROM MovieDirectors;

         SELECT * FROM nombre_de_la_vista;
         ```
    Esto debería mostrar los datos combinados de las dos tablas.

## Parte 2: Implementación de una Clase ModelBBDD en PHP para conectar con la base de datos y Mostrar Datos desde la Vista
 La clase __ModelBBDD__ está diseñada para gestionar la conexión a la base de datos y realizar consultas SQL de manera simplificada y reutilizable. Esta clase permite realizar operaciones como seleccionar todos los datos de una tabla o vista, filtrar datos por condiciones específicas y manejar conexiones de manera segura. A continuación, se explica cada sección de la clase paso a paso:
 ```php
    <?php
class ModelBBDD
{
    private static $db_host = 'localhost'; // Servidor de la base de datos
    private static $db_user = 'root';      // Usuario de la base de datos
    private static $db_pass = '';          // Contraseña de la base de datos
    private static $db_name = 'movies';    // Nombre de la base de datos
    private static $db_charset = 'utf8';   // Juego de caracteres de la conexión

    private $conn;    // Conexión a la base de datos
    private $query;   // Consulta SQL actual
    private $rows = array();  // Almacenamiento de resultados de consultas
    public $table;    // Nombre de la tabla o vista para la consulta

    // Constructor que acepta el nombre de la tabla o vista
    public function __construct($table) {
        $this->table = $table;
    }

    // Método para abrir la conexión a la base de datos
    private function db_open() {
        $this->conn = new mysqli(
            self::$db_host,
            self::$db_user,
            self::$db_pass,
            self::$db_name
        );
        $this->conn->set_charset(self::$db_charset);
    }

    // Método para cerrar la conexión a la base de datos
    private function db_close() {
        $this->conn->close();
    }

    // Método para ejecutar consultas de acción sin resultado (INSERT, UPDATE, DELETE)
    protected function set_query() {
        $this->db_open();
        $this->conn->query($this->query);
        $this->db_close();
    }

    // Método para ejecutar consultas con resultado (SELECT) y almacenar datos en $rows
    protected function get_query() {
        $this->db_open();
        $result = $this->conn->query($this->query);

        while ($this->rows[] = $result->fetch_assoc());
        $result->close();
        $this->db_close();

        return array_pop($this->rows);
    }

    // Método para buscar datos con coincidencia parcial utilizando LIKE
    public function getDatos($campo, $valor) {
        $valor = "'%" . addslashes($valor) . "%'";
        $this->query = "SELECT * FROM $this->table WHERE $campo LIKE $valor";
        
        $this->get_query();
        return !empty($this->rows) ? $this->rows : false;
    }

    // Método para obtener todos los datos de una tabla o vista, o filtrarlos por una condición específica
    public function get($campo = null, $valor = null) {
        $this->query = ($campo && $valor)
            ? "SELECT * FROM $this->table WHERE $campo = '" . $this->conn->real_escape_string($valor) . "'"
            : "SELECT * FROM $this->table";
        
        $this->get_query();
        return $this->rows;
    }
}
```
### Atributos:
  * host: Define la dirección del servidor de la base de datos, usualmente localhost en un entorno local.
  * usuario: El nombre de usuario para conectar a MySQL.
  * contrasena: La contraseña del usuario de la base de datos.
  * base_datos: El nombre de la base de datos.
### Explicación de los Métodos
1. __construct($table):
    * Constructor que recibe el nombre de la tabla o vista y lo asigna a la propiedad $table para que las consultas se dirijan a ella.
    ```php
    public function __construct($table) {
            $this->table = $table;
        }
        ```

2. **Método**db_open():
    * Método privado que abre la conexión a la base de datos usando mysqli y establece el conjunto de caracteres a utf8.
    ```php
    private function db_open() {
            $this->conn = new mysqli(
                self::$db_host,
                self::$db_user,
                self::$db_pass,
                self::$db_name
            );
            $this->conn->set_charset(self::$db_charset);
        }
    ```	

3. **Método**db_close():
    * Método privado que cierra la conexión a la base de datos. Es importante cerrar la conexión para liberar recursos cuando no se necesitan.
    
    ```php
    private function db_close() {
        $this->conn->close();}

    ```
4. **Método**set_query():
    * Ejecuta consultas que no devuelven resultados (como INSERT, UPDATE, DELETE).
    * Abre la conexión, ejecuta la consulta y luego cierra la conexión.
    ```php
        private function set_query() {
        $this->db_open();
        $this->conn->query($this->query);
        $this->db_close();}
        ```
 
 
5. **Método** get_query():
    * Este método ejecuta consultas que devuelven resultados, como SELECT.
    * Utiliza fetch_assoc() para almacenar los resultados en el array $rows.
    * Al final, cierra la conexión y devuelve los datos obtenidos.
 ```php
    private function get_query() {
     $this->db_open();
     $result = $this->conn->query($this->query);

     while ($this->rows[] = $result->fetch_assoc());
     $result->close();

     $this->db_close();

     return array_pop($this->rows);
 }
 ```
 6. **Método** getDatos($campo, $valor):
    * Este método generico busca datos en la tabla o vista utilizando LIKE para coincidencias parciales.
    * Toma el campo y el valor dinamicamente a buscar como parámetros, construye la consulta y utiliza get_query para ejecutarla.
    ```php	
    public function getDatos($campo, $valor) {
        $valor = "'%" . addslashes($valor) . "%'";
        $qry = "SELECT * FROM $this->table WHERE $campo LIKE $valor";
        $this->query = $qry;

        $this->get_query();
        return !empty($this->rows) ? $this->rows : false;
    }
   ```
 7. **Método** get($campo = null, $valor = null):
    * Este método generico genera una consulta SELECT para obtener todos los datos de una tabla o vista, o genera una consulta SELECT para obtener un dato  por una condición específica.
    * Si $campo y $valor no estan **null**, se filtra por esa condición; si no, se obtiene todo.
    ```php
    public function get($campo = null, $valor = null) {  
            $this->query = ($campo !== null && $valor !== null)
                ?"SELECT * FROM $this->table WHERE $campo = '$valor'"
                :"SELECT * FROM $this->table";
            
            $this->get_query();

            $num_rows = count($this->rows);

            $data = array();

            foreach ($this->rows as $key => $value) {
                array_push($data, $value);
            }
        
            return $data;
        }
    ```
# Documentación del Proyecto (Preguna 2)

## 1. Introducción
    Este proyecto  utiliza el patrón de diseño MVC (Modelo-Vista-Controlador) para gestionar una aplicación en PHP que se conecta a una base de datos, crea una vista en MySQL y muestra los resultados al usuario a través de una interfaz web.

    (permita buscar información de películas o series en una base de datos. El usuario puede buscar por distintos criterios, como título, actor/actriz, año de estreno, género, y categoría. Cuando el usuario hace una consulta, la aplicación muestra los resultados en una tabla)

## 2. Objetivo
    El objetivo de este proyecto es construir una pequeña aplicación en PHP basado en  arquitectura MVC que permita consultar una vista en la base de datos y presentar los resultados de forma organizada en una página web.
    Esta estructura separa el código en tres partes:
        * Modelo (Model): donde guardo la lógica para acceder a los datos en la base de datos.
        * Vista (View): donde defino el diseño que verá el usuario, o sea, la interfaz.
        * Controlador (Controller): actúa como intermediario entre la vista y el modelo. Llama al modelo para obtener datos y los envía a la vista correspondiente.

## 3. Estructura de Archivos
El proyecto está dividido en carpetas según el patrón MVC:

/assets
    ├── css
    ├── img
    └── js
/controllers
    ├── Autoload.php
    ├── Router.php
    ├── PreguntasController.php
    └── ViewController.php
/models
    └── Model.php
/views
    ├── header.php
    ├── footer.php
    ├── preguntas.php
    └── respuesta.php
index.php

## 4. Explicación del Código Paso a Paso
Voy a explicarte cada componente del proyecto:
### 4.1. Modelo (Model): ModelBBDD.php
Este archivo contiene la clase ModelBBDD, que se encarga de gestionar la conexión a la base de datos y ejecutar consultas SQL.tiene métodos para abrir y cerrar la conexión, ejecutar consultas y obtener resultados.
lo hemos explicado paso paso en la documentacion de la pregunta 1.

### 4.2. Vista (View):, footer.php, preguntas.php, respuesta.php
Las Vistas son los archivos donde está el HTML, css y js que el usuario ve en pantalla.
  * header.php: carga el encabezado de la página para todo la página. contiene  los enlaces de css y cdn segun nuestra necesidad en el  codigo (jequery, bootstrap, datatable, etc)
  * footer.php: carga el pie de página para todo la página. 
  * preguntas.php: carga el formulario de preguntas (las consultas) y esta hecho con html y utilizando el checkbox    para seleccionar las preguntas que se van a realizar y un campo de texto para introducir el valor de la pregunta.

     -En este archivo, el usuario elige cómo quiere buscar (por título, actor, año, etc.) y escribe el término de búsqueda en un campo de texto. Cuando hace clic en "Consultar", el formulario envía los datos al controlador para procesarlos. esta formulario tiene el metedo post para enviar los datos al controlador.
    ```html
    <div class="container">
        <h2>Consulta de Películas</h2>
        <form method="post" action="">
            <label>Elija una consulta:</label><br>
            <input type="radio" name="consultas" value="title"> Título<br>
            <input type="radio" name="consultas" value="actors"> Actor<br>
            <input type="radio" name="consultas" value="genres"> Género<br>
            <input type="radio" name="consultas" value="premiere"> Año<br>
            <input type="radio" name="consultas" value="category"> Categoría<br>
            <input type="hidden" name="r" value="resultados">
            <input type="text" name="palabra" required placeholder="Filtro">
            <button type="submit">Consultar</button>
        </form>
    </div>
    ```
* respuesta.php: carga el resultado de la consulta realizada en el formulario de preguntas.
    ### 1. Instanciación del Controlador
    Crea una nueva instancia de PreguntasController, que se encargará de realizar las consultas en la tabla **movieseries** de la base de datos.
    ```php
    $pregunta_controller = new PreguntasController('movieseries');
    ```
    ### 2. Verificación y Asignación de Campos de Búsqueda
    Verifica si el formulario fue enviado correctamente. y si La variable $_POST['r'] es igual 'respuesta', y  existir los campos consultas y palabra en el formulario.
    ```php
        if ($_POST['r'] == 'respuesta' && isset($_POST['consultas']) && isset($_POST['palabra'])) {}
        ```
    ### 3. Asignación de Campo y Valor
    una vez entramos en la condicion de la arriba , aqui determinado el ($campo) y el valor ($valor) en base a la opción seleccionada en el formulario (title, actors, premiere, genres, o category).
    ```php
    if ($_POST['consultas'] == 'title') {
        $campo = 'title';
        $valor = $_POST['palabra'];
    } elseif ($_POST['consultas'] == 'actors') {
        $campo = 'actors';
        $valor = $_POST['palabra'];
    } elseif ($_POST['consultas'] == 'premiere') {
        $campo = 'premiere';
        $valor = $_POST['palabra'];
    } elseif ($_POST['consultas'] == 'genres') {
        $campo = 'genres';
        $valor = $_POST['palabra'];
    } elseif ($_POST['consultas'] == 'category') {
        $campo = 'category';
        $valor = $_POST['palabra'];
    }
    ```	
    ### 4. Realizar la Consulta y Manejar Resultados Vacíos
    Aquí se Llama al método getDatos del controlador para obtener los datos que coincidan con el campo y valor especificados. Si no se encuentran resultados, se muestra un mensaje de alerta en la interfaz y se recarga la página de preguntas  para permitir otra consulta.

    ### 5. Creación de la Tabla HTML para Mostrar Resultados
    Crea la estructura HTML para una tabla, con encabezados específicos, y hemos utilizado aqui la datatable para mostrar los resultados de la consulta.
    ### 6. Rellenar la Tabla con los Resultados de la Consulta
    ### 3.1 Conector de la Base de Datos (ConectorBD.php)
    Esta clase se encarga de gestionar la conexión con la base de datos MySQL.

### 4.3. Controlador (Controller): 
actúa como un intermediario. Recibe las solicitudes del usuario (como hacer clic en "buscar"), consulta al modelo si necesita datos y luego selecciona qué vista mostrar.En mi proyecto hay 4 controladores:
* **Autoload - Autoload.php**: 
    * Este archivo contiene una clase llamada Autoload que se encarga de cargar automáticamente las clases de modelo y controlador cuando se necesitan, sin tener que escribir require en todas partes.Utilizando la funcion __spl_autoload_register__ 
    
     ```php
  
    class Autoload {
        public function __construct() {
            spl_autoload_register(function($class_name) {
                $models_path = './models/' . $class_name . '.php';
                $controllers_path = './controllers/' . $class_name . '.php';
                if(file_exists($models_path)) require_once($models_path);	
                if(file_exists($controllers_path)) require_once($controllers_path);
            });
        }
    }
    ```
    >[!NOTE]
    Como requerimiento, los nombres de los archivos deben ser iguales
     a los nombres de las clases que en ellos se encuentran.

    * **Router.php:**
    Este archivo determina qué vista debe cargarse en función de la URL. El Router maneja rutas específicas y delega la carga de vistas al controlador ViewController.
    revisa si el usuario ha enviado una solicitud de búsqueda. Si es así, carga la vista respuesta.php. Si no, carga la vista del formulario (preguntas.php).
    ```php
    class Router {
    public function __construct() {
        $controller = new ViewController();
        if (!isset($_POST['r'])) $controller->load_view('preguntas');
        else if ($_POST['r'] == 'resultados') $controller->load_view('respuesta');
    }
  }
  ```
  * **ViewController.php**
    Este archivo contiene la clase ViewController, que se encarga de cargar las vistas.
    ```php
    class ViewController {
	private static $view_path = './views/'; // Ruta donde se encuentran las vistas

	public function load_view($view) { //aqui pasamos el nombre de la vista como parametro para este funcion
		require_once( self::$view_path . 'header.php' );  // Carga el encabezado
		require_once( self::$view_path . $view . '.php' ); // Carga la vista solicitada , este vista controla el rutador(Router)
		require_once( self::$view_path . 'footer.php' );  // Carga el pie de página
	}
  }
  ```
  * **PreguntasController.php**
  Este archivo contiene la clase PreguntasController, que se encarga de manejar la lógica de la vista preguntas.php.
  Este clase se instancia la clase del ModelBBDD prara puede llamar los metedos del modelo para obtener los datos.
  ```php
  class PreguntasController{
    private $model;
    private $table;

    public function __construct($table) {
        $this->table = $table;
		// Instancia el modelo correspondiente a la tabla
		$this->model = new ModelBBDD($this->table);
	}
    public function getDatos($campo, $valor){
        return $this->model->getDatos($campo, $valor);
    }
    public function get($campo = null, $valor = null) {
        return $this->model->get($campo, $valor);
    }
  }
  ```
### 4.4 index.php
Este archivo es el punto de entrada de la aplicación. Inicializa la aplicación y carga el enrutador y Autoload.
```php
require_once('./controllers/Autoload.php'); 
$autoload = new Autoload();  
$app = new Router();        
 ``` 
### Requisitos Técnicos
* PHP 7.4+
* MySQL 5.7+
* Librerías:
    + jQuery
    + Bootstrap
    + DataTables
