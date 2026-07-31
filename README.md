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
-- =======================================================================================
-- ANÁLISE DE BASE DE DATA: DETENTOS SUBMETIDOS AO CORREDOR DA MORTE NO TEXAS (1976 - 2018)
-- =======================================================================================

-- 1. Média de idade dos condenados
-- Nota: Das 553 execuções analisadas, a idade média dos detentos foi de 39,47 anos.
SELECT 
  ROUND(AVG(Age), 2) AS idade_media 
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`;


-- 2. Média escolar (anos de estudo) dos detentos
SELECT 
  ROUND(AVG(Highest_Education_Level), 2) AS escolaridade_media 
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`;


-- 3. Idade máxima e mínima registrada entre os detentos
SELECT 
  MAX(Age) AS idade_maxima, 
  MIN(Age) AS idade_minima 
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`;


-- 4. Tempo de espera em anos entre a sentença de condenação e a execução
SELECT 
  Execution_number, 
  CONCAT(First_name, ' ', Last_name) AS nome_completo, 
  Date_of_conviction, 
  Execution_Date, 
  DATE_DIFF(Execution_Date, Date_of_conviction, YEAR) AS anos_de_espera, 
  CASE 
    WHEN DATE_DIFF(Execution_Date, Date_of_conviction, YEAR) <= 10 THEN 'Dentro do Limite de espera' 
    ELSE 'Acima do Limite de espera' 
  END AS periodo_espera  
FROM `projeto-final-503519.execucoes_projeto.corredor_morte`
ORDER BY anos_de_espera DESC;


-- 5. Volumetria de indivíduos agrupados por raça/etnia
-- Nota histórica: Distribuição de 245 brancos, 201 negros, 105 hispânicos e 2 de outras etnias.
SELECT 
  race, 
  COUNT(Execution_number) AS numero_detentos 
FROM `projeto-final-503519.execucoes_projeto.corredor_morte` 
GROUP BY race 
ORDER BY numero_detentos DESC;


-- 6. Ranking de execuções por condado utilizando CTE e Window Function (DENSE_RANK)
WITH numero_por_condado AS (
  SELECT 
    County AS Condado, 
    COUNT(Execution_number) AS Executions 
  FROM `projeto-final-503519.execucoes_projeto.corredor_morte` 
  GROUP BY County
)
SELECT 
  Condado,
  Executions,
  DENSE_RANK() OVER(ORDER BY Executions DESC) AS Ranking 
FROM numero_por_condado 
ORDER BY Executions DESC;


-- 7. Distribuição de execuções por distrito judicial (Tratamento de nulos com COALESCE)
SELECT 
  COALESCE(district, 'Desconhecido') AS distrito, 
  COUNT(Execution_number) AS Executions 
FROM `projeto-final-503519.execucoes_projeto.corredor_morte` 
GROUP BY district 
ORDER BY Executions DESC;


-- 8. Análise temporal: Volume de execuções por ano
SELECT 
  EXTRACT(YEAR FROM Execution_Date) AS Ano, 
  COUNT(Execution_Number) AS total_execucoes 
FROM `projeto-final-503519.execucoes_projeto.corredor_morte` 
GROUP BY Ano 
ORDER BY total_execucoes DESC;

