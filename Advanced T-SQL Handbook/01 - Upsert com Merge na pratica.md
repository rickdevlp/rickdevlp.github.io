## 🔄 Operações de UPSERT com MERGE e Auditoria de Log (OUTPUT $action)

Este documento tem por objetivo demonstrar uma situação real de utilização do Upsert com Merge. No contexto corporativo, eu costumo utilizar essas clausulas para gerar logs de acompanhamento que me indicarão se as tabelas que eu preciso foram atualizadas ou não.
  Com isso, eu consigo automatizar os meus processos de forma que, se houver um insert no meu log, significa que novos periodos foram atualizados e, caso seja feito apenas um update, é porque as tabelas não foram atualizadas ainda e apenas a data do processamento foi atualizada.


## 📌 Contexto e Aplicação Prática
Em rotinas de Carga e Carga Incremental (ETL/ELT), é muito comum a necessidade de atualizar registros existentes e inserir novos registros em uma única transação atômica. O comando `MERGE` do T-SQL resolve esse cenário de **UPSERT** de forma bastante performática.

Além disso, utilizando a cláusula `OUTPUT` acoplada ao qualificador `$action`, conseguimos rastrear exatamente quais linhas sofreram `UPDATE` e quais foram alvos de `INSERT`, permitindo alimentar tabelas de **log de auditoria** ou validar o comportamento do pipeline em tempo de execução.

---

## 🎯 Cenário de Negócio (AdventureWorks)
Neste exemplo, realizamos o controle e validação de totais de vendas diárias agrupadas por território (`TerritoryID`). 

1. Agrupamos a volumetria do dia a partir de uma CTE sobre a tabela `Sales.SalesOrderHeader`.
2. Executamos o `MERGE` contra nossa tabela de controle `Log_Valida_Vendas`.
3. Se o registro para a data e território já existir, atualizamos a quantidade e o valor total (`UPDATE`).
4. Se não existir, inserimos o novo registro (`INSERT`).
5. Capturamos a ação executada (`INSERT` ou `UPDATE`) usando a cláusula `OUTPUT $action`.

---

Você pode encontrar esse banco de dados através do link:
https://learn.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver17&tabs=ssms

<br>

````sql

use adventureworks2025;
go


-- Cria tabela de log para auditoria do merge/upsert
if object_id('dbo.log_valida_vendas') is not null 
drop table dbo.log_valida_vendas; 


create table dbo.log_valida_vendas (
    datapedido date not null,
    territoryid int not null,
    qtdpedidos int not null,
    totalvendas money not null,
    dataatualizacao datetime2 not null,
    constraint pk_log_valida_vendas primary key clustered (datapedido, territoryid)
);

declare @datafiltro date;
set @datafiltro = (select max(cast(orderdate as date)) from sales.salesorderheader);

declare @hoje datetime2 = sysdatetime();

declare @logauditoria table (
    acaoexecutada varchar(10),
    datapedido date,
    territoryid int,
    qtdpedidos int,
    totalvendas money
);

with cte_totaisvendas as (
    select 
        cast(orderdate as date) as datapedido,
        territoryid,
        count(salesorderid) as qtdpedidos,
        sum(totaldue) as totalvendas
    from sales.salesorderheader
    where cast(orderdate as date) = @datafiltro
    group by cast(orderdate as date), territoryid
)
merge dbo.log_valida_vendas as a
using cte_totaisvendas as b
    on (a.datapedido = b.datapedido and a.territoryid = b.territoryid)
when matched then
    update set 
        a.qtdpedidos = b.qtdpedidos,
        a.totalvendas = b.totalvendas,
        a.dataatualizacao = @hoje
when not matched by target then
    insert (datapedido, territoryid, qtdpedidos, totalvendas, dataatualizacao)
    values (b.datapedido, b.territoryid, b.qtdpedidos, b.totalvendas, @hoje)

-- Captura de log informando se a linha foi insert ou update
output 
    $action as acaoexecutada,
    inserted.datapedido,
    inserted.territoryid,
    inserted.qtdpedidos,
    inserted.totalvendas
into @logauditoria (acaoexecutada, datapedido, territoryid, qtdpedidos, totalvendas);


select 
    acaoexecutada,
    datapedido,
    territoryid,
    qtdpedidos,
    totalvendas
from @logauditoria;

````
