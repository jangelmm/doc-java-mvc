El MVC o Modelo-Visto-Controlador es un patrón de arquitectura de software que, utilizando 3 componentes separa la lógica de la aplicación de la lógica de la vista en una aplicación.

>[! info]
>Hasta el momento nuestras aplicaciones solo las hemos hecho como si fueran solo uno (un solo paquete) la interfaz gráfica, la lógica, eventos, etc... todo se colocaba en un solo paquete, en proyectos extensos es complicado entenderlo y dar mantenimiento, esto al estar mezclado. Para esto se creo este patrón.

 Es una arquitectura importante puesto que se utiliza tanto en componentes gráficos básicos hasta sistemas empresariales.

>[! note]
>A pesar de que el modelo se llama *Modelo - Vista - Controlador*
>Te recomiendo leer en el orden de *Vista - Modelo - Controlador*.

Recuerda que tienes que crear cuatro paquetes en NetBeans
- modelo
- vista
- controlador
- predeterminado (creado por el IDE)

## `ris:Database2` Modelo
Se encarga de los datos, generalmente consultando la base de datos.
Actualizaciones, consultas, búsquedas, etc.

- `Conexion.java`: En cargado de establecer la conexión con la Base de Datos.
- `Entidad.java`: Clase de cualquier Objeto (`Persona`, `Anmal`, etc.).
- `SQLEntidad.java`: Encargado de manipular la Base de Datos
- `Utilidad.java`: Cualquier programa extra que haga algo (encriptar, crear PDF, Excel, etc.)

>[! note]
>Pueden haber tantos `ClaseObjetos`, `SQLObjetos` y `Programa` según tus necesidades.
>En entornos empresariales, es recomendable utilizar un pool de conexiones en lugar de crear nuevas conexiones directamente.

### `Conexion.java`

La clase que siempre hemos usado para conectarnos la base de datos.

```Java impo:8
package modelo;

import java.sql.Connection;
import java.sql.DriverManager;

public class Conexion {
    //Variables de conexión                                       /******/
    public static final String URL = "jdbc:mysql://localhost:3306/base_datos?useTimeZone=true&serverTimezone=UTC&autoReconnect=true&useSSL=false";
    public static final String usuario = "root";
    public static final String contrasena = "password";
    
    //Métodos de conexión
    public Connection getConnection(){
        Connection conexion = null;
        
        try{
            conexion = DriverManager.getConnection(URL, usuario, contrasena);
        }catch(Exception ex){
            System.out.println("Error, " + ex);
        }
        
        return conexion;
    }
}
```

### `Entidad.java`

Puede ser cualquier clase que necesites.

```Java
package modelo;
public class Usuario {
    private int id;
    private String nombreUsuario;
    private String contrasena;
    private String nombreReal;
    private String correo;
    private String ultimaSesion;
    private int idTipoUsuario;
    private String nombreRol;

    //Getters y Setters
}
```

### `SQLEntidad.java`
Se usa para realizar modificaciones a la Base de Datos usando la `Entidad`, para insertar o recuperar usa los `Getters y Setters` de la clase, no uses los datos de la vista.

```Java
package modelo;

import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Connection;

public class SQLPersona extends Conexion{
    //Utilizamos variables de 
    PreparedStatement ps;  //realizar consultas
    ResultSet rs;  //Obtener datos
    
    //Método para insertar registros
    public boolean insertar(Persona persona){
		//...
    }
    public boolean buscar(Persona persona){
        //...
    }
    public boolean modificar(Persona persona){
        //...
    }
    public boolean eliminar(Persona persona){
        //...
    }
}

```

### `Utilidad.java`

Cualquier programa genérico que necesites durante la manipulación de la base de datos o el objeto.

```Java
public class Programa{
	//Programa que se encarga de hacer actividades como
	//Encriptar
	//Exportar PDF
	//Crear Excel
	//Etc
}
```

## `rir:PictureInPictureExit` Vista
Son la representación visual de los datos, todo lo que tenga que ver con la interfaz gráfica. Ni el modelo ni el controlador se preocupan de cómo se verán los datos, esa responsabilidad es únicamente de la vista.

- `JFrame`
- `JDialog`
- `JPanel`

>[! warning]
>Recuerda que una opción es que los atributos sean `Public`, aunque se recomienda ser privados o protegidos, y manejarse a través de métodos de acceso (`getters` y `setters`).

![](Adjuntos/Pasted%20image%2020240716174634.png)

## `fas:Gamepad` Controlador
Se encarga de controlar, recibe las órdenes del usuario a través de los eventos de la interfaz gráfica y se encarga de solicitar los datos al modelo y de comunicárselos a la vista.

> [! note]
> Se encarga de recibir las ordenes del usuario, cuando tenemos la interfaz tiene botones, los eventos recibían las ordenes, aquellos eventos estarán dentro del controlador.

- `ControladorClase`: Se encarga de unir la Base de Datos y parte lógica (modelo) con la Vista.

Va a requerir tener acceso a ambos elementos, para esto creamos como atributos los objetos de los paquetes anteriores.

```Java
package controlador;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import modelo.SQLPersona;
import modelo.Persona;
import vista.VistaPersona;
import java.sql.Date;
import javax.swing.JOptionPane;

public class ControladorPersona implements ActionListener{
    //Elementos a manipular
    private VistaPersona vista;
    private Persona persona;
    private SQLPersona modelo;
    
    //Constructor
    public ControladorPersona(VistaPersona vista, Persona persona, SQLPersona modelo) {
        this.vista = vista;
        this.persona = persona;
        this.modelo = modelo;
        
        //Indicar los elementos con ActionListener
        vista.btnNombre.addActionListener(this);
    }
    
    //Iniciar
    public void iniciar(){
        vista.setTitle("CRUD MVC");
        vista.setLocationRelativeTo(null);
        vista.txtId.setVisible(false);  //No se vea la caja id
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        //Botón Insertar
        if(e.getSource() == vista.btnInsertar){
            //Obtener los datos
            persona.setAtributo(vista.txtAtributo.getText());
            //...
            
            //Insertar los elementos
            if(modelo.insertar(persona)){
	            //...
                limpiarCajas();
            }
            else{
                //...
                limpiarCajas();
            }
            
        }
        //Botón Limpiar
        if(e.getSource() == vista.btnLimpiar){
            limpiarCajas();
        }
        //Botón Buscar
        if(e.getSource() == vista.btnBuscar){
            //Obtenemos la clave a buscar
            persona.setClave(vista.txtBuscar.getText());
            
            if(modelo.buscar(persona)){  //Se ejecuta el método
                //Agregamos los elementos a la GUI
                vista.txtAtributo.setText(persona.getAtributo());
                //...
            }
            else{
                //...
                limpiarCajas();
            }
        }
        //Botón Modificar
        if(e.getSource() == vista.btnModificar){
            //Obtener los datos
            persona.setAtributo(vista.txtAtributo.getText());
            //....
            
            if(modelo.modificar(persona)){  //Estatus
        
                limpiarCajas();
            }
            else{
                JOptionPane.showMessageDialog(null, "No se pudo modificar el registro", "Estatus", JOptionPane.ERROR_MESSAGE);
                limpiarCajas();
            }
        }
        //Botón Eliminar
        if(e.getSource() == vista.btnEliminar){
            //Obtenemos el ID
            persona.setIdPersona(Integer.parseInt(vista.txtId.getText()));
            
            //Realizamos le eliminación
            if(modelo.eliminar(persona)){  //Estatus
                JOptionPane.showMessageDialog(null, "Registro eliminado correctamente", "Estatus", JOptionPane.INFORMATION_MESSAGE);
                limpiarCajas();
            }
            else{
                JOptionPane.showMessageDialog(null, "No se pudo eliminar el registro", "Estatus", JOptionPane.ERROR_MESSAGE);
                limpiarCajas();
            }
        }
    } 
    
    public void limpiarCajas(){
        //Volver al estado inicial
        vista.txtId.setText(null);
        vista.txtBuscar.setText(null);
        vista.txtClave.setText(null);
        vista.txtNombre.setText(null);
        vista.txtDomicilio.setText(null);
        vista.txtCelular.setText(null);
        vista.txtCorreo.setText(null);
        vista.txtNacimiento.setText(null);
        vista.cboGenero.setSelectedIndex(0);
    }
}

```

>[! note]
>Considera separar la lógica en métodos privados si tienes operaciones complejas dentro de `actionPerformed`.

## `ris:Play` Run
Es la clase que crea por defecto  NetBeans, se encarga de arrancar tu programa, regularmente tiene el siguiente nombre
Si el nombre es *Proyecto-MVC-Java*
Se nombra

```
ProyectoMVCJava.java
``` 

**Ejemplo**

```Java
package ats.java.bvd.mvc.crud;

import controlador.ControladorPersona;
import modelo.ConsultasPersonas;
import modelo.Persona;
import vista.VistaPersona;

public class AtsJavaBvdMvcCrud {

    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        //Objetos utilizar de todas las clases
        VistaPersona vista = new VistaPersona();
        Persona persona = new Persona();
        ConsultasPersonas modelo = new ConsultasPersonas();

		//Controlador de esa clase
        ControladorPersona controlador = new ControladorPersona(vista, persona, modelo);
        
        //Iniciamos
        controlador.iniciar();
        vista.setVisible(true);
    }
}
```

### `ris:Play` Ejecución del Programa en muchas clases

En NetBeans (o cualquier otro IDE), la clase principal (`main`) es la encargada de iniciar la ejecución del programa. Esta clase suele tener un nombre descriptivo del proyecto, como `ProyectoMVCJava`. En proyectos grandes con múltiples vistas y controladores, es importante centralizar la lógica de inicio y proporcionar un punto de entrada claro para la aplicación.

#### Estructura básica de la clase principal

La estructura básica para una aplicación más grande podría ser algo como lo siguiente:

```java
package ats.java.bvd.mvc.crud;

import controlador.ControladorPersona;
import controlador.ControladorOtraEntidad;
import modelo.ConsultasPersonas;
import modelo.ConsultasOtraEntidad;
import modelo.Persona;
import modelo.OtraEntidad;
import vista.VistaPersona;
import vista.VistaOtraEntidad;

public class ProyectoMVCJava {

    public static void main(String[] args) {
        // Crear las vistas
        VistaPersona vistaPersona = new VistaPersona();
        VistaOtraEntidad vistaOtraEntidad = new VistaOtraEntidad();
        
        // Crear los modelos
        Persona persona = new Persona();
        OtraEntidad otraEntidad = new OtraEntidad();
        ConsultasPersonas modeloPersona = new ConsultasPersonas();
        ConsultasOtraEntidad modeloOtraEntidad = new ConsultasOtraEntidad();
        
        // Crear los controladores
        ControladorPersona controladorPersona = new ControladorPersona(vistaPersona, persona, modeloPersona);
        ControladorOtraEntidad controladorOtraEntidad = new ControladorOtraEntidad(vistaOtraEntidad, otraEntidad, modeloOtraEntidad);
        
        // Iniciar las vistas principales
        controladorPersona.iniciar();
        controladorOtraEntidad.iniciar();
        
        // Mostrar la vista principal de la aplicación
        vistaPersona.setVisible(true);  // Inicia mostrando la vista de Persona
    }
}
```

#### Manejo de múltiples vistas y controladores

Cuando se tienen múltiples vistas, es importante definir cuál es la vista principal que se muestra primero. También es común que algunas vistas se llamen entre sí, por ejemplo, una vista secundaria podría abrirse desde la principal. Para gestionar este comportamiento:

1. **Definir la vista principal:** Decide cuál es la vista principal que se mostrará al arrancar la aplicación.
  
2. **Conectar vistas y controladores:** Cada vista está asociada a un controlador, que maneja los eventos y la lógica de esa vista.

3. **Navegación entre vistas:** Si la aplicación requiere navegar entre diferentes vistas, los controladores deben tener acceso a las otras vistas que pueden ser abiertas. Esto se puede lograr pasando referencias de las vistas entre controladores o utilizando un "gestor de vistas" que maneje esta lógica.

**Ejemplo de navegación entre vistas:**

```java
public class ControladorPersona implements ActionListener {
    private VistaPersona vista;
    private Persona persona;
    private SQLPersona modelo;
    private VistaOtraEntidad vistaOtraEntidad;

    public ControladorPersona(VistaPersona vista, Persona persona, SQLPersona modelo, VistaOtraEntidad vistaOtraEntidad) {
        this.vista = vista;
        this.persona = persona;
        this.modelo = modelo;
        this.vistaOtraEntidad = vistaOtraEntidad;

        vista.btnAbrirOtraVista.addActionListener(this);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == vista.btnAbrirOtraVista) {
            vistaOtraEntidad.setVisible(true);  // Mostrar la otra vista
        }
        // Manejar otros eventos...
    }

    public void iniciar() {
        vista.setTitle("CRUD Persona");
        vista.setLocationRelativeTo(null);
        vista.setVisible(true);
    }
}
```

### Escalabilidad en proyectos grandes

Para proyectos más grandes, considera los siguientes consejos adicionales:
- **Modularidad:** Divide las funcionalidades en módulos o paquetes bien definidos. Esto hace que el proyecto sea más fácil de entender y mantener.
- **Gestión de eventos:** En lugar de manejar todos los eventos en una sola clase controladora, se pueden delegar eventos específicos a clases de controladores especializados.
- **Uso de patrones adicionales:** Considera utilizar otros patrones de diseño, como el *Facade* para simplificar interacciones entre subsistemas, o *Observer* para manejar eventos de forma más flexible.

Para dudas recurrentes te recomiendo leer [[1.1 Notas MVC]]

## `fas:Github` Ejemplos

- https://github.com/jangelmm/ats-java-bd-mvc.git
- https://github.com/jangelmm/ats-java-bvd-mvc-crud.git
- https://github.com/jangelmm/ats-java-bd-registro-usuarios.git
