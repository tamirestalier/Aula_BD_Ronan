# Atividade de Aprofundamento em Scripts SQL - Banco de Dados Sakila

## Item 1 - Tipos de JOIN

### Conceito
JOINs são operações que combinam dados de duas ou mais tabelas. Principais tipos:
- **INNER JOIN**: Retorna apenas registros com correspondência em ambas as tabelas.
- **LEFT JOIN**: Retorna todos os registros da tabela à esquerda, mesmo sem correspondência.
- **RIGHT JOIN**: Similar ao LEFT JOIN, mas prioriza a tabela à direita.

### 🎯 Desafio 1: Relatório de clientes e filmes alugados
```sql
SELECT 
    c.first_name,
    c.last_name,
    f.title
FROM customer c
INNER JOIN rental r ON c.customer_id = r.customer_id
INNER JOIN inventory i ON r.inventory_id = i.inventory_id
INNER JOIN film f ON i.film_id = f.film_id
ORDER BY c.last_name;
```
### 🎯 Desafio 2: Clientes sem locações
```sql
SELECT 
    c.first_name,
    c.last_name
FROM customer c
LEFT JOIN rental r ON c.customer_id = r.customer_id
WHERE r.rental_id IS NULL;
```
------------------------------------------------------------------------------------------------------
### Item 2 - Múltiplos JOINs
Conceito
Permitem consultar dados relacionados em várias tabelas em uma única query, conectando tabelas intermediárias.

🎯 Desafio 1: Histórico completo de locações
```sql
SELECT 
    c.first_name,
    c.last_name,
    f.title,
    r.rental_date,
    s.store_id
FROM customer c
INNER JOIN rental r ON c.customer_id = r.customer_id
INNER JOIN inventory i ON r.inventory_id = i.inventory_id
INNER JOIN film f ON i.film_id = f.film_id
INNER JOIN store s ON i.store_id = s.store_id
ORDER BY r.rental_date DESC;
```
🎯 Desafio 2: Pagamentos com detalhes de atendente
```sql
SELECT 
    c.first_name AS cliente,
    p.amount,
    s.first_name AS atendente
FROM payment p
INNER JOIN customer c ON p.customer_id = c.customer_id
INNER JOIN staff s ON p.staff_id = s.staff_id
ORDER BY p.amount DESC;
```
------------------------------------------------------------------------------------------------------
#### Item 3 - Funções na Cláusula SELECT
Conceito
Funções que transformam dados durante a consulta:

CONCAT(): Junta strings

UPPER(): Converte para maiúsculas

ROUND(): Arredonda números

 🎯 Desafio 1: Formatar nomes de clientes

```sql
SELECT 
    CONCAT(UPPER(first_name), ' ', UPPER(last_name)) AS nome_formatado
FROM customer;
```
🎯 Desafio 2: Arredondar valores de pagamentos

```sql
SELECT 
    payment_id,
    ROUND(amount, 1) AS valor_arredondado
FROM payment;
```
------------------------------------------------------------------------------------------------
### Item 4 - Subconsultas (Subqueries)
Conceito
Consultas aninhadas dentro de outras consultas, úteis para filtros complexos.

🎯 Desafio 1: Filmes nunca alugados

```sql
SELECT title FROM film
WHERE film_id NOT IN (
    SELECT DISTINCT film_id FROM inventory
);

🎯 Desafio 2: Clientes que gastaram acima da média

SELECT first_name, last_name
FROM customer
WHERE customer_id IN (
    SELECT customer_id FROM payment
    GROUP BY customer_id
    HAVING SUM(amount) > (SELECT AVG(total) FROM (
        SELECT SUM(amount) AS total FROM payment GROUP BY customer_id
    ) AS avg_totals)
);
```
-------------------------------------------------------------------------------------------------
### Item 5 - Subconsultas em Comandos DML
Conceito
Subconsultas podem ser usadas em INSERT, UPDATE e DELETE para operações baseadas em condições complexas.

🎯 Desafio 1: Inativar clientes sem locações
```sql
UPDATE customer  
SET active = 0  
WHERE customer_id NOT IN (  
    SELECT DISTINCT customer_id FROM rental  
);  

  
🎯 Desafio 2: Deletar pagamentos de filmes sem estoque
DELETE FROM payment  
WHERE rental_id IN (  
    SELECT rental_id FROM rental  
    WHERE inventory_id IN (  
        SELECT inventory_id FROM inventory  
        WHERE film_id NOT IN (SELECT film_id FROM inventory)  
    )  
);  

```
-------------------------------------------------------------------------------------------------------
### Item 6 - WITH AS (CTE) e DISTINCT
Conceito
CTE (WITH AS): Cria consultas temporárias nomeadas para melhorar legibilidade.

DISTINCT: Remove duplicatas dos resultados.
🎯 Desafio 1: Top 5 filmes mais alugados com CTE
```sql
WITH film_rentals AS (  
    SELECT  
        f.film_id,  
        f.title,  
        COUNT(r.rental_id) AS total_locacoes  
    FROM film f  
    JOIN inventory i ON f.film_id = i.film_id  
    JOIN rental r ON i.inventory_id = r.inventory_id  
    GROUP BY f.film_id  
)  
SELECT * FROM film_rentals  
ORDER BY total_locacoes DESC  
LIMIT 5;  

🎯 Desafio 2: Clientes únicos por loja com DISTINCT;

SELECT DISTINCT  
    s.store_id,  
    c.customer_id,  
    c.first_name  
FROM store s  
JOIN customer c ON s.store_id = c.store_id;  
```
--------------------------------------------------------------------------------------------------
### Item 7 - Funções de Janela (Window Functions)
Conceito
Funções que operam em um conjunto de linhas relacionadas à linha atual:

ROW_NUMBER(): Número sequencial único

RANK(): Rank com pulos em empates

DENSE_RANK(): Rank sem pular posições

🎯 Desafio 1: Ranking de clientes por gasto
```sql
SELECT  
    customer_id,  
    SUM(amount) AS total_gasto,  
    DENSE_RANK() OVER (ORDER BY SUM(amount) DESC) AS ranking  
FROM payment  
GROUP BY customer_id;  

🎯 Desafio 2: Número de locações por mês
SELECT  
    DATE_FORMAT(rental_date, '%Y-%m') AS mes,  
    COUNT(*) AS locacoes,  
    RANK() OVER (ORDER BY COUNT(*) DESC) AS ranking  
FROM rental  
GROUP BY mes;  
```
---------------------------------------------------------------------------------------------------

### Item 8 - Lógica Condicional com CASE
Conceito
Estrutura CASE WHEN para criar colunas condicionais.

-- Alternativa correta:
```sql
SELECT  
    c.customer_id,
    CASE  
        WHEN MAX(r.rental_date) IS NULL THEN 'Inativo'  
        WHEN MAX(r.rental_date) < DATE_SUB(NOW(), INTERVAL 6 MONTH) THEN 'Adormecido'  
        ELSE 'Ativo'  
    END AS status  
FROM customer c
LEFT JOIN rental r ON c.customer_id = r.customer_id
GROUP BY c.customer_id;
```
-----------------------------------------------------------------------------------------------------
### Item 9 - Combinando Resultados com UNION
Conceito
UNION: Remove duplicatas

UNION ALL: Mantém duplicatas

🎯 Desafio 1: Lista unificada de nomes (clientes e atores)
```sql
SELECT first_name, last_name, 'Cliente' AS tipo FROM customer  
UNION  
SELECT first_name, last_name, 'Ator' AS tipo FROM actor;  
-------
🎯 Desafio 2: Filmes alugados ou comprados
SELECT film_id, 'Aluguel' AS tipo FROM rental  
JOIN inventory USING(inventory_id)  
UNION  
SELECT film_id, 'Venda' FROM payment  
JOIN rental USING(rental_id)  
JOIN inventory USING(inventory_id);  
```
------------------------------------------------------------------------------------------------------

### Item 10 - Visões (Views)
Conceito
Tabelas virtuais baseadas em consultas SQL.

🎯 Desafio 1: Criar view de pagamentos detalhados
```sql
CREATE VIEW view_pagamentos AS  
SELECT  
    p.payment_id,  
    c.first_name AS cliente,  
    p.amount  
FROM payment p  
JOIN customer c ON p.customer_id = c.customer_id;  
-----
🎯 Desafio 2: Consultar view com filtro
SELECT * FROM view_pagamentos  
WHERE amount > 5.00;  
```
-----------------------------------------------------------------------------------------------------
### Item 11 - Filtragem com WHERE vs. HAVING
Conceito
WHERE: Filtra linhas antes do agrupamento

HAVING: Filtra grupos após o agrupamento

🎯 Desafio 1: Filmes alugados mais de 30 vezes
```sql
SELECT film_id, COUNT(*) AS locacoes  
FROM rental  
JOIN inventory USING(inventory_id)  
GROUP BY film_id  
HAVING locacoes > 30; 
----
🎯 Desafio 2: Clientes com gasto médio acima de X
SELECT  
    customer_id,  
    AVG(amount) AS media_pagamentos  
FROM payment  
GROUP BY customer_id  
HAVING media_pagamentos > 4.50;  
```
---------------------------------------------------------------------------------------------------
### Item 12 - Controle de Transações
Conceito
START TRANSACTION, COMMIT, e ROLLBACK para operações atômicas.

🎯 Desafio 1: Transação de aluguel
```sql
START TRANSACTION;  
INSERT INTO rental (...) VALUES (...);  
UPDATE inventory SET last_update = NOW() WHERE inventory_id = X;  
COMMIT;  
----
🎯 Desafio 2: Rollback em erro
BEGIN;  
DELETE FROM payment WHERE payment_id = 100;  
-- Se ocorrer erro:  
ROLLBACK;  
```
CORRIGIDO
```SQL
START TRANSACTION;
INSERT INTO rental (rental_date, inventory_id, customer_id, return_date, staff_id)
VALUES (NOW(), 1, 1, NULL, 1);
UPDATE inventory SET last_update = NOW() WHERE inventory_id = 1;
COMMIT;
```
---------------------------------------------------------------------------------------------------
### Item 13 - Gatilhos (Triggers)
Conceito
Código executado automaticamente em eventos (INSERT/UPDATE/DELETE).

🎯 Desafio 1: Trigger para log de atualizações
```sql
CREATE TRIGGER log_update_film  
AFTER UPDATE ON film  
FOR EACH ROW  
INSERT INTO film_log VALUES (OLD.film_id, NOW());  

----
🎯 Desafio 2: Validar estoque antes de aluguel
CREATE TRIGGER valida_estoque  
BEFORE INSERT ON rental  
FOR EACH ROW  
BEGIN  
    IF (SELECT COUNT(*) FROM inventory WHERE inventory_id = NEW.inventory_id) = 0 THEN  
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Item sem estoque';  
    END IF;  
END;  
```
-----------------------------------------------------------------------------------------------------
### Item 14 - Índices e Otimização
Conceito
Estruturas que aceleram buscas em colunas específicas.

🎯 Desafio 1: Criar índice para buscas por título
```sql
CREATE INDEX idx_film_title ON film(title);  
-----
🎯 Desafio 2: Analisar performance
EXPLAIN SELECT * FROM film WHERE title LIKE 'A%';  