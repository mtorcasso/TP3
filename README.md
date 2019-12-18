# Documentación TP3 Embebidos

## Secuencia de funciones invocadas en Example 1 - Creating Tasks

Como se ve en las siguientes figuras, las funciones invocadas son vTask, vTask2, prvSetupHardware, DEBUGOUT, xTaskCreate y vTaskScheduler.

### vTask1 y vTask2
Estas funciones que se encuentran en freetos_examples_1_to_9.c tienen el fin de declarar cada una de las tareas que se utilizarán a lo largo del programa. Ambas constan de apagar o prender un LED, imprimir por puerto serie cual tarea se está ejecutando y permanecer dentro de ella hasta que el tiempo de un delay finalice.
![nombre](link)

### prvSetupHardware
Está presente en el mismo archivo que las funciones anteriores, inicializa los periféricos como los LEDs (dejándolos apagados) y el clock.

### DEBUGOUT
Imprime por UART el string que se le pase por parámetro, como su nombre indica sirve para debuguear y verificar el correcto funcionamiento del software. Está en el archivo board_api.h.

### xTaskCreate
Es una macro que se ubica en task.h, se le pasa como parámetro un puntero a la tarea que se desea crear, un string con el nombre de dicha tarea y la prioridad entre otros.

### vTaskScheduler
Localizada en tasks.c cumple el rol de inicializar el kernel que es quien se ocupa de otorgarle tiempo de funcionamiento a cada una de las tareas. Puede ser subóptimo ya que pierde tiempo decidiendo como organizarlas.

## Tipos de datos, variables o macros utilizadas en Example 1 - Creating Tasks
En archivo freertos_examples_1_to_9.c se encuentran los siguientes:

1. mainDELAY_LOOP_COUNT, esta define la duración del delay implementado por el ciclo for.
2. pcTextForMain es un const char *. Es un string que guarda el nombre del ejemplo.
2. ul es del tipo volatile unsigned long, es el índice usado para implemetar el delay.

# Documentación de ejemplos 1 a 9

## Ejemplo 1
Este ejercicio plantea la declaración de dos tareas donde cada una prende y apaga un LED respectivamente. También imprime por UART un string diciendo que tarea se está ejecutando.
Las tareas se las declara de la siguiente forma:
![nombre](link)
Se observa que ambas tienen la misma prioridad, y como se está utilizando el método de *time slice* (el cual consiste en darle el mismo intervalo de tiempo para cada tarea), dichos strings pueden imprimirse interrumpidos:
![Impresión interrumpida](link)
O correctamente, según el valor que se le asigne a la macro configTICK_RATE_HZ:
![Impresión correcta](link)
El valor a asignarle dependerá de la tarea a realizar, como en este caso se debe mostrar un string, se debe asignarle un valor sufiente para que este llegue a completarse antes de que se empiece a imprimir el siguiente.
El diagrama temporal que muestra como pasa de una tarea a otra es el siguiente:
![Diagrama temporal](link)

## Ejemplo 2
Este ejemplo es similar al anterior, pero en vez de utilizar dos tareas, usa una sola a la cual se le pasa el string como parámetro y conmuta el estado del LED cada vez que ingresa.
![Ejemplo 2](link)

## Ejemplo 3
En este caso se le asigna una prioridad mayor a la tarea 2, por lo cual el *Scheduler* siempre le asignará los recursos a esta tarea. El diagrama temporal es el siguiente:
![Diagrama temporal Ej 3](link)

## Ejemplo 4 
En el caso anterior solo una tarea se ejecutaba por tener prioridad mayor y nunca ingresaba a la de menor prioridad. En este ejemplo se soluciona eso haciendo uso de un delay bloqueante.
La dinámica es ingresar a la tarea de mayor prioridad, ejecutarse y luego llegar al delay, una vez alcanzado este punto, dicha tarea se bloquea y le cede el procesamiento a la de siguiente prioridad y así sucesivamente. Una vez que el delay finaliza, la tarea se desbloquea y vuelve a ejecutarse. Esto sucede periódicamente.
![Diagrama temporal Ej 4](link)

## Ejemplo 5
Más alla de la periodicidad de las tareas del ejemplo pasado, la función vTaskDelay no asegura que la frecuencia con la que se ejecutan esté fija, ya que el tiempo en el que cada tarea abandona el estado de bloqueo es relativo al momento en el que llaman a dicha función.
La implementación correcta es con vTaskDelayUntil, en vez de usar como referencia cada llamada a vTaskDelay, usa los ticks, de esta forma la referencia no es cambiante sino que es única.

## Ejemplo 6
En este ejemplo se declaran 2 tareas, una periódica y una no periódica con dos instancias. La periódica tiene prioridad mayor por lo que es la primera en ejecutarse, luego entra en un estado de bloqueo que permite realizar ambas instancias de la tarea no periódica. Cuando finaliza el delay bloqueante de la periódica, el scheduler vuelve a darle los recursos.
![Diagrama temporal Ej 6](link)

## Ejemplo 8
En el ejemplo 8 se declaran dos tareas, la primera con mayor priodidad que la segunda, por lo tatnto será la primera en ejecutarse. Sin embargo, luego de finalizar su trabajo esta llama a la función vTaskPrioritySet. Como su nombre lo indica, tiene la capacidad de cambiarle la prioridad a una tarea. Lo que se hace es subirle la prioridad a la tarea 2, esto genera que es scheduler le provea el tiempo de procesamiento a la segunda tarea, mientras que dentro de esta, luego de completar su cometido decrementa su prioridad para volver a la tarea 1.

## Ejemplo 9
Aquí se muestra como se puede eliminar una tarea fácilmente. Se crea la tarea 1 en el main y se llama al scheduler para que realice su trabajo. Dentro de la tarea uno, luego de encender el LED, se crea la segunda task y se le asigna una prioridad mayor por lo que esta pasa a estar ejecutandose. Una vez cumplida su función, la segunda tarea se elimina a si misma con vTaskDelete y se retorna a la original.

# Documentación de ejemplos 10 a 16

## Ejemplo 10
Este ejemplo muestra la creación de una cola y como se envía y recibe información por ella utilizando varias tareas. La cola se la crea para contener datos del tipo long. La prioridad de las tareas que envían información son menores a la de la que recibe, por lo que cada vez que se le transmita un dato a la cola, la tarea de recepción se desbloqueará y hará uso de los recursos del sistema, es decir, nunca habrá más de un elemento en la cola.
![Diagrama temporal Ej 10](link)

## Ejemplo 11
En este caso las prioridades son al revés que en el ejemplo anterior, las tareas que envían información tienen una prioridad mayor, por lo que la tarea receptora no se activará hasta que la cola no esté llena, removiendo la información que se ubique en la primera ubicación y dejando un lugar para que las emisoras transmitan un nuevo dato. Es decir, una vez que la cola se llene, estará siempre de esta manera.
![Diagrama temporal Ej 11](link)

## Ejemplo 12
Se hace uso de un semáforo binario para desbloquear una tarea mediante una interrupción. Cada 500 ms se ingresa a una tarea periódica, esta activa por software una interrupción, la cual cumple la función de darle el semáforo a la tarea Handler, una vez que esta llega a su estado de bloqueo, vuelve a la periódica para luego volver a la espera de que se cumplan los 500 ms.
![Diagrama temporal Ej 12](link)

## Ejemplo 13
