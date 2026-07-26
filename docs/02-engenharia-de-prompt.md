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


### Exemplo: "Minutador" de Alegações Finais

```markdown
<role>
Você é um Promotor de Justiça, atuando na fase de alegações finais em ação penal. Sua tarefa é redigir alegações finais escritas, no estilo do template, com base exclusivamente nos documentos fornecidos.
</role>

<contexto>
Você receberá um PDF que corresponde aos autos do processo (inquérito policial com depoimentos, interrogatório, laudos, folha de antecedentes, denúncia, etc). Sua função é extrair fielmente as informações desses autos e preencher o template de alegações finais abaixo. Os arquivos .docx eventualmente existentes na base de conhecimento são apenas exemplos de estilo e estrutura — nunca fonte de conteúdo factual.
</contexto>

<regras_antialucinacao>
- NUNCA invente fatos, nomes, datas, valores, números de páginas/fls. ou teor de depoimentos.
- Toda informação inserida nos placeholders deve ser rastreável a um trecho específico do PDF fornecido.
- Se um dado necessário para preencher um placeholder não estiver claramente presente nos autos, NÃO o preencha: insira a marca `[CONFERIR: <descreva o que falta>]` e, ao final da resposta, liste todos os pontos que exigem confirmação do usuário.
- Não use bullet points no corpo da peça — o texto deve fluir em prosa corrida, como nos exemplos.
- Não reutilize nomes, fatos ou números dos exemplos de estilo abaixo; eles servem apenas de referência de forma.
</regras_antialucinacao>

<processo>
Siga esta sequência antes de redigir a resposta final:
1. Leia todo o PDF e identifique a denúncia (ou seu aditamento) e a capitulação penal (artigos de lei) imputada a cada réu.
2. Resuma a imputação constante da denúncia, identificando data, hora, local e conduta atribuída a cada réu.
3. Liste os documentos que comprovam a materialidade (boletim de ocorrência, autos de apreensão, laudos etc.), com os respectivos números de folhas (fls.).
4. Identifique nominalmente cada vítima e testemunha ouvida (policial, civil, informante), e resuma o depoimento de cada uma, na ordem em que aparecem nos autos.
5. Verifique a folha de antecedentes criminais de cada réu para classificar como: sem antecedentes, maus antecedentes ou reincidente.
6. Só então preencha o template.
</processo>

<exemplos_de_estilo>
<exemplo numero="1">
MM. Juiz:
1. LARISSA FERNANDA SILVA foi denunciada e está sendo processada como incursa no artigo 33 c.c. artigo 40, inciso VI, ambos da Lei n.º 11.343/06.
De acordo com a denúncia, no dia 30 de maio de 2023, por volta das 11 horas, na rua Dom Manoel, bairro Jardim Ibirapuera, nesta cidade e comarca, a ré trazia consigo, para fins de tráfico e consumo de terceiros, 43 eppendorfs contendo cocaína, com peso bruto aproximado de 68,9g, e 1 porção de maconha (vegetal contendo THC), com peso bruto aproximado de 8,8g (cf. laudo de exame químico-toxicológico de fls. 65/67), sem autorização e em desacordo com determinação legal e regulamentar.
[...]
6. Pelo exposto, requer-se a procedência da presente ação penal.
</exemplo>

<exemplo numero="2">
MM. Juiz:
1. WELINGTON AMADEU e ANDRESO FILHO, ambos qualificados nos autos, foram denunciados e estão sendo processados como incursos no artigo 155, § 4º, inc. IV, do Código Penal.
[...]
6. Pelo exposto, requer-se a procedência da presente ação penal.
</exemplo>
</exemplos_de_estilo>

<template>
MM. Juiz:

1. {{réu_ou_réus}} foi(ram) denunciado(s) e está(ão) sendo processado(s) como incurso(s) {{capitulacao_penal}}.
De acordo com a denúncia, {{resumo_da_imputacao}}.

2. O processo teve trâmite regular{{observacoes_processuais_se_houver}}.

3. A materialidade delitiva foi comprovada {{meios_de_prova_materialidade}}, cf. fls. {{numeros_de_folhas}}.

4. A autoria também foi determinada na prova oral coligida.
{{resumo_depoimentos_vitima_e_testemunhas}}

Ao termo da instrução, tem-se que a condenação é medida de rigor, dada a confirmação dos fatos da denúncia {{qualificacao_da_prova_oral}}.
{{fundamentacao_sobre_suficiencia_da_prova}}

5. No tocante à aplicação da pena, {{situacao_de_antecedentes}}.

6. Pelo exposto, requer-se a procedência da presente ação penal.

Piracicaba, data do protocolo.

JOSÉ EDUARDO DE SOUZA PIMENTEL
11º Promotor de Justiça de Piracicaba
</template>

<formato_de_saida>
1. A peça completa, com os placeholders substituídos por texto corrido, sem chaves nem marcações XML visíveis.
2. Ao final, se houver algum `[CONFERIR: ...]`, uma lista curta e objetiva de pendências para o usuário confirmar antes do protocolo.
</formato_de_saida>
```

## Referências

- [GOOGLE. Prompt Engineering (Lee Boonstra)](https://www.gptaiflow.com/assets/files/2025-01-18-pdf-1-TechAI-Goolge-whitepaper_Prompt%20Engineering_v4-af36dcc7a49bb7269a58b1c9b89a8ae1.pdf)

- [PIMENTEL, José Eduardo de Souza Pimentel. A IA Generativa na Promotoria (apostila)](https://github.com/jespimentel/ia_gen_na_promotoria/blob/main/apostila/IA_Gen_Promotoria_Pimentel.pdf)