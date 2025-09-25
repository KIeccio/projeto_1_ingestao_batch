# Ingestao de dados transacionais em batch via Airflow

Com o objetivo de demonstrar uma arquitetura medalhão (source → bronze → silver), este projeto realiza a coleta de dados transacionais diários. O pipeline completo é executado localmente, com a orquestração do Apache Airflow e o processamento de dados gerenciado pelo Apache Spark e Delta Lake.

>>OBS: foi escolhido PySpark por questões académicas. Em um contexto real, a massa de dados precisaria ser maior para justificar a necessidade da tecnologia. Mas, para utilização junto ao Delta, o PySpark foi a escolha mais simples.

## 📂 1. Estrutura do Projeto

```
project-root/
│
├── mock_data/
│   └── MOCK_DATA.csv              # Base de dados completa usada como origem
│
├── datalake/                      # Pasta externa contendo os dados organizados por camadas. Você precisa criar essa pasta e apontar na variável "DATALAKE_PATH" dentro dos arquivos
│   ├── source/
│   │   └── transaction_system/    # Arquivos CSV diários de entrada
│   ├── bronze/
│   │   └── transaction_data/      # Dados no formato Delta Lake (raw zone)
│   └── silver/
│       └── transaction_data/      # Dados tratados e deduplicados no formato Delta Lake
│
├── dags/
│   └── ingest_transaction_data.py # DAG do Airflow responsável pela ingestão diária. Você pode utilizar o comando 'cp' para copiar essa DAG para sua pasta de DAGs do Airflow.
│
├── scripts/
│   ├── mock_data_spliter.py       # Divide o MOCK_DATA.csv em 10 partes com datas diferentes
│   ├── ingest_source_to_bronze.py # Realiza ingestão dos arquivos CSV para camada bronze
│   └── ingest_bronze_to_silver.py # Atualiza a camada silver com dados incrementais da bronze
│
├── requirements.txt               # Bibliotecas necessárias para execução
└── README.md                      # Este arquivo
```

> ⚠️ Certifique-se de criar manualmente a pasta `datalake/` na raiz do projeto antes da execução e apontar o caminho absoluto nas respectivas variáveis.


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

## 👩🏽‍💻4. Como reproduzir cada etapa passo a passo:

### 1. Instale as dependências
```bash
pip install -r requirements.txt
```

### 2. Gere os arquivos simulados
```bash
python scripts/mock_data_spliter.py
```

### 3. Inicie o Airflow standalone
```bash
airflow standalone
```

### 4. Ative a DAG no painel do Airflow
Acesse o Airflow em [http://localhost:8080](http://localhost:8080) e ative a DAG `INGEST_TRANSACTION_DATA`.

> A DAG está agendada para rodar todos os dias às 03:00 (`0 3 * * *`), com suporte a **catchup** de dias anteriores. A simulação inclui falhas nos dias em que não existem arquivos.

