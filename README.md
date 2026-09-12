# MVP: Pipeline de Dados na Nuvem — Delivery Center

**Nome:** Emilly da Silva Santos  
**Matrícula:** 4052026000270  
**Data:**  
**Dataset:** Brazilian Delivery Center (Kaggle)

Trabalho da disciplina de Engenharia de Dados (PUC-Rio). Pipeline de dados end‑to‑end construído no Databricks, seguindo a Arquitetura Medalhão (Bronze → Silver → Gold).

---

## Contexto de Negócio e Perguntas

### Contexto

A **Delivery Center** é uma plataforma brasileira que integra lojistas e marketplaces para operacionalizar entregas de comida e produtos (*food & goods*) através de uma rede de hubs regionais, lojas parceiras e motoristas terceirizados.

Como em qualquer operação logística, falhas podem nascer em pontos muito diferentes do processo e cada um exige uma ação corretiva diferente.

Utilizando um recorte dessa operação (~370 mil pedidos), essa massa de dados brutos deverá ser convertida em insights, capazes de orientar decisões operacionais e comerciais de forma concreta.

### Problema de negócio

Quero entender quais fatores mais influenciam a performance e o valor das entregas na operação da Delivery Center, para identificar **onde** e **por quê** o processo falha, e permitir ação corretiva direcionada.

### Perguntas que deverão ser respondidas

**Visão geral**

1. Quais hubs apresentam a maior distância média de entrega, e isso se relaciona com o volume de pedidos atendidos por hub?  
2. O status da entrega (concluída, cancelada etc.) varia significativamente entre hubs ou estados?  
3. O segmento da loja influencia o valor médio dos pedidos?  
4. Como o volume e o valor total dos pedidos evoluem mês a mês no período coberto pelos dados?  
5. Quais estados concentram a maior receita total e qual é o ticket médio por estado?

**Diagnóstico de falhas**

7. Quais hubs têm a maior taxa de cancelamento de entregas em relação ao volume de pedidos recebidos?  
8. Existe relação entre distância de entrega e taxa de cancelamento?  
9. Algum tipo/modal de motorista concentra proporcionalmente mais entregas com status de falha?  
10. Pedidos com diferente de "pago" estão concentrados em algum canal, método de pagamento ou hub específico?

**Decomposição do tempo**

10. Qual é o tempo médio total do ciclo do pedido, da criação à finalização, por hub — e quais hubs estão significativamente acima da média geral da operação?  
11. Das etapas do processo (produção, expedição, trânsito, pausas), qual mais contribui para o tempo total do ciclo?  
12. Existe relação entre (tempo pausado) e o status final da entrega?  
13. Clientes que sofreram atraso/cancelamento voltam a comprar?

### Fonte dos dados

Dataset **Brazilian Delivery Center**, disponível no Kaggle:  
https://www.kaggle.com/datasets/nosbielcs/brazilian-delivery-center

**Licença de uso:** `Data files © Original Authors` (conforme indicado na aba "License" da página do Kaggle). Os direitos permanecem com os autores originais dos dados. O dataset é utilizado aqui exclusivamente para fins educacionais/acadêmicos.
