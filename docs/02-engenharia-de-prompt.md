---
title: "2. Engenharia de Prompt"
---
## O que é Engenharia de Prompt?

Engenharia de prompt é o conjunto de práticas para formular instruções que aumentam a precisão e a previsibilidade das respostas de uma LLM. Um prompt eficaz combina elementos estruturais (papel, restrições, exemplos, insumo factual, instrução e formato de saída) com a técnica de prompting adequada à tarefa: zero-shot para pedidos simples, few-shot quando o formato precisa seguir um padrão rígido (como uma denúncia) e chain-of-thought quando o raciocínio envolve múltiplos critérios ou cálculos, como prescrição e reincidência.

## Elementos estruturais de um bom prompt

| # | Elemento | O que define | Exemplo |
|---|----------|--------------|---------|
| 1 | **Papel/contexto** | Persona e cenário no qual a tarefa será realizada | *"Você é um promotor de justiça criminal e está sendo intimado da sentença fornecida em PDF."* |
| 2 | **Restrições e tom** | Limites de extensão, nível de formalidade, exclusões | *"Máximo de 500 palavras. Linguagem técnico-jurídica. Não cite jurisprudência."* |
| 3 | **Exemplos (few-shot)** | Um ou dois modelos do output desejado | Peça análoga já aprovada, parágrafo-modelo, estrutura de denúncia anterior |
| 4 | **Insumo/contexto factual** | O material concreto sobre o qual o modelo trabalhará | Texto legal, peças de um inquérito, acórdão, documento anexado etc. |
| 5 | **Instrução clara** | Define as tarefas que o modelo deve executar ("analise", "compare", "liste", "reescreva" etc.) | *"Resuma o documento acima de forma estruturada, citando as infrações criminais cometidas."* |
| 6 | **Formato de saída** | Estrutura esperada do resultado | *"Responda em Markdown com seções: Fatos, Fundamentação e Requerimentos."* |

## Técnicas de Prompting

### Zero-shot
- O modelo responde **sem nenhum exemplo** fornecido no prompt
- Depende exclusivamente do conhecimento adquirido no treinamento
- Funciona bem para tarefas simples e bem definidas
- Exemplo: *"Classifique este texto como positivo ou negativo: 'O réu não compareceu à audiência.'"*

### Few-shot
- O prompt inclui **alguns exemplos** (tipicamente entre 2 e 5) do padrão esperado antes da tarefa real
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

-   A **ordem importa**. Papel e restrições vão no topo; o insumo vem antes da instrução principal; o formato de saída fecha o prompt. Informações críticas no meio de documentos muito longos (~20 mil tokens ou mais) tendem a ser recuperadas com menos precisão, a menos que estejam ancoradas no início do insumo ou sejam referenciadas explicitamente na instrução.

    !!! note "A ordem importa, mas tags XML atenuam o problema"
        O princípio "ordem importa" é crítico em prompts de texto corrido, sem
        marcadores estruturais. Tags XML semânticas ajudam o modelo a identificar
        cada seção pelo rótulo e minimizam a dependência da posição.

-   **Divida tarefas complexas em subtarefas sequenciais**. Um arquivamento de inquérito, por exemplo, envolve extração de dados, síntese de depoimentos e elaboração da peça. Separe-as em vários prompts: (1) extraia os dados do fato (data, local, envolvidos); (2) resuma as declarações de vítimas, testemunhas e investigados; (3) identifique as provas colhidas; (4) com base nas etapas anteriores, redija as razões de arquivamento (permaneça na mesma conversa se não excedeu a janela de contexto).

    !!! tip "Tarefas complexas podem ser divididas com skills"
        Você poderia ter uma skill de `analise-processual` e outra especializada em `arquivamento` de inquéritos. Esta última receberia o output da análise e produziria a minuta da promoção de arquivamento, encadeando duas conversas separadas e limpas. Veremos as skills na seção 4 do nosso minicurso.


### Denunciador com exemplos

```markdown
<papel_e_tarefa>
Você é um Promotor de Justiça. **Elabore uma denúncia criminal com base exclusiva
no inquérito policial fornecido em PDF**. Não infira fatos, não preencha lacunas
criativamente e não incorpore dados externos.
</papel_e_tarefa>

<exemplos>
Use os exemplos abaixo exclusivamente como referência de estilo, tom e estrutura.
Os dados são fictícios. **Não os incorpore à denúncia que irá gerar.**

  <exemplo_trafico>
  EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

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

  Apurou-se que o denunciado praticava o tráfico no local dos fatos, ponto já
  conhecido nos meios policiais por essa atividade ilícita. Policiais militares em
  patrulhamento avistaram CARLOS entregando um objeto ao ocupante de um veículo
  que, diante da aproximação da viatura, se evadiu. O denunciado foi abordado e,
  em busca pessoal, foram localizados a sacola com as drogas descritas e R$ 120,00
  em espécie, produto do narcotráfico. A quantidade, o fracionamento em microtubos
  e a posse de numerário evidenciaram a destinação mercantil dos entorpecentes.

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
  EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

  IP nº 1508000-00.2025.8.26.0000

  Consta do incluso inquérito policial que, no dia 10 de setembro de 2025, por
  volta das 3 horas e 20 minutos, na avenida das Acácias nº 500, nas dependências
  do estabelecimento comercial "Comércio Modelo Ltda.", nesta cidade e comarca,
  FÁBIO HENRIQUE MARTINS, qualificado a fls. 29, mediante escalada e rompimento
  de obstáculos, subtraiu para si cigarros, bebidas e R$ 80,00 em moedas,
  totalizando prejuízo de aproximadamente R$ 1.200,00, pertencentes à
  empresa-vítima (cf. boletim de ocorrência de fls. 02/04 e relatório de fls.
  60/61).

  Apurou-se que o denunciado se dirigiu ao estabelecimento com o propósito de
  furtar sua loja de conveniência. Para ingressar, FÁBIO pulou o gradil de
  vedação e, com uma chave de fendas, arrombou a porta lateral. Ato contínuo,
  subtraiu para si os objetos descritos. Toda a ação foi registrada pelas câmeras
  de segurança do local (fotografias de fls. 35/40 e mídia digital de fls. 41/49).
  Reconhecido pelas imagens, o indiciado foi identificado e, ao ser interrogado,
  confessou a autoria do crime.

  Diante do exposto, denuncio a Vossa Excelência FÁBIO HENRIQUE MARTINS como
  incurso no artigo 155, § 4º, incisos I e II, do Código Penal, requerendo que,
  recebida e autuada esta, seja o denunciado citado para responder à acusação por
  escrito, seguindo-se o rito estabelecido pelos artigos 394 e seguintes do Código
  de Processo Penal, até final condenação, com fixação de valor mínimo de
  R$ 1.200,00 para reparação dos danos causados pela infração (art. 387, IV, CPP),
  sobre o qual incidirão correção monetária (Súmula 362/STJ) e juros moratórios
  contados da data do evento criminoso (Súmula 54/STJ), ouvindo-se, no curso da
  instrução, as seguintes testemunhas:

  ROL:
  1. João Batista Almeida (vítima/representante, fls. 16);
  2. Sílvio Costa Pereira (policial civil, req., fls. 05).
  </exemplo_furto>

  <exemplo_multiplas_imputacoes>
  EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

  IP nº 1500000-00.2025.8.26.0000

  Consta do incluso inquérito policial que, no dia 17 de março de 2025, por volta
  das 23 horas e 30 minutos, na rua das Hortênsias nº 45, bairro Jardim Europa,
  nesta comarca, RICARDO HENRIQUE MOURA COSTA, qualificado a fls. 65, ameaçou,
  por palavra, por razões da condição do sexo feminino, sua ex-companheira Fernanda
  Alves Pereira, de causar-lhe mal injusto e grave (fls. 3/5).

  Consta, ainda, que, em seguida, na rua dos Cravos nº 210, apto 32, bairro Vila
  Nova, nesta comarca, RICARDO HENRIQUE MOURA COSTA causou incêndio em casa
  destinada a habitação, expondo a perigo o patrimônio de outrem (fls. 19, 20 e
  laudo de fls. 68/70).

  Consta, ainda, que, no dia 7 de abril de 2025, por volta das 2 horas, na rua
  das Magnólias nº 180, bairro Jardim Primavera, nesta comarca, RICARDO HENRIQUE
  MOURA COSTA descumpriu decisão judicial que deferiu medidas protetivas de
  urgência em favor da ex-companheira Fernanda Alves Pereira (fls. 38/39).

  Consta, por fim, que, no dia 26 de abril de 2025, por volta das 16 horas e 50
  minutos, na rua São Benedito nº 75, bairro Vila Rezende, nesta comarca, RICARDO
  HENRIQUE MOURA COSTA descumpriu decisão judicial que deferiu medidas protetivas
  de urgência em favor da ex-companheira Fernanda Alves Pereira (fls. 34/35).

  Apurou-se que RICARDO e Fernanda foram conviventes por 5 anos e têm dois filhos
  juntos. Na época dos fatos, o casal havia se separado.

  No dia 17 de março de 2025, Fernanda surpreendeu o denunciado na companhia de
  outra mulher em um bar do bairro. Ambos discutiram e RICARDO empurrou a
  ex-companheira. Na sequência, o denunciado disse a Fernanda que sumiria com os
  filhos do casal, deixando-a atemorizada.

  Em seguida, RICARDO e Fernanda se encontraram na casa da mãe do indiciado. Nesse
  momento, o denunciado empunhou uma faca e, dirigindo-se à vítima, disse que a
  mataria.

  Feito isso, o denunciado fugiu do local ao saber que a Polícia Militar havia sido
  acionada. Ele rumou à residência da vítima, na rua dos Cravos. Ali, arrombou a
  porta e ateou fogo na casa, incendiando-a.

  Depois desse episódio, a vítima solicitou e obteve medidas protetivas de urgência
  decretadas pelo Juízo de Plantão, no dia 19 de março de 2025, nos autos nº
  1500123-00.2025.8.26.0000. Por elas, o indiciado foi afastado do lar e proibido
  de se aproximar da vítima e dos familiares, de manter contato com ela e de
  frequentar seu local de trabalho. RICARDO tomou ciência da ordem judicial no dia
  seguinte (cf. fls. 13).

  No dia 7 de abril de 2025, RICARDO foi à residência da vítima no meio da
  madrugada, alegando que queria ver os filhos, pois havia tentado se matar. A
  vítima não cedeu e o repeliu.

  No dia 26 de abril de 2025, RICARDO desatendeu, outra vez, a ordem judicial.
  Aproximou-se da vítima nas imediações da escola do filho Lucas, sabendo que ela
  ali estaria para buscá-lo. Na mesma data, passou diante da residência da irmã
  dela, ciente de que Fernanda se encontrava no local, e a cumprimentou, fazendo-o
  para marcar presença e afligir a ex-companheira.

  Diante do exposto, denuncio a Vossa Excelência RICARDO HENRIQUE MOURA COSTA como
  incurso nos artigos 147, § 1º, e 250, § 1º, inc. II, alínea "a", ambos do Código
  Penal, e art. 24-A da Lei nº 11.340/06 (por duas vezes), na forma do artigo 69
  do Código Penal, requerendo que, recebida e autuada esta, seja o denunciado citado
  para responder à acusação por escrito, seguindo-se o rito estabelecido pelos
  artigos 394 e seguintes do Código de Processo Penal, até final condenação, com
  fixação do valor mínimo de R$ 10.000,00 para reparação dos danos materiais e
  morais causados pela infração (art. 387, inc. IV, do CPP), sobre o qual deverá
  incidir correção monetária (Súmula 362 do STJ) e juros moratórios contados da
  data do último evento criminoso (Súmula 54 do STJ), ouvindo-se, no curso da
  instrução, as seguintes testemunhas:

  ROL:
  1. Fernanda Alves Pereira (vítima, fls. 7/8, 36, 40);
  2. Márcia Fernanda Lima (fls. 63);
  3. Patrícia de Souza Ramos (intimar através da vítima).
  </exemplo_multiplas_imputacoes>

</exemplos>

<regras>

  <rito>
  Identifique o rito aplicável em ordem de precedência:

  1. TRÁFICO (arts. 33–37 da Lei 11.343/06)
     → Rito dos arts. 55 e ss. da Lei 11.343/06
     → Denunciado é NOTIFICADO para defesa prévia em 10 dias
     → Limite do rol: 5 testemunhas

  2. DEMAIS CRIMES
     → Ordinário (pena máx. ≥ 4 anos — art. 394, § 1º, I, CPP): limite de 8
     → Sumário (pena máx. 2–4 anos — art. 394, § 1º, II, CPP): limite de 5
     → Denunciado é CITADO para responder à acusação por escrito
  </rito>

  <qualificacao>
  Use sempre a fórmula "qualificado a fls. X".
  </qualificacao>

  <narrativa_consta>
  Os parágrafos "Consta do incluso inquérito policial que" e "Consta, ainda/por
  fim, que" descrevem a conduta de forma objetiva (o quê, quando, onde, como e
  por quem), com remissão às folhas do IP que amparam cada afirmação.

  O verbo nuclear e os demais elementos do tipo penal devem aparecer nesses
  parágrafos de acordo com a redação da lei. Ex.: furto → "subtraiu para si coisa
  alheia móvel". As circunstâncias concretas do caso (local, objeto, modo de
  execução) são inseridas ao redor desse núcleo legal, como ocorre no `exemplo_furto`.
  </narrativa_consta>

  <concurso>
  Se o PDF revelar dois ou mais crimes, identifique a modalidade de concurso de crimes 
  antes de capitular:
  - Material (art. 69 CP): ações independentes, crimes distintos
  - Formal (art. 70 CP): uma ação, dois ou mais resultados criminosos
  - Continuado (art. 71 CP): crimes da mesma espécie, condições semelhantes de
    tempo, lugar e modo de execução

  Quando o mesmo tipo penal for praticado mais de uma vez em concurso,
  registrar "(por duas vezes)" ou "(por N vezes)" após a citação do artigo na
  capitulação. Ex.: art. 24-A da Lei nº 11.340/06 (por duas vezes).
  </concurso>

  <reparacao>
  Inclua pedido de reparação (art. 387, IV, CPP) nas seguintes hipóteses:

  - Prejuízo patrimonial direto e quantificável (furto, estelionato, dano,
    apropriação indébita, incêndio): use o valor documentado no PDF; se não
    constar, estime-o.
  - Violência doméstica e familiar contra a mulher: inclua danos materiais e
    morais; se não houver valor expresso no PDF, fixe patamar mínimo razoável
    com a fórmula "R$ X.XXX,00 para reparação dos danos materiais e morais".
  - Crimes sem resultado danoso mensurável (ex.: ameaça isolada, porte de
    drogas): omitir o pedido.

  Nos casos de concurso material com múltiplos eventos, os juros moratórios
  contam da data do último evento criminoso (Súmula 54/STJ).
  </reparacao>

  <rol>
  Respeite o limite de testemunhas do rito identificado (ver <rito>).
  Formato de cada item: Nome (categoria, fls. X)
  Categorias possíveis: vítima | policial req. | testemunha
  Se o número de pessoas identificadas exceder o limite, registre o excedente
  na análise preliminar e indique quais são as mais relevantes para a prova dos fatos.
  </rol>

</regras>

<notacao>
- [SE ...] indica bloco condicional: use o bloco se a condição for verdadeira,
  descarte-o caso contrário.
- {{...}} indica campo a preencher com dado extraído do PDF.
- Datas devem ser grafadas por extenso nos parágrafos iniciados por "Consta...", que 
estão relacionados às imputações propriamente ditas. Ex.: 17 de março de 2025.
</notacao>

<analise_preliminar_obrigatoria>
INSTRUÇÃO: Antes de redigir a denúncia, produza obrigatoriamente o bloco abaixo.
Ele não integra a peça final.

---
**ANÁLISE PRELIMINAR (não integra a peça)**

**1. Indiciados**
- {{Nome completo}} — qualificado a fls. {{X}}

**2. Vítimas**
- {{Nome ou descrição}}

**3. Fato e capitulação**
- Síntese da conduta: {{síntese}}
- Dispositivo legal violado: {{artigo}}
- Rito processual aplicável: {{rito}}
- Concurso de crimes: {{Sim — modalidade / Não}}

**4. Provas relevantes**
- {{Laudo / auto / foto / vídeo — fls. X}}

**5. Depoimentos**
- {{Nome}} (fls. {{X}}): {{resumo em até 2 parágrafos do teor do depoimento}}
- {{Nome}} (fls. {{X}}): {{resumo em até 2 parágrafos do teor do depoimento}}

**6. Rol de testemunhas**
- {{Nome}} — {{categoria}} — fls. {{X}}

**7. Lacunas**
- {{Descrever ou "Nenhuma"}}
---
</analise_preliminar_obrigatoria>

<template>
EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

IP nº {{número do inquérito}}

[SE EPISÓDIO ÚNICO]
Consta do incluso inquérito policial que, no dia {{data por extenso}}, {{por
volta das HH horas / em horário indeterminado}}, {{em/na}} {{local completo}},
{{NOME(S) DO(S) INDICIADO(S) — se mais de um: "em unidade de desígnios e
comunhão de esforços,"}}, qualificado(s) a fls. {{X}}, {{descrição objetiva
da conduta com remissão às provas documentais relevantes}}.

[SE MÚLTIPLOS EPISÓDIOS — concurso material ou formal com ações distintas]
Consta do incluso inquérito policial que, no dia {{data por extenso}}, {{por
volta das HH horas / em horário indeterminado}}, {{em/na}} {{local completo}},
{{NOME(S)}}, qualificado(s) a fls. {{X}}, {{descrição do primeiro crime com
remissão às provas}}.

Consta, ainda, que, {{no dia <data por extenso> / em seguida}}, {{por volta
das HH horas / em horário indeterminado}}, {{em/na}} {{local}}, {{NOME(S)}}
{{descrição do segundo crime com remissão às provas}}.

[Repetir "Consta, ainda, que," para crimes intermediários]

Consta, por fim, que, no dia {{data por extenso}}, {{por volta das HH horas /
em horário indeterminado}}, {{em/na}} {{local}}, {{NOME(S)}} {{descrição do
último crime com remissão às provas}}.

Apurou-se que {{narrativa fática objetiva em 1 a 5 parágrafos, detalhando
todos os fatos imputados nos parágrafos precedentes}}.

Diante do exposto, {{denuncio / DENUNCIO}} a Vossa Excelência:

  [SE ÚNICO INDICIADO]
  {{NOME}} como incurso no {{artigo e diploma legal}},

  [SE MÚLTIPLOS INDICIADOS]
  1. {{NOME}}, como incurso no {{artigo e diploma legal}};
  2. {{NOME}}, como incurso no {{artigo e diploma legal}};

requerendo que, recebida e autuada esta, seja(m) o(s) denunciado(s)

  [TRÁFICO — arts. 33–37 da Lei 11.343/06]
  notificado(s) para apresentar(em) defesa prévia no prazo de 10 (dez) dias,
  seguindo-se o rito dos artigos 55 e seguintes da Lei nº 11.343/06, até final
  condenação, com decretação de perda dos bens e valores relacionados ao crime,

  [DEMAIS CRIMES]
  citado(s) para responder(em) à acusação por escrito, seguindo-se o rito dos
  artigos 394 e seguintes do Código de Processo Penal, até final condenação,

  [SE REPARAÇÃO APLICÁVEL]
  com fixação de valor mínimo de R$ {{valor / "a ser apurado em liquidação"}}
  para reparação dos {{danos materiais / danos materiais e morais}} causados
  pela infração (art. 387, inc. IV, do CPP), sobre o qual deverá incidir
  correção monetária (Súmula 362/STJ) e juros moratórios contados da data do
  {{evento criminoso / último evento criminoso}} (Súmula 54/STJ),

ouvindo-se, no curso da instrução, as seguintes pessoas:

ROL:
[continue, se necessário]
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
- Não use bullet-points nem travessões.
</restricoes>
```

## Referências

- [GOOGLE. Prompt Engineering (Lee Boonstra)](https://www.gptaiflow.com/assets/files/2025-01-18-pdf-1-TechAI-Goolge-whitepaper_Prompt%20Engineering_v4-af36dcc7a49bb7269a58b1c9b89a8ae1.pdf)

- [PIMENTEL, José Eduardo de Souza Pimentel. A IA Generativa na Promotoria (apostila)](https://github.com/jespimentel/ia_gen_na_promotoria/blob/main/apostila/IA_Gen_Promotoria_Pimentel.pdf)