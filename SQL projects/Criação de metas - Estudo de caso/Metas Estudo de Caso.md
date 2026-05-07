# ✏️ Criação de metas - Estudo de caso

Este documento tem por objetivo demonstrar uma situação real de criação de metas de faturamento visando um crescimento organico com base em estudo geográfico utilizando o banco de dados AdventureWorks para exemplificar.

Você pode encontrar esse banco de dados através do link:
https://learn.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver17&tabs=ssms

## 📌 Solução

### 1. Identificar o percentual de faturamento em que cada região representou no ano anterior

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
group by
    a.territoryid,
    b.name
order by 1
````
**Resultado esperado:**

<img src="../graphs/graficopercmetas.jpg" alt="Gráfico de Percentual de Metas">
