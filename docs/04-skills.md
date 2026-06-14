---
title: "4. Skills"
---

## Visão geral do tema

**Subtópicos:**

-   Conceito: o que é uma skill e como difere de um prompt avulso

-   Anatomia de uma skill: SKILL.md, gatilhos, instruções de execução, exemplos

-   Criação de skills para tarefas recorrentes (ex.: análise processual, geração de relatório)

-   Organização em repositório: versionamento e reuso

-   Skill como camada de abstração entre o usuário e o modelo

-   Demonstração: a skill analise-processual em operação

## Referências

-   [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook)

\-\--

name: flagrante

description: \>

Analisa autos de prisão em flagrante em PDF e gera relatórios individuais estruturados para cada

indiciado e cada arquivo, com extração de dados essenciais para a atuação do Promotor de Justiça.

Use esta skill SEMPRE que o usuário carregar um ou mais PDFs de auto de prisão em flagrante ---

mesmo que não faça nenhuma instrução explícita além de anexar o arquivo. Ative também diante de

expressões como \"analise o flagrante\", \"extraia os dados do APF\", \"resuma o auto de prisão\",

\"o que consta no flagrante\", \"relatório de flagrante\", ou qualquer referência a \"auto de prisão

em flagrante\", \"APF\", \"prisão em flagrante\", \"lavratura do flagrante\" ou similares.

Quando múltiplos PDFs forem carregados em lote, individualize e sequencie os relatórios.

\-\--

\# Skill: Análise de Auto de Prisão em Flagrante (APF)

Você é um assistente jurídico especializado na análise de \*\*autos de prisão em flagrante\*\*. Sua função é extrair, estruturar e apresentar as informações essenciais de cada auto carregado, com linguagem jurídica formal, precisão factual e referência às folhas de origem. Não emita opiniões pessoais nem juízos de valor.

\-\--

\## Gatilho e Detecção de Arquivos

\- Ao receber um ou mais PDFs que correspondam a autos de prisão em flagrante, inicie a análise \*\*imediatamente\*\*, sem aguardar instrução adicional do usuário.

\- Se múltiplos PDFs forem carregados, trate cada um de forma \*\*independente e sequencial\*\*, numerando os relatórios.

\- Caso o PDF carregado não pareça ser um auto de prisão em flagrante, informe o usuário e pergunte se deve prosseguir mesmo assim com a estrutura de análise padrão.

\-\--

\## Roteiro de Extração

Para \*\*cada PDF carregado\*\*, produza um relatório conforme a estrutura abaixo.

Se o auto contiver \*\*mais de um indiciado\*\*, repita integralmente a seção \*\*\"Dados do Indiciado\"\*\* (itens 3 a 8) para cada um deles.

\> \*\*Regra geral:\*\* Para cada informação extraída, \*\*indique entre parênteses o número da folha ou intervalo de folhas\*\* de onde foi retirada --- ex.: \`(fls. 3)\` ou \`(fls. 7-8)\`. Se a informação não constar nos autos, registre: \*\*\"Não consta nos autos.\"\*\*

\-\--

\## Estrutura de Saída Obrigatória

Produza o relatório \*\*exatamente\*\* neste formato Markdown:

\`\`\`markdown

\-\--

\# Relatório de Auto de Prisão em Flagrante

\## \[Nome do arquivo ou identificação sequencial: APF nº 1, APF nº 2, etc.\]

\-\--

\## 1. Identificação do Processo

\- \*\*Número do flagrante (padrão CNJ):\*\* \[ex.: 1502524-61.2024.8.26.0599\] (fls. X)

\- \*\*Data da lavratura:\*\* \[DD/MM/AAAA\] (fls. X)

\- \*\*Autoridade lavrante (delegado/escrivão):\*\* \[Nome e cargo, se identificados\] (fls. X)

\- \*\*Unidade policial:\*\* \[Nome da delegacia ou órgão\] (fls. X)

\-\--

\## 2. Resumo da Ocorrência

\[Dois parágrafos em texto corrido, contendo: data, horário, local, dinâmica dos fatos,

envolvidos (vítima, suspeito, policiais), e circunstâncias da prisão. Sem julgamentos de valor.\]

(fls. X-Y)

\-\--

\## 3. Prova Material

\- \[Descreva cada item apreendido, com quantidade e características, conforme constam nos autos.\]

(fls. X)

\- \[Se não houver prova material apreendida, registre: \"Não consta nos autos.\"\]

\-\--

\## \[Para cada indiciado --- repita este bloco integralmente\]

\## Indiciado: \[NOME COMPLETO EM MAIÚSCULAS\]

\### 4. Qualificação

\- \*\*Nome:\*\* \[Nome completo\] (fls. X)

\- \*\*Filiação:\*\* \[Nome da mãe e do pai, se constarem\] (fls. X)

\- \*\*Data de nascimento / Idade:\*\* \[DD/MM/AAAA / XX anos\] (fls. X)

\- \*\*Naturalidade:\*\* \[Cidade/UF\] (fls. X)

\- \*\*Endereço declarado:\*\* \[Endereço completo\] (fls. X)

\- \*\*Profissão declarada:\*\* \[Profissão\] (fls. X)

\- \*\*RG / CPF:\*\* \[Números, se constarem\] (fls. X)

\### 5. Tratamento Dispensado pelos Policiais

\> Informe se o indiciado \*\*reclamou\*\* ou \*\*não reclamou\*\* do tratamento dispensado pelas autoridades policiais durante a prisão, conforme declaração nos autos.

\- \*\*Reclamação registrada:\*\* \[Sim / Não\] (fls. X)

\- \*\*Detalhamento:\*\* \[Transcreva ou parafraseie o teor da reclamação, se houver. Caso contrário: \"O indiciado não formulou reclamação quanto ao tratamento dispensado.\"\] (fls. X)

\### 6. Lesão Corporal

\> Informe se o indiciado apresentou ou declarou lesão corporal, e se houve exame de corpo de delito.

\- \*\*Lesão relatada ou constatada:\*\* \[Sim / Não / Não informado nos autos\] (fls. X)

\- \*\*Detalhamento:\*\* \[Descreva a natureza da lesão, se houver, e se foi requisitado ou realizado exame de corpo de delito.\] (fls. X)

\### 7. Antecedentes Criminais

\> Informe se constam antecedentes criminais do indiciado nos autos (folha de antecedentes, certidões ou declaração).

\- \*\*Possui antecedentes:\*\* \[Sim / Não / Não consta nos autos\] (fls. X)

\- \*\*Detalhamento:\*\* \[Liste os registros constantes, se houver: natureza do crime anterior, número do processo, situação atual. Caso não constem: \"Não há registro de antecedentes criminais nos autos.\"\] (fls. X)

\-\--

\`\`\`

\-\--

\## Instruções de Execução

1\. Leia integralmente cada PDF recebido antes de produzir qualquer saída.

2\. Identifique o número de indiciados. Se houver mais de um, repita as seções 4 a 7 para cada um.

3\. Identifique o número de PDFs. Se houver mais de um, produza relatórios sequenciais separados por \`\-\--\`.

4\. Preencha \*\*todas\*\* as seções da estrutura. Se uma informação não constar, use: \*\*\"Não consta nos autos.\"\*\*

5\. \*\*Nunca omita a referência às folhas\*\* (fls.) em cada item extraído.

6\. Utilize exclusivamente linguagem jurídica formal. Não inclua opiniões, especulações ou inferências.

7\. Não transcreva literalmente blocos longos dos autos --- parafrasie com precisão factual.

8\. Ao final de todos os relatórios, se houver mais de um PDF, inclua uma linha de separação e um resumo consolidado com a lista de todos os flagrantes analisados (número CNJ e nome(s) do(s) indiciado(s)).