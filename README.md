# MVP: Pipeline de Dados na Nuvem — Delivery Center

**Nome:** Emilly da Silva Santos  
**Matrícula:** 4052026000270  
**Data:**  -
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
 
> **Nota de processo:** A validação dos dados identificou 9.531 pedidos com mais de um registro em deliveries, o que gerava duplicidade na fact_orders. Para corrigir o problema, a tabela deliveries foi deduplicada antes do join, mantendo apenas um registro por pedido e priorizando pedidos com status DELIVERED.

## 4. Pipeline de Dados
 
### Organização do ETL

O pipeline foi implementado em um notebook no Databricks, versionado no repositório em `ETL Pipeline.ipynb`. O notebook é organizado em células curtas e intercaladas: para cada tabela, uma célula de markdown explica o que foi feito, seguida imediatamente da célula `%sql` que executa a transformação, célula a célula. Ele segue duas partes sequenciais:

**1. Bronze → Silver** (limpeza e padronização, sem mudar o significado dos dados)
 
| Tabela | Duplicatas/nulos removidos | Conversões de tipo | Demais campos |
|---|---|---|---|
| `orders` | `DISTINCT` + descarta `order_id IS NULL` | `order_amount`, `order_delivery_fee`, `order_delivery_cost` → `DECIMAL(10,2)`; os 8 `order_moment_*` → `TIMESTAMP` (via `try_to_timestamp`, formato `M/d/yyyy h:mm:ss a`); os 7 `order_metric_*` → `DECIMAL(10,2)` | `order_status`, `store_id`, `channel_id`, `order_created_day/month/year` mantidos como vieram |
| `deliveries` | `DISTINCT` + descarta `delivery_order_id IS NULL` | `delivery_distance_meters` → `BIGINT` | `driver_id`, `delivery_status` sem alteração |
| `stores` | `DISTINCT` + descarta `store_id IS NULL` | `store_plan_price` → `DECIMAL(10,2)` | `store_name`, `store_segment`, `store_latitude/longitude`, `hub_id` sem alteração |
| `hubs` | `DISTINCT` + descarta `hub_id IS NULL` | nenhuma | todos os campos mantidos como vieram |
| `drivers` | `DISTINCT` + descarta `driver_id IS NULL` | nenhuma | todos os campos mantidos como vieram |
| `channels` | `DISTINCT` + descarta `channel_id IS NULL` | nenhuma | todos os campos mantidos como vieram |
| `payments` | `DISTINCT` + descarta `payment_id IS NULL` | `payment_amount`, `payment_fee` → `DECIMAL(10,2)` | `payment_order_id`, `payment_method`, `payment_status` sem alteração |
 
**2. Silver → Gold** (remodelagem para o Esquema Estrela definido no Tópico 3)
 
| Tabela Gold | O que foi feito |
|---|---|
| `dim_stores` | `stores` + `hubs` unidas |
| `dim_drivers` | cópia direta de `silver.drivers`, sem alteração |
| `dim_channels` | cópia direta de `silver.channels`, sem alteração |
| `dim_date` | combinações únicas de `order_created_day/month/year` de `orders`, com chave substituta (`date_id`) gerada via `ROW_NUMBER()` |
| `fact_orders` | `orders` + `deliveries` unidas |
| `fact_payments` | cópia de `silver.payments`, renomeando `payment_order_id` → `order_id` |

> **Nota de processo:** Foi identificado um problema na conversão dos campos de data e hora (order_moment_*) devido à incompatibilidade de formato. A solução foi definir explicitamente o padrão da data e utilizar uma conversão mais segura (try_to_timestamp()), evitando falhas na geração da tabela.
 
### Como executar

1. Abrir o notebook `ETL Pipeline` no Databricks (dentro do Git folder do repositório).
2. Rodar todas as células em sequência ("Run All") — cada célula de markdown documenta a tabela que a célula `%sql` logo abaixo cria.
3. Conferir no Catalog Explorer que os schemas `delivery_center.silver` e `delivery_center.gold` passaram a ter as 7 e 6 tabelas, respectivamente.

### Evidências (screenshots)

- Criação de tabelas bronze e gold com o código executado sem erros.

<img width="1908" height="987" alt="image" src="https://github.com/user-attachments/assets/8daa62c9-34f3-404f-aa8d-9ef8aedeedcc" />

## 5. Qualidade de Dados
 
### Dimensões avaliadas e como
Cinco checagens rodadas no notebook `qualidade_dados.ipynb`, cobrindo:
 
| Dimensão | O que foi checado |
|---|---|
| Integridade dos Dados | Contagem de linhas Bronze × Silver em cada uma das 7 tabelas — a diferença mostra quantas linhas foram descartadas por duplicata exata ou por não ter chave primária |
| Conversão de Datas e Horários | Nulos introduzidos pela conversão dos 8 campos `order_moment_*` para `TIMESTAMP` — comparando a quantidade de nulos no Bronze (antes) com a do Silver (depois), não o valor absoluto |
| Consistência das Chaves Estrangeiras | Pedidos em `fact_orders` sem correspondência em alguma dimensão (`store_id`, `driver_id`, `channel_id` ou `date_id` nulos após o join) |
| Valores Numéricos Suspeitos | Valores negativos em campos que não fazem sentido de negócio (`order_amount`, `order_delivery_fee`, `order_delivery_cost`, `delivery_distance_meters`) |
| Unicidade das Dimensões | Confirmação de que as dimensões não têm chave primária duplicada |
 
### Resultados
 
**1ª Validação — Integridade dos Dados**
 
<img width="1342" height="543" alt="image" src="https://github.com/user-attachments/assets/78414624-468f-4947-9db5-13d1ed4dcaaa" />

**Resultado:** Silver ficou igual ou menor que Bronze em todas as tabelas ou seja critério atendido. `deliveries` foi a única com queda expressiva (20.189 linhas), mas por um motivo diferente das demais: não é duplicata exata nem falta de chave primária, é a deduplicação por `delivery_order_id` aplicada na correção da 3ª Validação (múltiplos registros de entrega para o mesmo pedido — ver nota de processo abaixo). As demais 6 tabelas não tinham nenhuma duplicata nem linha sem chave primária no Bronze.
 
**2ª Validação — Conversão de Datas e Horários**
 
<img width="1336" height="532" alt="image" src="https://github.com/user-attachments/assets/0999dfd1-b1c4-4728-8834-42008a7e4d9d" />

**Resultado:** Diferença zero em todos os campos, a conversão para `TIMESTAMP` não introduziu nenhum nulo novo. Todos os nulos já existiam no dado bruto (Bronze), incluindo o alto volume em `order_moment_delivered` (94,7%).
 
> **Nota de processo:** a validação começou medindo só o volume absoluto de nulos no Silver, com a régua "quanto menor, melhor". Isso levantou uma suspeita falsa: `order_moment_delivered` com 94,7% de nulos parecia um possível erro de formatação na conversão. Comparando com a contagem de nulos do Bronze (antes de qualquer conversão), veio a resposta real, o valor já nascia vazio na origem, então a régua certa não é "volume absoluto de nulos", e sim "a conversão criou algum nulo que não existia antes?". Com esse critério, o resultado é claramente positivo (diferença zero) e fica registrado aqui porque é o tipo de refinamento que só aparece testando a própria validação.
 
**3ª Validação — Consistência das chaves estrangeiras**
 
<img width="1326" height="512" alt="image" src="https://github.com/user-attachments/assets/fecf9227-b6d8-4fad-a436-2a39ebc1576f" />

> **Nota de processo — grão de `fact_orders` quebrado:** o achado mais importante desta validação não estava nas colunas checadas, e sim no `total_pedidos`: 378.902, um número **maior** que as 368.999 linhas de `silver.orders` — ou seja, `fact_orders` tinha mais de 1 linha por pedido em alguns casos, quebrando o grão definido no Tópico 3. Investigando, descobrimos 9.531 pedidos com mais de um registro em `silver.deliveries` (mesmo `delivery_order_id`, motoristas diferentes — e num caso, até status conflitante: `DELIVERED` e `CANCELLED` para o mesmo pedido). Corrigido deduplicando `deliveries` antes do join, mantendo 1 linha por pedido e priorizando `DELIVERED` sobre `CANCELLED`.
 
**Resultado após a correção:** `total_pedidos` = 368.999 — bate exatamente com `silver.orders`, confirmando que o grão foi restaurado. `sem_store_id` = 0, `sem_driver_id` = 25.583 (6,9% — caiu levemente dos 25.773 iniciais, já que a deduplicação passou a preferir a linha com motorista atribuído quando havia empate de status), `sem_channel_id` = 0, `sem_date_id` = 0.
 
**4ª Validação — Valores numéricos suspeitos**

<img width="1363" height="565" alt="image" src="https://github.com/user-attachments/assets/14758a22-101e-4bfe-afab-b71399329fa3" />

**Resultado:** Nenhuma inconsistência encontrada.
 
**5ª Validação — Unicidade das dimensões**
 
<img width="1332" height="577" alt="image" src="https://github.com/user-attachments/assets/04b91392-5280-4bde-ac52-2c926a459188" />

**Resultado:** Nenhuma chave primária duplicada.
 
## Análise de Dados (Etapa 4.5)
 
### Dashboard
 
📊 [Dashboard Power BI (.pbix)](https://drive.google.com/file/d/1H-NXhA-Mm4Ol5lWJZRrZZfQbNiXzTkXb/view?usp=drive_link) — arquivo hospedado no Google Drive (acima do limite de tamanho para versionamento no GitHub). Baixe e abra no Power BI Desktop para navegar pelo relatório completo (3 páginas: Visão Geral, Diagnóstico de Falhas, Tempo de Ciclo).
 
### Respostas às perguntas de negócio
 
**1. Quais hubs apresentam a maior distância média de entrega, e isso se relaciona com o volume de pedidos atendidos por hub?**
 
ELIXIR SHOPPING tem a maior distância mediana de entrega (14,4 km), cerca de **6 vezes maior** que a dos demais hubs (~2,3 km), mesmo com um volume de pedidos apenas mediano (~10,2 mil). Já hubs com volume muito maior, como SUBWAY, COFFEE e PURPLE, operam com distâncias menores. Isso indica que **o volume de pedidos não explica a distância de entrega** e sugere uma possível revisão da área de cobertura do ELIXIR SHOPPING.
*(Nota de processo: hubs com menos de 100 pedidos no período foram excluídos dessa análise, pois estava distorcendo a mediana, como visto em RED SHOPPING e HUBLESS SHOPPING.)*

<img width="632" height="375" alt="image" src="https://github.com/user-attachments/assets/0822f340-8767-42ed-a14b-f377c39891df" />

**2. O status da entrega varia significativamente entre hubs ou estados?**
 
**Sim**. O ELIXIR SHOPPING tem a maior taxa de cancelamento (13,6%) entre os hubs com mais de 100 pedidos, muito acima dos demais. O resultado reforça a análise anterior: além de ter a maior distância média de entrega, também é o hub com mais cancelamentos, **indicando um possível problema na sua área de cobertura**.
 
<img width="482" height="282" alt="image" src="https://github.com/user-attachments/assets/1644cafc-449d-4d99-8e6f-cbda0fcd9d0a" />

 
**3. O segmento da loja influencia o valor médio dos pedidos?**
 
Sim. **O segmento GOOD tem um ticket médio de R$ 249,33, mais que o dobro da média geral (R$ 105,15)** e bem acima do FOOD em todos os meses analisados. Isso mostra que os pedidos de GOOD geram mais receita por transação, tornando o segmento uma prioridade comercial.
 
<img width="631" height="362" alt="image" src="https://github.com/user-attachments/assets/af194b15-387c-4b33-ae8d-e0fe27ade447" />

**4. Como o volume e o valor total dos pedidos evoluem mês a mês no período coberto pelos dados?**
 
O volume de pedidos cresceu **45,1%** entre janeiro e abril de 2021, passando de 75 mil para 109 mil pedidos. Após uma leve queda em fevereiro, houve forte crescimento em março, seguido de estabilização em abril. Como a análise cobre apenas quatro meses, o resultado indica uma tendência de curto prazo e **não permite concluir sobre sazonalidade**.
 
<img width="577" height="342" alt="image" src="https://github.com/user-attachments/assets/7aac656f-bb34-4c06-bf3b-f6bcb8aea96a" />

**5. Quais estados concentram a maior receita total e qual é o ticket médio por estado?**
 
| Estado | Pedidos | Receita Total | Ticket Médio |
|---|---|---|---|
| SP | 168 Mil | R$ 21,45 Mi | R$ 127,97 |
| RJ | 138 Mil | R$ 12,31 Mi | R$ 89,50 |
| RS | 34 Mil | R$ 2,99 Mi | R$ 86,96 |
| PR | 29 Mil | R$ 2,04 Mi | R$ 69,58 |
 
São Paulo concentra a maior receita total (R$ 21,45 Mi) e também o maior ticket médio (R$ 127,97) — é o estado mais valioso tanto em volume quanto em valor por pedido. Rio de Janeiro vem em segundo em ambas as métricas (R$ 12,31 Mi / R$ 89,50). Rio Grande do Sul e Paraná têm volume e receita bem menores (~R$ 2–3 Mi cada) e ticket médio também mais baixo. A ordem de prioridade de investimento regional, tanto por escala quanto por valor médio de pedido, é: SP > RJ > RS > PR.
 
🔲 *print: cards de "Total de pedidos"/"Receita total" + gráfico "Ticket médio por segmento" (P3), filtrados por estado.*
 
**6. Quais hubs têm a maior taxa de cancelamento de entregas em relação ao volume de pedidos recebidos?**
 
Entre os hubs com volume suficiente (100+ pedidos), ELIXIR SHOPPING tem, disparado, a maior taxa de cancelamento (13,6%) — mais que o triplo do segundo colocado, STAR SHOPPING (4,1%), e muito acima da maioria dos hubs, que ficam na faixa de 1–3%. Esse é o terceiro achado consecutivo apontando para o mesmo hub (maior distância mediana na pergunta 1, maior taxa de cancelamento aqui e na pergunta 2), reforçando ELIXIR SHOPPING como prioridade máxima de intervenção operacional — provavelmente ligada ao tamanho excessivo da sua área de cobertura.
 
*Nota de processo: hubs com menos de 100 pedidos no período (ex: FUNK SHOPPING, RED SHOPPING, HUBLESS SHOPPING) foram excluídos dessa análise — com volume tão baixo, poucos cancelamentos já geram uma taxa percentual enganosamente alta (ex: FUNK SHOPPING chegava a aparecer com 31% de cancelamento, mas isso vinha de apenas 28 pedidos cancelados em um total de 89).*
 
🔲 *print: gráfico "Taxa de cancelamento por hub" (P6), com o filtro de 100+ pedidos aplicado.*
 
**7. Existe relação entre distância de entrega e taxa de cancelamento?**
 
Existe uma relação clara no extremo, mas não um padrão linear forte no restante da operação. ELIXIR SHOPPING é o único hub que se destaca nas duas métricas simultaneamente — maior distância mediana (14,4 km) e maior taxa de cancelamento (13,6%), muito acima de qualquer outro hub — o que sustenta a hipótese de que problemas de cobertura geográfica extrema levam a mais falhas de entrega. Já entre os demais hubs (distâncias entre 1,3 e 3,4 km), não há uma correlação forte: por exemplo, STAR SHOPPING tem distância baixa (1,76 km) mas cancelamento acima da média (4,05%). Ou seja, distância extrema é um fator de risco claro, mas não é o único nem o principal fator de cancelamento na faixa "normal" de operação.
 
🔲 *print: gráfico de dispersão "Distância × taxa de cancelamento" (P7), por hub e com o filtro de 100+ pedidos aplicado.*
 
**8. Algum tipo/modal de motorista concentra proporcionalmente mais entregas com status de falha?**
 
Não, entre os motoristas efetivamente designados: MOTOBOY (99,96%) e BIKER (99,97%) têm taxas de entrega praticamente idênticas e virtualmente perfeitas — a escolha do modal não é um fator de risco relevante. O problema real está nos pedidos sem motorista atribuído ("Em branco"): esse grupo concentra 27,7% de cancelamento, muito acima de qualquer modal designado. Ou seja, a falha não está ligada ao tipo de veículo/motorista usado na entrega, mas sim à etapa anterior — o processo de atribuição de motorista. Pedidos que não conseguem um motorista designado têm risco de cancelamento drasticamente maior do que qualquer diferença entre modais.
 
🔲 *print: gráfico "Status da entrega por modal do motorista" (P8).*
 
**9. Pedidos com pagamento não aprovado estão concentrados em algum canal, método de pagamento ou hub específico?**
 
Não foi identificada uma concentração relevante por canal. A taxa geral de pagamentos não concluídos é muito baixa (0,11%) e se distribui de forma relativamente uniforme entre os canais — o maior valor (SHOPP PLACE, 0,37%) ainda representa um volume pequeno em termos absolutos. O canal dominante em volume, FOOD PLACE (288.723 pagamentos, 78% do total), tem taxa praticamente igual à média geral (0,12%). Diferente das demais perguntas de diagnóstico, aqui não há um ponto de falha crítico a corrigir — o processo de cobrança se mostra estável na maior parte da operação.
 
🔲 *print: tabela "Pagamentos não concluídos por canal" (P9).*
 
**10. Qual é o tempo médio total do ciclo do pedido por hub, e quais estão significativamente acima da média geral da operação?**
 
Sim, há hubs significativamente acima da média — mais uma vez, ELIXIR SHOPPING é o caso extremo. Usando a mediana (mais robusta a valores extremos que a média — a média chegava a 3.037 min, claramente distorcida por outliers pontuais), ELIXIR SHOPPING ainda apresenta tempo de ciclo mediano de 1.566 minutos (~26h), contra uma faixa de 32 a 47 minutos em todos os demais hubs — quase 40× mais lento. Diferente do que ocorreu na distância (onde a mediana resolveu a distorção), aqui o valor segue extremamente alto mesmo após a correção, o que indica que não se trata apenas de outliers pontuais, e sim de um problema real e generalizado na operação desse hub — reforçando pela terceira vez que ELIXIR SHOPPING é a prioridade de investigação operacional (já era o hub com maior distância mediana e maior taxa de cancelamento).
 
*Nota de processo: como nas perguntas anteriores, aplicamos o filtro de hubs com 100+ pedidos (para descartar distorção por amostra pequena) e usamos mediana em vez de média, já que `order_metric_cycle_time` também apresenta outliers, assim como `delivery_distance_meters`. Ainda assim, o valor de ELIXIR SHOPPING permanece drasticamente acima dos demais — ao contrário da distância, aqui a distorção estatística não explica todo o efeito, sugerindo um gargalo operacional real nesse hub que mereceria investigação mais aprofundada (fora do escopo deste MVP).*
 
🔲 *print: gráfico "Tempo médio de ciclo por hub" (P10, com mediana).*
 
**11. Das etapas do processo, qual mais contribui para o tempo total do ciclo?**
 
A etapa de Produção é a que mais pesa no tempo total do ciclo, com média de 61,79 minutos (42,8% do tempo total), seguida por Trânsito, com 46,85 minutos (32,4%). As demais etapas — Expedição (19,08 min), Pausa (9,19 min), Deslocamento até a loja (4,77 min) e Coleta (2,76 min) — têm impacto bem menor. Isso indica que o investimento corretivo prioritário deve ir para a etapa de produção nas lojas (tempo de preparo do pedido), não para a frota de entrega, já que produção pesa quase 30% a mais que o trânsito no tempo total do processo.
 
*Achado complementar: em ELIXIR SHOPPING, a etapa de Produção sozinha chega a uma média de 1.748 minutos (~29h) — um valor completamente fora do padrão dos demais hubs (13 a 115 min) — confirmando que o gargalo de tempo de ciclo identificado na pergunta 10 para esse hub está concentrado na etapa de produção da loja, não no deslocamento do motorista.*
 
🔲 *print: tabela "Composição do tempo de ciclo por etapa" (P11).*
 
**12. Existe relação entre tempo pausado (`order_metric_paused_time`) e o status final da entrega?**
 
Sim. Pedidos que terminam CANCELLED têm tempo médio pausado de 12 minutos, 33% maior que os pedidos DELIVERED (9 minutos) — ou seja, pausas mais longas estão associadas a maior risco de cancelamento, sustentando a hipótese de um alerta proativo (pausa anormalmente longa como sinal de risco antes do desfecho final). O status DELIVERING aparece com o maior tempo pausado (19,6 min), mas isso reflete pedidos ainda em andamento no momento da coleta dos dados — a pausa continua "correndo" e não é comparável aos status já finalizados. A comparação relevante para o alerta proativo é, portanto, CANCELLED (12 min) vs. DELIVERED (9 min).
 
🔲 *print: gráfico "Tempo médio pausado por status final" (P12).*



