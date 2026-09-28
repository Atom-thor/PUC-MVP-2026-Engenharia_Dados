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
>1 - Como tem evoluído a participação do marketshare dos serviços de taxi legado frente aos serviços de corrida por aplicativo (For Hire Services)? Qual é o marketshare dos taxis no ano atual, 2026, e qual ano foi o ponto de inflexão quando as corridas do tipo "For Hire Services" passaram a dominar?
>
>2 - Qual a taxa de corridas disputadas e nulas para Taxis Verdes e Amarelos nos últimos 2 anos? Esse percentual tem aumentado ou diminuido?
>
>3 - Qual o método de pagamento predominante para os taxis verdes e amarelos nos últimos anos? Como essa distribuição mudou na última década?
>
>4 - Existem registros de corridas suspeitas realizadas por Taxis Amarelos e Verdes? Por corrida suspeita, entende-se como aquelas que não estão de acordo com a legislação de Taxis da cidade: Táxis amarelos e verdes não podem buscar passageiros fora da cidade de Nova Iorque, e Taxis Verdes não podem: Buscar passageiros de Aeroportos exceto quando a corrida tem tarifa pré combinada, buscar passageiros em zonas exclusivas de taxis amarelos.
>
>5 - Quais serviços de corrida por aplicativo ou taxi dominam em cada distrito? Quais são os 5 principais bairros de pico para embarque e desembarque durante o ano, considerando dias úteis e não úteis de 2025 e 2026?
>
>6 - Quais são os horários de pico, para embarque e desembarque, por distrito, em dias úteis e não úteis?
>
>7 - Como a remuneração total do motorista de serviços de corrida por aplicativo tem acompanhado a inflação geral desde 2019?
>
>8 - Como as tarifas base ao passageiro (sem incluir impostos, gorjetas e taxas) tem acompanhado a inflação geral americana, e a inflação especifica do segmento de transportes? As regulamentações do setor taxista fazem com que ela tenha sofrido menos ou mais reajustes na inflação com relação a corridas por aplicativo?

### 1.2 Dados Brutos

Para responder as perguntas de negócio, foram necessários conjuntos de dados brutos de 3 fontes diferentes:

* Os conjuntos de registros de viagens da [TLC](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) (`Yellow Taxi Trip Records`, `Green Taxi Trip Records`, `For Hire Vehicle Trip Records` e `High Volume For Hire Vehicle Trip Recods`) e suas tabelas auxiliares de consultas de zonas de táxi e conversão de códigos de base de despache de veículos para empresas de viagens por aplicativos credenciadas, disponibilizadas no site da TLC e no manual de uso do dataset ([Taxi Zone Lookup Table](https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv) e [trip_record_user_guide](https://www.nyc.gov/assets/tlc/downloads/pdf/trip_record_user_guide.pdf), respectivamente).

* Os índices históricos do [Consumer Price Index for Urban Consumers (CPI-U)](https://fred.stlouisfed.org/series/CPIAUCSL) para o índice geral de inflação, e o [Consumer Price Index Urban Consumers: Transportation in U.S. City Average (CPI-T)](https://fred.stlouisfed.org/series/CPITRNSL), disponibilizados no site do Federal Reserve Bank of St. Louis (FRED), sendo a fonte primária do dado o U.S. Bureau of Labor Statistics (BLS).

* Registros de feriados federais e estaduais observados no estado de Nova Iorque, para as análises de dias úteis vs. não úteis. Os dados foram consumidos pela biblioteca `holidays` do python.

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

## 2. Carga dos dados

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
### 2.1 Evidências

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


## 3. Modelagem e Catálogo de Dados

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

### 3.1 Escolhas de modelagem realizadas

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

- **fato_inflacao_cpi**:
  - Realizado um left join entre as tabelas silver `inflacao_cpi_u` e `inflacao_cpi_t`, por meio da coluna `observation_date`.
  - Realizada uma interpolação linear para preenchimento do mês de outubro de 2025, sem registro devido ao shutdown do governo americano, a fim de não prejudicar a aálise dos dados e resposta às perguntas de negócio.
 
 ### 3.2 Catálogo de dados

 O catálogo de dados da camada gold está descrito abaixo.

 #### **dim_tipo_veiculo**

>Tabela dimensão de informações de tipos de veículos relacionados a serviços de transporte na cidade de Nova Iorque. A tabela possuí informações cadastrais de tipo de veículo, informando a empresa de aplicativo de viagens associada, ou ao tipo de taxi relacionado. Também possui categorização de veículos entre os tipos "For Hire Services" e "Taxi", para diferenciar os taxis tradicionais de serviços de viagem por aplicativos, e código de identificação do veículo.

| Coluna       | Tipo     | Descrição                                                                                                                                                                                                                                                                                           |
|--------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Tipo_Veiculo | `string` | Nome das empresas de transporte por aplicativo com registros de cadastro nas bases da TLC e classificação de taxis entre amarelos e verdes. <br> --- <br>* Uber<br> * Lyft<br>  * Via<br>  * Juno<br>  * Taxi Amarelo<br>  * Taxi Verde |
| ID_Veiculo   | `string` | Código de referência para as empresas de transporte por aplicativo utilizado na tabela "forhirevehicleshighvolume", e ID de identificação criado para identificar taxis amarelos e verdes.<br> ---<br> - TA - Taxi Amarelo<br> - TV - Taxi Verde<br> - HV0002 - Juno<br> - HV0003 - Uber<br> - HV0004 - Via<br> - HV0005 - Lyft |
| Categoria    | `string` | Informa a categoria macro de veículo no cadastro.<br> --- <br>Valores aceitos:<br> * Taxi - Para taxis amarelos e verdes<br> * For Hire Service - Para serviços de transporte por aplicativo.                                                                                                                       |

 ##### Linhagem dos dados

<img width="1502" height="482" alt="image" src="https://github.com/user-attachments/assets/196804fb-2fb8-4f50-a4e6-8288974d3286" />
Figura 7 - Linhagem de dados da tabela dim_tipo_veiculo.

 #### **dim_calendario**

>A tabela contém dados de dimensão de calendário, com período inicial a partir de 2016, e compila informações de ano e número de mês, além de flags que indicam se a data referente é um feriado observado no estado de Nova Iorque, uma data de fim de semana, e se a data é um dia útil ou não útil, servindo para consumo de análises temporais na camada gold.

| Coluna           | Tipo      | Descrição                                                                                                  |
|------------------|-----------|------------------------------------------------------------------------------------------------------------|
| Data             | `date`    | Data de referência do calendário, com início a partir de 1/01/2016.                                        |
| Ano              | `int`     | Valor do ano referente a data.                                                                             |
| Nr_Mes           | `int`     | Valor numérico do mês referente a data.<br>---<br>Valores aceitos: 1 - 12.                                 |
| Fl_Fim_de_Semana | `boolean` | Informa se a data é fim de semana (sábado ou domingo).                                                     |
| Fl_Feriado       | `boolean` | Informa se a data é um feriado observado pelo estado de Nova Iorque (Federal/Estadual).                    |
| Fl_Dia_Util      | `boolean` | Informa se a data é um dia útil ou não útil, considerando as condições da data de feriado e fim de semana. |

 ##### Linhagem dos dados

<img width="1488" height="463" alt="image" src="https://github.com/user-attachments/assets/dc4c0bc5-42b2-4814-b22f-0e4cd763541b" />
Figura 8 - Linhagem de dados da tabela dim_calendario

#### **dim_zonas_de_taxi**

>A tabela contém informações sobre as zonas de taxi da Taxi and Limousine Comission (TLC) de Nova Iorque. A tabela identifica cada zona de serviço por um código numérico, agrupado por bairro, zona e zona de serviço.

| Coluna       | Tipo     | Descrição                                                                                                                                                                                                                                                                                                                                                                                   |
|--------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| IDLocal      | `int`    | Código numérico representando a zona de taxi da TLC. As zonas são aproximadamente baseadas nas áreas de tabulação de vizinhanças do departamento de planejamento urbano de Nova Iorque (Neighborhood tabulation areas - NTAs), e servem para aproximarem-se dos bairros, para poder se analisar os bairros onde um passageiro foi buscado e enviado.<br>---<br>* Faixa de valores:  1 - 265 |
| Distrito     | `string` | Descrição do distrito referente à zona de taxi. É composto pelos 5 distritos oficiais, acrescido de "Desconhecido", "Não Disponível - N/A" e o Aeroporto Internacional Newark Liberty.                                                                                                                                                                                                      |
| Zona         | `string` | Descrição dos bairros e aeroportos referentes à zona de taxi da TLC.                                                                                                                                                                                                                                                                                                                        |
| Zona_Servico | `string` | Descreve a zona de serviço de taxis e veículos a um nível macro. É composto por áreas de aeroportos, distritais, áreas de pickup/street hail exclusivas para taxis amarelos, e valores não disponíveis (N/A).                                                                                                                                                                               |
##### Linhagem dos dados

<img width="1502" height="462" alt="image" src="https://github.com/user-attachments/assets/48182002-3726-4ef0-befd-47233628ae78" />
Figura 9 - Linhagem de dados da tabela dim_zonas_de_taxi

#### **fato_demanda_horaria**

>Tabela fato de registros de viagens por táxis e serviços for hire, ao grão de dia x hora x data de embarque x data destino x hora embarque x hora destino x ID Veículo. Usada para cálculo de demanda de viagens por local de embarque, desembarque, hora e tipo de serviço de transporte.


| Coluna        | Tipo     | Descrição                                                                                             |
|---------------|----------|-------------------------------------------------------------------------------------------------------|
| Data_Embarque | `date`   | dateData de embarque do passageiro.                                                                   |
| Data_Destino  | `date`   | Data de desembarque do passageiro.                                                                    |
| Hora_Embarque | `int`    | Número da hora de embarque do passageiro. Formato 24 horas.                                           |
| Hora_Destino  | `int`    | Número da hora de desembarque do passageiro. Formato 24 horas.                                        |
| ID_Veiculo    | `string` | Código de identificação do tipo de taxi ou número de licença de serviço de transporte de alto volume. |
| ID_Embarque   | `double` | Código de área de embarque da TLC.<br>---<br>Valores aceitos: 1 - 265, exceto 264.                    |
| ID_Destino    | `double` | Código de área de desembarque da TLC.<br>---<br>Valores aceitos: 1 - 265, exceto 264.                 |
| Qtd_Viagens   | `bigint` | Quantidade total de viagens realizadas na agregação.                                                  |

##### Linhagem dos dados

<img width="1507" height="658" alt="image" src="https://github.com/user-attachments/assets/8e7a3917-bee9-49cb-baaa-660be7e9c308" />
Figura 10 - Linhagem dos dados da tabela fato_demanda_horaria

#### **fato_corrida_mensal**

> A tabela possuí um resumo de indicadores de viagens realizadas por taxis amarelos, verdes e for hire services agrupada ao grão mensal. São registradas informações de  remuneração do motorista, somatório da tarifa base, e quantidade de viagens, por método de pagamento, tipo de veículo e local de embarque.

| Coluna                | Tipo            | Descrição                                                                                                                                                                                                                                                                                                                       |
|-----------------------|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Data                  | `date`          | Mês de referência do registro, no formato dd-mm-aaaa, sendo registrado o último dia do mês para cada linha.                                                                                                                                                                                                                     |
| ID_Veiculo            | `string`        | Código de identificação do tipo de taxi ou número de licença de serviço de transporte de alto volume.                                                                                                                                                                                                                           |
| ID_Embarque           | `double`        | Código de área de embarque da TLC.<br>---<br>Valores aceitos: 1 - 265, exceto 264.                                                                                                                                                                                                                                              |
| Metodo_Pagamento      | `string`        | Descrição do método de pagamento escolhido pelo passageiro.<br><br>* Viagem "Flex Fare" <br>* Cartão de crédito <br>* Dinheiro<br>* Sem cobrança <br>* Disputado<br>* Desconhecido<br>* Viagem anulada<br>---<br>Esse campo é apenas preenchido para taxis amarelos e verdes (ID_Veiculo = TA ou TV).                           |
| Qtd_Viagens           | `bigint`        | Quantidade total de viagens realizadas no período.                                                                                                                                                                                                                                                                              |
| Tarifa_Base           | `decimal(12,2)` | Valor total da tarifa base do período cobrada ao passageiro, apurada via soma de "fare_amount" das bases de taxis amarelos e verdes, ou "base_passenger_fare". Não considera gorjetas, impostos e taxas adicionais.<br>Esse campo é apenas preenchido para taxis amarelos e verdes, e for hire services após fevereiro de 2019. |
| Remuneracao_Motorista | `decimal(12,2)` | Remuneração total do motorista. O campo é apenas preenchido para veículos do tipo "For Hire Service" (ID_Veiculo entre HV0002 e HV0005)<br>O campo é calculado a partir da soma de "driver_pay" e "tips" da tabela "forhirevehicleshighvolume"                                                                                  |

 ##### Linhagem dos dados

<img width="1502" height="723" alt="image" src="https://github.com/user-attachments/assets/19793595-d779-435a-afea-2292fed7feb1" />
Figura 11 - Linhagem dos dados da tabela fato_corrida_mensal


#### **fato_corridas_suspeitas**

> A tabela registra as viagens individuais de taxis amarelos e verdes que não cumpriram com alguma das seguintes normativas de operação no município de Nova Iorque:
>
>Táxis amarelos e verdes não podem realizar o embarque de passageiros fora das dependências da cidade;
Táxis verdes não podem realizar o embarque em aeroportos sem taxa negociada previamente, nem realizar embarques em zonas de serviço exclusivas para táxis amarelos.

| Coluna                | Tipo     | Descrição                                                                                                                                                                                                                    |
|-----------------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Provedor              | `string` | Nome do provedor de serviço de taxi.<br>---<br>Valores aceitos:<br>- Creative Mobile Technologies, LLC <br>- Curb Mobility, LLC <br>- Myle Technologies Inc<br>- Helix                                                       |
| Data_Embarque         | `date`   | Data do embarque do passageiro.                                                                                                                                                                                              |
| Data_Destino          | `date`   | Data de desembarque do passageiro.                                                                                                                                                                                           |
| Hora_Embarque         | `int`    | Número da hora de embarque do passageiro. Formato 24 horas.                                                                                                                                                                  |
| Hora_Destino          | `int`    | Número da hora de desembarque do passageiro. Formato 24 horas.                                                                                                                                                               |
| ID_Veiculo            | `string` | Código de identificação do tipo de taxi ou número de licença de serviço de transporte de alto volume.                                                                                                                        |
| ID_Embarque           | `bigint` | Zona de taxi da TLC onde o taxímetro foi acionado, representando o local de início da viagem.<br>---<br>Valor válido: 265                                                                                                    |
| ID_Destino            | `bigint` | Zona de taxi da TLC onde o taxímetro foi parado, representando o local de fim da viagem.<br>---<br>Intervalo válido: De 1 a 265, exceto 264                                                                                  |
| Metodo_Pagamento      | `string` | Descrição do método de pagamento escolhido pelo passageiro.<br><br>* Viagem "Flex Fare" <br>* Cartão de crédito <br>* Dinheiro<br>* Sem cobrança <br>* Disputado<br>* Desconhecido<br>* Viagem anulada                       |
| Tipo_Tarifa           | `string` | Tarifa efetiva ao final da viagem.<br>---<br>- Taxa padrão (Standard fare)<br>- JFK <br>- Newark <br>- Nassau ou Westchester <br>- Taxa negociada (Negotiated fare)<br>- Viagem em grupo (Group ride)<br>- Nulo/Desconhecido |
| Motivo_Irregularidade | `string` | Descrição do motivo de irregularidade da viagem.<br>---<br>- Embarque Fora de Nova Iorque<br>- Embarque em Aeroporto não negociado<br>- Embarque em zona de taxis amarelos                                                   |

 ##### Linhagem dos dados

<img width="1506" height="563" alt="image" src="https://github.com/user-attachments/assets/2fa9437c-e802-487d-94ac-4a9b103cdf55" />
Figura 12 - Linhagem dos dados da tabela fato_corridas_suspeitas

#### **fato_inflacao_cpi**

>A tabela contém dados do "Consumer Price Index" (CPI) dos Estados Unidos, referente ao indicador de inflação do pais. A tabela registra os índices de inflação urbana (CPI-U) e do segmento de transportes (CPI-T)
Os dados são divulgados mensalmente pelo "Bureau of Labor Statistics" (BLS), e pode ser consultado pelas bases de dados públicas da Reserva Federal Americana.
>
>Unidades: Índice 1982-1984=100, ajustado sazonalmente

| Coluna       | Tipo     | Descrição                                                                                                                                                                                                                                                             |
|--------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Data         | `date`   | Data de referência do índice de inflação.                                                                                                                                                                                                                             |
| Indice_CPI_U | `double` | O índice "Consumer Price Index for All Urban Consumers" é um índice de preços de bens e serviços pagos por consumidores urbanos, e é considerado o indicador oficial de inflação americana. <br>---<br>Unidades: Índice 1982-1984=100, Ajustado sazonalmente.         |
| Indice_CPI_T | `double` | O índice "Consumer Price Index for All Urban Consumers: Transportation in U.S. City Average" é um subtipo do índice CPI-U, e avalia a cesta de custos relacionadas ao segmento de transporte urbano.<br>---<br>Unidades: Índice 1982-1984=100, Ajustado sazonalmente. |

 ##### Linhagem dos dados

<img width="1492" height="562" alt="image" src="https://github.com/user-attachments/assets/9df9c4df-5714-474c-9280-6212aa16f574" />
Figura 12 - Linhagem dos dados da tabela fato_inflacao_cpi

---
### 3.3 Screenshot do sistema de catálogo

Segue abaixo a evidência da criação do sistema de catálogo, juntamente com a linhagem de algumas tabelas da camada bronze. Curiosamente, as tabelas bronze não estão visíveis na linhagem das tabelas gold.

<img width="1911" height="1020" alt="image" src="https://github.com/user-attachments/assets/be9c006b-c8b1-441a-9145-6f21c10f4912" />
Figura 13 - Screenshot do Unity Catalogue do Databricks. A imagem mostra a descrição da tabela "yellowtaxitripdata" da camada bronze.

<img width="1503" height="837" alt="image" src="https://github.com/user-attachments/assets/fc2d7368-e2ab-49c5-9689-d1b187ebca14" />
Figura 14 - Linhagem de dados da tabela "yellowtaxitripdata" da camada bronze.

<img width="1501" height="823" alt="image" src="https://github.com/user-attachments/assets/a0b9e119-9f9b-4dd4-8eff-574003ef92e6" />
Figura 15 - Linhagem de dados da tabela "greentaxitripdata" da camada bronze.

<img width="1508" height="558" alt="image" src="https://github.com/user-attachments/assets/70dfb3ed-9c87-4abf-a77d-f7e9b4d73797" />
Figura 16 - Linhagem de dados da tabela "forhirevehicleshighvolume" da camada silver.

---
## 4. Pipeline de dados

O pipeline de dados consiste em um notebook para cada etapa de transformação, iniciando-se na camada `staging` para o download dos arquivos `.parquet` dos datasets da TLC e feriados, seguindo para a camada `bronze` para a ingestão dos dados brutos, `silver` para aplicação de filtros de qualidade de dados e pequenas transformações de dados, e `gold` para a criação das agregações e aplicação de regras de negócio finais.

Todas as tabelas são carregadas em Delta, no modo `overwrite`: cada execução do pipeline reprocessa a camada por completo a partir da anterior, não havendo carga incremental entre execuções.

### Validação e unificação de schema (staging → bronze)

Foi identificado nas etapas iniciais do trabalho que a criação das tabelas na camada `bronze` falhava por divergências de schema nas tabelas da TLC, seja por criação de colunas novas no decorrer dos anos, criação de coluna nova com grafia diferente mas persistência da coluna antiga (e vazia) no mesmo arquivo `.parquet` (`Airport_fee` e `airport_fee`), gerando erro de duplicidade de campos, tanto quanto por variações do tipo de dado de campos entre anos ou entre meses distintos de um mesmo ano (ex: campo registrado como `long`, `int` e `double` em arquivos distintos). Como o Databricks Free Edition não disponibiliza o recurso `mergeSchema`, a unificação de schema entre os arquivos `.parquet` mensais precisou ser feita manualmente. O processo seguiu os seguintes passos:

1. Levantamento das divergências de schema campo a campo em todo o histórico de arquivos baixados de cada dataset, identificando variações de tipo, nome e presença de colunas ao longo dos anos.
2. Definição de um `StructType` único por dataset, adotando o tipo de dado mais permissivo encontrado para cada campo (de forma a acomodar qualquer variação presente no histórico sem perda ou erro de conversão).
3. Conversão do schema de cada arquivo `.parquet` para esse `StructType` definido, previamente à escrita na tabela `bronze`.
4. Escrita dos dados: a primeira execução cria a tabela `bronze` já com o schema unificado; execuções subsequentes fazem o append dos dados já convertidos. Como mencionado, cada execução do notebook realiza um overwrite completo da camada, então o append ocorre apenas dentro da própria execução, arquivo a arquivo, e não entre execuções distintas do pipeline.

### Qualidade de dados (bronze → silver)

Entre as camadas `bronze` e `silver` é aplicada a etapa de qualidade de dados, responsável por filtrar e tratar inconsistências identificadas nos dados brutos antes de sua utilização nas camadas analíticas (detalhes na seção "Qualidade de Dados" deste documento).

As etapas e notebooks utilizados no pipeline são descritas na tabela abaixo.

| Notebook | Camada / Processo | Entrega |
|---|---|---|
| [`02. Staging`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/02.%20Staging.ipynb) | Staging | Download dos arquivos `.parquet` (yellow, green, fhv, fhvhv) e dados auxiliares (feriados) |
| [`03. Análises de Schema`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/03.%20An%C3%A1lises%20de%20Schema.ipynb) | Análises de schema | Avaliação de diferenças de tipagem de dados nos campos dos arquivos `.parquet` das bases brutas da TLC. |
| [`04. Bronze`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/04.%20Bronze.ipynb) | Bronze | Unificação de schema e ingestão dos dados brutos: `yellowTaxiTripData`, `greenTaxiTripData`, `forHireVehicles`, `forHireVehiclesHighVolume`, `taxi_zone_lookup`, `fhv_base_lookup`, `Feriados_US_NY`, `Inflacao_CPI_U`, `Inflacao_CPI_T` |
| [`05. Data Quality`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/05.%20Data%20Quality.ipynb) | Qualidade dos Dados | Análise exploratória dos dados para avaliar completude, acurácia, acurácia e outliers |
| [`06. Silver`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/06.%20Silver.ipynb) | Silver | Filtros de qualidade e transformações |
| [`07. Gold`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/07.%20Gold.ipynb) | Gold | Modelo Dimensional: `dim_calendario`, `dim_zonas_de_taxi`, `dim_tipo_veiculo`, `fato_demanda_horaria`, `fato_corrida_mensal`, `fato_corridas_suspeitas`, `fato_inflacao_cpi` |

---
### 4.1 Screenshots e evidências de persistência de tabelas

<img width="326" height="381" alt="image" src="https://github.com/user-attachments/assets/50f1cee6-6862-48e3-b2af-ec03641b8f95" />

<img width="336" height="390" alt="image" src="https://github.com/user-attachments/assets/e53fd238-7ae6-4463-9aa2-6c9d06b6c2da" />

<img width="353" height="311" alt="image" src="https://github.com/user-attachments/assets/2ec9aefe-7d27-4883-9a74-d84b260fb8ea" />

### 4.2. Qualidade dos Dados

A qualidade dos dados foi explorada no notebook [`03. Análises de Schema`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/03.%20An%C3%A1lises%20de%20Schema.ipynb), entre as etapas da camada `staging` e `bronze` e no notebook [`05. Data Quality`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/05.%20Data%20Quality.ipynb), entre a camada `bronze` e `silver`.

Os principais problemas e resoluções abordados nos notebooks foram resumidos no quadro abaixo.

### Problemas de schema (staging → bronze)

| Problema | Onde | Tratamento |
|---|---|---|
| Tipos divergentes para o mesmo campo entre arquivos (`long`, `int`, `double`), ex: `PULocationID`, `DOLocationID`, `RatecodeID`, `passenger_count` | Todas as bases em `.parquet` da TLC | `StructType` único com o tipo mais permissivo por campo, e conversão de cada arquivo ao schema antes do `append` |
| Coluna renomeada convivendo com a antiga (vazia) no mesmo arquivo: `Airport_fee` × `airport_fee` | Amarelo e HVFHS (2025+) | Função `dedupe_airport_fee` une as duas em `airport_fee` |
| Colunas criadas ao longo do tempo (`dropoff_datetime`, `DOlocationID` na FHV; `cbd_congestion_fee` a partir de 2025) | FHV, todas as bases | Colunas ausentes preenchidas com `NULL` na conformação de schema |
| `mergeSchema` indisponível no Databricks Free Edition | Todas as bases | Diagnóstico com `leituraParquet` e conformação manual arquivo a arquivo |

### Problemas de conteúdo (bronze → silver)

| Problema | Base | Tratamento na silver |
|---|---|---|
| Desembarque anterior ao embarque | Amarelo, Verde, FHV, HVFHS | Registros removidos. Na FHV, o filtro é aplicado apenas a partir de 2018, pois até meados de 2017 o campo não era registrado na base |
| Mês/ano de embarque incompatível com o arquivo de origem (inclui datas fora de 2016 a 2026) | Amarelo, Verde, FHV | Registros removidos |
| Distâncias negativas | Amarelo, Verde | Removidos (`trip_distance >= 0`) |
| Distância zero | Amarelo, Verde | Removidos pela regra tarifa/distância (a divisão por zero resulta em nulo, e o registro é descartado) |
| Distância zero | HVFHS | Removidos (`trip_miles > 0`), pois não há como distinguir cancelamento de erro de coleta |
| Tarifas e gorjetas negativas ou absurdas (ex: tarifa máxima de USD 998.310 no amarelo; gorjeta de USD 900 no verde) | Amarelo e Verde | Removidas tarifas negativas e gorjetas negativas ou acima de USD 100 |
| Tarifa por milha fora do padrão (outliers) | Amarelo, Verde e HVFHS | Limiar pela regra do IQR: USD 30,75/mi (amarelo), USD 31/mi (verde) e USD 13,58/mi (HVFHS) |
| Valores negativos de tarifa base e remuneração do motorista | HVFHS | Removidos (`base_passenger_fare` e `driver_pay >= 0`) |
| `RatecodeID` nulo, fora do dicionário de dados | Amarelo, Verde | Registros removidos |
| `payment_type` nulo, fora do dicionário de dados | Verde | Registros removidos |
| Locais de embarque/desembarque desconhecidos (código 264) | Amarelo, Verde e HVFHS | Removidos (código 264), por não responderem às perguntas de negócio |
| Viagens de aplicativo na base FHV após a criação da HVFHS | FHV | Mantido apenas até jan/2019, quando a HVFHS passou a registrar essas viagens (Local Law 149/2018) |
| Base FHV mistura operadoras de aplicativo e outras bases | FHV | Filtro por `fhv_base_lookup`, mantendo apenas Uber, Lyft, Via e Juno |
| Campos de desembarque e localização não obrigatórios na FHV antes de 2018 | FHV | Validações de destino aplicadas somente a partir de 2018 |
| Feriados com data oficial e data observada duplicadas | Feriados | Mantida apenas a data efetivamente observada |
| Meses sem valor divulgado de inflação | CPI-U e CPI-T | Imputação pela média entre o mês anterior e o seguinte, sinalizada em `fl_imputado` |

## 5. Análise dos Dados

Os códigos usados para as análises de dados estão disponíveis em [`08. Análises`](https://github.com/Atom-thor/PUC-MVP-2026-Engenharia_Dados/blob/main/Notebooks/08.%20An%C3%A1lises.ipynb)

* 1 - Como tem evoluído a participação do marketshare dos serviços de táxi legado frente aos serviços de corrida por aplicativo (For Hire Services)? Qual é o marketshare dos táxis no ano atual, 2026, e qual ano foi o ponto de inflexão quando as corridas do tipo "For Hire Services" passaram a dominar?

<img width="1266" height="498" alt="image" src="https://github.com/user-attachments/assets/b7cdc61c-af61-4fe2-b2c1-6361203967fd" />
<br>
Pode se observar que o segmento "For Hire Services" já representava em 2016 39,1% do mercado de transportes pagos dentro da cidade. Esse segmento cresceu continuamente, deslocando a participação dos táxis tradicionais de forma acelerada até meados de 2020, quando amaduresceu com 85,2% de marketshare. A partir de então o segmento cresceu lentamente, chegando a 89,3% de participação do mercado em 2026.

O ponto de inflexão da dominância dos tradicionais táxis amarelos e verdes foi em meados de 2017, mais especificamente em janeiro, quando atingiu o marco de 50,37% de mercado.

Esse fenômeno mostra que os serviços de transporte por aplicativo de alto volume oferecem algum tipo de vantagem ao passageiro que os táxis não conseguiram competir, podendo ser desde a qualidade do veículo, valor cobrado ao consumidor, ou praticidade de uso, demandando apenas demandar uma viagem via celular.

* 2 - Qual a taxa de corridas disputadas e nulas para táxis verdes e amarelos nos últimos 4 anos? Esse percentual tem aumentado ou diminuido?

Corridas Disputadas
<img width="1262" height="492" alt="image" src="https://github.com/user-attachments/assets/0a1c31c4-6c88-4f5f-a5da-3c9ab5042077" />
<br>

Corridas anuladas
<img width="1256" height="497" alt="image" src="https://github.com/user-attachments/assets/6210d579-5fcf-4e6e-afe3-ef4614ae797a" />
<br>

Corridas sem cobrança
<img width="1261" height="492" alt="image" src="https://github.com/user-attachments/assets/3cbaf33e-adf1-4ca0-bfc0-1c5285e5e48f" />
<br>

Observa-se que, nos últimos quatro anos, as taxas de corridas disputadas, anuladas e sem cobrança representaram um percentual baixo (< 2%) do total de corridas dos táxis amarelos e verdes.

Não foram registradas corridas anuladas no período. No entanto, isso pode ser efeito dos filtros de qualidade aplicados na camada `silver`, como a remoção de corridas com início e fim desconhecidos, uma interação de filtros que não foi explorada na análise de dados.

A taxa de corridas disputadas permaneceu estável para os táxis verdes (0,07% a 0,09% do total), enquanto para os táxis amarelos aumentou de 0,63% em 2023 para 1,43% em 2025. No ano atual, esse percentual caiu para 0,59%, o que indica que os provedores de serviço do táxi amarelo podem ter realizado ajustes operacionais para melhorar o indicador.

Quanto às corridas sem cobrança, tanto os táxis verdes quanto os amarelos apresentaram leve queda no período. Os táxis amarelos reduziram a incidência de 0,35% em 2023 para 0,26% em 2026, enquanto os verdes reduziram de 0,33% para 0,20% no mesmo intervalo.

* 3 - Qual o método de pagamento predominante para os táxis verdes e amarelos nos últimos anos? Como essa distribuição mudou na última década?

Métodos de pagamento para táxis amarelos
<img width="1261" height="493" alt="image" src="https://github.com/user-attachments/assets/d0dcb68e-f78b-437c-a0b8-49379ec0f8e6" />
<br>

Métodos de pagamento para táxis verdes
<img width="1262" height="497" alt="image" src="https://github.com/user-attachments/assets/ee3a5a2f-1558-4781-96e9-b74787560098" />
<br>

Para os táxis amarelos, o cartão de crédito é a forma predominante de pagamento desde 2016, tendo crescido continuamente desde então até representar 88,5% do total de meios de pagamento, sem considerar as corridas disputadas, anuladas e desconhecidas.

Nos táxis verdes, a predominância do cartão de crédito é mais recente: até meados de 2020, a participação do cartão e a do dinheiro eram aproximadamente iguais, e foi a partir de 2021 que o cartão passou a crescer continuamente. O dinheiro ainda corresponde a 22,8% das transações, o que sugere diferenças de perfil entre os passageiros de Manhattan, onde predomina o táxi amarelo, e os dos demais distritos, onde operam os táxis verdes.

* 4 - Existem registros de corridas suspeitas realizadas por Taxis Amarelos e Verdes? Por corrida suspeita, entende-se como aquelas que não estão de acordo com a legislação de Taxis da cidade: Táxis amarelos e verdes não podem buscar passageiros fora da cidade de Nova Iorque, e Taxis Verdes não podem: Buscar passageiros de Aeroportos exceto quando a corrida tem tarifa pré combinada, buscar passageiros em zonas exclusivas de taxis amarelos.

Para responder à pergunta de negócio, foi selecionado uma janela de período dos últimos 3 anos.

Não conformidades por empresa prestadora de serviço

<img width="612" height="116" alt="image" src="https://github.com/user-attachments/assets/f551e815-49f3-4706-8fd2-c8b8c08e275b" />

<br>

Não conformidade por motivo

<img width="645" height="146" alt="image" src="https://github.com/user-attachments/assets/c22840ec-0d40-45ec-b2d3-7f5c22dcfaee" />
<br>

A empresa com maior incidência de irregularidades é a Curb Mobility, LLC, com 95.575 incidentes nos últimos três anos. Embora esse número seja significativamente superior ao da Creative Mobile Technologies, não é possível saber se a diferença decorre de uma maior irregularidade dessa empresa ou simplesmente de ela atender a uma quantidade maior de corridas de táxi.

De toda forma, existem de fato não conformidades e, entre as duas empresas, o principal motivo está relacionado ao embarque de passageiros por táxis verdes em zonas de atendimento exclusivo para táxis amarelos, o que é vetado pelos regulamentos da TLC.

Em seguida, o embarque de passageiros fora da cidade de Nova Iorque também ocorre com alta frequência. Trata-se de outra não conformidade, uma vez que a TLC não possui jurisdição para regulamentar serviços de *street hail* e *livery* fora da cidade. Vale notar que o desembarque fora da cidade é permitido; apenas o embarque é vetado.

A não conformidade de táxis verdes realizarem embarque em aeroportos sem tarifa negociada ocorreu apenas 390 vezes desde 2024. Isso sugere que essa infração é mais fiscalizada, ou que ela não gera vantagem direta ao motorista, ao contrário do embarque em zonas exclusivas de táxis amarelos, que pode ocorrer, por exemplo, quando o motorista já está no local desembarcando outro passageiro.

A existência desses registros aponta oportunidades de melhoria no *compliance* das empresas prestadoras desses serviços aos seus motoristas, no rastreamento de veículos e motoristas e na supervisão da TLC sobre as operadoras credenciadas.

* 5 - Quais serviços de corrida por aplicativo ou taxi dominam em cada distrito? Quais são os 5 principais bairros de pico para embarque e desembarque durante o ano, considerando dias úteis e não úteis de 2025 e 2026?

Predominância de provedor de serviço de transporte por distrito
<img width="1265" height="500" alt="image" src="https://github.com/user-attachments/assets/d50498ab-2a29-4109-ad02-d9db5f3b57ab" />
<br>

Em praticamente todos os distritos, o provedor dominante de viagens é a Uber, exceto para o aeroporto internacional Newark Liberty, onde os taxis amarelos ainda predominam, com 97,49% dos registros de embarque. Nos demais distritos da cidade, apenas Manhattan possuí os taxis amarelos como o segundo provedor mais comum, lugar esse tomado pela Lyft, outro provedor de corridas por aplicativo.

<br>
Principais locais para embarque
<img width="1263" height="542" alt="image" src="https://github.com/user-attachments/assets/aee7ac83-198e-4beb-97f1-fe01b37cc871" />
<br>
Os cinco principais bairros e locais de embarque, em ordem decrescente de demanda, são:

**Em fins de semana e feriados:**

1. Aeroporto JFK
2. Aeroporto de LaGuardia
3. East Village
4. Bushwick South
5. Crown Heights

**Em dias úteis:**

1. Aeroporto de LaGuardia
2. Aeroporto JFK
3. Midtown Center
4. Upper East Side South
5. Times Sq/Theatre District

Observou-se que há variação entre os bairros mais demandados nos dois tipos de dia e que o volume de viagens é menor nos fins de semana, o que indica um menor fluxo de pessoas no interior da cidade e também de deslocamentos de e para fora dela.

Principais locais para desembarque
<img width="1268" height="552" alt="image" src="https://github.com/user-attachments/assets/b027b8a4-ab0d-43ff-b670-a3c055dd3fa2" />
<br>

Os cinco principais bairros e locais de desembarque, em ordem decrescente de demanda, são:

**Em fins de semana e feriados:**

1. Local fora da cidade
2. Aeroporto JFK
3. Aeroporto LaGuardia
4. Crown Heights North
5. Bushwick South

**Em dias úteis:**

1. Local fora da cidade
2. Aeroporto LaGuardia
3. Aeroporto JFK
4. Midtown Center
5. Upper East Side North

O principal destino de desembarque em Nova Iorque é um local fora da cidade, o que indica um alto fluxo de pessoas saindo da cidade.

* 6 - Quais são os horários de pico, para embarque e desembarque, por distrito, em dias úteis e não úteis?

<img width="1267" height="412" alt="image" src="https://github.com/user-attachments/assets/c4075fdd-cc75-4783-bfef-9a54bcf540eb" />
<img width="1268" height="407" alt="image" src="https://github.com/user-attachments/assets/37b6817d-3779-4bb7-b402-a6c35bebb702" />

<br>

O distrito de Manhattan é, de longe, o local com maior fluxo de pessoas embarcando e desembarcando em toda a cidade, independentemente do período ser dia últil ou feriado/finais de semana. Pela leitura do gráfico, observase que o fluxo de carros é intenso a partir 6-7 da manhã, atingindo um pico entre as 18-19 horas da noite, e decrescendo nas horas seguintes. Nos finais de semana e feriados a demanda de veículos continua alta até meados de 2 horas da manhã.
A demanda nos outros distritos é insignificante, de acordo com a escala gráfica


* 8 - Como as tarifas base ao passageiro (sem incluir impostos, gorjetas e taxas) tem acompanhado a inflação geral americana, e a inflação especifica do segmento de transportes? As regulamentações do setor taxista fazem com que ela tenha sofrido menos ou mais reajustes na inflação com relação a corridas por aplicativo?

<img width="1273" height="638" alt="image" src="https://github.com/user-attachments/assets/9370d89d-7f2e-4f0d-a9da-bf13f9109eb0" />

<img width="465" height="111" alt="image" src="https://github.com/user-attachments/assets/5a27703f-1ea0-46f5-97ce-5949a8b9a380" />

<img width="1272" height="628" alt="image" src="https://github.com/user-attachments/assets/aee9f86c-0cd3-47da-8cdb-2017310a6bda" />


Pode-se observar que a remuneração dos motoristas do segmento "For Hire Services" tem crescido significativamente mais que a inflação geral. Para comparativos, a remuneração dos motoristas cresceu 83,5% com relação ao baseline de fevereiro de 2019, versus uma inflação acumulada de 31,3%.

Já o custo de transportes de táxi tem crescido 46,1% no mesmo período, versus uma inflação do segmento de transportes de 37,7%.

## Autoavaliação

Primeiramente, gostaria de agradecer à equipe de docentes da PUC pela mentoria e oportunidade de desenvolver esse trabalho. Independentemente da nota que eu eventualmente tirar nesse trabalho, sinto que aprendi muito sobre engenharia de dados e construção de pipelines.

Quanto ao desempenho do trabalho, sinto que eu poderia ter me dedicado mais à construção das etapas de processamento em si. Acabei gastando um tempo muito longo na ingestão dos dados por meio de download automático de arquivos parquet, e desenvolvimento de menos perguntas de negócio e maior focalização nas respostas analíticas, visto a complexidade da base de dados e grande possibilidade de perguntas de negócio a serem respondidas.

Como oportunidades de melhorias futuras do trabalho, entendo ser oportuno:

* A construção de um orquestrador para atualização mensal dos dados
* A ingestão da etapa de monitoramento de qualidade de dados e integridade de schema para monitoramento constante, visto que a TLC está constantemente mudando o schema das bases
* A ingestão automática de dados de inflação por meio dos bancos de dados do FRED
* A modificação do método de atualização das tabelas nas camadas bronze e silver, para que sejam atualizados pelo modo append em vez de overwrite, para economizar processamento do banco de dados
* A deleção automática de arquivos .parquet após uma certa janela de retenção, para economizar espaço de armazenamento.


---

Referências
===
https://taxicabs.nyc/

https://cityofnewyork.github.io/opendatatsm/LocalLaw11of2012.html
