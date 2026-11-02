# Grupo D — Exportações de Itajaí e contêineres

Trabalho das entregas M2.1 (proposta) e M2.2 (análise exploratória). A pergunta é quanto da oscilação mensal do número de contêineres cheios embarcados no Complexo de Itajaí pode ser antecipada com as exportações marítimas já declaradas em Itajaí. A tarefa é prever os contêineres cheios de um mês X usando só dados até X−1 (contêineres e peso líquido de X−1). Na M3, compara-se a regra “X repete os contêineres de X−1” com o peso líquido de X−1.

Nesta entrega não há modelo treinado. A comparação de erro fica para a M3. Os meses reservados para esse teste são os seis últimos em que as duas defasagens existem: 2025-09, 2025-10, 2025-11, 2025-12, 2026-01 e 2026-02 (antes de 2025 estar completo, a mesma regra dava 2025-02 a 2025-06 e 2026-02). Eles aparecem nos gráficos e ficam fora das contas da Pergunta 3. Nenhum filtro foi mudado por causa deles.

## O que foi entregue

| Arquivo | Papel |
|---|---|
| `Grupo_D_Projeto_IA.docx` | Seções 1 a 4 do template. Seções 5 e 6 ficam para a M3. |
| `Grupo_D_M2_2_EDA.ipynb` | Notebook da EDA, já executado. |
| `Grupo_D_M2_2_EDA.html` | O mesmo notebook exportado em HTML. |
| `Grupo_D_Fonte_A.csv` | Oito linhas da ANTAQ no recorte do alvo. |
| `Grupo_D_Fonte_B.csv` | Oito linhas do Comex no recorte de Itajaí. |
| `requirements.txt` | Versões das bibliotecas para rodar o notebook. |

O notebook também grava `outputs/painel_mensal.csv`, `outputs/diagnostico.json` e as figuras em `outputs/figuras/`, que estão no repositório.

## O que cada fonte vira

Uma linha do Comex é um NCM em um mês, país, UF do produtor e unidade da Receita. Uma linha da ANTAQ é um item de carga de uma atracação. Na análise, uma linha é um mês.

- **Alvo:** soma de `QTCarga` no Complexo Portuário Itajaí, carga conteinerizada, contêiner cheio, embarcado, `FlagAutorizacao` = S, `FlagMCOperacaoCarga` = 1, navegação de longo curso. O mês é o da desatracação. Portonave entra. Itapoá fica de fora.
- **Outra grandeza:** soma de `KG_LIQUIDO` na URF `0927800` e via marítima (`01`). `VL_FOB` vai para o painel e não entra nas perguntas. `QT_ESTAT` não é somado, porque a unidade muda com o produto.

Cada fonte é somada até o mês e as duas séries se juntam mês a mês. Mês sem arquivo não vira zero.

## Dados usados

Todos os arquivos foram baixados em 19/10/2026:

- **ANTAQ:** painel Estatístico Aquaviário, aba Base de Dados ([link](https://aquarela.antaq.gov.br/single/?appid=11e6d427-c9b8-4a56-8e0f-6fe38c361192&sheet=3d8a69c8-caf0-4525-8908-309acb65d562); caminho: gov.br/antaq/pt-br/assuntos/estatistica). Dados publicados até fev/2026 (painel atualizado em 01/04/2026). O endereço antigo `web3.antaq.gov.br` saiu do ar.
- **Comex Stat:** base bruta do MDIC, detalhada por NCM ([link](https://www.gov.br/mdic/pt-br/assuntos/comercio-exterior/estatisticas/base-de-dados-bruta)). Dados publicados até set/2026.

O notebook lê estes caminhos:

- `dados/antaq/{ano}/{ano}Atracacao.txt` e `{ano}Carga.txt`, de 2023 a 2026
- `dados/EXP_2023.csv` a `dados/EXP_2026.csv`
- `dados/dicionarios/URF.csv` e `VIA.csv`

Os dados estão no repositório, com uma exceção: o GitHub recusa arquivos acima de 100 MB. Por isso `{ano}Carga.txt` de 2023, 2024 e 2025 (405 a 453 MB) e `EXP_2023.csv` a `EXP_2025.csv` (104 a 114 MB) entram **zipados**, ao lado do original: `dados/antaq/{ano}/{ano}Carga.zip` e `dados/EXP_{ano}.zip`.

Não é preciso extrair à mão: a primeira célula do notebook extrai cada zip na própria pasta quando `{ano}Carga.txt` ou `EXP_{ano}.csv` ainda não existe. Os arquivos extraídos estão no `.gitignore` e não são versionados. Os demais arquivos (Atracação, `2026Carga.txt`, `EXP_2026.csv`) e os dicionários seguem em texto.

Nos arquivos da ANTAQ, 2023, 2024 e 2025 têm os doze meses e 2026 tem janeiro e fevereiro, o último mês que a agência publicou. Mês ausente no arquivo não foi preenchido com zero.

Os dicionários `dados/dicionarios/URF.csv` e `VIA.csv` só rotulam o recorte. Ficaram de fora da conta as taxas de ocupação, os tempos de atracação, as cargas por rio, hidrovia, região ou área, `Carga_Conteinerizada.txt` e `NCM.csv`.

## O que a EDA mostrou

De janeiro de 2023 a setembro de 2026 há 45 meses civis no Comex. As duas fontes coincidem em 38, de jan/2023 a fev/2026. Março a setembro de 2026 existem só no Comex, porque a ANTAQ não publicou esses meses.

- No mesmo mês, n = 38, correlação entre contêineres e kg = 0,90. As toneladas declaradas por contêiner vão de 18,9 a 33,5 (mediana 25,3).
- Com defasagem, sem os 6 meses reservados, n = 31, correlação de X com os contêineres de X−1 = 0,69 e com o kg de X−1 = 0,53.
- Outubro de 2023 é o menor mês do alvo (7.811 contêineres, mediana 13.333) e permanece na série. Nesse mês as atracações da Portonave caíram de 54 para 22.
- Nos 38 meses em comum, a cabotagem, fora do alvo, é 22,6% dos contêineres cheios embarcados.

Correlação não é erro de previsão. Ela não decide se o peso do mês anterior supera repetir o volume de X−1.

## Como rodar

Na raiz do repositório, com Python 3.12:

```
python -m venv .venv
.venv\Scripts\activate          # Linux/Mac: source .venv/bin/activate
pip install -r requirements.txt
```

Abra `Grupo_D_M2_2_EDA.ipynb` com o Python do `.venv` e execute de cima para baixo. Não há semente. Para gerar o HTML de novo: `jupyter nbconvert --to notebook --execute --inplace Grupo_D_M2_2_EDA.ipynb` e depois `jupyter nbconvert --to html Grupo_D_M2_2_EDA.ipynb`.

A primeira célula extrai os zips que faltarem e lê `dados/antaq/` e `dados/`. No Kaggle, se existir `/kaggle/input`, ela procura um conjunto com `antaq/{ano}/`, `comex/` e `dicionarios/`. Trocar o arquivo de um ano e rodar de novo muda os meses do painel.
