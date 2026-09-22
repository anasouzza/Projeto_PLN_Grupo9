# Projeto avaliativo da disciplina Linguagem Natural

## Do problema ao pipeline executável, avaliado e criticamente documentado

| Informação   | Descrição                                                                 |
| ------------ | -------------------------------------------------------------------------- |
| Universidade | *[completar]*                                                              |
| Curso        | *[completar]*                                                              |
| Disciplina   | Linguagem Natural                                                          |
| Professor    | *[completar]*                                                              |
| Projeto      | P03 — Classificação de solicitações de atendimento por setor de destino  |
| Grupo        | *[completar código/nome do grupo]*                                         |
| Integrantes  | Ana Clara de Souza, Leticia Vieira, Mariana Garcez                        |

## Apresentação

Este pacote reúne as evidências produzidas pelo grupo até a etapa de baseline e avaliação inicial do projeto P03. A documentação foi organizada para permitir a rastreabilidade entre a formulação do problema, os dados utilizados, a representação textual, o primeiro modelo supervisionado e os resultados observados.

A pergunta central do projeto é:
> Qual setor da instituição deve receber inicialmente esta solicitação de atendimento?

A unidade de análise é uma mensagem completa enviada por um usuário (e-mail, formulário, chat ou WhatsApp). O campo `texto` é a entrada principal e `setor_destino` é o alvo de classificação, com cinco categorias: `financeiro`, `secretaria_academica`, `suporte_tecnico`, `biblioteca` e `coordenacao`.

O resultado tem como finalidade apoiar o encaminhamento inicial de cada solicitação ao setor responsável, evitando uma nova triagem manual e reduzindo o tempo de resposta. O projeto não pretende resolver a solicitação, responder ao usuário, nem inferir sentimento ou urgência — casos ambíguos ou com evidência insuficiente devem ser sinalizados para revisão humana, não decididos automaticamente.

---

## Passos a concluir

O projeto é progressivo. As etapas já executadas neste pacote não representam o encerramento da investigação.

Ainda deverão ser realizadas as etapas previstas para a continuidade do projeto:

1. Analisar as evidências de módulos seguintes para identificar casos relevantes de erro, divergência ou ausência de diferença entre abordagens.
2. Investigar a diferença metodológica entre M14 (TF-IDF sobre texto preparado) e M15 (TF-IDF sobre texto original) descrita na Seção 7 abaixo.
3. Formular uma hipótese e uma questão experimental para a investigação própria do grupo.
4. Criar e documentar uma extensão própria com novos exemplos rotulados.
5. Executar o experimento de investigação própria e analisar seus resultados e erros.
6. Manter o conjunto de teste original protegido até que as decisões estejam congeladas.
7. Consolidar notebook final, corpus final ampliado, README final e documentação do uso de Inteligência Artificial.
8. Preparar a apresentação do grupo e a defesa individual.

---

## 1. Relação entre os arquivos deste pacote

| Arquivo                                          | Função                                                                                 | Relação com as demais etapas                                                                   |
| ------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `M05_Ficha_Inicial_Projeto_Integrador_LGN_v2.docx` | Formulação inicial do problema, pergunta, unidade, entrada, saída, critérios e limites | Define o que o projeto pretende responder antes da escolha da abordagem técnica                |
| `P03_solicitacoes_atendimento.csv`                 | Corpus didático original com 240 registros                                             | É a base efetivamente utilizada nos notebooks M14 e M15 incluídos neste pacote                 |
| `P03_solicitacoes_atendimento_v2.csv`              | Corpus-base v2 com 5000 registros                                                       | Está preservado como versão ampliada para a continuidade formal do projeto                     |
| `M14_Representacao_Textual_LGN_P03_executado.ipynb`| Representação textual                                                                  | Compara BoW, unigramas, bigramas e TF-IDF e registra o laboratório de decisão de representação |
| `M15_Notebook_de_Baseline_e_Avaliacao_Inicial_P03_executado.ipynb` | Baseline, primeiro modelo e avaliação inicial                          | Usa TF-IDF com unigramas e LogisticRegression, compara com a referência majoritária e avalia na validação |
| `README.md`                                        | Documentação e rastreabilidade                                                          | Relaciona os arquivos, decisões, resultados, limitações e próximos passos                      |

### Fluxo documental

```
M05_Ficha_Inicial_Projeto_Integrador_LGN_v2.docx
        ↓
Definição do problema e da tarefa
        ↓
P03_solicitacoes_atendimento.csv
        ↓
M14_Representacao_Textual_LGN_P03_executado.ipynb
Representação textual
        ↓
M15_Notebook_de_Baseline_e_Avaliacao_Inicial_P03_executado.ipynb
Baseline + LogisticRegression + avaliação
        ↓
README.md
Documentação consolidada

P03_solicitacoes_atendimento_v2.csv
        ↓
Corpus-base ampliado e preservado para continuidade do projeto
```

---

## 2. Definição do problema — M05

O documento `M05_Ficha_Inicial_Projeto_Integrador_LGN_v2.docx` registra a etapa inicial do projeto. O problema identificado foi o grande volume de solicitações de atendimento chegando de forma desorganizada por diferentes canais de comunicação, gerando demora na identificação e classificação dos atendimentos.

O usuário direto é o solicitante; a equipe responsável pelo recebimento das mensagens e os setores responsáveis pelo atendimento das demandas também usam o resultado. A instituição pode ser afetada indiretamente por atrasos ou encaminhamentos incorretos. Os dados utilizados são sintéticos e não correspondem a pessoas reais.

### 2.1 Pergunta operacional
> Qual setor da instituição deve receber inicialmente esta solicitação de atendimento?

### 2.2 Unidade de análise
Uma mensagem completa enviada por um usuário (e-mail, formulário, chat ou WhatsApp).

### 2.3 Entrada e saída
- Entrada: `texto` (apenas o texto da mensagem da solicitação)
- Saída: `setor_destino`

Categorias:
- `financeiro`
- `secretaria_academica`
- `suporte_tecnico`
- `biblioteca`
- `coordenacao`

### 2.4 Papel dos campos

| Campo                   | Papel                                                              |
| ------------------------ | -------------------------------------------------------------------- |
| `id`                    | Identificação e controle; não é feature                            |
| `texto`                 | Entrada textual principal                                          |
| `setor_destino`         | Target; nunca deve ser utilizado como entrada                      |
| `canal`                 | Metadado opcional; não utilizado inicialmente                      |
| `urgencia_referencia`   | Metadado de referência; fora do escopo desta etapa (não inferir urgência) |
| `particao_recomendada`  | Controle experimental de treino, validação e teste; não é feature  |

### 2.5 Casos ambíguos ou sem texto

Quando uma mensagem pertence a mais de um setor ou não tem informação suficiente, ela deve ser associada ao setor mais provável no contexto da mensagem; a revisão humana permanece necessária para os casos de baixa confiança. Textos ausentes ou sem informação suficiente não recebem automaticamente uma categoria.

### 2.6 Uso do resultado e fora do escopo

O resultado deve apoiar o encaminhamento automático da solicitação ao setor responsável, prevendo revisão humana quando necessária. Está fora do escopo: resolver a solicitação, responder ao usuário, ou identificar sentimento e urgência.

O erro mais grave identificado é encaminhar a solicitação a um setor errado, especialmente entre setores com escopos próximos, como `secretaria_academica` e `coordenacao`, pois gera atraso no atendimento e retrabalho de redirecionamento.

---

## 3. Dados e rastreabilidade das versões

### 3.1 `P03_solicitacoes_atendimento.csv`

O corpus didático original contém:

- 240 registros;
- 141 registros de treino, 49 de validação e 50 de teste;
- 3 textos ausentes;
- 3 duplicações textuais adicionais entre os textos não ausentes;
- distribuição por setor: `financeiro` (57), `secretaria_academica` (57), `suporte_tecnico` (50), `biblioteca` (39), `coordenacao` (37).

Esta é a versão utilizada pelos notebooks M14 e M15 presentes neste pacote.

### 3.2 `P03_solicitacoes_atendimento_v2.csv`

O corpus-base v2 contém:

- 5000 registros;
- 3500 registros de treino, 750 de validação e 750 de teste;
- 30 textos ausentes;
- 50 duplicações textuais adicionais entre os textos não ausentes;
- distribuição por setor: `financeiro` (1190), `secretaria_academica` (1190), `suporte_tecnico` (1040), `biblioteca` (810), `coordenacao` (770).

O v2 está incluído e preservado para a continuidade do projeto.

### 3.3 Nota de versão

Os resultados quantitativos apresentados neste README para M14 e M15 foram produzidos com o corpus didático de 240 registros, porque essa é a base efetivamente carregada nos notebooks incluídos.

O corpus v2 de 5.000 registros está preservado no pacote, mas os resultados de M14 e M15 não devem ser atribuídos a ele sem uma nova execução explicitamente registrada.

---

## 4. Preparação textual recuperada no M14

O M14 recupera as decisões de preparação registradas no Passo 9 do notebook e utiliza a seguinte configuração:

```
USAR_MINUSCULAS = True
REMOVER_STOPWORDS = False
PRESERVAR_NEGACAO = True
USAR_STEMMING = False
```

Como a remoção de stopwords está desativada, a decisão de preservar negações não tem efeito prático nesta execução — ela só passaria a valer se `REMOVER_STOPWORDS` fosse ativado. A preparação foi tratada como parte do experimento, não como uma "limpeza automática" do texto.

---

## 5. Representação textual — M14

O notebook `M14_Representacao_Textual_LGN_P03_executado.ipynb` investiga como transformar solicitações de atendimento em características numéricas antes do treinamento de um classificador, primeiro com três documentos de demonstração e depois com o corpus real do grupo.

### 5.1 Bag of Words

O Bag of Words representa ocorrência e frequência de termos. É simples e interpretável, porém perde grande parte da ordem das palavras.

### 5.2 Unigramas e bigramas

No notebook executado, sobre os 139 registros de treino com texto válido:

| Representação        | Documentos | Características |
| --------------------- | ---------- | ---------------- |
| BoW original          | 139        | 228               |
| BoW preparado         | 139        | 164               |
| Unigramas             | 139        | 164               |
| Unigramas + bigramas  | 139        | 572               |
| TF-IDF preparado      | 139        | 164               |

A preparação reduziu o vocabulário do BoW de 228 para 164 características (-64), efeito da conversão para minúsculas. Essa redução não foi interpretada automaticamente como melhora.

A adição de bigramas aumentou a representação de 164 para 572 características, acrescentando 408 features. Entre os exemplos observados no vocabulário de treino estão `abre mas`, `abrir as`, `acadêmica sobre` e `acadêmico desde`.

### 5.3 TF-IDF

No M14, o TF-IDF foi construído somente com os 139 textos válidos de treino e produziu uma matriz de dimensão `139 × 164`.

Entre os termos com maior peso nos primeiros documentos inspecionados apareceram `confirmada`, `disciplina`, `bolsa`, `desconto`, `reservar` e `saber`. Esses pesos foram utilizados apenas para interpretar a representação; não foram tratados como evidência de que esses termos seriam os melhores preditores das classes.

### 5.4 Decisão inicial do M14

Ao final do M14, o grupo registrou o TF-IDF sobre o texto preparado como representação inicial mais promissora.

**Representação que parece mais promissora:** TF-IDF sobre o texto preparado.

**Evidência observada:** o TF-IDF preparado manteve a mesma dimensionalidade do BoW preparado (139 documentos × 164 características) e, ao contrário da contagem simples, atribuiu pesos diferentes aos termos conforme sua distribuição no corpus de treino (por exemplo, `confirmada`, `disciplina`, `bolsa` e `reservar` receberam destaque nos documentos inspecionados).

**Principal vantagem:** pondera a importância relativa dos termos em vez de tratá-los todos igualmente, ajudando a distinguir termos característicos de cada mensagem sem aumentar a dimensionalidade como os bigramas fizeram (164 → 572).

**Principal limitação:** ainda perde grande parte da ordem das palavras e depende do vocabulário construído a partir do treino — não captura contexto local como os bigramas.

**O que ainda precisa ser comparado na próxima aula:** o desempenho de um classificador treinado sobre essa representação preparada, já que o M15 mediu TF-IDF apenas sobre o texto original (não o `texto_preparado`) — essa comparação direta ainda não foi feita.

> Esta decisão é inicial. O M14 não treinou classificador nem calculou métricas de desempenho.

Independentemente disso, o notebook confirma:
- corpus bruto preservado;
- desenvolvimento realizado apenas com treino;
- conteúdo do teste não utilizado para ajustes.

---

## 6. Baseline e avaliação inicial — M15

O notebook `M15_Notebook_de_Baseline_e_Avaliacao_Inicial_P03_executado.ipynb` introduz o primeiro experimento supervisionado do projeto.

Após remover 3 registros sem texto do corpus completo (240 → 237 registros válidos), a divisão observada foi: 139 de treino, 49 de validação e 49 de teste (preservado, apenas contado).

O fluxo utilizado foi:

```
texto
↓
TF-IDF com unigramas
↓
LogisticRegression
↓
setor previsto
↓
avaliação na validação
```

### 6.1 Referência majoritária

A baseline repete sempre a classe mais frequente no treino e não utiliza o conteúdo textual.

Distribuição do treino: `financeiro` (34), `secretaria_academica` (31), `suporte_tecnico` (29), `biblioteca` (24), `coordenacao` (21). A resposta mais frequente foi `financeiro`.

Resultados na validação:

| Métrica  | Referência majoritária |
| -------- | ------------------------ |
| Acurácia | 0,245                    |
| F1-macro | 0,079                    |

### 6.2 Representação utilizada

O M15 utilizou:

```
TfidfVectorizer(ngram_range=(1, 1))
```

diretamente sobre a coluna `texto` original (não sobre o `texto_preparado` gerado no M14).

As matrizes observadas foram:

- treino: `139 × 164`;
- validação: `49 × 164`.

O vetorizador foi ajustado somente no treino, e a validação recebeu apenas `transform`. Nenhuma matriz de teste foi criada.

### 6.3 Classificador supervisionado

O modelo utilizado foi:

```
LogisticRegression(max_iter=1000, random_state=42)
```

Ele recebeu as features TF-IDF do treino e os rótulos conhecidos para aprender relações estatísticas entre representação textual e setor de destino.

### 6.4 Resultados observados

| Abordagem                     | Acurácia | F1-macro |
| ------------------------------- | -------- | -------- |
| Referência majoritária          | 0,245    | 0,079    |
| LogisticRegression com TF-IDF   | 1,000    | 1,000    |

Como a acurácia na validação foi de 1,000, todos os 49 exemplos ficaram na diagonal principal da matriz de confusão, distribuídos conforme a composição da validação:

- financeiro: 12 acertos;
- secretaria_academica: 11 acertos;
- suporte_tecnico: 10 acertos;
- biblioteca: 9 acertos;
- coordenacao: 7 acertos.

Nenhuma confusão entre setores foi observada nesta partição.

### 6.5 Interpretação

Nesta validação, o classificador supervisionado foi consideravelmente melhor que a referência majoritária: acurácia de 1,000 contra 0,245, e F1-macro de 1,000 contra 0,079. A referência simples erra porque sempre responde `financeiro`; o classificador aprendeu a diferenciar os setores a partir das palavras de cada mensagem.

Esse resultado deve ser interpretado com cautela, pois o corpus é sintético e pequeno. Ele não permite afirmar que o modelo terá o mesmo desempenho com mensagens reais, mais confusas, nem que generalizará no teste — que ainda não foi utilizado. O microexperimento de vazamento permaneceu desativado (`EXECUTAR_TEMPORARIO = False`), o TF-IDF foi ajustado somente no treino e o conjunto de teste continuou protegido.

---

## 7. Continuidade entre M14 e M15

Os notebooks representam etapas diferentes do pipeline e possuem uma diferença metodológica que precisa permanecer documentada.

No M14, a representação de treino foi construída sobre `texto_preparado` (minúsculas aplicadas manualmente, `lowercase=False` no vetorizador, tokenização própria), resultando em 164 características.

No M15, o TF-IDF foi construído diretamente sobre a coluna `texto` original, usando `TfidfVectorizer(ngram_range=(1, 1))` com sua tokenização e minúsculas padrão — e também resultou em 164 características na matriz de treino.

A coincidência no número de características (164 em ambos) **não significa que as duas representações sejam idênticas**: os pipelines de tokenização e normalização são diferentes, e essa igualdade não foi verificada termo a termo em nenhum dos notebooks. Além disso, como o M15 mediu o TF-IDF sobre o texto original e não sobre o `texto_preparado` decidido no M14, os resultados do M15 não comprovam diretamente que a representação escolhida no M14 é melhor ou pior.

Essa diferença pode ser utilizada como ponto de investigação futura caso esteja alinhada às evidências e à trilha escolhida pelo grupo.

---

## 8. Pipeline construído até o momento

```
Definição do problema
↓
Pergunta operacional
↓
Corpus (P03)
↓
Inspeção dos dados
↓
Preparação textual
↓
Representação numérica
↓
Baseline majoritária
↓
TF-IDF
↓
LogisticRegression
↓
Validação
↓
Acurácia + F1-macro + matriz de confusão
↓
Interpretação
```

Esse pipeline representa o estágio inicial executável e avaliado do projeto. A investigação própria, a extensão do corpus, a análise aprofundada de erros e a avaliação final ainda pertencem às próximas etapas.

---

## 9. Separação entre treino, validação e teste

### Treino
Utilizado para aprender vocabulário, estatísticas da representação e parâmetros do classificador.

### Validação
Utilizada para produzir previsões, calcular métricas e interpretar o comportamento da abordagem durante o desenvolvimento.

### Teste
Permanece reservado para a etapa final, depois que as decisões forem congeladas. Nesta etapa, o teste foi usado apenas para conferência de contagem (49 registros), nunca transformado, previsto ou avaliado.

---

## 10. Limitações atuais

- os dados são sintéticos e didáticos;
- os resultados não representam solicitantes ou setores reais;
- existem 3 textos ausentes e 3 duplicações no corpus original;
- algumas solicitações podem envolver mais de um setor ou ter evidência insuficiente;
- uma única validação não garante comportamento futuro;
- o teste ainda não foi utilizado para avaliação;
- unigramas perdem grande parte da ordem das palavras;
- M14 e M15 constroem o TF-IDF a partir de colunas de texto diferentes (preparado vs. original), o que não deve ser tratado como equivalente apenas porque produziu o mesmo número de características;
- os resultados dos notebooks de 240 registros não devem ser apresentados como resultados do corpus v2 de 5.000 registros.

---

## 11. Reprodutibilidade

O pacote preserva as evidências necessárias para acompanhar o caminho experimental:

- corpus didático original (`P03_solicitacoes_atendimento.csv`);
- corpus-base v2 separado (`P03_solicitacoes_atendimento_v2.csv`);
- M05 com a formulação do problema;
- M14 executado com outputs de representação;
- M15 executado com baseline, modelo, métricas e interpretação;
- README relacionando arquivos, decisões e resultados.

Para reproduzir os notebooks deste pacote, os arquivos devem permanecer no mesmo diretório, e o corpus lógico esperado pelos notebooks é `P03_solicitacoes_atendimento.csv` (delimitador `;`, codificação UTF-8 com BOM).

As principais bibliotecas observadas são:

- pandas;
- matplotlib;
- scikit-learn;
- ambiente Jupyter Notebook compatível.

A execução deve respeitar a ordem das células, pois etapas posteriores dependem das variáveis criadas anteriormente.

---

## 12. Status atual

| Componente                         | Situação       |
| ------------------------------------- | --------------- |
| Formulação do problema — M05        | Concluída       |
| Corpus didático original            | Preservado      |
| Corpus-base v2                      | Preservado      |
| Representação textual — M14         | Concluída       |
| Baseline e avaliação inicial — M15  | Concluída       |
| README da etapa atual               | Consolidado     |
| Investigação própria do grupo       | A concluir      |
| Extensão própria do corpus          | A concluir      |
| Teste final                         | A concluir      |
| README final                        | A concluir      |
| Apresentação e defesa               | A concluir      |

---

## 13. Observação final de rastreabilidade

Este README foi construído para manter explícita a relação entre cada arquivo do pacote e a evidência que ele fornece.

Em particular:

- `M05_Ficha_Inicial_Projeto_Integrador_LGN_v2.docx` documenta a formulação inicial da tarefa;
- `P03_solicitacoes_atendimento.csv` é o corpus associado aos resultados atuais de M14 e M15;
- `P03_solicitacoes_atendimento_v2.csv` é a versão ampliada preservada para continuidade;
- `M14_Representacao_Textual_LGN_P03_executado.ipynb` documenta preparação, representação e a decisão inicial de representação;
- `M15_Notebook_de_Baseline_e_Avaliacao_Inicial_P03_executado.ipynb` documenta baseline, primeiro classificador e avaliação;
- `README.md` consolida a rastreabilidade entre problema, dados, método, resultados, limites e próximos passos.
