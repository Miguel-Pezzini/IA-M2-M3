# Grupo D — Exportações de Itajaí e contêineres

Trabalho das entregas M2.1 (proposta) e M2.2 (análise exploratória). A pergunta é quanto da oscilação mensal do número de contêineres cheios embarcados no Complexo de Itajaí pode ser antecipada com as exportações marítimas já declaradas em Itajaí. A tarefa é prever os contêineres cheios de um mês X usando só dados até X−1 (contêineres e peso líquido de X−1). Na M3, compara-se a regra “X repete os contêineres de X−1” com o peso líquido de X−1.

Nesta entrega não há modelo treinado. A comparação de erro fica para a M3. Os meses reservados para esse teste são 2025-02, 2025-03, 2025-04, 2025-05, 2025-06 e 2026-02. Eles continuam nos gráficos. Nenhum filtro foi mudado por causa deles.

## O que foi entregue

| Arquivo | Papel |
|---|---|
| `Grupo_D_Projeto_IA.docx` | Seções 1 a 4 do template. Seções 5 e 6 ficam para a M3. |
| `Grupo_D_M2_2_EDA.ipynb` | Notebook da EDA, já executado. |
| `Grupo_D_Fonte_A.csv` | Oito linhas da ANTAQ no recorte do alvo. |
| `Grupo_D_Fonte_B.csv` | Oito linhas do Comex no recorte de Itajaí. |

O notebook também grava `outputs/painel_mensal.csv`, `outputs/diagnostico.json` e as figuras em `outputs/figuras/`.

## O que cada fonte vira

Uma linha do Comex é um NCM em um mês, país, UF do produtor e unidade da Receita. Uma linha da ANTAQ é um item de carga de uma atracação. Na análise, uma linha é um mês.

- **Alvo:** soma de `QTCarga` no Complexo Portuário Itajaí, carga conteinerizada, contêiner cheio, embarcado, `FlagAutorizacao` = S, `FlagMCOperacaoCarga` = 1, navegação de longo curso. O mês é o da desatracação. Portonave entra. Itapoá fica de fora.
- **Outra grandeza:** soma de `KG_LIQUIDO` na URF `0927800` e via marítima (`01`). `VL_FOB` vai para o painel e não entra nas perguntas. `QT_ESTAT` não é somado, porque a unidade muda com o produto.

Cada fonte é somada até o mês e as duas séries se juntam mês a mês. Mês sem arquivo não vira zero.

## Dados usados

O notebook lê estes caminhos:

- `dados/antaq/{ano}/{ano}Atracacao.txt` e `{ano}Carga.txt`, de 2023 a 2026
- `dados/EXP_2023.csv` a `dados/EXP_2026.csv`
- `dados/dicionarios/URF.csv` e `VIA.csv`

Os dados estão no repositório, com uma exceção: o GitHub recusa arquivos acima de 100 MB. Por isso `{ano}Carga.txt` de 2023, 2024 e 2025 (405 a 453 MB) e `EXP_2023.csv` a `EXP_2025.csv` (104 a 114 MB) entram **zipados**, ao lado do original: `dados/antaq/{ano}/{ano}Carga.zip` e `dados/EXP_{ano}.zip`.

Depois de clonar, extraia cada zip na própria pasta, para que o notebook encontre `{ano}Carga.txt` e `EXP_{ano}.csv`. Os arquivos extraídos estão no `.gitignore` e não são versionados. Os demais arquivos (Atracação, `2026Carga.txt`, `EXP_2026.csv`) e os dicionários seguem em texto.

Nos arquivos de ANTAQ, 2023 e 2024 têm os doze meses, 2025 vai de janeiro a junho e 2026 tem janeiro e fevereiro. Isso descreve esses arquivos, não o calendário publicado pela agência. Mês ausente no arquivo não foi preenchido com zero.

Os dicionários `dados/dicionarios/URF.csv` e `VIA.csv` só rotulam o recorte. Ficaram de fora da conta as taxas de ocupação, os tempos de atracação, as cargas por rio, hidrovia, região ou área, `Carga_Conteinerizada.txt` e `NCM.csv`.

## O que a EDA mostrou

De janeiro de 2023 a setembro de 2026 há 45 meses civis no Comex. As duas fontes coincidem em 32. Nos arquivos de ANTAQ lidos, não há julho a dezembro de 2025 nem março a setembro de 2026.

- No mesmo mês, n = 32, correlação entre contêineres e kg = 0,94.
- Com defasagem, n = 30, correlação de X com os contêineres de X−1 = 0,69 e com o kg de X−1 = 0,54.
- Outubro de 2023 é o menor mês do alvo (7.811 contêineres, mediana 12.952,5) e permanece na série.
- Nos 32 meses em comum, a cabotagem, fora do alvo, é 21,8% dos contêineres cheios embarcados.

Correlação não é erro de previsão. Ela não decide se o peso do mês anterior supera repetir o volume de X−1.

## Como rodar

Abra `Grupo_D_M2_2_EDA.ipynb` e execute de cima para baixo. Usa pandas e matplotlib. Não há semente.

A primeira célula lê `dados/antaq/` e `dados/`. No Kaggle, se existir `/kaggle/input`, ela procura um conjunto com `antaq/{ano}/`, `comex/` e `dicionarios/`. Trocar o arquivo de um ano e rodar de novo muda os meses do painel.
