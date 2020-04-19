# Covid 19 en México

Resumen: Un archivo en formato `Rmd` para obtener el modelo exponencial del número de muertos diarios por Covid-19 en México.

Autor: Murpholinox Peligro

e-mail: murpholinox@gmail.com

fecha: 19 de abril del 200

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
<img src="https://raw.githubusercontent.com/murpholinox/plotcovid19mx/master/Rplot01.png" alt="Decesos" width="90%" />
<p class="caption">Decesos diarios en México por Covid-19</p>
</div>
