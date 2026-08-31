# 🟥 Auditoria Temporal dos Pedidos

## 1. Objetivo

A Auditoria Temporal teve como objetivo avaliar a consistência cronológica dos pedidos e investigar o comportamento temporal do processo de entrega.

Foram analisadas as principais etapas da jornada do pedido:

1. Compra;
2. Aprovação;
3. Encaminhamento à transportadora;
4. Entrega ao cliente;
5. Prazo estimado de entrega.

O objetivo não foi realizar uma análise estatística definitiva do desempenho logístico, mas verificar a qualidade temporal dos registros, identificar inconsistências, construir uma base temporal válida e levantar os principais sinais que deverão ser considerados nas etapas analíticas posteriores.

---

# 2. Base analisada

A análise teve como ponto de partida os pedidos classificados como `delivered`.

Total de pedidos classificados como entregues:

**96.478 pedidos**

Durante a preparação da análise temporal, foram identificados registros que não possuíam informações suficientes ou consistentes para participar integralmente dos cálculos temporais.

Após esse processo, a base `temporal_analysis` ficou com:

**96.455 pedidos**

Portanto:

* `delivered_orders`: 96.478 registros
* `temporal_analysis`: 96.455 registros
* registros fora da análise temporal: 23

A diferença foi investigada individualmente.

---

# 3. Integridade das datas

Foram avaliadas as principais colunas temporais relacionadas ao processo de entrega:

* `order_purchase_timestamp`
* `order_approved_at`
* `order_delivered_carrier_date`
* `order_delivered_customer_date`
* `order_estimated_delivery_date`

A auditoria identificou pedidos classificados como `delivered` sem data de entrega ao cliente.

Foram encontrados:

**8 pedidos `delivered` sem `order_delivered_customer_date`.**

Esses registros não permitem determinar corretamente a duração completa do processo até a entrega ao cliente.

Também foram identificados registros adicionais que não puderam compor a base temporal final de 96.455 registros.

---

# 4. Pedidos entregues sem data de entrega ao cliente

Foram investigados os 8 pedidos classificados como `delivered` que não apresentavam a data de entrega ao cliente.

A investigação foi complementada pelo cruzamento com:

* `order_items`
* `order_payments`

Todos os 8 pedidos investigados apresentaram registros correspondentes de itens e pagamentos.

Isso demonstra que os pedidos possuem evidências de existência comercial, embora apresentem ausência da data final de entrega no registro temporal.

### Conclusão

Esses casos devem ser tratados como **registros temporalmente incompletos**, e não como pedidos inexistentes ou necessariamente não entregues.

Durante a auditoria, esses registros não foram artificialmente preenchidos.

---

# 5. Construção da base temporal

Foi criada a `temporal_analysis`, contendo somente os registros considerados adequados para os cálculos temporais realizados nesta etapa.

Foram calculados indicadores referentes às seguintes etapas:

* tempo entre compra e aprovação;
* tempo entre aprovação e encaminhamento à transportadora;
* tempo entre transportadora e entrega ao cliente;
* tempo total até a entrega;
* atraso em relação à data estimada.

A construção dessa base permitiu separar problemas de qualidade dos dados de análises relacionadas ao desempenho das entregas.

---

# 6. Distribuição dos tempos

A análise descritiva inicial dos tempos apresentou os seguintes resultados:

| Indicador |  Aprovação | Até transportadora | Transportadora → cliente | Tempo total |
| --------- | ---------: | -----------------: | -----------------------: | ----------: |
| Média     |  0,43 dias |          2,80 dias |                9,33 dias |  12,56 dias |
| Mediana   |  0,01 dias |          1,82 dias |                7,10 dias |  10,22 dias |
| 75%       |  0,60 dias |          3,57 dias |               12,03 dias |  15,72 dias |
| Máximo    | 30,89 dias |        125,76 dias |              205,19 dias | 209,63 dias |

Os resultados demonstram elevada variabilidade principalmente nas etapas logísticas.

Também foram observados valores extremos, indicando que a distribuição dos tempos não é perfeitamente homogênea.

---

# 7. Inconsistências cronológicas

Foi realizada uma verificação específica para identificar durações negativas.

## 7.1. Tempo negativo até a transportadora

Foram identificados:

**1.350 registros**

com:

`carrier_days < 0`

Representação:

**1,40% da base temporal analisada.**

Esses registros apresentam uma inconsistência cronológica entre a aprovação e a data registrada para encaminhamento à transportadora.

Exemplo conceitual:

```text
Aprovação
        ↓
deveria ocorrer antes
        ↓
Transportadora
```

Quando a data registrada para a transportadora aparece antes da aprovação, o cálculo da duração produz um valor negativo.

### Interpretação

O resultado caracteriza uma **anomalia de consistência temporal**.

Não é possível concluir apenas com essa análise se os registros representam:

* erro de cadastro;
* problema de integração;
* atraso/erro de atualização;
* diferença semântica entre os eventos;
* ou outra característica da origem dos dados.

Portanto, os registros foram identificados e documentados, sem alteração artificial dos valores.

---

# 8. Tempo negativo entre transportadora e cliente

Foram identificados:

**23 registros**

com:

`delivery_days < 0`

Representando aproximadamente:

**0,02% da base temporal analisada.**

Esses registros apresentam uma inconsistência ainda mais direta na sequência temporal:

```text
Transportadora
        ↓
deveria ocorrer antes
        ↓
Entrega ao cliente
```

Porém, a data de entrega ao cliente aparece anterior à data registrada para a transportadora.

### Conclusão

Esses 23 registros devem ser considerados inconsistências temporais e não devem ser interpretados como tempos reais de entrega.

---

# 9. Pedidos atrasados

Na base temporal analisada foram identificados:

* **88.630 pedidos não atrasados**
* **7.825 pedidos atrasados**

Total:

**96.455 pedidos**

A proporção de pedidos atrasados foi de aproximadamente:

**8,11%**

Esse resultado indica que os atrasos representam uma parcela relevante dos pedidos analisados e justificam investigação operacional posterior.

---

# 10. Comparação entre pedidos atrasados e não atrasados

Foi realizada uma comparação das durações médias das etapas.

| Etapa                    | Não atrasados |  Atrasados | Diferença |
| ------------------------ | ------------: | ---------: | --------: |
| Aprovação                |     0,42 dias |  0,51 dias |     +0,09 |
| Até transportadora       |     2,58 dias |  5,32 dias |     +2,74 |
| Transportadora → cliente |     7,89 dias | 25,68 dias |    +17,80 |

O maior diferencial foi observado na etapa:

**Transportadora → cliente**

com diferença média de aproximadamente:

**17,80 dias.**

---

# 11. Principal insight operacional

A etapa que apresenta a maior diferença entre pedidos atrasados e não atrasados é o intervalo entre o encaminhamento à transportadora e a entrega ao cliente.

Enquanto os pedidos não atrasados apresentam média de aproximadamente:

**7,89 dias**

nessa etapa, os pedidos atrasados apresentam:

**25,68 dias**

Isso representa uma diferença de aproximadamente:

**17,80 dias.**

### Interpretação

Esse resultado sugere que a etapa final da logística possui forte relação com a ocorrência dos atrasos observados.

Entretanto, a auditoria não permite afirmar que a transportadora seja, isoladamente, a causa dos atrasos.

O resultado deve ser tratado como um **sinal para investigação posterior**.

---

# 12. Atrasos extremos

Foram identificados:

**360 pedidos**

com atraso superior a:

**30 dias.**

Foram observados casos extremos superiores a 100 dias, incluindo um caso com aproximadamente:

**188,98 dias de atraso.**

Esses resultados indicam a existência de uma cauda de valores extremos na distribuição dos atrasos.

Os valores extremos não foram removidos durante a auditoria, pois ainda não foi determinado se representam erros de dados ou eventos reais.

---

# 13. Comportamento dos atrasos ao longo dos anos

A proporção média de pedidos atrasados apresentou crescimento ao longo dos anos:

| Ano  | Proporção de atrasos |
| ---- | -------------------: |
| 2016 |                1,50% |
| 2017 |                6,63% |
| 2018 |                9,37% |

Esse comportamento sugere uma possível alteração do padrão de atrasos ao longo do período analisado.

O resultado deverá ser investigado posteriormente para verificar se existem fatores temporais, sazonais, logísticos ou relacionados à composição da base que expliquem esse comportamento.

---

# 14. Comportamento mensal

Também foram observadas diferenças na proporção de atrasos entre os meses.

Alguns dos maiores valores encontrados foram:

| Mês       | Proporção de atrasos |
| --------- | -------------------: |
| Março     |               17,15% |
| Novembro  |               14,31% |
| Fevereiro |               13,43% |

Esse resultado sugere possível comportamento sazonal ou concentração de determinados períodos com maior incidência de atrasos.

Entretanto, a existência de associação temporal não implica, por si só, causalidade.

---

# 15. Principais achados da auditoria

A Auditoria Temporal produziu os seguintes achados:

### 🟢 Achado 1 — Registros temporais incompletos

Foram identificados pedidos `delivered` sem data de entrega ao cliente.

Esses registros foram investigados e documentados como casos temporalmente incompletos.

### 🟡 Achado 2 — Inconsistências cronológicas

Foram identificados:

**1.350 registros — 1,40%**

com duração negativa entre aprovação e transportadora.

Também foram identificados:

**23 registros — 0,02%**

com duração negativa entre transportadora e cliente.

### 🔴 Achado 3 — Incidência de atrasos

Foram identificados:

**7.825 pedidos atrasados**, aproximadamente **8,11%** da base temporal analisada.

### 🚨 Achado 4 — Etapa com maior diferença

A maior diferença entre pedidos atrasados e não atrasados ocorreu entre:

**transportadora → cliente**

com diferença média de aproximadamente:

**17,80 dias.**

### 🚨 Achado 5 — Atrasos extremos

Foram identificados:

**360 pedidos com atraso superior a 30 dias**, incluindo casos superiores a 100 dias.

### 📅 Achado 6 — Variação temporal

A proporção de atrasos aumentou entre 2016, 2017 e 2018 e apresentou variações relevantes entre os meses.

---

# 16. Limitações

Os resultados desta auditoria devem ser interpretados considerando as seguintes limitações:

1. A existência de uma inconsistência temporal não determina automaticamente sua causa.
2. Valores extremos não foram automaticamente considerados erros.
3. A auditoria não teve como objetivo estabelecer causalidade.
4. Diferenças entre grupos representam associações observadas nos dados.
5. Registros inconsistentes deverão receber tratamento metodológico específico antes de determinadas análises estatísticas.
6. A análise temporal foi realizada sobre a base considerada válida para os cálculos desta etapa.

---

# 17. Decisões para as próximas etapas

Os achados desta auditoria deverão ser considerados na preparação da análise estatística.

Especialmente:

* registros com datas inconsistentes;
* tempos negativos;
* registros incompletos;
* valores extremos;
* atrasos;
* distribuição assimétrica dos tempos;
* possível comportamento sazonal;
* diferença expressiva entre pedidos atrasados e não atrasados.

Os dados não devem ser simplesmente excluídos sem justificativa.

Qualquer tratamento posterior deverá ser documentado e justificado de acordo com o objetivo da análise.

---

# 18. Conclusão

A Auditoria Temporal cumpriu seu objetivo de avaliar a consistência dos registros temporais e identificar os principais padrões e anomalias relacionados ao processo de entrega.

A base analisada apresentou poucos registros incompletos em termos proporcionais, mas revelou inconsistências cronológicas que precisam ser consideradas nas análises posteriores.

Do ponto de vista operacional, o principal achado foi a forte diferença no tempo entre o encaminhamento à transportadora e a entrega ao cliente quando comparados pedidos atrasados e não atrasados.

Também foram identificados atrasos extremos e variações relevantes na incidência de atrasos ao longo dos anos e meses.

Portanto, a auditoria fornece uma base de conhecimento suficiente para avançar para as etapas analíticas do projeto, mantendo registradas as limitações e anomalias que deverão ser consideradas no tratamento e na interpretação dos dados.

**Status: Auditoria Temporal — CONCLUÍDA.**
