
# 🛡️ Tratamento de Erros e Controle Transacional em T-SQL

## 📌 Contexto e Aplicação Prática
Em sistemas críticos de banco de dados, garantir a **atomicidade** das operações (propriedade ACID) é indispensável: ou todas as etapas de um processo são salvas com sucesso, ou nenhuma alteração deve persistir.

O uso do bloco `TRY...CATCH` integrado com transações explícitas garante que qualquer falha durante a execução de um script acione o cancelamento das alterações (*Rollback*) e grave os detalhes do erro para auditoria.

---

## 🎯 Cenário de Negócio (AdventureWorks)
Simulamos uma operação bancária/comercial de transferência de créditos entre dois clientes.

1. Inicia-se uma transação explícita com `BEGIN TRANSACTION`.
2. O sistema tenta debitar o saldo de uma conta e creditar em outra.
3. Se ocorrer qualquer erro (como erro de divisão por zero, violação de *constraint* ou chave primária), o controle é desviado imediatamente para o bloco `CATCH`.
4. Utiliza-se a função `XACT_STATE()` para verificar o estado da transação antes de executar o `ROLLBACK`, evitando erros adicionais ao tentar cancelar transações não ativas.
5. Os detalhes da falha (`ERROR_NUMBER`, `ERROR_MESSAGE`, etc.) são capturados e retornados.

---

## ⚠️ Boas Práticas
* **Uso do `XACT_STATE()`:** Retorna `1` se a transação é válida e pode sofrer *Commit*, `-1` se a transação está condenada e **deve** sofrer *Rollback*, e `0` se não há transação ativa. É mais seguro do que checar apenas `@@TRANCOUNT`.
* **Captação Detalhada:** Grave sempre `ERROR_PROCEDURE()`, `ERROR_LINE()` e `ERROR_MESSAGE()` em uma tabela de log de erros do sistema.
* **`SET XACT_ABORT ON`:** Recomendado no início das rotinas para garantir que erros graves façam o cancelamento automático da transação.


````sql
use adventureworks2025;
go


if object_id('dbo.contacorrente') is not null
    drop table dbo.contacorrente;

create table dbo.contacorrente (
    contaid int primary key,
    cliente varchar(50) not null,
    saldo money not null,
    constraint ck_saldo_positivo check (saldo >= 0) -- Força erro se o saldo ficar negativo
);

insert into dbo.contacorrente (contaid, cliente, saldo)
values (1001, 'ricardo', 500.00),
       (1002, 'ana', 200.00);

set xact_abort on;

declare @contaorigem int = 1001;
declare @contadestino int = 1002;
declare @valortransferencia money = 600.00; -- Valor propositalmente maior que o saldo para forçar o erro

begin try

    begin transaction;

    -- Debita da conta de origem
    update dbo.contacorrente
    set saldo = saldo - @valortransferencia
    where contaid = @contaorigem;

    -- Credita na conta de destino
    update dbo.contacorrente
    set saldo = saldo + @valortransferencia
    where contaid = @contadestino;

    -- Confirma as alterações se não houver erros
    commit transaction;
    print 'Transferência realizada com sucesso!';

end try
begin catch
    
    print 'Valor do xact_state: ' + cast(xact_state() as varchar(2)); 
    -- Como o valor do xact_state() é -1, o commit não é realizado, e a transação é cancelada (rollback) pois o saldo ficou negativo.

    if (xact_state()) <> 0
    begin
        rollback transaction;
        print 'Transação cancelada (rollback) devido a um erro durante a execução.';
    end

    -- Retorna detalhes do erro capturado
    select 
        error_number() as numerodoerro,
        error_severity() as gravidade,
        error_state() as estadodoerro,
        error_procedure() as storedprocedure,
        error_line() as linhadoerro,
        error_message() as mensagemdeerro;
end catch;
````
