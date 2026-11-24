
### Exercício 1 — Identificando redundâncias

Tabela `ALUNO_CURSO`:

| id_aluno | nome_aluno | id_curso | nome_curso | coordenador |
|----------|------------|----------|------------|-------------|
| 1 | Ana | 10 | Sistemas de Informação | Carlos |
| 2 | Bruno | 10 | Sistemas de Informação | Carlos |
| 3 | Maria | 20 | Ciência da Computação | Fernanda |
| 4 | Pedro | 10 | Sistemas de Informação | Carlos |
| 5 | Julia | 20 | Ciência da Computação | Fernanda |

**a) Informações armazenadas repetidamente:**

- `id_curso`
- `nome_curso`
- `coordenador`

Esses dados do curso se repetem para cada aluno matriculado nele.

**b) Por que essa repetição causa problemas:**

- Ocupa espaço desnecessário.
- Se precisar mudar o nome do curso ou o coordenador, tem que alterar em várias linhas.
- Risco de digitar errado em uma linha e ficar com informações diferentes para o mesmo curso.
- Se excluir um aluno, pode perder informação do curso.

**c) Se o coordenador do curso Sistemas de Informação mudar:**

Teriam que alterar as linhas da Ana, do Bruno e do Pedro. Se esquecer de alterar uma, o banco fica com dois coordenadores diferentes para o mesmo curso.

**d) Se o único aluno de determinado curso for excluído:**

Some junto com o aluno a informação do curso e do coordenador. O curso deixa de existir no banco, mesmo que ainda exista na vida real. Isso é perda de dados.

**e) Seria possível cadastrar um curso sem alunos nessa estrutura?**

Não. Como o curso só aparece ligado a um aluno, não tem onde cadastrar um curso que ainda não tenha nenhum aluno matriculado.

**f) Entidades que poderiam ser separadas:**

- ALUNO
- CURSO

---

### Exercício 2 — Anomalias de manipulação de dados

Relação:

```text
FUNCIONARIO_DEPARTAMENTO(
    id_funcionario PK,
    nome_funcionario,
    salario,
    id_departamento,
    nome_departamento,
    gerente_departamento
)
```

Regras: cada funcionário está em um único departamento; um departamento tem vários funcionários; cada departamento tem nome e gerente; o identificador do departamento determina o nome e o gerente.

**a) Exemplo de anomalia de inserção:**

Não é possível cadastrar o departamento "Marketing" sem ter pelo menos um funcionário para colocar nele, porque as colunas do departamento só existem junto com um funcionário.

**b) Exemplo de anomalia de atualização:**

Se o gerente do departamento TI muda de Carlos para Marcos, precisa atualizar todas as linhas dos funcionários do TI. Se esquecer de algum, fica com dois gerentes diferentes para o mesmo departamento.

**c) Exemplo de anomalia de exclusão:**

Se o único funcionário do departamento Financeiro for excluído, some junto com ele o nome e o gerente do departamento Financeiro.

**d) Informações da entidade FUNCIONARIO:**

- id_funcionario
- nome_funcionario
- salario
- id_departamento (ligação com o departamento)

**e) Informações da entidade DEPARTAMENTO:**

- id_departamento
- nome_departamento
- gerente_departamento

**f) Estrutura proposta:**

Duas tabelas:

```text
DEPARTAMENTO(
    id_departamento PK,
    nome_departamento,
    gerente_departamento
)

FUNCIONARIO(
    id_funcionario PK,
    nome_funcionario,
    salario,
    id_departamento FK
)
```

---

## Parte 2 — Dependências funcionais

### Exercício 3 — Identificando dependências funcionais

Relação:

```text
PRODUTO(
    id_produto PK,
    nome_produto,
    preco,
    id_categoria,
    nome_categoria
)
```

**a) Chave primária:**

`id_produto`

**b) Dependências funcionais:**

```text
id_produto → nome_produto
id_produto → preco
id_produto → id_categoria
id_categoria → nome_categoria
```

Ou resumindo:

```text
id_produto → nome_produto, preco, id_categoria
id_categoria → nome_categoria
```

**c) O identificador do produto determina o nome da categoria?**

Não diretamente. O `id_produto` determina o `id_categoria`, e o `id_categoria` determina o `nome_categoria`. É uma dependência indireta (transitiva).

**d) O identificador da categoria determina o nome do produto?**

Não. Uma categoria pode ter vários produtos com nomes diferentes. O `id_categoria` só determina o `nome_categoria`.

**e) Existe dependência transitiva?**

Sim. A dependência transitiva é:

```text
id_produto → id_categoria → nome_categoria
```

**f) A relação apresenta problemas de normalização?**

Sim. O nome da categoria se repete para todos os produtos da mesma categoria, e existe dependência transitiva em relação à chave primária.

---

### Exercício 4 — Chaves candidatas e atributos primos

Relação:

```text
ALUNO(
    id_aluno,
    cpf,
    nome,
    email
)
```

Regras: `id_aluno` identifica exclusivamente; `cpf` também identifica exclusivamente; CPF é obrigatório; nome pode se repetir; e-mail não garante unicidade.

**a) Chaves candidatas:**

- `id_aluno`
- `cpf`

**b) Chave primária escolhida:**

`id_aluno`

**c) Atributos primos:**

São os atributos que fazem parte de alguma chave candidata: `id_aluno` e `cpf`.

**d) Atributos não primos:**

`nome` e `email`

**e) Por que um atributo pode ser primo sem ser a chave primária?**

Porque ele pode identificar unicamente cada registro (é chave candidata), mas não foi escolhido como a chave primária oficial da tabela. No caso, o `cpf` poderia ser a PK, mas escolhemos o `id_aluno`.

**f) Implementação em SQL:**

```sql
CREATE TABLE aluno (
    id_aluno BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    cpf VARCHAR(14) NOT NULL UNIQUE,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100)
);
```

---

## Parte 3 — Primeira Forma Normal (1FN)

### Exercício 5 — Aplicando a Primeira Forma Normal

Tabela original `CLIENTE`:

| id_cliente | nome_cliente | telefones |
|------------|--------------|-----------|
| 1 | Ana | 9999-1111, 9888-2222 |
| 2 | Bruno | 9777-3333 |
| 3 | Carlos | 9666-4444, 9555-5555, 9444-6666 |

**a) Por que não atende à 1FN?**

Porque a coluna `telefones` guarda mais de um valor em uma única célula. Na 1FN, cada célula deve conter apenas um valor atômico.

**b) Atributo problemático:**

`telefones`

**c) Estrutura normalizada:**

Duas tabelas: `cliente` e `telefone`.

**d) Chaves primárias:**

- `cliente`: `id_cliente`
- `telefone`: `id_telefone` (ou a combinação `id_cliente` + `numero`)

**e) Chaves estrangeiras:**

`id_cliente` na tabela `telefone`, referenciando a tabela `cliente`.

**f) Cardinalidade:**

1 para N (um cliente pode ter vários telefones; cada telefone pertence a um único cliente).

**g) Modelo lógico relacional normalizado:**

```text
CLIENTE(
    id_cliente PK,
    nome_cliente
)

TELEFONE(
    id_telefone PK,
    numero VARCHAR(20) NOT NULL,
    id_cliente FK
)
```

### Parte prática — SQL

**h) Criação das tabelas:**

```sql
CREATE TABLE cliente (
    id_cliente BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_cliente VARCHAR(100) NOT NULL
);

CREATE TABLE telefone (
    id_telefone BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    numero VARCHAR(20) NOT NULL,
    id_cliente BIGINT NOT NULL,
    CONSTRAINT fk_telefone_cliente
        FOREIGN KEY (id_cliente)
        REFERENCES cliente(id_cliente)
);
```

**i) Inserir os três clientes:**

```sql
INSERT INTO cliente (nome_cliente) VALUES ('Ana');
INSERT INTO cliente (nome_cliente) VALUES ('Bruno');
INSERT INTO cliente (nome_cliente) VALUES ('Carlos');
```

**j) Inserir todos os telefones:**

```sql
INSERT INTO telefone (numero, id_cliente) VALUES ('9999-1111', 1);
INSERT INTO telefone (numero, id_cliente) VALUES ('9888-2222', 1);

INSERT INTO telefone (numero, id_cliente) VALUES ('9777-3333', 2);

INSERT INTO telefone (numero, id_cliente) VALUES ('9666-4444', 3);
INSERT INTO telefone (numero, id_cliente) VALUES ('9555-5555', 3);
INSERT INTO telefone (numero, id_cliente) VALUES ('9444-6666', 3);
```

**k) Consultar todos os clientes:**

```sql
SELECT * FROM cliente;
```

**l) Consultar telefones do cliente 1:**

```sql
SELECT * FROM telefone WHERE id_cliente = 1;
```

**m) Cadastrar novo telefone para o cliente 2:**

```sql
INSERT INTO telefone (numero, id_cliente) VALUES ('9111-2222', 2);
```

**n) Excluir apenas um telefone do cliente 3:**

```sql
DELETE FROM telefone WHERE id_telefone = (SELECT id_telefone FROM telefone WHERE id_cliente = 3 LIMIT 1);
```

> Como não podemos usar funções de agregação, usamos `LIMIT 1` para pegar um telefone do cliente 3 e excluí-lo.

---

### Exercício 6 — Grupos repetitivos

Estrutura original:

```text
ALUNO(
    id_aluno PK,
    nome_aluno,
    disciplina_1,
    disciplina_2,
    disciplina_3
)
```

**a) Problema da estrutura:**

As colunas `disciplina_1`, `disciplina_2` e `disciplina_3` formam um grupo repetitivo. Isso viola a 1FN porque não é uma estrutura flexível para armazenar várias disciplinas por aluno.

**b) Por que dificulta a manutenção:**

- Se um aluno cursar mais de 3 disciplinas, não tem onde guardar.
- Para consultar todas as disciplinas de um aluno, precisa olhar 3 colunas diferentes.
- Para adicionar ou remover uma disciplina, tem que alterar a estrutura da tabela.

**c) Se um aluno precisar cursar cinco disciplinas:**

Não teria colunas suficientes. Teríamos que alterar a tabela para adicionar `disciplina_4` e `disciplina_5`, o que não é prático.

**d) Modelo lógico proposto:**

```text
ALUNO(
    id_aluno PK,
    nome_aluno
)

DISCIPLINA(
    id_disciplina PK,
    nome_disciplina
)

MATRICULA(
    id_aluno FK,
    id_disciplina FK,
    PK (id_aluno, id_disciplina)
)
```

**e) Cardinalidade:**

N para N (muitos para muitos). Um aluno pode cursar várias disciplinas e uma disciplina pode ter vários alunos.

**f) Implementação em SQL:**

```sql
CREATE TABLE aluno (
    id_aluno BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_aluno VARCHAR(100) NOT NULL
);

CREATE TABLE disciplina (
    id_disciplina BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_disciplina VARCHAR(100) NOT NULL
);

CREATE TABLE matricula (
    id_aluno BIGINT NOT NULL,
    id_disciplina BIGINT NOT NULL,
    PRIMARY KEY (id_aluno, id_disciplina),
    CONSTRAINT fk_matricula_aluno
        FOREIGN KEY (id_aluno)
        REFERENCES aluno(id_aluno),
    CONSTRAINT fk_matricula_disciplina
        FOREIGN KEY (id_disciplina)
        REFERENCES disciplina(id_disciplina)
);
```

**g) Cadastrar três alunos e quatro disciplinas:**

```sql
INSERT INTO aluno (nome_aluno) VALUES ('Ana');
INSERT INTO aluno (nome_aluno) VALUES ('Bruno');
INSERT INTO aluno (nome_aluno) VALUES ('Maria');

INSERT INTO disciplina (nome_disciplina) VALUES ('Banco de Dados');
INSERT INTO disciplina (nome_disciplina) VALUES ('Algoritmos');
INSERT INTO disciplina (nome_disciplina) VALUES ('Engenharia de Software');
INSERT INTO disciplina (nome_disciplina) VALUES ('Redes');
```

**h) Registrar as matrículas dos alunos:**

```sql
-- Ana: Banco de Dados (1), Algoritmos (2), Engenharia de Software (3)
INSERT INTO matricula (id_aluno, id_disciplina) VALUES (1, 1);
INSERT INTO matricula (id_aluno, id_disciplina) VALUES (1, 2);
INSERT INTO matricula (id_aluno, id_disciplina) VALUES (1, 3);

-- Bruno: Banco de Dados (1), Redes (4)
INSERT INTO matricula (id_aluno, id_disciplina) VALUES (2, 1);
INSERT INTO matricula (id_aluno, id_disciplina) VALUES (2, 4);

-- Maria: Algoritmos (2)
INSERT INTO matricula (id_aluno, id_disciplina) VALUES (3, 2);
```

**i) Consulta de todas as matrículas do aluno 1:**

```sql
SELECT * FROM matricula WHERE id_aluno = 1;
```

---

## Parte 4 — Segunda Forma Normal (2FN)

### Exercício 7 — Dependência parcial

Relação:

```text
ITEM_PEDIDO(
    id_pedido,
    id_produto,
    nome_produto,
    quantidade
)
```

PK composta: `(id_pedido, id_produto)`

**a) Chave primária composta:**

`(id_pedido, id_produto)`

**b) Dependência funcional que determina quantidade:**

```text
(id_pedido, id_produto) → quantidade
```

**c) Dependência funcional que determina nome_produto:**

```text
id_produto → nome_produto
```

**d) Por que nome_produto depende apenas de parte da chave primária?**

Porque `nome_produto` depende só do `id_produto`, que é apenas uma parte da chave composta `(id_pedido, id_produto)`. Não precisa dos dois juntos para saber o nome do produto.

**e) A relação está na 1FN?**

Sim. Todos os valores são atômicos e não há grupos repetitivos.

**f) A relação está na 2FN?**

Não. Como `nome_produto` depende apenas de parte da chave composta (`id_produto`), há dependência parcial, o que viola a 2FN.

**g) Normalização até a 2FN:**

Tabela `produto`:

```text
PRODUTO(
    id_produto PK,
    nome_produto
)
```

Tabela `item_pedido`:

```text
ITEM_PEDIDO(
    id_pedido,
    id_produto FK,
    quantidade,
    PK (id_pedido, id_produto)
)
```

Também pode criar uma tabela `pedido` se quisermos representar os pedidos separadamente:

```text
PEDIDO(
    id_pedido PK
)
```

**h) Modelo lógico resultante:**

```text
PEDIDO(
    id_pedido PK
)

PRODUTO(
    id_produto PK,
    nome_produto
)

ITEM_PEDIDO(
    id_pedido FK,
    id_produto FK,
    quantidade,
    PK (id_pedido, id_produto)
)
```

### Parte prática — SQL

**i) Implementar tabelas normalizadas:**

```sql
CREATE TABLE pedido (
    id_pedido BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY
);

CREATE TABLE produto (
    id_produto BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_produto VARCHAR(100) NOT NULL
);

CREATE TABLE item_pedido (
    id_pedido BIGINT NOT NULL,
    id_produto BIGINT NOT NULL,
    quantidade INTEGER NOT NULL,
    PRIMARY KEY (id_pedido, id_produto),
    CONSTRAINT fk_item_pedido
        FOREIGN KEY (id_pedido)
        REFERENCES pedido(id_pedido),
    CONSTRAINT fk_item_produto
        FOREIGN KEY (id_produto)
        REFERENCES produto(id_produto)
);
```

**j) Chave primária composta na tabela de itens:**

Já foi definida acima com `PRIMARY KEY (id_pedido, id_produto)`.

**k) Chaves estrangeiras:**

Já foram definidas acima (`fk_item_pedido` e `fk_item_produto`).

**l) Cadastrar três pedidos:**

```sql
INSERT INTO pedido DEFAULT VALUES;
INSERT INTO pedido DEFAULT VALUES;
INSERT INTO pedido DEFAULT VALUES;
```

**m) Cadastrar dois produtos:**

```sql
INSERT INTO produto (nome_produto) VALUES ('Mouse');
INSERT INTO produto (nome_produto) VALUES ('Teclado');
```

**n) Cadastrar os itens apresentados na tabela original:**

```sql
-- Pedido 1: Mouse (10) quantidade 2, Teclado (20) quantidade 1
INSERT INTO item_pedido (id_pedido, id_produto, quantidade) VALUES (1, 1, 2);
INSERT INTO item_pedido (id_pedido, id_produto, quantidade) VALUES (1, 2, 1);

-- Pedido 2: Mouse (10) quantidade 3
INSERT INTO item_pedido (id_pedido, id_produto, quantidade) VALUES (2, 1, 3);

-- Pedido 3: Mouse (10) quantidade 1
INSERT INTO item_pedido (id_pedido, id_produto, quantidade) VALUES (3, 1, 1);
```

**o) Tentar cadastrar o mesmo produto duas vezes no mesmo pedido:**

```sql
INSERT INTO item_pedido (id_pedido, id_produto, quantidade) VALUES (1, 1, 5);
```

**p) Qual restrição impediu a operação?**

A chave primária composta `(id_pedido, id_produto)`. Como já existe a combinação (1, 1), o banco não permite duplicar essa mesma chave.

---

### Exercício 8 — Analisando a 2FN

Relação:

```text
VENDA(
    id_venda PK,
    data_venda,
    id_cliente,
    nome_cliente
)
```

**a) Chave primária:**

`id_venda`

**b) A chave primária é simples ou composta?**

Simples.

**c) Apresenta dependências parciais de uma chave candidata?**

Não. Dependência parcial só existe quando a chave primária é composta e um atributo não primo depende apenas de parte dela. Como a chave é simples, não há parte da chave para depender parcialmente.

**d) Ela atende à 2FN?**

Sim. Está na 1FN, tem apenas uma chave candidata simples (`id_venda`), então não há como haver dependência parcial.

**e) Pode apresentar problemas mesmo estando na 2FN?**

Sim. Pode ter dependência transitiva.

**f) Dependência funcional responsável pelo problema:**

```text
id_venda → id_cliente
id_cliente → nome_cliente
```

Logo: `id_venda → id_cliente → nome_cliente` (dependência transitiva).

**g) Qual forma normal deve ser analisada em seguida?**

A 3FN (Terceira Forma Normal), para eliminar a dependência transitiva.

---

## Parte 5 — Terceira Forma Normal (3FN)

### Exercício 9 — Dependência transitiva

Relação:

```text
ALUNO_CURSO(
    id_aluno PK,
    nome_aluno,
    id_curso,
    nome_curso,
    coordenador
)
```

**a) Chave primária:**

`id_aluno`

**b) Todas as dependências funcionais relevantes:**

```text
id_aluno → nome_aluno
id_aluno → id_curso
id_curso → nome_curso
id_curso → coordenador
```

**c) Dependências transitivas em relação à chave primária:**

```text
id_aluno → id_curso → nome_curso
id_aluno → id_curso → coordenador
```

**d) Por que a relação está na 2FN?**

Porque a chave primária é simples (`id_aluno`), então não existe dependência parcial de parte da chave.

**e) Por que a relação não está na 3FN?**

Porque há dependência transitiva: o nome do curso e o coordenador dependem do `id_curso`, que não é a chave primária.

**f) Normalizar até a 3FN:**

Tabela `curso`:

```text
CURSO(
    id_curso PK,
    nome_curso,
    coordenador
)
```

Tabela `aluno`:

```text
ALUNO(
    id_aluno PK,
    nome_aluno,
    id_curso FK
)
```

**g) Modelo lógico normalizado:**

```text
CURSO(
    id_curso PK,
    nome_curso,
    coordenador
)

ALUNO(
    id_aluno PK,
    nome_aluno,
    id_curso FK
)
```

**h) Chaves primárias e estrangeiras:**

- PK: `id_curso` em CURSO; `id_aluno` em ALUNO
- FK: `id_curso` em ALUNO, referenciando CURSO

### Parte prática — SQL

**i) Implementar tabelas normalizadas:**

```sql
CREATE TABLE curso (
    id_curso BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_curso VARCHAR(100) NOT NULL,
    coordenador VARCHAR(100) NOT NULL
);

CREATE TABLE aluno (
    id_aluno BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_aluno VARCHAR(100) NOT NULL,
    id_curso BIGINT NOT NULL,
    CONSTRAINT fk_aluno_curso
        FOREIGN KEY (id_curso)
        REFERENCES curso(id_curso)
);
```

**j) Cadastrar cursos:**

```sql
INSERT INTO curso (nome_curso, coordenador) VALUES ('Sistemas de Informação', 'Carlos');
INSERT INTO curso (nome_curso, coordenador) VALUES ('Ciência da Computação', 'Fernanda');
INSERT INTO curso (nome_curso, coordenador) VALUES ('Engenharia de Software', 'Roberto');
```

**k) Cadastrar cinco alunos:**

```sql
INSERT INTO aluno (nome_aluno, id_curso) VALUES ('Ana', 1);
INSERT INTO aluno (nome_aluno, id_curso) VALUES ('Bruno', 1);
INSERT INTO aluno (nome_aluno, id_curso) VALUES ('Maria', 2);
INSERT INTO aluno (nome_aluno, id_curso) VALUES ('Pedro', 1);
INSERT INTO aluno (nome_aluno, id_curso) VALUES ('Julia', 2);
```

**l) Alterar coordenador do curso Sistemas de Informação para Marcos:**

```sql
UPDATE curso SET coordenador = 'Marcos' WHERE id_curso = 1;
```

**m) Consultar o curso alterado:**

```sql
SELECT * FROM curso WHERE id_curso = 1;
```

**n) Por que não foi necessário atualizar os registros dos alunos?**

Porque o nome do curso e o coordenador estão armazenados apenas uma vez, na tabela `curso`. Os alunos só guardam o `id_curso`. Alterar uma linha na tabela `curso` atualiza a informação para todos os alunos automaticamente.

---

### Exercício 10 — Normalização de funcionários e departamentos

Dados:

| id_funcionario | nome_funcionario | salario | id_departamento | nome_departamento | gerente_departamento |
|----------------|------------------|---------|-----------------|-------------------|----------------------|
| 1 | Ana | 3500 | 10 | TI | Carlos |
| 2 | Bruno | 4200 | 10 | TI | Carlos |
| 3 | Maria | 3100 | 20 | Financeiro | Fernanda |
| 4 | Pedro | 3900 | 10 | TI | Carlos |
| 5 | Julia | 3700 | 20 | Financeiro | Fernanda |

**a) Dependências funcionais:**

```text
id_funcionario → nome_funcionario, salario, id_departamento
id_departamento → nome_departamento, gerente_departamento
```

**b) Atende à 1FN?**

Sim. Todos os valores são atômicos.

**c) Atende à 2FN?**

Sim. A chave primária é simples (`id_funcionario`), então não há dependência parcial.

**d) Atende à 3FN?**

Não. Há dependência transitiva:

```text
id_funcionario → id_departamento → nome_departamento, gerente_departamento
```

**e) Possíveis anomalias:**

- Inserção: não consigo cadastrar um departamento sem ter um funcionário.
- Atualização: se o gerente do TI muda, preciso atualizar em várias linhas de funcionários.
- Exclusão: se excluo o último funcionário de um departamento, perco os dados do departamento.

**f) Normalizar até a 3FN:**

Tabela `departamento`:

```text
DEPARTAMENTO(
    id_departamento PK,
    nome_departamento,
    gerente_departamento
)
```

Tabela `funcionario`:

```text
FUNCIONARIO(
    id_funcionario PK,
    nome_funcionario,
    salario,
    id_departamento FK
)
```

**g) Modelo lógico resultante:**

```text
DEPARTAMENTO(
    id_departamento PK,
    nome_departamento,
    gerente_departamento
)

FUNCIONARIO(
    id_funcionario PK,
    nome_funcionario,
    salario,
    id_departamento FK
)
```

### Parte prática — SQL

**h) Implementar tabelas normalizadas:**

```sql
CREATE TABLE departamento (
    id_departamento BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_departamento VARCHAR(100) NOT NULL,
    gerente_departamento VARCHAR(100) NOT NULL
);

CREATE TABLE funcionario (
    id_funcionario BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_funcionario VARCHAR(100) NOT NULL,
    salario NUMERIC(10,2) NOT NULL,
    id_departamento BIGINT NOT NULL,
    CONSTRAINT fk_funcionario_departamento
        FOREIGN KEY (id_departamento)
        REFERENCES departamento(id_departamento)
);
```

**i) Cadastrar departamentos:**

```sql
INSERT INTO departamento (nome_departamento, gerente_departamento) VALUES ('TI', 'Carlos');
INSERT INTO departamento (nome_departamento, gerente_departamento) VALUES ('Financeiro', 'Fernanda');
```

**j) Cadastrar funcionários:**

```sql
INSERT INTO funcionario (nome_funcionario, salario, id_departamento) VALUES ('Ana', 3500, 1);
INSERT INTO funcionario (nome_funcionario, salario, id_departamento) VALUES ('Bruno', 4200, 1);
INSERT INTO funcionario (nome_funcionario, salario, id_departamento) VALUES ('Maria', 3100, 2);
INSERT INTO funcionario (nome_funcionario, salario, id_departamento) VALUES ('Pedro', 3900, 1);
INSERT INTO funcionario (nome_funcionario, salario, id_departamento) VALUES ('Julia', 3700, 2);
```

**k) Aumentar em 10% o salário dos funcionários do departamento de TI:**

```sql
UPDATE funcionario SET salario = salario * 1.10 WHERE id_departamento = 1;
```

**l) Alterar o gerente do departamento Financeiro:**

```sql
UPDATE departamento SET gerente_departamento = 'Roberto' WHERE id_departamento = 2;
```

**m) Consultar os dados dos departamentos:**

```sql
SELECT * FROM departamento;
```

**n) Consultar funcionários do departamento de TI:**

```sql
SELECT * FROM funcionario WHERE id_departamento = 1;
```

**o) Tentar cadastrar um funcionário associado a um departamento inexistente:**

```sql
INSERT INTO funcionario (nome_funcionario, salario, id_departamento) VALUES ('João', 3000, 99);
```

**p) Explique o resultado obtido:**

O banco rejeita a inserção porque a chave estrangeira `id_departamento` exige que o valor exista na tabela `departamento`. Como não existe departamento com id 99, ocorre erro de integridade referencial.

---

## Parte 6 — Normalização e SQL

### Exercício 11 — Comparando uma estrutura não normalizada com uma normalizada

**Versão A — Estrutura não normalizada:**

```sql
CREATE TABLE produto_categoria (
    id_produto BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_produto VARCHAR(100) NOT NULL,
    preco NUMERIC(10,2) NOT NULL,
    id_categoria BIGINT NOT NULL,
    nome_categoria VARCHAR(100) NOT NULL
);
```

**b) Cadastrar cinco produtos em duas categorias:**

```sql
INSERT INTO produto_categoria (nome_produto, preco, id_categoria, nome_categoria)
VALUES ('Notebook', 3500.00, 10, 'Informática');

INSERT INTO produto_categoria (nome_produto, preco, id_categoria, nome_categoria)
VALUES ('Mouse', 80.00, 10, 'Informática');

INSERT INTO produto_categoria (nome_produto, preco, id_categoria, nome_categoria)
VALUES ('Teclado', 150.00, 10, 'Informática');

INSERT INTO produto_categoria (nome_produto, preco, id_categoria, nome_categoria)
VALUES ('Monitor', 1200.00, 20, 'Eletrônicos');

INSERT INTO produto_categoria (nome_produto, preco, id_categoria, nome_categoria)
VALUES ('Impressora', 900.00, 20, 'Eletrônicos');
```

**c) Consulta de todos os registros:**

```sql
SELECT * FROM produto_categoria;
```

**d) Alterar o nome de uma categoria em apenas um produto:**

```sql
UPDATE produto_categoria SET nome_categoria = 'Tecnologia' WHERE id_produto = 1;
```

**e) Consulta novamente:**

```sql
SELECT * FROM produto_categoria;
```

**f) Inconsistência gerada:**

Agora o produto 1 (Notebook) tem categoria "Tecnologia", enquanto os produtos 2 e 3 (Mouse e Teclado) ainda têm categoria "Informática", mesmo pertencendo à mesma categoria id 10. O mesmo identificador de categoria tem nomes diferentes.

**g) Implementar versão B:**

```sql
CREATE TABLE categoria (
    id_categoria BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_categoria VARCHAR(100) NOT NULL
);

CREATE TABLE produto (
    id_produto BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_produto VARCHAR(100) NOT NULL,
    preco NUMERIC(10,2) NOT NULL,
    id_categoria BIGINT NOT NULL,
    CONSTRAINT fk_produto_categoria
        FOREIGN KEY (id_categoria)
        REFERENCES categoria(id_categoria)
);
```

**h) Cadastrar categorias:**

```sql
INSERT INTO categoria (nome_categoria) VALUES ('Informática');
INSERT INTO categoria (nome_categoria) VALUES ('Eletrônicos');
```

**i) Cadastrar os mesmos produtos:**

```sql
INSERT INTO produto (nome_produto, preco, id_categoria) VALUES ('Notebook', 3500.00, 1);
INSERT INTO produto (nome_produto, preco, id_categoria) VALUES ('Mouse', 80.00, 1);
INSERT INTO produto (nome_produto, preco, id_categoria) VALUES ('Teclado', 150.00, 1);
INSERT INTO produto (nome_produto, preco, id_categoria) VALUES ('Monitor', 1200.00, 2);
INSERT INTO produto (nome_produto, preco, id_categoria) VALUES ('Impressora', 900.00, 2);
```

**j) Alterar o nome de uma categoria:**

```sql
UPDATE categoria SET nome_categoria = 'Tecnologia' WHERE id_categoria = 1;
```

**k) Consultar registros da tabela categoria:**

```sql
SELECT * FROM categoria;
```

**l) Por que a versão B reduz o risco de inconsistências?**

Porque o nome da categoria está armazenado em apenas um lugar (na tabela `categoria`). Alterar uma linha atualiza o nome para todos os produtos relacionados, sem risco de ficar com nomes diferentes para a mesma categoria.

**m) Comparar a quantidade de registros alterados:**

- Versão A: precisa alterar todos os produtos daquela categoria (no exemplo, 3 produtos).
- Versão B: precisa alterar apenas 1 registro (a categoria).

**Questão para discussão:**

> A normalização elimina todos os problemas de integridade de dados ou ainda precisamos utilizar restrições SQL?

**Resposta:**

Não elimina todos os problemas. A normalização organiza os dados e evita redundâncias, mas ainda precisamos de restrições SQL como `NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY` e `PRIMARY KEY` para garantir que dados inválidos não entrem no banco. Por exemplo, a normalização sozinha não impede que alguém coloque preço negativo ou cadastre um produto com categoria inexistente. As restrições são necessárias para completar a integridade.

---

### Exercício 12 — Migrando dados de uma tabela não normalizada

Tabela original `produto_original` já criada e populada conforme o enunciado.

**a) Problemas de normalização:**

O nome da categoria se repete para produtos da mesma categoria. Isso gera redundância e risco de inconsistência.

**b) Dependências funcionais:**

```text
id_produto → nome_produto, preco, id_categoria
id_categoria → nome_categoria
```

**c) Modelo lógico normalizado até a 3FN:**

```text
CATEGORIA(
    id_categoria PK,
    nome_categoria
)

PRODUTO(
    id_produto PK,
    nome_produto,
    preco,
    id_categoria FK
)
```

**d) Criar tabelas:**

```sql
CREATE TABLE categoria_normalizada (
    id_categoria BIGINT PRIMARY KEY,
    nome_categoria VARCHAR(100) NOT NULL
);

CREATE TABLE produto_normalizado (
    id_produto BIGINT PRIMARY KEY,
    nome_produto VARCHAR(100) NOT NULL,
    preco NUMERIC(10,2) NOT NULL,
    id_categoria BIGINT NOT NULL,
    CONSTRAINT fk_produto_cat
        FOREIGN KEY (id_categoria)
        REFERENCES categoria_normalizada(id_categoria)
);
```

**e) Transferir categorias (usando DISTINCT para evitar repetições):**

```sql
INSERT INTO categoria_normalizada (id_categoria, nome_categoria)
SELECT DISTINCT id_categoria, nome_categoria
FROM produto_original;
```

**f) Transferir produtos:**

```sql
INSERT INTO produto_normalizado (id_produto, nome_produto, preco, id_categoria)
SELECT id_produto, nome_produto, preco, id_categoria
FROM produto_original;
```

**g) Consultar dados das novas tabelas:**

```sql
SELECT * FROM categoria_normalizada;
SELECT * FROM produto_normalizado;
```

**h) Verificar se todos os produtos foram preservados:**

```sql
SELECT * FROM produto_normalizado;
```

Deve retornar os 6 produtos originais.

**i) Por que os identificadores originais das categorias devem ser preservados?**

Porque eles são usados como chave estrangeira na tabela `produto_normalizado`. Se mudássemos os ids, os produtos não saberiam mais a que categoria pertencem.

**j) Como a chave estrangeira contribui para a integridade após a normalização?**

Ela garante que um produto só pode referenciar uma categoria que realmente existe na tabela `categoria_normalizada`, evitando registros órfãos.

---

## Parte 7 — Desafio final de normalização

### Exercício 13 — Sistema de gerenciamento de pedidos

Relação original:

```text
PEDIDO_PRODUTO(
    id_pedido,
    id_produto,
    data_pedido,
    id_cliente,
    nome_cliente,
    telefone_cliente,
    nome_produto,
    preco_atual_produto,
    quantidade,
    preco_unitario_venda
)
```

PK composta: `(id_pedido, id_produto)`

#### Etapa 1 — Análise

**a) Todos os atributos:**

`id_pedido`, `id_produto`, `data_pedido`, `id_cliente`, `nome_cliente`, `telefone_cliente`, `nome_produto`, `preco_atual_produto`, `quantidade`, `preco_unitario_venda`.

**b) Chave primária:**

`(id_pedido, id_produto)`

**c) Dependências funcionais:**

```text
(id_pedido, id_produto) → data_pedido, quantidade, preco_unitario_venda
id_pedido → id_cliente, data_pedido
id_cliente → nome_cliente, telefone_cliente
id_produto → nome_produto, preco_atual_produto
```

**d) Dependências parciais:**

- `id_pedido → data_pedido` (parte da chave composta determina data do pedido)
- `id_produto → nome_produto, preco_atual_produto` (parte da chave determina dados do produto)
- `id_pedido → id_cliente` (parte da chave determina o cliente do pedido)

**e) Dependências transitivas:**

```text
id_pedido → id_cliente → nome_cliente, telefone_cliente
id_produto → nome_produto → preco_atual_produto (se nome determinar preço, mas aqui é id_produto → preco_atual_produto direto)
```

A principal transitiva é: `id_pedido → id_cliente → nome_cliente, telefone_cliente`.

**f) Por que não está na 2FN?**

Porque existem dependências parciais da chave composta. Por exemplo, `nome_produto` depende apenas de `id_produto`, que é parte da chave, não da chave inteira.

**g) Problemas que impedem a 3FN:**

Existe dependência transitiva: `id_pedido → id_cliente → nome_cliente`. O nome do cliente depende do cliente, não diretamente do pedido.

#### Etapa 2 — Normalização

**h) Normalizar até a 1FN:**

Já está na 1FN (valores atômicos, sem grupos repetitivos).

**i) Normalizar até a 2FN:**

Separar as tabelas:

```text
PEDIDO(
    id_pedido PK,
    data_pedido,
    id_cliente FK
)

PRODUTO(
    id_produto PK,
    nome_produto,
    preco_atual_produto
)

ITEM_PEDIDO(
    id_pedido FK,
    id_produto FK,
    quantidade,
    preco_unitario_venda,
    PK (id_pedido, id_produto)
)
```

Mas ainda tem `id_cliente` em `PEDIDO` com dependência transitiva para `nome_cliente` e `telefone_cliente`.

**j) Normalizar até a 3FN:**

Tabela `cliente`:

```text
CLIENTE(
    id_cliente PK,
    nome_cliente,
    telefone_cliente
)
```

Tabela `pedido`:

```text
PEDIDO(
    id_pedido PK,
    data_pedido,
    id_cliente FK
)
```

Tabela `produto`:

```text
PRODUTO(
    id_produto PK,
    nome_produto,
    preco_atual_produto
)
```

Tabela `item_pedido`:

```text
ITEM_PEDIDO(
    id_pedido FK,
    id_produto FK,
    quantidade,
    preco_unitario_venda,
    PK (id_pedido, id_produto)
)
```

**k) Modelo lógico final:**

```text
CLIENTE(
    id_cliente PK,
    nome_cliente,
    telefone_cliente
)

PEDIDO(
    id_pedido PK,
    data_pedido,
    id_cliente FK
)

PRODUTO(
    id_produto PK,
    nome_produto,
    preco_atual_produto
)

ITEM_PEDIDO(
    id_pedido FK,
    id_produto FK,
    quantidade,
    preco_unitario_venda,
    PK (id_pedido, id_produto)
)
```

**l) Chaves primárias:**

- `id_cliente` em CLIENTE
- `id_pedido` em PEDIDO
- `id_produto` em PRODUTO
- `(id_pedido, id_produto)` em ITEM_PEDIDO

**m) Chaves estrangeiras:**

- `id_cliente` em PEDIDO → CLIENTE
- `id_pedido` em ITEM_PEDIDO → PEDIDO
- `id_produto` em ITEM_PEDIDO → PRODUTO

**n) Cardinalidades:**

- CLIENTE → PEDIDO: 1 para N (um cliente faz vários pedidos)
- PEDIDO → ITEM_PEDIDO: 1 para N (um pedido tem vários itens)
- PRODUTO → ITEM_PEDIDO: 1 para N (um produto aparece em vários itens de pedido)

#### Etapa 3 — Implementação SQL

**o) Criar tabelas normalizadas:**

```sql
CREATE TABLE cliente (
    id_cliente BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_cliente VARCHAR(100) NOT NULL,
    telefone_cliente VARCHAR(20)
);

CREATE TABLE produto (
    id_produto BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nome_produto VARCHAR(100) NOT NULL,
    preco_atual_produto NUMERIC(10,2) NOT NULL CHECK (preco_atual_produto >= 0)
);

CREATE TABLE pedido (
    id_pedido BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    data_pedido DATE NOT NULL,
    id_cliente BIGINT NOT NULL,
    CONSTRAINT fk_pedido_cliente
        FOREIGN KEY (id_cliente)
        REFERENCES cliente(id_cliente)
);

CREATE TABLE item_pedido (
    id_pedido BIGINT NOT NULL,
    id_produto BIGINT NOT NULL,
    quantidade INTEGER NOT NULL CHECK (quantidade > 0),
    preco_unitario_venda NUMERIC(10,2) NOT NULL CHECK (preco_unitario_venda >= 0),
    PRIMARY KEY (id_pedido, id_produto),
    CONSTRAINT fk_item_pedido
        FOREIGN KEY (id_pedido)
        REFERENCES pedido(id_pedido),
    CONSTRAINT fk_item_produto
        FOREIGN KEY (id_produto)
        REFERENCES produto(id_produto)
);
```

**p) Restrições de integridade:**

Já foram definidas acima: `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, `CHECK`.

**q) Cadastrar três clientes:**

```sql
INSERT INTO cliente (nome_cliente, telefone_cliente) VALUES ('Ana', '1111-1111');
INSERT INTO cliente (nome_cliente, telefone_cliente) VALUES ('Bruno', '2222-2222');
INSERT INTO cliente (nome_cliente, telefone_cliente) VALUES ('Maria', '3333-3333');
```

**r) Cadastrar cinco produtos:**

```sql
INSERT INTO produto (nome_produto, preco_atual_produto) VALUES ('Mouse', 80.00);
INSERT INTO produto (nome_produto, preco_atual_produto) VALUES ('Teclado', 150.00);
INSERT INTO produto (nome_produto, preco_atual_produto) VALUES ('Monitor', 1200.00);
INSERT INTO produto (nome_produto, preco_atual_produto) VALUES ('Notebook', 3500.00);
INSERT INTO produto (nome_produto, preco_atual_produto) VALUES ('Impressora', 900.00);
```

**s) Cadastrar quatro pedidos:**

```sql
INSERT INTO pedido (data_pedido, id_cliente) VALUES ('2026-08-01', 1);
INSERT INTO pedido (data_pedido, id_cliente) VALUES ('2026-08-02', 2);
INSERT INTO pedido (data_pedido, id_cliente) VALUES ('2026-08-03', 1);
INSERT INTO pedido (data_pedido, id_cliente) VALUES ('2026-08-04', 3);
```

**t) Cadastrar pelo menos oito itens de pedido:**

```sql
INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario_venda) VALUES (1, 1, 2, 80.00);
INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario_venda) VALUES (1, 2, 1, 150.00);
INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario_venda) VALUES (2, 3, 1, 1200.00);
INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario_venda) VALUES (3, 1, 3, 80.00);
INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario_venda) VALUES (3, 4, 1, 3500.00);
INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario_venda) VALUES (4, 5, 1, 900.00);
INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario_venda) VALUES (4, 2, 2, 150.00);
INSERT INTO item_pedido (id_pedido, id_produto, quantidade, preco_unitario_venda) VALUES (2, 1, 1, 80.00);
```

**u) Consultar todos os pedidos:**

```sql
SELECT * FROM pedido;
```

**v) Consultar itens pertencentes a um pedido específico (exemplo: pedido 1):**

```sql
SELECT * FROM item_pedido WHERE id_pedido = 1;
```

**w) Atualizar o telefone de um cliente:**

```sql
UPDATE cliente SET telefone_cliente = '9999-9999' WHERE id_cliente = 1;
```

**x) Atualizar o preço cadastral de um produto:**

```sql
UPDATE produto SET preco_atual_produto = 100.00 WHERE id_produto = 1;
```

**y) Verificar se a alteração do preço cadastral preservou o preço unitário registrado nos pedidos anteriores:**

```sql
SELECT * FROM item_pedido WHERE id_produto = 1;
```

O preço unitário nos itens antigos continua sendo o valor original (80.00), não foi alterado com o preço cadastral.

**z) Por que essa separação é importante para manter o histórico das vendas?**

Porque o preço unitário de venda fica registrado na tabela `item_pedido`, separado do preço cadastral do produto. Se o preço do produto mudar no futuro, os pedidos antigos continuam com o valor que foi cobrado na época, preservando o histórico financeiro correto.

---

> **Entrega:** enviar este arquivo para o seu repositório no GitHub e colar o link no Google Classroom.

