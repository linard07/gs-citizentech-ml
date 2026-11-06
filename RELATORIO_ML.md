# RELATORIO_ML — CitizenTech: alerta de chuva forte em São Paulo

**Grupo:** Guilherme Linard (RM 555768), Lucas, Ali e Luquinha — Turma 3ESPZ — FIAP
**Repositório:** [colar link do GitHub] · **Vídeo:** [colar link do YouTube não listado]

## 1. Problema e dados reais

A CitizenTech quer avisar a população de São Paulo quando houver risco de desastres (alagamento, calor extremo, frio extremo) e orientá-la para pontos de segurança. Neste trabalho focamos em **chuva forte / alagamento**, porque o desafio pede um único modelo.

**Pergunta de ML:** com as condições meteorológicas de hoje, vai haver chuva forte (≥ 30 mm no dia) amanhã na cidade de São Paulo? É um problema de **classificação binária**.

**Dados reais e públicos:** INMET, Banco de Dados Históricos, estação automática **A701 – São Paulo / Mirante de Santana**, dados horários de **2013 a 2025** (https://portal.inmet.gov.br/dadoshistoricos). Foram **113.945 registros horários**, agregados em **4.695 dias utilizáveis**. Os eventos de chuva forte são raros: **177 dias (3,8%)**. Nenhum dado foi simulado. Mais detalhes em `ml/dados/README.md`.

**Limitações:** a chuva medida em uma estação é um *proxy* do risco de alagamento (não há medição direta de alagamento) e uma única estação não representa a cidade inteira.

## 2. Escolha do modelo

Usamos **Random Forest** porque (i) funciona bem com dados tabulares e relações não lineares entre pressão, temperatura e chuva; (ii) não exige normalização dos atributos; (iii) aceita `class_weight="balanced"`, importante para um evento raro; e (iv) informa a importância dos atributos, o que ajuda a explicar o alerta para o cidadão.

## 3. Preparação dos dados e divisão treino/teste

- Hora UTC convertida para o horário de Brasília e valores `-9999` tratados como ausentes.
- Agregação **diária**: chuva total e máxima por hora, temperatura (máx., mín., média), umidade, pressão e sua variação, vento e radiação.
- Atributos de memória (chuva e pressão de 1 e 2 dias antes) e sazonalidade (mês em seno/cosseno).
- Dias com menos de 20 horas válidas foram descartados; valores faltantes foram preenchidos com a mediana.
- **Alvo:** `chuva_forte_amanha = 1` se a chuva total do dia seguinte for ≥ 30 mm.

**Divisão temporal**, e não aleatória, para não vazar informação do futuro para o treino:

| Conjunto | Período | Dias | Eventos de chuva forte |
|---|---|---|---|
| Treino | 2013–2022 | 3.633 | 130 (3,6%) |
| Teste | 2023–2025 | 1.062 | 47 (4,4%) |

## 4. Treino com diferentes hiperparâmetros

Usamos `RandomizedSearchCV` com **15 combinações** de **3 hiperparâmetros** (`n_estimators`, `max_depth`, `min_samples_leaf`), validação cruzada `TimeSeriesSplit` com 5 divisões **somente no treino** e a métrica **PR-AUC**, adequada a classes desbalanceadas.

![Tabela com as 15 combinações](ml/figuras/tabela_hiperparametros.png)

![PR-AUC de treino e validação nas 15 combinações](ml/figuras/grafico_hiperparametros.png)

**Melhor combinação:** `n_estimators=300`, `max_depth=3`, `min_samples_leaf=20`, com PR-AUC de validação de **0,116** (desvio de 0,030). Note o padrão: os modelos profundos (por exemplo `max_depth=None`) chegam a PR-AUC de treino **1,000**, mas só **0,078** na validação. Eles decoram o treino. A busca escolheu justamente o modelo mais simples.

## 5. Resultados em treino e teste

Resultados do melhor modelo, com limiar de decisão de 0,5. A prevalência de chuva forte no teste (linha de base de um classificador aleatório para PR-AUC) é de **0,044**.

| Métrica | Treino | Teste |
|---|---|---|
| Precisão | 0,100 | 0,105 |
| Recall | 0,846 | 0,787 |
| F1 | 0,179 | 0,185 |
| PR-AUC | 0,185 | 0,126 |
| ROC-AUC | 0,847 | 0,792 |

![Treino × teste, curva precisão–recall e importância dos atributos](ml/figuras/resultados_treino_teste.png)

**Matriz de confusão no teste** (1.062 dias): 700 dias sem chuva forte e sem alarme; **37 acertos**; **315 alarmes falsos**; **10 eventos perdidos**.

Atributos mais importantes: temperatura mínima, pressão média, chuva máxima por hora e chuva total do dia, seguidos de temperatura média e mês do ano.

**Overfitting.** Nos modelos mais complexos, ele é forte: PR-AUC de treino perto de 1,0 contra menos de 0,10 na validação (gráfico acima). Por isso o modelo final é raso. Nele, a diferença entre treino e teste é pequena (ROC-AUC 0,847 contra 0,792; recall 0,846 contra 0,787), então o overfitting ficou controlado. A PR-AUC cai de 0,185 para 0,126, mas ainda fica cerca de **3 vezes acima do acaso** (0,044). O problema principal, então, não é decorar o treino: os atributos de uma única estação têm capacidade limitada de prever chuva forte no dia seguinte.

## 6. Conclusão e leitura para o cidadão

O modelo **encontra a maioria das chuvas fortes** (acertou 37 dos 47 eventos de teste, 79%), mas **erra muito ao avisar**: só cerca de 1 em cada 10 alertas (10,5%) corresponde a uma chuva forte de fato, e o alarme dispararia em cerca de um terço dos dias (352 de 1.062). Para o cidadão, isso significa que poucas enchentes passariam sem aviso, mas haveria tantos alarmes falsos que as pessoas poderiam parar de confiar no sistema.

Por isso, na CitizenTech, o resultado serve como um **nível de atenção** para apoiar a Defesa Civil, e **não** como gatilho de um alarme automático. Para melhorar: aumentar o limiar de decisão para reduzir alarmes falsos, usar previsão do tempo e radar além de observações, incluir mais estações e validar o alerta com registros reais de alagamentos (SP156, Infosiga). Calor e frio extremos ficam como modelos separados, em trabalhos futuros.

Os pontos de segurança (áreas refrigeradas, água mais barata, áreas governamentais) são **fictícios** neste protótipo. O gatilho do alarme usa apenas dados reais.
