# Data Intelligence no Estudo de Perfil Epidemiológico do Diabetes Mellitus — PNS 2013

Análise da prevalência e subnotificação de Diabetes Mellitus (DM) através da 
subamostra laboratorial da Pesquisa Nacional de Saúde (PNS 2013), evidenciando 
as lacunas de diagnóstico no sistema de saúde brasileiro.

## Estrutura do projeto

├── data/
│   ├── raw/                        # Microdados brutos PNS 2013 (não versionados)
│   └── processed/                  # CSVs do star schema
│       ├── Fato_Individual_Lab.csv
│       ├── Dim_Sociodemografia.csv
│       ├── Dim_Geografia.csv
│       ├── Dim_Perfil_Clinico.csv
│       ├── Dim_Complicacoes_Diabetes.csv
│       └── Dim_Estilo_Vida_Acesso.csv
├── notebooks/
│   └── limpeza_PNS_2013.ipynb      # ETL completo
├── dashboard/
│   ├── DM_Insight_PNS2013.pbix     # Dashboard Power BI
│   └── assets/                     # Imagens de fundo utilizadas no dashboard
├── .gitignore
└── README.md

## Fonte de dados

Pesquisa Nacional de Saúde 2013 — Subamostra de Exames Laboratoriais
Instituto Brasileiro de Geografia e Estatística (IBGE)

> Os resultados são expressos em proporções ponderadas. A subamostra 
> laboratorial da PNS 2013 não permite inferência de totais populacionais 
> absolutos (IBGE, Nota Técnica, 2020).

## Stack

- **Python 3** — pandas, numpy, rich (ETL e modelagem dimensional)
- **Power BI Desktop** — visualização, DAX e modelagem star schema

## Modelagem dimensional

Star Schema com 1 tabela fato e 5 dimensões.
O Peso Laboratorial (Peso_Lab) é incorporado como fator de expansão 
normalizado para cálculo de prevalências e proporções ponderadas,
seguindo as diretrizes metodológicas do IBGE para análise de microdados da PNS.

## Critério diagnóstico de DM

Classificação baseada em critério combinado (ADA, 2013):
- HbA1c ≥ 6,5%
- Autodeclaração de diagnóstico médico prévio
- Uso de medicamento para diabetes

## Principais achados

- Prevalência clínica de DM na subamostra: **9,81%**
- Taxa de subnotificação: **26,12%**
- Prevalência de pré-diabetes: **13,81%**

> Indivíduos subnotificados (HbA1c ≥ 6,5% sem diagnóstico prévio) não foram 
> submetidos às perguntas sobre complicações crônicas pelo desenho condicional 
> do questionário PNS 2013. Sua severidade real é desconhecida e possivelmente 
> subestimada.

## Licença

Dados públicos IBGE — uso livre para fins acadêmicos e de pesquisa.
