



Você pode encontrar esse banco de dados através do link:
https://learn.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver17&tabs=ssms

<br>

````sql

use adventureworks2025;
go

declare @shortdate int = year(getdate()) * 100 + month(getdate());
declare @nometabela_setup nvarchar(128) = 'fat_' + cast(@shortdate as varchar(6));
declare @sql_setup nvarchar(max);

-- 1.1 drop table dinâmico
if object_id('dbo.' + quotename(@nometabela_setup)) is not null
begin
    set @sql_setup = N'drop table dbo.' + quotename(@nometabela_setup) + N';';
    exec sp_executesql @sql_setup;
end;

-- 1.2 create table dinâmico
set @sql_setup = N'
create table dbo.' + quotename(@nometabela_setup) + N' (
    id int identity(1,1) primary key,
    territoryid int,
    totalvendas money,
    datacarga datetime default getdate()
);';
exec sp_executesql @sql_setup;

-- 1.3 insert dinâmico
set @sql_setup = N'
insert into dbo.' + quotename(@nometabela_setup) + N' (territoryid, totalvendas)
values (1, 15000.50), (2, 32000.00), (3, 8500.75);';
exec sp_executesql @sql_setup;


declare @nometabela nvarchar(128);
declare @sql nvarchar(max);

-- 1. monta o nome da tabela com o sufixo da variável
set @nometabela = 'fat_' + cast(@shortdate as varchar(6));

-- 2. monta a instrução sql utilizando quotename para segurança
set @sql = N'select id, territoryid, totalvendas, datacarga from dbo.' 
          + quotename(@nometabela) 
          + N' where totalvendas > @valorminimo;';

-- 3. executa a consulta passando parâmetros de filtro com segurança
declare @valorfiltro money = 10000.00;

exec sp_executesql 
    @stmt = @sql,
    @params = N'@valorminimo money',
    @valorminimo = @valorfiltro;

print 'Consulta executada com sucesso na tabela: ' + @nometabela;

''''
