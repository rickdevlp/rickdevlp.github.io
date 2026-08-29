# 🧠 Advanced T-SQL Handbook & Best Practices

![SQL Server](https://img.shields.io/badge/SQL%20Server-2025-CC292B?style=flat&logo=microsoftsqlserver&logoColor=white)
![T-SQL](https://img.shields.io/badge/Language-T--SQL-0078D4?style=flat)
![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-success)

Bem-vindo(a) ao **Advanced T-SQL Handbook**! Este repositório é um guia prático focado em padrões avançados de desenvolvimento, controle transacional, manipulação de dados em larga escala e otimização no **Microsoft SQL Server**.

---

## 🎯 Por que criei este repositório?

Este material foi idealizado a partir de um cenário real de uma conversa informal com amigos que desejam ingressar na área de dados e/ou já trabalham com isso.

O objetivo principal é ir além dos conceitos básicos de SQL, demonstrando:
* **Casos de uso reais do mercado:** Como resolver problemas do dia a dia de forma eficiente e escalável.
* **Arquitetura e Boas Práticas:** Padrões de escrita de código limpo, tratamento de erros e integridade transacional.
* **Performance & Set-Based Logic:** Comparativos práticos sobre quando usar cada técnica priorizando o desempenho do banco de dados.

---

## 📚 Tópicos & Conteúdo do Guia

O repositório é organizado em diretórios práticos contendo explicações teóricas e scripts `.sql` executáveis baseados no banco de dados de exemplo **AdventureWorks**:

| Módulo | Tópico Técnico | Descrição & Conceitos Chave |
| :---: | :--- | :--- |
| **01** | **[UPSERT com MERGE & Log de Auditoria](https://github.com/rickdevlp/rickdevlp.github.io/blob/f2eeea947596fb18a96bce4fdea6295e88ba516e/Advanced%20T-SQL%20Handbook/01%20-%20Upsert%20com%20Merge%20na%20pratica.md)** | Sincronização atômica de dados utilizando a cláusula `OUTPUT $action` para auditoria de `INSERT` e `UPDATE`. |
| **02** | **[SQL Dinâmico & Segurança](https://github.com/rickdevlp/rickdevlp.github.io/blob/956aa6ed05ccbcd9556b12e822cde4a489551150/Advanced%20T-SQL%20Handbook/02%20-%20SQL%20Dinamico.md)** | Execução de consultas dinâmicas com `sp_executesql`, prevenção de **SQL Injection** e parametrização. |
| **03** | **[Tratamento de Erros (Try Catch) & Transações](https://github.com/rickdevlp/rickdevlp.github.io/blob/e4c8f5b62e200d14074e711db44e4067da9244d4/Advanced%20T-SQL%20Handbook/03%20-%20Try%20Catch%20e%20Transac.md)** | Uso robusto de `TRY...CATCH`, controle de `TRANSACTION` e verificação de `XACT_STATE()` para rollbacks seguros. |
| **04** | **Cursores vs. Set-Based Operations** | Análise de performance comparando processamento linha a linha (*Cursors*) contra operações em conjunto e *Window Functions*. |
| **05** | **CTEs Recursivas & Hierarquias** | Consultas em estruturas hierárquicas e árvores de decisão utilizando Common Table Expressions. |

---

## 🛠️ Tecnologias e Ferramentas

* **SGBD:** Microsoft SQL Server (compatível com 2019/2022/2025)
* **Banco de Dados de Teste:** `AdventureWorks2025`
* **IDE Recomendada:** SQL Server Management Studio (SSMS) ou Azure Data Studio

---

## 👤 Autor

Desenvolvido por **Ricardo** 👋  
* 💼 **Especialidade:** Engenharia de Dados / SQL Server / Databricks / Teradata / SAS
* 🌐 **Portfólio Principal:** [rickdevlp.github.io](https://rickdevlp.github.io)
