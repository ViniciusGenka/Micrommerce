
# Micrommerce

Esse projeto, apelidado de Micrommerce, é uma abstração de um sistema de **E-commerce**, com o intuito de praticar principalmente a arquitetura de **Microsserviços** por meio do framework **Spring Boot**, com a linguagem java.

## Serviços e Funcionalidades

#### Catalog Service (Serviço de Catálogo)

- Adicionar produto ao catálogo
    - Recebe uma requisição HTTP síncrona para adicionar um produto ao catálogo.
    - Publica um evento de criação de catálogo, com os dados do produto.

#### Inventory Service (Serviço de Inventário)
- Atrelar quantidade de estoque a um produto
    - Consome o evento de criação de catálogo, relacionando um produto a uma quantidade de estoque.
- Obter disponibilidade de produtos
    - Recebe uma requisição HTTP síncrona para retornar a disponibilidade de uma lista de produtos.
- Decrementar o estoque de produtos
    - Recebe uma requisição HTTP síncrona para decrementar o estoque de um produto na quantidade requisitada.

#### Order Service (Serviço de Pedido)
- Emitir um pedido
    - Recebe uma requisição HTTP síncrona para emitir um pedido.
    - Realiza uma requisição HTTP síncrona ao Serviço de Inventário verificando a disponibilidade dos produtos requisitados.
    - Caso disponíveis, realiza uma requisição HTTP síncrona ao Serviço de Inventário decrementando o estoque dos produtos nas respectivas quantidades.
    - Caso o pedido seja emitido com sucesso, publica um evento de emissão de pedido com os dados do pedido.

#### Notification Service (Serviço de Pedido)
- Enviar email
    - Consome o evento de emissão de pedido, enviando um email ao comprador com os dados do pedido.
## Tecnologias Utilizadas

- Eureka Server (Service Discovery)
- Spring Cloud Gateway (API Gateway)
- Spring Cloud OpenFeign (Comunicação Síncrona)
- Apache Kafka (Comunicação Assíncrona)
- Micrometer + Zipkin (Distributed Tracing)
- Prometheus + Grafana (Observabilidade)
- MySQL + MongoDB (Bancos de Dados SQL e NoSQL)
- Cache (Performance em Consultas)
- Lock Otimista (Consistência em Race Conditions)
- Flyway (Migrations)
- JUnit e Mockito (Testes Automatizados)
- Docker (Conteinerização)
- Spring Email (Envio de emails)
## Segurança e Consistência em E-commerces
No desenvolvimento de um sistema de e-commerce, precisa-se levar em conta alguns pontos de segurança que podem acarretar em problemas de consistência e integridade de dados caso não sejam abordados de maneira eficiente.

#### 1. Registros Históricos
Quando clientes compram num e-commerce, muitos esperam que esses dados de pedidos continuem acessíveis para análises futuras. Portanto, caso seja requisitado que o sistema supra essa demanda, deve-se ter cuidado com mutações de dados que possam acarretar num impacto em cadeia.

Exemplo: a simples mudança de residência de um cliente, e a consequente alteração ou deleção do endereço antigo no sistema, pode levar a inconsistências nos registros de pedidos, indicando que pedidos antigos foram enviados para o novo endereço ou, caso o endereço antigo seja completamente deletado, pode levar os pedidos antigos a referenciarem um endereço inexistente.

Soluções: uma solução é a criação de um novo endereço a cada atualização de dados, juntamente com o chamado "soft delete", que utiliza um campo adicional na entidade a fim de sinalizar sua atividade ou inatividade, de forma que ela não precise ser deletada fisicamente para que deixe de ser utilizada na aplicação. Entretanto, um dos downsides dessa abordagem é o aumento do tempo de vida dos dados no sistema, que passa a ser "infinito", já que nunca será apagado, trazendo a necessidade de lidar com esse aumento no volume de dados, seja aumentando a capacidade de armazenamento ou executando limpezas periódicas.

#### 2. Concorrência e Race Conditions

Uma das abordagens para aumento na escalabilidade de aplicações é a concorrência. Ela permite que os recursos sejam utilizados de forma mais eficiente por meio da execução paralela de tarefas. Porém, como tudo possue downsides, ela traz consigo o problema da "condição de corrida". Quando um mesmo recurso é manipulado por diferentes tarefas de forma simultânea, podem acontecer inconsistências no estado desse recurso, caracterizando a condição de corrida.

Exemplo: caso mais de um cliente realize uma compra ao mesmo tempo em cima do mesmo produto, um dos clientes poderia pagar por um produto que nunca seria entregue, caso o estoque não seja suficiente para todos, ou também poderia fazer com que o estoque do produto fosse decrementado em apenas uma unidade, mesmo que multiplos clientes tenham comprado o produto.

Solução: uma das soluções mais comuns para esse tipo de problema são os locks, porém também traz downsides, que além de uma certa complexidade dependendo da situação, também acaba tirando uma parcela do ganho de escalabilidade obtido pela utilização de concorrência. Um lock no contexto de concorrência é o bloqueio de operações de leitura ou atualização de um recurso, impedindo que tarefas diferentes acessem sobre o mesmo recurso enquanto esse lock estiver ativo, eliminando a concorrência e por consequência a condição de corrida. No caso do exemplo, caso a abordagem de locks seja implementada em cima da quantidade em estoque de um produto, apenas um cliente por vez poderia acessar esse dado, evitando as inconsistências na hora da compra.
