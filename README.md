# TP-SdR - Simulación de incendio y evacuación

Esta simulación muestra cómo evacuar un edificio ante un incendio. Está hecha para que los parámetros sean lo más realistas posible, usando unidades en metros, segundos y metros por segundo.

## 1. Tiempo y velocidad de la simulación

- **Unidades**: Distancias en metros (m), tiempos en segundos (s) y velocidades en metros por segundo (m/s).
- **Independiente de los cuadros por segundo**: Todo el movimiento, el humo y el fuego se calculan con el tiempo transcurrido entre cuadros, multiplicado por la velocidad global de la simulación. Esto hace que el resultado sea igual aunque el ordenador vaya más rápido o más lento.
- **Humo**: Para ahorrar recursos, las partículas de humo se actualizan en grupos (1 de cada 4), pero se ajusta el paso de tiempo para que su movimiento siga siendo correcto y realista.

## 2. Comportamiento de las personas

### 2.1 Velocidad de caminata

Cada persona camina a una velocidad distinta, considerando que son adultos y adolescentes:

- Velocidad media: 2,50 m/s
- Variación entre personas: ±0,60 m/s
- Velocidad mínima: 1,50 m/s
- Velocidad máxima (carrera): 4,00 m/s
- Cuando están "deambulando" (sin dirigirse a la salida aún): caminan un 65% de su velocidad normal.

### 2.2 Tiempo de reacción al incendio

Antes de empezar a evacuar, las personas tardan un tiempo en reaccionar. Esto se modela con una distribución log-normal:

- Tiempo típico (mediana) de pre-movimiento: 20 segundos.
- Ese tiempo varía entre 2 y 180 segundos según la persona.

### 2.3 ¿Cuándo empiezan a evacuar?

Una persona comienza a evacuar cuando ocurre alguno de estos casos:

- **Humo o calor**: Cuando hay suficiente humo cerca (o la zona está marcada como peligrosa), empiezan a prepararse para salir.
- **Véase a otros evacuar (contagio)**: Si ven a otra persona yendo hacia una salida a menos de 0,9 metros, pueden decidir evacuar también (con un pequeño retraso aleatorio entre 1 y 3 segundos).

### 2.4 Comportamiento durante la evacuación

- **Desorientación**: En algunos momentos (aproximadamente un 3% por segundo mientras evacuan), una persona puede perder momentáneamente la dirección hacia la salida. Esto dura entre 2,5 y 6 segundos, no ocurre muy cerca de las puertas (a menos de 3,5 m) y, después de recuperarse, no vuelve a pasar durante 6 segundos.
- **Laboratorios**: Cada laboratorio tiene su propio color para distinguirlos. Cada persona recuerda de qué laboratorio salió, para mantener ese color durante la simulación.
- **Evitar choques**: Las personas intentan no chocar entre sí (radio aproximado de 0,28 m). También esquivan obstáculos y usan pequeños giros para sortear aglomeraciones.
- **Llegada a la salida**: Cuando una persona llega cerca de una salida y va hacia ella, sale del edificio y desaparece de la simulación.

## 3. Humo, fuego y peligrosidad

### 3.1 Cómo se mueve el humo

- **Altura**: El humo sube desde el fuego hasta cerca del techo (aprox. 2,55 m). Luego se extiende por el techo y, con el tiempo, va bajando lentamente.
- **Velocidades aproximadas**: Subida entre 1,0 y 3,0 m/s, expansión lateral bajo techo entre 0,30 y 1,00 m/s, descenso muy lento entre 0,01 y 0,03 m/s.
- **Emisión**: Cada foco de incendio genera unas 48 partículas de humo por segundo.

### 3.2 Efecto del humo sobre la velocidad (visibilidad)

Cuanto más humo hay en el aire, más lento camina la gente (efecto observado en estudios de incendios). En esta simulación:

- Con poco humo: caminan casi a velocidad normal.
- Con humo moderado: reducen la velocidad.
- Con mucho humo: caminan bastante más lento (hasta alrededor del 20% de su velocidad normal).

Esto se aplica de forma gradual según la cantidad de humo en cada zona.

### 3.3 Efecto del humo sobre la salud (dosis acumulada)

El humo no solo dificulta ver, también puede afectar con el tiempo. En esta simulación se usa un modelo simplificado de **FED (Fractional Effective Dose)**, donde el valor va de 0 a 1. Cuando alcanza **FED = 1.0**, la persona resulta incapaz de evacuar y se marca como fallecida.

La dosis se acumula solo cuando la densidad de humo supera **0,5** (unidades del mapa de calor). Cuanto mayor es la densidad (con respecto a la densidad máxima), más rápido crece el FED.

**Escala simplificada (orientativa):**

| Densidad de humo (relativa) | Tiempo aproximado para llegar a FED = 1 |
|---|---|
| Muy densa (cerca de la fuente) | ~30 segundos de exposición continua |
| Moderada | Entre 30 y ~60 segundos, dependiendo de la concentración |
| Baja (por debajo de ~0,5) | Sin acumulación de dosis |

**Detalles del modelo:**
- Densidad mínima para acumular dosis: **0,5**
- Tiempo base para FED=1 a densidad máxima: **30 s**
- Umbral para iniciar evacuación por humo: densidad **0,8**
- Si una persona deja la zona con mucho humo, deja de acumular dosis.

**Nota**: Esto es una simplificación didáctica, basada en conceptos del modelo FED de Purser. No pretende reemplazar cálculos de ingeniería de protección contra incendios.

## 4. Controles de la simulación

- **Velocidad de simulación**: Permite acelerar o ralentizar todo el tiempo (entre 0,5× y 8×).
- **Generar humo**: Activa o desactiva la emisión de humo.
- **Abrir ventanas**: Abre ciertas aberturas para permitir que salga humo.
- **Modo evacuación**: Activa los disparadores para que la gente empiece a evacuar.
- **Ver personas / Mover personas**: Muestra u oculta a las personas, o pausa su movimiento.
