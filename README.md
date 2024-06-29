---
title: "Melquisadec"
output:
  word_document: default
  pdf_document: default
  html_document:
    df_print: paged
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE, message = FALSE,warning = FALSE)
```

# Serie histórica da exportação brasileira em kg
## Resumo 




# Introdução.
A proposta de exploração no mercado brasileiro se iniciou desde a época em que era colônia de Portugal, nesta época inicia se a exploração do ouro e comecialização no mercado internacional, posteriormente a exportação do Pau Brasil, açucar e café (Contine et al 2012). O Brasil se tornou um dos maiores exportadores de diversas materia prima nas últimas duas décadas, no cenário internacional, principalmente as commodities agrícolas que de certa forma elevou o percentual de exportação brasileira (Souza e veríssimo 2013), o Brasil se destaca em algumas culturas como soja, milho e café, de modo geral o país está entre os maiores exportadores mundiais no que tange a culturas de produção de grãos(CONAB 2022), partir deste crescimento faz se necessários modelos de previsões que viabiliza o conhecimento prévio do comportamento do mercado internacional, tais modelos facilitam na tomada de decisões dos setores estratégicos do país. Dentre os modelos de previsões destaca-se as series temporais. Series temporais são sequências de observações de uma variável ao longo do tempo e a partir destas observaçoes é possível estender um horizonte de previsão para a variável de interesse.  
Neste trabalho considerou-se uma serie histórica de exportação Brasileira, e a partir de então descrever com modelo de series temporais um horizonte de previsão para as próximas exportações do País através do modelo Arima sazonal.o chamado modelo ARIMA sazonal, ou SARIMA (MORETTIN & TOLOI, 2004). Estes modelos são importantes, pois levam em consideração a sazonalidade estocástica dos dados. Quando o período s=12, o modelo denominado SARIMA de ordem (p,d,q) × (P,D,Q)12, é dado por: φ(X)Φ(X12)∆ d∆D12 Zt = θ(X)Θ(X)at

```{r}
setwd(dirname(rstudioapi::getActiveDocumentContext()$path))
if (!require("pacman")) install.packages("pacman")
p_load(tidyverse, rmdformats,forecast,ggthemes,tseries,magrittr,astsa, rmarkdown,RColorBrewer,extRemes,data.table,quantmod,Quandl, sf,tidyr, ggplot2,readxl,xlsx,openxlsx,stringr,vroom,egg,writexl,knitr,cowplot,forecTheta,
      Mcomp, kableExtra)

```

```{r}
treino2=read.table("dados.txt", header = T)
validacao=read.table("dadosva.txt", header = T)
nrow(validacao)
xtreino=treino2[1:228,]
xteste=validacao[1:60,]

x<- ts(xtreino, start=1999, frequency = 12) #transformando numa serie temporal
x
xx=ts(xteste, start = 2018, frequency = 12)
xx
#h=length(xx)
```
## Materias e Métodos
### Dados
A serie histórica da exportação foi obtido no site da Receita Federal Brasileira, o conjunto de dados são referente a observações de quilos exportados a partir do ano de 1999 até 2022, porém o conjunto foi particionado para conjunto de treinamento e conjunto de validação, para o treinamento foi considerado as observaçoes de 1999 até o ano de 2018 e para validação foi considerado do ano 2018 a 2022. Os modelos proposto para esta análise foram arima sazonal com os dados originais, arima sozonal tomado por uma transformação de boxcox nos dados
A definição matemática do modelo ARIMA sazonal segundo BOX e JENKINS (1974) é representada da seguinte forma:
𝜙(𝐵)Φ(𝐵𝑠)𝑊𝑡 = 𝜃(𝐵)Θ(𝐵𝑠)𝑎𝑡com: 
𝑊𝑡 = ∇𝑠𝑑∇𝑑𝑋𝑡e com os seguintes operadores:
- auto-regressivo não sazonal -\phi 𝜙(𝐵) = (1 − 𝛼1𝐵 − ⋯ − 𝛼𝑝𝐵𝑝);
- auto-regressivo sazonal - Φ(𝐵𝑠) = (1 − 𝜙𝑠𝐵𝑠 − ⋯ − 𝜙𝑝𝐵𝑃𝑠);
– média móvel não sazonal.- θ(B) = (1 + 𝛽1𝐵 + ⋯ + 𝛽𝑞𝐵𝑞;Θ(𝐵𝑠) = (1 + 𝜃𝑠𝐵𝑠 − ⋯ − 𝜃𝑝𝐵𝑄𝑠) 
– operador média móvel sazonal;∇𝑑
– operador diferença não sazonal de ordem d;∇𝑠𝑑
– operador diferença sazonal de ordem D.

Foram utilizados os modelos de alisamento exponencial (ETS) e OS modelos ETS com a serie tomada por uma transformação de boxcox.
Os modelos ETS em série temporal são modelos que avaliam erro, tendência e sazonalidade. O erro pode ser multiplicativo (M) ou aditivo (A), a tendência pode não ter (N), ser multiplicativa (M), multiplicativa amortecida (Md), aditiva (A) ou aditiva amortecida (Ad) e a sazonalidade pode não ocorrer (N), ser multiplicativa (M) ou aditiva (A). Equações do modelo ETS com erros aditivos e multiplicativos podem ser encontrados em Hyndman et al. (2008).

Para o critério de escolha do melhor modelo foram utilizados a técnica de janela deslizante, o modelo que obter o menor erro absoluto é o escolhido para fazer previsões O software utilizado para tratamento dos dados foi o software Livre R TEAM 2022.  

## Resultados
Foi realizado a decomposição da serie histórica para verificar o comportamento dos dados, utilizando a função mstl do R, esta função divide os termos de tendência, sazonalidade e Ruido da serie, pode-se identificar na figura 1 os termos de tendência, ou seja, houve um aumento no na exportação Brasileira comparando os anos de 1999 a 2018, percebe-se também um padrão sazonal, justificável pois grandes volumes de exportações são referntes ao setor agrario e este setor possui épocas determinadas para colheitas, o que geram ciclos sazonais, e por fim o termo de Ruido da série que aparenta um ruido aleatório.

```{r}
mstl(x, lambda = NULL) %>% autoplot(facet=TRUE, size=0.7, color ="red") + theme_minimal()
```
![mstl.png](grafico/mstl.png)

Para aplicação dos modelos classes arimas é necessario que a serie seja estacionaria (Moretin 2004)  Se uma série {Zt} é não estacionária, logo faz se uma diferença do tipo Wt = Zt - Zt-1 , este processo é feito recursivamente até que a serie se torne estacionária, como visto na decomposição na figura 1 os dados apresentam tendência de crescimento e para fazer com que a serie seja estacionária foram necessários aplicar uma diferenciação, esta função de difernça ja esta implementada no software R basta acionar a função ndifss() para checar se necessita de diferença simples e caso necessite aplica a diferença e nsdiffs() para checar a sazonalidade esta função retorna a quantidade de difenças para que a serie se torne estacionária. Após este procedimento verificou que apenas uma diferença simples e a serie se tornou estacionária.Na figura 2 tem-se as funções de autocorrelação(ACF) e autocorrelação parcial (PACF), estas funções são as representações gráficas dos coeficientes de autocorrelação em função dos retardos (BOX, JENKINS E REINSEL, 1994), na medida em que este gráfico demostram os lags escapam das bandas limites significa que ainda há autocorrelação e o processo não se tornou estacionários, e por estes gráficos em alguns casos pode-se deduzir a ordem do modelo, neste trabalho foi possivel deduzir apenas a ordem d=1 e D=0, não sendo possível obter um modelo completo de forma visual.


```{r}
x %>% ndiffs()
dados.diff<- diff(x)
dados.diff %>% ndiffs() #checando
# diferença sazonal
nsdiffs(dados.diff)
```



```{r}
par(mfrow=c(2,1))
acf(dados.diff, 60)

#autocorrelação parcial (PACF)
pacf(dados.diff, 60)
```
![acfpacfserie.png](grafico/acfpacfserie.png)


Nota-se no gráfico acima que não há índícios de autocorrelação espúria, ou seja, a série é estacionária. Há um conceito em que se ecolhe a ordem do modelo gráficamente, porém neste trabalho adotamos uma abordagem simplista, utilizando o próprio software para escolha do modelo então utilizou-se a função auto.arima do software com parâmetros d=1 e D=0, foram testados todas as possíveis combinações de zero a três dos parâmetros p,q,P,Q o modelo que obteve menor AIC corrigido foi selecionado como o melhor modelo para ser ajustado, logo para o modelo 1 obteve-se um ARIMA(2,1,0)X(1,0,0)12.



```{r}
# depois da escolha deduzir o modelo
MODELO1 = auto.arima(x, d=1, D= 0, max.p = 3,max.q = 3, max.P = 3, max.Q = 3, trace=FALSE, allowdrift = FALSE,
stepwise=FALSE)
MODELO1
```
Após a seleção do modelo para serie de dados originais realizou-se uma transformação de boxcox nos dados e realizou-se novamente o mesmo processo feito no passo anterior, porém para serie de dados histórica transformada.

```{r}
# aplicando a Transformação de BOXcox na serie
data_Bxcx <- BoxCox(x,lambda="auto")
plot(cbind(x,data_Bxcx))

```
Após a realização da transformação de boxcox avaliamos que não houve uma diferença elevada no padrão de tendência e nem no padrão da sazonalidade isso indica que para as analises gráficas não obteve diferenças significativa na dedução do modelo. Foram realizadas nesta serie teste de tendência, retornando o valor 1 sugerindo uma diferença simples para a serie. aplicada esta diferença a serie se tornou estacionária. Após este procedimento foram plotados o ACF e PACF da serie transformada a figura abaixo demonstra esses gráficos.

```{r}
#aplicando a diferença simples
data_Bxcx %>% ndiffs()
data_Bxcx.diff<- diff(data_Bxcx)
data_Bxcx.diff %>% ndiffs() #checando
nsdiffs(dados.diff)

```

```{r}
# depois da serie diferenciada analisar o acf e pacf
par(mfrow=c(2,1))
acf(data_Bxcx.diff, lag=4*12)
pacf(data_Bxcx.diff, lag=4*12)
```

![acfpacfseriet.png](grafico/acfpacfseriet.png)



Para identificar o modelo utilizou-se a abordagem simplista para estimar os parâmetros p,q,P,Q, sendo d=1 e D=0. Foram testados os valores de destes parâmetros de 0 a 3 e o modelo com melhor AICc foi escolhido foi um arima (2,1,1)x(1,0,0)12.

```{r}
# depois da escolha deduzir o modelo
MODELO2 = auto.arima(data_Bxcx, d=1, D= 0, max.p = 3,max.q = 3, max.P = 3, max.Q = 3, trace=FALSE, allowdrift = FALSE,
stepwise=FALSE)
MODELO2
```


O terceiro modelo foi proposto como alternativa para os modelos arimas, os modelos de alisamento exponencial ETS, para os dados sem a transformação de boxcox, o modelo obtido foi um ETS ajustado com tendencia multiplicativa, sazonalidade aditiva e o termo de erro multiplicativo, A figura demostra a serie decomposta no termo de level, serie original e a sazonalidade. 


```{r}
#modelo ETS
MODELO.ETS1 <- ets(x)

MODELO3<-MODELO.ETS1
MODELO3

#modelo ets(MAM)

```
```{r}
plot(MODELO.ETS1)
```




O quarto modelo proposto Foi o modelo ETS holt winter, com Erro multiplicativo, Tendência aditiva e sazonalidade Multiplicativa, a figura mostra a decomposição da serie em termos de level, sazonalidade e serie observada. O modelo foi gerado através de uma função automática no R que calcula os parâmetros do modelO.
Após o ajuste dos modelos foram avaliados os resíduos.
```{r}
#Modelo ETS com transformação de Boxcox

MODELO.ETS2 <- ets(x, lambda = "auto")
plot(MODELO.ETS2)
MODELO4<-MODELO.ETS2

#mODELO ETS(A,A,A)
```
```{r}
MODELO4
```


```{r}
#analise para o modelo 1
E1 <- MODELO1$residuals
E1 %>% plot(main = "Resíduos do modelo ARIMA(0,1,3)(2,0,0)[12] ")
```
![residuomodelo1.png](grafico/residuomodelo1.png)

Nas figuras acima demonstra o comportamento dos resíduos quanto a aleatoridade, o gráfico qqplot demonstra se os resíduos são normais e o acf demonstra se ainda ha autorrelação, ou seja, ele indica se os resíduos são ou não independentes, porém além da anlálise gráfica faz se necessaria um avaliação através de testos estatísticos sob hipótese nula a rejeição ou não da normalidade, independencia e estacionaridade. Para os modelos selecionados foram avaliados estas pressuposições através dos testes kpss (Testa estacionaridade), Box Ljung teste (Testa a idependencia dos resíduos) e shapiro wilks (Testa a normaliade dos resíduos)
```{r}
par(mfrow=c(2,1))
qqnorm(E1); qqline(E1); acf(E1, lag.max=12*5)
```

```{r}
E1=MODELO1$residuals
tseries::kpss.test(E1)
Box.test(E1, lag=15, type = "Ljung-Box")
shapiro.test(E1)
```

Para o modelo 1 (Arima sazonal), os resíduos passaram no teste a 5% de significancia para estacionaridade e independencia, para o teste de normalidade não foi possível rejeitar a hipótese nula, ou seja, os resíduos não são normais, porém se o nível de sinificancia a ser considerado for de 1% todos os pressupostos seriam atendidos.

```{r}
E2=MODELO2$residuals
E2 <- MODELO2$residuals %>% plot(main = "Residuo Modelo2")
```
![residuomodelo2.png](grafico/residuomodelo2.png)

```{r}
E3=MODELO3$residuals
E3 <- MODELO3$residuals %>% plot(main = "Residuo Modelo3")
```
![residuomodelo3.png](grafico/residuomodelo3.png)

```{r}
E4=MODELO4$residuals
E4 <- MODELO4$residuals %>% plot(main = "Residuo Modelo4")
```
![residuomodelo4.png](grafico/residuomodelo4.png)

Gráficamente parece que a normalidade não foi bem comportada em todos os casos, de qualquer optamos por realizar alguns teste tanto para normalidade quanto para idependência. Adotamos teste de Shapiro e teste de Ljung Box. Caso a normalidade não for atendida proceguimos com as análises. Para estimativas pontuais não ha problemas o pressuposto de normalidade ser quebrado, o que implica é na estimativa intervalar e para contornar este problema pode construir intervalos de confiança via booststrap

```{r}
MODELO2
E2=MODELO2$residuals
tseries::kpss.test(E2)
Box.test(E2, lag=15, type = "Ljung-Box")
shapiro.test(E2)

```
Para o modelo 2, aplicando os mesmo testes tem-se que os resíduos foram independentes, a serie é estacionária porém os resíduos não são normais a 5%.


```{r}
E3=MODELO3$residuals
tseries::kpss.test(E3)

## Teste de independência

Box.test(E3, lag=15, type = "Ljung-Box")

## Teste de normalidade
shapiro.test(E3)
```

```{r}
E4=MODELO4$residuals
tseries::kpss.test(E4)

## Teste de independência

Box.test(E4, lag=15, type = "Ljung-Box")

## Teste de normalidade
shapiro.test(E4)
```

# Janela deslizante
A janela deslizante é um procedimento usado para analisar dados sequenciais ao longo do tempo. Funciona criando subconjuntos dos dados com um tamanho fixo, movendo essa janela ao longo da série temporal para capturar diferentes segmentos. A cada passo, a janela se desloca uma unidade (por exemplo, um dia, uma semana, um mês) para frente, incluindo um novo ponto de dados e excluindo o ponto mais antigo. Esse método permite suavizar flutuações, identificar tendências, padrões e anomalias, além de ser útil em previsões e na modelagem de séries temporais. A janela deslizante é frequentemente aplicada em análises financeiras, meteorológicas e de dados de sensores, entre outras áreas. Dito isto nos aplicamos a janela deslizante e para cada ponto calculamos a diferença absoluta (MAE - erro médio absoluto) entre o valor predito e o valor observado e plotamos no gráfico para os 4 modelos.

```{r, cache=TRUE}
n=length(x) 
corte=n-14

################
#

f_arima1 <- function(y, h){
  fit <- Arima(y, order=c(2, 1, 0), seasonal=c(1, 0, 0), include.mean=F, lambda=NULL)
  forecast(fit, h,bootstrap=T)
}
f_arima2 <- function(y, h){
  fit <- Arima(y, order=c(2, 1, 1), seasonal=c(1, 0, 0), include.mean=F, lambda='auto')
  forecast(fit, h,bootstrap=T)
}
f_ets1 <- function(y, h){
  fit <- ets(y)
  forecast(fit, h,bootstrap=T)
}
f_ets2 <- function(y, h){
  fit <- ets(y, lambda='auto')
  forecast(fit, h,bootstrap=T)
}
CV_arima1 <- x %>% tsCV(forecastfunction=f_arima1, h=5, initial=n-14)
CV_arima2 <- x %>% tsCV(forecastfunction=f_arima2, h=5, initial=n-14)
CV_ets1 <- x %>% tsCV(forecastfunction=f_ets1, h=5, initial=n-14)
CV_ets2 <- x %>% tsCV(forecastfunction=f_ets2, h=5, initial=n-14)
MAE_arima1 <- CV_arima1 %>% abs() %>% colMeans(na.rm=T)
MAE_arima2 <- CV_arima2 %>% abs() %>% colMeans(na.rm=T)
MAE_ets1 <- CV_ets1 %>% abs() %>% colMeans(na.rm=T)
MAE_ets2 <- CV_ets2 %>% abs() %>% colMeans(na.rm=T)
tab <- cbind(MAE_arima1, MAE_arima2, MAE_ets1, MAE_ets2)
tab %>%
  kable(
    col.names=c('ARIMA', 'ARIMA + Box-Cox', 'ETS', 'ETS + Box-Cox'),
    caption='MAE por horizonte de predição.',
    digits=0,
    format.args=list(decimal.mark=',', scientific=F),
    align='c'
  ) %>%
  kable_styling(
    position='center',
    bootstrap_options=c('striped', 'hover', 'condensed', 'responsive')
  )
tab_plot <- tab %>%
  as.data.frame() %>%
  mutate(Horizonte=1:5) %>%
  gather(key='Modelo', value='MAE', -Horizonte)
tab_plot %>%
  ggplot(aes(x=Horizonte, y=MAE)) +
  geom_line(aes(color=Modelo)) +
  scale_color_manual(
    values=c('black', 'red', '#0000AA', 'darkgreen'),
    breaks=c('MAE_arima1', 'MAE_arima2', 'MAE_ets1', 'MAE_ets2'),
    labels=c('ARIMA', 'ARIMA + Box-Cox', 'ETS', 'ETS + Box-Cox')
  ) +
  theme_bw()

```
![maecorreto.png](grafico/maecorreto.png)

A escolha ideal para os modelos são aqueles que obtem menor erro médio absoluto, nota-se que no desempenho do modelo temos dois cenario para a serie suavizada (Trasnformação de boxcox) temos os menores erros por conta da escala dos dados. Mas para este trabalho fizermos a previsão pontual e intervalar para os 4 modelos para fins de testes. 
As previsões pontual e intervalares foram obtidas por conta do espaço de tabelas omitiremos os resultados das saídas das funções abaixo deixaremos apenas as representações gráficas de previsão


```{r, cache=TRUE}
h=48
preds1 <- forecast(MODELO1, h=h, level=95)
preds2 <- forecast(MODELO2, h=h, level=95)
preds3 <- forecast(MODELO3, h=h, level=95)
preds4 <- forecast(MODELO4, h=h, level=95)
pontual <- t(cbind(xx, preds1$mean, preds2$mean, preds3$mean, preds4$mean))
colnames(pontual) <- 1:h
row.names(pontual) <- c('Observado', 'ARIMA', 'ARIMA + Box-Cox', 'ETS', 'ETS + Box-Cox')
pontual %>%
  kable(
    caption='Previsões pontuais por horizonte de predição.',
    digits=0,
    format.args=list(decimal.mark=',', scientific=F),
    align='c'
  ) %>%
  kable_styling(
    position='center',
    bootstrap_options=c('striped', 'hover', 'condensed', 'responsive')
  )

```



```{r, cache=TRUE}
## Ajuste e previsão do modelo e cálculo do erro absoluto médio da previsão
# tables



intervalares <- t(cbind(xx, preds1$lower, preds1$upper, preds2$lower, preds2$upper,
                        preds3$lower, preds3$upper, preds4$lower, preds4$upper))
colnames(intervalares) <- 1:h
row.names(intervalares) <- c('Observado', 'ARIMA Inf', 'ARIMA Sup', 'ARIMA + Box-Cox Inf',
                             'ARIMA + Box-Cox Sup', 'ETS Inf', 'ETS Sup', 'ETS + Box-Cox Inf',
                             'ETS + Box-Cox Sup')
intervalares %>%
  kable(
    caption='Previsões intervalares de 95% de confiança por horizonte de predição.',
    digits=0,
    format.args=list(decimal.mark=',', scientific=F),
    align='c'
  ) %>%
  kable_styling(
    position='center',
    bootstrap_options=c('striped', 'hover', 'condensed', 'responsive')
  )


```


```{r, cache=TRUE}

# plots
plot_preds <- function(mod, nome='') {
  vec <- c(nome, 'Observado')
  cores <- c('#0000AA', 'red')
  names(cores) <- vec
  preds <- forecast(mod, h=h, level=95)
  plot_obj <- x %>%
    autoplot() + xlab('Ano') + ylab('Valor Observado') + theme_bw() +
    autolayer(preds, series=nome) +
    autolayer(xx, series='Observado') +
    scale_colour_manual(
      values=cores,
      breaks=vec,
      name='')
  return(plot_obj)
}
plot_preds(MODELO1, 'ARIMA')








```
![previmodelo1.png](grafico/prev1.png)



```{r, cache=TRUE}
plot_preds(MODELO2, 'ARIMA + Box-Cox')


```
![previmodelo2.png](grafico/previmodelo2.png)


```{r, cache=TRUE, warning=FALSE}
plot_preds(MODELO3, 'ETS')
```
![previmodelo3.png](grafico/previmodelo3.png)



```{r, cache=TRUE, warning=FALSE}
plot_preds(MODELO4, 'ETS + Box-Cox')
```

![previmodelo4.png](grafico/previmodelo4.png)

Com isso concluimos a previsão dos 4 modelos observe que o ajuste do modelo 1, 3 e 4 foram razoavelmente bom, ja o modelo 2 é totalmente inadequado para uso.  para finalizar o trabalho foram testado diversos outros modelos como métricas de Benchmark e utizou-se o menor erro médio absoluto para dizer qual modelo apresentaria melhor perfomace. Omitimos a tabela neste markdown porém o código abaixo gera as métricas de Benchmark

```{r, warning=FALSE, cache=TRUE}
# Benchmark comparison
preds <- list(
  'ARIMA' = forecast(MODELO1, h=h),
  'ARIMA + Box-Cox' = forecast(MODELO2, h=h),
  'ETS' = forecast(MODELO3, h=h),
  'ETS + Box-Cox' = forecast(MODELO4, h=h),
  'auto.arima' = forecast(auto.arima(x), h=h),
  'SES' = ses(x, h=h),
  'Holt' = holt(x, h=h),
  'sltf' = stlf(x, h=h),
  'BATS' = forecast(bats(x), h=h),
  'TBATS' = forecast(tbats(x), h=h),
  'Bagged ETS' = forecast(baggedETS(x), h=h)
)
mae <- unlist(lapply(preds, function(m) return(mean(abs(xx - m$mean)))))
final <- data.frame(MAE=mae)
final %>%
  kable(
    caption='MAE nos dados de teste.',
    digits=0,
    format.args=list(decimal.mark=',', scientific=F),
    align='c'
  ) %>%
  kable_styling(
    position='center',
    bootstrap_options=c('striped', 'hover', 'condensed', 'responsive')
  )
```

# Referencias Bibliográficas
BOX, G.E., JENKINS, G.M. and REINSEL, G. C. (1994). Time Series: Forecasting and 
Control (3rd edition). Prentice Hall.

HYNDMAN, R. J. et al. Previsão com suavização exponencial: A abordagem do espaço de estados. Berlim: Springer-Verlag, 2008

R Development Core Team (2009). R: A language and environment for statistical computing. R Foundation for Statistical Computing, Vienna, Austria. ISBN 3-900051-07-0, URL http://www.R-project.org.
