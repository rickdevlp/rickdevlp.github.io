# ✏️ Criação de metas - Estudo de caso

Este documento tem por objetivo demonstrar uma situação real de criação de metas de faturamento visando um crescimento organico com base em estudo geográfico utilizando o banco de dados AdventureWorks para exemplificar.

Você pode encontrar esse banco de dados através do link:
https://learn.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver17&tabs=ssms

## 📌 Solução

### 1. Identificar o percentual de faturamento em que cada região representou no ano anterior
<br>

````sql
;with faturamento as (
    select
        sum(subtotal)                                                   as Faturamento

    from sales.salesorderheader
    where
        orderdate between datefromparts(year(getdate()) - 1, 1, 1) and datefromparts(year(getdate()) - 1, 12, 31)
)
select
    a.territoryid                                                       as TerritoryId,
    b.name                                                              as Regiao,
    sum(subtotal)                                                       as TotalVendas,
    round(sum(subtotal) / (select faturamento from faturamento), 2)     as PercentualVendas

from sales.salesorderheader a
join sales.salesterritory b on a.territoryid = b.territoryid
where
    orderdate between datefromparts(year(getdate()) - 1, 1, 1) and datefromparts(year(getdate()) - 1, 12, 31)
group by a.territoryid, b.name
order by 1
````
**Resultado esperado:**

<img src="../../graphs/graficopercmetas.jpg" alt="Gráfico de Percentual de Metas">
<br>

### 2. Identificar a curva de sazonalidade para compreender quais são os meses com maior e menor movimento de faturamento por região
<br>

````sql
select
    a.territoryid                                                       as territoryid,
    b.name                                                              as regiao,
    format(orderdate, 'yyyyMM')                                         as mes,
    sum(subtotal)                                                       as totalvendas,
    round(
        sum(subtotal) / sum(sum(subtotal)) over(partition by a.territoryid), 
    4)                                                                  as curva_sazonalidade

from sales.salesorderheader a
join sales.salesterritory b on a.territoryid = b.territoryid
where
    orderdate between datefromparts(year(getdate()) - 1, 1, 1) and datefromparts(year(getdate()) - 1, 12, 31)
group by a.territoryid, b.name, format(orderdate, 'yyyyMM')
order by 1, 3
````
**Resultado esperado:**

<img src="../../graphs/graficopercmetas2.jpg" alt="Gráfico de Percentual de Metas 2">
<br>

### 3. Realizar um estudo para compreender o percentual de crescimento/decrescimento dos últimos 5 anos YoY (Year over Year)
<br>

````sql
;with vendas_anuais as (
    select
        grouping(b.name)                                                 as total,
        a.territoryid,
        isnull(b.name, 'Total Geral')                                    as regiao,
        year(a.orderdate)                                                as ano,
        sum(a.subtotal)                                                  as faturamento_atual

    from sales.salesorderheader a
    join sales.salesterritory b on a.territoryid = b.territoryid
    where 
        year(a.orderdate) >= year(getdate()) - 5
    group by rollup (year(a.orderdate), (a.territoryid, b.name))
    having year(a.orderdate) is not null
),
comparativo_yoy as (
    select
        total,
        territoryid,
        regiao,
        ano,
        faturamento_atual,
        lag(faturamento_atual) over (partition by regiao order by ano)   as faturamento_anterior

    from vendas_anuais
)
select
    regiao,
    ano,
    format(faturamento_atual, 'c', 'pt-br')                              as faturamento_atual,
    format(faturamento_anterior, 'c', 'pt-br')                           as faturamento_anterior,
    round(
        ((faturamento_atual - faturamento_anterior) / nullif(faturamento_anterior, 0)) * 100, 
    2)                                                                   as perc_crescimento_yoy

from comparativo_yoy
where ano is not null
order by total,regiao, ano desc;
````
<br>

📈 Análise de Faturamento Consolidado e YoY
<br>
Abaixo, apresento o desempenho global de vendas. O destaque fica para o crescimento sólido de 38,18% em 2024, seguido por um ajuste no volume de transações em 2025.

| Métrica Geral | Ano | Faturamento | YoY (%) |
| :--- | :---: | :--- | :---: |
| 🔴 **Total Geral** | **2025** | **R$ 20.008.518,36** | `▼ -54,18%` |
| 🟢 **Total Geral** | **2024** | **R$ 43.671.889,50** | `▲ +38,18%` |
| 🟢 **Total Geral** | **2023** | **R$ 31.604.921,95** | `▲ +117,05%` |
| ⚪ **Total Geral** | **2022** | **R$ 14.561.051,59** | `-` |

<br>

**Resultado esperado:**

<details>
  <summary>📂 Clique aqui para expandir a tabela de crescimento YoY por região</summary>

| regiao | ano | faturamento_atual | faturamento_anterior | perc_crescimento_yoy |
|:--- |:---:|:--- |:--- |:---:|
| Australia | 2025 | R$ 2.751.945,92 | R$ 4.246.450,55 | -35,19 |
| Australia | 2024 | R$ 4.246.450,55 | R$ 2.114.048,37 | 100,86 |
| Australia | 2023 | R$ 2.114.048,37 | R$ 1.542.891,12 | 37,01 |
| Australia | 2022 | R$ 1.542.891,12 | NULL | NULL |
| Canada | 2025 | R$ 2.389.120,60 | R$ 6.231.209,96 | -61,65 |
| Canada | 2024 | R$ 6.231.209,96 | R$ 5.580.848,05 | 11,65 |
| Canada | 2023 | R$ 5.580.848,05 | R$ 2.154.591,84 | 159,02 |
| Canada | 2022 | R$ 2.154.591,84 | NULL | NULL |
| Central | 2025 | R$ 955.864,73 | R$ 2.994.225,38 | -68,07 |
| Central | 2024 | R$ 2.994.225,38 | R$ 2.758.256,66 | 8,55 |
| Central | 2023 | R$ 2.758.256,66 | R$ 1.200.662,23 | 129,72 |
| Central | 2022 | R$ 1.200.662,23 | NULL | NULL |
| France | 2025 | R$ 1.664.041,62 | R$ 3.816.543,33 | -56,39 |
| France | 2024 | R$ 3.816.543,33 | R$ 1.557.152,94 | 145,09 |
| France | 2023 | R$ 1.557.152,94 | R$ 213.817,76 | 628,26 |
| France | 2022 | R$ 213.817,76 | NULL | NULL |
| Germany | 2025 | R$ 1.548.206,97 | R$ 2.570.269,35 | -39,76 |
| Germany | 2024 | R$ 2.570.269,35 | R$ 550.070,74 | 367,26 |
| Germany | 2023 | R$ 550.070,74 | R$ 246.860,54 | 122,82 |
| Germany | 2022 | R$ 246.860,54 | NULL | NULL |
| Northeast | 2025 | R$ 778.377,79 | R$ 2.631.259,60 | -70,41 |
| Northeast | 2024 | R$ 2.631.259,60 | R$ 2.773.503,83 | -5,12 |
| Northeast | 2023 | R$ 2.773.503,83 | R$ 756.233,26 | 266,75 |
| Northeast | 2022 | R$ 756.233,26 | NULL | NULL |
| Northwest | 2025 | R$ 2.994.492,87 | R$ 6.018.432,58 | -50,24 |
| Northwest | 2024 | R$ 6.018.432,58 | R$ 4.254.635,19 | 41,45 |
| Northwest | 2023 | R$ 4.254.635,19 | R$ 2.817.381,91 | 51,01 |
| Northwest | 2022 | R$ 2.817.381,91 | NULL | NULL |
| Southeast | 2025 | R$ 875.604,21 | R$ 2.399.947,38 | -63,51 |
| Southeast | 2024 | R$ 2.399.947,38 | R$ 2.651.805,10 | -9,49 |
| Southeast | 2023 | R$ 2.651.805,10 | R$ 1.952.298,38 | 35,82 |
| Southeast | 2022 | R$ 1.952.298,38 | NULL | NULL |
| Southwest | 2025 | R$ 3.966.506,68 | R$ 9.121.931,65 | -56,51 |
| Southwest | 2024 | R$ 9.121.931,65 | R$ 7.786.323,61 | 17,15 |
| Southwest | 2023 | R$ 7.786.323,61 | R$ 3.309.847,66 | 135,24 |
| Southwest | 2022 | R$ 3.309.847,66 | NULL | NULL |
| United Kingdom | 2025 | R$ 2.084.356,96 | R$ 3.641.619,72 | -42,76 |
| United Kingdom | 2024 | R$ 3.641.619,72 | R$ 1.578.277,47 | 130,73 |
| United Kingdom | 2023 | R$ 1.578.277,47 | R$ 366.466,89 | 330,67 |
| United Kingdom | 2022 | R$ 366.466,89 | NULL | NULL |
| Total Geral | 2025 | R$ 20.008.518,36 | R$ 43.671.889,50 | -54,18 |
| Total Geral | 2024 | R$ 43.671.889,50 | R$ 31.604.921,95 | 38,18 |
| Total Geral | 2023 | R$ 31.604.921,95 | R$ 14.561.051,59 | 117,05 |
| Total Geral | 2022 | R$ 14.561.051,59 | NULL | NULL |
|  | 

</details>

<br>

# 🎯 Planejamento Estratégico 2026

### 📊 Média Anual Consolidada (Baseline)

 **R$ 27.461.595,35**  
 *Valor médio calculado com base no histórico de faturamento consolidado (2022-2025).*
 >

---

### 🧮 Memória de Cálculo: Recuperação da Média
Para retornar ao patamar médio de operação e mitigar a retração observada no último período, calculamos a variação percentual necessária:

<br>

*   **Diferença bruta:** `R$ 27.461.595,35` - `R$ 20.008.518,36` = **`R$ 7.453.076,99`**
*   **Cálculo de Recuperação:** `R$ 7.453.076,99` / `R$ 20.008.518,36` ≈ **`37,25%`**
*   **Status:** Ponto de equilíbrio para retomada da média histórica.

---

### 🚀 Definição da Meta para 2026
Para o ciclo de 2026, foi estabelecida uma meta de crescimento de **40%**. 

Esta métrica foi definida através do cruzamento da **Média de Recuperação (37,25%)** com um incremento de **2,75% em eficiência operacional**. O objetivo estratégico é superar a média histórica e reestabelecer a trajetória de crescimento observada nos anos anteriores.

#### 📉 Projeção de Cenários
| Cenário | % Crescimento | Faturamento Projetado | Objetivo Estratégico |
| :--- | :---: | :--- | :--- |
| **Base (Média)** | `37,25%` | `R$ 27.461.595,35` | Estabilização e Retomada |
| **Alvo (Estratégico)** | **`40,00%`** | **`R$ 28.011.925,70`** | **Expansão e Performance** |

---
<br>

### 4. Após levantar todas as informações necessárias, conseguimos distribuir as metas em cada mês de acordo com a sua respectiva sazonalidade.
<br>

````sql
;with sazonalidade as (
    select
        a.territoryid                                                    as territoryid,
        b.name                                                           as regiao,
        format(orderdate, 'yyyyMM')                                      as mes,
        sum(subtotal)                                                    as totalvendas_ant,
        sum(sum(subtotal)) over(partition by a.territoryid)              as total_anual_regiao,
        cast(sum(subtotal) as float) / cast(sum(sum(subtotal)) over(partition by a.territoryid) as float) as curva_sazonalidade
    from sales.salesorderheader a
    join sales.salesterritory b on a.territoryid = b.territoryid
    where
        orderdate between datefromparts(year(getdate()) - 1, 1, 1) and datefromparts(year(getdate()) - 1, 12, 31)
    group by a.territoryid, b.name, format(orderdate, 'yyyyMM'))
select     
    territoryid,
    regiao,
    cast(mes as int) + 100                                                as mes,
    format(totalvendas_ant, 'c', 'pt-br')                                 as totalvendas_ant,
    format(totalvendas_ant * 1.4, 'c', 'pt-br')                           as meta,
    format(curva_sazonalidade, 'p2', 'pt-br')                             as peso_sazonalidade_perc,
    format((curva_sazonalidade * 0.4 * total_anual_regiao), 'c', 'pt-br') as valor_ajustado_crescimento
from sazonalidade;
````
<br>

**Resultado esperado:**

### 📈 Resumo Executivo: Metas 2026 por Região

| Região | Faturamento Base (2025) | Meta Consolidada (2026) | Crescimento Previsto (R$) | Representatividade (%) |
|:---|---:|---:|---:|---:|
| **Southwest** | R$ 3.966.506,68 | R$ 5.553.109,34 | R$ 1.586.602,67 | 19,82% |
| **Northwest** | R$ 2.994.492,87 | R$ 4.192.290,03 | R$ 1.197.797,15 | 14,97% |
| **Australia** | R$ 2.751.945,91 | R$ 3.852.724,28 | R$ 1.100.778,36 | 13,75% |
| **Canada** | R$ 2.389.120,61 | R$ 3.344.768,83 | R$ 955.648,24 | 11,94% |
| **United Kingdom** | R$ 2.084.356,97 | R$ 2.918.099,74 | R$ 833.742,77 | 10,42% |
| **France** | R$ 1.664.041,62 | R$ 2.329.658,26 | R$ 665.616,64 | 8,32% |
| **Germany** | R$ 1.548.206,97 | R$ 2.167.489,75 | R$ 619.282,78 | 7,74% |
| **Central** | R$ 955.864,73 | R$ 1.338.210,63 | R$ 382.345,90 | 4,78% |
| **Southeast** | R$ 875.604,20 | R$ 1.225.845,90 | R$ 350.241,68 | 4,38% |
| **Northeast** | R$ 778.377,79 | R$ 1.089.728,92 | R$ 311.351,12 | 3,89% |
| **TOTAL GERAL** | **R$ 20.008.518,35** | **R$ 28.011.925,68** | **R$ 8.003.407,31** | **100,00%** |


<details>
  <summary>📂 Clique aqui para expandir a tabela com as metas distribuidas por região</summary>
    
| ID | Região | Mês | Total Vendas Ant. | Meta (2026) | Peso Sazonal (%) | Valor Ajustado |
|---:|:---|:---:|---:|---:|---:|---:|
| 1 | Northwest | 202601 | R$ 607.727,83 | R$ 850.818,97 | 20,29% | R$ 243.091,13 |
| 1 | Northwest | 202602 | R$ 565.133,58 | R$ 791.187,01 | 18,87% | R$ 226.053,43 |
| 1 | Northwest | 202603 | R$ 764.651,83 | R$ 1.070.512,57 | 25,54% | R$ 305.860,73 |
| 1 | Northwest | 202604 | R$ 791.969,19 | R$ 1.108.756,86 | 26,45% | R$ 316.787,68 |
| 1 | Northwest | 202605 | R$ 255.561,44 | R$ 357.786,02 | 8,53% | R$ 102.224,58 |
| 1 | Northwest | 202606 | R$ 9.449,00 | R$ 13.228,60 | 0,32% | R$ 3.779,60 |
| 2 | Northeast | 202601 | R$ 194.783,81 | R$ 272.697,34 | 25,02% | R$ 77.913,53 |
| 2 | Northeast | 202602 | R$ 174.119,55 | R$ 243.767,38 | 22,37% | R$ 69.647,82 |
| 2 | Northeast | 202603 | R$ 166.341,96 | R$ 232.878,74 | 21,37% | R$ 66.536,78 |
| 2 | Northeast | 202604 | R$ 243.081,53 | R$ 340.314,14 | 31,23% | R$ 97.232,61 |
| 2 | Northeast | 202605 | R$ 50,94 | R$ 71,32 | 0,01% | R$ 20,38 |
| 3 | Central | 202601 | R$ 256.402,82 | R$ 358.963,95 | 26,82% | R$ 102.561,13 |
| 3 | Central | 202602 | R$ 151.633,70 | R$ 212.287,18 | 15,86% | R$ 60.653,48 |
| 3 | Central | 202603 | R$ 249.607,25 | R$ 349.450,15 | 26,11% | R$ 99.842,90 |
| 3 | Central | 202604 | R$ 298.183,67 | R$ 417.457,14 | 31,20% | R$ 119.273,47 |
| 3 | Central | 202605 | R$ 37,29 | R$ 52,21 | 0,00% | R$ 14,92 |
| 4 | Southwest | 202601 | R$ 784.696,69 | R$ 1.098.575,37 | 19,78% | R$ 313.878,68 |
| 4 | Southwest | 202602 | R$ 868.434,37 | R$ 1.215.808,11 | 21,89% | R$ 347.373,75 |
| 4 | Southwest | 202603 | R$ 904.593,03 | R$ 1.266.430,24 | 22,81% | R$ 361.837,21 |
| 4 | Southwest | 202604 | R$ 1.004.963,53 | R$ 1.406.948,94 | 25,34% | R$ 401.985,41 |
| 4 | Southwest | 202605 | R$ 395.283,03 | R$ 553.396,24 | 9,97% | R$ 158.113,21 |
| 4 | Southwest | 202606 | R$ 8.536,03 | R$ 11.950,44 | 0,22% | R$ 3.414,41 |
| 5 | Southeast | 202601 | R$ 196.595,55 | R$ 275.233,78 | 22,45% | R$ 78.638,22 |
| 5 | Southeast | 202602 | R$ 140.974,79 | R$ 197.364,71 | 16,10% | R$ 56.389,92 |
| 5 | Southeast | 202603 | R$ 265.875,42 | R$ 372.225,59 | 30,36% | R$ 106.350,17 |
| 5 | Southeast | 202604 | R$ 272.044,48 | R$ 380.862,28 | 31,07% | R$ 108.817,79 |
| 5 | Southeast | 202605 | R$ 38,98 | R$ 54,57 | 0,00% | R$ 15,59 |
| 5 | Southeast | 202606 | R$ 74,98 | R$ 104,97 | 0,01% | R$ 29,99 |
| 6 | Canada | 202601 | R$ 557.273,93 | R$ 780.183,50 | 23,33% | R$ 222.909,57 |
| 6 | Canada | 202602 | R$ 463.353,97 | R$ 648.695,55 | 19,39% | R$ 185.341,59 |
| 6 | Canada | 202603 | R$ 509.015,69 | R$ 712.621,96 | 21,31% | R$ 203.606,28 |
| 6 | Canada | 202604 | R$ 719.801,26 | R$ 1.007.721,76 | 30,13% | R$ 287.920,50 |
| 6 | Canada | 202605 | R$ 129.609,60 | R$ 181.453,44 | 5,42% | R$ 51.843,84 |
| 6 | Canada | 202606 | R$ 10.066,16 | R$ 14.092,62 | 0,42% | R$ 4.026,46 |
| 7 | France | 202601 | R$ 244.887,56 | R$ 342.842,58 | 14,72% | R$ 97.955,02 |
| 7 | France | 202602 | R$ 179.874,97 | R$ 251.824,96 | 10,81% | R$ 71.949,99 |
| 7 | France | 202603 | R$ 770.150,97 | R$ 1.078.211,36 | 46,28% | R$ 308.060,39 |
| 7 | France | 202604 | R$ 272.268,16 | R$ 381.175,42 | 16,36% | R$ 108.907,26 |
| 7 | France | 202605 | R$ 193.594,91 | R$ 271.032,87 | 11,63% | R$ 77.437,96 |
| 7 | France | 202606 | R$ 3.265,05 | R$ 4.571,07 | 0,20% | R$ 1.306,02 |
| 8 | Germany | 202601 | R$ 285.501,64 | R$ 399.702,30 | 18,44% | R$ 114.200,66 |
| 8 | Germany | 202602 | R$ 317.763,94 | R$ 444.869,52 | 20,52% | R$ 127.105,58 |
| 8 | Germany | 202603 | R$ 396.192,41 | R$ 554.669,37 | 25,59% | R$ 158.476,96 |
| 8 | Germany | 202604 | R$ 340.935,16 | R$ 477.309,22 | 22,02% | R$ 136.374,06 |
| 8 | Germany | 202605 | R$ 204.448,46 | R$ 286.227,84 | 13,21% | R$ 81.779,38 |
| 8 | Germany | 202606 | R$ 3.365,36 | R$ 4.711,50 | 0,22% | R$ 1.346,14 |
| 9 | Australia | 202601 | R$ 632.175,27 | R$ 885.045,38 | 22,97% | R$ 252.870,11 |
| 9 | Australia | 202602 | R$ 419.688,75 | R$ 587.564,26 | 15,25% | R$ 167.875,50 |
| 9 | Australia | 202603 | R$ 576.172,49 | R$ 806.641,49 | 20,94% | R$ 230.469,00 |
| 9 | Australia | 202604 | R$ 642.917,66 | R$ 900.084,72 | 23,36% | R$ 257.167,06 |
| 9 | Australia | 202605 | R$ 472.262,88 | R$ 661.168,03 | 17,16% | R$ 188.905,15 |
| 9 | Australia | 202606 | R$ 8.728,86 | R$ 12.220,40 | 0,32% | R$ 3.491,54 |
| 10 | United Kingdom | 202601 | R$ 516.382,61 | R$ 722.935,65 | 24,77% | R$ 206.553,04 |
| 10 | United Kingdom | 202602 | R$ 284.901,69 | R$ 398.862,37 | 13,67% | R$ 113.960,68 |
| 10 | United Kingdom | 202603 | R$ 385.300,49 | R$ 539.420,68 | 18,49% | R$ 154.120,19 |
| 10 | United Kingdom | 202604 | R$ 636.594,11 | R$ 891.231,75 | 30,54% | R$ 254.637,64 |
| 10 | United Kingdom | 202605 | R$ 257.171,96 | R$ 360.040,74 | 12,34% | R$ 102.868,78 |
| 10 | United Kingdom | 202606 | R$ 4.006,11 | R$ 5.608,55 | 0,19% | R$ 1.602,44 |
|    |
</details>
    
Nota Metodológica:
Os valores de 2026 foram projetados aplicando um markup de crescimento de 40% sobre o faturamento realizado no primeiro semestre de 2025. A distribuição mensal respeita a curva de sazonalidade histórica de cada território, garantindo que o esforço de vendas esteja alinhado com o comportamento de compra regional.

<br>

### EXTRA. Inserção de dados prévios de 2026
<br>

Nota: A versão do banco de dados que estou utilizando para esse estudo é o AdventureWorks2025, que possui dados apenas até 2025. Como a finalidade é mostrar como a meta se comporta em relação às vendas, criei uma
simulação randomica dos dados de 2026 até a data atual. Como o objetivo desse estudo é didático, abaixo demonstro como realizei essa inserção. 

⚠️ Importante!! Para a inserção dos dados é importante sempre utilizar transac para validar se o que será inserido está correto então, caso queira realizar essa mesma inserção, descomente a linha do commit e comente
a linha do rollback apenas quando tiver a certeza de que os dados satisfazem a sua necessidade.
<br>

````sql
begin tran;

insert into sales.salesorderheader (
    revisionnumber, orderdate, duedate, shipdate, status, onlineorderflag,
    purchaseordernumber, accountnumber, customerid, salespersonid, territoryid,
    billtoaddressid, shiptoaddressid, shipmethodid, creditcardid, creditcardapprovalcode,
    currencyrateid, subtotal, taxamt, freight, comment, rowguid, modifieddate
)
select 
    revisionnumber,
    dateadd(year, 1, orderdate) as orderdate, 
    dateadd(year, 1, duedate) as duedate,
    dateadd(year, 1, shipdate) as shipdate,
    status,
    onlineorderflag,
    purchaseordernumber,
    accountnumber,
    customerid,
    salespersonid,
    territoryid,
    billtoaddressid,
    shiptoaddressid,
    shipmethodid,
    creditcardid,
    creditcardapprovalcode,
    currencyrateid,
    subtotal * (0.7 + (rand(checksum(newid())) * 0.6)) as subtotal,
    taxamt * (0.7 + (rand(checksum(newid())) * 0.6)) as taxamt,
    freight * (0.7 + (rand(checksum(newid())) * 0.6)) as freight,
    'Simulação Vendas 2026' as comment,
    newid() as rowguid,
    getdate() as modifieddate
from sales.salesorderheader
where 
    orderdate >= '2025-01-01' 
    and orderdate <= dateadd(year, -1, getdate());


select top 100 *
from sales.salesorderheader
where orderdate >= '2026-01-01';

--commit tran; 
rollback tran;
````
<br>
