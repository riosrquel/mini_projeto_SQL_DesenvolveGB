# T1EngenhariaDados - Mini Projeto de Pipeline de Dados e Análise com SQL

## Contexto do Projeto

A **Livraria DevSaber** é uma loja online que iniciou suas primeiras vendas e registrou os dados em uma planilha. Nosso desafio foi transformar essa planilha em um **mini data warehouse** no **Google BigQuery** para permitir análises de negócio mais eficientes e escaláveis.

Este projeto foi desenvolvido como parte do exercício **T1EngenhariaDados**, utilizando o dataset `treinamento_Raquel`.

---

## Objetivo

- Criar o schema no BigQuery com as tabelas **Clientes, Produtos e Vendas**.
- Inserir os dados fornecidos nas tabelas correspondentes.
- Realizar consultas SQL para extrair insights de negócio.
- Criar uma **VIEW** para simplificar consultas futuras.

---

## Estrutura do Pipeline de Dados

### 1. Criação do Dataset
- Dataset criado: `treinamento_Raquel`.
- Local de armazenamento: conforme configuração do projeto no BigQuery.

### 2. Criação das Tabelas
- **Clientes**: armazenar informações únicas dos clientes.
- **Produtos**: armazenar informações únicas dos produtos.
- **Vendas**: tabela fato que conecta clientes e produtos.

```sql
-- Exemplo de criação de tabela Clientes
CREATE OR REPLACE TABLE `seu_projeto.treinamento_Raquel.Clientes` (
    ID_Cliente INT64,
    Nome_Cliente STRING,
    Email_Cliente STRING,
    Estado_Cliente STRING
);
```

3. Inserção de Dados

```sql
 -- Inserção de clientes
INSERT INTO `seu_projeto.treinamento_Raquel.Clientes` (ID_Cliente, Nome_Cliente, Email_Cliente, Estado_Cliente)
VALUES
    (1, 'Ana Silva', 'ana.s@email.com', 'SP'),
    (2, 'Bruno Costa', 'b.costa@email.com', 'RJ'),
    (3, 'Carla Dias', 'carla.d@email.com', 'SP'),
    (4, 'Daniel Souza', 'daniel.s@email.com', 'MG');

-- Inserção de produtos
INSERT INTO `seu_projeto.treinamento_Raquel.Produtos` (ID_Produto, Nome_Produto, Categoria_Produto, Preco_Produto)
VALUES
    (101, 'Fundamentos de SQL', 'Dados', 60.00),
    (102, 'Duna', 'Ficção Científica', 80.50),
    (103, 'Python para Dados', 'Programação', 75.00),
    (104, 'O Guia do Mochileiro', 'Ficção Científica', 42.00);

-- Inserção de vendas
INSERT INTO `seu_projeto.treinamento_Raquel.Vendas` (ID_Venda, ID_Cliente, ID_Produto, Data_Venda, Quantidade)
VALUES
    (1, 1, 101, '2024-01-15', 1),
    (2, 2, 102, '2024-01-18', 1),
    (3, 3, 103, '2024-02-02', 2),
    (4, 1, 102, '2024-02-10', 1),
    (5, 4, 101, '2024-02-20', 1),
    (6, 2, 104, '2024-03-05', 1);
```
4. Consultas de Análise

1️⃣ Clientes de SP
```sql
SELECT Nome_Cliente
FROM `seu_projeto.treinamento_Raquel.Clientes`
WHERE Estado_Cliente = 'SP';
```
2️⃣ Produtos da categoria "Ficção Científica"
```sql
SELECT Nome_Produto
FROM `seu_projeto.treinamento_Raquel.Produtos`
WHERE Categoria_Produto = 'Ficção Científica';
```
3️⃣ Todas as vendas detalhadas
```sql
SELECT
    C.Nome_Cliente,
    P.Nome_Produto,
    V.Data_Venda
FROM `seu_projeto.treinamento_Raquel.Vendas` AS V
JOIN `seu_projeto.treinamento_Raquel.Clientes` AS C ON V.ID_Cliente = C.ID_Cliente
JOIN `seu_projeto.treinamento_Raquel.Produtos` AS P ON V.ID_Produto = P.ID_Produto
ORDER BY V.Data_Venda;
```
4️⃣ Valor total de cada venda
```sql
SELECT
    V.ID_Venda,
    (V.Quantidade * P.Preco_Produto) AS Valor_Total
FROM `seu_projeto.treinamento_Raquel.Vendas` AS V
JOIN `seu_projeto.treinamento_Raquel.Produtos` AS P ON V.ID_Produto = P.ID_Produto;
```
5️⃣ Produto mais vendido
```sql
SELECT
    P.Nome_Produto,
    SUM(V.Quantidade) AS Total_Quantidade_Vendida
FROM `seu_projeto.treinamento_Raquel.Vendas` AS V
JOIN `seu_projeto.treinamento_Raquel.Produtos` AS P ON V.ID_Produto = P.ID_Produto
GROUP BY P.Nome_Produto
ORDER BY Total_Quantidade_Vendida DESC
LIMIT 1;
```

5. Criação da VIEW
```sql
   CREATE OR REPLACE VIEW `seu_projeto.treinamento_Raquel.v_relatorio_vendas_detalhado` AS
SELECT
    V.ID_Venda,
    V.Data_Venda,
    C.Nome_Cliente,
    C.Estado_Cliente,
    P.Nome_Produto,
    P.Categoria_Produto,
    V.Quantidade,
    P.Preco_Produto,
    (V.Quantidade * P.Preco_Produto) AS Valor_Total
FROM `seu_projeto.treinamento_Raquel.Vendas` AS V
JOIN `seu_projeto.treinamento_Raquel.Clientes` AS C ON V.ID_Cliente = C.ID_Cliente
JOIN `seu_projeto.treinamento_Raquel.Produtos` AS P ON V.ID_Produto = P.ID_Produto;
```
💡 Vantagens da VIEW:

- Automatiza consultas complexas com JOINs.

- Atualiza valores automaticamente se os dados das tabelas mudarem.

- Facilita o acesso e padroniza relatórios para toda a equipe.

## Perguntas do Mini Projeto

### 1️⃣ Sobre planilhas e análises
**Pergunta:** Por que uma planilha não é ideal para uma empresa que quer analisar suas vendas a fundo?  
**Resposta:**  
- Planilhas têm limitações de escala e desempenho para grandes volumes de dados.  
- Não garantem integridade referencial entre clientes, produtos e vendas.  
- Análises complexas, como agregações, filtros e cruzamentos (JOINs), precisam ser feitas manualmente, aumentando a chance de erro.

**Pergunta:** Que tipo de perguntas vocês acham que o dono da livraria gostaria de responder com esses dados?  
**Resposta:**  
- Quais produtos e categorias vendem mais?  
- Quem são os clientes mais frequentes ou que geram maior receita?  
- Qual o total de vendas por período ou por região?  
- Existe sazonalidade nas compras?  
- Quais clientes ou produtos são potenciais para ações de marketing ou promoções?

---

### 2️⃣ Sobre a modelagem das tabelas
**Pergunta:** Com base nos dados brutos, quais outras duas tabelas precisamos criar? Que colunas e tipos de dados elas teriam?  
**Resposta:**  
- **Clientes**: ID_Cliente (INT64), Nome_Cliente (STRING), Email_Cliente (STRING), Estado_Cliente (STRING)  
- **Produtos**: ID_Produto (INT64), Nome_Produto (STRING), Categoria_Produto (STRING), Preco_Produto (NUMERIC)

**Pergunta:** Se o BigQuery não tem chaves estrangeiras, como garantimos que um `ID_Cliente` na tabela de vendas realmente existe na tabela de clientes?  
**Resposta:**  
- Garantimos a integridade logicamente, usando **JOINs** nas consultas para relacionar as tabelas corretamente.

---

### 3️⃣ Sobre inserção de dados
**Pergunta:** Por que é uma boa prática inserir os clientes e produtos em suas próprias tabelas antes de inserir os dados de vendas?  
**Resposta:**  
- Evita duplicidade de registros.  
- Mantém consistência referencial lógica.  
- Facilita atualizações futuras nos dados de clientes ou produtos.

**Pergunta:** Em um cenário com milhões de vendas por dia, o `INSERT INTO` seria a melhor abordagem?  
**Resposta:**  
- Não. Para grandes volumes, é mais eficiente usar **carga em massa (bulk load)** ou pipelines automatizados, pois o `INSERT INTO` linha a linha seria muito lento.

---

### 4️⃣ Sobre Views
**Pergunta:** Qual é a principal vantagem de usar uma `VIEW` em vez de simplesmente salvar o código em um arquivo de texto?  
**Resposta:**  
- A `VIEW` automatiza consultas complexas, padroniza resultados e atualiza os dados automaticamente conforme as tabelas originais mudam.  
- Facilita o acesso de todos os usuários sem precisar reescrever a lógica.

**Pergunta:** Se o preço de um produto mudar na tabela `Produtos`, o `Valor_Total` na `VIEW` será atualizado automaticamente na próxima vez que a consultarmos?  
**Resposta:**  
- Sim. Como a `VIEW` é uma consulta dinâmica, qualquer alteração nas tabelas base é refletida automaticamente nos resultados.



