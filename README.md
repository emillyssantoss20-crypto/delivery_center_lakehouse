# MVP: Pipeline de Dados na Nuvem — Delivery Center

**Nome:** Emilly da Silva Santos  
**Matrícula:** 4052026000270  
**Data:**  
**Dataset:** Brazilian Delivery Center (Kaggle)

Trabalho da disciplina de Engenharia de Dados (PUC-Rio). Pipeline de dados end‑to‑end construído no Databricks, seguindo a Arquitetura Medalhão (Bronze → Silver → Gold).

---

## 1. Contexto de Negócio e Perguntas

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

## 2. Carga dos Dados
 
### Estratégia de coleta
O dataset é estático, então a carga foi feita pelo caminho mais simples, download do arquivo e upload para a nuvem.
 
### Passo a passo
 
**1. Download do Kaggle**
O arquivo `.zip` do dataset foi baixado diretamente da página do Kaggle (link e licença documentados no Tópico 1) e extraído localmente, resultando em 7 arquivos CSV — um por entidade: `orders`, `deliveries`, `stores`, `hubs`, `drivers`, `channels` e `payments`.
 
**2. Criação da estrutura de destino no Unity Catalog**
Antes de subir qualquer arquivo, foi criada a organização que recebe os dados, seguindo a Arquitetura Medalhão:
- Catálogo `delivery_center` (tipo Standard, storage padrão do metastore)
- Três schemas dentro dele: `bronze` (dados brutos, sem transformação), `silver` (dados limpos — Etapa 4.4) e `gold` (dados modelados para análise — Etapa 4.3/4.4)
  
**3. Upload de cada CSV como tabela Bronze**
Usando o recurso **"Create table from file"** do Catalog Explorer (compute: *Serverless Starter Warehouse*), cada um dos 7 CSVs foi enviado individualmente para dentro do schema `delivery_center.bronze`, com o nome da tabela correspondendo ao nome da entidade (ex: `delivery_center.bronze.orders`). O Databricks já inferiu automaticamente o tipo de cada coluna a partir do conteúdo do arquivo (inteiros, strings, decimais, datas/horas).
 
> **Nota de processo:** na primeira tentativa, os 7 arquivos foram selecionados juntos no upload, e a interface tentou combinar colunas de arquivos diferentes em uma única tabela (misturando, por exemplo, colunas de motoristas com colunas de lojas). O erro foi identificado no preview de colunas antes da criação da tabela, e corrigido subindo os arquivos **um de cada vez**. Essa checagem do preview antes de confirmar a criação foi o que permitiu pegar o problema a tempo.
 
**4. Conferência final**
Ao final, o schema `bronze` ficou com as 7 tabelas esperadas, cada uma com o schema correto (colunas e tipos batendo com o arquivo de origem correspondente):
`delivery_center.bronze.orders`, `.deliveries`, `.stores`, `.hubs`, `.drivers`, `.channels`, `.payments`.
 
### Evidências (screenshots)
- Criação do catálogo `delivery_center` e dos schemas `bronze`/`silver`/`gold`
- Tela de preview de colunas durante o "Create table from file" (incluindo o erro identificado e a correção)
- Lista final das 7 tabelas dentro do schema `bronze`
*(Screenshots já capturados ao longo do processo — anexar as imagens correspondentes nesta seção do documento final.)*
 
### Scripts
Não se aplica nesta etapa — a carga foi feita via interface gráfica do Databricks (upload direto), sem necessidade de notebook ou script de ingestão, já que o volume e a natureza estática do dataset não justificavam automação nesta fase. A automação (leitura programática e transformação) começa na Etapa 4.4 (Pipeline de Dados/ETL), via notebooks PySpark/SQL versionados no repositório GitHub.
 
---
