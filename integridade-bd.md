# Atividade — Integridade de Dados
#### Arthur Spirandelli - 144136

---

## Parte A — O que é integridade de dados?

Integridade de dados é, basicamente, garantir que as informações guardadas no banco de dados estejam corretas, façam sentido e não se contradigam.

Imagina que o banco de dados é como o caderno de anotações de uma loja. Se no caderno tiver dois clientes com o mesmo CPF, um produto com preço negativo ou uma venda feita para um cliente que não existe, esse caderno para de ser confiável. Qualquer decisão tomada com base nele vai estar errada.

Por isso, integridade de dados significa criar regras para impedir que informações erradas, duplicadas ou sem sentido entrem no banco. Não é só confiar na pessoa que digita; o próprio banco de dados tem que rejeitar o que não fizer sentido.

---

## Parte B — Integridade de entidade

**O que ela procura garantir?**
Que cada registro da tabela seja único e possa ser identificado sem confusão. Não pode haver duas entidades iguais sem jeito de diferenciá-las.

**Qual é a relação com a chave primária (PK)?**
A chave primária é o que obriga a integridade de entidade funcionar, porque ela identifica unicamente cada linha da tabela.

**Uma chave primária pode ter valores repetidos?**
Não. Tem que ser único.

**Uma chave primária pode ser nula?**
Não. Se for nula, não dá para identificar o registro.

**O que acontece se essas regras não forem respeitadas?**
Ficam registros duplicados, não dá para saber qual é qual, e qualquer atualização ou exclusão fica perigosa porque pode atingir o registro errado.

**Exemplo próprio:**

Tabela `FUNCIONARIO(matricula, nome, cargo)`, onde `matricula` é a chave primária.

Se a PK permitisse repetir, poderíamos ter dois funcionários com a mesma matrícula e não saberíamos a quem se refere um pagamento ou uma alteração de cargo. Se permitisse nulo, teríamos funcionário sem identificação, impossibilitando localizá-lo.

---

## Parte C — Integridade referencial

**O que ela procura garantir?**
Que a chave estrangeira (FK) sempre aponte para um registro que realmente existe na outra tabela. Não pode ter referência para coisa que não existe.

**Qual é a relação com a FK?**
A FK é o elo entre as tabelas. A integridade referencial é a regra de que esse elo não pode estar quebrado.

**Uma FK pode referenciar um registro inexistente?**
Não, enquanto a regra estiver ativa o banco rejeita.

**O que é um registro órfão?**
É um registro na tabela "filha" cuja FK não tem correspondência na tabela "pai". Exemplo: um pedido para o cliente 999, mas esse cliente não existe. Está órfão porque não tem dono.

**O que acontece ao excluir um registro que está sendo referenciado?**
Ficam registros órfãos. Para evitar isso, o banco pode impedir a exclusão, excluir também os registros dependentes, ou deixar a FK vazia/com valor padrão.

**Exemplo próprio com duas tabelas:**

Tabelas `CLIENTE(idCliente, nome)` e `PEDIDO(idPedido, idCliente FK)`.

Se existir um pedido com `idCliente = 50`, mas não existir cliente com esse código, esse pedido é órfão: aponta para um cliente que não existe.

Se excluirmos o cliente 1, que tem pedidos, esses pedidos ficam sem dono, a menos que criemos uma regra para lidar com isso.

---

## Parte D — Integridade de domínio

**O que significa o domínio de um atributo?**
É o conjunto de valores válidos que aquele atributo pode receber, incluindo tipo, formato e regras de negócio.

**Como determinar quais valores são válidos?**
Pela regra de negócio, pelo tipo de dado e por validações específicas (ex: preço não negativo, estoque não negativo, e-mail no formato certo).

**O tipo de dado ajuda a controlar o domínio?**
Sim, porque restringe se é número, texto ou data. Mas não é suficiente sozinho. Um campo numérico ainda pode receber um preço negativo, por exemplo.

**Existem restrições além do tipo de dado?**
Sim: `CHECK`, `NOT NULL`, `UNIQUE`, `DEFAULT`, entre outras.

**Tabela PRODUTO(idProduto, nome, preco, estoque):**

| Atributo | O que deveria permitir | O que NÃO deveria permitir | Justificativa |
|----------|------------------------|----------------------------|---------------|
| preço | Número igual ou maior que zero (ex: 0,00; 120,00) | Valores negativos (ex: -50,00) | Preço negativo não faz sentido numa venda. |
| estoque | Número inteiro igual ou maior que zero (ex: 0; 5; 15) | Valores negativos (ex: -3) | Não existe estoque negativo fisicamente. |

---

## Parte II — Problemas no banco da loja

Abaixo estão nove problemas encontrados (o mínimo era oito).

| Problema encontrado | Por que é um problema? | Tipo de integridade | Regra que deveria existir |
|---------------------|------------------------|---------------------|---------------------------|
| Cliente 2 aparece duas vezes (Bruno e Carlos) | Dois registros com o mesmo identificador; não dá para diferenciá-los. | Entidade | `idCliente` não pode se repetir (é PK). |
| Cliente 4 sem nome ("—") | Cliente incompleto, sem identificação significativa. | Entidade / Domínio | `nome` não pode ser nulo ou vazio. |
| E-mail `ana@email.com` repetido (clientes 1 e 5) | Dois clientes com o mesmo e-mail geram confusão de contato. | Domínio | `email` deve ser único (`UNIQUE`). |
| Produto Mouse com preço -50,00 | Preço negativo não tem sentido comercial. | Domínio | `preco` deve ser maior ou igual a zero. |
| Produto Monitor com estoque -3 | Estoque negativo é impossível fisicamente. | Domínio | `estoque` deve ser maior ou igual a zero. |
| Produto 13 sem nome ("—") | Produto sem nome não pode ser identificado nem vendido. | Domínio | `nome` não pode ser nulo ou vazio. |
| Pedido 101 com valor -100,00 | Valor negativo de pedido não faz sentido. | Domínio | `valor` deve ser maior ou igual a zero. |
| Pedido 102 referencia cliente 99, que não existe | Cria registro órfão: pedido aponta para cliente inexistente. | Referencial | `idCliente` em `PEDIDO` deve referenciar cliente existente (`FOREIGN KEY`). |
| Pedido 103 sem data ("—") | Pedido sem data fica incompleto e dificulta relatórios. | Domínio | `data` não pode ser nula (`NOT NULL`). |

---

## Parte III — Regras para cada tabela

### Tabela CLIENTE

| Código | Regra | Categoria |
|:------:|-------|-----------|
| [E] R01 | Todo cliente deve ter um identificador único (`idCliente` é PK). | Entidade |
| [E] R02 | O identificador do cliente não pode ser nulo. | Entidade |
| [D] R03 | Todo cliente deve ter nome obrigatório (não nulo nem vazio). | Domínio |
| [D] R04 | O e-mail de cada cliente deve ser único (não pode se repetir). | Domínio |

### Tabela PRODUTO

| Código | Regra | Categoria |
|:------:|-------|-----------|
| [E] R05 | Todo produto deve ter um identificador único (`idProduto` é PK). | Entidade |
| [E] R06 | O identificador do produto não pode ser nulo. | Entidade |
| [D] R07 | O preço do produto não pode ser negativo (`preco >= 0`). | Domínio |
| [D] R08 | O estoque do produto não pode ser negativo (`estoque >= 0`). | Domínio |
| [D] R09 | O nome do produto é obrigatório (não nulo nem vazio). | Domínio |

### Tabela PEDIDO

| Código | Regra | Categoria |
|:------:|-------|-----------|
| [E] R10 | Todo pedido deve ter um identificador único (`idPedido` é PK). | Entidade |
| [E] R11 | O identificador do pedido não pode ser nulo. | Entidade |
| [R] R12 | O `idCliente` do pedido deve referenciar um cliente existente na tabela `CLIENTE` (FK). | Referencial |
| [D] R13 | O valor do pedido não pode ser negativo (`valor >= 0`). | Domínio |
| [D] R14 | A data do pedido é obrigatória (não pode ser nula). | Domínio |

---

## Parte IV — Termos pesquisados

| Termo | O que procura garantir? | Exemplo de situação útil |
|-------|-------------------------|--------------------------|
| `PRIMARY KEY` | Garante que cada registro da tabela seja único e não nulo, servindo como identificador da entidade. | `idCliente` na tabela `CLIENTE`, para ninguém ter o mesmo identificador. |
| `FOREIGN KEY` | Garante que um valor em uma tabela corresponda a uma chave primária existente em outra tabela. | `idCliente` na tabela `PEDIDO`, para um pedido só pertencer a um cliente cadastrado. |
| `NOT NULL` | Garante que o atributo sempre tenha um valor (não pode ficar em branco). | `nome` do cliente, para impedir cadastro sem nome. |
| `UNIQUE` | Garante que não existam valores repetidos na coluna. | `email` do cliente, para impedir dois clientes com o mesmo e-mail. |
| `CHECK` | Garante que o valor do atributo atenda a uma condição lógica ou intervalo permitido. | `preco >= 0`, para impedir preço negativo de produto. |
| `DEFAULT` | Define um valor padrão a ser usado quando nada for informado na inserção. | `estoque = 0`, para um produto novo começar com estoque zero se nada for dito. |

---

## Faça as conexões

| Necessidade | Mecanismo que você utilizaria |
|-------------|-------------------------------|
| Identificar cada cliente de maneira única | `PRIMARY KEY` em `idCliente` |
| Impedir um cliente sem nome | `NOT NULL` em `nome` |
| Impedir dois clientes com o mesmo e-mail | `UNIQUE` em `email` |
| Impedir preço negativo | `CHECK (preco >= 0)` |
| Garantir que um pedido pertença a um cliente existente | `FOREIGN KEY` em `idCliente` referenciando `CLIENTE(idCliente)` |
| Fazer um produto iniciar com estoque zero quando nenhum valor for informado | `DEFAULT 0` em `estoque` |

### Pergunta final

**Existe relação entre os problemas de integridade estudados e os mecanismos encontrados? Explique.**

Sim. Cada mecanismo existe justamente para prevenir um tipo específico de problema de integridade.

- Problemas de **integridade de entidade** (registro duplicado, identificador nulo) são prevenidos com `PRIMARY KEY` e `NOT NULL`.
- Problemas de **integridade referencial** (registro órfão, referência a cliente inexistente) são prevenidos com `FOREIGN KEY`.
- Problemas de **integridade de domínio** (preço negativo, estoque negativo, campo obrigatório vazio, e-mail duplicado) são prevenidos com `CHECK`, `NOT NULL`, `UNIQUE` e `DEFAULT`.

Os mecanismos da Parte IV são a forma prática de aplicar as regras de integridade pesquisadas na Parte I. Sem eles, o banco permitiria dados errados; com eles, o próprio banco rejeita qualquer informação que não faça sentido, mantendo tudo confiável.


