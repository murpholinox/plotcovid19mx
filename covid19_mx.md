---
title: "Covid 19 Mexico"
author: "Murpholinox Peligro"
date: "19 de abril 2020"
output:
  html_document: 
    keep_md: yes
  pdf_document:
    keep_tex: yes
---



```r
# Carga paquetes
library(ggplot2)
library(ggdark)
library(latex2exp)
library(lubridate)
library(dplyr)
# Configura el directorio de trabajo
setwd("/home/murphy/Repos/plotcovid19mx")
```


```bash
# Descarga datos del European CDC
wget -O full.csv https://opendata.ecdc.europa.eu/covid19/casedistribution/csv
# Obtiene líneas correspondientes a México
grep Mex full.csv > mex.csv
# Solo ocupamos la columna 1 y 6  (fecha, decesos por día)
awk -F "," '{print $1"," $6}' mex.csv > clean.csv
# Añade nombre a las columnas
echo "fecha,decesos" >> clean.csv
# Ahora ordena los datos
tac clean.csv > clean_r.csv
```

```
## --2020-04-19 00:52:18--  https://opendata.ecdc.europa.eu/covid19/casedistribution/csv
## Resolving opendata.ecdc.europa.eu (opendata.ecdc.europa.eu)... 212.181.0.63
## Connecting to opendata.ecdc.europa.eu (opendata.ecdc.europa.eu)|212.181.0.63|:443... connected.
## HTTP request sent, awaiting response... 301 Moved Permanently
## Location: https://opendata.ecdc.europa.eu/covid19/casedistribution/csv/ [following]
## --2020-04-19 00:52:24--  https://opendata.ecdc.europa.eu/covid19/casedistribution/csv/
## Reusing existing connection to opendata.ecdc.europa.eu:443.
## HTTP request sent, awaiting response... 200 OK
## Length: 594309 (580K) [application/octet-stream]
## Saving to: ‘full.csv’
## 
##      0K .......... .......... .......... .......... ..........  8%  141K 4s
##     50K .......... .......... .......... .......... .......... 17%  277K 3s
##    100K .......... .......... .......... .......... .......... 25%  279K 2s
##    150K .......... .......... .......... .......... .......... 34%  282K 2s
##    200K .......... .......... .......... .......... .......... 43%  285K 1s
##    250K .......... .......... .......... .......... .......... 51%  281K 1s
##    300K .......... .......... .......... .......... .......... 60%  984K 1s
##    350K .......... .......... .......... .......... .......... 68%  376K 1s
##    400K .......... .......... .......... .......... .......... 77%  289K 0s
##    450K .......... .......... .......... .......... .......... 86%  281K 0s
##    500K .......... .......... .......... .......... .......... 94%  264K 0s
##    550K .......... .......... ..........                      100% 36.2M=2.0s
## 
## 2020-04-19 00:52:26 (294 KB/s) - ‘full.csv’ saved [594309/594309]
```


```r
# Carga los datos semi limpios
datos <- read.csv("~/Repos/plotcovid19mx/clean_r.csv")
# Cambia el formato de la fecha
datos$newdate <- lubridate::dmy(datos$fecha)
```


```r
# Ahora crea una nueva variable con nuevo formato para la fecha
xmax <- max(length(datos$fecha))
datos$number <- seq(0,xmax-1)
# Necesitamos los días del outbreak en México (después del 20-marzo)
smalldf<-datos %>%
  filter(number >= 72) 
# Ordena los datos
x<-smalldf$number
x<-x-71
y<-smalldf$decesos
nice<-tibble(x,y)
# Guarda datos finales
write.csv(nice, file="~/Repos/plotcovid19mx/nice.csv")
# Crea una gráfica base
p <- ggplot(data = nice, aes(x=x, y=y)) + geom_point()
```


```r
# Crea el modelo exponencial
m <-nls(y~a*exp(b*x), start = list(a=0.01, b=0.15))
# Imprime información del modelo
summary(m)
```

```
## 
## Formula: y ~ a * exp(b * x)
## 
## Parameters:
##   Estimate Std. Error t value Pr(>|t|)    
## a  2.41583    0.89536   2.698   0.0117 *  
## b  0.10731    0.01411   7.603 2.78e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 8.711 on 28 degrees of freedom
## 
## Number of iterations to convergence: 10 
## Achieved convergence tolerance: 2.018e-06
```


```r
# Agrega el ajuste con los parámetros del modelo
p2 <- p +
  stat_smooth(method = 'nls', formula = y ~ a * exp(b * x), se=FALSE,
              method.args = list(start = list(a = 1.7, b =  0.12))) +
# la ecuación de la exponencial,
      annotate("label", x=5, y=30, label=TeX('$y  =  2.4  e^{0.11  x }$')) +
# los títulos necesarios,
  ylab("Decesos") + xlab("Día") +  ggtitle("Decesos por covid-19 (20-03/19-04)") 
# y cambia el tema base dependiendo del formato de salida
if (knitr::is_html_output()) {
  p2 + dark_theme_gray(base_size = 15)
} else if (knitr::is_latex_output()) {
  p2 + theme_light(base_size = 15)
}
```


```r
  knitr::include_graphics('/home/murphy/Repos/plotcovid19mx/Rplot01.png')
```

<div class="figure" style="text-align: center">
<img src="/home/murphy/Repos/plotcovid19mx/Rplot01.png" alt="Decesos" width="90%" />
<p class="caption">Decesos</p>
</div>
