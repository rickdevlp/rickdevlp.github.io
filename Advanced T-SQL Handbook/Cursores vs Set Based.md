# ⚡ Cursores vs. Operações Baseadas em Conjuntos (Set-Based)

## 📌 Contexto e Aplicação Prática
O relacional do SQL Server foi projetado para operar sobre **conjuntos de dados** (vetoriais). O uso de **Cursores** força o motor do banco de dados a iterar linha a linha, o que gera um overhead massivo de memória, uso excessivo de I/O e *locks* de recursos.

Apesar disso, existem casos legítimos para o uso de cursores (ex.: scripts de manutenção, execução de rotinas administrativas linha a linha, ou chamadas de Stored Procedures externas por registro). Para a manipulação de dados de negócio, no entanto, abordagens *Set-Based* usando **Window Functions** ou **CTEs** devem sempre ser a primeira opção.

---

## 🎯 Cenário de Negócio (AdventureWorks)
Neste estudo de caso, calculamos o **Total Acumulado de Vendas** (*Running Total*) por cliente ao longo do tempo na tabela `Sales.SalesOrderHeader`.

Comparamos duas abordagens para resolver o mesmo problema:
1. **Abordagem com Cursor:** Lê cliente por cliente, acumulando o valor em uma variável via *loop*.
2. **Abordagem Set-Based:** Utiliza a *Window Function* `SUM() OVER(PARTITION BY ... ORDER BY ...)` em uma única instrução SQL.

---

## 📊 Comparativo de Abordagens

| Métrica / Critério | Abordagem com Cursor | Abordagem Set-Based (Window Function) |
| :--- | :--- | :--- |
| **Paradigmas** | Imperativo (linha a linha / RBAR) | Declarativo (conjunto de dados) |
| **Desempenho** | Lento em grande volumetria (alto custo de CPU) | Extremamente rápido e otimizado pelo Engine |
| **Uso de Recursos** | Abre alocações na TempDB e ponteiros de memória | Processado diretamente na memória em batch |
| **Complexidade** | Exige `DECLARE`, `OPEN`, `FETCH`, `CLOSE` e `DEALLOCATE` | Resolvido em uma única query `SELECT` limpa |

---

## ⚠️ Boas Práticas
* **Regra de Ouro:** Só utilize cursores se for estritamente necessário (ex.: rotinas DDL administrativas em loop).
* **Propriedades do Cursor:** Se precisar usar um cursor, declare-o como `LOCAL FAST_FORWARD` ou `LOCAL READ_ONLY FORWARD_ONLY` para minimizar a alocação de recursos.
* **Substitutos Modernos:** Substitua cursores por *Window Functions* (`SUM() OVER`, `ROW_NUMBER()`, `LEAD()`, `LAG()`) ou tabelas temporárias parametrizadas.

# ⏰ Avaliação de tempo entre cursor e Set-Based

````sql
use adventureworks2025;
go

-- Teste de tempo avaliando a utilização do cursor
set nocount on;
declare @startcursor datetime = getdate();

-- Tabela temporária para receber o resultado do cursor
if object_id('tempdb..#resultadocursor') is not null drop table #resultadocursor;
create table #resultadocursor (
    customerid int,
    salesorderid int,
    orderdate datetime,
    totaldue money,
    runningtotal money
);

declare @customerid int, @salesorderid int, @orderdate datetime, @totaldue money;
declare @clienteatual int = -1;
declare @acumulado money = 0;

-- Declaração do cursor com opções otimizadas de leitura rápida
declare curvendas cursor local fast_forward for
    select customerid, salesorderid, orderdate, totaldue
    from sales.salesorderheader
    order by customerid, orderdate, salesorderid;

open curvendas;

fetch next from curvendas into @customerid, @salesorderid, @orderdate, @totaldue;

while @@fetch_status = 0
begin
    if @customerid <> @clienteatual
    begin
        set @clienteatual = @customerid;
        set @acumulado = 0;
    end

    set @acumulado = @acumulado + @totaldue;

    insert into #resultadocursor (customerid, salesorderid, orderdate, totaldue, runningtotal)
    values (@customerid, @salesorderid, @orderdate, @totaldue, @acumulado);

    fetch next from curvendas into @customerid, @salesorderid, @orderdate, @totaldue;
end;

close curvendas;
deallocate curvendas;

print 'Tempo de execução (cursor): ' + cast(datediff(ms, @startcursor, getdate()) as varchar(10)) + ' ms';

-- Teste de tempo avaliando a utilização por set-based
set nocount on;
declare @startsetbased datetime = getdate();

if object_id('tempdb..#resultadosetbased') is not null drop table #resultadosetbased;

select 
    customerid,
    salesorderid,
    orderdate,
    totaldue,
    sum(totaldue) over(
        partition by customerid 
        order by orderdate, salesorderid
        rows between unbounded preceding and current row
    ) as runningtotal
into #resultadosetbased
from sales.salesorderheader;

print 'Tempo de execução (set-based): ' + cast(datediff(ms, @startsetbased, getdate()) as varchar(10)) + ' ms';
````

Resultado:

Tempo de execução (cursor): 870 ms

Tempo de execução (set-based): 73 ms


Horário de conclusão: 2026-08-29T23:39:57.5598367-03:00

# 🎯 Quando utilizar cursores?

A utilização de cursores passa a ser completamente necessária quando existe a necessidade do código ler linha a linha. 
Rotinas de manutenção administrativa em nível de banco de dados (operações DDL) são o exemplo perfeito em que o uso de um cursor é totalmente justificado. 
Como o comando ALTER INDEX não aceita execução baseada em conjuntos (Set-Based), é necessário iterar sobre a lista de tabelas/índices e disparar o comando dinâmico tabela por tabela.

No código abaixo, represento de forma prática a utilização eficiente de cursores para rebuildar índices e torná-los performáticos novamente.

````sql
use adventureworks2025;
go

set nocount on;

if object_id('tempdb..#indicesparamanutencao') is not null 
    drop table #indicesparamanutencao;

create table #indicesparamanutencao (
    esquema nvarchar(128),
    tabela nvarchar(128),
    indice nvarchar(128),
    percentualfragmentacao float
);

insert into #indicesparamanutencao (esquema, tabela, indice, percentualfragmentacao)
select 
    s.name as esquema,
    t.name as tabela,
    i.name as indice,
    ps.avg_fragmentation_in_percent as percentualfragmentacao
from sys.dm_db_index_physical_stats(db_id(), null, null, null, 'limited') ps
inner join sys.tables t on ps.object_id = t.object_id
inner join sys.schemas s on t.schema_id = s.schema_id
inner join sys.indexes i on ps.object_id = i.object_id and ps.index_id = i.index_id
where ps.avg_fragmentation_in_percent > 10.0 -- Apenas índices com fragmentação > 10%
  and i.name is not null;

declare @esquema nvarchar(128);
declare @tabela nvarchar(128);
declare @indice nvarchar(128);
declare @fragmentacao float;
declare @sqlinstruction nvarchar(max);

declare curmanutencao cursor local fast_forward for
    select esquema, tabela, indice, percentualfragmentacao
    from #indicesparamanutencao;

open curmanutencao;

fetch next from curmanutencao into @esquema, @tabela, @indice, @fragmentacao;

while @@fetch_status = 0
begin
    -- Se a fragmentação for <= 30%, faz reorganize. se for maior, faz rebuild.
    if @fragmentacao <= 30.0
    begin
        set @sqlinstruction = N'alter index ' + quotename(@indice) + N' on ' 
                            + quotename(@esquema) + N'.' + quotename(@tabela) 
                            + N' reorganize;';
    end
    else
    begin
        set @sqlinstruction = N'alter index ' + quotename(@indice) + N' on ' 
                            + quotename(@esquema) + N'.' + quotename(@tabela) 
                            + N' rebuild;';
    end

    print 'executando: ' + @sqlinstruction;
    exec sp_executesql @sqlinstruction;

    fetch next from curmanutencao into @esquema, @tabela, @indice, @fragmentacao;
end;

close curmanutencao;
deallocate curmanutencao;
drop table #indicesparamanutencao;
````
