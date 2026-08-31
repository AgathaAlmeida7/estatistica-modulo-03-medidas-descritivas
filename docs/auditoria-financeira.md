# 🟥 Auditoria Financeira dos Pedidos

## 1. Objetivo

A Auditoria Financeira teve como objetivo verificar a consistência financeira dos pedidos, comparando as informações presentes nas diferentes tabelas relacionadas à composição e ao pagamento dos pedidos.

Foram utilizadas principalmente as tabelas:

* `orders`
* `order_items`
* `order_payments`

A auditoria buscou identificar possíveis inconsistências entre:

* pedidos;
* itens associados aos pedidos;
* valores dos itens;
* fretes;
* pagamentos realizados;
* quantidade de pagamentos;
* valores financeiros registrados.

O objetivo desta etapa não foi realizar uma análise financeira de desempenho, mas verificar a qualidade e a coerência dos dados financeiros antes das análises posteriores.

---

# 2. Tabelas utilizadas

## `orders`

Utilizada como tabela principal dos pedidos.

O campo de relacionamento utilizado foi:

`order_id`

## `order_items`

Utilizada para verificar:

* existência de itens;
* quantidade de itens;
* valor dos produtos;
* valor do frete.

Principais campos utilizados:

* `order_id`
* `order_item_id`
* `price`
* `freight_value`

## `order_payments`

Utilizada para verificar:

* existência de pagamentos;
* quantidade de pagamentos;
* tipo de pagamento;
* número de parcelas;
* valor pago.

Principais campos utilizados:

* `order_id`
* `payment_sequential`
* `payment_type`
* `payment_installments`
* `payment_value`

---

# 3. Integridade dos relacionamentos

A primeira etapa da auditoria verificou se os pedidos possuíam registros correspondentes nas tabelas de itens e pagamentos.

O relacionamento principal utilizado foi:

```text
orders
   │
   ├── order_id ──→ order_items
   │
   └── order_id ──→ order_payments
```

Essa verificação permitiu identificar diferentes situações:

1. pedido com itens e pagamentos;
2. pedido com itens, mas sem pagamentos;
3. pedido com pagamentos, mas sem itens;
4. pedido sem itens e sem pagamentos.

Essas situações foram tratadas como cenários de investigação e não como erros automaticamente.

---

# 4. Pedidos sem itens

Foram investigados pedidos que não possuíam registros correspondentes na tabela `order_items`.

A ausência de itens representa uma inconsistência potencial porque, conceitualmente, um pedido comercial normalmente deveria possuir pelo menos um item associado.

Entretanto, a ausência pode estar relacionada a:

* problema de integração;
* falha de carga;
* pedido cancelado;
* inconsistência na origem;
* diferença no modelo de armazenamento.

Portanto, a ausência de itens foi tratada como **sinal de qualidade dos dados**, e não como prova automática de erro comercial.

---

# 5. Pedidos sem pagamentos

Também foram investigados pedidos sem registros correspondentes na tabela `order_payments`.

Essa situação pode representar diferentes cenários, dependendo do contexto do pedido:

* pagamento ainda não registrado;
* falha de integração;
* pedido cancelado;
* pedido não concluído;
* inconsistência de carga;
* outro comportamento válido do sistema.

Por isso, a ausência de pagamento não foi interpretada isoladamente como fraude, erro ou perda financeira.

---

# 6. Comparação entre itens e pagamentos

Uma das principais verificações da auditoria foi comparar os valores registrados em `order_items` com os valores registrados em `order_payments`.

Para cada pedido, foram calculados:

### Valor dos produtos

Soma de:

`price`

### Valor do frete

Soma de:

`freight_value`

### Valor financeiro dos itens

Foi considerado o valor:

```text
valor dos produtos + valor do frete
```

### Valor total dos pagamentos

Soma de:

`payment_value`

Essa comparação permitiu identificar possíveis diferenças entre o valor associado à composição do pedido e o valor registrado como pagamento.

---

# 7. Interpretação das diferenças financeiras

Uma diferença entre o valor dos itens e o valor pago não significa automaticamente que exista um erro.

Ela pode estar relacionada a:

* parcelamento;
* múltiplos registros de pagamento;
* diferenças de cobrança;
* descontos;
* ajustes;
* outras regras comerciais;
* inconsistências nos dados.

Por esse motivo, a auditoria buscou primeiro identificar os casos discrepantes e compreender seu comportamento antes de classificá-los como problemas definitivos.

---

# 8. Múltiplos pagamentos

Foi verificada a possibilidade de um mesmo `order_id` possuir mais de um registro em `order_payments`.

Essa situação é válida dentro do modelo dos dados.

Um pedido pode possuir múltiplos registros de pagamento devido a diferentes formas de pagamento ou registros associados à mesma transação.

Por isso, a análise financeira não deve considerar apenas a existência de um pagamento por pedido.

O valor financeiro deve ser consolidado por:

`order_id`

por meio da soma dos valores registrados em `payment_value`.

---

# 9. Valores financeiros

Foram avaliadas as principais variáveis financeiras relacionadas aos pedidos.

O objetivo foi identificar:

* valores ausentes;
* valores nulos;
* valores negativos;
* valores inconsistentes;
* discrepâncias entre itens e pagamentos;
* possíveis valores extremos.

Essas verificações são importantes porque valores financeiros incorretos podem comprometer diretamente médias, totais, distribuições e indicadores financeiros calculados posteriormente.

---

# 10. Valores extremos

Valores financeiros muito elevados ou muito baixos foram tratados como candidatos à investigação.

Um valor extremo não foi automaticamente considerado erro.

A existência de um valor elevado pode representar:

* pedido realmente de alto valor;
* compra com vários itens;
* pedido com frete elevado;
* comportamento comercial legítimo.

Portanto, valores extremos devem ser investigados antes de qualquer exclusão.

---

# 11. Principais conclusões da auditoria

A auditoria financeira permitiu estabelecer uma visão estruturada da relação entre:

```text
Pedido
  ↓
Itens
  ↓
Valor dos produtos
  ↓
Frete
  ↓
Pagamentos
```

As verificações realizadas permitiram identificar situações que precisam ser consideradas nas análises posteriores, especialmente relacionadas à integridade dos relacionamentos e à comparação entre valores de itens e pagamentos.

Os registros não foram alterados ou excluídos simplesmente por apresentarem comportamento diferente do esperado.

A auditoria teve como finalidade **identificar e documentar situações que exigem tratamento ou interpretação específica**.

---

# 12. Decisões metodológicas

A partir da auditoria financeira, ficou estabelecido que as análises posteriores deverão:

1. considerar o relacionamento entre `orders`, `order_items` e `order_payments`;
2. consolidar registros de itens por `order_id` quando necessário;
3. consolidar pagamentos por `order_id` quando necessário;
4. evitar interpretar diferenças financeiras isoladamente;
5. investigar valores extremos antes de removê-los;
6. documentar qualquer exclusão ou tratamento realizado;
7. diferenciar ausência de informação de erro confirmado.

---

# 13. Limitações

A auditoria financeira possui algumas limitações.

A existência de uma diferença entre tabelas não permite determinar automaticamente a causa da inconsistência.

Para estabelecer a causa definitiva de determinadas discrepâncias, seria necessário conhecer regras de negócio e informações adicionais do sistema de origem.

Portanto, os achados desta auditoria devem ser interpretados como:

* evidências de qualidade dos dados;
* sinais de possíveis inconsistências;
* pontos de atenção para as análises posteriores.

Não devem ser tratados automaticamente como falhas operacionais ou financeiras confirmadas.

---

# 14. Impacto nas análises posteriores

Os resultados desta auditoria deverão ser considerados durante a preparação da base analítica.

Especialmente nas análises relacionadas a:

* receita;
* valor médio dos pedidos;
* ticket médio;
* frete;
* formas de pagamento;
* parcelamento;
* comportamento financeiro dos pedidos.

As métricas financeiras deverão utilizar agregações adequadas para evitar duplicidade causada pelo relacionamento entre pedidos, itens e pagamentos.

---

# 15. Conclusão

A Auditoria Financeira cumpriu seu objetivo de verificar a consistência básica das informações financeiras e dos relacionamentos entre pedidos, itens e pagamentos.

A investigação permitiu identificar os principais pontos que devem ser considerados antes da realização das análises financeiras e estatísticas.

O principal resultado desta etapa não é simplesmente determinar quais registros são "certos" ou "errados", mas estabelecer **quais características dos dados precisam ser consideradas para que as análises posteriores sejam confiáveis**.

Dessa forma, a base financeira encontra-se documentada e preparada para as próximas etapas do projeto, respeitando as limitações e regras identificadas durante a auditoria.

**Status: Auditoria Financeira — CONCLUÍDA.**
