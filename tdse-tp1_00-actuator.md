Actuator (LED)

Paso 10 - Estados y Excitaciones:

Estados del actuador (LED):
OFF: El LED se encuentra apagado (Estado inicial).
ON: El LED se encuentra encendido.

Excitaciones (Eventos):
EvON: Evento o señal para encender el LED.
EvOFF: Evento o señal para apagar el LED.

Paso 11 - Comportamiento de las transiciones:

Si el sistema se encuentra en el estado OFF y ocurre el evento EvON, transiciona al estado ON.
Si el sistema se encuentra en el estado ON y ocurre el evento EvOFF, transiciona al estado OFF.

### Actuator Statechart - State Transition Table

| Current State | Event | [Guard] | Next State | Actions |
| :---: | :---: | :---: | :---: | :---: |
| **OFF** | EvON | | **ON** | |
| **ON** | EvOFF | | **OFF** | |
