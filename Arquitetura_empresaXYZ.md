### Carlos Reis
### Case técnico para o Grupo SBF - Arquiteto de Dados
# Arquitetura de Plataforma de Dados: Empresa XYZ 


Este documento apresenta uma proposta para a arquitetura de uma plataforma de dados autônoma, pensada para atender aos requisitos específicos e desafios da empresa XYZ. As tecnologias e ferramentas sugeridas foram selecionadas com base em sua eficácia, escalabilidade e adaptabilidade, garantindo uma solução duradoura que pode evoluir de acordo com as demandas de negócio. Para isso, foi utilizado um misto da minha experiência pessoal com referências literárias, principalmente da obra `“Fundamentals of Data Engineering”`, de Joe Reis e Matt Housley, distribuído pela `O'Reilly`. 

Neste cenário, me propus a desenvolver uma solução de Data Lakehouse baseada na cloud da Google (`GCP`). Algumas ferramentas externas e de código aberto, como o `Apache Kafka`, `Beam` e `Airflow`, complementarão a arquitetura para não engessar as escolhas, pois são componentes facilmente desacopláveis da plataforma. A GCP foi escolhida como plataforma de cloud, pois é a que melhor se integra nativamente às tecnologias citadas. A seguir, eu explico as escolhas de tecnologias e seus papéis na arquitetura proposta. 

#### Diagrama da arquitetura final:

![Arquitetura](arquitetura_sbf.jpg)

## Ingestão: 

Precisamos de uma ferramenta que nos possibilite consumir dados rapidamente, pois temos relatórios que precisam ser entregues em 2 minutos ou menos. O `Apache Kafka` é uma plataforma de streaming de dados que nos fornece poder para customizar conectores e ter uma ingestão contínua de maneira rápida e segura, além de uma ótima integração com outros componentes da arquitetura. 

#### Outras possibilidades:

Para a utilização do Kafka na GCP, a melhor forma de uso seria através do `Confluent Cloud`, que oferece uma versão escalável no ambiente de cloud. Dentro da GCP também temos o `Pub/Sub`, uma versão nativa de mensageria por filas que oferece as mesmas funcionalidades principais do Kafka. Apesar de ser uma opção nativa da GCP, o Pub/Sub não tem a mesma flexibilidade na criação de conectores e também não oferece features importantes como persistência em disco, replicação e partições de mensagens, como temos no Kafka. Outra razão para a não utilização do Pub/Sub é a ideia de sermos flexíveis quanto a plataforma de cloud; caso um dia haja razão para uma migração de plataforma, quanto menos ferramentas dependentes dela, melhor.


## Armazenamento: 

Os dados ingeridos pelo Kafka podem ter vários consumidores, um deles será integrado ao `Google Cloud Storage (GCS)`, uma plataforma de armazenamento de objetos que servirá como nosso principal backup de dados no formato em que eles são entregues pela fonte, uma das bases da arquitetura de `Lakehouse`. As vantagens do `GCS` incluem preço de armazenamento, classificação dinâmica de dados, tolerância a falhas e segurança. \
A classificação dinâmica é especialmente importante na escolha de um object storage, pois temos a opção de classificar um dado como `Standard`, `Nearline`, `Coldline` e `Archive`, definido pela frequência de acesso aos dados,tendo uma diferente precificação para cada.

## Processamento: 

Após ter uma maneira segura, rápida e consistente de ingerir e manter os dados na nuvem, precisamos de uma ferramenta para fazer transformações, sejam elas simples como limpeza e validação ou complexas como aplicação de regras de negócio e agregações. Levando em consideração que estamos tratando de um pipeline que precisa lidar com batch e streaming de dados, escolhi o `Dataflow` como principal plataforma de processamento. O Dataflow é implementado no modelo Beam, que oferece uma maneira unificada de processar dados em `batch` e `streaming`, diferenciando a janela de tempo trabalhada nos `jobs`, o que facilita o desenvolvimento e ajuda na manutenção e monitoramento dos pipelines.

### Orquestração e monitoramento de jobs

O Airflow é uma das principais tecnologias de orquestração e monitoramento da atualidade, é bem estabelecida na orquestração de pipelines em batch e nos traz a possibilidade de criar componentes customizados para testes, alertas e monitoramento tanto dos nossos pipelines em batch quanto em streaming. Na GCP, o Airflow pode ser escalado utilizando o `Cloud Composer`, uma solução que cria componentes de infra de maneira nativa, tornando os nossos servidores parametrizáveis.

## Data Warehouse

Em uma implementação de Lakehouse, esta é a espinha dorsal que une a escalabilidade e flexibilidade de um data lake com a confiabilidade, desempenho e usabilidade de um data warehouse, fornecendo uma plataforma unificada para armazenamento, processamento, análise e visualização de dados. Hoje, uma das melhores tecnologias para este fim é o BigQuery, uma plataforma completa e robusta, nativa da GCP, facilmente integrável aos componentes que já temos na arquitetura e a outros que ainda podem surgir, compatível inclusive com outras plataformas de cloud, como a AWS.

## Visualização

Nossos dados estruturados e semiestruturados e catalogados no BigQuery podem ser conectados nativamente a uma vasta gama plataformas de visualização, que, como descrito nos requisitos, utilizaremos mais uma, o que não seria problema nesta arquitetura

## Governança e DataOps

Escolhio as duas soluções nativas da GCP, `Data Catalog`, que se integra facilmente ao Bigquery, e `Cloud Build`, que facilita a implementação de um pipeline robusto com poucos cliques, poupando tempo de implementação de outras ferramentas como `Jenkins`.
