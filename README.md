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
O arquivo `.zip` do dataset foi baixado diretamente da página do Kaggle e extraído localmente, resultando em 7 arquivos CSV um por entidade: `orders`, `deliveries`, `stores`, `hubs`, `drivers`, `channels` e `payments`.
 
**2. Criação da estrutura de destino no Unity Catalog**
Antes de subir qualquer arquivo, foi criada a organização que recebe os dados, seguindo a Arquitetura Medalhão:
- Catálogo `delivery_center`
- Três schemas dentro dele: `bronze` (dados brutos, sem transformação), `silver` (dados limpos) e `gold` (dados modelados para análise)
  
**3. Upload de cada CSV como tabela Bronze**
Usando o recurso **"Create table from file"** do Catalog Explorer, cada um dos 7 CSVs foi enviado individualmente para dentro do schema `delivery_center.bronze`, com o nome da tabela correspondendo ao nome da entidade. O Databricks já inferiu automaticamente o tipo de cada coluna a partir do conteúdo do arquivo.
 
**4. Conferência final**
Ao final, o schema `bronze` ficou com as 7 tabelas esperadas, cada uma com o schema correto.
 
### Evidências (screenshots)

- Criação do catálogo `delivery_center` e dos schemas `bronze`/`silver`/`gold`

<img width="1908" height="983" alt="image" src="https://github.com/user-attachments/assets/748daf79-2938-41a7-80fe-058d371bc3e4" />

- Inclusão dos documentos na pasta `bronze`

<img width="1906" height="986" alt="image" src="https://github.com/user-attachments/assets/6c56d8b8-6d88-40ed-a13a-933348230d42" />

### Scripts
Não se aplica nesta etapa, a carga foi feita via interface gráfica do Databricks (upload direto), sem necessidade de notebook ou script de ingestão, já que o volume e a natureza estática do dataset não justificavam automação nesta fase.
 
## 3. Modelagem e catálogo
 
### Modelo escolhido: Esquema Estrela
Entre Estrela, Snowflake e Flat, o **Esquema Estrela** foi escolhido porque com o este modelo, qualquer uma delas precisa de no máximo 1 join por dimensão. O Snowflake normalizaria demais para o volume do dataset, e o Flat duplicaria dados de loja/hub em cada linha sem necessidade.

<img width="1192" height="907" alt="image" src="https://github.com/user-attachments/assets/fd3e8a1d-c06e-42c5-9744-6a4d080ce4d1" />

### Catálogo de Dados
 
Uma tabela por base (fonte) do schema `delivery_center.bronze`, com o nome de cada coluna, sua descrição de negócio, o formato em que ela chega na camada Bronze (tipo inferido automaticamente pelo Databricks a partir do CSV — confirmar com `DESCRIBE TABLE delivery_center.bronze.<nome>` e ajustar se divergir) e o formato final que ela assume no modelo Estrela (camada Gold), já refletindo renomeações e conversões de tipo feitas na transformação.
 
#### `orders`
 
| Coluna | Descrição do Dado | Formato Bronze | Formato Final |
|---|---|---|---|
| `order_id` | Identificador único do pedido | BIGINT | BIGINT — chave primária de `fact_orders` |
| `store_id` | Loja que originou o pedido | BIGINT | BIGINT — FK para `dim_stores` |
| `channel_id` | Canal de origem do pedido | BIGINT | BIGINT — FK para `dim_channels` |
| `order_status` | Status final do pedido | STRING | VARCHAR(50) |
| `order_amount` | Valor total do pedido | DOUBLE | DECIMAL(10,2) |
| `order_delivery_fee` | Taxa de entrega cobrada no pedido | DOUBLE | DECIMAL(10,2) |
| `order_delivery_cost` | Custo operacional da entrega para a Delivery Center | DOUBLE | DECIMAL(10,2) |
| `order_created_day` | Dia de criação do pedido | BIGINT | INT — consumido na geração de `dim_date.date_id`, não persiste como coluna própria em `fact_orders` |
| `order_created_month` | Mês de criação do pedido | BIGINT | INT — idem |
| `order_created_year` | Ano de criação do pedido | BIGINT | INT — idem |
| `order_moment_created` | Timestamp de criação do pedido | STRING *(confirmar — pode já vir como TIMESTAMP)* | TIMESTAMP, convertido com `to_timestamp()` |
| `order_moment_accepted` | Timestamp de aceite do pedido pela loja | STRING *(confirmar)* | TIMESTAMP |
| `order_moment_ready` | Timestamp em que o pedido ficou pronto | STRING *(confirmar)* | TIMESTAMP |
| `order_moment_collected` | Timestamp de coleta pelo motorista | STRING *(confirmar)* | TIMESTAMP |
| `order_moment_in_expedition` | Timestamp de início da expedição | STRING *(confirmar)* | TIMESTAMP |
| `order_moment_delivering` | Timestamp de início do trajeto até o cliente | STRING *(confirmar)* | TIMESTAMP |
| `order_moment_delivered` | Timestamp de entrega ao cliente | STRING *(confirmar)* | TIMESTAMP |
| `order_moment_finished` | Timestamp de finalização do pedido | STRING *(confirmar)* | TIMESTAMP |
| `order_metric_collected_time` | Duração até a coleta pelo motorista | DOUBLE | DECIMAL(10,2) |
| `order_metric_paused_time` | Duração em pausa durante o processo | DOUBLE | DECIMAL(10,2) |
| `order_metric_production_time` | Duração da produção do pedido na loja | DOUBLE | DECIMAL(10,2) |
| `order_metric_walking_time` | Duração de caminhada do motorista | DOUBLE | DECIMAL(10,2) |
| `order_metric_expediton_speed_time` | Duração da velocidade de expedição | DOUBLE | DECIMAL(10,2) |
| `order_metric_transit_time` | Duração em trânsito até o cliente | DOUBLE | DECIMAL(10,2) |
| `order_metric_cycle_time` | Duração total do ciclo do pedido, da criação à finalização | DOUBLE | DECIMAL(10,2) |
 
#### `deliveries`
 
| Coluna | Descrição do Dado | Formato Bronze | Formato Final |
|---|---|---|---|
| `delivery_order_id` | Pedido ao qual a entrega pertence (chave de junção com `orders.order_id`) | BIGINT | BIGINT — usado no join que monta `fact_orders`; não persiste como coluna própria no modelo final |
| `driver_id` | Motorista responsável pela entrega | BIGINT | BIGINT — FK para `dim_drivers`, trazido para dentro de `fact_orders` |
| `delivery_status` | Status da entrega | STRING | VARCHAR(50) |
| `delivery_distance_meters` | Distância percorrida na entrega, em metros | BIGINT | BIGINT |
 
#### `stores`
 
| Coluna | Descrição do Dado | Formato Bronze | Formato Final |
|---|---|---|---|
| `store_id` | Identificador único da loja | BIGINT | BIGINT — chave primária de `dim_stores` |
| `store_name` | Nome da loja | STRING | VARCHAR(255) |
| `store_segment` | Segmento/categoria de atuação da loja | STRING | VARCHAR(100) |
| `store_plan_price` | Valor do plano contratado pela loja na plataforma | DOUBLE | DECIMAL(10,2) |
| `store_latitude` | Latitude da loja | DOUBLE | DOUBLE |
| `store_longitude` | Longitude da loja | DOUBLE | DOUBLE |
| `hub_id` | Hub ao qual a loja está associada | BIGINT | BIGINT — usado no join com `hubs` que achata os atributos do hub dentro de `dim_stores` |
 
#### `hubs`
 
| Coluna | Descrição do Dado | Formato Bronze | Formato Final |
|---|---|---|---|
| `hub_id` | Identificador único do hub | BIGINT | BIGINT — usado apenas no join; incorporado em `dim_stores`, sem tabela própria no modelo final |
| `hub_name` | Nome do hub | STRING | VARCHAR(255) |
| `hub_city` | Cidade do hub | STRING | VARCHAR(100) |
| `hub_state` | UF do hub | STRING | VARCHAR(2) |
| `hub_latitude` | Latitude do hub | DOUBLE | DOUBLE |
| `hub_longitude` | Longitude do hub | DOUBLE | DOUBLE |
 
#### `drivers`
 
| Coluna | Descrição do Dado | Formato Bronze | Formato Final |
|---|---|---|---|
| `driver_id` | Identificador único do motorista | BIGINT | BIGINT — chave primária de `dim_drivers` |
| `driver_modal` | Modal de entrega utilizado pelo motorista (ex.: moto, bike, carro) | STRING | VARCHAR(50) |
| `driver_type` | Tipo de vínculo do motorista com a operação | STRING | VARCHAR(50) |
 
#### `channels`
 
| Coluna | Descrição do Dado | Formato Bronze | Formato Final |
|---|---|---|---|
| `channel_id` | Identificador único do canal do pedido | BIGINT | BIGINT — chave primária de `dim_channels` |
| `channel_name` | Nome do canal de venda | STRING | VARCHAR(100) |
| `channel_type` | Tipo/categoria do canal | STRING | VARCHAR(50) |
 
#### `payments`
 
| Coluna | Descrição do Dado | Formato Bronze | Formato Final |
|---|---|---|---|
| `payment_id` | Identificador único do pagamento | BIGINT | BIGINT — chave primária de `fact_payments` |
| `payment_order_id` | Pedido ao qual o pagamento se refere | BIGINT | BIGINT — renomeado para `order_id`, FK para `fact_orders` |
| `payment_amount` | Valor pago nesta transação | DOUBLE | DECIMAL(10,2) |
| `payment_fee` | Taxa cobrada sobre o pagamento | DOUBLE | DECIMAL(10,2) |
| `payment_method` | Método de pagamento utilizado | STRING | VARCHAR(50) |
| `payment_status` | Status do pagamento | STRING | VARCHAR(50) |
 
*(Nota: os campos marcados "confirmar" nos timestamps de `orders` dependem de como o Databricks inferiu o tipo no upload do CSV — rodar `DESCRIBE TABLE delivery_center.bronze.orders` e ajustar a coluna "Formato Bronze" se o tipo real vier diferente de STRING. Complementar opcional: os mesmos textos de descrição podem ser colados no campo "Comment" de cada coluna, na aba "Columns" de cada tabela no Unity Catalog, deixando a documentação também visível direto no Databricks.)*
 


