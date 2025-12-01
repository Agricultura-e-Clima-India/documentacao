# Documentação: Pipeline de Dados para Sistema de Recomendação Agrícola

---

## 1. Visão Geral do Projeto

Este documento consolida a documentação completa da organização, detalhando a arquitetura, os componentes tecnológicos e os guias de execução para cada um dos três repositórios. O objetivo do projeto é a construção de um **Sistema de Recomendação Agrícola Inteligente**, um pipeline de dados ponta a ponta que processa, enriquece e analisa dados agrícolas da Índia para gerar insights e otimizar a tomada de decisão no campo.

O projeto evolui através de três implementações distintas:

1.  **`pipeline`**: Uma abordagem inicial utilizando **Pandas** para o ETL, ideal para datasets menores e prova de conceito.
2.  **`pipeline-spark`**: Uma evolução para **Apache Spark**, permitindo o processamento de dados em larga escala de forma distribuída e performática.
3.  **`airflow-pipeline-spark`**: A versão final e mais robusta, onde o pipeline Spark é totalmente orquestrado e automatizado com **Apache Airflow**, utilizando Docker para criar um ambiente de produção autocontido.

> O problema central que o projeto aborda é a dependência de métodos agrícolas tradicionais, que são vulneráveis às mudanças climáticas e ineficientes no uso de recursos. Ao integrar dados de produção, clima e solo, o sistema visa fornecer recomendações que aumentem a produtividade, promovam a sustentabilidade e reduzam os riscos financeiros para os agricultores.

---

## 2. Guias de Instalação e Configuração de Pré-requisitos

Para executar os diferentes estágios do projeto, várias ferramentas são necessárias. Esta seção serve como um guia de instalação didático e centralizado.

### 2.1. WSL 2: O Subsistema do Windows para Linux

O WSL 2 permite executar um ambiente Linux completo diretamente no Windows, sendo a base recomendada para desenvolvimento com Docker.

1.  **Instalação Automática:**
    *   Abra o **PowerShell** ou o **Prompt de Comando do Windows** como **Administrador**.
    *   Execute o seguinte comando. Ele cuidará de tudo: habilitará os recursos necessários, baixará o kernel Linux e instalará a distribuição Ubuntu por padrão.
        ```powershell
        wsl --install
        ```
2.  **Reinicialização:**
    *   Reinicie o seu computador quando solicitado para concluir a instalação.
3.  **Verificação:**
    *   Após reiniciar, abra o menu Iniciar e procure por "Ubuntu". Ao abri-lo, um terminal Linux será iniciado. Na primeira vez, pode ser necessário criar um nome de usuário e senha para o seu ambiente Linux.

### 2.2. Docker Desktop com Integração WSL 2

O Docker é a ferramenta essencial para executar o pipeline orquestrado com Airflow, pois gerencia os contêineres de cada serviço.

1.  **Download:**
    *   Baixe o instalador oficial a partir do site: [**Docker Desktop for Windows**](https://www.docker.com/products/docker-desktop).
2.  **Instalação:**
    *   Execute o arquivo baixado. Durante o processo de instalação, a opção **"Use WSL 2 instead of Hyper-V"** deve estar marcada. Esta é a configuração padrão e recomendada.
3.  **Verificar a Integração:**
    *   Após a instalação, inicie o Docker Desktop.
    *   Vá para **Settings (ícone de engrenagem) > Resources > WSL Integration**.
    *   Na lista, certifique-se de que a chave de integração para a sua distribuição Linux (ex: `Ubuntu`) esteja **ativada**. Isso permite que o Docker seja usado de forma nativa dentro do seu terminal WSL.

### 2.3. Python e Ambientes Virtuais

É uma boa prática isolar as dependências de cada projeto usando ambientes virtuais.

1.  **Pré-requisito:**
    *   Tenha o Python (versão 3.10 ou 3.11 recomendada) instalado no seu sistema.
2.  **Criação e Ativação:**
    *   Navegue até a pasta raiz de um dos projetos (`pipeline-main` ou `pipeline-spark-main`) pelo terminal.
    *   Execute os comandos abaixo para criar e ativar o ambiente:
        ```bash
        # 1. Criar o ambiente virtual na pasta .venv
        python -m venv .venv

        # 2. Ativar o ambiente
        # No Windows (PowerShell)
        .\.venv\Scripts\Activate

        # No Linux/macOS ou no terminal WSL
        source .venv/bin/activate
        ```
    *   Quando o ambiente está ativo, o nome `(.venv)` aparecerá no início do seu prompt de terminal.

### 2.4. Java (Apenas para `pipeline-spark-main` em modo local)

O Apache Spark é escrito em Scala e executado na JVM (Java Virtual Machine). Portanto, o Java é um pré-requisito para rodar PySpark localmente.

1.  **Instalação:**
    *   Recomenda-se o OpenJDK (versão 8 ou 11). Você pode baixá-lo de fontes como o [Adoptium](https://adoptium.net/).
2.  **Configuração da Variável de Ambiente `JAVA_HOME`:**
    *   Após a instalação, você precisa configurar a variável de ambiente `JAVA_HOME` para apontar para o diretório de instalação do Java.
3.  **Verificação:**
    *   Abra um novo terminal e execute:
        ```bash
        java -version
        ```
    *   Se a instalação foi bem-sucedida, você verá a versão do Java instalada.

### 2.5. Credenciais da API do Kaggle

Para baixar os datasets de origem, é necessário ter as credenciais da API do Kaggle.

1.  **Gerar o Token:**
    *   Faça login no site do [Kaggle](https://www.kaggle.com/).
    *   Acesse as configurações da sua conta: `https://www.kaggle.com/settings`.
    *   Role a página até a seção **API** e clique em **"Create New Token"**.
    *   Isso fará o download de um arquivo chamado `kaggle.json`.
2.  **Posicionar o Arquivo:**
    *   Você precisa mover esse arquivo para um local específico. No seu terminal (Linux/macOS/WSL), execute:
        ```bash
        # Cria o diretório .kaggle se ele não existir
        mkdir -p ~/.kaggle

        # Move o arquivo baixado para o diretório correto
        # (Ajuste o caminho para a sua pasta de Downloads)
        mv ~/Downloads/kaggle.json ~/.kaggle/kaggle.json

        # Define as permissões corretas para o arquivo
        chmod 600 ~/.kaggle/kaggle.json
        ```
    *   Para o projeto com Airflow, o arquivo `kaggle.json` deve ser colocado na pasta `config/` do projeto.

---

## 3. Repositório 1: `pipeline-main` (Implementação com Pandas)

Esta é a primeira versão do pipeline, servindo como base e prova de conceito. Utiliza Pandas para todas as etapas de ETL e carrega os dados em bancos de dados SQLite e PostgreSQL.

### 3.1. Arquitetura e Estrutura de Dados

O pipeline segue a arquitetura Medallion, processando os dados em três camadas.

| Camada | Arquivo de Saída | Descrição das Transformações |
| :--- | :--- | :--- |
| **Bronze** | `data/bronze/dados_brutos.csv` | **Dados Brutos:** Apenas padroniza os nomes das colunas para o formato `snake_case` (minúsculas com underscores). |
| **Silver** | `data/silver/dados_limpos.csv` | **Dados Limpos:** Aplica remoção de duplicatas, tratamento de nulos, padronização de valores textuais (`UPPER()`), cria uma chave `state_district`, filtra culturas de interesse (`MAIZE`, `RICE`, `WHEAT`, `BARLEY`) e remove dados inválidos (`area` ou `yield` zerados). |
| **Gold** | `data/gold/*.csv` | **Dados Agregados:** Gera 7 tabelas de agregação prontas para BI, como `producao_anual_cultura.csv` e `desempenho_regiao_cultura.csv`. |

### 3.2. Modelagem do Banco de Dados

Os dados são carregados em dois bancos de dados com modelos diferentes:

-   **SQLite (`pipeline_agricultura.db`):** Implementa um **Schema Star (Estrela)**, ideal para consultas analíticas. Contém uma tabela `fato_producao` central e tabelas de dimensão (`dim_regiao`, `dim_cultura`, `dim_temporada`).
-   **PostgreSQL:** Carrega as 7 tabelas agregadas da camada Gold em um esquema relacional simples para acesso direto.

### 3.3. Guia de Execução

1.  **Configurar Variáveis de Ambiente (para PostgreSQL):**
    *   Na raiz do projeto, crie um arquivo `.env` a partir do modelo `.env.example`.
    *   Preencha as credenciais do seu banco de dados PostgreSQL.
2.  **Criar e Ativar Ambiente Virtual:**
    *   Siga as instruções da seção **2.3**.
3.  **Instalar Dependências:**
    ```bash
    pip install -r requirements.txt
    ```
4.  **Executar os Notebooks Jupyter:**
    *   Execute os notebooks na seguinte ordem para garantir o fluxo correto dos dados:
        1.  `notebooks/01_bronze_layer.ipynb`
        2.  `notebooks/02_silver_layer.ipynb`
        3.  `notebooks/03_gold_layer.ipynb`
        4.  `notebooks/04_load_to_sqlite.ipynb`
        5.  `notebooks/05_create_database_pg_schema.ipynb`
        6.  `notebooks/06_load_to_postgres.ipynb`
        7.  `notebooks/analysis/data_quality_report.ipynb` (Opcional)
        8.  `notebooks/analysis/SQL_queries.ipynb` (Para análises)

---

## 4. Repositório 2: `pipeline-spark` (Implementação com Spark)

Esta versão migra a lógica de processamento de Pandas para Apache Spark, permitindo escalabilidade e performance. O formato de armazenamento padrão é o **Parquet**, que é otimizado para cargas de trabalho analíticas.

### 4.1. Arquitetura e Estrutura de Dados

| Camada | Arquivo de Saída | Descrição das Transformações |
| :--- | :--- | :--- |
| **Bronze** | `data/bronze/dados_brutos.parquet` | Une os datasets de produção, chuva e temperatura. Renomeia colunas climáticas e adiciona uma coluna `data_ingestao`. |
| **Silver** | `data/silver/dados_limpos.parquet` | Normaliza nomes, imputa valores numéricos com a mediana, preenche textos com "Unknown", remove outliers (usando IQR), padroniza texto e filtra as culturas de interesse. |
| **Gold** | `data/gold/*.parquet` | Gera agregações de negócios similares à versão Pandas, mas salvas como arquivos Parquet individuais. |

### 4.2. Guia de Execução (Duas Opções)

#### Opção A: Execução Local

1.  **Pré-requisitos:**
    *   Python 3.11+ (com ambiente virtual ativado - Seção 2.3).
    *   Java 8+ (JDK) instalado e configurado (Seção 2.4).
    *   Credenciais do Kaggle configuradas (Seção 2.5).
2.  **Instalar Dependências:**
    ```bash
    pip install -r requirements.txt
    ```
3.  **Executar os Notebooks:**
    *   Comece pelo notebook `notebooks/00_setup_spark_parquet.ipynb`. Ele serve como um assistente para validar seu ambiente (checa Java, cria pastas) e permite criar o `kaggle.json` se necessário.
    *   Prossiga com os notebooks de `01` a `06` na ordem numérica.

#### Opção B: Execução com Docker (Recomendado)

Este método é mais simples, pois não requer a instalação local de Java ou Spark.

1.  **Pré-requisitos:**
    *   Docker Desktop instalado e em execução (Seção 2.2).
2.  **Iniciar o Contêiner Jupyter/PySpark:**
    *   Abra um terminal (PowerShell no Windows ou o terminal do WSL) na raiz do projeto.
    *   Execute o comando abaixo. Ele baixa a imagem `jupyter/pyspark-notebook`, inicia um contêiner e mapeia a pasta atual do seu projeto para o diretório `/home/jovyan/work` dentro do contêiner.
        ```bash
        # Para Windows (PowerShell)
        docker run --rm -it -p 8888:8888 -v ${PWD}:/home/jovyan/work jupyter/pyspark-notebook:latest

        # Para Linux/macOS ou WSL
        docker run --rm -it -p 8888:8888 -v $(pwd):/home/jovyan/work jupyter/pyspark-notebook:latest
        ```
3.  **Acessar o Jupyter:**
    *   O terminal exibirá uma URL contendo um token de acesso (ex: `http://127.0.0.1:8888/?token=...`). Copie e cole essa URL no seu navegador.
4.  **Executar os Notebooks:**
    *   Dentro do ambiente Jupyter, navegue até a pasta `work/` (que é o seu projeto) e execute os notebooks na ordem numérica, começando pelo `00_setup_spark_parquet.ipynb`.

---

## 5. Repositório 3: `airflow-pipeline-spark` (Orquestração com Airflow)

Esta é a implementação final, que encapsula todo o pipeline Spark em um ambiente de produção automatizado com Apache Airflow. Todos os serviços são gerenciados por Docker Compose.

### 5.1. Stack de Tecnologia e Arquitetura

-   **Orquestração:** Apache Airflow (com CeleryExecutor para tarefas assíncronas).
-   **Processamento de Dados:** Apache Spark.
-   **Formato de Tabela:** **Delta Lake**, que adiciona uma camada de confiabilidade sobre o Parquet, trazendo transações ACID, versionamento de dados e outras funcionalidades.
-   **Data Warehouse:** PostgreSQL.
-   **Contêineres:** Docker e Docker Compose.
-   **Validação de Dados:** Scripts de validação inspirados em Great Expectations são executados após as camadas Silver e Gold.

O fluxo de dados é o mesmo da arquitetura Medallion, mas cada etapa (Bronze, Silver, Gold, Validação, Carga) é uma **tarefa** dentro de um **DAG (Directed Acyclic Graph)** no Airflow.

### 5.2. Estrutura do Projeto

-   **`docker-compose.yaml`**: Arquivo central que define e orquestra todos os serviços: `airflow-webserver`, `airflow-scheduler`, `airflow-worker`, `postgres` e `redis`.
-   **`Dockerfile`**: Define a imagem customizada do Airflow, instalando Java e as dependências Python do `requirements.txt`.
-   **`dags/`**: Contém o `pipeline_agricultura_dag.py`, que define a sequência e as dependências das tarefas do pipeline.
-   **`spark_jobs/`**: Scripts Python modulares que contêm a lógica de processamento Spark para cada camada. São chamados pelos operadores do Airflow.
-   **`config/`**: Arquivos de configuração, como o `airflow.cfg` e o `kaggle.json`.
-   **`db_setup/`**: Scripts para criar o banco de dados e o esquema de tabelas no PostgreSQL antes da carga de dados.

### 5.3. Guia de Execução (Ambiente WSL + Docker)

1.  **Pré-requisitos:**
    *   WSL 2 e Docker Desktop devidamente instalados e integrados (Seções 2.1 e 2.2).
2.  **Configuração:**
    *   Clone o repositório para dentro do seu ambiente WSL.
    *   Navegue até a pasta raiz do projeto no terminal WSL.
    *   Renomeie o arquivo `.env~` para `.env`. Este arquivo pode ser usado para definir o `AIRFLOW_UID` e evitar problemas de permissão de arquivos entre o contêiner e o sistema de arquivos do host.
    *   Coloque seu arquivo `kaggle.json` (Seção 2.5) dentro da pasta `config/`.
3.  **Iniciar os Serviços:**
    *   No terminal WSL, na raiz do projeto, execute o comando:
        ```bash
        docker-compose up --build
        ```
    *   A flag `--build` força a construção da imagem customizada do Airflow na primeira vez. O processo pode levar alguns minutos. Você verá os logs de todos os serviços sendo exibidos no terminal.
4.  **Acessar a Interface do Airflow:**
    *   Após a inicialização dos serviços, abra seu navegador e acesse: **`http://localhost:8080`**.
    *   Faça login com as credenciais padrão: `airflow` / `airflow`.
5.  **Executar o Pipeline (DAG):**
    *   Na lista de DAGs, localize o `pipeline_agricultura`.
    *   Ative o DAG usando o botão de toggle à esquerda do nome.
    *   Para uma execução imediata, clique no nome do DAG para abrir a visualização de grade (Grid View) e, em seguida, clique no botão "Play" (▶️ `Trigger DAG w/ config`) no canto superior direito.
    *   Você poderá acompanhar o progresso de cada tarefa (Bronze, Silver, Gold, etc.) em tempo real pela interface.

### 5.4. Fluxo de Tarefas do DAG

O DAG `pipeline_agricultura` define a seguinte sequência de tarefas:

1.  **`ensure_database_exists`**: Verifica se o banco de dados `pipeline_db` existe no PostgreSQL. Se não existir, ele é criado.
2.  **`recreate_schema_tables`**: Cria (ou recria) as tabelas necessárias no PostgreSQL, incluindo as tabelas do esquema estrela e as tabelas agregadas da camada Gold.
3.  **`run_spark_bronze`**: Executa o script `spark_jobs/bronze.py` para processar a camada Bronze.
4.  **`run_spark_silver`**: Executa o script `spark_jobs/silver.py` para processar a camada Silver.
5.  **`run_validation_silver`**: Executa o script `spark_jobs/validate.py` para validar a qualidade dos dados da camada Silver.
6.  **`run_spark_gold_bi`**: Executa o script `spark_jobs/gold.py` para processar a camada Gold (agregações de BI e Feature Store para ML).
7.  **`run_validation_gold_fs`**: Valida os dados da camada Gold e da Feature Store.
8.  **`run_spark_load_postgres`**: Executa o script `spark_jobs/load_to_postgres.py` para carregar os dados validados no PostgreSQL.

As dependências entre as tarefas garantem que cada etapa só seja executada após a conclusão bem-sucedida da anterior.

### 5.5. Configuração do Banco de Dados PostgreSQL

O projeto utiliza o PostgreSQL de duas formas:

1.  **Backend do Airflow**: O serviço `postgres` no `docker-compose.yaml` serve como o banco de dados de metadados do Airflow. Esse banco é criado automaticamente e se chama `airflow`.
2.  **Data Warehouse**: Um segundo banco, chamado `pipeline_db`, é criado programaticamente pela tarefa `ensure_database_exists` e é usado para armazenar os dados processados.

As configurações de conexão estão centralizadas no arquivo `db_setup/config.py`. Por padrão, o projeto utiliza as seguintes credenciais:

-   **Host:** `postgres` (nome do serviço no Docker Compose)
-   **Porta:** `5432`
-   **Banco de Dados:** `pipeline_db`
-   **Usuário:** `airflow`
-   **Senha:** `airflow`

Essas configurações podem ser sobrescritas através de variáveis de ambiente (`DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASS`) se necessário.

### 5.6. Esquema do Banco de Dados (Data Warehouse)

O Data Warehouse PostgreSQL implementa um **esquema estrela** para otimizar consultas analíticas. A estrutura é a seguinte:

#### Tabela Fato

-   **`fato_producao`**: Tabela central que contém as métricas de produção e chaves estrangeiras para as dimensões.
    -   Colunas: `id_fato`, `id_regiao`, `id_cultura`, `id_data`, `id_estacao`, `production`, `yield`, `area`, `rainfall_annual`, `temperature_annual`.

#### Tabelas de Dimensão

-   **`dim_regiao`**: Detalhes geográficos.
    -   Colunas: `id_regiao`, `state`, `district`, `state_district`.
-   **`dim_cultura`**: Detalhes sobre as culturas.
    -   Colunas: `id_cultura`, `crop`.
-   **`dim_data`**: Detalhes temporais (ano).
    -   Colunas: `id_data`, `crop_year`.
-   **`dim_estacao`**: Detalhes sobre as estações do ano.
    -   Colunas: `id_estacao`, `season`.

#### Tabelas Agregadas (Camada Gold)

Além do esquema estrela, as sete tabelas agregadas da camada Gold também são carregadas no PostgreSQL para acesso direto por ferramentas de BI:

1.  `producao_anual_cultura`
2.  `tendencia_anual_rendimento`
3.  `desempenho_regiao_cultura`
4.  `volatilidade_rendimento_regiao`
5.  `benchmark_regional_rendimento`
6.  `perfil_climatico_regiao_cultura`
7.  `analise_sazonal_clima`

---

## 6. Descrição dos Dados Utilizados

O projeto integra três datasets distintos, cada um fornecendo uma dimensão específica do ecossistema agrícola:

### 6.1. Dataset Principal: Estatísticas de Produção Agrícola (Crop Production Statistics)

Este é o dataset central do projeto, contendo o histórico de produção agrícola da Índia ao longo de mais de três décadas.

-   **Registros:** Aproximadamente 345 mil linhas.
-   **Tamanho:** 21.14 MB.
-   **Estrutura:** 8 colunas (4 String, 2 Integer, 2 Decimal).
-   **Variáveis-chave:**
    -   `STATE` e `DISTRICT`: Localização geográfica.
    -   `CROP_YEAR` e `SEASON`: Dimensão temporal.
    -   `AREA` e `PRODUCTION`: Métricas de cultivo.
    -   **`YIELD`** (Rendimento): A métrica mais importante, calculada como produção em toneladas por hectare. Serve como o principal indicador de eficiência agrícola.

### 6.2. Dataset de Enriquecimento: Dados Climáticos e Agrícolas da Índia (1961-2018)

Este dataset complementa o principal, adicionando variáveis climáticas contextuais.

-   **Arquivos:** 51 arquivos CSV.
-   **Tamanho:** 1.25 MB.
-   **Estrutura:** 710 colunas no total, distribuídas entre os arquivos.
-   **Finalidade:** Enriquecer o dataset principal com dados de precipitação (RAINFALL) e temperatura média anual/bimestral, permitindo análises de correlação entre clima e produção.

### 6.3. Dataset de Contexto Macroeconômico: Dados de Temperatura da Superfície Terrestre (Climate Change: Earth Surface Temperature Data)

Fornece uma visão de longo prazo das tendências climáticas.

-   **Período:** Desde 1750 até o presente.
-   **Tamanho total:** 600.63 MB.
-   **Estrutura:** 32 colunas (10 Decimal, 9 String, 6 Uuid, 7 Other).
-   **Variáveis-chave:**
    -   `dt`: Data do registro.
    -   `LandAverageTemperature`, `LandMaxTemperature`, `LandMinTemperature`: Métricas de temperatura.
-   **Finalidade:** Permitir a análise do impacto das mudanças climáticas de longo prazo na agricultura local.

---

## 7. Análises e Consultas Disponíveis

Após o processamento e carga dos dados, diversas análises estão disponíveis através de consultas SQL. Os notebooks `SQL_queries.ipynb` (nos repositórios `pipeline-main` e `pipeline-spark-main`) contêm 12 consultas abrangentes que exploram a modelagem dimensional.

Exemplos de análises incluídas:

| Tipo de Análise | Exemplo de Consulta |
| :--- | :--- |
| **Benchmark** | Top 5 Regiões com Rendimento Acima da Média Nacional (RICE) |
| **Risco** | Top 5 Regiões Mais/Menos Estáveis por Volatilidade (MAIZE/WHEAT) |
| **Fatores Ideais** | Perfil Climático da Região Mais Produtiva (RICE) |
| **Tendência** | Crescimento Anual do Rendimento (WHEAT) |
| **Modelagem** | Top 10 Regiões por Produção Total (Usando JOINs com Dimensões) |

---

## 8. Qualidade dos Dados

O projeto implementa validações rigorosas de qualidade de dados, especialmente na versão com Airflow. Após o processamento da camada Silver, um relatório de qualidade é gerado, avaliando:

-   **Completude:** Percentual de valores não nulos em colunas críticas.
-   **Unicidade:** Verificação de duplicatas.
-   **Validade:** Verificação de que valores numéricos (como `Production` e `Yield`) não são negativos.

No repositório `pipeline-main`, o relatório de qualidade gerado mostra:

-   **Completude Geral:** 100.00%
-   **Unicidade:** 100.00% (0 duplicatas encontradas)
-   **Score Geral:** 100.00%
-   **Classificação:** EXCELENTE

---

## 9. Troubleshooting e Dicas

### 9.1. Problemas Comuns no `pipeline-spark`

**Erro: `JAVA_HOME is not set`**

-   **Causa:** O Spark não consegue encontrar a instalação do Java.
-   **Solução:** Certifique-se de que o Java está instalado e que a variável de ambiente `JAVA_HOME` está configurada corretamente (Seção 2.4).

**Erro: `ModuleNotFoundError: No module named 'pyspark'`**

-   **Causa:** As dependências não foram instaladas no ambiente virtual.
-   **Solução:** Ative o ambiente virtual e execute `pip install -r requirements.txt`.

### 9.2. Problemas Comuns no `airflow-pipeline-spark-main`

**Erro: `docker-compose: command not found`**

-   **Causa:** O Docker Compose não está instalado ou não está no PATH.
-   **Solução:** O Docker Desktop para Windows já inclui o Docker Compose. Certifique-se de que o Docker Desktop está em execução e que você está usando o terminal WSL com a integração ativada.

**Erro: `Bind for 0.0.0.0:8080 failed: port is already allocated`**

-   **Causa:** A porta 8080 já está sendo usada por outro serviço.
-   **Solução:** Pare o serviço que está usando a porta ou altere a porta do Airflow no `docker-compose.yaml` (ex: `"8081:8080"`).

**Erro: `Permission denied` ao executar `docker-compose up`**

-   **Causa:** Problemas de permissão de arquivos entre o host e os contêineres.
-   **Solução:** Certifique-se de que o arquivo `.env` contém a linha `AIRFLOW_UID=1000` (ou o UID do seu usuário no WSL, que você pode obter com o comando `id -u`).

**O DAG não aparece na interface do Airflow**

-   **Causa:** Erro de sintaxe no arquivo `pipeline_agricultura_dag.py` ou dependências faltando.
-   **Solução:** Verifique os logs do `airflow-scheduler` executando `docker-compose logs airflow-scheduler` para identificar o erro.

### 9.3. Dicas Gerais

-   **Reprocessamento:** Se você precisar reprocessar os dados, basta executar novamente os notebooks (para `pipeline-main` e `pipeline-spark-main`) ou acionar o DAG manualmente no Airflow. As escritas são feitas em modo `overwrite` por padrão.
-   **Limpeza de Dados:** Se você quiser limpar os dados processados, basta deletar as pastas `data/bronze/`, `data/silver/` e `data/gold/` dentro de cada projeto.
-   **Parar os Serviços do Airflow:** Para parar todos os contêineres, pressione `Ctrl+C` no terminal onde o `docker-compose up` está rodando, ou execute `docker-compose down` em outro terminal na pasta do projeto.

---

## 10. Melhorias Futuras e Otimizações

Durante a análise do projeto, foram identificadas oportunidades de otimização, especialmente para a versão com Airflow:

### 10.1. Problema Principal: Execução do Spark em Modo Local

Atualmente, os trabalhos Spark são executados em modo `local` dentro do worker do Airflow. Isso limita severamente a performance e não utiliza o poder de processamento distribuído do Spark.

### 10.2. Recomendação de Alto Impacto

1.  **Desacoplar o Spark do Airflow:** Migrar do `PythonOperator` para o `SparkSubmitOperator` no Airflow.
2.  **Utilizar um Cluster Spark:** Configurar um cluster Spark dedicado (Standalone, EMR, Databricks) e submeter os trabalhos a ele. Isso permitiria o processamento verdadeiramente distribuído e escalável.

### 10.3. Otimizações Secundárias

-   **Otimizar Código Spark:**
    -   Evitar `inferSchema` ao ler CSVs, definindo o schema explicitamente.
    -   Reparticionar dados para combater o problema de "arquivos pequenos" (small files problem).
    -   Refatorar lógicas de agregação e carga para reduzir shuffles.
-   **Otimizar Delta Lake:**
    -   Executar o comando `OPTIMIZE` periodicamente para compactar arquivos Delta.
    -   Usar `Z-Ordering` para otimizar consultas em colunas específicas.

---