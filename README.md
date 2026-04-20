# Data Intelligence no Estudo de Perfil Epidemiológico do Diabetes Mellitus
### Análise da Prevalência e Subnotificação de DM · Subamostra Laboratorial da PNS 2013

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white"/>
  <img src="https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white"/>
  <img src="https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black"/>
  <img src="https://img.shields.io/badge/Star_Schema-Dimensional-7F77DD?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Fonte-IBGE_PNS_2013-085041?style=for-the-badge"/>
</p>

---

## 📌 Sobre o Projeto

A crescente prevalência do **Diabetes Mellitus (DM)** consolidou-se como um dos maiores desafios da saúde pública brasileira. Entre 2014 e 2024, o Brasil registrou **1.476.002 internações hospitalares** vinculadas à doença, com custo médio de **R$ 832,93 por internação**, valor que sobe para **~R$ 1.250,00** na faixa de 20 a 39 anos, evidenciando o impacto severo sobre a população economicamente ativa *(Bezerra et al., 2025)*.

Este projeto aplica técnicas de **Data Intelligence e Business Intelligence** sobre a subamostra laboratorial da **Pesquisa Nacional de Saúde (PNS) 2013**, cruzando dados clínicos reais (HbA1c, pressão arterial, antropometria) com perfis sociodemográficos para:

- 📊 **Quantificar** a prevalência ponderada de DM na população adulta brasileira
- 🔍 **Evidenciar** a taxa de subnotificação (diabéticos que desconhecem sua condição)
- 🗺️ **Mapear** disparidades regionais e sociodemográficas no acesso ao diagnóstico
- ⚕️ **Estratificar** o perfil de severidade clínica da população diabética
  
> **Autor:** Lorenzo de Ávila Carpes

---

## 🔗 Links

| Recurso | Link |
|---|---|
| 📊 Dashboard publicado | [Acessar no Power BI Service](https://app.powerbi.com/seu-link-aqui) |
| 🗄️ Microdados PNS 2013 | [IBGE — PNS Exames Laboratoriais](https://ftp.ibge.gov.br/PNS/2013/Divulgacoes/Outros/Exames/BASE_EXAMES_PNS_2013.txt) |
| 📖 Dicionário de variáveis | [FTP IBGE — Documentação](https://ftp.ibge.gov.br/PNS/2013/Divulgacoes/Outros/Exames/Dicionario_Exames_PNS_2013.xlsx) |
| 📋 Nota Técnica Peso Lab | [Nota Técnica Oficial IBGE](https://ftp.ibge.gov.br/PNS/2013/Divulgacoes/Outros/Exames/Nota_tecnica_Exames_Laboratoriais_PNS.pdf) |

---

## 🏗️ Arquitetura da Solução
A solução separa a camada de processamento estatístico (Python) da camada de visualização (Power BI). Toda a lógica de negócio envolvendo limpeza, categorização, ponderação e modelagem dimensional é resolvida no ETL, deixando o DAX restrito a agregações contextuais sobre uma base já estruturada.

<img width="671" height="461" alt="PROCESSO DE DESACOPLAMENTO E INTELIGÊNCIA DE DADOS drawio" src="https://github.com/user-attachments/assets/23d27d54-3031-4134-8d85-f7798cf9e22c" />

---

## 🔄 Pipeline de Dados (ETL)

O processamento foi estruturado em quatro etapas documentadas no notebook `limpeza_PNS_2013.ipynb`:

### Etapa 1 — Redução de Dimensionalidade e Limpeza
- Filtragem do dataset bruto: **509 → 32 variáveis** essenciais para análise epidemiológica
- Remoção de registros com inconsistências críticas ou exames laboratoriais inválidos
- Resultado: **8.952 registros qualificados**

### Etapa 2 — Padronização e Tradução Semântica
- Mapeamento dos **códigos alfanuméricos IBGE** para nomenclaturas clínicas legíveis
- Aplicação de dicionários de tradução via `dict.map()` para variáveis categóricas
- Padronização em formato Snake Case para compatibilidade com Power BI

### Etapa 3 — Engenharia de Atributos
```python
# IMC calculado a partir das métricas antropométricas
df_modelagem['IMC'] = (peso_num / (altura_m ** 2)).round(2)

# Discretização de variáveis contínuas para dimensões
df_modelagem['Faixa_Etaria']         = pd.cut(idade_num, bins=[0,29,45,59,150], ...)
df_modelagem['Faixa_Dias_Exercicio'] = pd.cut(dias_num,  bins=[-1,0,2,4,7], ...)

# Classificação algorítmica do Perfil de Morbidade (np.select em cascata)
df_modelagem['Perfil_Morbidade'] = np.select(condicoes, rotulos,
                                              default='Não Diabético')
```

### Etapa 4 — Modelagem Dimensional (Star Schema)
- Geração de **Surrogate Keys** via `drop_duplicates()` + `reset_index()`
- Propagação das FKs à tabela fato via `merge()`
- Exportação com `encoding='utf-8-sig'` para compatibilidade com Power BI no Windows

---

## ⭐ Modelagem Dimensional — Star Schema

<img width="2121" height="1036" alt="Modelo Dimensional — Star Schema" src="https://github.com/user-attachments/assets/d51c7a62-9e1d-4979-bd6d-caa96edef73e" />


### 📐 O Peso Laboratorial (`Peso_Lab`)

> O banco de dados incorpora o **Peso Laboratorial (Peso_Lab)**, variável recalibrada pelo IBGE para corrigir distorções decorrentes do desenho amostral complexo e das elevadas taxas de não-resposta na etapa de coleta biológica (~52% de perda). O peso foi **normalizado para o tamanho da subamostra (n = 8.952)**, garantindo que as taxas, médias e proporções reflitam o perfil epidemiológico ajustado, embora não permitam a projeção de volumes populacionais absolutos *(IBGE, Nota Técnica, 2020)*.

**Padrão de medida ponderada no DAX:**
```dax
// Média ponderada: padrão aplicado a todas as métricas contínuas
Media_HbA1c_Ponderada =
VAR Tabela_Valida = FILTER(Fato_Individual_Lab,
                            NOT(ISBLANK(Fato_Individual_Lab[Hemoglobina_Glicada])))
RETURN
DIVIDE(
    SUMX(Tabela_Valida, Fato_Individual_Lab[Hemoglobina_Glicada] * Fato_Individual_Lab[Peso_Lab]),
    SUMX(Tabela_Valida, Fato_Individual_Lab[Peso_Lab]),
    BLANK()
)
```

---

## 📊 O Dashboard — Storytelling por Página

### Página 1 · Panorama da Prevalência e Lacuna Diagnóstica
<img width="1742" height="976" alt="image" src="https://github.com/user-attachments/assets/f81e3a5e-a037-4cd5-bb2c-4a99f2a792e2" />

Visão geral sobre o volume de amostras analisadas, incidência de pré-diabetes e disparidade de gênero entre indivíduos subnotificados. KPIs centrais do trabalho expostos na abertura.
* Prevalência de DM (9,81%), subnotificação (26,12%), pré-diabetes (13,81%) e cobertura laboratorial. Distribuição do status glicêmico e comparativo de subnotificação por sexo e faixa etária.
* Para cada 4 diabéticos identificados laboratorialmente, 1 desconhece sua condição. Somado ao pré-diabetes, mais de 23% da subamostra está em trajetória de risco metabólico progressivo.

### Página 2 · Análise Ponderada de Indicadores Metabólicos e Status Glicêmico
<img width="1745" height="982" alt="image" src="https://github.com/user-attachments/assets/92bc8ffe-48dd-4240-9547-41da85dd76a8" />

Correlação entre medidas físicas (IMC e Circunferência Abdominal) e os níveis de HbA1c, segmentados pelo status glicêmico. Gráfico de dispersão com limiares diagnósticos (HbA1c ≥ 6,5% e IMC = 25) dividindo o espaço em 4 quadrantes de risco.
A progressão é linear e coerente:
 
| Status | IMC médio | Circ. Abdominal |
|---|---|---|
| Normal | 25,83 kg/m² | 89 cm |
| Pré-diabetes | 28,05 kg/m² | — |
| Diabetes | 29,24 kg/m² | 101 cm |

 * Valida **IMC e circunferência abdominal como preditores de triagem de baixo custo**, permitindo que agentes comunitários priorizem encaminhamentos para testagem glicêmica mesmo sem queixas clínicas.

### Página 3 · Geografia da Prevalência e da Subnotificação de DM
<img width="1738" height="975" alt="image" src="https://github.com/user-attachments/assets/57968a8a-16b1-4fe2-9f89-57e671466b38" />

Matrizes de calor cruzando **Região × Faixa Etária** para prevalência e subnotificação.
* A Região Norte tem a **menor prevalência declarada (6,85%)** e a **maior subnotificação (38,62%)**, chegando a **88,08% entre jovens de 18–29 anos** na subamostra.
* Dados autodeclarados subestimam sistematicamente a carga real das regiões mais vulneráveis.
  
### Página 4 · Atividade Física, Controle Glicêmico e Gravidade Clínica
<img width="1739" height="973" alt="image" src="https://github.com/user-attachments/assets/72a393bc-c6f3-4aeb-a360-50502f3bf19a" />

Panorama comparativo entre frequência de exercícios, HbA1c média e perfil de morbidade. Inclui análise da anomalia detectada no grupo de 3-4 dias de exercício (possível causalidade reversa).

* Sedentários apresentam o pior controle glicêmico (**HbA1c 7,42%**). Contudo, o grupo de **3–4 dias** apresenta glicemia e gravidade maiores que o de **1–2 dias**, é possível que ocorra uma **causalidade reversa** de pacientes com quadros já graves que são prescritos a exercícios por indicação médica, invertendo a relação aparente na subamostra.
* Frequência de exercício isolada é um **preditor ambíguo** em dados transversais. Programas de atividade física em saúde pública devem ser acompanhados de **monitoramento laboratorial contínuo**, especialmente em grupos já em tratamento.
  
### Página 5 · Funil de Severidade e Desfechos Clínicos
<img width="1738" height="972" alt="image" src="https://github.com/user-attachments/assets/2c917677-981f-498f-a838-f9bd3c8df45f" />

Afunilamento da amostra diagnosticada conforme o agravamento do quadro metabólico. Percentuais calculados **exclusivamente dentro da população com diagnóstico formal**, respeitando o desenho condicional do questionário PNS 2013.

* Entre os diabéticos diagnosticados:
- **6,58%** já atingiram estágio **Grave** (infarto, AVC ou amputação)
- **23,19%** estão no estágio **Moderado** (complicações instaladas ou internação prévia)
- Quase **30%** já apresenta algum desfecho clínico severo

* Cada avanço ao estágio grave representa internações de maior complexidade, elevando o custo médio por internação para valores significativamente superiores a **R$ 832,93**
> ⚠️ **Nota metodológica:** Indivíduos subnotificados (HbA1c ≥ 6,5% sem diagnóstico prévio) não foram submetidos às perguntas sobre complicações crônicas pelo desenho condicional do questionário PNS 2013. Sua severidade real é desconhecida e possivelmente subestimada.


### Página 6 · Estratificação da Subnotificação por Fatores Socioeconômicos
<img width="1741" height="977" alt="image" src="https://github.com/user-attachments/assets/f7094b83-4fed-4264-a31b-75e63dfd9049" />

Árvore de decomposição da taxa de subnotificação por **nível de instrução**, **posse de plano de saúde** e **raça/cor**. Evidencia os determinantes sociais que modulam o acesso ao diagnóstico.

* O achado mais contraintuitivo do projeto: o **nível de instrução é um preditor de subnotificação mais forte do que possuir plano de saúde.** Entre indivíduos com Ensino Fundamental Completo, a diferença entre ter ou não plano é de apenas **0,66 ponto percentual** (39,47% vs. 38,81%). A barreira ao diagnóstico é **cognitiva e sociocultural antes de ser financeira.**
---

## 🔎 Principais Insights

| Indicador | Resultado |
|---|---|
| Prevalência de DM (ponderada) | **9,81%** |
| Taxa de subnotificação | **26,12%** (~1 em cada 4 diabéticos) |
| Prevalência de pré-diabetes | **13,81%** |
| Cobertura HbA1c válida | **95,7%** da subamostra |
| IMC médio — Normal vs. Diabetes | **25,83** → **29,24 kg/m²** |
| Circ. Abdominal — Normal vs. Diabetes | **89 cm** → **101 cm** |
| Pior subnotificação regional | **Norte: 38,62%** |
| Pior subnotificação etária | **Jovens nortistas 18–29 anos: 88,08%** |
| Casos graves entre diagnosticados | **6,58%** |

> 💡 **Achado crítico:** O nível de instrução é um preditor de subnotificação mais forte que a posse de plano de saúde. Para indivíduos com baixa escolaridade, o acesso ao setor privado não garante diagnóstico melhor (diferença de apenas 3,46 p.p. entre ter ou não plano de saúde).

---
## 🎛️ Filtros Globais do Dashboard

Todas as páginas compartilham um painel de filtros interativos que permitem
segmentar a análise por subgrupos populacionais específicos:

| Filtro | Tipo | Dimensão |
|---|---|---|
| Sexo | Seleção categórica | Dim_Sociodemografia |
| Faixa Etária | Seleção categórica | Dim_Sociodemografia |
| IMC | Intervalo numérico | Fato_Individual_Lab |
| Circunferência Abdominal | Intervalo numérico | Fato_Individual_Lab |

> 💡 Os filtros de IMC e Circunferência Abdominal permitem isolar
> subgrupos de risco metabólico específicos, por exemplo, analisar
> a subnotificação exclusivamente entre obesos (IMC ≥ 30) ou entre
> indivíduos com risco abdominal elevado (CA > 88 cm em mulheres
> e > 102 cm em homens).

---
## 🛠️ Stack Tecnológica

| Camada | Tecnologia | Uso |
|---|---|---|
| Extração e Transformação | Python 3 · Pandas · NumPy | ETL, limpeza, engenharia de atributos |
| Documentação do código | Rich | Inspeção e print formatado no notebook |
| Modelagem Dimensional | Star Schema (1 Fato · 5 Dims) | Arquitetura para BI |
| Visualização | Power BI Desktop | Dashboard interativo, 6 páginas |
| Linguagem analítica | DAX | Medidas ponderadas, KPIs epidemiológicos |
| Controle de versão | Git · GitHub | Versionamento do projeto |

---

## 📁 Estrutura do Projeto

```
data-intelligence-diabetes-pns2013/
├── data/
│   ├── raw/                        # Microdados brutos PNS 2013 (não versionados)
│   └── processed/                  # CSVs do star schema (gerados pelo notebook)
│       ├── Fato_Individual_Lab.csv
│       ├── Dim_Sociodemografia.csv
│       ├── Dim_Geografia.csv
│       ├── Dim_Perfil_Clinico.csv
│       ├── Dim_Complicacoes_Diabetes.csv
│       └── Dim_Estilo_Vida_Acesso.csv
├── notebooks/
│   └── limpeza_PNS_2013.ipynb      # Pipeline ETL completo e documentado
├── dashboard/
│   ├── DM_Insight_PNS2013.pbix     # Dashboard Power BI
│   └── assets/                     # Imagens de fundo utilizadas no dashboard
├── .gitignore
└── README.md
```

---

## 🚀 Como Reproduzir

**1. Clone o repositório**
```bash
git clone https://github.com/seu-usuario/data-intelligence-diabetes-pns2013.git
cd data-intelligence-diabetes-pns2013
```

**2. Instale as dependências Python**
```bash
pip install pandas numpy rich
```

**3. Baixe os microdados**

Acesse o [FTP do IBGE](https://ftp.ibge.gov.br/PNS/2013/Microdados/) e salve o arquivo bruto em `data/raw/`.

**4. Execute o notebook ETL**
```bash
jupyter notebook notebooks/limpeza_PNS_2013.ipynb
```
Os CSVs serão gerados automaticamente em `data/processed/`.

**5. Abra o dashboard**

Abra `dashboard/DM_Insight_PNS2013.pbix` no Power BI Desktop e atualize a fonte de dados apontando para a pasta `data/processed/`.

---

## ⚠️ Premissas e Limitações

- Os dados refletem o cenário epidemiológico de **2013** (revisado em 2020)
- A variável `Diagnostico_Diabetes` depende de **autorrelato**, sujeita a viés de memória e compreensão
- Em conformidade com a Nota Técnica do IBGE, os resultados são apresentados em **proporções e taxas**, não em volumes populacionais absolutos
- Indivíduos subnotificados **não responderam** às perguntas sobre complicações crônicas pelo desenho condicional do questionário PNS 2013

---

## 📚 Referências

BEZERRA, Elvis Félix Severo et al. **Morbidade e gastos públicos hospitalares por Diabetes Mellitus no Brasil**. [S. l.: s. n.], 2025.

INSTITUTO BRASILEIRO DE GEOGRAFIA E ESTATÍSTICA. **Pesquisa Nacional de Saúde 2013: nota técnica – exames laboratoriais**. Rio de Janeiro: IBGE, 2013. Disponível em: https://ftp.ibge.gov.br/PNS/2013/Divulgacoes/Outros/Exames/Nota_tecnica_Exames_Laboratoriais_PNS.pdf. Acesso em: 12 abr. 2026.

---

<p align="center">
  Desenvolvido por <strong>Lorenzo de Ávila Carpes</strong>
</p>
