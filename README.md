# MVP Engenharia de Dados: Dinâmicas e comportamento das viagens de táxi e serviços de transporte por aplicativo em Nova Iorque

Pipeline no Databricks Free Edition sobre o conjunto de dados da Taxi & Limousine Commission (TLC) da cidade de Nova Iorque.

 > Nome: João Bruno de Almeida Morais  
 > Matrícula: 4052026000080
---
## 1. Contexto de Negócio e Perguntas

O sistema de táxis tradicionais na cidade de Nova Iorque é composto pelos famosos táxis amarelos e os verdes, ambos licenciados e regulamentados pela Taxi and Limousine Commission (TLC), que também é responsável por regulamentar a operação de serviços de limousines de luxo, rádiotáxis e serviços de transporte por aplicativo de alto volume, como Uber e Lyft. A TLC também determina algumas regras gerais de operação desses serviços. Os táxis amarelos possuem permissão de realizar embarque e desembarque em todos os cinco distritos da cidade, enquanto os táxis verdes foram criados para operar nas áreas periféricas com baixa taxa de serviço dos táxis amarelos. Os táxis verdes possuem permissão para realizar o desembarque em todos os distritos, mas são vetados de aceitar o embarque na área central de Manhattan, abaixo da Rua 110 Oeste (West Side) e abaixo da Rua 96 Leste (East Side), e não podem realizar o embarque em aeroportos a menos que a viagem tenha sido pré-combinada com antecedência. Como ambos os serviços são operados pela cidade de Nova Iorque, eles não podem realizar o embarque de passageiros fora das dependências da cidade, e são os únicos autorizados a aceitar o embarque por aceno na rua ("street hails"), sendo que os táxis verdes só podem fazê-lo fora da área restrita de Manhattan (Taxi Cabs, 2026).

Nas últimas décadas, esse sistema de táxis regulados vem enfrentando uma das transformações mais rápidas e visíveis do setor de transportes nos Estados Unidos: a ascensão dos aplicativos de transporte por app (Uber, Lyft, Via) frente ao modelo tradicional. Esse fenômeno gera questões de interesse para políticas públicas de mobilidade, regulação de tarifas, congestionamento urbano e renda de motoristas, temas que a própria TLC monitora de perto ao ponto de ter criado, em 2019, uma categoria regulatória específica (High Volume For Hire Service) para empresas de alto volume.

Do ponto de vista de um gestor de mobilidade urbana, de um analista de políticas públicas ou mesmo de um investidor observando o setor, entender a dinâmica competitiva entre esses dois modelos de negócio é fundamental: Quando exatamente o mercado virou a favor dos apps? Essa dominância é uniforme em todos os 5 distritos da cidade, ou ainda existem locais onde os serviços legados de táxi ainda predominam? Os padrões de demanda de embarque e desembarque são os mesmos em dias úteis e não úteis? Quais são os bairros com maiores picos de demanda, e quais são os horários da rush na cidade?

Por fim, temos ainda questões de custo ao consumidor e remuneração do motorista de aplicativos de transporte. Enquanto a tarifa do táxi tradicional é definida por regulação municipal (portanto previsível e sujeita a reajustes formais), a tarifa dos aplicativos segue a lógica de precificação dinâmica baseada em oferta e demanda. Isso levanta questões sobre o impacto do custo desse tipo de transporte no bolso do consumidor, e como ele tem se reajustado frente aos custos de transporte urbano em geral. Ainda, os motoristas de transporte por aplicativos não são funcionários formais, mas sim, prestadores autônomos de serviço, e como a remuneração total deles tem evoluído nos últimos anos, e ela tem sido o suficiente para superar a inflação dos últimos anos?

### 1.1 Perguntas de negócio
---
>
>1 - Como tem evoluído a participação do marketshare dos serviços de taxi legado frente aos serviços de corrida por aplicativo (For Hire Services)? Qual é o >marketshare dos taxis no ano atual, 2026, e qual ano foi o ponto de inflexão quando as corridas do tipo "For Hire Services" passaram a dominar?
>
>2 - Qual a taxa de corridas disputadas e nulas para Taxis Verdes e Amarelos nos últimos 2 anos? Esse percentual tem aumentado ou diminuido?
>
>3 - Qual o método de pagamento predominante para os taxis verdes e amarelos nos últimos anos? Como essa distribuição mudou na última década?
>
>4 - Existem registros de corridas suspeitas realizadas por Taxis Amarelos e Verdes? Por corrida suspeita, entende-se como aquelas que não estão de acordo com a >legislação de Taxis da cidade: Taxis Amarelos não podem buscar passageiros fora da cidade de Nova Iorque, e Taxis Verdes não podem: Buscar passageiros fora de Nova >Iorque, Buscar passageiros de Aeroportos exceto quando a corrida tem tarifa pré combinada, buscar passageiros em zonas exclusivas de taxis amarelos (abaixo da rua XX >de Manhattan).
>
>5 - Quais serviços de corrida por aplicativo ou taxi dominam em cada distrito? Quais são os 5 principais bairros de pico para embarque e desembarque durante o ano, >considerando dias úteis e não úteis de 2025 e 2026?
>
>6 - Quais são os horários de pico, para embarque e desembarque, por distrito, em dias úteis e não úteis?
>
>7 - Como a remuneração total do motorista de serviços de corrida por aplicativo tem acompanhado a inflação geral desde 2019?
>
>8 - Como as tarifas base ao passageiro (sem incluir impostos, gorjetas e taxas) tem acompanhado a inflação geral americana, e a inflação especifica do segmento de >transportes? As regulamentações do setor taxista fazem com que ela tenha sofrido menos ou mais reajustes na inflação com relação a corridas por aplicativo?

### 1.2 Dados Brutos

Para responder as perguntas de negócio, foram necessários conjuntos de dados brutos de 3 fontes diferentes:

* Os conjuntos de registros de viagens da [TLC](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) (`Yellow Taxi Trip Records`, `Green Taxi Trip Records`, `For Hire Vehicle Trip Records` e `High Volume For Hire Vehicle Trip Recods`) e suas tabelas auxiliares de consultas de zonas de táxi e conversão de códigos de base de despache de veículos para empresas de viagens por aplicativos credenciadas, disponibilizadas no site da TLC e no manual de uso do dataset ([Taxi Zone Lookup Table](https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv) e [trip_record_user_guide](https://www.nyc.gov/assets/tlc/downloads/pdf/trip_record_user_guide.pdf), respectivamente).

* Os índices históricos do [Consumer Price Index for Urban Consumers (CPI-U)](https://fred.stlouisfed.org/series/CPIAUCSL) para o índice geral de inflação, e o [Consumer Price Index Urban Consumers: Transportation in U.S. City Average (CPI-T)](https://fred.stlouisfed.org/series/CPITRNSL), disponibilizados no site do Federal Reserve Bank of St. Louis (FRED), sendo a fonte primária do dado o U.S. Bureau of Labor Statistics (BLS).

* Registros de feriados federais e estaduais observados no estado de Nova Iorque, para as análises de dias úteis vs. não úteis. Os dados foram consumidos pela bilioteca `holidays` do python.

### 1.3 Licenças de uso

#### Dados de registros de viagens da TLC

Os dados são disponibilizados como conjuntos de dados públicos (*public data sets*) da cidade de Nova Iorque, conforme a *Local Law 11 de 2012* que rege a publicação de dados no portal municipal. Os principais pontos aplicáveis são:

- **Sem restrições de acesso**: os dados podem ser usados livremente, sem necessidade de registro, licença ou restrições de uso, desde que a fonte, a versão do conjunto de dados e quaisquer modificações realizadas sejam explicitamente identificadas por quem os disponibilizar a terceiros.
- **Isenção de garantias**: os dados são fornecidos apenas para fins informativos. A cidade não garante a completude, exatidão, conteúdo ou adequação dos dados para qualquer finalidade específica.
- **Isenção de responsabilidade**: a cidade não se responsabiliza por deficiências nos dados ou em aplicações de terceiros que os utilizem.

<img width="1225" height="180" alt="image" src="https://github.com/user-attachments/assets/ea841483-9e8a-41b0-bff0-42674c5963ec" />
Figura 1 - Descrição dos termos de uso dos dados públicos da cidade de Nova Iorque.

#### Dados de índices de inflação da BLS, consumidos via FRED

Como o U.S. Bureau of Labor Statistics é uma agência do governo federal americano, todas as informações publicadas, seja em mídias físicas ou eletrônicas são de domínio público. A agência libera o uso sem necessidade de autorização prévia, mas é solicitado que seja referenciada a fonte do dado.

> Licença completa disponível em: https://www.bls.gov/opub/copyright-information.htm

#### Dados de feriados da biblioteca holidays

A biblioteca `holidays` (Vacanza Team e colaboradores, incluindo dr-prodigy e ryanss) é distribuída sob a **Licença MIT**. A licença permite uso, cópia, modificação e distribuição livres, desde que o aviso de copyright original e a permissão de licença sejam mantidos junto ao software. O software é fornecido "como está", sem garantias de qualquer tipo.

> Copyright (c) Vacanza Team and individual contributors (see CONTRIBUTORS file)
> Copyright (c) dr-prodigy <dr.prodigy.github@gmail.com>, 2017-2023
> Copyright (c) ryanss <ryanssdev@icloud.com>, 2014-2017
>
> Licença completa disponível em: https://github.com/vacanza/holidays/blob/dev/LICENSE

### 1.4 Estrutura dos dados

#### Dados de registros de viagens da TLC

| Tabela                        | Estrutura  | Observação |
|-------------------------------|------------|------------|
| Yellow Taxi Trip Records      | VendorID, tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance, RatecodeID, store_and_fwd_flag, PULocationID, DOLocationID, payment_type, fare_amount, extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount, congestion_surcharge, airport_fee, cbd_congestion_fee | Estrutura obtida via [dicionário de dados do provedor](https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf) |
| Green Taxi Trip Records       | VendorID, lpep_pickup_datetime, lpep_dropoff_datetime, passenger_count, trip_distance, RatecodeID, store_and_fwd_flag, PULocationID, DOLocationID, payment_type, fare_amount, extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount, cbd_congestion_fee, congestion_surcharge, trip_type | Estrutura obtida via [dicionário de dados do provedor](https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_green.pdf) |
| For Hire Services             | Affiliated_base_number, pickup_datetime, dropOff_datetime, DOlocationID, PUlocationID, SR_Flag, dispatching_base_num | Estrutura obtida via [dicionário de dados do provedor](https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_fhv.pdf) |
| High Volume For Hire Services | originating_base_num, dispatching_base_num, request_datetime, on_scene_datetime, pickup_datetime, dropoff_datetime, DOLocationID, PULocationID, access_a_ride_flag, airport_fee, base_passenger_fare, bcf, cbd_congestion_fee, congestion_surcharge, driver_pay, hvfhs_license_num, sales_tax, shared_match_flag, shared_request_flag, tips, tolls, trip_miles, trip_time, wav_match_flag, wav_request_flag | Estrutura obtida via [dicionário de dados do provedor](https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_hvfhs.pdf) |
| fhv_base_lookup | High_Volume_License_Number, License_Number, App_Company_Affiliation| Estrutura copiada do [manual de uso do dataset](https://www.nyc.gov/assets/tlc/downloads/pdf/trip_record_user_guide.pdf) |
| taxi_zone_lookup | LocationID, Borough, Zone, service_zone | Estrutura consultada diretamente da [fonte em .csv](https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv) |

#### Dados de índices de inflação da BLS, consumidos via FRED

| Tabela  | Estrutura  |
|---------|------------|
| CPIAUCSL | observation_date, CPIAUCSL |
| CPITRNSL | observation_date, CPITRNSL |

#### Dados de feriados da biblioteca holidays

| Tabela  | Estrutura  |
|---------|------------|
| feriados_us_ny (nome dado durante a ingestão na bronze) | date, holiday_name |

### 2. Carga dos dados

O notebook [`01. Preparação`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/01.%20Prepara%C3%A7%C3%A3o.ipynb) realizou a criação do catálogo `MVP`, e schemas para cada etapa do pipeline:
- Landing: Para o download dos dados brutos em volumes dedicados, explicitado a seguir;
- Bronze: Para a materialização das tabelas contendo os dados brutos;
- Silver: Para a materialização das tabelas com filtros de qualidade aplicados;
- Gold: Para as tabelas dimensão e fato finais.

O notebook em seguida criou volumes dedicados para cada tipo de dataset consumido, na camada `staging`:
- yellow_taxi: Para download de arquivos `.parquet` da base de dados Yellow Taxi Trip Data;
- green_taxi: Para download de arquivos `.parquet` da base de dados Green Taxi Trip Data;
- fhv: Para download de arquivos `.parquet` da base de dados For Hire Vehicles Taxi Trip Data;
- fhvhv: Para download de arquivos `.parquet` da base de dados High Volume For Hire Vehicles Taxi Trip Data;
- taxi_zones: Para o download do arquivo `.csv` de consulta de distritos, bairros e zonas de serviço por ID de zona de táxi.
- misc: Para o download de arquivos de outras naturezas não especificadas nos demais volumes.

Em sequência, o notebook [`02. Staging`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/02.%20Staging.ipynb) realizou o download sequencial dos arquivos `.parquet` das bases de táxi amarelo, verde, FHV (For-Hire Vehicle) e HVFHS (High Volume For-Hire Services) em seus respectivos volumes dedicados, utilizando funções específicas. Os dados foram baixados a partir de 2016 para táxi amarelo, verde e FHV, e a partir de fevereiro/2019 para o HVFHS, quando as viagens de aplicativos de alto volume passaram a ser registradas em base própria.

O notebook também foi utilizado para consumir os dados de feriados da biblioteca `holidays` do python, e salvou-a no volume `misc`.

Os dados de inflação e tabelas auxiliares de zonas de táxi e empresas afiliadas às bases FHV e HVFHS foram carregadas por meio de upload manual a partir de suas respectivas fontes ([CPIAUCSL](https://fred.stlouisfed.org/series/CPIAUCSL), [CPITRNSL](https://fred.stlouisfed.org/series/CPITRNSL), [zonas de táxi](https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv), [bases e empresas filiadas](https://www.nyc.gov/assets/tlc/downloads/pdf/trip_record_user_guide.pdf)). A tabela de empresas afiliadas aos serviços for-hire `fhv_lookup.csv` foi construída manualmente em uma planilha Excel e posteriormente convertida em `.csv`, com base no [manual de uso do dataset](https://www.nyc.gov/assets/tlc/downloads/pdf/trip_record_user_guide.pdf) da TLC.

A ingestão dos dados na camada bronze é abordada nos tópicos seguintes.

---
#### 2.1 Evidências

<img width="1287" height="697" alt="Captura de tela 2026-09-27 161223" src="https://github.com/user-attachments/assets/129debe1-20b3-4cba-8cab-8f659e3a593b" />
Figura 2 - Log dos últimos downloads dos datasets da TLC, realizados pela função.  
<br><br>

<img width="1492" height="892" alt="image" src="https://github.com/user-attachments/assets/bb4d5776-e485-4dd0-b92f-294a32f04e21" />
Figura 3 - Evidência da persistência dos dados brutos da base da TLC nos volumes.  <br><br>
<br><br>

<img width="1270" height="343" alt="Captura de tela 2026-09-27 161234" src="https://github.com/user-attachments/assets/3b9494f9-cd7f-4d76-9e37-7ef919c84491" />
Figura 4 - Função usada para consumo de dados de feriados.

<br><br>
<img width="549" height="284" alt="image" src="https://github.com/user-attachments/assets/212f539a-9e79-4d27-b832-52f40877bc79" />
<img width="549" height="218" alt="image" src="https://github.com/user-attachments/assets/d6d1c63f-edb7-41f1-b4e9-52eb86a869d5" />

Figuras 5 e 6 - Bases de dados enviadas manualmente ao volume `misc` e `taxi_zones`

### 2. Modelagem e Catálogo de Dados

Modelo de dados: Esquema estrela com 4 tabelas fato e 3 dimensões.

<img width="632" height="795" alt="image" src="https://github.com/user-attachments/assets/4ef655ce-3c95-44ba-adad-1805f6338004" />

Dimensões criadas: 

- **dim_calendario**: Usada para agregações e filtros de ano e mês, feriados e dias úteis/não úteis.
- **dim_tipo_veiculo**: Usada para agregar os códigos de base FHS e HVFHV em suas respectivas empresas de viagem por aplicativo afiliadas (Uber, Lyft, Via), tipos de táxi (amarelo, verde), e por tipo de veículo (taxi, For Hire Service).
- **dim_zonas_de_taxi**: Usado para segmentar e agregar os códigos de localização de embarque e desembarque nos respectivos distritos, bairros, e zonas de serviço de referência.

Fatos criadas:

- **fato_corrida_mensal**: Tabela criada para responder às perguntas 1, 2, 3, 5, 7. Realiza uma agregação de informações de viagens na granularidade mês x Local Embarque x Veículo x Método Pagamento.
  
- **fato_demanda_horaria**: Tabela criada para responder às perguntas 5 e 6. Agrega informações de viagem ao grão dia x hora embarque x hora desembarque x local embarque x local desembarque x veículo.

- **fato_corridas_suspeitas**: Tabela criada para responder a pergunta 4. Retorna as informações individuais de viagens que não cumprem a legislação municipal de operação de taxis amarelos e verdes.

#### 2.1 Escolhas de modelagem realizadas

- **dim_tipo_veiculo**: Para permitir que a dimensão possa ser usada para todos os tipos de veículos do dataset, independentemente de serem do tipo `taxi` ou `for hire services`, foi criada uma chave substituta para os táxis amarelos (`ID_Veiculo = "TA"`), e táxis verdes (`ID_Veiculo = TV`), definindo a `Categoria` de ambos como "Taxi", e o `Tipo_Veiculo` em `Taxi Amarelo` ou `Taxi Verde`.

- **dim_calendario**: Foi utilizado como flag de feriado apenas os feriados de fato observados no estado de Nova Iorque. Alguns feriados foram disponibilizados na biblioteca `holidays` como contendo tanto a data oficial, quanto o dia de fato observado, quando o primeiro não estava adequado às regras de observância de feriados americanos. Essa regra específica foi tratada na camada `silver`.

- **fato_corrida_mensal**:
  - Para poder-se ter o histórico de viagens realizadas por serviços de corrida por aplicativo de alto volume anteriores a 2019, foi utilizada a base `For Hire Services`, filtrando os números de base de despache coincidentes com os presentes na tabela `fhv_base_lookup`. Dados de distância, tarifas e remuneração ao motorista não estão presentes para essa categoria entre os períodos de 2016 a jan/2019;
  - Os indicadores de tarifa base, remuneração do motorista e quantidade de corridas estão agregadas como soma ao grão mensal, a fins de possibilitar o cálculo de médias e indicadores independentemente a agregação utilizada nas análises finais.
  - Para taxis amarelos e verdes, foi usada a chave substituta `TA` e `TV`, respectivamente, no campo `ID_Veiculo` para viabilizar o relacionamento com a `dim_tipo_veiculo`.
  - Para veiculos do tipo For Hire Services nos períodos anteriores a fev/2019, foi feita a substituição do número de base de despache pelo código de operadora credenciada do serviço high volume for hire usada após fev/2019, a fins de simplificar a modelagem de dados na `dim_tipo_veiculo`, mantendo-a enxuta, com um `ID_Veiculo` por tipo de operadora/taxi. A base legada For Hire Services possuia mais de 10 códigos de base de despache distintos relacionados a um único operador credencialdo (ex: Uber).
 
 - **fato_demanda_horaria**:
  - Para poder-se ter o histórico de viagens realizadas por serviços de corrida por aplicativo de alto volume anteriores a 2019, foi utilizada a base `For Hire Services`, filtrando os números de base de despache coincidentes com os presentes na tabela `fhv_base_lookup`. Dados de data, hora e local de desembarque não estão disponíveis para períodos anteriores a meados de 2017, quando a TLC passou a oficialmente exigir o registro para For Hire Services;
  - Para taxis amarelos e verdes, foi usada a chave substituta `TA` e `TV`, respectivamente, no campo `ID_Veiculo` para viabilizar o relacionamento com a `dim_tipo_veiculo`.
  - Para veiculos do tipo For Hire Services nos períodos anteriores a fev/2019, foi feita a substituição do número de base de despache pelo código de operadora credenciada do serviço high volume for hire usada após fev/2019, a fins de simplificar a modelagem de dados na `dim_tipo_veiculo`, mantendo-a enxuta, com um `ID_Veiculo` por tipo de operadora/taxi. A base legada For Hire Services possuia mais de 10 códigos de base de despache distintos relacionados a um único operador credencialdo (ex: Uber).
  - As colunas de embarque e desembarque são ingeridas originalmente no formato datetime na bronze. Para a gold, é realizada a separação de datas como tipo date, e horários são truncados ao número inteiro da hora.
  - A tabela possui seus registros agregados ao grão de dia e hora.

 - **fato_corridas_suspeitas**:
  - Apenas são registradas viagens realizadas por táxis amarelos e verdes, que possuem restrições a locais de operação de serviço não aplicáveis a categoria for hire services;
  - Para taxis amarelos e verdes, foi usada a chave substituta `TA` e `TV`, respectivamente, no campo `ID_Veiculo` para viabilizar o relacionamento com a `dim_tipo_veiculo`.
  - Não foi realizada nenhuma agregação de grão para essa tabela.
  - Apenas são carregados registros que não cumprem alguma das normas de operação dos taxis: Não é permitido realizar o embarque de passageiros fora da cidade de Nova Iorque, taxis verdes não podem realizar o embarque em aeroportos a menos que a tarifa seja combinada previamente, nem podem realizar o embarque de passageiros na zona de serviço exclusiva para táxis amarelos.

---
### 2. Modelagem e Catálogo de Dados

Modelo de dados: Esquema estrela com 4 tabelas fato e 3 dimensões.

<img width="632" height="795" alt="image" src="https://github.com/user-attachments/assets/4ef655ce-3c95-44ba-adad-1805f6338004" />

Dimensões criadas: 

- **dim_calendario**: Usada para agregações e filtros de ano e mês, feriados e dias úteis/não úteis.
- **dim_tipo_veiculo**: Usada para agregar os códigos de base FHS e HVFHV em suas respectivas empresas de viagem por aplicativo afiliadas (Uber, Lyft, Via), tipos de táxi (amarelo, verde), e por tipo de veículo (táxi, For-Hire Service).
- **dim_zonas_de_taxi**: Usada para segmentar e agregar os códigos de localização de embarque e desembarque nos respectivos distritos, bairros, e zonas de serviço de referência.

Fatos criadas:

- **fato_corrida_mensal**: Tabela criada para responder às perguntas 1, 2, 3, 5, 7. Realiza uma agregação de informações de viagens na granularidade mês x Local Embarque x Veículo x Método Pagamento.
  
- **fato_demanda_horaria**: Tabela criada para responder às perguntas 5 e 6. Agrega informações de viagem ao grão dia x hora embarque x hora desembarque x local embarque x local desembarque x veículo.

- **fato_corridas_suspeitas**: Tabela criada para responder à pergunta 4. Retorna as informações individuais de viagens que não cumprem a legislação municipal de operação de táxis amarelos e verdes.

#### 2.1 Escolhas de modelagem realizadas

- **dim_tipo_veiculo**: Para permitir que a dimensão possa ser usada para todos os tipos de veículos do dataset, independentemente de serem do tipo `taxi` ou `for hire services`, foi criada uma chave substituta para os táxis amarelos (`ID_Veiculo = "TA"`) e táxis verdes (`ID_Veiculo = "TV"`), definindo a `Categoria` de ambos como "Taxi", e o `Tipo_Veiculo` em `Taxi Amarelo` ou `Taxi Verde`.

- **dim_calendario**: Foi utilizado como flag de feriado apenas os feriados de fato observados no estado de Nova Iorque. Alguns feriados foram disponibilizados na biblioteca `holidays` como contendo tanto a data oficial quanto o dia de fato observado, quando o primeiro não estava adequado às regras de observância de feriados americanos. Essa regra específica foi tratada na camada `silver`.

- **fato_corrida_mensal**:
  - Para poder-se ter o histórico de viagens realizadas por serviços de corrida por aplicativo de alto volume anteriores a 2019, foi utilizada a base `For Hire Services`, filtrando os números de base de despacho coincidentes com os presentes na tabela `fhv_base_lookup`. Dados de distância, tarifas e remuneração ao motorista não estão presentes para essa categoria entre os períodos de 2016 a jan/2019;
  - Os indicadores de tarifa base, remuneração do motorista e quantidade de corridas estão agregados como soma ao grão mensal, a fim de possibilitar o cálculo de médias e indicadores independentemente da agregação utilizada nas análises finais.
  - Para táxis amarelos e verdes, foi usada a chave substituta `TA` e `TV`, respectivamente, no campo `ID_Veiculo` para viabilizar o relacionamento com a `dim_tipo_veiculo`.
  - Para veículos do tipo For Hire Services nos períodos anteriores a fev/2019, foi feita a substituição do número de base de despacho pelo código de operadora credenciada do serviço high volume for hire usado após fev/2019, a fim de simplificar a modelagem de dados na `dim_tipo_veiculo`, mantendo-a enxuta, com um `ID_Veiculo` por tipo de operadora/táxi. A base legada For Hire Services possuía mais de 10 códigos de base de despacho distintos relacionados a um único operador credenciado (ex: Uber).
 
 - **fato_demanda_horaria**:
  - Para poder-se ter o histórico de viagens realizadas por serviços de corrida por aplicativo de alto volume anteriores a 2019, foi utilizada a base `For Hire Services`, filtrando os números de base de despacho coincidentes com os presentes na tabela `fhv_base_lookup`. Dados de data, hora e local de desembarque não estão disponíveis para períodos anteriores a meados de 2017, quando a TLC passou a exigir oficialmente o registro para For Hire Services;
  - Para táxis amarelos e verdes, foi usada a chave substituta `TA` e `TV`, respectivamente, no campo `ID_Veiculo` para viabilizar o relacionamento com a `dim_tipo_veiculo`.
  - Para veículos do tipo For Hire Services nos períodos anteriores a fev/2019, foi feita a substituição do número de base de despacho pelo código de operadora credenciada do serviço high volume for hire usado após fev/2019, a fim de simplificar a modelagem de dados na `dim_tipo_veiculo`, mantendo-a enxuta, com um `ID_Veiculo` por tipo de operadora/táxi. A base legada For Hire Services possuía mais de 10 códigos de base de despacho distintos relacionados a um único operador credenciado (ex: Uber).
  - As colunas de embarque e desembarque são ingeridas originalmente no formato datetime na bronze. Para a gold, é realizada a separação de datas como tipo date, e horários são truncados ao número inteiro da hora.
  - A tabela possui seus registros agregados ao grão de dia e hora.

 - **fato_corridas_suspeitas**:
  - Apenas são registradas viagens realizadas por táxis amarelos e verdes, que possuem restrições a locais de operação de serviço não aplicáveis à categoria for hire services;
  - Para táxis amarelos e verdes, foi usada a chave substituta `TA` e `TV`, respectivamente, no campo `ID_Veiculo` para viabilizar o relacionamento com a `dim_tipo_veiculo`.
  - Não foi realizada nenhuma agregação de grão para essa tabela.
  - Apenas são carregados registros que não cumprem alguma das normas de operação dos táxis: não é permitido realizar o embarque de passageiros fora da cidade de Nova Iorque; táxis verdes não podem realizar o embarque em aeroportos a menos que a tarifa seja combinada previamente, nem podem realizar o embarque de passageiros na zona de serviço exclusiva para táxis amarelos.

- **fato_**

---

Referências
===
Fonte: https://taxicabs.nyc/
https://cityofnewyork.github.io/opendatatsm/LocalLaw11of2012.html
