

# ✏️ Reestruturação de tabelas - Estudo de caso (Em construção)

Este documento tem por objetivo demonstrar uma situação real de transformação de uma tabela em um Data Warehouse qualificado, com a criação de tabelas fato e dimensão.


````sql
drop table if exists #TransacoesBancarias_Bruto;
drop table if exists Fato_Transacoes; 
drop table if exists Dim_Funcionario;
drop table if exists Dim_Empresa;
drop table if exists Dim_Localidade;
go

create table #TransacoesBancarias_Bruto (
    ID int identity(1,1),
    CPF varchar(50),
    CNPJ varchar(50),
    Nome_Funcionario varchar(150),
    Empresa varchar(150),
    Cidade varchar(100),
    Estado varchar(50),
    Pais varchar(50),
    CEP varchar(50),
    Telefone varchar(50),
    Cartao_Credito varchar(50),
    Cartao_Debito varchar(50),
    Valor_Transacao varchar(50), 
    Data_Transacao varchar(50),  
    Moeda varchar(10),
    Bandeira_Cartao varchar(50),
    Produto_Cartao varchar(50)
);

insert into #TransacoesBancarias_Bruto (CPF, CNPJ, Nome_Funcionario, Empresa, Cidade, Estado, Pais, CEP, Telefone, Cartao_Credito, Cartao_Debito, Valor_Transacao, Data_Transacao, Moeda, Bandeira_Cartao, Produto_Cartao)
values
('123.456.789-00', '12.345.678/0001-99', 'Ricardo Silva', 'Tech Solutions LTDA', 'São Paulo', 'SP', 'Brasil', '01310-100', '(11) 98888-7777', '4111222233334444', null, '1500.50', '2026-05-15 14:30:00', 'BRL', 'Visa', 'Platinum'),
('98765432100', '98765432000188', 'ANTONIO CARLOS DE SOUZA', 'Banco Alfa S/A', 'Rio de Janeiro', 'RJ', 'Brazil', '20040000', '21977776666', null, '5000111122223333', '2.450,75', '16/05/2026', 'BRL', 'MASTERCARD', 'Black'),
('11122233344', null, '  Maria   Eduarda Lima  ', 'Global Corp', 'Miami', null, 'USA', '33101', '+1 305-555-0199', '378282246310005', null, '120.00', '2026/05/14', 'USD', 'Amex', 'Corporate'),
('123.456.789-00', '12.345.678/0001-99', 'Ricardo Silva', 'Tech Solutions LTDA', 'Sao Paulo', 'S.P.', 'BR', '01310100', '11988887777', '4111222233334444', null, '1500.50', '2026-05-15 14:31:00', 'R$', 'visa', 'platinum'),
(null, '00.111.222/0001-33', 'João nulo', 'Empresa Fantasia', 'SãO PaUlO', 'SP', 'Brasil', '00000-000', 'N/A', '4111222233334444', '5000111122223333', '-50.00', '2026-05-01', 'BRL', 'Elo', 'Grafite'),
('444.555.666-77', null, 'Andr Alcantara', 'Comércio de Doces', 'Curitiba', 'PR', 'Brasil', '80010-000', '41 999998888', null, null, '0.00', '2026-05-10', 'BRL', 'Master', 'Standard');

create table Dim_Funcionario (
    Funcionario_ID int identity(1,1) primary key,
    CPF char(11) not null unique, 
    Nome_Funcionario varchar(150) not null
);

create table Dim_Empresa (
    Empresa_ID int identity(1,1) primary key,
    CNPJ char(14) null, 
    Nome_Empresa varchar(150) not null
);

create table Dim_Localidade (
    Localidade_ID int identity(1,1) primary key,
    Cidade varchar(100) not null,
    Estado char(2) not null,
    Pais varchar(50) not null,
    CEP char(8) not null
);

create table Fato_Transacoes (
    Transacao_ID int identity(1,1) primary key,
    Funcionario_ID int references Dim_Funcionario(Funcionario_ID),
    Empresa_ID int references Dim_Empresa(Empresa_ID),
    Localidade_ID int references Dim_Localidade(Localidade_ID),
    Telefone_Contato varchar(20), 
    Numero_Cartao varchar(16) not null, 
    Tipo_Cartao varchar(10) not null,   
    Bandeira_Cartao varchar(20) not null,
    Produto_Cartao varchar(30) not null,
    Valor_Transacao decimal(18,2) not null, 
    Data_Transacao datetime not null,       
    Moeda char(3) not null                  
);

create nonclustered index IX_Fato_Transacoes_Data on Fato_Transacoes(Data_Transacao);
create nonclustered index IX_Fato_Transacoes_Funcionario on Fato_Transacoes(Funcionario_ID);
go

drop table if exists #Dados_Limpos;

with CTE_Pre_Limpeza as (
    select 
        nullif(replace(translate(CPF, '.-/', '   '), ' ', ''), '') as CPF_Limpo,
        nullif(replace(translate(CNPJ, '.-/', '   '), ' ', ''), '') as CNPJ_Limpo,
        replace(replace(trim(Nome_Funcionario), '  ', ' '), '  ', ' ') as Nome_Limpo,
        trim(Empresa) as Empresa_Limpa,
        case 
            when Cidade like 'S%o P%ulo' then 'São Paulo'
            else trim(Cidade)
        end as Cidade_Limpa,
        case 
            when Estado in ('SP', 'S.P.', 'SãO PaUlO') then 'SP'
            when Estado is null then 'NE' 
            else upper(trim(Estado))
        end as Estado_Limpo,
        case 
            when Pais in ('Brasil', 'Brazil', 'BR') then 'Brasil'
            else trim(Pais)
        end as Pais_Limpo,
        replace(trim(CEP), '-', '') as CEP_Limpo, 
        coalesce(Cartao_Credito, Cartao_Debito) as Numero_Cartao,
        case when Cartao_Credito is not null then 'Crédito' else 'Débito' end as Tipo_Cartao,
        upper(left(Bandeira_Cartao, 1)) + lower(substring(Bandeira_Cartao, 2, len(Bandeira_Cartao))) as Bandeira_Limpa, 
        try_cast(
            case 
                when Valor_Transacao like '%,%' then replace(replace(Valor_Transacao, '.', ''), ',', '.')
                else Valor_Transacao
            end as decimal(18,2)
        ) as Valor_Limpo,
        coalesce(
            try_convert(datetime, Data_Transacao, 120), 
            try_convert(datetime, Data_Transacao, 103), 
            try_convert(datetime, replace(Data_Transacao, '/', '-'), 120) 
        ) as Data_Limpa,
        case when Moeda = 'R$' then 'BRL' else upper(trim(Moeda)) end as Moeda_Limpa,
        Telefone,
        Produto_Cartao
    from #TransacoesBancarias_Bruto
    where CPF is not null 
      and (Cartao_Credito is not null or Cartao_Debito is not null) 
)
select * into #Dados_Limpos from CTE_Pre_Limpeza;
go


insert into Dim_Funcionario (CPF, Nome_Funcionario)
select distinct CPF_Limpo, Nome_Limpo 
from #Dados_Limpos;

insert into Dim_Empresa (CNPJ, Nome_Empresa)
select distinct CNPJ_Limpo, Empresa_Limpa 
from #Dados_Limpos 
where Empresa_Limpa is not null;

insert into Dim_Localidade (Cidade, Estado, Pais, CEP)
select distinct Cidade_Limpa, Estado_Limpo, Pais_Limpo, CEP_Limpo 
from #Dados_Limpos;
go


insert into Fato_Transacoes (Funcionario_ID, Empresa_ID, Localidade_ID, Telefone_Contato, Numero_Cartao, Tipo_Cartao, Bandeira_Cartao, Produto_Cartao, Valor_Transacao, Data_Transacao, Moeda)
select 
    f.Funcionario_ID,
    e.Empresa_ID,
    l.Localidade_ID,
    d.Telefone,
    d.Numero_Cartao,
    d.Tipo_Cartao,
    d.Bandeira_Limpa,
    d.Produto_Cartao,
    d.Valor_Limpo,
    d.Data_Limpa,
    d.Moeda_Limpa
from #Dados_Limpos d
inner join Dim_Funcionario f 
    on d.CPF_Limpo collate database_default = f.CPF collate database_default
left join Dim_Empresa e 
    on d.CNPJ_Limpo collate database_default = e.CNPJ collate database_default 
    or (d.CNPJ_Limpo is null and e.CNPJ is null)
inner join Dim_Localidade l 
    on d.Cidade_Limpa collate database_default = l.Cidade collate database_default
    and d.Estado_Limpo collate database_default = l.Estado collate database_default
    and d.Pais_Limpo collate database_default = l.Pais collate database_default
    and d.CEP_Limpo collate database_default = l.CEP collate database_default;
go
````
