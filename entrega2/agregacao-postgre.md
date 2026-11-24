
Resolução dos exercícios da aula sobre funções de agregação (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP BY`, `HAVING`, `FILTER`, `STRING_AGG`, `ARRAY_AGG`, `BOOL_AND`, `BOOL_OR`).

Todos os comandos foram pensados para rodar no SQL Editor do Supabase/PostgreSQL.

---

## Parte 2 — Preparando o banco

Primeiro, crio a tabela e insiro os dados do exemplo.

### Criação da tabela

```sql
CREATE TABLE produto_agregacao (
    id_produto BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    categoria VARCHAR(50) NOT NULL,
    preco NUMERIC(10,2) NOT NULL CHECK (preco >= 0),
    estoque INTEGER NOT NULL DEFAULT 0 CHECK (estoque >= 0)
);
```

### Inserção dos dados

```sql
INSERT INTO produto_agregacao (nome, categoria, preco, estoque)
VALUES
('Notebook', 'Informática', 3500.00, 5),
('Mouse', 'Informática', 80.00, 20),
('Teclado', 'Informática', 150.00, 15),
('Monitor', 'Informática', 1200.00, 8),
('Cadeira', 'Móveis', 600.00, 10),
('Mesa', 'Móveis', 900.00, 4),
('Armário', 'Móveis', 1500.00, 3),
('Caneta', 'Papelaria', 5.00, 100),
('Caderno', 'Papelaria', 25.00, 50),
('Borracha', 'Papelaria', 3.00, 80);
```

### Verificação dos dados

```sql
SELECT * FROM produto_agregacao;
```

Resultado esperado:

| id_produto | nome | categoria | preco | estoque |
|------------|------|-----------|-------|---------|
| 1 | Notebook | Informática | 3500,00 | 5 |
| 2 | Mouse | Informática | 80,00 | 20 |
| 3 | Teclado | Informática | 150,00 | 15 |
| 4 | Monitor | Informática | 1200,00 | 8 |
| 5 | Cadeira | Móveis | 600,00 | 10 |
| 6 | Mesa | Móveis | 900,00 | 4 |
| 7 | Armário | Móveis | 1500,00 | 3 |
| 8 | Caneta | Papelaria | 5,00 | 100 |
| 9 | Caderno | Papelaria | 25,00 | 50 |
| 10 | Borracha | Papelaria | 3,00 | 80 |

---

## Parte 14 — Exercícios

### Nível 1 — Funções básicas

**1. Conte todos os produtos.**

```sql
SELECT COUNT(*) AS total_produtos
FROM produto_agregacao;
```

Resultado: `10`.

**2. Conte as categorias distintas.**

```sql
SELECT COUNT(DISTINCT categoria) AS total_categorias
FROM produto_agregacao;
```

Resultado: `3` (Informática, Móveis, Papelaria).

**3. Calcule o total de unidades em estoque.**

```sql
SELECT SUM(estoque) AS total_unidades_estoque
FROM produto_agregacao;
```

Resultado: `295`.

**4. Calcule o valor financeiro total do estoque.**

```sql
SELECT SUM(preco * estoque) AS valor_total_estoque
FROM produto_agregacao;
```

Resultado: `47040,00`.

**5. Calcule o preço médio.**

```sql
SELECT ROUND(AVG(preco), 2) AS preco_medio
FROM produto_agregacao;
```

Resultado: `796,30`.

**6. Encontre o menor e o maior preço.**

```sql
SELECT
    MIN(preco) AS menor_preco,
    MAX(preco) AS maior_preco
FROM produto_agregacao;
```

Resultado: `menor_preco = 3,00` e `maior_preco = 3500,00`.

**7. Mostre todos os produtos empatados no menor preço.**

```sql
SELECT nome, preco
FROM produto_agregacao
WHERE preco = (SELECT MIN(preco) FROM produto_agregacao);
```

Resultado: Borracha, `3,00`.

---

### Nível 2 — GROUP BY

**8. Conte os produtos por categoria.**

```sql
SELECT
    categoria,
    COUNT(*) AS total_produtos
FROM produto_agregacao
GROUP BY categoria
ORDER BY categoria;
```

Resultado esperado:

| categoria | total_produtos |
|-----------|----------------|
| Informática | 4 |
| Móveis | 3 |
| Papelaria | 3 |

**9. Calcule o estoque total por categoria.**

```sql
SELECT
    categoria,
    SUM(estoque) AS estoque_total
FROM produto_agregacao
GROUP BY categoria
ORDER BY categoria;
```

Resultado esperado:

| categoria | estoque_total |
|-----------|---------------|
| Informática | 48 |
| Móveis | 17 |
| Papelaria | 230 |

**10. Calcule preço médio, mínimo e máximo por categoria.**

```sql
SELECT
    categoria,
    ROUND(AVG(preco), 2) AS preco_medio,
    MIN(preco) AS menor_preco,
    MAX(preco) AS maior_preco
FROM produto_agregacao
GROUP BY categoria
ORDER BY categoria;
```

Resultado esperado:

| categoria | preco_medio | menor_preco | maior_preco |
|-----------|-------------|-------------|-------------|
| Informática | 1232,50 | 80,00 | 3500,00 |
| Móveis | 1000,00 | 600,00 | 1500,00 |
| Papelaria | 11,00 | 3,00 | 25,00 |

**11. Calcule o valor financeiro do estoque por categoria.**

```sql
SELECT
    categoria,
    SUM(preco * estoque) AS valor_estoque
FROM produto_agregacao
GROUP BY categoria
ORDER BY categoria;
```

Resultado esperado:

| categoria | valor_estoque |
|-----------|---------------|
| Informática | 12400,00 |
| Móveis | 11700,00 |
| Papelaria | 2940,00 |

**12. Liste alfabeticamente os produtos de cada categoria em uma string.**

```sql
SELECT
    categoria,
    STRING_AGG(nome, ', ' ORDER BY nome) AS produtos
FROM produto_agregacao
GROUP BY categoria
ORDER BY categoria;
```

Resultado esperado:

| categoria | produtos |
|-----------|----------|
| Informática | Monitor, Mouse, Notebook, Teclado |
| Móveis | Armário, Cadeira, Mesa |
| Papelaria | Borracha, Caderno, Caneta |

---

### Nível 3 — WHERE, HAVING e FILTER

**13. Conte por categoria somente produtos com preço acima de R$ 100.**

```sql
SELECT
    categoria,
    COUNT(*) AS total_produtos
FROM produto_agregacao
WHERE preco > 100
GROUP BY categoria
ORDER BY categoria;
```

Resultado esperado:

| categoria | total_produtos |
|-----------|----------------|
| Informática | 3 |
| Móveis | 3 |

**14. Mostre categorias com pelo menos três produtos.**

```sql
SELECT
    categoria,
    COUNT(*) AS total_produtos
FROM produto_agregacao
GROUP BY categoria
HAVING COUNT(*) >= 3
ORDER BY categoria;
```

Resultado esperado:

| categoria | total_produtos |
|-----------|----------------|
| Informática | 4 |
| Móveis | 3 |
| Papelaria | 3 |

**15. Mostre categorias com preço médio acima de R$ 500.**

```sql
SELECT
    categoria,
    ROUND(AVG(preco), 2) AS preco_medio
FROM produto_agregacao
GROUP BY categoria
HAVING AVG(preco) > 500
ORDER BY preco_medio DESC;
```

Resultado esperado:

| categoria | preco_medio |
|-----------|-------------|
| Informática | 1232,50 |
| Móveis | 1000,00 |

**16. Em uma linha, conte todos os produtos, os de preço acima de R$ 1000 e os com estoque abaixo de 10.**

```sql
SELECT
    COUNT(*) AS total_produtos,
    COUNT(*) FILTER (WHERE preco > 1000) AS produtos_caros,
    COUNT(*) FILTER (WHERE estoque < 10) AS estoque_baixo
FROM produto_agregacao;
```

Resultado esperado:

| total_produtos | produtos_caros | estoque_baixo |
|----------------|----------------|---------------|
| 10 | 3 | 4 |

**17. Mostre categorias cujo valor total de estoque supera R$ 10.000.**

```sql
SELECT
    categoria,
    SUM(preco * estoque) AS valor_estoque
FROM produto_agregacao
GROUP BY categoria
HAVING SUM(preco * estoque) > 10000
ORDER BY valor_estoque DESC;
```

Resultado esperado:

| categoria | valor_estoque |
|-----------|---------------|
| Informática | 12400,00 |
| Móveis | 11700,00 |

---

### Nível 4 — JOIN e agregação

Para os exercícios abaixo, crio um esquema normalizado separado para não conflitar com a tabela anterior.

**Criação das tabelas normalizadas:**

```sql
CREATE TABLE categoria_normalizada (
    id_categoria BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR(50) NOT NULL
);

CREATE TABLE produto_normalizado (
    id_produto BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco NUMERIC(10,2) NOT NULL CHECK (preco >= 0),
    estoque INTEGER NOT NULL DEFAULT 0 CHECK (estoque >= 0),
    id_categoria BIGINT NOT NULL,
    CONSTRAINT fk_produto_categoria
        FOREIGN KEY (id_categoria)
        REFERENCES categoria_normalizada(id_categoria)
);
```

**Inserção de categorias e produtos:**

```sql
INSERT INTO categoria_normalizada (nome) VALUES ('Informática');
INSERT INTO categoria_normalizada (nome) VALUES ('Móveis');
INSERT INTO categoria_normalizada (nome) VALUES ('Papelaria');
INSERT INTO categoria_normalizada (nome) VALUES ('Brinquedos');

INSERT INTO produto_normalizado (nome, preco, estoque, id_categoria)
VALUES
('Notebook', 3500.00, 5, 1),
('Mouse', 80.00, 20, 1),
('Teclado', 150.00, 15, 1),
('Monitor', 1200.00, 8, 1),
('Cadeira', 600.00, 10, 2),
('Mesa', 900.00, 4, 2),
('Armário', 1500.00, 3, 2),
('Caneta', 5.00, 100, 3),
('Caderno', 25.00, 50, 3),
('Borracha', 3.00, 80, 3);
```

**18. Conte produtos por categoria incluindo categorias vazias.**

```sql
SELECT
    c.nome AS categoria,
    COUNT(p.id_produto) AS total_produtos
FROM categoria_normalizada c
LEFT JOIN produto_normalizado p
    ON p.id_categoria = c.id_categoria
GROUP BY c.id_categoria, c.nome
ORDER BY c.nome;
```

Resultado esperado:

| categoria | total_produtos |
|-----------|----------------|
| Brinquedos | 0 |
| Informática | 4 |
| Móveis | 3 |
| Papelaria | 3 |

**19. Mostre somente categorias vazias.**

```sql
SELECT
    c.nome AS categoria
FROM categoria_normalizada c
LEFT JOIN produto_normalizado p
    ON p.id_categoria = c.id_categoria
GROUP BY c.id_categoria, c.nome
HAVING COUNT(p.id_produto) = 0
ORDER BY c.nome;
```

Resultado esperado:

| categoria |
|-----------|
| Brinquedos |

**20. Calcule quantidade, média e valor do estoque por categoria.**

```sql
SELECT
    c.nome AS categoria,
    COUNT(p.id_produto) AS quantidade_produtos,
    ROUND(AVG(p.preco), 2) AS preco_medio,
    COALESCE(SUM(p.preco * p.estoque), 0) AS valor_estoque
FROM categoria_normalizada c
LEFT JOIN produto_normalizado p
    ON p.id_categoria = c.id_categoria
GROUP BY c.id_categoria, c.nome
ORDER BY c.nome;
```

Resultado esperado:

| categoria | quantidade_produtos | preco_medio | valor_estoque |
|-----------|---------------------|-------------|---------------|
| Brinquedos | 0 | NULL | 0 |
| Informática | 4 | 1232,50 | 12400,00 |
| Móveis | 3 | 1000,00 | 11700,00 |
| Papelaria | 3 | 11,00 | 2940,00 |

**21. Explique por que `COUNT(*)` pode dar resultado incorreto em um `LEFT JOIN`.**

Em um `LEFT JOIN`, quando uma categoria não tem nenhum produto, o lado direito da junção produz `NULL` para todas as colunas da tabela `produto_normalizado`. Mesmo assim, existe uma linha no resultado (a linha da categoria). `COUNT(*)` conta todas as linhas, então retornaria `1` para uma categoria vazia, enquanto `COUNT(p.id_produto)` ignora os `NULL` e retorna corretamente `0`.

---

### Desafio

Para o desafio, crio as tabelas `cliente`, `pedido` e `item_pedido`.

**Criação das tabelas:**

```sql
CREATE TABLE cliente (
    id_cliente BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);

CREATE TABLE pedido (
    id_pedido BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_cliente BIGINT NOT NULL,
    data_pedido DATE NOT NULL,
    CONSTRAINT fk_pedido_cliente
        FOREIGN KEY (id_cliente)
        REFERENCES cliente(id_cliente)
);

CREATE TABLE item_pedido (
    id_item BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_pedido BIGINT NOT NULL,
    id_produto BIGINT NOT NULL,
    quantidade INTEGER NOT NULL CHECK (quantidade > 0),
    preco_unitario NUMERIC(10,2) NOT NULL CHECK (preco_unitario >= 0),
    CONSTRAINT fk_item_pedido
        FOREIGN KEY (id_pedido)
        REFERENCES pedido(id_pedido)
);
```

**Inserção de dados:**

```sql
INSERT INTO cliente (nome) VALUES ('Ana');
INSERT INTO cliente (nome) VALUES ('Bruno');
INSERT INTO cliente (nome) VALUES ('Carlos');
INSERT INTO cliente (nome) VALUES ('Daniela');

INSERT INTO pedido (id_cliente, data_pedido)
VALUES
(1, '2026-08-01'),
(1, '2026-08-05'),
(2, '2026-08-02'),
(3, '2026-08-03'),
(3, '2026-08-10'),
(3, '2026-08-12');

INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario)
VALUES
(1, 1, 1, 3500.00),  -- Pedido 1 de Ana: 1 Notebook
(1, 2, 2, 80.00),   -- Pedido 1 de Ana: 2 Mouse
(2, 3, 1, 150.00),  -- Pedido 2 de Ana: 1 Teclado
(3, 4, 1, 1200.00), -- Pedido de Bruno: 1 Monitor
(4, 5, 1, 600.00),  -- Pedido 1 de Carlos: 1 Cadeira
(5, 6, 2, 900.00),  -- Pedido 2 de Carlos: 2 Mesa
(5, 7, 1, 1500.00), -- Pedido 2 de Carlos: 1 Armário
(6, 8, 5, 5.00);    -- Pedido 3 de Carlos: 5 Caneta
```

**a. Total de pedidos por cliente, incluindo clientes sem pedidos.**

```sql
SELECT
    c.id_cliente,
    c.nome,
    COUNT(p.id_pedido) AS total_pedidos
FROM cliente c
LEFT JOIN pedido p
    ON p.id_cliente = c.id_cliente
GROUP BY c.id_cliente, c.nome
ORDER BY c.nome;
```

Resultado esperado:

| id_cliente | nome | total_pedidos |
|------------|------|---------------|
| 1 | Ana | 2 |
| 2 | Bruno | 1 |
| 3 | Carlos | 3 |
| 4 | Daniela | 0 |

**b. Valor total de cada pedido.**

```sql
SELECT
    p.id_pedido,
    cli.nome AS cliente,
    SUM(ip.quantidade * ip.preco_unitario) AS valor_total_pedido
FROM pedido p
INNER JOIN cliente cli ON cli.id_cliente = p.id_cliente
INNER JOIN item_pedido ip ON ip.id_pedido = p.id_pedido
GROUP BY p.id_pedido, cli.nome
ORDER BY p.id_pedido;
```

Resultado esperado:

| id_pedido | cliente | valor_total_pedido |
|-----------|---------|--------------------|
| 1 | Ana | 3660,00 |
| 2 | Ana | 150,00 |
| 3 | Bruno | 1200,00 |
| 4 | Carlos | 600,00 |
| 5 | Carlos | 3300,00 |
| 6 | Carlos | 25,00 |

**c. Total gasto por cliente.**

```sql
SELECT
    c.id_cliente,
    c.nome,
    COALESCE(SUM(ip.quantidade * ip.preco_unitario), 0) AS total_gasto
FROM cliente c
LEFT JOIN pedido p ON p.id_cliente = c.id_cliente
LEFT JOIN item_pedido ip ON ip.id_pedido = p.id_pedido
GROUP BY c.id_cliente, c.nome
ORDER BY c.nome;
```

Resultado esperado:

| id_cliente | nome | total_gasto |
|------------|------|-------------|
| 1 | Ana | 3810,00 |
| 2 | Bruno | 1200,00 |
| 3 | Carlos | 3925,00 |
| 4 | Daniela | 0,00 |

**d. Ticket médio por cliente.**

```sql
SELECT
    c.id_cliente,
    c.nome,
    ROUND(
        COALESCE(SUM(ip.quantidade * ip.preco_unitario), 0)::numeric
        / NULLIF(COUNT(p.id_pedido), 0)
    , 2) AS ticket_medio
FROM cliente c
LEFT JOIN pedido p ON p.id_cliente = c.id_cliente
LEFT JOIN item_pedido ip ON ip.id_pedido = p.id_pedido
GROUP BY c.id_cliente, c.nome
ORDER BY c.nome;
```

Resultado esperado:

| id_cliente | nome | ticket_medio |
|------------|------|--------------|
| 1 | Ana | 1905,00 |
| 2 | Bruno | 1200,00 |
| 3 | Carlos | 1308,33 |
| 4 | Daniela | NULL |

> Para Daniela, o ticket médio é `NULL` porque ela não tem pedidos (divisão por zero evitada com `NULLIF`).

**e. Clientes com total gasto acima de R$ 1.000.**

```sql
SELECT
    c.id_cliente,
    c.nome,
    COALESCE(SUM(ip.quantidade * ip.preco_unitario), 0) AS total_gasto
FROM cliente c
LEFT JOIN pedido p ON p.id_cliente = c.id_cliente
LEFT JOIN item_pedido ip ON ip.id_pedido = p.id_pedido
GROUP BY c.id_cliente, c.nome
HAVING COALESCE(SUM(ip.quantidade * ip.preco_unitario), 0) > 1000
ORDER BY total_gasto DESC;
```

Resultado esperado:

| id_cliente | nome | total_gasto |
|------------|------|-------------|
| 3 | Carlos | 3925,00 |
| 1 | Ana | 3810,00 |
| 2 | Bruno | 1200,00 |

**f. Produto mais vendido por quantidade, preservando empates.**

Para isso, somo a quantidade vendida de cada produto e mostro os que tiveram a maior quantidade.

```sql
SELECT
    ip.id_produto,
    SUM(ip.quantidade) AS quantidade_total_vendida
FROM item_pedido ip
GROUP BY ip.id_produto
HAVING SUM(ip.quantidade) = (
    SELECT MAX(qtd) FROM (
        SELECT SUM(quantidade) AS qtd
        FROM item_pedido
        GROUP BY id_produto
    ) AS sub
)
ORDER BY ip.id_produto;
```

Resultado esperado (com os dados de exemplo):

| id_produto | quantidade_total_vendida |
|------------|--------------------------|
| 8 | 5 |

> O produto 8 (Caneta) vendeu 5 unidades, que é a maior quantidade. Se houvesse empate, todos os produtos empatados apareceriam.

---

## Observações importantes

- `COUNT(*)` conta todas as linhas, incluindo aquelas com valores `NULL`.
- `COUNT(coluna)` ignora valores `NULL` nessa coluna.
- `COUNT(DISTINCT coluna)` conta apenas valores distintos e não nulos.
- `WHERE` filtra linhas antes do agrupamento; `HAVING` filtra grupos após o agrupamento.
- `NULLIF(valor, 0)` evita divisão por zero.
- `COALESCE(valor, 0)` substitui `NULL` por zero quando isso faz sentido para o negócio.



