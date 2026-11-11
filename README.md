# TP-SdR - Simulación de incendio y evacuación

Esta simulación muestra cómo se propagan el fuego y el humo en los laboratorios de un edificio real y cómo evacúa la gente. Es una **simulación didáctica y cualitativa**: usa unidades en metros, segundos y metros por segundo, pero sus parámetros son supuestos sin calibrar y no reemplaza ningún estudio de seguridad contra incendios.

## 1. Tiempo y velocidad de la simulación

- **Unidades**: Distancias en metros (m), tiempos en segundos (s) y velocidades en metros por segundo (m/s).
- **Independiente de los cuadros por segundo**: Todo el movimiento, el humo y el fuego se calculan con el tiempo transcurrido entre cuadros, multiplicado por la velocidad global de la simulación. Esto hace que el resultado sea igual aunque el ordenador vaya más rápido o más lento.
- **Humo**: Para ahorrar recursos, las partículas de humo se actualizan en grupos (1 de cada 4), pero se ajusta el paso de tiempo para que su movimiento siga siendo correcto.

## 2. Comportamiento de las personas

### 2.1 Velocidad de desplazamiento

Cada persona se mueve a una velocidad distinta, considerando que son adultos y adolescentes:

- Velocidad media: 1,40 m/s
- Variación entre personas: ±0,25 m/s
- Velocidad mínima: 0,80 m/s
- Velocidad máxima: 2,08 m/s
- Cuando están "deambulando" (sin dirigirse a la salida aún): se mueven al 65% de su velocidad normal.

Como referencia, la documentación de simuladores de evacuación (que cita la guía de la SFPE) usa un perfil estándar de 1,19 m/s con un desvío de 0,25 m/s, y estudios de sensibilidad entre 0,60 y 2,08 m/s. La media de 1,40 m/s es un supuesto propio, algo por encima, para reflejar el apuro de una emergencia.

### 2.2 Tiempo de reacción al incendio

Antes de empezar a evacuar, las personas tardan un tiempo en reaccionar. Esto se modela con una distribución log-normal:

- Tiempo típico (mediana) de pre-movimiento: 20 segundos sin alarma 
- Ese tiempo varía entre 2 y 180 segundos según la persona.

### 2.3 ¿Cuándo empiezan a evacuar?

Con la opción "Evacuar al detectar humo" activada, una persona comienza a evacuar cuando ocurre alguno de estos casos:

- **Humo o calor**: Cuando la densidad de humo cerca alcanza 0,8 o su sala está marcada como Peligrosa, Muy peligrosa o Mortal.
- **Fuego cerca**: Si hay un foco a menos de 1 metro en la misma sala, reaccionan en 3 segundos.
- **Ver a otros evacuar (contagio)**: Si ven a otra persona yendo hacia una salida a menos de 0,9 metros, pueden decidir evacuar también (con un pequeño retraso aleatorio entre 1 y 3 segundos).

### 2.4 Comportamiento durante la evacuación

- **Salida**: Cada persona va a la salida más cercana (hay dos). Esquiva los focos de fuego (radio de 2,2 m) y puede cambiar de salida si el fuego le corta el camino. Al llegar, sale del edificio y desaparece de la simulación.
- **Desorientación**: En algunos momentos (aproximadamente un 3% por segundo mientras evacúan), una persona puede perder la dirección y dirigirse a un punto de otro laboratorio. Esto dura entre 2,5 y 6 segundos, no ocurre cerca de las salidas (a menos de 3,5 m) y, después de recuperarse, no vuelve a pasar durante 6 segundos.
- **Laboratorios**: Hay cuatro (Rojo, Verde, Naranja y Azul), cada uno con su color. Cada persona recuerda de qué laboratorio salió para mantener ese color.
- **Evitar choques**: Las personas intentan no chocar entre sí (radio aproximado de 0,28 m), esquivan obstáculos y abren las puertas a su paso.

## 3. Humo, fuego y peligrosidad

### 3.1 Cómo se mueve el humo

- **Altura**: El humo sube desde el fuego hasta cerca del techo (aprox. 2,55 m). Luego se extiende por el techo y, con el tiempo, va bajando lentamente. Pasa de una sala a otra según el estado de puertas y vidrios.
- **Velocidades aproximadas**: Subida entre 1,0 y 3,0 m/s, expansión lateral bajo techo entre 0,30 y 1,00 m/s, descenso muy lento entre 0,01 y 0,03 m/s.
- **Emisión**: Cada foco de incendio genera unas 48 partículas de humo por segundo.

### 3.2 Efecto del humo sobre la velocidad (visibilidad)

Cuanto más humo hay en el aire, más lento se mueve la gente. Es un supuesto del proyecto, no un valor medido. La densidad va de 0 a 20:

| Densidad de humo | Velocidad respecto de la normal |
|---|---|
| menos de 2 | 100% |
| 2 a 5 | 80% |
| 5 a 9 | 60% |
| 9 a 14 | 40% |
| 14 o más | 20% |

### 3.3 Efecto del humo sobre la salud (dosis acumulada)

Se usa un modelo simplificado de **FED (Fractional Effective Dose)**: un valor de 0 a 1 que se acumula mientras la persona está en una zona con humo. Cuando alcanza **FED = 1.0**, la persona queda incapacitada y se marca como fallecida.

La dosis se acumula solo cuando la densidad supera **0,5**, y crece de forma lineal con la densidad: `(d - 0,5) / (20 - 0,5) / 30` por segundo.

**Tiempo aproximado para llegar a FED = 1 con exposición continua (orientativo):**

| Densidad de humo | Tiempo |
|---|---|
| 20 (máxima, cerca de la fuente) | ~30 segundos |
| 14 | ~45 segundos |
| 9 | ~70 segundos |
| 5 | ~2 minutos |
| 0,5 o menos | Sin acumulación de dosis |

Si una persona deja la zona con humo, deja de acumular dosis.

### 3.4 Fuego

- El fuego crece de forma gradual: la intensidad aumenta 1/300 por segundo, a partir de 0,35 (focos manuales) o 0,15 (focos propagados).
- **Propagación**: un foco puede generar otro nuevo a 1,0–2,8 m de distancia, entre 55 y 65 segundos después.
- **Límites**: hasta 14 focos manuales y 50 en total en escritorio (5 y 20 en celular).
- **Sala de servidores**: si un foco aparece sobre un rack, este se incendia de inmediato y explota 15 segundos después. Si hay fuego en la sala, el rack se incendia a los 10 segundos y explota 15 segundos más tarde.

### 3.5 Estado de cada sala y vidrios

Cada sala se clasifica como **Normal, Peligroso, Muy peligroso o Mortal** según la porción de su superficie con humo y la densidad media. Ese estado se muestra en el panel de seguridad, dispara la evacuación y, si está activada la opción, rompe los vidrios de la sala cuando llega a Mortal.

**Nota**: Esto es una simplificación didáctica, basada en conceptos del modelo FED de Purser. El humo es un sistema de partículas con reglas simples, no una simulación de fluidos, y el comportamiento de las personas está inspirado en la literatura de evacuación pero sin validar. No pretende reemplazar cálculos de ingeniería de protección contra incendios.

## 4. Controles de la simulación

- **Generar Incendio / Quitar Incendios**: Coloca un foco con un clic sobre el mapa, o los elimina todos.
- **Velocidad de simulación**: Permite acelerar o ralentizar todo el tiempo (entre 0,5× y 8×).
- **Ventilación**: Romper vidrios automáticamente si la sala es Mortal, reparar vidrios, detener el humo, y limpiar el humo y enfriar.
- **Personas**: Mostrar u ocultar, habilitar el movimiento, activar "Evacuar al detectar humo", elegir cuántas hay por laboratorio (Rojo máx. 38, Verde 16, Naranja 11, Azul 47) y reubicarlas en sus computadoras.
- **Teclas**: `P` muestra el panel de FPS, `1` el mapa de calor y `2` la vista de cortes.
- **Modo POV** (solo escritorio): doble clic sobre una persona para ver por sus ojos. Arrastrar para mirar; el botón "Manual" permite manejarla con `W A S D` o las flechas; "Salir" o `Esc` vuelve a la vista general.
- **Celular**: la interfaz se adapta (botones "Mapa" y "Corte") y no tiene modo POV.

## 5. Visualización y rendimiento

- **Mapa de calor**: muestra la densidad de humo sobre una grilla de 80 × 80 celdas.
- **Vista de cortes**: cuatro vistas (Trasera, Frente, Izquierda y Derecha) con plano de corte para ver el humo por dentro. También hay un minimapa.
- **Rendimiento**: el panel de FPS muestra cuadros por segundo, tiempo de cuadro y llamadas de dibujo. En celular además se reduce el humo y la cantidad de focos, y no se usan luces de fuego.

## 6. Créditos

- Tecnologías: Html, CSS, JavaScript, Three.js, AutoCAD, Blender y DiceBear para los avatares. Se usó inteligencia artificial como apoyo para la programación; el modelo del edificio, las medidas, las decisiones de diseño y las pruebas fueron del equipo.
