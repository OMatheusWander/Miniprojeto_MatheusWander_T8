# Mini-Projeto Avaliativo – Módulo 1 · Análise de Dados com Python

**Aluno:** Matheus Wander da Silva  
**Turma:** T8

## Objetivo

Fazer uma Análise Exploratória de Dados (AED) da base **Varejo**: diagnosticar a qualidade dos dados, tratar os
problemas com decisões justificadas, resumir o comportamento de compra e exportar uma base tratada
(`varejo_limpo.csv`) que pode ser gerada de novo a partir da base original.

## Estrutura do repositório

```
Miniprojeto_MatheusWander_T8/
├── Miniprojeto_MatheusWander_T8.ipynb   # notebook comentado (código + análise)
├── dados/
│   └── Base_Varejo_csv.zip              # base ORIGINAL, como baixada do Kaggle
├── varejo_limpo.csv                     # base TRATADA (gerada pelo notebook)
├── requirements.txt
└── README.md
```

## Origem da base

- **Fonte:** Kaggle, dataset [`namespaiva/base-varejo`](https://www.kaggle.com/datasets/namespaiva/base-varejo/data)
- **Arquivo usado:** `Base Varejo.csv` (48,8 MB, criado no Kaggle em 03/04/2023), compactado em `dados/Base_Varejo_csv.zip`
- **Versão/data do download:** versão disponível em 02/11/2026
- **Formato:** CSV separado por `;`, 830.000 linhas e 14 colunas (4 delas vazias)

## O que cada linha representa

Cada linha é **um item (produto) dentro de uma compra**. Uma compra (`CO_ID`) tem vários itens, sempre do mesmo cliente e
da mesma data, e cada cliente tem atributos fixos. Por isso a análise separa **itens** (linhas), **compras** (`CO_ID`
distintos) e **clientes** (`CL_ID` distintos). A base **não tem preço, valor nem quantidade**, então não permite calcular
faturamento.

## Colunas utilizadas

| Original | Na base tratada | Significado |
|---|---|---|
| `DATA` | `data` | data da compra (convertida para data) |
| `CO_ID` | `id_compra` | identificador da compra |
| `CL_ID` | `id_cliente` | identificador do cliente (1.000 clientes) |
| `CL_GENERO` | `genero` | gênero do cliente (F/M) |
| `CL_EC` | `estado_civil_cod` | estado civil, codificado de 1 a 5 (a base não traz o dicionário) |
| `CL_FHL` | `qtd_filhos` | número de filhos do cliente (0 a 4) |
| `CL_SEG` | `segmento` | segmento do cliente (A, B, C) |
| `PR_ID` | `id_produto` | identificador do produto (229) |
| `PR_CAT` | `categoria` | categoria do produto |
| `PR_NOME` | `produto` | nome do produto |
| `Unnamed: 10` a `13` | *(removidas)* | colunas vazias criadas por `;` sobrando no fim das linhas |

## Decisões de limpeza

| Problema encontrado | Quantidade | Decisão e justificativa |
|---|---|---|
| Colunas `Unnamed` vazias | 4 colunas (100% vazias) | **Removidas**: são defeito de formatação, não dados. |
| Valores ausentes (`NaN`) nas colunas reais | 0 | Nada a tratar. |
| Textos vazios ou só com espaços | 0 | Nada a tratar. |
| `#N/D` (erro de PROCV do Excel) em categoria e produto | 3.650 itens, todos do produto 107 | **Preenchidos** com `NAO IDENTIFICADO`. O item, a compra e o cliente são válidos, então remover apagaria compras reais, e não há como deduzir a categoria sem inventar dado. |
| Nome fora da caixa alta (`REFRIGERANTE LIMaO`) | 7.320 itens | **Padronizado** com `strip().upper()` para `REFRIGERANTE LIMAO`. |
| Erro de digitação `HAMBUGUER` | 1 nome de produto | **Corrigido** para `HAMBURGUER`. |
| Linhas idênticas | 96.553 (11,6%) | **Mantidas.** É o mesmo produto mais de uma vez na mesma compra. Sem coluna de quantidade, essa é a forma de registrar várias unidades. As repetições estão espalhadas por 14.566 das 18.471 compras e quase nunca passam de 4, um padrão de carrinho de compras, não de erro de carga. **Duplicatas confirmadas: 0.** |
| Gênero codificado (F/M) | — | **Mantido**: o código é consistente, sem variações. |

**Resultado:** 830.000 linhas antes e 830.000 depois (nenhum registro removido); de 14 para 10 colunas.

## Explicação do processo

A **leitura** acontece na seção 2 do notebook, com `pd.read_csv("dados/Base_Varejo_csv.zip", sep=";")`, que lê o CSV
direto do zip original. A **transformação** fica concentrada na função `tratar_dados()` (seção 4), que devolve uma cópia
tratada sem alterar a base bruta; a função `diagnosticar()` é aplicada antes e depois para comparar a qualidade. As
análises usam a função `gerar_resumo()` (seção 7). A **gravação** acontece na seção 8, com
`df.to_csv("varejo_limpo.csv", index=False)`, conferida relendo o arquivo. Uma decisão que afetou diretamente a
qualidade da análise foi **não remover as linhas repetidas**: se elas fossem tratadas como duplicatas, a base perderia
11,6% dos itens e as métricas de volume (itens por compra, participação das categorias) ficariam subestimadas, sem
nenhuma evidência de erro que justificasse a remoção.

## Conclusões

1. **A base tem 830.000 itens, 18.471 compras e 1.000 clientes**, entre jan/2019 e dez/2022. Cada compra tem em média
   **~45 itens** (de 1 a 89), um perfil de "compra do mês".
2. **ALIMENTOS responde por 52,4% dos itens**, seguida de **HIGIENE (18,7%)** e **LIMPEZA (17,6%)**. ACESSORIOS é a menor
   categoria (1,8%).
3. **Os clientes têm em média 1,14 filho** (mínimo 0, máximo 4), mas **52,7% não têm filhos**. A média não representa o
   cliente típico.
4. **Mulheres são 51,9% dos clientes e 52,1% dos itens**, com o mesmo tamanho de cesta dos homens (45,0 × 44,9 itens por
   compra). O gênero, sozinho, não diferencia o comportamento de compra.
5. **O segmento B concentra 64,5% dos clientes** e 63,9% dos itens.

**Recomendação:** priorizar ALIMENTOS em abastecimento e promoções e explorar promoções cruzadas com HIGIENE e LIMPEZA,
que aparecem juntas em 92% das compras. Antes de segmentar campanhas por gênero ou número de filhos, vale validar se
esses atributos influenciam a compra, porque nesta base eles não mostraram diferença.

**Limitação:** a base não tem preço nem quantidade, então não mede faturamento e conta cada linha repetida como uma
unidade. Além disso, o filtro de itens infantis (fralda, papinha, chupeta, mordedor) mostrou que **todos os 1.000
clientes** os compraram, inclusive os 527 sem filhos, sinal de que os dados podem ser **simulados**. As conclusões
descrevem a base, não necessariamente um varejo real.

## Como executar

**Localmente**

```bash
git clone https://github.com/OMatheusWander/Miniprojeto_MatheusWander_T8.git
cd Miniprojeto_MatheusWander_T8
pip install -r requirements.txt
jupyter notebook Miniprojeto_MatheusWander_T8.ipynb
```

**No Google Colab:** abra o notebook e rode, na primeira célula:

```python
!git clone https://github.com/OMatheusWander/Miniprojeto_MatheusWander_T8.git
%cd Miniprojeto_MatheusWander_T8
```

Depois execute todas as células (*Ambiente de execução → Executar tudo*). O CSV de entrada é lido de
`dados/Base_Varejo_csv.zip`, e `varejo_limpo.csv` é gravado na raiz do repositório.

## Referências

- Documentação do pandas: <https://pandas.pydata.org/docs/>
- Materiais e exemplos das aulas do curso Análise de Dados com Python (T8)
