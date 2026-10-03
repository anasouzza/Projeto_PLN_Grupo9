# Projeto Integrador — Linguagem Natural (P90100051)

**Turma:** 4169CIAM2A1_P1
**Grupo:** Grupo P03
**Integrantes:** Ana Clara de Souza, Leticia Vieira, Mariana Garcez
**Projeto escolhido:** P03 — Encaminhamento de solicitações de atendimento
**Trilha (M4):** A — Classes e ambiguidades
**Última atualização:** M4 (versão final)

## Problema

Grandes volumes de solicitações de atendimento chegam de forma desorganizada por
diferentes canais de comunicação, gerando demora na identificação e no
encaminhamento correto de cada demanda.

- **Usuário direto:** o solicitante.
- **Envolvidos:** a equipe responsável pelo recebimento das mensagens e os setores
  responsáveis pelo atendimento das demandas.
- **Decisão apoiada:** organizar e encaminhar cada solicitação para o setor
  responsável.
- **Recorte:** organização, classificação e encaminhamento das solicitações
  recebidas pelos diferentes canais para o setor responsável. Ficam fora do
  escopo: a resolução da demanda em si, o acompanhamento completo do atendimento
  e a comunicação de retorno ao solicitante.

## Pergunta operacional

**Qual setor da instituição deve receber inicialmente esta solicitação de
atendimento?**

- **Unidade de análise:** uma mensagem completa enviada por um usuário (e-mail,
  formulário, chat ou WhatsApp).
- **Entrada permitida:** apenas o texto da mensagem da solicitação.
- **Saída esperada:** um único setor de destino, entre `financeiro`,
  `secretaria_academica`, `suporte_tecnico`, `biblioteca` e `coordenacao`.
- **Uso do resultado:** encaminhar automaticamente a solicitação ao setor
  responsável, prevendo revisão humana nos casos de baixa confiança.
- **Fora do escopo:** resolver a solicitação, responder ao usuário ou identificar
  sentimento e urgência.
- **Regra para casos ambíguos:** quando uma mensagem parecer pertencer a mais de
  um setor ou não tiver informação suficiente, ela é associada ao setor mais
  provável no contexto da mensagem, sinalizada para revisão humana.

## Pergunta de pesquisa (M4 — Trilha A)

Quais pares de setores concentram os casos mais difíceis do P03 (erros das duas
representações) e o que esses pares têm em comum?

## Corpus

O projeto passou por duas fases de dados. Os corpora são separados e nenhum foi
alterado após o download.

| Fase | Arquivo | Registros | Uso |
|---|---|---|---|
| M2/M3 | `P03_solicitacoes_atendimento.csv` | 240 | preparação, representação e baseline inicial (M14 e M15) |
| M4 | `P03_solicitacoes_atendimento_v2.csv` | 5.000 | comparação A×B, análise de erros e teste final (M16) |
| M4 | `P03_extensao_TrilhaA.csv` | 120 | extensão própria, mantida separada do corpus-base |

### Corpus inicial (240 registros)

- **Formato:** CSV em UTF-8 com BOM, delimitador `;`, cabeçalho na primeira linha.
- **Conteúdo:** 240 solicitações sintéticas em português brasileiro.
- **Campos:**

  | Campo | Papel | Descrição |
  |---|---|---|
  | `id` | identificador | identifica cada solicitação de forma única |
  | `texto` | entrada | texto da mensagem enviada pelo usuário |
  | `setor_destino` | saída (rótulo) | setor correto de encaminhamento — **não é entrada do modelo** |
  | `canal` | referência | canal de origem da mensagem (e-mail, aplicativo, etc.) |
  | `urgencia_referencia` | referência | não deve ser usada como atalho para prever a saída |
  | `particao_recomendada` | controle | separa treino / validação / teste — nunca é atributo de entrada |

- **Qualidade observada:** o corpus contém, intencionalmente, textos ausentes,
  duplicações, variações de caixa, espaços extras, grafias sem acento e emojis.
  Esses problemas não foram corrigidos no arquivo original; as decisões de
  preparação estão documentadas nos notebooks M14 e M15.
- **Partições:** treino (141), validação (49) e teste (50). O teste concentra
  maior ambiguidade de propósito e não foi usado para ajustar nenhuma decisão.

### Corpus-base do M4 (v2, 5.000 textos)

Corpus sintético mantido sem alteração, usado nas partições de treino, validação
e teste do notebook M16 (o teste final tem 750 mensagens).

### Extensão própria (Trilha A, 120 exemplos)

Colunas do CSV: `id`, `texto`, `setor_destino` (rótulo), `par_alvo`,
`n_pedidos` (1 ou 2), `marcador_prioridade` (sim/nao), `ambiguo` (sim/nao),
`setor_alternativo`, `origem`, `revisado_pelo_grupo`.

- 60 textos com um pedido, 42 com dois pedidos e marcador, e 18 com dois pedidos
  sem marcador.
- 3 pares, com 40 textos de cada: biblioteca × secretaria, financeiro × secretaria
  e coordenação × suporte.
- **Regra de rótulo:** vale o pedido marcado; sem marcador, vale o primeiro pedido.

## Origem e licença

Corpus sintético, criado para uso didático na disciplina de Linguagem Natural
(2026), pelo professor Rogerio Mandelli. Não contém dados de pessoas ou
organizações reais e não sustenta conclusões sobre populações, organizações ou
serviços reais.

Licença: **CC BY 4.0** — https://creativecommons.org/licenses/by/4.0/
Atribuição sugerida: Disciplina de Linguagem Natural, 2026 — Prof. Rogerio Mandelli.

## Critério de utilidade e comparação

O resultado será útil se ajudar a encaminhar cada solicitação para o setor
correto, evitando triagem manual e reduzindo o tempo de encaminhamento. A
comparação mínima é entre um baseline de classe majoritária (setor mais
frequente no treino) e um classificador supervisionado, avaliados por acurácia
e F1-macro, com leitura por classe. No M4, a comparação principal é entre duas
representações do mesmo classificador: **A** (unigramas, ngram (1,1)) e **B**
(unigramas + bigramas, ngram (1,2)).

## Resultados

### Etapa M2/M3 — preliminar (corpus de 240, partição de validação)

O teste permaneceu preservado nesta etapa.

| Abordagem | Acurácia | F1-macro |
|---|---|---|
| Referência majoritária (sempre "financeiro") | 0,245 | 0,079 |
| Classificador supervisionado (TF-IDF + regressão logística) | 1,000 | 1,000 |

O classificador superou amplamente a referência majoritária, que erra muito por
ignorar as classes menos frequentes. O resultado de 100% deve ser lido com
cautela: o corpus era sintético, pequeno (49 registros de validação) e com
vocabulário muito distintivo por setor. Por isso, no M4 a análise foi refeita
no corpus maior (v2), onde a tarefa se mostrou bem menos trivial.

### Etapa M4 — final (corpus v2 e extensão)

| Conjunto | Acurácia | F1-macro | Erros |
| --- | --- | --- | --- |
| Validação, A (1,1) | 0,935 | 0,934 | 49 |
| Validação, B (1,2) | 0,960 | 0,959 | 30 |
| Teste, B | 0,951 | 0,949 | 37 de 750 |
| Extensão, B | 0,683 | 0,677 | 38 de 120 |

Na extensão: um pedido 53/60, dois pedidos com marcador 21/42, dois pedidos sem
marcador 8/18.

## Conclusão

**Hipótese enfraquecida.** Na validação, 14 dos 30 erros de B ficaram em 3 pares,
mas no teste só 9 dos 37 caíram nesses pares. O que se repetiu foi a dificuldade
com textos de dois pedidos. O marcador de prioridade não ajudou (21/42 com
marcador e 8/18 sem), mas essa leitura foi feita depois de ver os resultados e é
apenas exploratória.

## Erros relevantes e riscos

O erro mais grave é encaminhar a solicitação a um setor errado, gerando atraso e
retrabalho de redirecionamento. O risco mais provável é a confusão entre setores
com escopos próximos. Esse risco foi apontado no M05 para `secretaria_academica`
e `coordenacao`, e a análise do M4 mostrou que, além dos pares de escopo próximo,
as mensagens com dois pedidos são a principal fonte de dificuldade.

## Limites de uso e limitações

- O corpus é sintético; nenhuma conclusão deste projeto deve ser estendida a
  pessoas, setores ou instituições reais.
- O sistema não deve resolver a solicitação, responder ao usuário nem inferir
  sentimento ou urgência — apenas indicar o setor de destino.
- Casos ambíguos devem ser sinalizados para revisão humana, não decididos
  automaticamente com falsa confiança.
- A extensão é sintética, tem só 3 pares e nenhum par de controle, então não testa
  a concentração por par.
- O estilo da extensão é diferente do corpus-base (0,683 contra 0,951); parte da
  queda pode vir daí.
- Os 3 pares foram escolhidos depois de ver os erros da validação.
- Poucos erros por par, um classificador, uma partição e nenhum intervalo de
  confiança.
- Os rótulos dos 18 casos sem marcador seguem uma regra do grupo (primeiro
  pedido), então são ambíguos por natureza.

## Como executar

1. Colocar `P03_solicitacoes_atendimento_v2.csv` e `P03_extensao_TrilhaA.csv` na
   mesma pasta do notebook `M16_P03_final.ipynb`.
2. Instalar: `pip install pandas numpy matplotlib scikit-learn`.
3. Abrir o notebook e usar "Run all". Não há novo `fit` depois da validação: o
   modelo B fica congelado.

Os notebooks M14 e M15 (etapa inicial) usam o corpus de 240 registros e podem ser
executados da mesma forma, com o CSV original na mesma pasta.

## Estrutura do repositório

```
/
├── README.md
├── Done_M05.docx
├── LEIA-ME.txt
├── M04_Instrumento_preenchido.docx
├── P03_solicitacoes_atendimento.csv        (corpus inicial, 240 registros, preservado)
├── P03_solicitacoes_atendimento_v2.csv     (corpus-base do M4, 5.000 textos, sem alteração)
├── P03_extensao_TrilhaA.csv                (extensão própria, 120 exemplos)
├── M14_Representacao_Textual_LGN_P03_executado.ipynb
├── M15_Notebook_de_Baseline_e_Avaliacao_Inicial_P03_executado.ipynb
└── M16_P03_final.ipynb                     (comparação A×B, extensão e teste final)
```

## Declaração de uso de IA

O assistente de IA Claude (Anthropic) apoiou a organização, a execução e a
documentação dos notebooks M14 e M15 e deste README, sob revisão e validação do
grupo. No M4, o Claude gerou os 120 exemplos da extensão a partir de instruções
do grupo (coluna `origem`); a revisão dos rótulos pelo grupo está na coluna
`revisado_pelo_grupo`. O teste só foi usado depois de congelar a configuração B.
