# **Análise de Dados: Execuções da Pena de Morte no Texas (Desde 1976)**

##  **Visão Geral do Projeto**
Este projeto foi desenvolvido como parte dos meus estudos em SQL utilizando o **Google BigQuery**. O objetivo foi analisar o conjunto de dados históricos dos indivíduos executados no estado do Texas desde 1976, extraindo métricas demográficas, temporais e geográficas sobre a aplicação da pena de morte.

## Software Utilizados
* **Google BigQuery** (Ambiente Cloud Data Warehouse)
* **SQL Padrão (Standard SQL)**

## Consultas Desenvolvidas e Conceitos Aplicados

Durante o desenvolvimento deste projeto, apliquei conceitos práticos para responder às seguintes perguntas da base em analise:

### 1. Organização de Queries Complexas (CTEs)
Utilização de tabelas temporárias expressas para segmentar a análise em etapas limpas, facilitando a leitura do código e a reutilização de colunas calculadas.
* **Técnica aplicada:** Expressões de Tabela Comuns (`WITH`).

### 2. Ranqueamento de Dados (Window Functions)
Criação de rankings para identificar, por exemplo, os anos com maior pico de execuções ou os condados com maior incidência de casos, utilizando funções de janela.
* **Técnica aplicada:** Funções de ranqueamento sobre partições de dados.

### 3. Análise do Tempo de Espera (Gargalo Judicial)
Cálculo exato em dias/anos do intervalo entre a data da condenação e a data da execução. Criação de faixas críticas de tempo para identificar indivíduos que excederam o limite padrão de espera.
* **Técnicas aplicadas:** `DATE_DIFF` para cálculo de intervalos e `CASE WHEN` para a categorização lógica dos períodos.

### 4. Perfil Sociodemográfico e Métricas de Idade
* **Métricas Etárias:** Identificação da idade mínima, máxima e média (`MIN`, `MAX`, `AVG`).
* **Tendência Temporal:** Extração do ano a partir de um campo de data com `EXTRACT(YEAR FROM Execution_Date)`.
* **Perfil por Raça e Escolaridade:** Contagem por etnia e cálculo da média escolar dos indivíduos.
* **Análise Geográfica:** Volume de execuções distribuído por condado (County) e por distrito do Texas.

## 📈 **Consultas realizadas:**
-----ANALISE DE BASE DE DADOS À DETENTOS SUBMETIDOS AO CORREDOR DE MORTE NO TEXAS ENTRE 1976 A 2018
---Verificar a media de idade dos condenados ao corredor da morte no Texas
SELECT DISTINCT ROUND(AVG(Age), 2) AS idade_media
FROM
  `projeto-final-503519.execucoes_projeto.corredor_morte`  ----Desde o dia que se iniciou com processo de condenacoes à morte do texas, dentre as 553 analisadas a idade media dos detentos era de 39,47 de idade;

---Media escolar
SELECT DISTINCT ROUND(AVG(Highest_Education_Level), 2) AS escolaridade_media
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`
---idade maxima e minima dos detentos
SELECT DISTINCT MAX(Age) AS idade_maxima, MIN(Age) AS idade_minima
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`

---Quanto tempo de espera levou entre a sentença até a execução do detento?
SELECT DISTINCT
  Execution_number,
  CONCAT(First_name, " ", Last_name) AS nome_completo,
  Date_of_conviction,
  Execution_Date,
  DATE_DIFF(Execution_Date, Date_of_conviction, YEAR) AS anos_de_espera,
  CASE
    WHEN DATE_DIFF(Execution_Date, Date_of_conviction, YEAR) <= 10
      THEN "Dentro do Limite de espera"
    ELSE "Acima do Limite de espera"
    END AS periodo_espera
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`

---Agrupar os individuos por raça
SELECT DISTINCT race, COUNT(Execution_number) AS numero_detentos
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`
GROUP BY 1
ORDER BY
  2 DESC
    ---Desde os anos de 1976 até 2018 no texas, foram levados ao corredor da morte 245 individuos de raça branca, 201 de raça negra e 105 de raça Espânica. outros individuos foi em numero de 2, sendo que estes estavam em proporcionalidade entre asiaticos e outro (de raça não esclarecida)
    ----Execucoes por condado
    WITH numero_por_condado AS (
      SELECT DISTINCT County AS Condado, COUNT(Execution_number) AS Executions
      FROM `projeto-final-503519.execucoes_projeto.corredor_morte`
      GROUP BY 1
    )
SELECT *, DENSE_RANK() OVER (ORDER BY Executions DESC) AS Ranking
FROM numero_por_condado
ORDER BY 2 DESC

---Decricao de execucoes por distrito
SELECT DISTINCT
  COALESCE(district, "Desconhecido") AS district,
  COUNT(Execution_number) AS Executions
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`
GROUP BY 1
ORDER BY 2 DESC

----Execucoes por ano
SELECT DISTINCT
  EXTRACT(YEAR FROM Execution_Date) AS Year, COUNT(Execution_Number) AS contagem
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`
GROUP BY 1
ORDER BY 2 DESC


