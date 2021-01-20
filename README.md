# Importar-precio-de-acciones-de-Yahoo-a-RStudio

---
title: "Importar precio de acciones de Yahoo Finance a Rstudio"
author: "Naren Castellón"
date: "01/19/2021"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

Lo primero que debemos hacer para poder importa los precio de acciones desde **_Yahoo_ _finance_** es el conocer el código bursátil. 

Un código bursátil es una serie exclusiva de letras o números que se utiliza para identificar a las empresas que cotizan en bolsa. Cuando una empresa decide cotizar en bolsa, decidirá en qué bolsa cotizar y, una vez elegida, elegirá un código bursátil único para diferenciarse de las demás empresas en la misma bolsa.

En ocasiones también se llaman denominación abreviada del valor o stock ticker. El nombre stock ticker se deriva de la época en que los códigos bursátiles se transmitían a través de cintas de teletipo (ticker tape, en inglés).

**Por ejemplo:**

* **AAPL:** Apple
* **MSFT:** Microsoft
* **AMZN:** Amazon
* **EBAY:** eBay

Una vez que conoscamos el código o stock ticker, procedemos a importar la información o movimiento de cada accion que hallamos escogido antes para su análisis.

Usaremos el paquete `quantmod` para descargar el precio de las acciones.

**Quantmod** significa    marco de modelos financieros cuantitativos ''. Tiene dos funciones principales:

* descargar datos. 
* gráficos.

Antes de comenzar, usemos el siguiente código para instalar y cargar **quantmod**.

```{r message=FALSE, warning=FALSE}
#install.packages("quantmod")
library(quantmod)
library(knitr)
```

## **Descarga de datos**

Utilizaremos la función `getSymbols` para obtener datos de yahoo o Google (el predeterminado es yahoo)

```{r message=FALSE}
getSymbols("AAPL")#extraemos los datos financiero de AAPL
```

Veamos qué hay dentro de los datos.
```{r}
head(AAPL,n=10)
```
Esto se denomina datos de precios OHLCVA. Tenemos precios de apertura, máximos, mínimos, apertura, cierre, volumen y cierre ajustado. Máximo y mínimo son los precios máximos y mínimos del día de negociación. Apertura y cierre son los precios de apertura y cierre del día de negociación. El volumen es el número de acciones negociadas en el día de negociación. El precio de cierre ajustado es el precio de cierre que se ajusta al evento posterior al cierre del mercado, como la división de acciones y los dividendos.

Para extraer columnas, usamos Op, Hi, Lo, Cl, Vo y Ad

```{r}
Open <- Op(AAPL)   #Precio de apertura
High <- Hi(AAPL)    # Precio alto
Low <- Lo(AAPL)  # Precio bajo
Close<- Cl(AAPL)   # Precio de cierre
Volume <- Vo(AAPL)   #Volumen
AdjClose <- Ad(AAPL) # Cierre de ajustado
```

Si solo desea importar en una fecha determinada, por ejemplo. 2000-01-01 al 2015-09-25, podemos restringir el conjunto de datos para descargar.

```{r}
getSymbols("AAPL", from='2000-01-01',to='2015-09-25')
```
Para visualizarlo
```{r}
head(AAPL)
```

Alternativamente, podemos cargar la serie completa y restringir usando la función `xts last ()`

```{r}
getSymbols("AAPL")
```
```{r}
AAPL <- xts::last(AAPL,'1 year')
head(AAPL)
```

Alternativamente, podemos tomar los primeros 3 años usando `first ()`:

```{r}
getSymbols("AAPL")
AAPL <- first(AAPL,'3 years')
head(AAPL)
```

También podemos importar dos o tres acciones al mismo tiempo usando un vector.

```{r}
getSymbols(c("AAPL","MSFT"))
```
Alternativamente, podemos asignar un vector de existencias e importar según el vector.

```{r}
stocklist <- c("AAPL", "MSFT", "AMZN", "EBAY")
getSymbols(stocklist)
head(stocklist,10)
```
El paquete también puede importar acciones no estadounidenses.

```{r}
getSymbols("0941.hk")
#Em caso de erro, tente o seguinte:
#getSymbols("0941.HK",src="yahoo", auto.assign=FALSE)
```
Si hay un error, intente lo siguiente en su lugar:
```{r eval=FALSE}
getSymbols("0941.HK",src="yahoo", auto.assign=F)
```

Ahora veamos si los datos son correctos:
```{r}
head(`0941.HK`)
```
Además de los precios, a menudo nos interesa el volumen de operaciones. Además, nos gustaría encontrar el volumen a lo largo del tiempo: semanal, mensual, trimestral y anual. Podemos usar aplicar y sumar para calcular la suma móvil del volumen para cada período distinto.

```{r}
WeekVoYa<- apply.weekly(Vo(AAPL),sum)
head(WeekVoYa,10)
# suma de lunes a viernes

```

```{r}
MonthVoYa <- apply.monthly(Vo(AAPL),sum)
tail(MonthVoYa,10)
# suma al mes
```

```{r}
QuarterVoYa <- apply.quarterly(Vo(AAPL),sum)
head(QuarterVoYa,10)
# suma a un cuarto
```

```{r}
YearVoYa <- apply.yearly(Vo(AAPL),sum)
head(YearVoYa,10)
tail(YearVoYa,10)
# suma al año
```

En algún caso, nos interesa el promedio que la suma. Entonces podemos usar apply y mean. A continuación, calculamos el volumen promedio semanal de AAPL.
```{r}
WeekAveVoClYa<- apply.weekly(Vo(AAPL),mean)
head(WeekAveVoClYa,10)
```

## Gráfico
```{r}
chartSeries(AAPL, name="Gráfico de APPLE",
            type="line",
            subset='2019-01::2020-12',
            theme=chartTheme('black'))
```
