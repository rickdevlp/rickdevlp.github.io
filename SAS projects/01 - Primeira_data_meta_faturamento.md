
# 💳 Identificação da primeira data de meta de faturamento via SAS

O objetivo deste estudo é demonstrar de forma prática como apurar o momento exato em que os clientes atingem a meta de faturamento. 

No contexto de negócio, a data dessa transação define em qual **safra** será contabilizada a pontuação do gerente de contas (por exemplo: se o cliente atingiu o acúmulo de R$ 500,00 em janeiro, a pontuação é atribuída à safra de janeiro).  

<br>

> [!NOTE]
> **Governança de Dados:** Por questões de segurança da informação, todos os dados utilizados são sintéticos e os critérios reais de apuração da campanha não serão descritos neste artigo.  

---
<br>

Para reproduzir este código gratuitamente em um ambiente de estudos, acesse o **[SAS OnDemand for Academics (ODA)](https://welcome.oda.sas.com/)**.

<br>

```sas
/* Criação da tabela com dados randômicos gerados via IA */
data work.transacoes_cartao;
    length cliente_id $8 cartao_id $12 data_transacao 8 valor_transacao 8;
    format data_transacao ddmmyy10. valor_transacao commax10.2;

    /* --- CLIENTE 1 --- */
    cliente_id = "CLI_0001"; cartao_id = "CARD_AAAA_11";
    data_transacao = '02JAN2026'd; valor_transacao =  80.00; output;
    data_transacao = '05JAN2026'd; valor_transacao = 120.00; output;
    data_transacao = '10JAN2026'd; valor_transacao = 150.00; output;
    data_transacao = '15JAN2026'd; valor_transacao = 180.00; output;
    data_transacao = '20JAN2026'd; valor_transacao =  90.00; output;
    data_transacao = '25JAN2026'd; valor_transacao = 210.00; output;
    data_transacao = '01FEB2026'd; valor_transacao =  50.00; output;
    data_transacao = '05FEB2026'd; valor_transacao = 300.00; output;
    data_transacao = '12FEB2026'd; valor_transacao = 110.00; output;
    data_transacao = '20FEB2026'd; valor_transacao =  75.00; output;

    /* --- CLIENTE 2 --- */
    cliente_id = "CLI_0002"; cartao_id = "CARD_BBBB_22";
    data_transacao = '01JAN2026'd; valor_transacao =  30.00; output;
    data_transacao = '08JAN2026'd; valor_transacao =  45.00; output;
    data_transacao = '14JAN2026'd; valor_transacao =  60.00; output;
    data_transacao = '22JAN2026'd; valor_transacao =  50.00; output;
    data_transacao = '30JAN2026'd; valor_transacao =  85.00; output;
    data_transacao = '05FEB2026'd; valor_transacao =  40.00; output;
    data_transacao = '12FEB2026'd; valor_transacao = 110.00; output; 
    data_transacao = '18FEB2026'd; valor_transacao = 150.00; output; 
    data_transacao = '25FEB2026'd; valor_transacao =  95.00; output;
    data_transacao = '28FEB2026'd; valor_transacao = 200.00; output;

    /* --- CLIENTE 3 --- */
    cliente_id = "CLI_0003"; cartao_id = "CARD_CCCC_33";
    data_transacao = '03JAN2026'd; valor_transacao = 750.00; output;
    data_transacao = '09JAN2026'd; valor_transacao = 120.00; output;
    data_transacao = '15JAN2026'd; valor_transacao =  80.00; output;
    data_transacao = '21JAN2026'd; valor_transacao = 300.00; output;
    data_transacao = '28JAN2026'd; valor_transacao = 150.00; output;
    data_transacao = '02FEB2026'd; valor_transacao =  90.00; output;
    data_transacao = '10FEB2026'd; valor_transacao = 400.00; output;
    data_transacao = '16FEB2026'd; valor_transacao =  65.00; output;
    data_transacao = '22FEB2026'd; valor_transacao = 180.00; output;
    data_transacao = '27FEB2026'd; valor_transacao = 210.00; output;
run;

/* A utilização do proc sort garante que os dados estejam ordenados por cartão e data de transação */
proc sort data=work.transacoes_cartao out=work.transacoes_ordenadas;
    by cartao_id data_transacao;
run;

data work.primeira_data_meta (keep=cliente_id cartao_id data_meta_atingida valor_acumulado_meta num_transacao_atingida);
    set work.transacoes_ordenadas;
    by cartao_id;

/* Garante que as variáveis acumulado_valor, ja_atingiu_meta e qtd_transacoes não sejam zeradas ao passar para a próxima transação iniciando com 0. */
    retain acumulado_valor ja_atingiu_meta qtd_transacoes 0;
    retain data_meta_atingida .;

    format data_meta_atingida ddmmyy10. valor_acumulado_meta commax10.2;

/* Declaração de vetor com 3 variáveis */
    array controle_meta[3] acumulado_valor ja_atingiu_meta qtd_transacoes;

/* Reset a cada novo cartao */
    if first.cartao_id then do;
        controle_meta[1] = 0; /* acumulado_valor */
        controle_meta[2] = 0; /* ja_atingiu_meta */
        controle_meta[3] = 0; /* qtd_transacoes */
        data_meta_atingida = .;
    end;

/* Incrementar transações e somatório */
    controle_meta[3] = controle_meta[3] + 1;
    controle_meta[1] = controle_meta[1] + valor_transacao;

/* Validação da regra de negocio de acionamento unico */
    if controle_meta[1] >= 500 and controle_meta[2] = 0 then do;
        controle_meta[2] = 1; 
        data_meta_atingida = data_transacao;
        valor_acumulado_meta = controle_meta[1]; 
        num_transacao_atingida = controle_meta[3];
        output;
    end;
run;

proc print data=work.primeira_data_meta noobs;
run;
```

Resultado:
<img width="872" height="163" alt="image" src="https://github.com/user-attachments/assets/409d5c49-7975-45ef-a1d7-061a261c0877" />
