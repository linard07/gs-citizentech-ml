# Dados — GS CitizenTech (IA & ML)

## Fonte
- **Instituição:** INMET (Instituto Nacional de Meteorologia)
- **Base:** Banco de Dados Históricos (dados horários de estações automáticas)
- **Link:** https://portal.inmet.gov.br/dadoshistoricos
- **Arquivos usados:** `https://portal.inmet.gov.br/uploads/dadoshistoricos/AAAA.zip` (um por ano, de 2013 a 2025)
- **Estação:** A701, São Paulo / Mirante de Santana
- **Período:** 2013 a 2025
- **Natureza:** dados reais e públicos. Nenhum dado simulado.

## Variáveis utilizadas (horárias, agregadas por dia)
Precipitação, pressão atmosférica, temperatura do ar, umidade relativa, velocidade do vento e radiação global.

## Resultado do processamento
- 113.945 registros horários (de 31/12/2012 22h a 31/12/2025 20h, horário de Brasília).
- 4.695 dias utilizáveis após a limpeza (dias com pelo menos 20 horas válidas e com o dia seguinte observado).
- Treino (2013–2022): 3.633 dias, 130 com chuva forte (3,6%). Teste (2023–2025): 1.062 dias, 47 com chuva forte (4,4%).

## Arquivos
- Os dados brutos (zips do INMET) não estão no repositório porque são grandes; o notebook baixa tudo automaticamente.
- Ao rodar o notebook, ele gera `dados/inmet_a701_diario_processado.csv`, a tabela diária final usada no modelo (atributos e alvo `chuva_forte_amanha`).

## Como reproduzir
1. `pip install -r ../requirements.txt`
2. Abra `../GS_CitizenTech_ML.ipynb` (no Google Colab ou no Jupyter) e rode todas as células. O notebook baixa os zips para `dados/raw/` sozinho.

## Tratamento
- Valores `-9999` tratados como ausentes.
- Hora UTC convertida para horário de Brasília.
- Dias com menos de 20 horas válidas descartados; valores faltantes dos atributos preenchidos com a mediana.
- Alvo: `chuva_forte_amanha = 1` se a chuva total do dia seguinte for ≥ 30 mm.

## Limitações
- Uma única estação representa a cidade toda; chuva localizada pode não ser captada.
- Chuva diária é *proxy* de risco de alagamento, não uma medição direta de alagamento.
