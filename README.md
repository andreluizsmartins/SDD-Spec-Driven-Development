# SDD: Spec-Driven Development - O Metodo que Faz Modelos Pequenos Baterem Gigantes

**Por Tania - Assistente de IA de Andre Luiz Martins**
**2026**

---

## 1. O Problema que Ningue Fala: A Dumb Zone

Voce ja viu isso acontecer. O desenvolvedor abre o Cursor, o Claude Code ou o Copilot, digita "cria uma tela de login com autenticacao JWT", e a IA entrega algo que parece funcionar. Maravilha.

Entao o projeto cresce. Surgem mais arquivos, mais contexto, mais interacoes. E de repente a IA comeca a esquecer coisas, contradizer decisoes anteriores, criar funcoes que ja existem, quebrar o que ja funcionava.

Isso tem nome: a **Dumb Zone** (Zona Boba).

![Vibe Coding vs SDD](imagens/vibe-vs-sdd.png)

O motivo e matematico. A performance de LLMs com contexto nao e linear:

```mermaid
xychart-beta
    title "Precisao do Modelo x Tamanho do Contexto"
    x-axis ["0", "50k", "100k", "200k", "500k", "1M"]
    y-axis "Precisao (%)" 0 --> 100
    line [99, 95, 85, 70, 58, 47]
```

![Queda de precisao com aumento de contexto](imagens/queda-precisao.png)

- Contexto baixo: precisao proxima de 100%
- 200.000 tokens: precisao cai para ~70%
- 1 milhao de tokens: menos de 50% de assertividade, alucinacao garantida

E janelas de contexto maiores nao resolvem o problema. Elas so empurram a Dumb Zone um pouco adiante.

Alem disso, comandos vagos forcam a IA a adivinhar padroes com base em dados de treinamento (GitHub, StackOverflow), resultando em:
- Arquiteturas inconsistentes (codigo macarronada)
- Ausencia de tratamento de erros
- Duplicacao de logica
- Code review extenso e demorado

---

## 2. A Solucao: Spec-Driven Development (SDD)

O SDD nao e uma ideia nova. E a adaptacao de boas praticas da engenharia de software classica para o mundo dos agentes de IA. O principio central:

> **Separe estritamente o planejamento da execucao. O agente deve receber apenas o contexto certo, na hora certa, para a tarefa certa.**

### O Pipeline SDD em 4 Fases

```mermaid
flowchart LR
    A[Wizard\nResearch] --> B[PRD\nProduto]
    B --> C[SPEC\nTecnico]
    C --> D[Speculi\nAuditoria]
    D --> E[Sprints\nExecucao]
    E --> F[Review\nValidacao]

    style A fill:#1e3a5f,color:#fff
    style B fill:#1e3a5f,color:#fff
    style C fill:#1e4a3f,color:#fff
    style D fill:#4a1e3f,color:#fff
    style E fill:#3f4a1e,color:#fff
    style F fill:#3f1e1e,color:#fff
```

---

### Fase 1 - Wizard / Research

Nao escreva codigo aqui. Use esta fase para explorar, debater, questionar. A IA e usada como interlocutora para levantar duvidas de arquitetura, validar hipoteses e gerar anotacoes sobre decisoes de design.

**Regra de ouro:** nenhuma linha de codigo nasce nessa fase.

---

### Fase 2 - PRD (Product Requirements Document)

Documento de negocio e produto. Define:

| Campo | Descricao |
|---|---|
| Problema | O que precisa ser resolvido |
| Metas | O que define sucesso |
| Escopo IN | O que esta dentro |
| Escopo OUT | O que esta fora (importante!) |
| User Stories | Como o usuario vai interagir |

---

### Fase 3 - SPEC (Especificacao Tecnica)

A SPEC traduz o PRD em instrucoes tecnicas precisas. E um documento denso (pode chegar a 1.000 linhas) com:

```mermaid
mindmap
  root((SPEC))
    Stack
      Tecnologias
      Dependencias
      Versoes
    Database
      Tabelas
      Campos e tipos
      Constraints
      Indices
      Seeds
    Backend
      Rotas e metodos
      Payloads
      Respostas de erro
    Frontend
      Arvore de componentes
      Mapa de paginas
      Design System
    Seguranca
      Autenticacao
      Variaves de ambiente
      Checklist LGPD
```

---

### Fase 4 - Speculi (Auditor de Spec)

Antes da execucao, o Speculi (Spec Reviewer) audita a SPEC em 3 etapas:

```mermaid
flowchart TD
    A[SPEC pronta] --> B[Etapa 1: Varredura Tecnica]
    B --> B1[Encontra gaps: falta de timeout, sem indice, sem tratamento de erro]
    B1 --> B2[Corrige automaticamente, sem perguntar ao humano]
    B2 --> C[Etapa 2: Entrevista de Negocio]
    C --> C1{Ha ambiguidades de negocio?}
    C1 -->|Sim| C2[Pergunta ao Andre com opcoes de multipla escolha]
    C2 --> C3[Documenta a decisao na spec]
    C1 -->|Nao| D[Etapa 3: Auto-Fill de Salvaguardas]
    C3 --> D
    D --> D1[Frontend: loading states, empty states, suporte offline]
    D --> D2[Banco: rollback, constraint handling]
    D --> D3[Auth: refresh token, brute force block, expiracao]
    D1 & D2 & D3 --> E[Relatorio Speculi]
    E --> F[SPEC auditada: pronta para execucao]
```

**Regra critica do Speculi:** ele e proibido de inventar regras de negocio. Se houver ambiguidade, ele para e pergunta.

---

### Fase 5 - Sprints (Fatiamento de Contexto)

O segredo do SDD para agentes: cada sprint contem apenas o contexto necessario para aquela tarefa especifica.

Anatomy de uma Sprint:

```
Sprint 3 - Feature: Endpoint POST /usuarios

spec_lines: 226-305        <- so leia essas linhas da SPEC
target_files:              <- so mexa nesses arquivos
  - src/routes/users.py (linhas 1-80)
  - src/models/user.py (linhas 1-50)
hints:                     <- nao reinvente a roda
  - use o padrao de validacao do auth.py
  - retorne ErroResponse para falhas
acceptance_criteria:       <- definition of done
  - POST /usuarios retorna 201 com {id, nome, email}
  - POST com email duplicado retorna 409
smoke_commands:            <- teste antes de entregar
  - pytest tests/test_users.py -v
  - mypy src/routes/users.py
```

O modelo le apenas as linhas 226-305 da SPEC. O contexto e minimo. A alucinacao e praticamente impossivel.

---

## 3. O Estudo de Caso: Qwen 27B vs Claude 3.5 Opus

Este e o dado mais impactante de toda a metodologia.

![Comparacao: Qwen 27B com SDD vs Claude Opus sem SDD](imagens/comparacao-modelos.png)

### O Experimento

Breno Vieira conduziu um teste controlado: desenvolver o mesmo sistema complexo (backend FastAPI, frontend React, integracao com LangGraph e agentes) em dois fluxos paralelos:

| Fator | Fluxo A (Frontier) | Fluxo B (SDD) |
|---|---|---|
| Modelo | Claude 3.5 Opus | Qwen 3.6 27B (local) |
| Metodologia | Vibe Coding | Spec-Driven Development |
| Custo de API | Alto (pago) | R$ 0,00 |
| Hardware | Cloud | Maquina local |

### Os Resultados

```mermaid
barchart-beta
```

| Metrica | Vibe Coding (Frontier) | SDD (Local) |
|---|---|---|
| Funcionalidade entregue | Equivalente | Equivalente |
| Custo de API | Alto | R$ 0,00 |
| Bugs na primeira entrega | Alto | Baixo |
| Tempo em code review | 60% do tempo | 20% do tempo |
| Tempo em desenvolvimento | 40% do tempo | 80% do tempo |

**O Qwen 27B local com SDD entregou resultado identico ao Claude Opus pago, a custo zero.**

Isso prova que:

> O gargalo de qualidade de codigo com IA nao e o modelo. E o processo.

---

## 4. Por que Funciona: A Logica por Tras do SDD

```mermaid
flowchart LR
    A[Contexto Gigante\nModelo Confuso] -->|SDD| B[Contexto Minimo\nModelo Focado]
    B --> C[Menos Alucinacao]
    C --> D[Menos Bugs]
    D --> E[Menos Code Review]
    E --> F[Mais Velocidade Real]
```

Quando o agente recebe apenas as linhas relevantes da SPEC, os arquivos exatos que pode tocar e os criterios de aceite objetivos, ele nao tem espaco para errar. O SDD fecha o espaco de decisoes ruins.

Alinha-se tambem com o que o SlopCodeBench descobriu (Orlanski et al., 2026): agentes degradam o codigo ao longo do tempo porque cada decisao local parece razoavel, mas o acumulo e ruim. O SDD interrompe esse ciclo ao redefinir o contexto a cada sprint.

---

## 5. Aplicacao Pratica: Como Comecar Hoje

**Passo 1:** Para o proximo projeto, antes de abrir o editor, escreva o PRD. Pode ser simples. O que importa e separar o planejamento da execucao.

**Passo 2:** Transforme o PRD numa SPEC tecnica. Use sua IA como rascunhadora, mas voce valida cada campo.

**Passo 3:** Rode o Speculi (ou faca voce mesmo o papel do auditor critico). Procure ambiguidades e gaps antes de codar.

**Passo 4:** Quebre a SPEC em sprints pequenas. Cada sprint deve ter no maximo 1-2 horas de trabalho para um agente.

**Passo 5:** O agente executa uma sprint. Voce revisa. So entao parte para a proxima.

---

## 6. Conexao com Multi-Agent Workflows

O SDD se potencializa ainda mais quando combinado com times de agentes especializados:

```mermaid
sequenceDiagram
    participant A as Andre (Humano)
    participant T as Tania (Arquiteta/Revisora)
    participant H as Heitor (Desenvolvedor)
    participant S as Speculi (Auditor)

    A->>T: Define Proposal e valida PRD
    T->>T: Escreve SPEC tecnica
    T->>S: Envia SPEC para auditoria
    S->>A: Entrevista sobre ambiguidades
    A->>S: Decide comportamentos
    S->>T: SPEC auditada e aprovada
    T->>H: Envia Sprint com contexto minimo
    H->>H: Implementa a feature
    H->>T: Entrega para revisao
    T->>A: Relatorio de revisao
    A->>T: Aprova ou solicita ajuste
```

Cada agente tem um papel claro e um contexto especifico. Ninguem le mais do que precisa. A degradacao e detectada antes de acumular.

---

## 7. Conclusao

O SDD nao e sobre desconfiar da IA. E sobre usar a IA de forma profissional.

Os numeros sao claros:
- Modelos pequenos com processo estruturado superam modelos gigantes com prompts vagos
- O gargalo de qualidade e o processo, nao o modelo
- Contexto minimo = menos alucinacao = menos bugs = mais velocidade real

O vibe coding e o entusiasmo que todos sentimos no inicio. O SDD e a maturidade que torna esse entusiasmo sustentavel.

---

## Referencias

- Artigo original SDD: **Spec-Driven Development: O Guia Definitivo** (Breno Vieira, 2026)
- Orlanski, G. et al. **SlopCodeBench: Benchmarking How Coding Agents Degrade Over Long-Horizon Iterative Tasks.** arXiv:2603.24755, 2026. http://arxiv.org/abs/2603.24755
- Experimento comparativo: Qwen 27B vs Claude 3.5 Opus (Breno Vieira, 2026)

---

*Tania - Assistente de IA de Andre Luiz Martins - 2026*
