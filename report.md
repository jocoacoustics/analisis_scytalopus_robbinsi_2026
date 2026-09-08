# Informe científico interactivo

> **Frontera de inferencia.** La validación positiva confirma una vocalización de *Scytalopus robbinsi* en un segmento; no identifica individuos, abundancia ni ocupación. La ausencia en audios revisados no equivale a ausencia ecológica.

{{PROBLEM_GRID}}

{{KPI_GRID}}

{{CONCEPT_BLOCK}}

{{QA_NOTE}}

## 1. Diseño del estudio, cobertura, curado temporal y validación
La hora de cada detección se conserva en la hora local registrada en Ecuador. El instante exacto del evento combina la hora de inicio de la grabación con su posición temporal dentro del audio. Antes de analizar o representar los datos se exige que cada registro humano coincida exactamente con su evento acústico y, cuando existe una inferencia correspondiente, también con esa inferencia. Si una validación humana no tiene pareja exacta en las inferencias, se conserva como evidencia humana y se informa como advertencia de trazabilidad; nunca se fuerza un emparejamiento.

El mapa descriptivo responde **dónde, cuándo y con qué nivel de validación se realizó el monitoreo**. El color del punto identifica campaña, no intensidad biológica. **Todas las campañas** controla conjuntamente las capas temporales; carreteras, división política e hidrografía permanecen como capas auxiliares independientes. **□ Sin validación** identifica puntos sin segmentos humanos revisados y funciona como una capa independiente del selector. **◆ Conflicto** significa que un mismo segmento exacto recibió decisión humana positiva y negativa; tener segmentos positivos y negativos distintos en un punto no es conflicto.

{{MAP_DESC_IFRAME}}

<details class="math-details" markdown="1"><summary>Auditoría y curado temporal</summary>
La auditoría temporal exige una diferencia de 0 s entre el registro original y el maestro humano y, cuando existe una inferencia correspondiente, también entre el registro original y esa inferencia. Un evento humano sin pareja exacta se trata como advertencia de trazabilidad, no como diferencia temporal. El curado abre un nuevo bloque solo con huecos de 15 días o más y conserva el bloque temporal dominante; todo excluido queda auditado.
</details>

### Perfil comparativo por punto × campaña

El radar usa una escala radial común 0–100 % respecto del máximo global de cada variable y conserva los valores naturales en hover. **Esfuerzo de validación = número de segmentos de audio distintos revisados.** Las trazas y tablas se ordenan por presencias validadas.

{{RADAR_IFRAME}}

El heatmap resume las mismas variables sin imprimir valores dentro de las celdas; el valor real permanece en hover.

{{HEATMAP_IFRAME}}

La tabla técnica se muestra directamente y conserva búsqueda, filtros, orden, paginación, ajuste de columnas y desplazamiento horizontal/vertical.

{{SUMMARY_TABLE_IFRAME}}


## 2. ¿Cuándo aparecen respuestas extraordinarias y recurrentes del clasificador?
La serie es evento-a-evento. El selector principal es el punto; la campaña elegida se informa en el título. El orden de puntos prioriza presencias validadas. Serie y matriz día×hora usan la misma hora local auditada y las marcas ★/× corresponden al segmento exacto validado.

En la vista de **Score del clasificador** se añade, sin retirar ninguna capa existente, un círculo azul abierto para cada evento candidato que el **modelo ganador** clasifica como positivo usando el umbral que maximiza MCC en predicciones OOF. La línea de score, los perfiles circadianos punto/unidad/paisaje, Hora P99 y las marcas humanas permanecen intactos.

{{SERIES_IFRAME}}

<details class="math-details" markdown="1"><summary>Perfil circadiano basal y anomalía robusta</summary>
Para cada punto \(p\) y hora local \(h\) reunimos los scores del clasificador observados en esa hora:

\[
S_{p,h}=\{s_j:\;p_j=p,\;h_j=h\}.
\]

El **perfil basal** es simplemente lo habitual para ese punto a esa hora: su línea central es la mediana \(\widetilde s_{p,h}\) y la banda muestra P10–P90. Para medir cuánto se separa un evento de ese comportamiento normal usamos, por primera vez, la **desviación absoluta mediana (MAD)**:

\[
\operatorname{MAD}_{p,h}=\operatorname{mediana}_{s_j\in S_{p,h}}
\left|s_j-\widetilde s_{p,h}\right|.
\]

Conceptualmente, MAD es una medida robusta de dispersión: dice cuánto suelen apartarse los scores de su mediana sin dejar que unos pocos valores extremos dominen la escala. La anomalía del evento \(e\) se calcula como

\[
A_e=\frac{s_e-\widetilde s_{p,h_e}}
{\max\!\left(1.4826\,\operatorname{MAD}_{p,h_e},\,0.01\right)}.
\]

Lectura simple: \(A_e\approx0\) significa un score normal para ese punto y hora; valores positivos indican una respuesta por encima del basal y valores negativos, por debajo. El factor 1,4826 pone MAD en una escala comparable con la desviación estándar cuando la distribución es aproximadamente normal; el piso 0,01 evita divisiones inestables cuando el perfil es casi plano. Esto describe la salida del clasificador, no una probabilidad de presencia.
</details>

## 3. ¿Dónde se concentra la intensidad relativa de confirmación acústica?
La **tasa de confirmación acústica** sigue siendo el cociente entre segmentos con presencia validada y segmentos revisados. No es detectabilidad ecológica. Todos los puntos pertinentes permanecen visibles; únicamente los que alcanzan el mínimo de revisión configurado sostienen el dominio y la superficie. Los que no alcanzan ese mínimo se muestran en gris.

El mapa publica un **IDW local de la intensidad relativa observada**, usando la misma escala visual de los puntos: el valor cero aparece en rojo, cualquier confirmación positiva entra en la zona amarilla y los valores relativamente altos avanzan hacia verde. Fuera del soporte no se dibuja superficie.

{{MAP_CONFIRM_IFRAME}}

<details class="math-details" markdown="1"><summary>Tasa e intensidad relativa de confirmación</summary>
La tasa natural es:

\[
C_{p,c}=\frac{N_{p,c,+}}{N_{p,c,revisados}}.
\]

Sea \(Q^{+}_{0.95,c}\) el Q95 de las tasas positivas entre puntos con soporte suficiente. Con \(a_C=0.45\):

\[
I^{C}_{p,c}=\begin{cases}
0, & C_{p,c}=0,\\
a_C+(1-a_C)\min\!\left(1,\frac{C_{p,c}}{Q^{+}_{0.95,c}}\right), & C_{p,c}>0.
\end{cases}
\]

El IDW local usa luego \(I^C\) con los mismos parámetros \(k=6\) y \(p=2.4\). El hover conserva tasa real, positivos, revisados, Q95 positivo e intensidad relativa. Un valor alto con poco esfuerzo no entra a la superficie si no alcanza el mínimo contractual de segmentos revisados.
</details>


## 4. ¿Qué aprendemos de la muestra humana de scores altos?
La revisión humana fue muestral: en cada punto se eligieron principalmente los scores más altos. La densidad muestra dónde se concentra cada grupo; la ECDF del eje derecho indica qué proporción ya se acumuló al avanzar de izquierda a derecha. La comparación conserva únicamente **TP y FP**: cada densidad y cada ECDF pueden activarse por separado para contrastar cómo se distribuyen los eventos confirmados y descartados.

Un evento sin revisión **no es negativo**: sencillamente no tiene estado de verdad humana. Por eso el modelo aprende únicamente con eventos revisados e inequívocos.

{{V40_VALIDATION_AUDIT_IFRAME}}

<details class="math-details" markdown="1"><summary>Estados de verdad y dominio válido de comparación</summary>
Sea \(\mathcal E\) el conjunto de eventos acústicos. Primero separamos tres grupos, antes de asignar una etiqueta:

\[
\mathcal E_R=\{e\in\mathcal E:\;e\text{ fue revisado por una persona}\},
\qquad
\mathcal C=\{e\in\mathcal E_R:\;e\text{ tiene decisiones contradictorias}\}.
\]

Los eventos válidos para comparar y entrenar son simplemente

\[
\mathcal V=\mathcal E_R\setminus\mathcal C.
\]

Solo dentro de \(\mathcal V\) definimos la verdad humana:

\[
T:\mathcal V\longrightarrow\{0,1\},
\qquad
T(e)=
\begin{cases}
1, & \text{vocalización confirmada},\\
0, & \text{evento revisado y descartado}.
\end{cases}
\]

Así, un evento **no revisado** pertenece a \(\mathcal E\setminus\mathcal E_R\): sigue siendo candidato y nunca se transforma en negativo. Un conflicto pertenece a \(\mathcal C\) y se excluye del entrenamiento. Como \(\mathcal V\) proviene principalmente de scores altos seleccionados para revisión, las métricas describen ese dominio revisado y no la sensibilidad global sobre todos los eventos.
</details>


## 5. ¿La forma de la serie separa verdaderos y falsos?
Un **evento** es una ventana temporal propuesta por el clasificador dentro de una grabación. Se identifica por punto, inicio del audio, segundos inicial/final y modelo. Su tiempo exacto es el inicio de la grabación más el segundo inicial de la ventana.

Para comparar formas, cada evento revisado se centra en \(t=0\) y se conserva únicamente el contexto del mismo audio. El selector cambia el ancho de los bins entre 2,5, 5 y 10 segundos. Para TP y FP se muestran por separado la mediana y la región P10–P90; ambas aparecen explícitamente en la leyenda.

{{V40_SIGNAL_IFRAME}}

<details class="math-details" markdown="1"><summary>Evento, bins y baseline robusto del punto</summary>
Representamos un evento como

\[
e=(p,c,a,[u_0,u_1],s).
\]

Aquí \(p\) es el punto, \(c\) la campaña, \(a\) la grabación, \([u_0,u_1]\) la ventana en segundos y \(s\) su score. Definimos una sola vez \(t_a\) como la hora local de inicio de la grabación \(a\); por tanto, el tiempo exacto del evento es

\[
t_e=t_a+u_0.
\]

Para un ancho \(b\in\{2.5,5,10\}\) s y un centro relativo \(\tau\), el perfil toma, dentro de esa misma grabación, el máximo score que cae en el bin:

\[
m_{e,b}(\tau)=\max\left\{s_j:\;a_j=a_e,\;t_j-t_e\in\left[\tau-\frac b2,\tau+\frac b2\right)\right\}.
\]

En cada \(\tau\), la curva es la mediana de \(m_{e,b}(\tau)\) entre eventos TP o FP y la región sombreada contiene P10–P90. Curva y región aparecen como elementos independientes de la leyenda para poder ocultarlas por separado.

Para el baseline del punto reutilizamos la misma medida robusta ya definida en el perfil circadiano, ahora calculada sobre todos los scores del punto:

\[
\widetilde s_p=\operatorname{mediana}_{j:p_j=p}(s_j),
\qquad
\operatorname{MAD}_p=\operatorname{mediana}_{j:p_j=p}\left|s_j-\widetilde s_p\right|,
\]

\[
z^{rob}_{e,p}=\frac{s_e-\widetilde s_p}{\max(1.4826\,\operatorname{MAD}_p,0.01)}.
\]

En este contexto, \(z^{rob}_{e,p}\) responde una pregunta simple: ¿qué tan excepcional es el score del evento respecto del comportamiento habitual de **ese punto**?
</details>


## 6. ¿La evidencia persiste en audios cercanos durante horas o días?
El protocolo nominal registra 2 minutos y reposa 13 minutos. El tiempo de reposo no se rellena con ceros. Las cajas comparan TP y FP usando solamente audios realmente disponibles alrededor del evento revisado.

El gráfico ofrece dos escalas de lectura publicadas: **horas y días**. La vista por horas mantiene la transformación visual \(\log(1+r)\) para separar mejor las primeras escalas; la vista por días llega hasta **30 días**. El cambio de vista no modifica datos ni cálculos. En cada escala la vela TP queda a la izquierda y la FP a la derecha. TP usa Calla Green y FP usa Rooibos.

{{V40_NEIGHBOR_IFRAME}}

<details class="math-details" markdown="1"><summary>Persistencia multiescala corregida por esfuerzo</summary>
Sea \(A_r(e)\) el conjunto de grabaciones **existentes** del mismo punto y campaña cuyo inicio cae dentro de \(\pm r\) alrededor del audio del evento \(e\). Un audio aporta evidencia cuando su score máximo alcanza o supera el percentil 95 del punto:

\[
I(a,p)=\mathbf 1\!\left\{\max_{j\in a}s_j\ge Q_{0.95,p}\right\}.
\]

La persistencia observada es

\[
D_r(e)=
\begin{cases}
\dfrac{\sum_{a\in A_r(e)}I(a,p_e)}{|A_r(e)|}, & |A_r(e)|>0,\\[6pt]
\text{no definida}, & |A_r(e)|=0.
\end{cases}
\]

Se evalúan radios descriptivos desde 5 segundos hasta 30 días; M3 continúa usando únicamente las escalas contractuales hasta 12 horas. El denominador es \(|A_r(e)|\), no el número teórico de ciclos 2/13; por tanto, un intervalo sin archivo de audio es tiempo no observado y jamás se convierte en cero acústico. La escala \(x=\log(1+r)\) cambia únicamente la separación gráfica de la vista horaria.
</details>


## 7. ¿Qué modelo reconoce mejor una presencia en puntos no vistos?
La comparación concentra toda la etapa de modelado. **P99** permanece como referencia estadística histórica; M0–M3 añaden contexto progresivamente; **T0** resume la forma temporal fina mediante una regresión logística balanceada; **T1** recibe exactamente la misma información en un modelo no lineal; y **N0** parte de T0 y aprende una corrección pequeña basada en estadística robusta y anomalías multiescala. **T2** recibe esa misma información ampliada en una regresión logística balanceada para comprobar si la ganancia proviene de las nuevas características o de la no linealidad neuronal.

La evaluación es deliberadamente estricta: **todos los modelos reciben exactamente los mismos folds de puntos**. Un fold es un grupo completo de puntos que permanece fuera del entrenamiento durante una ronda. Al reunir las predicciones de todas las rondas obtenemos predicciones OOF: cada evento fue evaluado cuando su punto era desconocido para el modelo. Por eso la comparación responde a la pregunta útil: **¿funciona en puntos que nunca vio?**

<table class="book-table model-table">
<thead><tr><th>Nombre</th><th>Qué información utiliza</th></tr></thead><tbody>
<tr><td><strong>P99 histórico</strong></td><td>Referencia de respuestas extraordinarias ajustadas por esfuerzo; no compite por ser ganador supervisado.</td></tr>
<tr><td><strong>M0</strong></td><td>Solo el score acústico del evento.</td></tr>
<tr><td><strong>M1</strong></td><td>Score + momento del día, época del año, año y campaña.</td></tr>
<tr><td><strong>M2</strong></td><td>M1 + comportamiento habitual del punto + forma intra-audio.</td></tr>
<tr><td><strong>M3</strong></td><td>M2 + persistencia entre grabaciones hasta 12 horas.</td></tr>
<tr><td><strong>T0</strong></td><td>Score + vecindad temporal fina dentro del mismo audio; conserva la formulación que mejor ha funcionado.</td></tr>
<tr><td><strong>T1</strong></td><td>Exactamente la misma información de T0 mediante CatBoost.</td></tr>
<tr><td><strong>T2</strong></td><td>Regresión logística balanceada con toda la información robusta/anómala diseñada para N0; las distancias a clases se estiman solo con el training del fold.</td></tr>
<tr><td><strong>N0</strong></td><td>T0 como ancla + una red pequeña que aprende interacciones entre estadísticos robustos, formas anómalas y distancia a patrones positivos/negativos.</td></tr>
</tbody></table>

Las **ausencias validadas** son tan importantes como las presencias: representan falsos positivos reales y enseñan qué formas temporales no corresponden a la especie. La validación humana funciona exclusivamente como respuesta a aprender; nunca entra como una característica predictora.

El gráfico muestra **AP, MCC y F1 máximo**. La regla de selección permanece fija: **AP → MCC → F1**. P99 sigue visible para comparación histórica, pero no puede ganar el benchmark supervisado.

{{V40_BENCHMARK_SUMMARY}}

{{V40_BENCHMARK_IFRAME}}

{{V40_GENERALIZATION_IFRAME}}

<details class="math-details" markdown="1"><summary>T0, T2 y N0 · forma temporal, anomalías, prueba lineal y red restringida</summary>
**T0: la hipótesis que ya funciona.** El clasificador produce una secuencia de scores dentro de cada audio. Para un evento central con score \(s_0\), T0 conserva valores vecinos a segundos y resume cuatro escalas temporales, \(r\in\{6,15,30,60\}\) s. En cada escala calcula tamaño de muestra, media, mediana, mínimo, máximo, desviación estándar, rango, prominencia, fracción y área sobre el P95 del punto; además usa duración de la racha sobre P95, pendientes antes/después y diferencias con los vecinos inmediatos. Ninguna vecindad cruza de un audio a otro.

Si \(\phi_j\) es la característica temporal fina número \(j\), T0 estima

\[
\widehat p^{(T0)}=\sigma\!\left(\beta_0+\beta_s\widetilde s_0+\sum_{j=1}^{q}\theta_j\widetilde\phi_j\right),
\qquad \sigma(u)=\frac{1}{1+e^{-u}}.
\]

Aquí \(\widehat p^{(T0)}\) es la probabilidad estimada; \(\beta_0\) es el intercepto; \(\beta_s\) y \(\theta_j\) son coeficientes aprendidos; \(q\) es el número de características; y la tilde indica imputación y estandarización. T0 usa ponderación balanceada de clases para que la gran cantidad de ausencias no silencie a las presencias.

**Qué añade N0.** N0 no reemplaza T0 ni recibe una serie cruda esperando que descubra todo desde cero. Para cada radio \(r\) añade cuartiles \(Q_{1,r}\) y \(Q_{3,r}\), rango intercuartílico

\[
IQR_r=Q_{3,r}-Q_{1,r},
\]

y desviación absoluta mediana

\[
MAD_r=\operatorname{mediana}\!\left(|s-\operatorname{mediana}(s)|\right).
\]

Con ellos mide qué tan extraordinario es el evento respecto a su vecindad mediante un z robusto

\[
z^{rob}_r=\frac{s_0-\operatorname{mediana}_r}{1.4826\,MAD_r+\varepsilon},
\]

y una prominencia relativa

\[
P_r^*=\frac{s_0-\operatorname{mediana}_r}{IQR_r+\varepsilon}.
\]

\(\varepsilon\) es un número positivo pequeño que evita dividir por cero. N0 también resume asimetría antes/después, energía por encima de la mediana local, entropía de la forma, número de cruces del P95 y duración del episodio central a 50 %, 75 % y 90 % de la altura local. Así diferencia un pico aislado, una racha coherente y ruido alto persistente aunque compartan un máximo parecido.

**Un mismo lenguaje para cuatro escalas.** Para cada radio construimos un vector \(\boldsymbol\phi_r\) con los mismos tipos de estadísticos. Un encoder compartido aprende

\[
\mathbf h_r=\operatorname{LayerNorm}\!\left(\operatorname{SiLU}(W\boldsymbol\phi_r+\mathbf b)\right),
\qquad \operatorname{SiLU}(u)=u\,\sigma(u).
\]

\(W\) y \(\mathbf b\) son parámetros aprendidos. El mismo encoder se usa en 6, 15, 30 y 60 s: la red aprende qué significa una media, un IQR o una anomalía una sola vez y después compara cómo cambia esa evidencia con la escala temporal.

**Aprender también de lo que no es.** Dentro de cada fold, y solo con sus datos de entrenamiento, se estima un centro positivo \(\boldsymbol\mu_1\), un centro negativo \(\boldsymbol\mu_0\) y una covarianza regularizada de los negativos \(\Sigma_0\). Para un evento representado por \(\mathbf x\), N0 calcula distancias a ambos prototipos y una distancia de Mahalanobis al fondo negativo:

\[
D_0(\mathbf x)=\sqrt{(\mathbf x-\boldsymbol\mu_0)^T\Sigma_0^{-1}(\mathbf x-\boldsymbol\mu_0)}.
\]

Esta cantidad pregunta cuánto se aparta el evento de las formas que los expertos ya identificaron como falsos positivos. Ni \(\boldsymbol\mu_0\), ni \(\boldsymbol\mu_1\), ni \(\Sigma_0\) se calculan con el fold de prueba.

**T2: ¿bastan las nuevas características sin red?** T2 usa la misma ancla T0 de N0, la unión de las características de T0, los estadísticos robustos/anómalos multiescala y las cinco medidas geométricas de clase calculadas exclusivamente con el training del fold. El logit de T0 se incorpora como predictor adicional y, después de imputar y estandarizar, se ajusta una regresión logística balanceada:

\[
\widehat p^{(T2)}=\sigma\!\left(\alpha_0+\sum_{j=1}^{m}\alpha_j\widetilde x_j\right).
\]

Así T2 funciona como control de parsimonia: si supera T1, las nuevas características contienen señal adicional; si N0 supera también a T2, la ganancia restante puede atribuirse a interacciones/no linealidad y no solo a disponer de más predictores.

**T0 como ancla.** Sea

\[
\ell^{(T0)}=\log\!\left(\frac{\widehat p^{(T0)}}{1-\widehat p^{(T0)}}\right)
\]

el logit de T0. La red combina los embeddings de las cuatro escalas, las posiciones temporales finas y las distancias de anomalía para producir una corrección \(\Delta\). La salida final es

\[
\widehat p^{(N0)}=\sigma\!\left(\ell^{(T0)}+\Delta\right),
\qquad |\Delta|\le1.5.
\]

La última capa comienza exactamente en cero, por lo que antes de aprender \(\Delta=0\) y N0 reproduce T0. La red solo puede mejorar si encuentra interacciones que T0 no capturó.

**Desbalance y pérdida.** Sea \(y_i=1\) para una presencia validada y \(y_i=0\) para una ausencia validada. En cada training fold se calcula \(w_+=n_-/n_+\), donde \(n_+\) y \(n_-\) son las cantidades de positivos y negativos disponibles únicamente en ese training. N0 minimiza

\[
\mathcal L=-\frac1n\sum_{i=1}^{n}\left[w_+y_i\log p_i+(1-y_i)\log(1-p_i)\right]
+\lambda\frac1n\sum_{i=1}^{n}\Delta_i^2.
\]

Aquí \(p_i\) es la probabilidad N0, \(n\) el número de eventos de entrenamiento y \(\lambda=0.02\) penaliza correcciones innecesariamente grandes. La primera parte permite aprovechar todos los negativos sin dejar que dominen el aprendizaje; la segunda conserva T0 cuando la evidencia para corregirlo es débil.

**Folds idénticos y métricas.** Si \(G(p)\in\{1,\ldots,K\}\) es el fold del punto \(p\), en la ronda \(k\) todos los modelos entrenan con puntos \(G(p)\ne k\) y se prueban en \(G(p)=k\). La asignación se genera una sola vez. AP mide el ranking precisión–recall; MCC utiliza los cuatro resultados de la matriz de confusión; F1 combina precisión y recall:

\[
F1=\frac{2TP}{2TP+FP+FN},
\qquad
MCC=\frac{TP\,TN-FP\,FN}{\sqrt{(TP+FP)(TP+FN)(TN+FP)(TN+FN)}}.
\]

\(TP\) son presencias correctamente reconocidas, \(FP\) ausencias confundidas con presencia, \(FN\) presencias perdidas y \(TN\) ausencias correctamente descartadas. El ganador sigue decidiéndose con **AP → MCC → F1**.
</details>

T0 permanece siempre visible como referencia supervisada fuerte. N0 solo se conserva como mejora si demuestra, en esos mismos puntos no vistos, que aporta información adicional.

## 8. ¿Qué puntos conviene validar ahora?

El modelo continúa prediciendo **eventos** y la cola científica por evento se conserva como artefacto reproducible. Para la decisión operativa, sin embargo, la tabla publicada resume **una fila por punto**: así evita que decenas de ventanas del mismo sitio ocupen la pantalla y convierte la salida en una lista de lugares donde vale la pena invertir revisión humana.

En puntos sin validación se ordena por la sospecha agregada del ganador. Un punto donde ya se revisaron audios y solo hubo ausencias vuelve a la lista únicamente si el ganador supera su umbral operacional MCC OOF. Los puntos con presencia confirmada salen de esta cola de descubrimiento. Los campos de fecha, ventana, archivo y ruta pertenecen al **evento no revisado de mayor probabilidad** dentro del punto y sirven como representante accionable.

{{V40_CANDIDATES_IFRAME}}

## 9. ¿Dónde sospecha algo el modelo ganador?
El mapa parte de la misma sospecha puntual que colorea cada círculo: una observación por punto y vista. Ese resumen puntual —no cada evento individual— alimenta la superficie. La interpolación publicada usa **IDW local dentro del dominio de soporte**, porque este método conserva de forma estable el contraste espacial observado y reproduce exactamente el valor de cada estación. Fuera del soporte no se dibuja superficie. Las probabilidades científicas permanecen en su escala natural; la transformación P05–P95 se utiliza únicamente para color.

{{MAP_MODEL_IFRAME}}

<details class="math-details" markdown="1"><summary>De la sospecha puntual a la superficie · soporte local, IDW y escala de color</summary>
**1. Qué valor entra a la interpolación.** El modelo ganador produce una probabilidad para cada evento candidato. El mapa resume primero la evidencia en cada punto y campaña. Sea \(p\) un punto, \(c\) una campaña y \(n_{p,c}\) el número de eventos candidatos disponibles. Ordenamos sus probabilidades de mayor a menor y llamamos \(\widehat p_{(j),p,c}\) a la \(j\)-ésima probabilidad más alta. Usamos como máximo cinco eventos:

\[
k_{p,c}=\min(5,n_{p,c}),
\qquad
S_{p,c}=\frac{1}{k_{p,c}}\sum_{j=1}^{k_{p,c}}\widehat p_{(j),p,c}.
\]

\(S_{p,c}\in[0,1]\) es la **sospecha puntual** mostrada por el círculo; \(k_{p,c}\) es el número efectivo de eventos utilizados. Para la vista de todas las campañas se vuelve a construir una única observación espacial por punto antes de interpolar.

**2. Dónde permitimos interpolar: dominio de soporte local.** Una superficie no debe inventar continuidad donde no existen grabadoras. Sea \(x\) una posición cualquiera y sea \(d_{(r)}(x)\) la distancia desde \(x\) hasta su \(r\)-ésimo punto de monitoreo más cercano. Usamos hasta cuatro vecinos; si una campaña tiene menos estaciones, se emplea el máximo orden disponible. Con \(m_c=\min(4,N_c-1)\), donde \(N_c\) es el número de estaciones de la campaña, el núcleo del soporte es

\[
D_c=\left\{x:\ d_{(1)}(x)\le D_{1,c}\ \land\ d_{(m_c)}(x)\le D_{m,c}\right\}.
\]

\(D_{1,c}\) limita cuánto puede alejarse una celda de la estación más cercana y \(D_{m,c}\) limita cuánto puede separarse del conjunto local de estaciones. Ambos límites se obtienen de forma robusta a partir de las distancias entre puntos de esa campaña. Se añaden pequeñas zonas ancla alrededor de las estaciones para conservar su entorno inmediato. **Fuera de \(D_c\) no afirmamos cero: simplemente no hay superficie publicada.**

**3. IDW local: los puntos cercanos pesan más.** Para una posición \(x\in D_c\), sea \(N_6(x)\) el conjunto de hasta seis estaciones más cercanas. Si \(S_i\) es la sospecha observada en la estación \(x_i\) y \(d(x,x_i)\) su distancia a \(x\), la superficie es

\[
\widehat S_{IDW}(x)=
\frac{\sum_{i\in N_6(x)} w_i(x)S_i}{\sum_{i\in N_6(x)} w_i(x)},
\qquad
w_i(x)=\frac{1}{\max(d(x,x_i),35)^{2.4}}.
\]

\(w_i(x)\) es el peso de la estación \(i\): disminuye al aumentar la distancia. El piso de 35 m evita pesos numéricamente extremos cuando dos coordenadas son casi coincidentes y el exponente \(2.4\) hace que la influencia sea marcadamente local. Cuando \(x\) coincide exactamente con una estación, el algoritmo devuelve su valor observado sin promediarlo con otros puntos.

**4. Qué se publica.** La superficie visible de la campaña \(c\) es

\[
\widehat S_c^{\,pub}(x)=
\begin{cases}
\widehat S_{IDW}(x), & x\in D_c,\\[4pt]
\text{transparente}, & x\notin D_c.
\end{cases}
\]

La transparencia significa **sin soporte para interpolar**, no ausencia de la especie. Las marcas ★, ×, ◆ y □ son observaciones humanas puntuales y nunca se interpolan.

**5. Cómo se transforma la probabilidad en color.** La interpolación se calcula en la escala natural de probabilidad. Solo después se aplica una escala visual robusta común a todas las campañas. Sea \(P_{05}\) el percentil 5 y \(P_{95}\) el percentil 95 de las sospechas puntuales observadas usadas como referencia cromática. Para cualquier valor interpolado \(\widehat S(x)\),

\[
C(x)=\operatorname{clip}\!\left(\frac{\widehat S(x)-P_{05}}{P_{95}-P_{05}},0,1\right).
\]

\(C(x)\in[0,1]\) controla únicamente el color; \(\operatorname{clip}\) significa limitar el resultado al intervalo de 0 a 1. \(\widehat S(x)\) continúa siendo la probabilidad interpolada real. Por eso los valores científicos permanecen sin transformar en los puntos y sus popups.

**Lectura final.** La superficie representa **evidencia espacial del modelo ganador dentro del soporte de muestreo**. No debe interpretarse como distribución completa, ocupación, abundancia ni ausencia ecológica fuera del dominio observado.
</details>

## 10. ¿Dónde predice mayor actividad acústica de la especie?
El mapa anterior resume **fuerza de evidencia**: qué tan convincentes fueron los mejores candidatos encontrados en cada punto. Este mapa responde otra pregunta: **¿dónde aparece evidencia recurrente del ganador cuando corregimos por cuánto se monitoreó?**

Primero se buscan máximos locales por encima del P95 habitual del punto, sin utilizar etiquetas humanas para decidir dónde mirar. El modelo ganador evalúa esos candidatos y se conserva como positivo solo lo que supera el umbral MCC definido con predicciones OOF. Ventanas positivas próximas dentro del mismo audio se reúnen en un único episodio para evitar contar varias veces una misma vocalización.

La tasa publicada es **episodios acústicos predichos por 1.000 horas monitoreadas**. Se interpreta como actividad acústica del modelo, no como abundancia, ocupación ni número de individuos.

{{MAP_ACTIVITY_IFRAME}}

<details class="math-details" markdown="1"><summary>Actividad acústica, episodios, esfuerzo e IDW local</summary>
Sea \(N_{p,c}\) el número de episodios positivos reconocidos en el punto \(p\) durante la campaña \(c\), después de unir ventanas próximas del mismo audio. Sea \(H_{p,c}\) el número de horas monitoreadas. La actividad acústica predicha es

\[
A_{p,c}=1000\,\frac{N_{p,c}}{H_{p,c}}.
\]

\(A_{p,c}\) queda expresada en **episodios positivos por 1.000 h**. Para todas las campañas no se promedian tasas: primero se suman episodios y esfuerzo,

\[
A_{p,ALL}=1000\,\frac{\sum_c N_{p,c}}{\sum_c H_{p,c}}.
\]

Así una campaña corta no pesa lo mismo que una larga. El IDW local interpola los valores observados \(A_i\) de los puntos vecinos dentro del dominio de soporte:

\[
\widehat A(x)=\frac{\sum_{i=1}^{k}w_i(x)A_i}{\sum_{i=1}^{k}w_i(x)},
\qquad
w_i(x)=\frac{1}{\max(d_i(x),d_{min})^2}.
\]

Aquí \(x\) es una ubicación donde queremos visualizar la superficie; \(d_i(x)\) es su distancia al punto observado \(i\); \(k\) es el número de vecinos locales utilizados; y \(d_{min}\) evita un peso infinito cuando la consulta coincide con una estación. El color usa una escala robusta común P05–P95 solo para visualización; la tasa natural permanece en los popups.
</details>

## 11. Ambiente y modelo integrado
El análisis ambiental extiende al **ganador acústico-temporal**. Cuando existe información ambiental, M4 añade esas covariables y reutiliza exactamente los mismos folds por punto. La unión se realiza por punto, campaña y año. Si la información ambiental no está disponible, el gráfico conserva el resultado acústico-temporal y lo informa sin alterar las demás comparaciones.

{{V40_ENV_IFRAME}}

<details class="math-details" markdown="1"><summary>Extensión ambiental</summary>
M4 conserva la **familia del ganador** y añade elevación, pendiente, bosque multiescala, continuidad forestal, borde y presión antrópica. Si gana un modelo tabular, esas covariables se incorporan junto a sus predictores; si gana N0, complementan su representación robusta sin sustituir el ancla de T0. Los folds de prueba permanecen idénticos. La continuidad que alcanza 1985 se interpreta como límite inferior; no se imponen límites geográficos o altitudinales duros.
</details>
