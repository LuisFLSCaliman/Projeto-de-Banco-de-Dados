# Sistema de Banco de Dados para Pet Shop

Projeto da disciplina de **Banco de Dados** que modela o banco de dados de um Pet Shop: clientes, pets, funcionários, serviços, agendamentos, produtos, vendas e pagamentos. O repositório reúne o minimundo, o Modelo Entidade-Relacionamento (MER), o modelo relacional, os scripts de criação das tabelas e consultas SQL de exemplo.

## Autores

| Nome | Matrícula |
|------|-----------|
| Victor Alves Pimenta de Oliveira | 322060043 |
| Luis Filipe Lamonica Salvador Caliman | 121060022 |

## Minimundo

O Pet Shop atende clientes que possuem pets registrados no sistema.

- **Cliente:** possui nome, endereço (atributo composto por rua, número, bairro e CEP), múltiplos telefones e múltiplos e-mails.
- **Pet:** pertence a exatamente um cliente. Cada cliente tem um ou mais pets. É identificado por nome, espécie, raça e data de nascimento.
- **Serviço:** banho, tosa, consulta veterinária, vacinação etc., com descrição e preço. É sempre realizado por um **funcionário** (veterinário ou atendente), que pode ter especialização.
- **Agendamento:** o cliente agenda um serviço para um pet, com data e hora e status (pendente, concluído ou cancelado). Um pet pode ter vários agendamentos, mas cada agendamento tem um único serviço.
- **Produto:** rações, brinquedos, acessórios etc., com nome, categoria, preço e quantidade em estoque.
- **Venda:** associada ao cliente, com data, valor total e método de pagamento (crédito, débito, PIX ou dinheiro). Cada venda tem um ou mais itens, registrados em `ItensVenda`, com quantidade e subtotal.

## Modelo Entidade-Relacionamento

O MER tem as entidades **Cliente**, **Telefone**, **Email**, **Pet**, **Funcionário**, **Serviço**, **Agendamento**, **Produto**, **Venda**, **Pagamento** e **ItensVenda**.

Principais relacionamentos:

| Relacionamento | Cardinalidade |
|----------------|---------------|
| Cliente **possui** Pet | 1:N |
| Cliente **possui** Telefone / Email | 1:N |
| Pet **tem** Agendamento | 1:N |
| Agendamento **refere-se** a Serviço | N:1 |
| Funcionário **presta** Serviço | 1:N |
| Cliente **realiza** Venda | 1:N |
| Venda **contém** ItensVenda | 1:N |
| ItensVenda **refere-se** a Produto | N:1 |
| Venda **possui** Pagamento | 1:N |

> **Observação:** no diagrama, o relacionamento entre Cliente e Venda aparece como "Cliente compra Venda". Deve ser lido como "Cliente **realiza** uma compra".

## Modelo relacional

```
Cliente      = {id_cliente (PK), nome, rua, bairro, numero_res}
Telefone     = {id_cliente (FK), telefone (PK)}
Email        = {id_cliente (FK), email (PK)}
Pet          = {id_pet (PK), nome, especie, raca, data_nascimento, id_cliente (FK)}
Funcionario  = {id_funcionario (PK), nome, cargo, especialidade}
Servico      = {id_servico (PK), descricao, preco}
Agendamento  = {id_agendamento (PK), data_hora, status, id_pet (FK), id_servico (FK), id_funcionario (FK)}
Produto      = {id_produto (PK), nome, categoria, preco, estoque}
Venda        = {id_venda (PK), data_venda, valor_total, id_cliente (FK)}
Pagamento    = {id_pagamento (PK), forma_pagamento, data_pagamento, valor_pago, id_venda (FK)}
Itens_Venda  = {id_venda (FK), id_produto (FK), quantidade, valor_unitario, subtotal}
```

As tabelas `Telefone` e `Email` têm chave primária composta, para representar os atributos multivalorados do cliente. `ItensVenda` usa chave primária composta por `(id_venda, id_produto)`.

## Criação das tabelas

O script SQL (MySQL) cria as 11 tabelas na seguinte ordem, respeitando as dependências entre chaves estrangeiras:

1. `Cliente`
2. `Telefone` (`ON DELETE CASCADE`)
3. `Email` (`ON DELETE CASCADE`)
4. `Pet`
5. `Funcionario`
6. `Servico`
7. `Agendamento`
8. `Produto`
9. `Venda`
10. `Pagamento`
11. `ItensVenda`

Exemplo:

```sql
CREATE TABLE Cliente (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    rua VARCHAR(100),
    bairro VARCHAR(60),
    numero_res VARCHAR(10),
    cidade VARCHAR(60)
);
```

## Dados de exemplo

O banco foi populado com dados de teste: 2 clientes (Ana Martins e Carlos Silva), 2 pets (Thor, um labrador, e Mingau, um gato siamês), 2 funcionários, 2 serviços (Banho e Tosa Completo e Consulta Veterinária), 2 agendamentos, 2 produtos (Ração Premium 10kg e Coleira Antipulgas), 2 vendas, 2 pagamentos e 3 itens de venda. Cada tabela foi verificada com `SELECT * FROM <tabela>;`.

## Consultas SQL

### 1. Consulta com duas tabelas, sem `GROUP BY`

Lista os pets com o nome do cliente dono do animal.

```sql
SELECT
    Pet.nome AS nome_pet,
    Cliente.nome AS nome_cliente,
    Pet.especie,
    Pet.raca
FROM Pet
JOIN Cliente ON Pet.id_cliente = Cliente.id_cliente;
```

### 2. Consulta com `GROUP BY` e duas tabelas

Quantidade de agendamentos por serviço.

```sql
SELECT
    Servico.descricao AS servico,
    COUNT(Agendamento.id_agendamento) AS total_agendamentos
FROM Servico
JOIN Agendamento ON Servico.id_servico = Agendamento.id_servico
GROUP BY Servico.descricao;
```

### 3. Consulta com três tabelas ou mais

Lista as vendas com o nome do cliente, o valor total e a quantidade total de itens.

```sql
SELECT
    Venda.id_venda,
    Cliente.nome AS cliente,
    Venda.data_venda,
    SUM(ItensVenda.quantidade) AS total_itens,
    Venda.valor_total
FROM Venda
JOIN Cliente ON Venda.id_cliente = Cliente.id_cliente
JOIN ItensVenda ON Venda.id_venda = ItensVenda.id_venda
GROUP BY
    Venda.id_venda,
    Cliente.nome,
    Venda.data_venda,
    Venda.valor_total
ORDER BY Venda.id_venda;
```
