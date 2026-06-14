---
title: "3. Agentes do Copilot"
---

## O que são "Agentes" Copilot

- **Autonomia não é "sim ou não", é uma escala.** Vai do que só responde quando você pede até o que age sozinho. A Microsoft chama tudo de "agente", mas há níveis bem diferentes.

- **O que seria um agente "de verdade":** decide sozinho o que fazer; executa ações (não só escreve); faz vários passos em sequência; e começa por conta própria.

- **A maioria dos agentes criados no Copilot:** tem uma persona (instruções) e consulta uma base de conhecimento. Você pede, ele responde e para. É um **assistente bem configurado, não um agente que age sozinho.**

- **E quando ele consulta a web?** O que importa não é *acessar* a internet, e sim *quem decide os passos*:
    - busca simples (uma consulta e responde) → continua reativo, só com fonte mais ampla;
    - busca que itera sozinha (formula buscas, segue links, refaz) → aí, sim, sobe na escala.

- **O Copilot Studio completo:** pode usar ferramentas, decidir sozinho o que consultar e ser disparado automaticamente → **Aí, sim, age como agente.**

- **Na Promotoria, controlar é melhor que automatizar tudo:** vale mais um assistente que para e espera sua revisão do que um que age e decide tudo sozinho.

## Agente declarativo X Agente autônomo 

| Critério | Agente declarativo (Copilot) | Agentes autônomo (Claude Code, Codex, Devin etc.) |
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

### Criação de agentes sem programação
A configuração é feita por interface gráfica, sem código:

- **Persona e escopo:** nome, instruções de sistema, tom e limites de atuação do agente
- **Base de conhecimento:** upload de arquivos (modelos, tabelas, referências) que o agente consulta para gerar respostas
- **Distribuição interna:** compartilhamento direto no Teams ou SharePoint, sem publicação externa
- **Configuração controlada:** sem memória entre sessões, sem conectores ativos, sem acesso a APIs externas (o agente responde só com o que você definiu).

### Resultado prático
Um agente configurado com boas instruções e base de conhecimento adequada
entrega respostas consistentes, no padrão da unidade, acessíveis a qualquer
membro da equipe, sem exigir que cada usuário domine prompts ou ferramentas de IA.

## Função "Conhecimento"

**O que é?** É uma base consultável por recuperação (RAG): o agente busca trechos relevantes desses arquivos quando precisa, em vez de tê-los sempre fixos no contexto. 

- No prompt (instruções): tudo que precisa ser aplicado *sempre*, de forma determinística.

- Na base de conhecimento: material de referência estável, que é consultado conforme o caso. 

**Cuidado com vazamento dos exemplos.** Se colocar os modelos como conhecimento, mantenha a marcação de "dados fictícios — referência só de estilo, não incorporar" dentro do próprio arquivo, porque o agente pode recuperar e copiar nomes/valores fictícios. A restrição que reinserimos no prompt continua valendo.

- Exemplo: 
1. Catálogo de modelos de denúncia;
2. Manual com regras

Advertência:

- **Não confie no Conhecimento para regras obrigatórias.** A recuperação é probabilística; o trecho relevante pode não ser trazido numa consulta específica. Logo, as travas antialucinação ("base exclusiva no PDF", "NÃO CONSTA NO PDF", proibição de inferência) e o template *ficam no prompt*, nunca só no conhecimento.

!!! tip "Aponte para o arquivo, não para o site"
    Se a sua licença não permitir carregar arquivos na base de conhecimento,
    insira a URL direta do arquivo no SharePoint no campo correspondente.
    Evite apontar para o site ou a biblioteca inteira (quanto menor o escopo,
    mais precisa a recuperação).

    Outra vantagem: para atualizar a base, basta editar o arquivo no SharePoint.
    O agente passa a usar a versão nova automaticamente, sem reconfiguração.

    Atenção às permissões: o agente acessa o arquivo com as credenciais do
    usuário. Se o arquivo estiver em biblioteca restrita, outros membros da
    equipe podem não conseguir recuperar o conteúdo ao usar o agente.

## Exemplos

### "Agente" **Denunciador** com base de conhecimento

```markdown
<papel_e_tarefa>
Você é um Promotor de Justiça. Elabore uma denúncia criminal com base EXCLUSIVA no inquérito policial fornecido em PDF. Não infira fatos, não preencha lacunas criativamente e não incorpore dados externos.
</papel_e_tarefa>

<modelos_de_estilo>
A base de conhecimento contém o arquivo "modelos_estilo_denuncia" com exemplos de denúncia por tipo penal e outras especificidades (tráfico, furto, múltiplas imputações). ANTES de redigir, consulte o modelo cujo tipo penal mais se aproxime do caso e siga seu tom, estrutura e redação forense. Os dados dos modelos são FICTÍCIOS: use-os apenas como referência de estilo; NUNCA incorpore nomes, valores ou fatos deles à peça gerada.
</modelos_de_estilo>

<estilo>
Tom objetivo, impessoal e técnico. Nomes de réus em CAIXA ALTA. Linguagem forense (ex.: "trazia consigo, para fins de tráfico"; "subtraiu para si"; "ofendeu a integridade corporal"). Cada afirmação fática remete às folhas do IP (cf. fls. X).
</estilo>

<formatacao>
Redija a peça em PROSA CORRIDA. NÃO use marcadores, listas, negrito, títulos ou linhas em branco entre as orações do pedido final. Lista numerada SÓ no ROL de testemunhas. Vários crimes de um MESMO réu vão na mesma frase, separados por vírgula, encerrando com a forma de concurso (ex.: "incurso nos artigos 129, §13, e 163, caput, ambos do Código Penal, na forma do art. 69 do CP").
</formatacao>

<regras>
RITO (precedência):
1. TRÁFICO (arts. 33–37, Lei 11.343/06): rito dos arts. 55 e ss. da Lei 11.343/06; réu NOTIFICADO p/ defesa prévia em 10 dias; rol máx. 5.
2. DEMAIS CRIMES: réu CITADO p/ responder à acusação por escrito (arts. 394 e ss. CPP). Ordinário (pena máx. ≥4 anos — art. 394, §1º, I): rol máx. 8. Sumário (pena máx. 2–4 anos — art. 394, §1º, II): rol máx. 5.

QUALIFICAÇÃO: sempre a fórmula "qualificado a fls. X".

NARRATIVA: os parágrafos "Consta..." descrevem cada conduta (o quê, quando, onde, como, por quem) com remissão às fls. e com o verbo nuclear e os elementos do tipo na redação legal (ex.: lesão em VD → "ofendeu a integridade corporal de [vítima], por razões da condição do sexo feminino"; furto → "subtraiu para si coisa alheia móvel"); as circunstâncias concretas ficam ao redor do núcleo. Use "Consta, ainda/por fim, que" apenas para introduzir IMPUTAÇÕES distintas. Em seguida, abra UM ÚNICO bloco "Apurou-se que" — narrativa corrida do modus operandi em 1 a 5 parágrafos. NÃO repita "Apurou-se" no início dos parágrafos seguintes nem fragmente a narrativa por crime.

CONCURSO (se 2+ crimes, identificar a modalidade antes de capitular):
- Material (art. 69): ações independentes, crimes distintos.
- Formal (art. 70): uma ação, dois ou mais resultados.
- Continuado (art. 71): mesma espécie, condições semelhantes de tempo, lugar e modo.
Mesmo tipo penal praticado mais de uma vez: registrar "(por N vezes)" após o artigo.

COERÊNCIA: cada vítima e cada conduta típica apurada gera uma imputação própria. A capitulação deve refletir TODAS as imputações listadas na análise preliminar (ex.: lesão contra a esposa E lesão contra a filha = duas imputações).

REPARAÇÃO (art. 387, IV, CPP):
- Prejuízo patrimonial quantificável (furto, estelionato, dano, apropriação indébita, incêndio): usar o valor do PDF; se ausente, estimar.
- Violência doméstica e familiar contra a mulher: incluir danos MATERIAIS E MORAIS e fixar valor mínimo razoável; NÃO usar "a ser apurado em liquidação".
- Sem dano mensurável (ameaça isolada, porte de droga): omitir o pedido.
- Concurso material com múltiplos eventos: juros da data do último evento (Súmula 54/STJ); correção (Súmula 362/STJ).

ROL: respeitar o limite do rito. Formato: Nome (categoria, fls. X). Categorias: vítima | policial req. | testemunha. Excedente vai para a análise preliminar, indicando os mais relevantes.

LAUDOS PENDENTES: se o IP indicar laudo requisitado e não juntado, protestar pela juntada no pedido final.
</regras>

<notacao>
[SE ...] = bloco condicional. {{...}} = campo a extrair do PDF. Datas por extenso nos parágrafos "Consta..." (ex.: 29 de abril de 2026, por volta das 19 horas e 6 minutos).
</notacao>

<analise_preliminar_obrigatoria>
Antes de redigir a denúncia, produza obrigatoriamente o bloco abaixo. Ele NÃO integra a peça final.

ANÁLISE PRELIMINAR (não integra a peça)
1. Indiciados: {{Nome completo}} — qualificado a fls. {{X}}
2. Vítimas: {{Nome ou descrição}}
3. Fato e capitulação: síntese / dispositivo(s) violado(s) — UM por conduta/vítima / rito / concurso (Sim — modalidade ou Não)
4. Provas relevantes: {{laudo / auto / foto / vídeo — fls. X}}
5. Depoimentos: {{Nome}} (fls. {{X}}): resumo em até 2 parágrafos
6. Rol de testemunhas: {{Nome}} — {{categoria}} — fls. {{X}}
7. Lacunas: {{descrever ou "Nenhuma"}}
</analise_preliminar_obrigatoria>

<template>
EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

IP nº {{número do inquérito}}

[SE EPISÓDIO ÚNICO]
Consta do incluso inquérito policial que, no dia {{data por extenso}}, {{por volta das ... / em horário indeterminado}}, {{em/na}} {{local completo}}, {{NOME(S); se mais de um: "em unidade de desígnios e comunhão de esforços,"}}, qualificado(s) a fls. {{X}}, {{descrição objetiva da conduta com remissão às provas}}.

[SE MÚLTIPLAS IMPUTAÇÕES]
Consta do incluso inquérito policial que, no dia {{data por extenso}}, [...], {{NOME(S)}}, qualificado(s) a fls. {{X}}, {{descrição da 1ª imputação com remissão às provas}}.
Consta, ainda, que, {{no mesmo contexto / no dia ...}}, [...], {{NOME(S)}} {{descrição da imputação com remissão às provas}}. [repetir "Consta, ainda, que," para imputações intermediárias]
Consta, por fim, que, [...], {{NOME(S)}} {{descrição da última imputação com remissão às provas}}.

Apurou-se que {{narrativa fática corrida em 1 a 5 §§, do modus operandi, SEM repetir "Apurou-se" nos parágrafos seguintes}}.

Diante do exposto, {{denuncio / DENUNCIO}} a Vossa Excelência {{NOME}} como incurso {{no artigo X / nos artigos X, Y e Z}} {{, na forma do art. 69/70/71 do CP, se concurso}}, requerendo que, recebida e autuada esta, seja o denunciado {{[TRÁFICO] notificado para apresentar defesa prévia no prazo de 10 (dez) dias, seguindo-se o rito dos arts. 55 e ss. da Lei nº 11.343/06 / [DEMAIS] citado para responder à acusação por escrito, seguindo-se o rito dos arts. 394 e ss. do CPP}}, até final condenação, {{[SE REPARAÇÃO] com fixação de valor mínimo de R$ {{valor}} para reparação dos {{danos materiais / danos materiais e morais}} causados pela infração (art. 387, inc. IV, do CPP), sobre o qual deverá incidir correção monetária (Súmula 362/STJ) e juros moratórios contados da data do {{evento / último evento criminoso}} (Súmula 54/STJ),}} {{[SE LAUDO PENDENTE] protestando, desde já, pela juntada dos laudos requisitados a fls. {{X}},}} ouvindo-se, no curso da instrução, as seguintes pessoas:

ROL:
1. {{Nome}} ({{categoria}}, fls. {{X}});
2. {{Nome}} ({{categoria}}, fls. {{X}}).
</template>

<restricoes>
- Toda informação factual deve vir exclusivamente do PDF do caso. Escreva "NÃO CONSTA NO PDF" quando faltar dado essencial e registre a lacuna na análise preliminar.
- Proibido reproduzir CPF, RG ou endereço no corpo da peça.
- Proibido incorporar nomes, valores ou fatos dos modelos de estilo.
- Proibido preencher lacunas com inferências ou suposições.
</restricoes>
```

```txt
=====================================================================
MODELOS DE ESTILO — DENÚNCIA CRIMINAL
Base de conhecimento do agente "Denunciador"
=====================================================================

INSTRUÇÕES DE USO
---------------------------------------------------------------------
- Este arquivo serve EXCLUSIVAMENTE como referência de estilo, tom e
  estrutura para a redação de denúncias criminais.
- TODOS os dados abaixo (nomes, datas, locais, valores, números de
  fls. e de IP) são FICTÍCIOS.
- NÃO incorpore nenhum dado destes modelos à denúncia gerada. Os fatos
  da peça devem vir exclusivamente do inquérito policial em PDF do caso
  concreto.
- Selecione o modelo cujo tipo penal mais se aproxime do caso e siga
  seu fraseado forense, a estrutura dos parágrafos "Consta..." e
  "Apurou-se que", a capitulação e o formato do rol.

Modelos disponíveis:
  1. Tráfico de drogas — réu único, rito da Lei 11.343/06.
  2. Furto qualificado — réu único, com pedido de reparação.
  3. Múltiplas imputações — concurso material, violência doméstica.


=====================================================================
MODELO 1 — TRÁFICO DE DROGAS
=====================================================================

EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

IP nº 1500000-00.2026.8.26.0000

Consta do incluso inquérito policial que, no dia 15 de março de 2026, por volta das 21 horas, na rua das Palmeiras, s/nº, esquina com a rua dos Ipês, bairro Jardim das Flores, nesta cidade e comarca, CARLOS ROBERTO LIMA, qualificado a fls. 05, trazia consigo, para fins de tráfico e entrega a consumo de terceiros, 87 microtubos de cocaína, com massa líquida de 21,7g, e 34 pedras de "crack" (derivado da cocaína), com massa líquida de 6,8g, substâncias entorpecentes que determinam dependência física e psíquica, sem autorização e em desacordo com determinação legal e regulamentar (cf. auto de apreensão de fls. 14 e laudo pericial de fls. 58/60).

Apurou-se que o denunciado praticava o tráfico no local dos fatos, ponto já conhecido nos meios policiais por essa atividade ilícita. Policiais militares em patrulhamento avistaram CARLOS entregando um objeto ao ocupante de um veículo que, diante da aproximação da viatura, se evadiu. O denunciado foi abordado e, em busca pessoal, foram localizados a sacola com as drogas descritas e R$ 120,00 em espécie, produto do narcotráfico. A quantidade, o fracionamento em microtubos e a posse de numerário evidenciaram a destinação mercantil dos entorpecentes.

Diante do exposto, DENUNCIO a Vossa Excelência CARLOS ROBERTO LIMA como incurso no artigo 33, caput, da Lei nº 11.343/06, requerendo que, recebida e autuada esta, seja o denunciado notificado para apresentar defesa prévia no prazo de 10 (dez) dias, seguindo-se o rito dos artigos 55 e seguintes da referida lei, até final condenação, com decretação de perda dos bens e valores relacionados ao crime, ouvindo-se, em instrução, as seguintes testemunhas:

ROL:
1. Marcos Aparecido Sousa (policial militar, req., fls. 03);
2. Ricardo Ferreira dos Santos (policial militar, req., fls. 04).


=====================================================================
MODELO 2 — FURTO QUALIFICADO
=====================================================================

EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

IP nº 1508000-00.2025.8.26.0000

Consta do incluso inquérito policial que, no dia 10 de setembro de 2025, por volta das 3 horas e 20 minutos, na avenida das Acácias nº 500, nas dependências do estabelecimento comercial "Comércio Modelo Ltda.", nesta cidade e comarca, FÁBIO HENRIQUE MARTINS, qualificado a fls. 29, mediante escalada e rompimento de obstáculos, subtraiu para si cigarros, bebidas e R$ 80,00 em moedas, totalizando prejuízo de aproximadamente R$ 1.200,00, pertencentes à empresa-vítima (cf. boletim de ocorrência de fls. 02/04 e relatório de fls. 60/61).

Apurou-se que o denunciado se dirigiu ao estabelecimento com o propósito de furtar sua loja de conveniência. Para ingressar, FÁBIO pulou o gradil de vedação e, com uma chave de fendas, arrombou a porta lateral. Ato contínuo, subtraiu para si os objetos descritos. Toda a ação foi registrada pelas câmeras de segurança do local (fotografias de fls. 35/40 e mídia digital de fls. 41/49). Reconhecido pelas imagens, o indiciado foi identificado e, ao ser interrogado, confessou a autoria do crime.

Diante do exposto, denuncio a Vossa Excelência FÁBIO HENRIQUE MARTINS como incurso no artigo 155, § 4º, incisos I e II, do Código Penal, requerendo que, recebida e autuada esta, seja o denunciado citado para responder à acusação por escrito, seguindo-se o rito estabelecido pelos artigos 394 e seguintes do Código de Processo Penal, até final condenação, com fixação de valor mínimo de R$ 1.200,00 para reparação dos danos causados pela infração (art. 387, IV, CPP), sobre o qual incidirão correção monetária (Súmula 362/STJ) e juros moratórios contados da data do evento criminoso (Súmula 54/STJ), ouvindo-se, no curso da instrução, as seguintes testemunhas:

ROL:
1. João Batista Almeida (vítima/representante, fls. 16);
2. Sílvio Costa Pereira (policial civil, req., fls. 05).


=====================================================================
MODELO 3 — MÚLTIPLAS IMPUTAÇÕES (CONCURSO MATERIAL)
=====================================================================

EXCELENTÍSSIMO SENHOR JUIZ DE DIREITO DA ____ VARA CRIMINAL DA COMARCA DE PIRACICABA-SP

IP nº 1500000-00.2025.8.26.0000

Consta do incluso inquérito policial que, no dia 17 de março de 2025, por volta das 23 horas e 30 minutos, na rua das Hortênsias nº 45, bairro Jardim Europa, nesta comarca, RICARDO HENRIQUE MOURA COSTA, qualificado a fls. 65, ameaçou, por palavra, por razões da condição do sexo feminino, sua ex-companheira Fernanda Alves Pereira, de causar-lhe mal injusto e grave (fls. 3/5).

Consta, ainda, que, em seguida, na rua dos Cravos nº 210, apto 32, bairro Vila Nova, nesta comarca, RICARDO HENRIQUE MOURA COSTA causou incêndio em casa destinada a habitação, expondo a perigo o patrimônio de outrem (fls. 19, 20 e laudo de fls. 68/70).

Consta, ainda, que, no dia 7 de abril de 2025, por volta das 2 horas, na rua das Magnólias nº 180, bairro Jardim Primavera, nesta comarca, RICARDO HENRIQUE MOURA COSTA descumpriu decisão judicial que deferiu medidas protetivas de urgência em favor da ex-companheira Fernanda Alves Pereira (fls. 38/39).

Consta, por fim, que, no dia 26 de abril de 2025, por volta das 16 horas e 50 minutos, na rua São Benedito nº 75, bairro Vila Rezende, nesta comarca, RICARDO HENRIQUE MOURA COSTA descumpriu decisão judicial que deferiu medidas protetivas de urgência em favor da ex-companheira Fernanda Alves Pereira (fls. 34/35).

Apurou-se que RICARDO e Fernanda foram conviventes por 5 anos e têm dois filhos juntos. Na época dos fatos, o casal havia se separado.

No dia 17 de março de 2025, Fernanda surpreendeu o denunciado na companhia de outra mulher em um bar do bairro. Ambos discutiram e RICARDO empurrou a ex-companheira. Na sequência, o denunciado disse a Fernanda que sumiria com os filhos do casal, deixando-a atemorizada.

Em seguida, RICARDO e Fernanda se encontraram na casa da mãe do indiciado. Nesse momento, o denunciado empunhou uma faca e, dirigindo-se à vítima, disse que a mataria.

Feito isso, o denunciado fugiu do local ao saber que a Polícia Militar havia sido acionada. Ele rumou à residência da vítima, na rua dos Cravos. Ali, arrombou a porta e ateou fogo na casa, incendiando-a.

Depois desse episódio, a vítima solicitou e obteve medidas protetivas de urgência decretadas pelo Juízo de Plantão, no dia 19 de março de 2025, nos autos nº 1500123-00.2025.8.26.0000. Por elas, o indiciado foi afastado do lar e proibido de se aproximar da vítima e dos familiares, de manter contato com ela e de frequentar seu local de trabalho. RICARDO tomou ciência da ordem judicial no dia seguinte (cf. fls. 13).

No dia 7 de abril de 2025, RICARDO foi à residência da vítima no meio da madrugada, alegando que queria ver os filhos, pois havia tentado se matar. A vítima não cedeu e o repeliu.

No dia 26 de abril de 2025, RICARDO desatendeu, outra vez, a ordem judicial. Aproximou-se da vítima nas imediações da escola do filho Lucas, sabendo que ela ali estaria para buscá-lo. Na mesma data, passou diante da residência da irmã dela, ciente de que Fernanda se encontrava no local, e a cumprimentou, fazendo-o para marcar presença e afligir a ex-companheira.

Diante do exposto, denuncio a Vossa Excelência RICARDO HENRIQUE MOURA COSTA como incurso nos artigos 147, § 1º, e 250, § 1º, inc. II, alínea "a", ambos do Código Penal, e art. 24-A da Lei nº 11.340/06 (por duas vezes), na forma do artigo 69 do Código Penal, requerendo que, recebida e autuada esta, seja o denunciado citado para responder à acusação por escrito, seguindo-se o rito estabelecido pelos artigos 394 e seguintes do Código de Processo Penal, até final condenação, com fixação do valor mínimo de R$ 10.000,00 para reparação dos danos materiais e morais causados pela infração (art. 387, inc. IV, do CPP), sobre o qual deverá incidir correção monetária (Súmula 362 do STJ) e juros moratórios contados da data do último evento criminoso (Súmula 54 do STJ), ouvindo-se, no curso da instrução, as seguintes testemunhas:

ROL:
1. Fernanda Alves Pereira (vítima, fls. 7/8, 36, 40);
2. Márcia Fernanda Lima (fls. 63);
3. Patrícia de Souza Ramos (intimar através da vítima).

```

## Referências

-   [Microsoft. Learn Copilot Studio](https://learn.microsoft.com/pt-br/microsoft-copilot-studio/)

-   [Microsoft. Learn Copilot Microsoft 365](https://learn.microsoft.com/pt-br/copilot/microsoft-365/)