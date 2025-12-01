# 📄 Documentação de Participação do Projeto de Pipeline de Dados

**FACULDADE DONADUZZI**
**BIOPARK EDUCAÇÃO - 1º PERÍODO CD e Al**

Com o objetivo de documentar e esclarecer a elaboração do nosso projeto aplicado, desenvolvido para a disciplina de **Pipeline de dados**, apresentamos este detalhamento das contribuições. Este registro serve para garantir a transparência e o reconhecimento do esforço de toda a equipe no resultado final.

---

## 📅 Informações Essenciais

* **Trabalho:** Pipeline de dados
* **Orientador:** Wesley Antonio Santos de Andrade Sobreira
* **Data de Referência:** 27 de novembro de 2025
* **Local:** Toledo, Paraná

---

## 👥 Membros do Grupo

1.  **Braian Cauan Marqueto**
2.  **João Hélio dos Santos Glinski**
3.  **Luiz Henrique Zavatini Feltrin**
4.  **Marco Antonio Wandraski**

---




As seções a seguir especificam o papel de cada integrante, as responsabilidades principais e as etapas do projeto que foram executadas individualmente.

### 1. Braian Cauan Marqueto

| Categoria | Detalhes |
| :--- | :--- |
| **Função Principal** | **Especialista Postgres** |
| **Atividades Realizadas** | * Configuração de adaptação de **SQLite para Postgres**.<br>* Desenvolvimento de testes local e nuvem do carregamento dos dados.<br>* Funcionalidades de variáveis do Postgres. |
| **Arquivos Modificados** | `database.py` (modificado), `05_create_database_pg_schema.ipynb` (modificado), `06_load_to_postgres.ipynb` (modificado), `05_create_database_pg_schema_spark.ipynb` (modificado), `06_load_to_postgres_spark.ipynb` (modificado), `config.py` (modificado). |

### 2. João Hélio dos Santos Glinski

| Categoria | Detalhes |
| :--- | :--- |
| **Função Principal** | **Analista de Suporte e Estruturação Técnica** |
| **Atividades Realizadas** | * **Refatoração e migração dos scripts** de processamento de Pandas para **Apache Spark (PySpark)**.<br>* Idealizador e Editor do **vídeo de apresentação**.<br>* Documentação de participação do grupo.|
| **Arquivos Criados/Modificados** | **Fase 3:** `Spark_Pipeline.ipynb`<br>**Fase 4:** `participacao.md`<br>**Fase 4:** `Pipeline.mp4`

### 3. Luiz Henrique Zavatini Feltrin

| Categoria | Detalhes |
| :--- | :--- |
| **Função Principal** | **Engenheiro de Dados & Infraestrutura (DevOps & Orquestração)** |
| **Atividades Realizadas** | * **Exploração inicial de dados** e definição do dataset base (**Agricultura e Clima**).<br>* Estruturação da camada de persistência e scripts de carga para **PostgreSQL**.<br>* Configuração de ambiente e **containerização com Docker e Docker Compose**.<br>* Desenvolvimento e manutenção da **DAG de orquestração no Apache Airflow**.<br>* Criação e implementação de módulos de **utilitários de banco de dados (`db_utils`), validação de dados e monitoramento do pipeline**.<br>* Gerenciamento de credenciais e configurações de ambiente. |
| **Arquivos Relevantes** | **Fase 01 (Exploração):** `pipeline.ipynb` (Google Colab)<br>**Fase 03 (Postgres):** `database.py` (criado), `SQL_queries.ipynb` (criado), `06_load_to_postgres.ipynb` (criado), edições em arquivos de *layers* e *data quality*<br>**Fase 04 (Infra/Airflow):** `Dockerfile` (criado), `docker-compose.yaml` (editado), `monitoring.py` (criado), `validate.py` (criado), `db_utils.py` (criado), `pipeline_agricultura_dag.py` (editado)<br>**Fase 04 (Spark Jobs):** Edições em todos os *Spark Jobs* (`bronze.py`, `silver.py`, `gold.py`, `load_to_postgres.py`, `spark_session_manager.py`) |

### 4. Marco Antonio Wandraski

| Categoria | Detalhes |
| :--- | :--- |
| **Função Principal** | **Engenheiro de Dados Full Cycle & Documentação Técnica** |
| **Atividades Realizadas** | * Definição técnica e documentação da proposta inicial e requisitos.<br>* **Desenvolvimento integral do Pipeline ETL (Fase 02)** com ingestão e refinamento.<br>* Análise estatística, detecção e tratamento de **outliers/anomalias**.<br>* Fundamentação teórica e documentação da **Arquitetura Medallion** (Bronze, Silver, Gold). <br>* **Refatoração e migração dos scripts** de Pandas para **Apache Spark (PySpark)**.<br>* **Co-desenvolvimento e implementação da orquestração** do pipeline via **Apache Airflow**.<br>* Elaboração de documentação do projeto e guias de funcionamento. |
| **Arquivos Relevantes** | **Fase 02 (ETL Inicial):** Criação dos notebooks de layers (`#bronze_layer.ipynb`, `#silver_layer.ipynb`, `#gold_layer.ipynb`), `data_quality_report.ipynb` (criado), `load_to_database.ipynb` (criado)<br>**Fase 03 (Migração Spark):** Criação de todos os notebooks Spark (`00_setup_spark_parquet.ipynb` a `06_load_to_postgres_spark.ipynb`), `SQL_queries_spark.ipynb` (criado)<br>**Fase 04 (Documentação):** `Documentação-completa.md`<br>**Outros:** Link para Arquivos iniciais Airflow: `https://encurtador.com.br/gshn` |

---