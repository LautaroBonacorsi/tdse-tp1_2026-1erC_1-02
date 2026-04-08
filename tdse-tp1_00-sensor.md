El modelo Sensor define eventos de entrada que capturan de manera continua el estado eléctrico de los periféricos. Este módulo, que abstrae el funcionamiento de la bobina magnética, la cámara de entrada y el botón del dispensador, reacciona a dos excitaciones físicas principales detectadas en los pines del microcontrolador. El evento EV_BTN_UP se registra cuando se lee un nivel lógico alto, indicando que el dispositivo se encuentra en su posición de reposo. Por el contrario, la detección de un nivel lógico bajo genera el evento EV_BTN_DOWN, señalando el accionamiento del contacto físico por parte del usuario o la presencia de un vehículo. 

Dado que los interruptores mecánicos y los sensores presentan rebotes e inestabilidades durante las conmutaciones, el modelo implementa acciones sobre variables de control internas para realizar un filtrado (debouncing) no bloqueante. Se define una variable entera denominada tick, inicializada en cero, junto con dos constantes: DEL_BTN_MAX y DEL_BTN_MIN. Cuando el sistema detecta una transición desde un estado estable, ejecuta una primera acción preparatoria asignando el valor máximo al temporizador, expresado como tick = DEL_BTN_MAX, para iniciar una ventana de validación.

Mientras la máquina se encuentra transitando los estados inestables de flanco de bajada o de subida, la ejecución cíclica de 1 milisegundo evalúa el tiempo transcurrido mediante condiciones de guarda. Si la variable de control es estrictamente mayor al umbral mínimo definido, el sistema ejecuta repetidamente la acción de decrementar el contador, codificada como tick--. En el instante exacto en que la condición de guarda verifica que el temporizador ha alcanzado el límite inferior, es decir tick == DEL_BTN_MIN, y la excitación física concuerda con la transición esperada, el modelo ejecuta su acción de salida definitiva.

Esta última acción de salida es la responsable de emitir o despachar un evento validado hacia el módulo System, actuando como puente de comunicación en la arquitectura modular. Dependiendo del rol específico asignado a cada uno de los tres sensores configurados por el equipo, el mensaje despachado variará para identificar la fuente de la señal. El sensor de la cámara ejecuta la acción de levantar el evento EV_SYS_XX_CMR , el sensor de la bobina emite el evento EV_SYS_XX_COIL, y de manera análoga, el sensor del pulsador notifica mediante EV_SYS_XX_BTN. Esta estrategia asegura que el procesamiento central reciba señales limpias y consolidadas, manteniendo la CPU liberada durante los transitorios físicos.

A continuación se presenta la tabla de transición de estados. En esta tabla generalizada, las acciones de salida se referencian genéricamente como raise EV_SYS_XX_DOWN o raise EV_SYS_XX_UP. En la implementación real del código en C, esta nomenclatura se reemplazará por EV_SYS_XX_CMR, EV_SYS_XX_COIL o EV_SYS_XX_BTN dependiendo de si la instancia del sensor corresponde a la cámara, a la bobina magnética o al pulsador del equipo.

| Current State | Event | [Guard] | Next State | Actions |
| --- | --- | --- | --- | --- |
| ST_BTN_XX_UP  | EV_BTN_XX_UP | | ST_BTN_XX_UP | |
| ST_BTN_XX_UP  | EV_BTN_XX_DOWN | | ST_BTN_XX_FALLING | tick = DEL_BTN_XX_MAX |
| ST_BTN_XX_FALLING | EV_BTN_XX_UP | [tick > 0] | ST_BTN_XX_FALLING | tick-- |
| ST_BTN_XX_FALLING | EV_BTN_XX_UP | [tick == 0] | ST_BTN_XX_UP | |
| ST_BTN_XX_FALLING | EV_BTN_XX_DOWN | [tick > 0] | ST_BTN_XX_FALLING | tick-- |
| ST_BTN_XX_FALLING | EV_BTN_XX_DOWN | [tick = 0] | ST_BTN_XX_DOWN | raise EV_SYS_XX_DOWN |
| ST_BTN_XX_DOWN | EV_BTN_XX_DOWN | | ST_BTN_XX_DOWN | |
| ST_BTN_XX_DOWN | EV_BTN_XX_UP | | ST_BTN_XX_RISING | tick = DEL_BTN_XX_MAX |
| ST_BTN_XX_RISING | EV_BTN_XX_DOWN | [tick > 0] | ST_BTN_XX_RISING | tick-- |
| ST_BTN_XX_RISING | EV_BTN_XX_DOWN | [tick = 0] | ST_BTN_XX_DOWN | |
| ST_BTN_XX_RISING | EV_BTN_XX_UP | [tick > 0] | ST_BTN_XX_RISING | tick-- |
| ST_BTN_XX_RISING | EV_BTN_XX_UP | [tick = 0] | ST_BTN_XX_UP | raise EV_SYS_XX_UP |
