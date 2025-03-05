---
layout: post
title:  "Patrones Singleton y Lazy Inizialization en C# y Java"
categories: dessign patterns
author:
- Gonzalo García
meta: "dessign patterns"
modified_date:   2025-03-05

---

Existe una gran variedad de patrones de diseño ampliamente utilizados en el desarrollo de software para añadir determinadas funcionalidades a nuestro código o conseguir estructuras más limpias y fáciles de mantener. 

Una de las referencias fundamentales en este campo es el libro Design Patterns: Elements of Reusable Object-Oriented Software (1994), donde los patrones quedan divididos en tres categorías:

- **Creacionales**: Se encargan de crear los objetos de forma controlada evitando que el programador los tenga que instanciar por su cuenta. Dentro de esta categoría existen también patrones encargados de clonar objetos y de limitar las clases para que sólo exista una instancia de la misma. Este último caso sería el del patrón Singleton.
- **Estructurales**: Este grupo de patrones da pautas para componer y adaptar unos objetos con otros de forma que podamos generar grupos agregados con nuevas funcionalidades.
- **Conductuales**: En esta categoría se incluyen patrones que permiten la comunicación entre objetos, así como la adaptación de los objetos a distintas circunstancias modificando su comportamiento en tiempo de ejecución.

<br>

---

<br>

#### Singleton

El patrón **Singleton** es uno de los más utilizados debido a que permite restringir el uso de una clase limitándola a una única instancia de la misma en situaciones donde queramos mantener un estado global de la clase que pueda coordinar y compartir información con el resto de objetos que accedan a ella. Para su implementación se requiere que los constructores de la clase sean privados, así como un método público de acceso que nos permita acceder a la instancia.

Ejemplo de implementación del patrón **Singleton** en **Java**

{% highlight java %}
public class EjemploSingleton {
     //Instancia única a la que podremos acceder desde otras clases.  
     private static EjemploSingleton instance = new EjemploSingleton();
     //Constructor privado  
     private EjemploSingleton() {}
 
     //Método público para acceder a la instancia.  
     public static EjemploSingleton getInstance() {    
          return instance; 
     }
     //Resto de métodos...
}
{% endhighlight %}


La implementación propuesta permite acceder a la instancia única de la clase (por tanto marcada como static) a partir el método estático de acceso público getInstance(), que simplemente se encarga de devolver la referencia a la instancia. El constructor privado garantiza que no podamos crear desde fuera más instancias de la clase mediante la cláusula new().

Es posible sellar la clase todavía más mediante el uso de la palabra clave final. Si marcamos la clase como final impedimos que ésta se pueda extender para que no haya subclases de ella.

> public final class EjemploSingleton ... {...}


Por otro lado, nuestra implementación inicial asigna una instancia en el momento de la ejecución mediante new EjemploSingleton(); no obstante, esa instancia única a la que el resto de clases tiene acceso podría ser modificada desde la propia clase EjemploSingleton con métodos que la reasignen. Una forma de impedir que la instancia de la clase se pueda reasignar es usando de nuevo la palabra final sobre la variable que almacena la referencia. En ese caso la variable no se podrá volver a inicializar.

> private static final EjemploSingleton instance = new EjemploSingleton();

En C#, el uso de la notación {get; set;} permite reescribir el ejemplo de forma más compacta. En este lenguaje, la palabra reservada sealed es la equivalente a final para clases en Java, impidiendo la herencia.

{% highlight C# %}
public class EjemploSingleton {
     //Instancia única a la que podremos acceder desde otras clases.  
     private static EjemploSingleton instance = new EjemploSingleton();
     //Constructor privado  
     private EjemploSingleton() {}
 
     //Método público para acceder a la instancia.  
     public static EjemploSingleton getInstance() {    
          return instance; 
     }
     //Resto de métodos...
}
{% endhighlight %}


Al igual que en el caso anterior, el private set permite reasignar la instancia de la clase de forma interna en momentos posteriores. Si preferimos que la instancia no se pueda volver a inicializar bastaría con eliminar el private set.

> public static EjemploSingleton instance { get; } = new EjemploSigleton();

<br>

---

<br>


#### Lazy Initialization

Otra técnica muy extendida en el desarrollo de software es la conocida como Lazy Initialization, que permite garantizar que sólo se instancia el mínimo número de objetos necesarios o que retrasa hasta el último momento la instanciación de una clase.

{% highlight java %}
public class Empleado{
     //Campos de la clase a los que accedemos mediante Getters y Setters
     private String ID;
     private String nombre;
 
     //Mapa estático de la clase donde almacenamos los empleados instanciados mediante su ID.  
     private static Map<String, Empleado> empleados = new HashMap<String, Empleado>(); 
 
     //Constructor privado. 
     //Sólo se accede a empleados por medio del método de obtener empleados.  
     private Empleado() { } 
 
     //Método para obtener empleados. 
     //Si ya existe estará contenido en el Mapa y nos devolverá la instancia. 
     //Sólo en el caso de no existir todavía creará un nuevo empleado y lo añadirá al Mapa para futuras búsquedas.
   
     public static Empleado obtenerEmpleadoPorID(string ID) {     
     Empleado empleado;
     //Comprobamos si el empleado buscado ya está registrado como existente en el Mapa
     
     if (!empleados.containsKey(ID)) {       
          //Lazy Instantiation       
          empleado = new Empleado(); 
 
          //Aprovechamos para asociar el ID al nuevo usuario.
          empleado.setID(ID); 
 
          //Añadimos el nuevo empleado al Mapa      
          empleados.put(ID,empleado);     
     } else {
          //Si el empleado ya existe, lo obtenemos del Mapa
          empleado = empleados.get(ID);     
     } 
          return empleado;   
     } 
 
     //Getters/Setters...  
     //Resto de métodos...
}
{% endhighlight %}

En el ejemplo se muestra un caso típico donde una clase mantiene un registro estático de las instancias que se han ido generando. En el momento de solicitar una de las instancias primero comprueba si ésta ya existe accediendo al Mapa que las almacena. Sólo en el caso de que no exista creará una nueva instancia y la añadirá al Mapa para permitir su seguimiento.

Finalmente, es posible combinar ambos patrones y hacer que el Singleton sólo se inicialice bajo demanda en el momento de solicitarlo utilizando **Lazy Initialization**.

{% highlight java %}
public final class EjemploSingleton {
    //La referencia a la instancia única de la clase se inicia como nula y evitamos llamar al constructor.
    //En este caso no la podemos marcar como final porque no se podría reasignar. 
    private static EjemploSingleton
    instance = null; 
   
    //Constructor privado
    private EjemploSingleton() {} 
   
    //Método de acceso a la instancia de la clase.
    public static EjemploSingleton getInstance(){ 
        //Al ser solicitado por primera vez la variable instance será null. 
        //Sólo en este momento inicializamos la referencia estática a la clase
        if (instance == null) { 
            instance = new EjemploSingleton();    
        }
        //En posteriores llamadas devolveremos directamente la referencia.
        return instance;
    } 
}
{% endhighlight %}

Para concluir, mostramos un ejemplo real de Lazy Inizialitacion en un método que forma parte de un algoritmo de búsqueda de caminos (A* Pathfinding) utilizado para obtener un nodo de la red.

{% highlight java %}
//Crea u obtiene nodos bajo demanda
private PathNode GetNodeFromGrid(int i, int j){
    PathNode node = Grid.TryGet(i, j);
 
    //Crear nuevo nodo y añadir valores por defecto
    if(node == null){
        //Lazy initialization
        node = new PathNode(i,j);
        node.DefaultValues();                
 
        //Contrastar con lista de obstáculos
        bool? obstacle = StaticObstacles.TryGet(i,j);
        if(obstacle != null && obstacle == true){
            node.isWalkable = false;
        }
 
        //Añadir dificultad al terreno
        int? cost = TerrainCost.TryGet(i,j);
        if(cost != null){
            node.intrinsicCost = (int)cost;
        }
 
        //Añadir al grid
        Grid.Put(i,j,node);
    }            
 
    return node;
}
{% endhighlight %}

El método trata de obtener una instancia de la clase PathNode almacenada en una colección bidilmensional llamada Grid. En primer lugar intenta obtener la instancia almacenada en la coordenada (i,j) direcamente mediante el método Grid.TryGet(i, j); si no la encuentra pasa a generarla mediante Lazy initialization. En otras palabras: sólo se creará un nuevo PathNode y se incluirá en la estrucutra Grid si ésta no estaba previamente contenida en la colección.

<br>

---

<br>

#### Referencias

- Design Patterns: Elements of Reusable Object-Oriented Software (Addison Wesley professional computing series). Erich Gamma, John Vlissides, Richard Helm, Ralph Johnson.
- [Microsoft Docs](https://docs.microsoft.com/es-es/dotnet/framework/performance/lazy-initialization)
- [Tutorials Point](https://www.tutorialspoint.com/design_pattern/singleton_pattern.htm)
- [Wikipedia Lazy Initialization](https://en.wikipedia.org/wiki/Lazy_initialization)
- [Wikipedia Singleton Pattern](https://en.wikipedia.org/wiki/Singleton_pattern)
- [Wikipedia Software Design Pattern](https://en.wikipedia.org/wiki/Software_design_pattern)
