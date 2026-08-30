# 🌳 CTEs Recursivas & Estruturas Hierárquicas

## 📌 Contexto e Aplicação Prática
Em bancos de dados relacionais, dados hierárquicos costumam ser armazenados através do padrão **Adjacency List** (onde uma linha possui uma chave estrangeira apontando para a chave primária do seu pai na mesma tabela, como `GerenteID` $\rightarrow$ `BusinessEntityID`).

Para consultar e navegar por todos os níveis dessa árvore (do topo da diretoria até a base operacional), a técnica ideal no SQL Server é a **CTE Recursiva**. Ela permite percorrer a hierarquia em profundidade ou largura dinamicamente em um único conjunto de dados (*Set-Based*).

---

## 🎯 Cenário de Negócio (AdventureWorks)
Neste estudo de caso, montamos o organograma completo da empresa utilizando a tabela `HumanResources.Employee`.

A CTE Recursiva é dividida em três partes principais:
1. **Anchor Member (Membro Ancoragem):** A consulta base que localiza o topo da hierarquia (ex.: o CEO, onde `OrganizationNode` ou a relação de gerência é nula/inicial).
2. **Recursive Member (Membro Recursivo):** A consulta que faz o `JOIN` da CTE com a própria tabela para encontrar os subordinados diretos do nível anterior.
3. **Condition / MAXRECURSION:** A condição de parada automática quando não houver mais filhos, além da trava de segurança `MAXRECURSION` para evitar *loops* infinitos.

---

## 📊 Benefícios do Uso de CTEs Recursivas

| Benefício | Descrição |
| :--- | :--- |
| **Simplicidade Declarativa** | Substitui algoritmos complexos de *loop* por uma única consulta legível. |
| **Cálculo de Níveis (Depth)** | Permite criar colunas dinâmicas de nível (`Nivel + 1`) e caminho completo (`CaminhoHierarquico`). |
| **Desempenho Otimizado** | Processado nativamente pelo motor do SQL Server como uma operação em conjunto. |

---

## ⚠️ Boas Práticas
* **Trava de Segurança (`OPTION (MAXRECURSION n)`):** Por padrão, o SQL Server limita a recursão a 100 níveis. Se a sua árvore for maior, ajuste o limite (ou use `0` para ilimitado com cautela).
* **Indescartável em Grafos/Árvores:** Use para construir caminhos (*breadcrumbs*) e calcular totais acumulados de cima para baixo na hierarquia.

````sql
use adventureworks2025;
go

set nocount on;

with cte_organograma as (
    select 
        e.businessentityid as funcionarioid,
        p.firstname + ' ' + p.lastname as nomefuncionario,
        e.jobtitle as cargo,
        e.organizationnode,
        cast(null as int) as gerenteid,
        cast(p.firstname + ' ' + p.lastname as varchar(max)) as caminhohierarquico,
        0 as nivel
    from humanresources.employee e
    inner join person.person p on e.businessentityid = p.businessentityid
    where e.organizationnode = '/' 
       or e.organizationnode.GetAncestor(1) is null

    union all

    select 
        sub.businessentityid as funcionarioid,
        psub.firstname + ' ' + psub.lastname as nomefuncionario,
        sub.jobtitle as cargo,
        sub.organizationnode,
        pai.funcionarioid as gerenteid,
        cast(pai.caminhohierarquico + ' > ' + psub.firstname + ' ' + psub.lastname as varchar(max)) as caminhohierarquico,
        pai.nivel + 1 as nivel
    from humanresources.employee sub
    inner join person.person psub on sub.businessentityid = psub.businessentityid
    inner join cte_organograma pai on sub.organizationnode.GetAncestor(1) = pai.organizationnode
)
select 
    funcionarioid,
    replicate('--- ', nivel) + nomefuncionario as organogramavisual,
    cargo,
    nivel,
    caminhohierarquico
from cte_organograma
order by organizationnode
option (maxrecursion 100); -- Proteção contra loop infinito
````
