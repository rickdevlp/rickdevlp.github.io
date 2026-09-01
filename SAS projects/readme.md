# ⚡ Advanced SAS Programming & Data Engineering

![SAS](https://img.shields.io/badge/SAS-ODA-blue?style=flat-square&logo=sas)
![Language](https://img.shields.io/badge/Language-SAS_DATA_Step-00569B?style=flat-square)
![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-brightgreen?style=flat-square)

Bem-vindo(a) ao **Advanced SAS Programming & Data Engineering**! Este repositório é um guia prático focado em técnicas avançadas de processamento de dados, otimização de memória procedural, manipulação de estados e automação utilizando a linguagem **SAS**.

---

## 🎯 Por que criei este repositório?

Este material foi idealizado a partir de desafios reais vivenciados no dia a dia corporativo em engenharia e análise de dados, onde a eficiência no processamento de grandes volumes transacionais é fundamental.

O objetivo principal é ir além dos scripts básicos de extração, demonstrando:

* **Casos de uso reais do mercado:** Como resolver regras de negócio complexas de forma escalável e performática.
* **Manipulação Avançada de Memória:** Uso de vetores (`ARRAY`), controle de persistência inter-linhas (`RETAIN`) e flags de grupo (`FIRST.` / `LAST.`).
* **Performance & Single-Pass Logic:** Comparativos e padrões de projeto para processar milhões de registros em uma única passagem pelos dados.

---

## 📚 Tópicos & Conteúdo do Guia

O repositório é organizado em módulos práticos contendo explicações teóricas, scripts `.sas` executáveis e logs de execução limpos:

| Módulo | Tópico Técnico | Descrição & Conceitos Chave |
| :---: | :--- | :--- |
| **01** | [**Threshold de Faturamento com ARRAY & RETAIN**](https://github.com/rickdevlp/rickdevlp.github.io/blob/86f3038ca54074aa20ad2ec7323a9f2316201c8c/SAS%20projects/01%20-%20Primeira_data_meta_faturamento.md) | Identificação da 1ª data exata em que um cartão atinge o limite de R$ 500,00 utilizando `ARRAY`, `RETAIN`, controle por `FIRST.cartao_id` e `OUTPUT` condicional. |
| **02** | **Macros & Automação de Processos** *(Em breve)* | Criação de rotinas dinâmicas e reutilizáveis utilizando a linguagem SAS Macro (`%MACRO`, `%DO`, `%SYSFUNC`). |
| **03** | **Otimização de Merges & Hash Tables** *(Em breve)* | Comparativo de performance entre `PROC SORT + MERGE` clássico contra carregamento em memória via `DECLARE HASH`. |

---

## 🛠️ Ambiente de Execução

Todos os códigos deste repositório são compatíveis e foram testados na nuvem através do **[SAS OnDemand for Academics (ODA)](https://welcome.oda.sas.com/)**.
