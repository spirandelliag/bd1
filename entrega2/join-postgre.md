## Parte 2 — Preparação do ambiente

### 3. Criação das tabelas

```sql
CREATE TABLE categoria (
    id_categoria BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);

CREATE TABLE produto (
    id_produto BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco NUMERIC(10,2) NOT NULL CHECK (preco >= 0),
    id_categoria BIGINT NOT NULL,

    CONSTRAINT fk_produto_categoria
        FOREIGN KEY (id_categoria)
        REFERENCES categoria(id_categoria)
);
```

### 4. Inserção dos dados

```sql
INSERT INTO categoria (nome)
VALUES
('Informática'),
('Móveis'),
('Papelaria'),
('Limpeza');

INSERT INTO produto (nome, preco, id_categoria)
VALUES
('Notebook', 3500.00, 1),
('Mouse', 80.00, 1),
('Mesa', 900.00, 2),
('Cadeira', 600.00, 2),
('Caneta', 5.00, 3);
```

Os IDs gerados são:
- Categorias: 1 = Informática, 2 = Móveis, 3 = Papelaria, 4 = Limpeza
- Produtos: 1 = Notebook, 2 = Mouse, 3 = Mesa, 4 = Cadeira, 5 = Caneta

A categoria Limpeza (id 4) não possui produtos propositalmente.

---

## Parte 3 — INNER JOIN

### 6. Primeiro exemplo

```sql
SELECT
    produto.nome,
    produto.preco,
    categoria.nome
FROM produto
INNER JOIN categoria
    ON produto.id_categoria = categoria.id_categoria;
```

### 7. Identificando colunas com AS

```sql
SELECT
    produto.nome AS produto,
    produto.preco,
    categoria.nome AS categoria
FROM produto
INNER JOIN categoria
    ON produto.id_categoria = categoria.id_categoria;
```

Resultado:

| produto | preco | categoria |
|---------|-------|-----------|
| Notebook | 3500,00 | Informática |
| Mouse | 80,00 | Informática |
| Mesa | 900,00 | Móveis |
| Cadeira | 600,00 | Móveis |
| Caneta | 5,00 | Papelaria |

### 8. Apelidos para tabelas

```sql
SELECT
    p.nome AS produto,
    p.preco,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria;
```

---

## Parte 4 — JOIN combinado com WHERE e ORDER BY

### 10. Filtrando registros relacionados

```sql
SELECT
    p.nome AS produto,
    p.preco,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria
WHERE c.nome = 'Informática';
```

Resultado:

| produto | preco | categoria |
|---------|-------|-----------|
| Notebook | 3500,00 | Informática |
| Mouse | 80,00 | Informática |

### 11. Múltiplas condições

```sql
SELECT
    p.nome AS produto,
    p.preco,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria
WHERE c.nome = 'Informática'
  AND p.preco > 100;
```

Resultado: Notebook, 3500,00, Informática.

### 12. Ordenando resultados

```sql
SELECT
    p.nome AS produto,
    p.preco,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria
ORDER BY p.preco DESC;
```

Resultado ordenado do mais caro para o mais barato.

---

## Parte 5 — LEFT JOIN

### 14. Conceito e exemplo

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto
FROM categoria c
LEFT JOIN produto p
    ON c.id_categoria = p.id_categoria
ORDER BY c.nome, p.nome;
```

Resultado:

| categoria | produto |
|-----------|---------|
| Informática | Mouse |
| Informática | Notebook |
| Limpeza | NULL |
| Móveis | Cadeira |
| Móveis | Mesa |
| Papelaria | Caneta |

A categoria Limpeza aparece com produto NULL porque não tem produtos associados.

### 15. Categorias sem produtos

```sql
SELECT c.nome AS categoria
FROM categoria c
LEFT JOIN produto p
    ON c.id_categoria = p.id_categoria
WHERE p.id_produto IS NULL;
```

Resultado:

| categoria |
|-----------|
| Limpeza |

---

## Parte 6 — RIGHT JOIN

Equivalente ao LEFT JOIN com a ordem das tabelas invertida:

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto
FROM produto p
RIGHT JOIN categoria c
    ON p.id_categoria = c.id_categoria;
```

Resultado: mesmo do LEFT JOIN anterior (todas as categorias aparecem, inclusive Limpeza com produto NULL).

---

## Parte 7 — FULL OUTER JOIN

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto
FROM categoria c
FULL OUTER JOIN produto p
    ON c.id_categoria = p.id_categoria;
```

Neste exemplo, como todo produto possui categoria obrigatória, o resultado é semelhante ao LEFT JOIN iniciado por categoria. A diferença aparece quando há linhas sem correspondência em ambos os lados.

---

## Parte 8 — CROSS JOIN

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto
FROM categoria c
CROSS JOIN produto p;
```

Com 4 categorias e 5 produtos, resultam 20 combinações (produto cartesiano). Não utiliza condição ON.

---

## Parte 9 — Comparação dos tipos de JOIN

| Tipo | O que retorna |
|------|---------------|
| INNER JOIN | Apenas linhas que satisfazem a condição em ambas as tabelas |
| LEFT JOIN | Todas as linhas da esquerda + correspondências da direita |
| RIGHT JOIN | Todas as linhas da direita + correspondências da esquerda |
| FULL OUTER JOIN | Todas as linhas de ambos os lados |
| CROSS JOIN | Todas as combinações possíveis |

---

## Parte 10 — JOIN com três ou mais tabelas

### Criação das tabelas para o exercício de pedidos

```sql
CREATE TABLE cliente (
    id_cliente BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);

CREATE TABLE pedido (
    id_pedido BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    data_pedido DATE NOT NULL,
    id_cliente BIGINT NOT NULL,

    CONSTRAINT fk_pedido_cliente
        FOREIGN KEY (id_cliente)
        REFERENCES cliente(id_cliente)
);

CREATE TABLE item_pedido (
    id_item BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    id_pedido BIGINT NOT NULL,
    id_produto BIGINT NOT NULL,
    quantidade INTEGER NOT NULL CHECK (quantidade > 0),

    CONSTRAINT fk_item_pedido
        FOREIGN KEY (id_pedido)
        REFERENCES pedido(id_pedido),

    CONSTRAINT fk_item_produto
        FOREIGN KEY (id_produto)
        REFERENCES produto(id_produto)
);
```

### Inserção de dados

```sql
INSERT INTO cliente (nome) VALUES ('Ana');
INSERT INTO cliente (nome) VALUES ('Bruno');
INSERT INTO cliente (nome) VALUES ('Carlos');
INSERT INTO cliente (nome) VALUES ('Daniela');

INSERT INTO pedido (data_pedido, id_cliente)
VALUES
('2026-08-01', 1),
('2026-08-05', 1),
('2026-08-02', 2),
('2026-08-03', 3),
('2026-08-10', 3),
('2026-08-12', 3);

INSERT INTO item_pedido (id_pedido, id_produto, quantidade)
VALUES
(1, 1, 1),
(1, 2, 2),
(2, 3, 1),
(3, 4, 1),
(4, 5, 1),
(5, 1, 2),
(6, 3, 3);
```

### Consulta com JOIN em três tabelas

```sql
SELECT
    c.nome AS cliente,
    p.id_pedido,
    p.data_pedido,
    pr.nome AS produto,
    ip.quantidade
FROM cliente c
INNER JOIN pedido p
    ON c.id_cliente = p.id_cliente
INNER JOIN item_pedido ip
    ON p.id_pedido = ip.id_pedido
INNER JOIN produto pr
    ON ip.id_produto = pr.id_produto;
```

---

## Parte 14 — Exercícios práticos

### Nível 1 — INNER JOIN

**1. Nome de cada produto e sua categoria.**

```sql
SELECT
    p.nome AS produto,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria;
```

**2. Nome, preço e categoria, em ordem decrescente de preço.**

```sql
SELECT
    p.nome AS produto,
    p.preco,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria
ORDER BY p.preco DESC;
```

**3. Somente produtos de Informática.**

```sql
SELECT
    p.nome AS produto,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria
WHERE c.nome = 'Informática';
```

**4. Produtos com preço superior a R$ 500 e suas categorias.**

```sql
SELECT
    p.nome AS produto,
    p.preco,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria
WHERE p.preco > 500;
```

**5. Produtos de Informática ou Móveis, em ordem alfabética.**

```sql
SELECT
    p.nome AS produto,
    c.nome AS categoria
FROM produto p
INNER JOIN categoria c
    ON p.id_categoria = c.id_categoria
WHERE c.nome IN ('Informática', 'Móveis')
ORDER BY p.nome;
```

---

### Nível 2 — LEFT JOIN

**6. Liste todas as categorias, incluindo as que não possuem produtos.**

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto
FROM categoria c
LEFT JOIN produto p
    ON c.id_categoria = p.id_categoria
ORDER BY c.nome;
```

**7. Identifique as categorias sem produtos.**

```sql
SELECT c.nome AS categoria
FROM categoria c
LEFT JOIN produto p
    ON c.id_categoria = p.id_categoria
WHERE p.id_produto IS NULL;
```

**8. Liste todas as categorias e associe somente produtos com preço superior a R$ 100, preservando categorias sem correspondências.**

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto,
    p.preco
FROM categoria c
LEFT JOIN produto p
    ON c.id_categoria = p.id_categoria
   AND p.preco > 100
ORDER BY c.nome;
```

---

### Nível 3 — Comparações

**9. Execute uma consulta com INNER JOIN e outra com LEFT JOIN; compare os resultados.**

INNER JOIN:

```sql
SELECT c.nome AS categoria, p.nome AS produto
FROM categoria c
INNER JOIN produto p
    ON c.id_categoria = p.id_categoria;
```

LEFT JOIN:

```sql
SELECT c.nome AS categoria, p.nome AS produto
FROM categoria c
LEFT JOIN produto p
    ON c.id_categoria = p.id_categoria;
```

Comparação: o INNER JOIN não mostra a categoria Limpeza (sem produtos). O LEFT JOIN mostra todas as categorias, incluindo Limpeza com produto NULL.

**10. Reescreva uma consulta com LEFT JOIN usando RIGHT JOIN e preserve o resultado.**

LEFT JOIN:

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto
FROM categoria c
LEFT JOIN produto p
    ON c.id_categoria = p.id_categoria;
```

RIGHT JOIN equivalente:

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto
FROM produto p
RIGHT JOIN categoria c
    ON p.id_categoria = c.id_categoria;
```

**11. Execute um CROSS JOIN e explique a quantidade de linhas.**

```sql
SELECT
    c.nome AS categoria,
    p.nome AS produto
FROM categoria c
CROSS JOIN produto p;
```

Quantidade de linhas: 4 categorias × 5 produtos = 20 combinações. Cada categoria combina com cada produto, independente de existirem relacionados.

---

### Nível 4 — Modelagem e consulta

**12. Crie cliente e pedido em um relacionamento 1:N. Cadastre três clientes e quatro pedidos, deixando um cliente sem pedidos.**

Tabelas já criadas acima. Dados inseridos: 4 clientes (Ana, Bruno, Carlos, Daniela) e 6 pedidos. Daniela (id 4) não possui pedidos.

**a) Mostrar os pedidos com seus clientes.**

```sql
SELECT
    c.nome AS cliente,
    p.id_pedido,
    p.data_pedido
FROM cliente c
INNER JOIN pedido p
    ON c.id_cliente = p.id_cliente
ORDER BY p.data_pedido;
```

**b) Mostrar todos os clientes, inclusive os sem pedidos.**

```sql
SELECT
    c.nome AS cliente,
    p.id_pedido
FROM cliente c
LEFT JOIN pedido p
    ON c.id_cliente = p.id_cliente
ORDER BY c.nome;
```

**c) Mostrar apenas clientes sem pedidos.**

```sql
SELECT c.nome AS cliente
FROM cliente c
LEFT JOIN pedido p
    ON c.id_cliente = p.id_cliente
WHERE p.id_pedido IS NULL;
```

**d) Ordenar pedidos por data.**

```sql
SELECT
    c.nome AS cliente,
    p.data_pedido
FROM cliente c
INNER JOIN pedido p
    ON c.id_cliente = p.id_cliente
ORDER BY p.data_pedido;
```

**e) Mostrar os pedidos de um cliente específico (exemplo: Ana).**

```sql
SELECT
    p.id_pedido,
    p.data_pedido
FROM cliente c
INNER JOIN pedido p
    ON c.id_cliente = p.id_cliente
WHERE c.nome = 'Ana';
```

---

## Observações importantes

- `INNER JOIN` retorna apenas combinações que satisfazem a condição `ON`.
- `LEFT JOIN` preserva todas as linhas da tabela da esquerda.
- Em `LEFT JOIN`, colunas da tabela da direita recebem `NULL` quando não há correspondência.
- Não use `COUNT(*)` para contar itens em `LEFT JOIN` quando quiser ignorar linhas sem correspondência; use `COUNT(coluna_da_direita)`.
- Qualifique as colunas com apelidos de tabela (`p.nome`, `c.nome`) para evitar ambiguidade.
- Filtros em colunas da tabela opcional (`WHERE p.preco > 100`) transformam `LEFT JOIN` em `INNER JOIN`; coloque a condição em `ON` se quiser preservar as linhas sem correspondência.





