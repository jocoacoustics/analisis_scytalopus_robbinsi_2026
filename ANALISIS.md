# ANALISIS · v4.6.3

## Pregunta nueva de esta iteración

La versión añade **T2 · logística robusta/anómala** para separar dos hipótesis que N0 mezclaba: (1) que las nuevas características robustas/anómalas contienen más señal que T0/T1 y (2) que hace falta una red neuronal para aprovecharla. T2 recibe toda la información diseñada para N0 que puede expresarse como predictores tabulares y añade las cinco características de geometría de clase calculadas exclusivamente con el training de cada fold. Luego usa una regresión logística balanceada.

La comparación crítica queda **T1 vs T2 vs N0** sobre exactamente los mismos folds por punto y con la misma regla **AP → MCC → F1**. Si T2 supera T1, la ingeniería de características aporta señal aunque el modelo siga siendo lineal. Si además N0 supera T2, existe evidencia de ganancia por interacciones/no linealidad neuronal. Si T2 empata o supera N0, se prefiere la explicación más parsimoniosa.

El producto experimental de actividad acústica / 1.000 h permanece fuera del flujo activo. Esta versión se concentra exclusivamente en el benchmark comparable, Top-5, cola de validación e IDW local del ganador.


## 0. Herencia obligatoria

v4.1 tiene como padre **v3.3**. Conserva sin reinterpretar el maestro canónico, curado temporal, auditoría de hora local, P99, serie evento-a-evento, radar, cartografía de intensidad relativa, semántica ★/×/◆, dominio de soporte y QA de publicación. Ninguna salida v3.3 se elimina para introducir el modelo supervisado.

El cambio es mayor porque incorpora una nueva inferencia: estimar, dentro del dominio muestreado por validación humana, la probabilidad de que un evento candidato corresponda realmente a *Scytalopus robbinsi* usando score + contexto temporal y, cuando esté disponible, ambiente.

## 1. Verdad humana y sesgo de selección

La validación fue **muestral, no exhaustiva**: se enviaron principalmente los scores más altos de cada punto. Primero se separan los eventos revisados \(\mathcal E_R\). Dentro de ellos, \(\mathcal C\) contiene las contradicciones exactas; por tanto, el dominio válido de comparación es

\[
\mathcal V=\mathcal E_R\setminus\mathcal C.
\]

Solo sobre \(\mathcal V\) se define la verdad humana:

\[
T(e)=
\begin{cases}
1, & \text{confirmado como } S.\ robbinsi,\\
0, & \text{revisado y descartado.}
\end{cases}
\qquad e\in\mathcal V.
\]

Los eventos no revisados quedan fuera de \(\mathcal E_R\), y los conflictos se conservan para auditoría pero se excluyen del entrenamiento. Precisión, recall y AP se interpretan **en el dominio de scores altos muestreado para validación**; no son sensibilidad global sobre todos los scores.

## 2. Baseline propio de cada punto

Los puntos pueden tener distribuciones de score muy distintas. Para cada punto \(p\) se estima sobre toda la serie curada del modelo una distribución basal \(F_p\), independiente de la etiqueta humana. Se conservan mediana, P90, P95 y P99. La **desviación absoluta mediana (MAD)** se define una sola vez como \(MAD(S_p)=\operatorname{mediana}_{s\in S_p}|s-\operatorname{mediana}(S_p)|\); es una medida robusta de dispersión. Se derivan:

\[
z^{rob}_{pt}=\frac{s_{pt}-\operatorname{mediana}(S_p)}{\max(1.4826\,MAD(S_p),0.01)}
\]

 y el rango percentilar empírico del score dentro del punto. Esto permite distinguir un score absoluto alto de una señal excepcional respecto del comportamiento normal del sitio.

## 3. Estructura dentro del audio

Para cada evento candidato se reconstruye el contexto del **mismo audio** en ventanas de ±5, ±15, ±30 y ±60 s. Se calculan máximo, media, desviación, número de eventos, prominencia del score central respecto de la mediana local y área positiva por encima del baseline P95 del punto.

Los perfiles relativos TP/FP se muestran con bins elegibles de 2,5, 5 y 10 s, mediana y P10–P90 para cada clase. Son análisis descriptivos de la salida del clasificador; no se interpretan como vocalizaciones continuas si el modelo no generó ventanas continuas.

## 4. Continuidad entre audios · 15 min nominales, hasta 12 h

El protocolo nominal es 2 min grabando + 13 min reposo, aproximadamente cuatro audios/hora. Los 13 min apagados son **no observados**, nunca ceros.

Para una ventana temporal \(W_r(t)\) de radio \(r\in\{0.25,0.5,1,2,4,6,12\}\) h:

\[
D_r(t)=\frac{\#\{a\in W_r(t):\max(score_a)\ge P95_p\}}
               {\#\{a\in W_r(t):a\;\text{existe}\}}.
\]

También se conserva máximo/mediana de los máximos por audio, número de audios disponibles y distancia temporal al audio vecino con evidencia. El denominador siempre es esfuerzo observado. La publicación puede mostrar escalas descriptivas adicionales en segundos, minutos, horas y días hasta 30 días; esas vistas no añaden predictores a M3, cuyo límite científico sigue siendo 12 h.

## 5. Modelos incrementales y temporalidad fina

**P99 histórico se conserva siempre como referencia base** y nunca se elimina de la comparación. No compite por ser el ganador supervisado.

M0–M3 se mantienen para comparabilidad histórica y usan regresión logística regularizada:

\[
M_0=s,
\]
\[
M_1=s+\text{hora circular}+\text{día del año circular}+\text{año}+\text{campaña},
\]
\[
M_2=M_1+\text{baseline del punto}+\text{forma intra-audio},
\]
\[
M_3=M_2+\text{continuidad multiescala entre audios hasta 12 h}.
\]

Se añaden dos modelos que explotan explícitamente la granularidad fina de la salida del clasificador. La ventana acústica original es de aproximadamente 5 s y avanza cada 3 s; por tanto, las predicciones vecinas poseen estructura temporal y solapamiento. Las nuevas variables se calculan **solo dentro del mismo audio de 2 min**: está prohibido cruzar el intervalo nominal de 13 min no observado.

Para radios \(r\in\{6,15,30,60\}\) s se calculan media, mediana, mínimo, máximo, desviación, rango, prominencia del evento, fracción y área por encima de P95 y duración de la racha contigua sobre P95. Se añaden lags explícitos alrededor de \(t\) y pendientes pre/post 15 s.

\[
T_0=\text{regresión logística}(s+\text{vecindad temporal fina}),
\]
\[
T_1=\text{CatBoost}(s+\text{la misma vecindad temporal fina}).
\]

T0 y T1 reciben exactamente el mismo vector de entrada. Así se separa la ganancia por **información temporal** de la ganancia por **no linealidad**.

El modelo ganador se selecciona entre M0, M1, M2, M3, T0, T1 y N0 mediante la regla lexicográfica **AP → MCC → F1**. P99 queda visible como referencia no elegible. T0 permanece siempre en el benchmark como baseline supervisado fuerte y se conserva sin cambios. N0 parte del logit de T0 y aprende una corrección residual pequeña a partir de estadística robusta multiescala y medidas de anomalía; las etiquetas humanas se usan únicamente como objetivo.

Cuando existen covariables ambientales, M4 se redefine como:

\[
M_4=\text{ganador acústico-temporal}+\text{ambiente},
\]

conservando la familia del ganador: si gana T1, M4 usa CatBoost; si gana un modelo logístico, M4 mantiene regresión logística; si gana N0, el ambiente complementa la representación temporal/anómala sin sustituir el ancla T0. Los folds no cambian.

## 6. Generalización sin fuga

Está prohibido separar filas al azar. El esquema primario es `StratifiedGroupKFold` por `punto`, con objetivo K=5 y reducción automática si la distribución real de clases/grupos no permite cinco folds válidos. Todos los eventos de un punto viajan juntos.

Como prueba secundaria se usa `LeaveOneGroupOut` por `campaign_id`, conservando solo folds con ambas clases en entrenamiento y test. No se exige holdout especial por año.

## 7. Métricas y criterio de selección

El criterio principal es **Average Precision (AP)**, porque resume la calidad del ranking a través de la curva precisión–recall:

\[
AP\approx\sum_k(R_k-R_{k-1})P_k.
\]

Como criterio global se calcula el **Matthews Correlation Coefficient (MCC)**, que utiliza TP, TN, FP y FN:

\[
MCC=\frac{TP\,TN-FP\,FN}{\sqrt{(TP+FP)(TP+FN)(TN+FP)(TN+FN)}}.
\]

También se conserva el **F1 máximo**:

\[
F1=\frac{2TP}{2TP+FP+FN}.
\]

La regla de ganador es **máximo AP; empate → máximo MCC; empate → máximo F1**. Balanced Accuracy, precisión, recall, especificidad y los recalls condicionados a precisión 95 %/98 % permanecen como diagnósticos. El umbral de MCC/F1 se obtiene de predicciones OOF, nunca mirando un punto usado como test durante entrenamiento.

P99 es la referencia histórica obligatoria y no participa en la selección de ganador supervisado.

## 8. Priorización de validación

La **ciencia predictiva permanece a nivel evento**: el ganador produce una probabilidad para cada evento candidato y `data/processed/scytalopus_cola_validacion.parquet` conserva la cola auditable de eventos no revisados. Ninguna agregación por punto altera el entrenamiento, las probabilidades ni la selección AP → MCC → F1.

La publicación deriva una vista ejecutiva con **una fila por punto**. Se excluyen puntos con presencia ya confirmada; se incluyen puntos todavía sin validación y, cuando corresponde, puntos con ausencia revisada cuyo evento no revisado más fuerte supera el umbral MCC OOF del ganador. Para cada punto se muestra como representante el evento no revisado de mayor probabilidad y la sospecha del punto resume el top-5 de probabilidades no revisadas.

Los campos `recorded_at`, ventana, `source_name` y ruta de audio pertenecen únicamente a ese evento representante. La ruta solo se publica si puede resolverse de forma determinista; está prohibida la búsqueda recursiva heurística de WAV.

## 9. Raster de sospecha del modelo

Por coordenada × campaña se define:

\[
S(x,c)=\frac1{k_x}\sum_{j=1}^{k_x}\hat p_{(j)}(x,c),
\qquad k_x=\min(5,n_{x,c}),
\]

donde \(\hat p_{(j)}\) son las cinco probabilidades candidatas más altas producidas por el **ganador AP → MCC → F1**. `Todas las campañas` vuelve a agrupar eventos antes de calcular el top-k; no promedia rasters.

`S` se interpola **dentro del mismo dominio de soporte local** mediante IDW local. El valor de cada celda es un promedio ponderado de las estaciones vecinas, con pesos que decrecen con la distancia. Fuera del soporte: transparencia. La superficie significa *sospecha/evidencia supervisada del modelo*, no distribución, ocupación ni ausencia ecológica.

El mapa 9 de la publicación usa **Leaflet directo** a partir de `templates/MAPA_MODELO_BASE.html`. Cada campaña y `Todas las campañas` tienen un `L.imageOverlay` georreferenciado con bounds `[[sur,oeste],[norte,este]]`. La estación espacial es exactamente la misma fila punto×vista que colorea el círculo; está prohibido derivar el raster desde eventos/coordenadas crudas. La superficie publicada usa IDW local en escala natural y la escala global robusta P05–P95 se aplica únicamente después de interpolar para producir el color. ★/×/◆/□, puntos, IDs y dominio son capas independientes y nunca se interpolan. El popup debe usar la misma estructura visual del mapa de confirmación y mostrar el conjunto de campos científicos aprobado.

## 10. Ambiente · puente Pyrrhura/Earth Engine

El release incluye el contrato y extractor reproducible, pero el ZIP de código no inventa IDs de assets externos. Cuando el parquet ambiental esté disponible se integran: elevación, pendiente, bosque 250/500/1000 m, continuidad forestal robusta, distancia firmada al borde, cobertura/distancia antrópica, superficie construida y opcionalmente distancia a cauce/estructura del bosque.

La continuidad que llega a 1985 es un **límite inferior**, no edad absoluta. El año ambiental corresponde a la campaña o al año disponible más próximo, con lag auditado. Está prohibido imponer rangos geográficos/altitudinales duros porque sería circular si el objetivo es descubrir distribución.

## 11. Estado del arte en RAW

`data/raw/Sci-Bot. Estado del arte Scitalopus robbinsi.html` y su carpeta `_files` son material de procedencia/bibliografía. El modelo no usa el texto como predictor. Las afirmaciones ecológicas del informe final deben remitir a las fuentes primarias identificadas allí, no a Sci-Bot como autoridad final.

## 12. Cierre contractual

El release se bloquea si se pierde cualquier contrato visual o científico heredado de v3.3, si `NA` se trata como negativo, si existe split aleatorio por evento, si se rellenan pausas de grabación como ceros o si ★/×/◆ se interpolan como superficie biológica.


## Nota histórica · experimentos neuronales previos

Las arquitecturas neuronales directas y residuales ensayadas antes de v4.6.3 se conservan únicamente en el historial de cambios. No forman parte del benchmark activo. v4.6.3 reinicia la línea neuronal con N0 anclado a T0 y deja T0 intacto como referencia fuerte.


## v4.1 · contrato de caché por dependencias

Este parche **no modifica la definición científica de M0–M4**. Materializa por separado baseline, resumen por audio, features acústico-temporales top-N, extras de eventos validados, etiquetas humanas, join supervisado, folds, benchmark, predicciones, cola, resumen punto×campaña, raster numérico y M4.

Regla bloqueante: si la firma científica de un artefacto no cambia y el artefacto está sano, debe leerse mediante `_cache_ok()` y no recalcularse. `mtime` y cambios visuales no son identidad científica.

La caché de señal top-N depende de `raw_ia` y parámetros de señal, **no de `raw_human`**. Las etiquetas dependen del match IA×humano. Por ello nuevas validaciones pueden invalidar entrenamiento/predicción sin obligar a recalcular las ventanas acústicas ya existentes. Eventos validados fuera del top-N usan un caché pequeño separado de extras.

El raster de sospecha se guarda en forma numérica (`z`, máscara, bounds y dominio); paleta/opacidad se aplican al render. Así, un cambio visual nunca repite Kriging/IDW.


## v4.1.1 · hotfix de ejecución
Corrige la frontera SERIES_CACHE (`validadores`) / maestro (`source_name`) sin modificar la ciencia M0–M4 ni las firmas científicas de v4.1.

## v4.1.2 · hotfix de símbolos de ejecución

Corrige los nombres globales ausentes usados por las figuras supervisadas (`px`, `go`) y la referencia inexistente `circadian_sig` del manifest. Se usa la firma circadiana ya calculada `CIRCADIAN_SIG_ID`. No cambia ninguna definición, predictor, fold, métrica, salida espacial ni firma científica de M0–M4.

## v4.1.3 · reparación del benchmark y del estado humano

Esta versión deriva del notebook v4.1.2 realmente ejecutado. Normaliza `fold` a texto antes de persistir el benchmark porque la columna combina índices, agregados e identificadores de campaña. El estado humano punto×campaña selecciona los campos contractuales `ausencia_revisada` y `conflicto_punto_campania` y valida su disponibilidad antes del mapa. No cambia predictores, folds, métricas, modelos, superficies, cachés científicos ni interpretación.

## v4.1.4 · reparación de coordenadas espaciales

La cartografía crea `features_spatial` uniendo `latitud/longitud` desde `pc` por `punto + campaign_id`. `features` y la ciencia supervisada permanecen intactos.

## v4.1.5 · cierre de ejecución y lectura científica

Las auditorías del curado se cachean fuera de la carpeta de versión y se materializan incondicionalmente antes del gate final. La persistencia descriptiva se extiende a 24 horas y 1/2/3/5/7 días sin añadir esas escalas a M3. La publicación elimina versiones visibles, explica M0–M3, separa AP/recall/F1 y exige que el raster quede unido al mapa Leaflet.


## v4.1.8 · reparación visual, métrica y de runtime

Este parche no cambia los predictores M0–M4 ni la probabilidad candidata de M3. Corrige el piso 0,01 de la anomalía circadiana para que coincida con la ecuación publicada; el benchmark deja de contar el punto terminal artificial de la curva precisión–recall y reporta `NA` cuando ningún umbral real alcanza 95 %/98 % de precisión; amplía únicamente la persistencia descriptiva hasta 30 días; refuerza raster/capas del mapa de sospecha y hace que M4 reutilice los folds ya asignados. Solo se invalidan los cachés dependientes de estas correcciones.


## Corrección operativa v4.1.8

- `data/` es la única base persistente entre versiones; los derivados ausentes se reconstruyen allí antes de publicar.
- `validaciones.xlsx` se fija una vez por sesión desde sus bytes exactos para evitar fallos transitorios del mount de Google Drive.
- El raster de sospecha agregado debe ser visible aun si falla el JS del selector; el selector tiene fallback propio.
- M4 intenta materializar su parquet ambiental desde `data/` general y Earth Engine configurado antes de declararse no disponible.


## v4.2 · extensión científica bloqueante

- P99 es una referencia obligatoria en el benchmark y no puede retirarse por conveniencia visual.
- M0–M3 se mantienen sin reinterpretación; T0 y T1 añaden temporalidad fina a segundos.
- T0/T1 deben usar exactamente el mismo conjunto de variables `fine_*`.
- T1 usa CatBoost; la presencia de CatBoost es dependencia explícita del notebook.
- La selección operacional es AP → MCC → F1 sobre OOF agrupado por punto.
- `probabilidad_modelo_v4` es un nombre de columna estable por compatibilidad, pero su contenido proviene del **ganador seleccionado**, no necesariamente M3.
- M4 significa ganador + ambiente y reutiliza los mismos folds.
- El mapa 10 interpola exclusivamente la probabilidad agregada del ganador, dentro del soporte local, con Leaflet directo. Está prohibido volver a hardcodear M3 o a depender de inyección Folium+JS para mostrar el raster.


## Consolidación v4.3

- P99 se conserva como baseline estadístico y artefacto diagnóstico; su mapa sale de la publicación principal.
- T0/T1, folds por punto, AP→MCC→F1, probabilidades por evento y raster del ganador permanecen científicamente intactos respecto de v4.2.3.
- La tabla publicada de validación cambia de grano: una fila por punto, usando un evento no revisado representativo; el artefacto científico de cola continúa a nivel evento.
- El radar usa la leyenda Plotly nativa para recuperar clic/doble clic.
- La serie temporal conserva el ciclo circadiano/P99/verdad humana y añade únicamente la decisión positiva del ganador con umbral MCC OOF.
- El popup del mapa del ganador se materializa completo en Python; Leaflet no reconstruye ni interpreta campos científicos.

## v4.4 · N0, red convolucional temporal 1D

N0 se añade como modelo supervisado elegible junto con M0–M3, T0 y T1. Para cada evento central toma la secuencia de scores del **mismo audio** en una malla regular de 3 s desde −30 hasta +30 s. El tensor de entrada tiene dos canales: score y máscara de observación. El intervalo no grabado entre audios nunca forma parte de esa secuencia.

La arquitectura es compacta para limitar sobreajuste: proyección 1×1 de 2 a 8 canales, dos convoluciones 1D de kernel 3 y 16 canales con ReLU, concatenación de promedio y máximo global por canal, capa densa de 16 unidades con ReLU y dropout 0,20, y salida binaria sigmoide. La pérdida de entrenamiento es BCE sin ponderación; el optimizador es AdamW con tasa 1e−3 y weight decay 1e−4, 100 épocas y batch máximo 256.

N0 usa exactamente los mismos folds externos agrupados por punto que los demás modelos. La BCE enseña los pesos; AP juzga el ranking OOF, MCC aporta el umbral operacional y F1 funciona como tercer criterio de desempate. La regla de ganador es **AP → MCC → F1** y P99 permanece como referencia no elegible.

Si N0 gana y se dispone de covariables ambientales, M4 conserva la representación temporal convolucional y concatena las covariables ambientales después del pooling temporal, antes de la capa densa. Los folds no cambian.


## v4.6.3 · T0 preservado y N0 neuronal sobre estadística robusta/anómala

T0 conserva exactamente la formulación temporal fina que demostró mejor generalización. N0 no reemplaza esa hipótesis: parte de la predicción de T0 y aprende únicamente una corrección residual. Para cada radio de 6, 15, 30 y 60 s se añaden Q1, Q3, IQR, MAD, anomalía robusta, prominencia normalizada por IQR, asimetría temporal, energía sobre la mediana local, entropía, cruces de P95 y duración central a 50/75/90 % del pico.

Las cuatro escalas comparten el mismo encoder neuronal. El modelo añade distancias a los centroides positivo/negativo y una distancia de Mahalanobis respecto del patrón negativo; todo parámetro dependiente de la clase se estima únicamente con el training del fold. La capa residual se inicializa en cero, por lo que N0 comienza exactamente en T0. La pérdida usa BCE ponderada por la relación negativos/positivos del training y penaliza correcciones grandes.

Todos los modelos usan los mismos folds agrupados por punto. La validación humana permanece separada de las variables predictoras. La selección sigue AP → MCC → F1.

La cartografía separa dos productos: la fuerza de evidencia Top-5 del ganador y la actividad acústica predicha por 1.000 h, calculada agrupando ventanas positivas próximas del mismo audio en episodios. Ambos mapas publicados usan IDW local dentro del soporte.
