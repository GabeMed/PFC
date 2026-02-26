# IA Operacional — Alinhamento de Escopo
### Projeto de Fim de Curso — IME
### Fevereiro 2026

---

## 1. O Problema

O Exército mantém um acervo extenso de manuais de campanha que formam a base doutrinária das operações. Na prática, a consulta a essa doutrina é difícil: o volume é grande, a resposta a uma situação tática frequentemente exige cruzar informações de múltiplos manuais, existe uma hierarquia de precedência entre as normas que nem sempre é evidente, e boa parte do conteúdo depende de diagramas e simbologia tática.

Mas o problema mais sutil é outro: quando um militar considera diferentes cursos de ação, ele precisa não apenas saber o que a doutrina recomenda — ele precisa entender por que determinadas alternativas *não devem* ser adotadas, e qual norma de precedência superior fundamenta essa restrição. Essa verificação cruzada é trabalhosa, sujeita a erro, e hoje depende inteiramente da experiência individual do operador.

A proposta deste projeto é construir a **IA Operacional**: um sistema de consulta doutrinária que não apenas recupera informações, mas raciocina sobre a doutrina e é capaz de explicar por que alternativas táticas violariam normas vigentes de precedência superior.

---

## 2. A Contribuição Central: Raciocínio Contrafactual

O diferencial do sistema é o que chamamos de **raciocínio contrafactual**. Quando o usuário faz uma pergunta sobre uma situação tática, o sistema:

1. **Recupera e recomenda** o curso de ação previsto na doutrina, com citação exata de fontes (manual, capítulo, seção, página).

2. **Busca alternativas** — identifica outras abordagens ou manobras que poderiam parecer aplicáveis àquela situação tática.

3. **Verifica cada alternativa contra a hierarquia de precedência** — consultando manuais de nível superior para identificar se alguma norma restringe ou contraindica aquela alternativa.

4. **Explica o contrafactual** — para cada alternativa que viola uma norma de precedência superior, gera uma explicação rastreável: *"A Manobra B, embora descrita no Manual X para contextos similares, contraria a diretriz do Manual Y (precedência superior), que estabelece [citação]. Portanto, não é aplicável nesta situação."*

Esse mecanismo é viável no nosso contexto porque operamos sobre um **corpus fechado com regras de precedência conhecidas**. Não estamos pedindo à IA que imagine cenários hipotéticos — estamos pedindo que faça uma rodada adicional de busca e compare o que encontrou contra uma hierarquia conhecida. Os blocos de construção necessários (busca semântica, mapa de precedência, geração com citação) já são componentes padrão de sistemas RAG avançados. O contrafactual emerge da orquestração desses componentes, não de uma capacidade nova e não testada.

Esse tipo de raciocínio não foi encontrado em nenhum sistema quando fizemos na nossa pesquisa, e acreditamos que seria um diferencial interessante para uma tese de fim de curso.

---

## 3. Arquitetura e Capacidades de Suporte

O motor de contrafactuais se apoia em uma infraestrutura de busca e raciocínio que inclui:

**Fluxo com Grafo de Estados (LangGraph):** a consulta do usuário passa por um fluxo estruturado — classificação de intenção, busca primária, geração de resposta, busca de alternativas, verificação de precedência, síntese contrafactual. Cada etapa é um nó no grafo de estados. O módulo contrafactual pode ser ativado ou desativado por configuração sem afetar o funcionamento base.

**Busca Hierárquica em Dois Níveis:** em vez de buscar em todo o acervo simultaneamente, o sistema primeiro identifica os manuais mais relevantes à pergunta (usando resumos dos manuais como primeiro filtro), e depois faz a busca detalhada apenas dentro desses manuais. Essa abordagem é similar ao que já se faz com o chunk-0 de resumo nas collections, mas com um mecanismo de filtragem em profundidade mais estruturado.

**Mapa de Precedência Doutrinária:** um grafo de relacionamentos entre os manuais (qual substitui qual, qual complementa qual, qual referencia qual), construído inicialmente com apoio da área negocial e mantido como configuração do sistema. O motor contrafactual consulta esse mapa para determinar qual fonte tem prioridade quando há conflito. A IA pode auxiliar na pré-extração das referências cruzadas entre manuais, mas o mapa será validado por especialistas antes de entrar em operação.

**Busca Híbrida com Chunks Contextualizados:** combinação de busca por palavras-chave (BM25, adequada para terminologia militar e jargões) e busca semântica vetorial, com reranqueamento. Cada trecho armazenado carrega metadados (manual de origem, capítulo, seção, página), eliminando a perda de contexto típica de sistemas RAG convencionais.

**Arquitetura Agnóstica a Modelo e Interoperável (MCP):** o sistema funciona com qualquer LLM (GPT, Claude, Llama, Qwen), qualquer modelo de embedding, e qualquer banco vetorial — a troca entre provedores é feita por configuração, sem alteração de código. As capacidades do sistema são expostas via Model Context Protocol (MCP), o padrão aberto de integração de agentes de IA adotado por OpenAI, Google, Microsoft e Anthropic sob a Linux Foundation. Isso permite que qualquer sistema externo (como o EBOT) consuma as capacidades de raciocínio doutrinário como ferramenta nativa.

**Processamento de Conteúdo Visual:** integração de modelos de visão (como ColPali) para processar diagramas, quadros organizacionais e simbologia tática. Essa capacidade depende provavelmente de um acesso a GPU — sem ela, o sistema opera normalmente com processamento textual e o processamento visual fica como aprimoramento documentado na arquitetura.

---

## 4. O Que NÃO Faz Parte do Escopo

- Não é ferramenta de redação, revisão ou alteração de doutrina — é exclusivamente consulta.
- Não opera com dados classificados — o protótipo funciona no nível de segurança do acervo fornecido.
- A integração de interface (front-end) com o EBOT — nosso escopo seria o motor de raciocínio (API + MCP).
- O protótipo não é um sistema de produção com escalabilidade horizontal ou controle de acesso — esses aspectos são documentados na arquitetura mas não implementados.
- Não detecta automaticamente atualizações de doutrina — trabalhamos com um snapshot fixo do acervo.

---

## 5. Entregáveis

**Documentação de Arquitetura de Software:** diagramas UML (Casos de Uso, Componentes, Sequência, Classes, Implantação), especificação de interfaces, decisões de projeto e blueprint para eventual implantação em produção.

**Protótipo Funcional:** sistema demonstrando o fluxo completo — busca hierárquica, busca híbrida com reranqueamento, geração de respostas com rastreabilidade de fontes, motor de raciocínio contrafactual doutrinário, memória conversacional, exposição MCP, e troca de modelos por configuração (demonstrada com pelo menos 2 LLMs diferentes).

**Relatório de Avaliação:** métricas quantitativas via framework RAGAS (Faithfulness, Answer Relevancy, Context Precision, Context Recall), avaliação específica da acurácia contrafactual (capacidade de identificar corretamente quais alternativas violam quais normas), e validação qualitativa por conhecedores do domínio.

**Monografia de Fim de Curso.**

---

## 6. Cronograma Proposto

| Período | Atividade |
| :--- | :--- |
| **Mar–Abr** | Formalização do escopo, configuração do ambiente de desenvolvimento, obtenção e análise do acervo de manuais, construção do mapa de precedência, revisão bibliográfica. |
| **Abr–Mai** | Pipeline de ingestão: processamento dos PDFs, chunking hierárquico, geração de resumos por manual, armazenamento vetorial com metadados. |
| **Mai–Jul** | Pipeline de busca e raciocínio: busca hierárquica em dois níveis, busca híbrida com reranqueamento, implementação do grafo de estados (LangGraph), interfaces agnósticas, motor de raciocínio contrafactual. |
| **Jul–Ago** | Exposição MCP, integração de múltiplos LLMs, memória conversacional, refinamento do motor contrafactual. |
| **Ago–Set** | Avaliação: métricas RAGAS, testes de acurácia contrafactual, validação com especialistas, refinamentos. |
| **Set–Out** | Redação final da monografia e preparação para defesa. |

---

## 7. Riscos Identificados

| Risco | Mitigação |
| :--- | :--- |
| **Mapa de precedência incompleto ou ambíguo** | A IA auxilia na pré-extração de referências cruzadas, mas o mapa é validado por especialista antes de entrar em operação. Começamos com um subconjunto de manuais e expandimos incrementalmente. |
| **Acesso a GPU** | O processamento visual (ColPali) requer GPU. Sem ela, o sistema funciona integralmente com processamento textual. A capacidade visual estaria documentada na arquitetura e seria ativada quando o recurso estiver disponível. |
| **Qualidade do raciocínio contrafactual** | O contrafactual opera sobre busca em corpus fechado com precedência conhecida, limitando o risco de alucinação. Avaliação rigorosa com conjunto de teste curado por especialistas. O sistema funciona completamente sem o módulo contrafactual — ele pode ser desativado por configuração. |

---

## 8. O Que Precisamos Para Iniciar

1. Validação e refinamento deste escopo
2. Acesso ao acervo de manuais de campanha (um subconjunto representativo já permite iniciar)
3. Informações sobre a hierarquia de precedência entre os manuais (mesmo que parcial)
4. Definição sobre a possibilidade de uso de GPU (servidor do IME ou EB)
5. Ponto de contato com a equipe do EBOT para alinhamento de interface
