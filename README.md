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
1 - Como tem evoluído a participação do marketshare dos serviços de taxi legado frente aos serviços de corrida por aplicativo (For Hire Services)? Qual é o marketshare dos taxis no ano atual, 2026, e qual ano foi o ponto de inflexão quando as corridas do tipo "For Hire Services" passaram a dominar?

2 - Qual a taxa de corridas disputadas e nulas para Taxis Verdes e Amarelos nos últimos 2 anos? Esse percentual tem aumentado ou diminuido?

3 - Qual o método de pagamento predominante para os taxis verdes e amarelos nos últimos anos? Como essa distribuição mudou na última década?

4 - Existem registros de corridas suspeitas realizadas por Taxis Amarelos e Verdes? Por corrida suspeita, entende-se como aquelas que não estão de acordo com a legislação de Taxis da cidade: Taxis Amarelos não podem buscar passageiros fora da cidade de Nova Iorque, e Taxis Verdes não podem: Buscar passageiros fora de Nova Iorque, Buscar passageiros de Aeroportos exceto quando a corrida tem tarifa pré combinada, buscar passageiros em zonas exclusivas de taxis amarelos (abaixo da rua XX de Manhattan).

5 - Quais serviços de corrida por aplicativo ou taxi dominam em cada distrito? Quais são os 5 principais bairros de pico para embarque e desembarque durante o ano, considerando dias úteis e não úteis de 2025 e 2026?

6 - Quais são os horários de pico, para embarque e desembarque, por distrito, em dias úteis e não úteis?

7 - Como a remuneração total do motorista de serviços de corrida por aplicativo tem acompanhado a inflação geral desde 2019?

8 - Como as tarifas base ao passageiro (sem incluir impostos, gorjetas e taxas) tem acompanhado a inflação geral americana, e a inflação especifica do segmento de transportes? As regulamentações do setor taxista fazem com que ela tenha sofrido menos ou mais reajustes na inflação com relação a corridas por aplicativo?

Referências
===
Fonte: https://taxicabs.nyc/
