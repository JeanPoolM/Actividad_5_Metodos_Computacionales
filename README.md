# TAREA NO5. FLUJO CUASIDINAMICÓ CON RANDOM WALK Y AJUSTE AUTOMATICO DE TAPS.

**Autor : Jean Pool Marín Betancur**
**Mail :** jeanpool.marin@utp.edu.co
**Fecha de entrega : 2025-04-06**

En el presente proyecto se verá evidenciado la implementación del código en **Julia** para la simulación de un flujo cuasiestático con random walk y ajuste automático de taps. El código incluye el cálculo estático del flujo de potencia, y flujo cuasidinamico a partir de la creación de un flujo de potencia con random walk. Además, se implementa el ajuste automatico de taps evaluando la regulación de tensión en los nodos de **baja tensión**de los transformadores existentes en la red.

---
### Estructura.

Para el desarrollo de este proyecto se dieron una serie de pasos a seguir para la implementación del flujo de carga cuasi-dinámico con método de punto fijo, los cuales se describen a continuación:

1) Calcular la Matriz de Admitancia Nodal Separada ($Y_{nn}$ y $Y_{ns}$) $\\$
Para comenzar se realizó el cálculo de la matriz de admitancia nodal o matriz $Y_{bus}$ utilizando parte de los códigos realizados en las actividades anteriores, sin embargo para el objetivo de esta actividad se separó además esta matriz en 2 submatrices $Y_{nn}$ y $Y_{ns}$, donde $Y_{nn}$ es la submatriz de admitancia entre nodos y $Y_{ns}$ es la submatriz de admitancia entre nodos y subestaciones.$\\$
Cabe resaltar que ambas matrices omiten el nodo slack, sin embargo para su construcción fue necesaria la transformación de las compensaciones especificadas en la red a valores de impedancia, además de la consideración de la impedancia vista por la red con respecto al tap de los transformadores.$$\\$$

2) Calculo de Flujo Estático por Punto Fijo.$\\$
Una vez obtenida las submatrices $Y_{nn}$ y $Y_{ns}$, se procedió a realizar el cálculo del flujo de potencia estático por punto fijo basados en la expresión explicada en el marco teórico.
$$T(V_n) = Y_{nn}^{-1}\left(\left(\frac{S_n}{V_n}\right)^* - Y_{ns}V_s\right)$$
Por lo cual fué necesario la creación de una función llamada *Flujo_punto_fijo()*, en la cual se calculó previamente el vector de potencias inyectadas $S_n$ utilizando los datos aportados de la base de datos *loads.csv* y *gen.csv*, y se inicializó un vector de tensiones nodales $V_n$ con los valores más probables en cada nodo ($V_n = 1.0 \angle0^\circ$), y se definio también la tensión en el nodo slack $V_s$ con un valor de $1.0 \angle0^\circ$; Luego se ejecutó el flujo de carga estatico y se obtuvieron los valores de las tensiones nodales, finalmente se redefinió el vector con estas tensiones anexandole el valor de tensión del nodo slack.$$\\$$

3) Generación de perfiles de demanda por medio del random walk.
$\\$
Para la generación de perfiles de demanda se utilizó el método de random walk, el cual consiste en generar un perfil de demanda aleatorio para cada nodo de PQ de la red, para esto se tuvo en cuenta los nodos PQ de flujo estático, este método genera una distribución de demanda aleatoria para cada intervalo de tiempo especificado, para este caso se utilizó un intervalo de tiempo de 1 hora dividido en muestras de 1 minuto y se aplicó una distribución *"normal"*, además se consideró tanto la potencia activa como la reactiva, para lo cual mediante esta función se obtenía una matriz de valores complejos por cada minuto.$$\\$$

4) Simulación cuasi-dinámica con random walk para la demanda y generación constante.
$\\$
Para la simulación cuasi-dinámica se utilizó el perfil de demanda generado por el random walk, y se consideró la generación constante, para esto se utilizó la función *Flujo_cuasidinamico()* la cual recibe como parámetros el perfil de demanda generado por el random walk, la generación constante, y la matriz de admitancia nodal separada, y devuelve el flujo de carga (tensiones nodales) para cada intervalo de tiempo especificado.$$\\$$

5) Ajuste automático de taps.
$\\$
Para el ajuste automático de taps se utilizó la función *Ajuste_aut_taps()* la cual recibe como parámetros el flujo de carga cuasi-dinámico, la base datos de la líneas, los perfiles de demanda generados por el random walk, la generación constante y la base de datos de los transformadores; internamente esta función calcula inicialmente el flujo de carga cuasidinamico base, posteriormente realiza una busqueda de las tensiones en los nodos de baja tensión de los transformadores conectados en la red, y las modifica si es necesario para mantener la tensión en los nodos de baja tensión dentro de un rango de tolerancia especificado, para esto recalcula el flujo de carga cuasidinamico y ajusta los taps de los transformadores a su vez. Finalmente devuelve el flujo de carga cuasidinamico ajustado y los taps de los transformadores ajustados en cada intervalo de 5 minutos.

---
### Marco Teórico.

El flujo de carga es una herramienta fundamental en el análisis de sistemas eléctricos de potencia. Su propósito es determinar el estado operativo del sistema, incluyendo las tensiones nodales (*Magnitud y Ángulo*) y los flujos de potencia de las líneas, a partir de condicones conocidad como la topología del sistema, las potencias generadas y demandadas, y las características de las líneas. Este problema es no líneasl debido  a la relación entre las variables de tensión y las ecuaciones de potencia.

Las principales variables nodales son:

* $P_g$ : Potencia activa generada.

* $P_d$ : Potencia activa demandada.

* $Q_g$ : Potencia reactiva generada.

* $Q_d$ : Potencia reactiva demandada.

* $V_n$ : Tensión en el nodo n.

* $\theta_n$ : Ángulo de fase en el nodo n.

#### 1) Tipos de Nodos.

* Nodo Slack: Actúa como la referencia del sistema, con una tensión fija tipicamente de $V_s = 1.0 \angle0^\circ$. Representa la subestación principal.

* Nodos PV: Nodos generación que controlan la magnitud de la tensión y la potencia activa ($|V| y P_g$ conocidos).

* Nodos PQ: Nodos de carga o generación con potencia activa o reactiva especificadas ($P$ y $Q$ conocidos).

#### 2) Método de punto Fijo.

El método de punto fijo es una técnica iterativa para resolver ecuaciones no lineales de la forma $x=T(x)$, donde $T$ es una función contractiva. En el flujo de potencia, este método se emplea para determinar las tensiones nodales que satisfacen las ecuaciones de potencia.

##### A) Definición Matemática.

Dada una función $T: B \to B$ en un espacio de Banach $B$ (como $\mathbb{C}^2$), se dice que $T$ es una contracción si:

$$||T(x)-T(y)|| \leq k ||x-y|| \ \ con \ \ k<1$$

Por el teorema del punto fijo  de Banach, $T$ tiene un único punto fijo  $x^*$ tal que $x^* = T(x^*)$. Este punto se aproxima iterativamente mediante:

$$X^{k+1}=T(x^{k})$$

partiendo de un valor inicial $x^0$, y la secuencia {$x^{(k)}$} converge a $x^*$.

##### B) Aplicación al Flujo de Potencia.

En el flujo de potencia, el sistema se divide en:

* Nodo Slack *(s)*: Con tensión conocida $V_s$.

* Nodos No Slack *(n)*: Cuyas tensiones $V_n$ se calculan.

La matriz de admitancia nodal $[Y]$ se particiona en submatrices:
$$
\begin {bmatrix}
I_s  \\
I_n
\end{bmatrix} =
\begin {bmatrix}
Y_{ss} & Y_{sn} \\
Y_{ns} & Y_{nn}
\end{bmatrix}*
\begin {bmatrix}
V_s  \\
V_n
\end{bmatrix}
$$

Para los nodos no slack, la potencia aparente se relaciona con las corrientes mediante:

$$S_n = V_n \circ I_n^*$$

Donde  $\circ$ indica el producto de *Hadamard* (elemento a elemento).

Despejando $I_n$ y sustituyendo en la ecuación de corrientes, se obtiene:

$$ V_n = Y_{nn}^{-1}\left(\left(\frac{S_n}{V_n}\right)^* - Y_{ns}V_s\right)$$

Esta ecuación se reescribe como $V_n = T(V_n)$, donde:

$$T(V_n) = Y_{nn}^{-1}\left(\left(\frac{S_n}{V_n}\right)^* - Y_{ns}V_s\right)$$

El método de punto fijo se aplica iterativamente:

$$V_n^{(k+1)} = T(V_n^{(k)}$$

usando una estimación inicial como $V_n^{(0)} = 1 \angle0^\circ$ para todos los nodos no slack.

##### c) Criterio de Convergencia.

La convergencia se evalúa con la normal del error entre iteraciones:

$$error = ||V_n^{(k+1)}-V_n^{(k)}||$$

donde $||.||$ es la norma euclidiana. El proceso puede detenerse cuando el error cae por debajo de una tolerancia o tras un número fijo de iteraciones.

#### 3) Flujo de Potencia Cuasi Dinámico.

El flujo de potencia cuasi dinámico extiende el análisis estático para modelar variaciones temporales en las condiciones del sistema, como cambios en la generación y la demanda a lo largo del tiempo.

Para cada intervalo de tiempo se actualizan las potencias inyectadas $S_n$ según la generación y demanda del intervalo, y se calcula el flujo de potencia estatico con el método de punto fijo para obtener las tensiones nodales $V_n$.

Este enfoque permite estudiar el comportamiento del sistema bajo diferentes escenarios temporales de manera eficiente, sin recurrir a simulaciones dinámicas completas.

---
### Funciones

*Librerias necesarias*

    - using LinearAlgebra
    - using PrettyTables
    - using SparseArrays
    - using Distributions
    - using Clustering
    - using DataFrames
    - using Statistics
    - using Random
    - using Plots
    - using CSV

*Calcular_Ynn_Yns()*
*Requiere*

    Calcula Ynn y Yns para cada nodo y subestación.
    Entradas : 
              Lines : Dataframe con la información de las líneas.
              Trafos : Dataframe con la información de las subestaciones.
    Salidas : 
              Ynn : Matriz de admisibilidad de nodos.
              Yns : Matriz de admisibilidad de subestaciones.
              Ybus : Matriz de admisibilidad de nodos y subestaciones.

*Flujo_punto_fijo()*
*Requiere*

    Función que calcula el flujo estático optimizado de la red.
    Entradas : 
            lines : Dataframe con la información de las líneas.
            compensation : Dataframe con la información de la compensación.
            trafos : Dataframe con la información de los transformadores.
            loads : Dataframe con la información de las cargas.
            gen : Dataframe con la información de las generaciones.
    Salidas :   
            Vns : Vector con los valores de tensión en los nodos.

*random_walk_complex()*
*Requiere*

    Simula un random walk de un número complejo durante un tiempo determinado
    Entradas : 
            minutes : Valor int que representa el tiempo de simulación en minutos.
            nodes : Valor int que representa el número de nodos en la red.
            distribution : Valor string que representa la distribución de probabilidad de cada paso del random walk.
            params : Valor array que contiene los parámetros de la distribución.
    Salidas :
            matriz_rw : Matriz de tamaño (nodes, minutes) que contiene la información de la demanda de cada nodo.
            en los minutos establecidos.

*flujo_cuasidinamico*
*Requiere*

    Función que simula el flujo cuasidinamico en una red dada la generación y demanda a lo largo de un periodo de tiempo.
    Entradas : 
              Ynn : Matriz de admisibilidad de nodos.
              Yns : Matriz de admisibilidad de subestaciones y nodos.
              rw : Matriz de random walk de tamaño (minutes, nodes) que contiene la información dada por el random walk.
              gen : Matriz de generación de tamaño (minutes, nodes) que contiene la información dada por la generación.
    Salidas :  
              Vns_df : Dataframe que contiene la información de flujo de carga en cada nodo a lo largo del tiempo.

*Ajuste_aut_taps()*
*Requiere*

    Función para el ajuste de taps optimizado para cada transformador(taps en el LV) de la red.
    Entradas : 
            lines : Dataframe que contiene la información de los transformadores de la red.
            comp : Dataframe que contiene la información de los nodos de la red.
            trafos : Dataframe que contiene la información de los transformadores de la red.
            rw : Dataframe que contiene la información de los perfiles de demanda generados por el random walk.
            gen : Dataframe que contiene la información de los generadores de la red.
    Salidas : 
            taps_df : Dataframe que contiene la información de los taps optimizados para cada transformador de la red.
            Vns_5min : Dataframe que contiene la información de los valores de tensión en cada nodo de 
            la red para cada intervalo de 5 minutos.