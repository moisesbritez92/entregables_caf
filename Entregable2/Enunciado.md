# Enunciado — Clasificador Multiclase

## Contexto y objetivo
- Objetivo: Generación de un clasificador multiclase de 
objetos basado en datos radar 2D

## Clases y datos
### Datos de interés I
- Radar Detection Interface (RDI)

### Datos de interés II
• Variable respuesta (y): Clase del objeto detectado
• Coche (0)
• Vehiculos grande (1)
• Camión (2)
• Bus (3)
• Bicicleta (5)
• Persona (7)
• Grupo de personas (8)
• Otros objeto dinámico (10)
• Background estatico (11)

### Datos de interés III
• Variables explicativas (X)
• range_sc: distancia radial [m] a la detección en el sistema de coordenadas 
relativas al sensor.
• azimuth_sc: ángulo de azimut [radianes] a la detección detección en el 
sistema de coordenadas relativas al sensor.
• radar_cross_section: Radar Cross Section [dBsm] de la detección.
• radial_velocity: velocidad radial [m/s] medida para esta detección.
• vr_compensated: velocidad radial [m/s] para la detección pero compensada 
por el movimiento del ego-vehículo.
• x_cc: posición X de la detección [m] en el sistema de coordenadas relativas 
al ego-vehículo.
• y_cc: posición Y de la detección [m] en el sistema de coordenadas relativas 
al ego-vehículo.
• x_seq: posición X de la detección [m] en el sistema de coordenadas global.
• y_seq: posición X de la detección [m] en el sistema de coordenadas global.

### Datos de interés IV
• Asumir cada detección (registro) como un objeto independiente. 
Simplificación del problema.
• A medida que trabajamos con ángulos mayores en valor absoluto 
de azimut, la fiabilidad es menor.


# Cargar los datos “Entregable2.RData” (0.5 puntos)

Reasignar las etiquetas de autobús (3) y camión (2) a vehículo grande 
(1) y la etiqueta grupo de personas (8) a persona (7) en la variable 
respuesta y. (0.5 puntos)


# Separar los datos iniciales (X e y) en “train” y “validation” (0.5 puntos)
• Train: Primeros 200k registros
• Validation: Siguientes 200k registros

Graficar el valor de Radar Cross Section (RCS) mediana ± mad para las  clases Persona (7) y Vehículo grande (1) en función de la distancia (range)  en el set de “Train”. ¿Existe diferencia significativa en términos de RCS entre la clase Persona (7) y Vehículo grande (1)? Justifica tu respuesta.(1.5 puntos)

Generar un modelo multiclase de objetos y obtener el AP (Average Precision) por 
cada clase vs resto y el mAP (mean Average Precision) en entrenamiento y en 
validación (3 puntos)
NOTA1: Se recomienda añadir interacciones entre variables explicativas.
NOTA2: Con limitación de recursos computacionales, se recomienda tomar un muestreo en el set de training.

¿Los valores de AP por clase vs resto mejoran en distancias más cercanas y ángulos de azimuth más próximos a 0? Justifica la respuesta. (2 puntos)

Elegir un punto de operación (umbral) de la curva de PR asociada a la clase persona vs resto en entrenamiento y comprobar los valores de Precision y Recall en validación correspondientes al umbral previamente definido en entrenamiento. Se pide priorizar la precisión del modelo ¿Los resultados son similares entre training y validation? Justifica la respuesta (2 puntos)
