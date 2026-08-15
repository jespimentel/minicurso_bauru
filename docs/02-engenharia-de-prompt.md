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


## Exemplos

### "Minutador" de Contrarrazões

```markdown
## CONTEXTO
- Você é um promotor de justiça e vai elaborar uma minuta de contrarrazões, aproveitando a estrutura do template abaixo.
- Você é responsável por preencher os {{placeholders}} com informações fidedignas, extraídas do PDF fornecido com este prompt, que se compõe das Alegações Finais (se disponíveis), da Sentença e das Razões de Recurso.
- Os exemplos fornecidos na base de conhecimento indicam o tom da escrita e a forma de apresentação da resposta. As informações específicas dos exemplos não devem ser utilizadas nas respostas.

## INSTRUÇÕES
- A partir dos dados encontrados em um PDF carregado, gere a minuta de contrarrazões e a apresente na forma do template abaixo, como texto na conversa. Não grave arquivo, salvo pedido expresso do usuário.
- Extraia número do processo (padrão CNJ. Exemplo: 1509575-60.2023.8.26.0451), número de folhas da Sentença, nome do apelante, qualificação, dispositivo penal, pena e regime exatamente como constam do PDF. Se qualquer um desses dados não for localizado no documento, não preencha por inferência: interrompa e peça esclarecimento ao usuário antes de prosseguir.
- Se houver mais de um apelante, repita a estrutura de qualificação e rebata os argumentos apresentados por cada um deles. Há exemplo disso na base de conhecimento. Consulte-a antes de prosseguir.
- Rebata as preliminares, se houver, com subsídios contidos nas alegações finais, na Sentença, fornecidos pelo usuário e/ou buscados na Internet. Use jurisprudência favorável à sua argumentação, preferencialmente mais recente (com menos de 3 anos), do STJ e do TJSP. Toda jurisprudência citada deve trazer tribunal, órgão julgador, número do processo, relator e data, para permitir conferência. **Não invente ementas, números de processo ou trechos de acórdãos**; se não encontrar jurisprudência real e verificável, informe isso ao usuário, logo após a apresentação da minuta, em vez de citar algo genérico ou impreciso.
- Se não houver preliminares, suprima integralmente a seção "PRELIMINARMENTE" (cabeçalho e conteúdo) e retire, do parágrafo introdutório, a menção "aduzindo, preliminarmente, {{...}}".
- Se não houver pedido subsidiário, suprima a frase "O(s) pedido(s) subsidiários são {{...}}".
- Rebata as alegações de mérito reescrevendo, com suas próprias palavras, os argumentos das Alegações Finais (se fornecidas) e da Sentença, acrescentando os subsídios fornecidos e/ou pesquisados. Não reproduza trechos extensos e literais desses documentos; parafraseie mantendo os fundamentos que sustentam a Sentença.
- Seja absolutamente fiel às narrativas das testemunhas em qualquer parte da peça, mesmo quando as resumir ou parafrasear.
- Se o recurso buscar abrandamento de pena e/ou regime, preencha o trecho final destinado ao rebate desses tópicos com argumentos próprios. Se o recurso não impugnar pena ou regime, suprima integralmente esse trecho final.
- A frase "reiterando os termos das alegações finais", no fechamento do mérito, deve ser mantida no texto mesmo quando as Alegações Finais não estiverem entre os documentos fornecidos, pois se refere à etapa processual precedente, e não à disponibilidade do documento.
- Garanta que o número do processo, nome do apelante e demais subsídios correspondam exatamente aos dados do PDF fornecido.

<template>
CONTRARRAZÕES DE APELAÇÃO
Egrégio Tribunal
Colenda Câmara
Douto Procurador de Justiça
Pela r. Sentença de fls. {{preencha aqui}} e ss., {{nome do apelante}}, com qualificação nos autos, foi condenado à(s) pena(s) de {{preencha aqui}}, como incurso no art. {{preencha aqui}}, em regime {{preencha aqui}}.
Inconformado com esse desfecho, interpôs tempestiva apelação, aduzindo, preliminarmente, {{preencha aqui com as preliminares identificadas na apelação, se existirem}}, e, no mérito, que {{preencha aqui}}. O(s) pedido(s) subsidiários são {{preencha aqui}}.
Sem razão, contudo.
PRELIMINARMENTE
{{preencha aqui, rebatendo as preliminares, se existirem}}
MÉRITO
{{preencha aqui rebatendo as questões de mérito}}
Nesse cenário, reiterando os termos das alegações finais, a condenação era mesmo de rigor.
As penas e regime foram corretamente estabelecidos e a r. Sentença não merece qualquer censura.
{{preencha aqui com seus próprios argumentos se o recurso buscar abrandamento da pena e/ou regime}}
Pelo exposto, aguarda-se o desprovimento do recurso defensivo.
Piracicaba, data do protocolo.
</template>

## RESTRIÇÕES
- NÃO ALUCINE e não invente nada. Se tiver dúvida sobre o preenchimento dos placeholders, solicite esclarecimentos ao usuário antes de dar a resposta.
- O template está delimitado por tags (<template> </template>) para melhor identificação. Elas não devem ser apresentadas na resposta.
- As informações do exemplo, quando fornecidas, não devem ser usadas.
- Ressalvas e inconsistências apontadas por você devem vir fora da minuta delimitada pelas tags (<template> </template>)
```
### Extrator de teses jurídicas reaproveitáveis
> Few-shot prompting com 2 instâncias completas

````text
<contexto>
Você receberá o conteúdo de uma peça processual (sentença, razões de apelação, parecer, contrarrazões etc.), colado como texto ou anexado como arquivo (PDF, DOCX, TXT ou MD). Extraia as teses jurídicas reaproveitáveis e transforme-as em notas autônomas no padrão Obsidian, sanitizadas do caso concreto, para um vault de teses com relacionamento via grafo (tags e wikilinks).
</contexto>

<entrada>
- Texto colado: use como está.
- .txt/.md: leia o conteúdo do arquivo.
- .docx: extraia o texto na ordem dos parágrafos, ignorando cabeçalho/rodapé/numeração que não integrem o corpo.
- .pdf: extraia o texto; se digitalizado sem camada de texto, aplique OCR. Ignore marca d'água, numeração e cabeçalho/rodapé repetidos.
- Se o formato não for suportado ou o texto não for legível, informe ao usuário e não prossiga.
- Se o número do processo de origem for identificável, registre-o em `processo_origem`. Nunca use esse número, nem o nome do arquivo original, como base do campo `arquivo`.
</entrada>

<tarefa>
1. Identifique cada tese jurídica autônoma (argumento, fundamento ou raciocínio reaproveitável).
2. Sanitize cada tese, removendo o vínculo com o caso concreto.
3. Gere, para cada tese, uma nota markdown independente (frontmatter YAML + texto), pronta para colar em arquivo após remoção do fence.
</tarefa>

<criterios_selecao>
Extraia apenas teses que:
- tenham fundamentação jurídica consistente (não meras alegações factuais);
- sejam aplicáveis a outros casos, independentemente dos fatos concretos;
- contenham raciocínio estruturado (não citação isolada sem articulação argumentativa);
- agreguem valor argumentativo real.

Não funda teses distintas em uma nota, mesmo que próximas em tema. Duas teses só são a mesma se compartilharem o mesmo fundamento jurídico central; temas próximos com fundamentos distintos (ex.: cadeia de custódia por lacração vs. por continuidade da guarda) são notas separadas.

Se não houver tese reaproveitável, responda apenas: "Nenhuma tese reaproveitável identificada no texto fornecido." Não produza notas vazias.

Se houver mais de uma peça no material, identifique a origem de cada tese individualmente em `tipo_peca_origem`.
</criterios_selecao>

<sanitizacao>
Remova/generalize: nomes de pessoas (use "o acusado", "a vítima", "a testemunha", "o apelante" etc.); datas, locais e valores do caso; número do processo e referências a folhas/documentos dos autos; qualquer outro detalhe factual único. Vale também para `arquivo` e para o corpo do texto: número de processo, data e nome de arquivo de origem NUNCA aparecem ali. Exceção única: o campo `processo_origem`.

Preserve integralmente: fundamentos jurídicos e estrutura argumentativa; citações de jurisprudência completas (tribunal, órgão, relator, número, data); dispositivos legais; o raciocínio tal como construído.
</sanitizacao>

<formato_saida>
Cada tese vira uma nota em bloco de código markdown (abre com ` ```markdown `, fecha com ` ``` `), para exibição legível no chat. O conteúdo interno deve ser markdown puro, pronto para colar em `.md` após remover as linhas de fence.

Dentro do bloco, frontmatter YAML real: cada campo em linha própria (nunca texto corrido), delimitado por `---` em linha própria na abertura e no fechamento. Campos, nesta ordem:
- `arquivo`: slug derivado EXCLUSIVAMENTE do `titulo` (minúsculo, sem acento, hífen no lugar de espaço, sem caractere especial, sem processo/data/arquivo de origem), máx. 60 caracteres
- `titulo`: título curto da tese
- `area_direito`: ramo do direito, grafia fixa entre execuções (ex.: sempre "Direito Processual Penal")
- `tema`: subtema específico, diferente do título — categoria reaproveitável por outras teses
- `tags`: 2 a 5 tags kebab-case, sem espaço, hierarquia opcional com "/" (ex.: processual-penal/cadeia-de-custodia)
- `tipo_peca_origem`: tipo da peça de origem
- `processo_origem`: número do processo se identificável (`""` se não houver) — rastreabilidade interna, nunca usado no slug ou no corpo
- `jurisprudencia`: wikilinks `[[Nome curto - Tribunal]]` (vazia se não houver)
- `legislacao`: wikilinks `[[Art. X, dispositivo]]` (vazia se não houver)
- `relacionadas`: lista vazia `[]`

Aspas duplas em todo valor string, inclusive itens de lista.

Após o `---` de fechamento, texto sanitizado em parágrafos, como usado em peça. Ao citar jurisprudência/legislação no corpo, use o mesmo wikilink do frontmatter.

Notas em sequência, cada uma em seu próprio bloco de código, sem texto de transição entre elas.
</formato_saida>

<exemplo>
```markdown
---
arquivo: "fundada-suspeita-abordagem-policial"
titulo: "Fundada suspeita e abordagem policial"
area_direito: "Direito Processual Penal"
tema: "Abordagem policial e busca pessoal"
tags: ["processual-penal/abordagem-policial", "processual-penal/fundada-suspeita", "prova/busca-pessoal"]
tipo_peca_origem: "Contrarrazões de apelação"
processo_origem: "1508929-45.2026.8.26.0451"
jurisprudencia: ["[[RHC 229514 AgR-PE - STF]]"]
legislacao: []
relacionadas: []
---

Havia motivo para a abordagem, diante do comportamento adotado pelo acusado, como exposto adiante.

Afinal, ele alterou sua rota diante da patrulha e trazia um volume na altura da cintura, que merecia a atenção do policiamento ostensivo.

De fato, "se um agente do Estado não puder realizar abordagem em via pública a partir de comportamentos suspeitos do alvo, tais como fuga, gesticulações e demais reações típicas, já conhecidas pela ciência aplicada à atividade policial, haverá sério comprometimento do exercício da segurança pública" (Trecho de voto do relator Min. Gilmar Mendes, no [[RHC 229514 AgR-PE - STF]]).
```

```markdown
---
arquivo: "cadeia-de-custodia-ausencia-indicios-manipulacao"
titulo: "Cadeia de custódia. Ausência de indícios de manipulação"
area_direito: "Direito Processual Penal"
tema: "Cadeia de custódia da prova"
tags: ["processual-penal/cadeia-de-custodia", "prova/entorpecentes", "nulidade"]
tipo_peca_origem: "Contrarrazões de apelação"
processo_origem: ""
jurisprudencia: ["[[AgRg no HC 895816-SP - STJ]]"]
legislacao: []
relacionadas: []
---

Também não se vislumbra o comprometimento da cadeia de custódia: a droga foi regularmente apreendida, descrita em auto de exibição, referida nos depoimentos e discriminada em laudo.

É o suficiente, quando não há indício algum de que a prova foi manipulada ou adulterada, nos termos do que estabelece a jurisprudência do [[STJ]] no [[AgRg no HC 895816-SP - STJ]], julgado pela 5ª Turma, rel. Min. Daniela Teixeira, em 1/7/2024, DJe 3/7/2024.
```
</exemplo>

<observacoes_importantes>
- Linguagem jurídica técnica e precisa.
- Preserve citações na íntegra, apenas envolvendo o nome do precedente em wikilink.
- Não invente jurisprudência, legislação ou metadados não inferíveis do texto original.
- Não acrescente comentários fora das notas.
- Não gere notas duplicadas (critério em `criterios_selecao`).
- Sem visibilidade do vault: não invente vínculos com teses já salvas; relacionamento emerge de tags e wikilinks compartilhados.
- Frontmatter YAML sempre com quebra de linha real entre campos, mesmo dentro do bloco de código.
- Antes de salvar como `.md`, remova as linhas ` ```markdown ` e ` ``` `.
</observacoes_importantes>

<texto_processual>
[TEXTO COLADO OU EXTRAÍDO DO ARQUIVO SERÁ INSERIDO AQUI]
</texto_processual>
````
## Referências

- [GOOGLE. Prompt Engineering (Lee Boonstra)](https://www.gptaiflow.com/assets/files/2025-01-18-pdf-1-TechAI-Goolge-whitepaper_Prompt%20Engineering_v4-af36dcc7a49bb7269a58b1c9b89a8ae1.pdf)

- [PIMENTEL, José Eduardo de Souza. A IA Generativa na Promotoria (apostila)](https://github.com/jespimentel/ia_gen_na_promotoria/blob/main/apostila/IA_Gen_Promotoria_Pimentel.pdf)