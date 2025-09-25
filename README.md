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


Esta seção detalha os passos necessários para replicar a arquitetura de ingestão de dados em sua própria máquina. Certifique-se de ter todos os pré-requisitos instalados e configurados antes de começar.

A ideia é que qualquer pessoa consiga seguir o guia e replicar a sua arquitetura de ingestão de dados.

### 1. Pré-requisitos e Configuração Inicial:

Para rodar este projeto, você precisará de um ambiente com Python e o Apache Airflow já configurado. Se você ainda não tem o Airflow rodando, siga a documentação oficial para configurá-lo.

- Verifique se o Airflow está rodando: Certifique-se de que o seu ambiente do Airflow está acessível e operacional.

- Crie a estrutura de pastas do Data Lake: No diretório raiz do seu projeto, crie uma pasta que servirá como o seu Data Lake. Você pode chamá-la de `data_lake`. Dentro dela, crie as subpastas que representam as camadas da arquitetura medallion:

- data_lake/bronze
- data_lake/silver
- data_lake/source

### 2. Geração e Preparação dos Dados:

A sua arquitetura depende de dados transacionais. Siga os passos abaixo para gerar e preparar os arquivos de origem:

- Gerar os dados: Acesse o site [Mockaroo](https://www.mockaroo.com/) e crie um arquivo CSV com 1.000 linhas, usando as colunas transaction_id (Row Number), customer_id (Number) e amount (Number).

- Dividir os dados: Divida o arquivo CSV original em dez arquivos menores e realize a ingestão na pasta `source`. Nomeie-os de forma sequencial utilizando timestamp (por exemplo, transacoes_dia_2025-09-04.csv, transacoes_dia_2025-09-03.csv, transacoes_dia_2025-09-02 etc.). No meu projeto eu realizei essa ingestão no diretório `transaction_system`.

### 3. Criação do DAG e Lógica de Ingestão

Agora, você precisa criar o Directed Acyclic Graph (DAG) que orquestrará as etapas de ingestão.

- **Crie o arquivo do DAG:** Dentro da pasta `dags` do seu projeto Airflow, crie um arquivo Python, por exemplo, `dag_medallion.py`.

- **Escreva o código:** O código do DAG deve conter os `tasks` (tarefas) para cada etapa do seu projeto:
    
    - **Task 1: Source -> Bronze:** Uma tarefa que lê os arquivos CSV da pasta `source`, converte-os para o formato Parquet e os armazena na pasta `bronze`. 
    
    - **Task 2: Bronze -> Silver:** Uma tarefa que lê os arquivos Parquet da camada `bronze`, cria um Delta Table e armazena os dados na pasta `silver`. Esta é a etapa ideal para futuras transformações, mas neste projeto inicial, será apenas uma conversão de formato.
    

Ao executar o DAG, ele lerá os arquivos, processará-os e moverá os dados entre as camadas, simulando o fluxo de uma arquitetura de dados real.

É importante que as variáveis de ambientes estejam previamente configuradas no seu código, elas devem apontar para os seus respectivos diretórios no Data Lake: 

1. DATALAKE_PATH = `'/caminho/para/seu/projeto/datalake'`
2. SOURCE_PATH = `f"{DATALAKE_PATH}'/source/transaction_system/{arquivo}.csv'"`
3. BRONZE_PATH = `f"{DATALAKE_PATH}'/bronze/transaction_data'"`
4. SILVER_PATH = `f"{DATALAKE_PATH}'/silver/transaction_data'"`

## 5. Execução da DAG no Airflow

Com o código da DAG criado e o seu ambiente do Airflow configurado, a última etapa é orquestrar o processo. O Airflow vai detectar o seu novo arquivo de DAG e mostrá-lo na interface web, permitindo que você o execute.

### Passo a Passo

1. **Acessar a Interface do Airflow**: Abra seu navegador e navegue até a URL da sua interface web do Airflow. Geralmente, é `http://localhost:8080`.

2. **Verificar a DAG**: Na página principal, você deverá ver a sua DAG, nomeada de acordo com o `dag_id` definido no seu arquivo Python (por exemplo, `dag_medallion`). Por padrão, a DAG é criada desabilitada.

3. **Habilitar a DAG**: Clique no botão para habilitar a DAG (geralmente um botão de "liga/desliga" ou um slider). Isso fará com que o Airflow passe a monitorar a DAG e a execute de acordo com o seu `schedule_interval`.

4. **Executar a DAG Manualmente (Opcional)**: Para testar a pipeline imediatamente, você pode acionar a execução manualmente. Na interface da DAG, clique no botão de "play" e selecione "Trigger DAG". Isso iniciará uma nova execução imediatamente.

5. **Monitorar a Execução**: Após acionar a DAG, você pode acompanhar o progresso das tarefas clicando nela. A visualização **Graph View** ou **Gantt Chart** são excelentes para ver o status de cada tarefa (se está rodando, teve sucesso ou falhou).

Ao finalizar a execução com sucesso, você verá os resultados no seu **data lake**.

- A pasta `data_lake/bronze` conterá os arquivos transformados no formato **Delta** (ou Parquet, dependendo da sua implementação) para cada dia de processamento.

- A pasta `data_lake/silver` conterá a tabela **Delta** limpa e atualizada, pronta para ser consumida pela camada Gold.
