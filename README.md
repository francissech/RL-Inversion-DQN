# Estrategia de inversión con aprendizaje por refuerzo

**Autor:** Francis Segovia Chaves  
**Programa:** Maestría en Inteligencia Artificial — Universidad de La Sabana  
**Asignatura:** Simulación y Aprendizaje por Refuerzo

## Objetivo

Construir y evaluar un agente Deep Q Network (DQN) que decida qué
proporción del capital invertir en SPY, utilizando datos históricos
y considerando costos de transacción.

El agente se compara con comprar y mantener y con una estrategia
aleatoria en un período que no se utilizó para entrenar ni seleccionar
la recompensa.

## Datos y separación temporal

Se utilizaron datos diarios de SPY descargados desde Yahoo Finance
mediante yfinance, con precios ajustados y fechas entre 2015 y 2025.

| Período | Fechas | Registros utilizables |
|---|---|---:|
| Entrenamiento | 2015-02-02 a 2021-12-31 | 1743 |
| Validación | 2022-01-03 a 2023-12-29 | 501 |
| Prueba final | 2024-01-02 a 2025-12-31 | 502 |

Los indicadores se calculan con información presente y pasada.
El normalizador se ajusta exclusivamente con entrenamiento.

## Entorno de inversión

- Capital inicial: 100 000 USD simulados.
- Estado: retorno diario, distancia relativa a la media móvil de
  20 días, volatilidad de 20 retornos y proporción actualmente invertida.
- Acciones: exposición objetivo de 0 %, 50 % o 100 %.
- Costo: 0,1 % del importe comprado o vendido.
- Decisión: después del cierre del día observado.
- Ejecución: apertura del siguiente día.
- Valoración: cierre del siguiente día.
- Participaciones fraccionarias, sin apalancamiento ni ventas en corto.
- Sin intereses sobre efectivo ni liquidación obligatoria al final.

La primera fila de cada período proporciona la observación inicial.
La primera operación ocurre en la apertura de la segunda fila.

## Algoritmo y recompensas

Se implementó DQN con Stable-Baselines3 y un entorno compatible
con Gymnasium.

Configuración principal:

- Dos capas ocultas de 64 neuronas.
- Learning rate: 0.0001.
- Batch size: 64.
- Factor de descuento: 0.99.
- Replay buffer: 50 000 transiciones.
- Inicio del aprendizaje: 2000 pasos.
- Presupuesto: 50 000 pasos por entrenamiento.
- Semillas de entrenamiento: 42, 7 y 21.
- Ejecución en CPU.

Se compararon dos recompensas:

1. **Versión 1:** retorno neto del portafolio.
2. **Versión 2:** retorno neto menos una penalización de rotación,
   calculada como 0.001 × importe operado / capital anterior.

La penalización adicional modifica la recompensa de aprendizaje;
no descuenta una comisión adicional del capital.

La versión 2 se seleccionó por su mayor retorno promedio en
validación. Se evaluaron sus tres semillas en prueba sin ajustes
posteriores.

## Resultados finales

Evaluación en 2024–2025:

| Estrategia | Retorno acumulado | Máximo drawdown | Costos totales USD |
|---|---:|---:|---:|
| Comprar y mantener | 48,39 % | 18,76 % | 99,90 |
| Aleatorio — promedio de 10 semillas | −0,23 % | 15,63 % | 22 574,40 |
| DQN v2 — promedio de 3 semillas | 16,42 % | 17,07 % | 12 196,09 |

La desviación estándar del retorno del DQN fue de 17,08 puntos
porcentuales entre semillas.

**Conclusión:** el DQN superó al agente aleatorio en retorno promedio,
pero no superó a comprar y mantener. Los resultados mostraron
variabilidad importante entre entrenamientos y costos elevados.

El máximo drawdown promedio corresponde al promedio de los
drawdowns individuales, no al drawdown de una curva promedio.

## Archivos del repositorio

| Archivo o carpeta | Contenido |
|---|---|
| `Actividad_RL_Inversion.ipynb` | Código, explicaciones y salidas del experimento |
| `data/` | Datos originales y períodos preparados |
| `models/` | Modelos entrenados, normalizador y configuraciones |
| `results/` | Figuras, métricas, historiales y registros |
| `requirements.txt` | Versiones de las bibliotecas |
| `Informe_RL_Inversion.pdf` | Informe con metodología, resultados y discusión |

## Cómo ejecutar el proyecto

1. Descargar el repositorio mediante **Code → Download ZIP**.
2. Descomprimirlo.
3. Crear un entorno de Python 3.11.
4. Instalar las dependencias desde una terminal ubicada en la
   carpeta del proyecto:

   ```bash
   python -m pip install -r requirements.txt
   ```

5. Abrir `Actividad_RL_Inversion.ipynb` en Jupyter y seleccionar
   el entorno preparado.
6. Ejecutar las celdas en orden, revisando su contenido antes de
   ejecutarlas. Las celdas con `learn()` entrenan los modelos
   y pueden tardar varios minutos.
7. Para conservar el experimento original, utilizar los CSV
   incluidos en `data/` y evitar sobrescribirlos con una nueva descarga.

## Consultar los resultados sin entrenar

El notebook contiene las salidas guardadas y puede revisarse
sin ejecutar el entrenamiento. Las métricas y figuras también
están disponibles en `results/`.

Para cargar un modelo guardado en Python:

```python
from stable_baselines3 import DQN
import joblib

modelo = DQN.load("models/dqn_v2.zip")
escalador = joblib.load("models/escalador.joblib")
```

Después se deben definir el entorno y la función de evaluación
incluidos en el notebook y cargar el período correspondiente.

Los archivos `.zip` de `models/` son modelos de Stable-Baselines3:
no deben descomprimirse manualmente.

## Limitaciones

Se estudió un solo activo y un solo período de prueba.
La simulación no incluye spread, deslizamiento, impuestos ni
impacto de mercado. Los precios ajustados se utilizan como una
serie de simulación y no como precios históricos exactos de ejecución.

Tres semillas permiten observar variabilidad, pero no establecer
una superioridad estadística general. Los resultados corresponden
a un ejercicio académico.

## Referencias

- Sutton, R. S., y Barto, A. G. (2018).
  *Reinforcement Learning: An Introduction*, segunda edición.
- [Documentación de Gymnasium](https://gymnasium.farama.org/)
- [Documentación de DQN en Stable-Baselines3](https://stable-baselines3.readthedocs.io/en/master/modules/dqn.html)
- [Documentación de yfinance](https://ranaroussi.github.io/yfinance/)
