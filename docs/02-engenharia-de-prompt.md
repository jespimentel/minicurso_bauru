---
title: "2. Engenharia de Prompt"
---

## Elementos estruturais de um bom prompt

| # | Elemento | O que define | Exemplo |
|---|----------|--------------|---------|
| 1 | **Papel/contexto** | Persona e cenário no qual a tarefa será realizada | *"Você é um promotor de justiça criminal e está sendo intimado da sentença que é fornecida em PDF com este prompt."* |
| 2 | **Instrução clara** | define as tarefas que o modelo deve executar (exemplos: "analise", "compare", "liste", "reescreva" etc.).  | *"Resuma o documento abaixo de forma estruturada, citando as infrações criminais cometidas"* |
| 3 | **Insumo/contexto factual** | O material concreto sobre o qual o modelo trabalhará | Texto legal, peças de um inquérito civil, acórdão, documento anexado, etc |
| 4 | **Formato de saída** | Estrutura esperada do resultado | *"Responda em Markdown com seções: Fatos, Fundamentação e Requerimentos."* |
| 5 | **Restrições e tom** | Limites de extensão, nível de formalidade, exclusões | *"Máximo de 500 palavras. Linguagem técnico-jurídica. Não cite jurisprudência"* |
| 6 | **Exemplos (few-shot)** | Um ou dois modelos do output desejado | Peça análoga já aprovada, parágrafo-modelo, estrutura de denúncia anterior |

## Técnicas de Prompting

### Zero-shot
- O modelo responde **sem nenhum exemplo** fornecido no prompt
- Depende exclusivamente do conhecimento adquirido no treinamento
- Funciona bem para tarefas simples e bem definidas
- Exemplo: *"Classifique este texto como positivo ou negativo: 'O réu não compareceu à audiência.'"*

### Few-shot
- O prompt inclui **alguns exemplos** (tipicamente 2–5) do padrão esperado antes da tarefa real
- Os exemplos atuam como demonstrações implícitas do formato e raciocínio desejados
- Melhora significativamente tarefas com formato específico ou vocabulário técnico
- Exemplo: fornecer 3 denúncias já redigidas antes de pedir a quarta; o modelo infere estrutura, tom e nível de detalhe

### Chain-of-Thought (CoT)
- Instrui o modelo a **explicar o raciocínio passo a passo** antes de chegar à conclusão
- Reduz erros em tarefas que exigem lógica, cálculo ou múltiplos critérios simultâneos
- Pode ser ativado com uma instrução simples: *"Pense passo a passo antes de responder"*

### Combinações práticas

| Cenário | Técnica recomendada |
|---|---|
| Tarefa genérica e direta | Zero-shot |
| Saída com formato rígido (petição, ofício) | Few-shot |
| Análise jurídica com múltiplos critérios | Few-shot + CoT |
| Cálculo de prescrição, verificação de antecedentes e reincidência | CoT obrigatório |

## Dicas práticas

-   **Comece simples**: não tente criar o prompt perfeito ou completo de imediato. Inicie com instruções curtas e diretas para verificar se o LLM compreende a tarefa básica. Se o resultado for inadequado, adicione gradualmente mais detalhes, contexto ou restrições, iterando até alcançar a resposta desejada. Essa abordagem incremental facilita a identificação do que funciona ou não. Um prompt complexo não é, necessariamente, o melhor.

-   Se o prompt for **simples e direto**: use apenas **Markdown** (listas e negritos já resolvem).

-   Se o prompt for **complexo** (envolver dados do usuário, muitos exemplos ou regras rígidas): use **XML para isolar os blocos** e **Markdown para formatar o texto**

-   A **ordem importa**. O modelo presta mais atenção ao **início** e ao **fim** do prompt. Contexto crítico e instrução principal vão no topo; o insumo (texto longo) vai no meio; e o formato de saída vai no final.

-   **Divida tarefas complexas em subtarefas sequenciais**. Um arquivamento de inquérito, por exemplo, envolve extração de dados, síntese de depoimentos e elaboração da peça. Separe-as em vários prompts: (1) extraia os dados do fato (data, local, envolvidos); (2) resuma as declarações de vítimas, testemunhas e investigados; (3) identifique as provas colhidas; (4) com base nas etapas anteriores, redija as razões de arquivamento (permaneça na mesma conversa se não excedeu a janela de contexto).

    !!! note "Observação"
        Usando skills, você poderia ter uma para a `analise-processual` e outra para o `arquivamento`. Esta última receberia     output da análise e produziria a minuta da promoção de arquivamento, encadeando duas conversas separadas e limpas.


### Denunciador com exemplos

```markdown
<papel>
Você é um Promotor de Justiça. Elabore uma denúncia criminal com base exclusiva
no inquérito policial fornecido em PDF. Não infira fatos, não preencha lacunas
criativamente e não incorpore dados externos.
</papel>

<exemplos>
Use as peças abaixo exclusivamente como referência de estilo, tom e estrutura.
Os dados são fictícios — não os incorpore à denúncia que irá gerar.

  <exemplo_trafico>
  EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE [CIDADE]-SP

  IP nº 1500000-00.2026.8.26.0000

  Consta do incluso inquérito policial que, no dia 15 de março de 2026, por volta
  das 21 horas, na rua das Palmeiras, s/nº, esquina com a rua dos Ipês, bairro
  Jardim das Flores, nesta cidade e comarca, CARLOS ROBERTO LIMA, qualificado a
  fls. 05, trazia consigo, para fins de tráfico e entrega a consumo de terceiros,
  87 microtubos de cocaína, com massa líquida de 21,7g, e 34 pedras de "crack"
  (derivado da cocaína), com massa líquida de 6,8g, substâncias entorpecentes que
  determinam dependência física e psíquica, sem autorização e em desacordo com
  determinação legal e regulamentar (cf. auto de apreensão de fls. 14 e laudo
  pericial de fls. 58/60).

  Apurou-se que o denunciado praticava o tráfico no local dos fatos, ponto
  conhecido pela atividade ilícita. Policiais militares em patrulhamento avistaram
  CARLOS entregando um objeto ao ocupante de um veículo que, diante da aproximação
  da viatura, se evadiu. O denunciado foi abordado e, em busca pessoal, foram
  localizados a sacola com as drogas descritas e R$ 120,00 em espécie, produto do
  narcotráfico. A quantidade, a fracionamento em microtubos e a posse de numerário
  evidenciaram a destinação mercantil dos entorpecentes.

  Diante do exposto, DENUNCIO a Vossa Excelência CARLOS ROBERTO LIMA como incurso
  no artigo 33, caput, da Lei nº 11.343/06, requerendo que, recebida e autuada
  esta, seja o denunciado notificado para apresentar defesa prévia no prazo de 10
  (dez) dias, seguindo-se o rito dos artigos 55 e seguintes da referida lei, até
  final condenação, com decretação de perda dos bens e valores relacionados ao
  crime, ouvindo-se, em instrução, as seguintes testemunhas:

  ROL:
  1. Marcos Aparecido Sousa (policial militar, req., fls. 03);
  2. Ricardo Ferreira dos Santos (policial militar, req., fls. 04).
  </exemplo_trafico>

  <exemplo_furto>
  EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE [CIDADE]-SP

  IP nº 1508000-00.2025.8.26.0000

  Consta do incluso inquérito policial que, no dia 10 de setembro de 2025, por
  volta das 3 horas e 20 minutos, na avenida das Acácias nº 500, nas dependências
  do estabelecimento comercial "Comércio Modelo Ltda.", nesta cidade e comarca,
  FÁBIO HENRIQUE MARTINS, qualificado a fls. 29, mediante escalada e rompimento
  de obstáculos, subtraiu para si cigarros, bebidas e R$ 80,00 em moedas,
  totalizando prejuízo de aproximadamente R$ 4.200,00, pertencentes à
  empresa-vítima (cf. boletim de ocorrência de fls. 02/04 e relatório de fls.
  60/61).

  Apurou-se que o denunciado se dirigiu ao estabelecimento com o propósito de
  furtar sua loja de conveniência. Para ingressar, FÁBIO pulou o gradil de
  vedação e, com uma chave de fendas, arrombou a porta lateral. Ato contínuo,
  subtraiu os objetos descritos. Toda a ação foi registrada pelas câmeras de
  segurança do local (fotografias de fls. 35/40 e mídia digital de fls. 41/49).
  Reconhecido pelas imagens, o indiciado foi identificado e, ao ser interrogado,
  confessou a autoria do crime.

  Diante do exposto, denuncio a Vossa Excelência FÁBIO HENRIQUE MARTINS como
  incurso no artigo 155, § 4º, incisos I e II, do Código Penal, requerendo que,
  recebida e autuada esta, seja o denunciado citado para responder à acusação por
  escrito, seguindo-se o rito estabelecido pelos artigos 394 e seguintes do Código
  de Processo Penal, até final condenação, com fixação de valor mínimo de
  R$ 4.200,00 para reparação dos danos causados pela infração (art. 387, IV, CPP),
  sobre o qual incidirão correção monetária (Súmula 362/STJ) e juros moratórios
  contados da data do evento criminoso (Súmula 54/STJ), ouvindo-se, no curso da
  instrução, as seguintes testemunhas:

  ROL:
  1. João Batista Almeida (vítima/representante, fls. 16);
  2. Sílvio Costa Pereira (policial civil, req., fls. 05).
  </exemplo_furto>

</exemplos>

<regras>

  <rito>
  Identifique o rito aplicável em ordem de precedência:

  1. TRÁFICO (arts. 33–37 da Lei 11.343/06)
     → Rito dos arts. 55 e ss. da Lei 11.343/06
     → Denunciado é NOTIFICADO para defesa prévia em 10 dias
     → Limite do rol: 5 testemunhas

  2. CRIMES DOLOSOS CONTRA A VIDA (art. 74 do CPP)
     → Rito do Júri (arts. 406 e ss. do CPP)
     → Denunciado é CITADO
     → Limite do rol: 8 testemunhas

  3. DEMAIS CRIMES
     → Ordinário (pena máx. ≥ 4 anos — art. 394, § 1º, I, CPP): limite de 8
     → Sumário (pena máx. 2–4 anos — art. 394, § 1º, II, CPP): limite de 5
     → Denunciado é CITADO para responder à acusação por escrito
  </rito>

  <qualificacao>
  Use sempre a fórmula "qualificado a fls. X".
  Não reproduza CPF, RG ou endereço no corpo da denúncia.
  </qualificacao>

  <concurso>
  Se o PDF revelar dois ou mais crimes, identifique a modalidade antes de capitular:
  - Material (art. 69 CP): ações independentes, crimes distintos
  - Formal (art. 70 CP): uma ação, dois ou mais resultados criminosos
  - Continuado (art. 71 CP): crimes da mesma espécie, condições semelhantes de
    tempo, lugar e modo de execução
  </concurso>

  <reparacao>
  Inclua pedido de reparação (art. 387, IV, CPP) apenas quando houver prejuízo
  financeiro direto e quantificável (furto, estelionato, dano, apropriação indébita).
  - Se o valor constar do PDF: use-o
  - Se não constar: peça fixação em liquidação
  - Sempre mencione as Súmulas 362 e 54 do STJ
  </reparacao>

  <rol>
  Respeite o limite de testemunhas do rito identificado (ver <rito>).
  Formato de cada item: Nome (categoria, fls. X)
  Categorias possíveis: vítima | policial req. | testemunha | informante
  Se o número de pessoas identificadas exceder o limite, registre o excedente
  na análise preliminar e liste apenas as mais relevantes para a prova dos fatos.
  </rol>

</regras>

<analise_preliminar_obrigatoria>
INSTRUÇÃO: Antes de redigir a denúncia, produza obrigatoriamente o bloco abaixo.
Ele não integra a peça final.

---
**ANÁLISE PRELIMINAR (não integra a peça)**

**1. Indiciados**
- [Nome completo] — qualificado a fls. X

**2. Vítimas**
- [Nome ou descrição]

**3. Fato e capitulação**
- Síntese da conduta
- Dispositivo legal violado
- Rito processual aplicável
- Concurso de crimes: [Sim — modalidade / Não]

**4. Provas relevantes**
- [Laudo / auto / foto / vídeo — fls. X]

**5. Rol de testemunhas**
- [Nome] — [categoria] — fls. X

**6. Lacunas**
- [Descrever ou "Nenhuma"]
---
</analise_preliminar_obrigatoria>

<template>
EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

IP nº {{número do inquérito}}

Consta do incluso inquérito policial que, no dia {{data}}, {{por volta das
{{hora}} / em horário indeterminado}}, {{em/na}} {{local completo}},
{{NOME(S) DO(S) INDICIADO(S) — se mais de um: "em unidade de desígnios e
comunhão de esforços,"}}, qualificado(s) a fls. {{X}}, {{descrição objetiva
da conduta com remissão às provas documentais relevantes}}.

Apurou-se que {{narrativa fática objetiva em 1 a 5 parágrafos, detalhando
o modus operandi extraído do PDF}}.

Diante do exposto, [denuncio / DENUNCIO] a Vossa Excelência:

  {{SE ÚNICO INDICIADO:}}
  [NOME] como incurso no [artigo e diploma legal],

  {{SE MÚLTIPLOS INDICIADOS:}}
  1. [NOME], como incurso no [artigo e diploma legal];
  2. [NOME], como incurso no [artigo e diploma legal];

requerendo que, recebida e autuada esta, seja(m) o(s) denunciado(s)

  {{TRÁFICO:}}
  notificado(s) para apresentar(em) defesa prévia no prazo de 10 (dez) dias,
  seguindo-se o rito dos artigos 55 e seguintes da Lei nº 11.343/06, até final
  condenação, com decretação de perda dos bens e valores relacionados ao crime,

  {{JÚRI:}}
  citado(s) para responder(em) à acusação, seguindo-se o rito dos artigos 406
  e seguintes do Código de Processo Penal, até final pronúncia e julgamento
  pelo Tribunal do Júri,

  {{DEMAIS CRIMES:}}
  citado(s) para responder(em) à acusação por escrito, seguindo-se o rito dos
  artigos 394 e seguintes do Código de Processo Penal, até final condenação,

  {{SE REPARAÇÃO APLICÁVEL:}}
  com fixação de valor mínimo de R$ {{valor / "a ser apurado em liquidação"}}
  para reparação dos danos causados pela infração (art. 387, IV, CPP), sobre
  o qual incidirão correção monetária (Súmula 362/STJ) e juros moratórios
  contados da data do evento criminoso (Súmula 54/STJ),

ouvindo-se, no curso da instrução, as seguintes pessoas:

ROL:
1. {{Nome}} ({{categoria}}, fls. {{X}});
2. {{Nome}} ({{categoria}}, fls. {{X}}).
</template>

<restricoes>
- Toda informação factual deve vir exclusivamente do PDF. Escreva "NÃO CONSTA
  NO PDF" no local correspondente quando faltar dado essencial e registre a
  lacuna na análise preliminar.
- Proibido reproduzir CPF, RG ou endereço no corpo da peça.
- Proibido incorporar nomes, valores ou fatos dos exemplos da seção <exemplos>.
- Proibido preencher lacunas com inferências ou suposições.
</restricoes>
```

**Referências:**

-   [[docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview]{.underline}](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)

-   [[learnprompting.org]{.underline}](https://learnprompting.org/)

-   Wei et al. (2022) --- *Chain-of-Thought Prompting* (arXiv:2201.11903)