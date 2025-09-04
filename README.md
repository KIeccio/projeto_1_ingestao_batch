# Ingestao de dados transacionais em batch via Airflow

## 💡1. Intuito do projeto: 

A ideia do projeto foi simular um cenários real que um engenheiro de dados tende a passar no dia-a-dia para transferir o máximo de conhecimento possível para os meus projetos práticos.

Esse projeto é uma arquitetura medallion de ingestão de dados em batch, que se trata de uma das coisas que um engenheiro de dados mais utiliza no seu dia-a-dia.

## 🧰 2. Tech Stack:

- **Linguagem:** Python
- **Processamento:** PySpark, Pandas
- **Orquestração:** Apache Airflow
- **Data Lake & Arquitetura:** Medallion Architecture, Delta Lake
- **Armazenamento:** Parquet
- **Geração de dados:** Mockaroo

## 🧠 3. Raciocínio:

1. Geração dos dados: 
O processo começou com a geração de dados fictícios utilizando o site _Mockaroo_. Foi criado um arquivo CSV com 1.000 linhas com três colunas principais: transaction_id, customer_id e amount. 

2. Organização e ingestão inicial:
Os arquivos CSV foram armazenados em uma pasta **source**, que serviu como fonte de dados. Em seguida, os dados foram ingeridos em dez partes distintas, cada uma com um **timestamp** associado, possibilitando um processamento dinâmico por meio do **Airflow**.

3. Arquitetura do Data Lake:
Criei um repositório que atuou como **Data Lake**, seguindo a **arquitetura medallion**, composta pelas camadas: Bronze e Silver. A ingestão ocorreu da seguinte forma: 
- **Source → Bronze**: os dados foram ingeridos em formato parquet. 
- **Bronze → Silver**: os dados foram transformados em uma **tabela Delta**, preparando-os para uma futura camada **Gold**.

Neste ponto, não foram aplicadas regras de negócio, mas em projetos reais costuma-se incluir processos como: Exclusão de dados nulos, validações via regex e padronizações diversas.

4. Considerações sobre formatos:
Algumas literaturas defendem que a camada **Bronze** deve manter os arquivos exatamente como na fonte original (por exemplo, CSV → CSV). Outras abordagens sugerem transformar os arquivos originais em uma **tabela Delta** com estratégia _append only_, deixando a etapa de processamento para a camada **Silver**.

Ambas as práticas possuem **vantagens e desvantagens** e a escolha depende da **metodologia** ou **requisitos do projeto**.
