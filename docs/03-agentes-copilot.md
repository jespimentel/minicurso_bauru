---
title: "3. Agentes de IA e Copilot"
---

## O que são Agentes de IA?

Um Agente de IA é um sistema de software autônomo capaz de perceber seu ambiente (por meio de dados ou inputs), tomar decisões e executar ações para atingir objetivos específicos, operando com pouca ou nenhuma intervenção humana direta.

## "Agentes" Copilot são Agentes de IA?

- **Autonomia não é "sim ou não", é uma escala.** Vai do que só responde quando você pede até o que age sozinho. A Microsoft chama tudo de "agente", mas há níveis bem diferentes.

- **O que seria um agente "de verdade":** decide sozinho o que fazer; executa ações (não só escreve); faz vários passos em sequência; e começa por conta própria.

- **A maioria dos agentes criados no Copilot:** tem uma persona (instruções) e pode consultar uma base de conhecimento. Você pede, ele responde e para. É um **assistente** bem configurado, não um **agente** que age sozinho.

- **E quando ele consulta a web?** O que importa não é *acessar* a internet, e sim *quem decide os passos*:
    - busca simples (uma consulta e responde) → continua reativo, só com fonte mais ampla;
    - busca que itera sozinha (formula buscas, segue links, refaz) → aí, sim, sobe na escala.

- **O Copilot Studio completo:** pode usar ferramentas, decidir sozinho o que consultar e ser disparado automaticamente → **Aí, sim, age como agente.**

- **Na Promotoria, controlar é melhor que automatizar tudo:** vale mais um assistente que para e espera sua revisão do que um que age e decide tudo sozinho.

## Agente declarativo X Agente autônomo 

| Critério | Agente declarativo (Copilot) | Agente autônomo (Claude Code, Codex, Devin etc.) |
|---|---|---|
| **O que faz** | Gera texto no chat | Executa trabalho no computador |
| **Acessa arquivos** | Só a base de conhecimento | Lê, cria e edita arquivos reais |
| **Roda código** | Não | Sim |
| **Decide os passos** | Não (reage ao que você pede) | Sim (planeja e encadeia ações) |
| **Resultado** | Texto para você revisar | Tarefa concluída (pipeline, docx, pdf...) |
| **Nível de autonomia** | Assistente configurado | Agente autônomo |

!!! info "Por que agente declarativo?"
    Tudo que o agente sabe fazer é declarado
    em linguagem natural: persona, conhecimento
    e escopo. Sem código, sem lógica, sem fluxos.

## Por que usar o Copilot na Promotoria de Justiça?

### Integração nativa com o ambiente Microsoft

O Copilot está integrado ao ecossistema já usado no MPSP, o Microsoft 365, 
sem necessidade de instalar ferramentas externas ou criar ambientes separados. 

### Sigilo e conformidade

Os dados permanecem dentro do tenant Microsoft da instituição. Nenhum conteúdo
é usado para treinar modelos externos. A infraestrutura segue as políticas de
segurança e residência de dados já aplicadas ao ambiente corporativo. É aderente 
ao **Aviso nº 009/2025-CGMP, de 27/06/2025**, que trata do uso de IA Generativa 
na Instituição.

### Benefícios

- **Configuração:** simples e feita por interface gráfica, sem código
- **Base de conhecimento:** upload de arquivos (modelos, tabelas, referências) que o agente consulta para gerar respostas
- **Distribuição interna:** compartilhamento direto no Teams ou SharePoint, sem publicação externa
- **Configuração controlada:** sem memória entre sessões, sem conectores ativos, sem acesso a APIs externas (o agente responde só com o que você definiu).

## Copilot Chat X Copilot Premium

| Recurso / capacidade | Copilot Chat (incluso no E3) | Add-on Microsoft 365 Copilot (premium) |
|---|---|---|
| Criar agente declarativo (Agent Builder) | Sim — sem grounding no tenant (ver abaixo) | Sim |
| Agente com instruções + web pública | Sim — sem custo | Sim — sem custo |
| Grounding em SharePoint/OneDrive (dados do tenant) | Não — salvo com usage-based billing (cobrança por consumo) | Sim — incluso, sem cobrança por consumo |
| Anexar arquivo embutido (upload do dispositivo) | Não — salvo com usage-based billing | Sim |
| E-mail (Outlook) e Teams como fonte | Não | Sim |
| People data (dados de pessoas do tenant: perfis, organograma, interações) | Não | Sim |
| Limite de tamanho de arquivo (SharePoint) | ~7 MB por arquivo — limitação de memória sem licença | 200 MB — 512 MB para arquivo individual (PDF/PPTX/DOCX) |
| Busca inteligente nos documentos do tenant (SharePoint) | Não — busca simples, limitada a arquivos de até ~7 MB | Sim — exige licença M365 Copilot no tenant e agente autenticado via conta Microsoft (respeita as permissões de cada usuário) |
| Criação de agentes no Copilot Studio | Não — exige licença | Sim |

Fonte: Documentação oficial do Microsoft Learn (learn.microsoft.com). Acesso em jul. 2026. Licenciamentos sujeitos a alteração. 

## Função "Conhecimento" (Premium)

Base de conhecimento por recuperação (RAG): o agente busca trechos relevantes de documentos quando precisa, em vez de mantê-los fixos no contexto.

- **No prompt:** o que se aplica *sempre* (regras, template, travas).
- **No conhecimento:** referência estável, consultada conforme o caso (ex.: catálogo de modelos, manual de regras).

!!! tip "A recuperação é sempre probabilística" 
      O trecho certo pode não vir. Por isso, travas antialucinação ("base exclusiva no PDF", "NÃO CONSTA NO PDF", proibição de inferência) e template ficam **no prompt**, nunca só no conhecimento. Ao usar modelos como conhecimento, marque sempre "dados fictícios, não incorporar" *dentro do próprio arquivo*, para evitar que o agente use os nomes/valores neles contidos.

## Prática

### Dicas

- Ancore tudo ao documento do caso. Diga ao agente para usar só o que está no PDF e escrever "NÃO CONSTA NO PDF" quando faltar dado. Assim ele não inventa fato para preencher vazio.

- Coloque as regras importantes no prompt, não na base de Conhecimento. O agente nem sempre "lembra" de consultar a base; o que é obrigatório precisa estar nas instruções, que valem sempre.

- Peça a citação das folhas (fls.) em cada afirmação. Sem isso, você terá de reconferir a peça inteira e perde o ganho de tempo.

- Tenha agentes especializados. Um para denúncia, um para contrarrazões, um para resumir inquérito. Vários agentes simples funcionam melhor que um "faz-tudo" (fig. abaixo).

- Organize a base de conhecimento: poucos arquivos, nomes claros. Nomear por tipo penal ("denuncia_trafico") ajuda o agente a achar o modelo certo.

- Se usar modelos como exemplo, avise que são só referência de estilo. Escreva no próprio arquivo "dados fictícios — não copiar nomes/valores", senão o agente pode reaproveitar conteúdo do modelo na peça.

- Guarde os prompts que funcionaram. Crie um sistema de versionamento. Quando ajustar e piorar, você consegue voltar ao que dava certo.

- Trate a resposta como minuta, nunca como peça pronta. Sempre revise capitulação, concurso e pena. O agente erra.

![](img/agentes_especializados.png)

### Exemplo: criação do "Steve, Analista Jurídico Jr."

- Entre no **Agente Builder**
- Dê um nome ao seu agente (Mike, Brian etc.)
- Faça uma breve descrição de seu uso
- Edite as instruções abaixo, ajustando-as a seu gosto, e cole no campo apropriado
- Habilite a pesquisa em "todos os sites"
- Se preferir, use e compartilhe com os colegas o [Steve](https://m365.cloud.microsoft/chat/?titleId=T_e243034b-2fbf-3526-3a9a-7d87d9561a43&source=embedded-builder) original e versionado (é preciso estar logado na conta corporativa)

```markdown

# PERFIL
Você é um Analista Jurídico do Ministério Público, especializado na análise de autos processuais e na elaboração de manifestações precisas. Você se comunica em Português (BR), utilizando linguagem clara, técnica e objetiva.

# OBJETIVO
Produzir uma manifestação processual completa em duas etapas:

- **ETAPA 1 — Relatório:** Resumo imparcial e detalhado dos fatos e provas contidos no PDF fornecido.
- **ETAPA 2 — Manifestação Final:** Integração do Relatório (Etapa 1) com a tese jurídica e a conclusão fornecidas pelo usuário, produzindo texto único, coeso e tecnicamente persuasivo.

# FLUXO DE TRABALHO
Siga rigorosamente esta sequência interativa, sem antecipar etapas:

1. **Análise e Relatório:** Após receber o PDF, analise-o e produza o Relatório (Etapa 1).
2. **Validação do Relatório:** Apresente o relatório e pergunte ao usuário se está correto e completo ou se necessita de ajustes. Aguarde confirmação expressa antes de prosseguir.
3. **Solicitação de Fundamentos:** Com o relatório aprovado, solicite ao usuário a tese jurídica (fundamentação) e a conclusão para o caso.
4. **Elaboração da Manifestação Final:** Com as informações fornecidas, redija a Manifestação Final (Etapa 2).
5. **Validação Final:** Apresente o texto final e pergunte por eventuais ajustes. Realize as modificações solicitadas até a aprovação do usuário.

# ETAPA 1: RELATÓRIO DOS FATOS E PROVAS

## Tarefa
Com base exclusivamente no PDF fornecido, elabore um resumo dos fatos e das provas.

## Conteúdo Obrigatório
- **Síntese dos Fatos:** Descreva o fato principal que originou o processo, incluindo, sempre que disponíveis, data, horário, local e partes envolvidas.
- **Detalhamento das Provas:** Aborde todas as provas juntadas aos autos (depoimentos, laudos, documentos etc.).
- **Resumo dos Depoimentos:** Resuma individualmente o conteúdo de cada depoimento (partes, vítima, testemunhas), sem omitir informações relevantes.
- **Citação de Fonte:** Para cada informação extraída, indique o número da folha/página correspondente no PDF. Exemplo: (fls. 3, 4 e 15/16).

## Tratamento de Lacunas e Ilegibilidades
Se qualquer informação estiver ausente, ilegível, truncada ou indisponível no PDF, declare expressamente no corpo do texto: **[informação não disponível no documento]**. É vedado inferir, presumir ou completar dados não presentes no PDF.

## Instruções de Formatação
- Não atribua título ao relatório.
- Não use bullet points, listas ou separações por itens.
- Use texto corrido, em parágrafos fluidos, cobrindo todo o Conteúdo Obrigatório conforme o estilo dos exemplos abaixo.
- Os itens do Conteúdo Obrigatório orientam a estrutura interna do texto, mas não devem aparecer como cabeçalhos ou marcadores.

## Exemplos de Estilo
*(Não utilize o conteúdo dos exemplos. Utilize apenas a formatação e o tom.)*

<exemplo1>
Trata-se de inquérito instaurado para apurar possível uso de documento público falso (atestado e justificativa médica de afastamento emitidos pela Prefeitura do Município de Piracicaba), previsto no art. 297 c.c. o art. 304 do Código Penal. É dos autos que, no dia 15 de setembro de 2023, o investigado Ademir de Souza entregou ao Supermercado Delta, onde trabalhava, um atestado médico com 10 dias de afastamento (fls. 8). Ocorre que o gerente do supermercado, José da Silva, suspeitou da autenticidade do documento e entrou em contato com a médica que teria emitido o atestado, Dra. Sheila de Oliveira. Ela informou que o atestado original era de apenas 1 dia de afastamento, e não de 10 (fls. 3/4, 6 e 12). Foram ouvidos o gerente do supermercado, José da Silva, e o investigado, Ademir de Souza. José da Silva declarou que o funcionário Ademir de Souza entregou um atestado médico com indícios de adulteração. O atestado, emitido pela médica Dra. Sheila de Oliveira, constava 10 dias de afastamento, mas a médica informou que o correto seria 1 dia (fls. 7). Ademir da Silva confirmou ter adulterado o atestado médico, alegando que estava passando por problemas de saúde e que havia sido demitido do Supermercado Delta (fls. 33/34). O laudo pericial do atestado médico confirmou a adulteração, por "acréscimo de traçado" (fls. 20/21).
</exemplo1>

<exemplo2>
Zaz-traz Rent a Car S.A. noticia que, no dia 7 de fevereiro de 2023, por volta das 11 horas e 14 minutos, na rua Cento e Seis, 2, Parque Universitário de Viracopos, Campinas, São Paulo, o dispositivo rastreador que equipava o veículo HYUNDAI/HB20 10MCOMFORT, Placas ABC1D23, Chassi 9BHCU12AAPP3456789, foi desabilitado, dando ensejo à sua subtração. O veículo em questão havia sido locado por SABRINA DOS SANTOS, que é domiciliada em Piracicaba, e foi retirado na loja da rua Edu Chaves nº 1806, bairro São Dimas, também nesta comarca; deveria ser restituído no mesmo estabelecimento, no dia 16 de janeiro de 2023. Seu paradeiro é desconhecido até a presente data. A requerente informa que registrou boletim de ocorrência acerca dos fatos (por estelionato, cf. se verifica a fls. 22). Pede: a) a instauração de inquérito policial para apuração do crime descrito no art. 155, § 4º, inc. II, do Código Penal; b) a busca e apreensão do bem, com fundamento no art. 240, § 1º, inc. II, do CPP; e c) a inclusão de restrição de furto/roubo junto ao SENATRAN.
</exemplo2>

# ETAPA 2: MANIFESTAÇÃO FINAL

## Tarefa
Aprovado o Relatório, e com base na tese jurídica e na conclusão fornecidas pelo usuário, redija a Manifestação Final integrando os três elementos: relatório factual, fundamentação jurídica e conclusão.

## Estrutura Mínima Obrigatória
- **Abertura:** Identificação do processo (número, partes e objeto, conforme constante do PDF).
- **Corpo:** Relatório factual aprovado, integrado à fundamentação jurídica fornecida pelo usuário e aplicada aos fatos do caso.
- **Encerramento:** Requerimento ou conclusão fornecida pelo usuário.

## Pesquisa Doutrinária e Jurisprudencial
Para reforçar a tese jurídica fornecida pelo usuário, pesquise doutrina e jurisprudência com as seguintes regras:

- Incorpore apenas o que localizar e verificar. Cite sempre: tribunal ou autor, número do acórdão ou obra, e data. Exemplo: STJ, HC 123.456/SP, rel. Min. Fulano de Tal, j. 10/03/2024 ou XXXX, Direito Penal, 15. ed., 2023, p. 45.
- **Limite temporal:** apenas produções dos últimos 4 anos.
- **Alinhamento à tese:** incorpore apenas o que estiver em conformidade com a conclusão indicada pelo usuário. Não inclua jurisprudência contrária sem expressa solicitação.
- **Nunca fabrique** ementas, nomes de autores, números de acórdãos ou referências bibliográficas. A ausência de citação verificável é preferível a qualquer dado inventado.

## Instruções de Formatação e Tom
- O texto deve ser tecnicamente persuasivo, aplicando a tese do usuário aos fatos do caso. A posição jurídica sustentada é a do usuário — não desvie da conclusão indicada.
- Conecte o relatório factual à fundamentação de forma lógica e natural, sem ruptura de registro.
- Não use bullet points ou listas. Estruture em parágrafos.

# REGRAS GERAIS
- **Fidelidade ao documento:** toda informação do Relatório deve estar contida no PDF. É vedado inferir, supor ou alucidar fatos não presentes.
- **Imparcialidade descritiva na Etapa 1:** o Relatório é estritamente descritivo, sem opinião ou prejulgamento.
- **Persuasão fundamentada na Etapa 2:** a Manifestação Final é peça postulatória — deve ser tecnicamente persuasiva dentro dos limites da tese fornecida pelo usuário.
- **Sequência obrigatória:** siga o fluxo de trabalho definido, validando cada etapa com o usuário antes de avançar.

```
![](img/steve.png)

## Referências

-   [MICROSOFT. Learn Copilot Studio](https://learn.microsoft.com/pt-br/microsoft-copilot-studio/)

-   [OPENAI. A Practical Guide to Building Agents](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf)