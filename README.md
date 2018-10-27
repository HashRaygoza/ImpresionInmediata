# Ejecutar periódicamente una función en Java

Una de las situaciones con las que seguro tendrá que lidiar al realizar una aplicación es ejecutar un método en forma periódica cada vez que pase un intervalo de tiempo especificado, ya sea para verificar el estado de un sensor, consultar o actualizar una base de datos, generar un reporte o enviar datos a un servidor, pero no se preocupe, hacer esto en Java es muy sencillo gracias a las clases *Timer* y *TimerTask.*

## Timer

La clase *Timer* nos permite ejecutar una función en forma periódica a un intervalo especificado, su uso es bastante sencillo basta con crear un objeto *Timer* y usar el método *scheduleAtFixedRate* el cual toma tres argumentos que son los siguientes:

* task, un objeto TimerTask cuyo método run se ejecutara al intervalo indicado
* delay, la cantidad de milisegundos que queremos esperar antes de comenzar
* period, cada cuanto en milisegundo queremos ejecutar el método run del objeto TimerTask

Esto en código se ve de la siguiente manera:

```java
temporizador.scheduleAtFixedRate(tarea, 0, 1000*segundos);
```
## TimerTask

Ya definimos cuando y cada cuanto queremos que una función se ejecute, ahora llego el momento de definir dicha función, para hacer esto debemos crear una subclase de *TimerTask* y redefinir el método run de modo que ejecute el código que nosotros queramos, como se ve en el siguiente ejemplo:

```java
package mx.com.hash.tareaprogramada;

import java.util.TimerTask;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 *
 * @author david
 */
public class Tarea extends TimerTask {
    static private final Logger LOGGER = Logger.getLogger("mx.com.hash.tareaprogramada.Tarea");
    private Integer contador;    
    
    public Tarea() {
        contador = 0;
    }

    @Override
    public void run() {
        LOGGER.log(Level.INFO, "Numero de ejecución {0}", contador);
        contador++;
    }
    
}
```

De nuevo dentro del método run puede poner el código que quiera, llamar a otras clases y demas, no sienta que debe limitarse a funciones de la subclase de *TimerTask*.

Otro detalle a recordar es que el *Timer* llama al método *run* del objeto que le pasamos, de modo que si almacena información en ese objeto esta estará disponible entre cada ejecución del Timer, esto quedara mas claro en el ejemplo.

## Ejemplo

Para dejar mas en claro todo esto hagamos un pequeño ejemplo, llamando a una función cada 5 segundos que nos escriba en pantalla cuantas veces hemos llamado a la función, para esto usaremos el siguiente código.

```java
package mx.com.hash.tareaprogramada;

import java.util.Timer;
import java.util.logging.Logger;

/**
 *
 * @author david
 */
public class TareaProgramada {
    static private final Logger LOGGER = Logger.getLogger("mx.com.hash.tareaprogramada.TareaProgramada");
    
    static public void main(String[] args){
        Tarea tarea = new Tarea();
        Timer temporizador = new Timer();
        Integer segundos = 5;
        
        temporizador.scheduleAtFixedRate(tarea, 0, 1000*segundos);
    }
}
```

```java
package mx.com.hash.tareaprogramada;

import java.util.TimerTask;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 *
 * @author david
 */
public class Tarea extends TimerTask {
    static private final Logger LOGGER = Logger.getLogger("mx.com.hash.tareaprogramada.Tarea");
    private Integer contador;    
    
    public Tarea() {
        contador = 0;
    }

    @Override
    public void run() {
        LOGGER.log(Level.INFO, "Numero de ejecución {0}", contador);
        contador++;
    }
    
}
```

Y al ejecutarlo veremos lo siguiente:

```
oct 27, 2018 2:59:06 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 0
oct 27, 2018 2:59:11 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 1
oct 27, 2018 2:59:16 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 2
oct 27, 2018 2:59:21 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 3
oct 27, 2018 2:59:26 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 4
oct 27, 2018 2:59:31 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 5
oct 27, 2018 2:59:36 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 6
oct 27, 2018 2:59:41 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 7
oct 27, 2018 2:59:46 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 8
oct 27, 2018 2:59:51 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 9
oct 27, 2018 2:59:56 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 10
oct 27, 2018 3:00:01 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 11
oct 27, 2018 3:00:06 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 12
```

Como puede ver el objeto tarea no se destruye durante la ejecución del Timer, por lo que la información en el persiste e ejecución en ejecución.

## ¿Que pasa si la función tarda mucho en ejecutarse?

Un caso que puede presentarse es que la función tarde tanto en ejecutarse que llegue el momento de volverla a ejecutar y aun no halla acabado, cosa muy posible si depende de conexiones a base de datos, servidores externos o conexiones, ¿En ese caso que pasaría?

Bueno hagamos la prueba, modifiqué la clase Tarea para que quede así

```java
package mx.com.hash.tareaprogramada;

import java.util.TimerTask;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 *
 * @author david
 */
public class Tarea extends TimerTask {
    static private final Logger LOGGER = Logger.getLogger("mx.com.hash.tareaprogramada.Tarea");
    private Integer contador;    
    
    public Tarea() {
        contador = 0;
    }

    @Override
    public void run() {
        LOGGER.log(Level.INFO, "Numero de ejecución {0}", contador);
        contador++;
        
        try {
            // Con esto hacemos que la funcion tarde *mas* en ejecutarse que
            // el periodo especificado
            Thread.sleep(10000);
        } catch (InterruptedException ex) {
            LOGGER.log(Level.SEVERE, "Error de interrupcion");
        }
    }
    
}
```

Lo que hacemos aquí es agregar un retraso a la función *run* de modo que tarde 10 segundos en ejecutarse pero no modificamos lo demas, de modo que el temporizador ejecutara cada 5 segundos una función que tarda 10 segundos en ejecutarse, el resultado se ve a continuación.

```
oct 27, 2018 3:01:17 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 0
oct 27, 2018 3:01:27 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 1
oct 27, 2018 3:01:37 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 2
oct 27, 2018 3:01:47 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 3
oct 27, 2018 3:01:57 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 4
oct 27, 2018 3:02:07 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 5
oct 27, 2018 3:02:17 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 6
oct 27, 2018 3:02:27 AM mx.com.hash.tareaprogramada.Tarea run
INFORMACIÓN: Numero de ejecución 7
```

Como ve el retraso entre llamadas es el indicado por la función, el Timer no llamara al método run sino hasta que este halla acabado por lo que no debe preocuparse de que queden cosas incompletas, lo que si puede pasar es que si ya paso el periodo entre llamadas el método run se llame inmediatamente después de terminar la llamada que tardo demas.

