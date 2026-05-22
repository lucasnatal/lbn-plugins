# Claude Code Plugins — Manual de Referência

> **Premissa deste documento.** A documentação oficial da Anthropic descreve o sistema *como ele foi especificado*. O plugin **superpowers** (v5.1.0, Jesse Vincent / Prime Radiant) é o estudo de caso mais maduro do ecossistema atual — testado adversarialmente com `evals/` próprio, mantido em produção, e com decisões de design empiricamente justificadas.
>
> Este documento toma a arquitetura do superpowers como **padrão de fato** e usa a documentação oficial como referência secundária. Onde os dois divergem, partimos da hipótese que o superpowers tem razão e tentamos *inferir o motivo* da escolha. Onde a doc oficial é silente, descrevemos o padrão observado em plugins reais.
>
> **Convenções:**
> - 🟠 **Padrão observado** em plugins maduros (= superpowers como referência)
> - 🔵 **Especificação oficial** da Anthropic (`code.claude.com/docs`, `platform.claude.com/docs`)
> - 🟢 **Convenção de engenharia** anterior a LLMs
> - 🟣 **Pesquisa acadêmica** citada
> - 🔴 **Divergência** entre padrão observado e especificação oficial

---

# Parte I — Anatomia de um plugin que funciona

Esqueleto real do `superpowers/`:

```
superpowers/
├── .claude-plugin/        # Manifesto Claude Code
│   ├── plugin.json
│   └── marketplace.json
├── .codex-plugin/         # Manifesto OpenAI Codex (com interface visual)
│   └── plugin.json
├── .cursor-plugin/        # Manifesto Cursor
│   └── plugin.json
├── .opencode/             # Plugin JS para OpenCode
│   ├── INSTALL.md
│   └── plugins/superpowers.js
├── gemini-extension.json  # Manifesto Gemini CLI
├── package.json           # Para instalação OpenCode/npm
│
├── hooks/
│   ├── hooks.json         # Mapeamento eventos → comandos (Claude Code)
│   ├── hooks-cursor.json  # Mapeamento alternativo (Cursor)
│   ├── run-hook.cmd       # Wrapper polyglot Windows/Unix
│   └── session-start      # Bash que injeta o bootstrap
│
├── skills/                # 14 skills, todas com SKILL.md
│   └── <14 skills>
│
├── docs/                  # Specs, plans, e documentação interna
│   ├── plans/
│   └── superpowers/
│       ├── plans/
│       └── specs/
│
├── evals/                 # Harness próprio de avaliação adversarial
├── tests/                 # Testes técnicos da infra do plugin
├── scripts/               # Build/release scripts
│
├── CLAUDE.md → AGENTS.md  # Diretrizes (single source via symlink)
├── GEMINI.md              # Override para Gemini CLI
├── README.md
├── RELEASE-NOTES.md
├── CODE_OF_CONDUCT.md
├── LICENSE
└── .version-bump.json
```

## O que NÃO está aqui (e por quê)

🔴 **Não há `agents/`.** A documentação oficial 🔵 e o `example-plugin` da Anthropic apresentam `agents/` como uma das seções padrão. O superpowers não tem. Inferência do motivo: ver Parte III.

🔴 **Não há `commands/`.** A doc oficial 🔵 ainda lista `commands/<nome>.md` como formato de slash command. O superpowers usa exclusivamente `skills/<nome>/SKILL.md` para slash commands também — formato funcionalmente equivalente, mais flexível, e que evita o split de duas pastas para o mesmo conceito.

🔴 **Não há `.mcp.json`.** O superpowers é zero-dependency por design (regra explícita no `CLAUDE.md`). Plugins que precisam de MCP ficam separados.

A *ausência* destes três é tão informativa quanto a presença dos demais.

## O que está presente além do óbvio

🟠 **Múltiplos manifestos por harness** — não é "também roda no Cursor", é cinco manifestos paralelos cada um com particularidades. Multi-harness é default, não afterthought.

🟠 **`evals/` separado de `tests/`** — `tests/` testa o código da infraestrutura (hooks, scripts). `evals/` testa **comportamento de skills sob pressão**: um harness próprio que dispatcha cenários adversariais a subagentes para verificar compliance. A doc oficial 🔵 admite literalmente *"We do not currently provide a built-in way to run these evaluations"*, então o superpowers construiu o seu.

🟠 **`docs/superpowers/specs/` e `docs/superpowers/plans/`** — não documentação de uso, mas o histórico do próprio desenvolvimento do plugin seguindo seu próprio pipeline (brainstorm → spec → plan → implementation). Comer a própria comida.

🟠 **`CLAUDE.md → AGENTS.md` symlink** — um arquivo, dois nomes. Garante que harnesses que procuram `AGENTS.md` (Codex) e harnesses que procuram `CLAUDE.md` (Claude Code) leiam o mesmo conteúdo. `GEMINI.md` é separado porque o Gemini CLI tem mapeamento de tools próprio.

🟠 **`.version-bump.json`** — versionamento centralizado entre `package.json`, `plugin.json` (×3), `marketplace.json`, `gemini-extension.json`. Plugin maduro em múltiplos harnesses precisa de single source of version.

---

# Parte II — Skills: a unidade real de comportamento

## 1. SKILL.md (estrutura padrão)

```markdown
---
name: nome-com-hifens
description: Use when [condições específicas de trigger]
---

# Nome da skill

[corpo em markdown]
```

🔵 Spec oficial: dois campos obrigatórios (`name` ≤ 64 chars, `description` ≤ 1024 chars). Total de frontmatter ≤ 1024 chars.

### A regra de description (e a divergência mais importante)

🔴 **A descoberta empírica mais bem documentada do superpowers** diverge frontalmente da doc oficial:

🔵 Documentação Anthropic (`anthropic-best-practices.md` linha 199):
> *"The description field enables Skill discovery and should include both what the Skill does and when to use it."*

🟠 Superpowers (`writing-skills/SKILL.md` linhas 150-172):
> *"Description = When to Use, NOT What the Skill Does. NEVER summarize the skill's process or workflow."*

**O motivo registrado** (com caso documentado): quando a description resumia o workflow, o modelo seguia a description como *atalho* e nunca lia o SKILL.md completo. Mudaram para "Use when..." apenas e o comportamento foi corrigido.

**Implicação prática:** *trigger conditions only, third person, sem explicar processo*.

```yaml
# ❌ description: "Use when executing plans - dispatches subagent per task with code review"
# ✅ description: "Use when executing implementation plans with independent tasks in the current session"
```

### Outros campos (slash commands)

```yaml
---
name: meu-comando
description: Use when [...]
argument-hint: <obrigatorio> [opcional]
allowed-tools: [Read, Glob, Grep, Bash]
model: sonnet
---
```

🔵 Campos `argument-hint`, `allowed-tools`, `model`, `version` documentados oficialmente. 🟠 No padrão observado, `allowed-tools` é raramente usado — quando você quer pre-aprovar tools, isso normalmente vai em `.claude/settings.json`.

## 2. O Bootstrap — a peça arquitetural central

🟠 Esta é provavelmente a inovação mais importante do superpowers. Não está documentada na doc oficial 🔵 como padrão recomendado — é solução para um problema que a Anthropic não trata.

**Problema:** 🔵 No modelo oficial, skills são *lazy*. Apenas frontmatter (`name` + `description`) é injetado no system prompt na inicialização. O corpo só é lido quando o modelo decide invocar a skill. Mas se o modelo não invoca, a skill nunca atua. Skills opt-in podem ser ignoradas.

**Solução do superpowers** (`hooks/session-start`):

```bash
# A cada SessionStart (incluindo /clear, /compact):
using_superpowers_content=$(cat "${PLUGIN_ROOT}/skills/using-superpowers/SKILL.md")

session_context="<EXTREMELY_IMPORTANT>
You have superpowers.
**Below is the full content of your 'superpowers:using-superpowers' skill...**
${using_superpowers_content}
</EXTREMELY_IMPORTANT>"

# Output JSON com additionalContext via stdout:
printf '{"hookSpecificOutput":{"hookEventName":"SessionStart","additionalContext":"%s"}}' "$session_context"
```

**O que isso faz:** uma skill (`using-superpowers`) sai do modelo lazy e vira **eager** — injetada inteira no system prompt a cada sessão, envolta em `<EXTREMELY_IMPORTANT>`.

**Por que essa skill especificamente:** é o *meta-skill* que ensina ao modelo o protocolo de invocar outras skills. Sem ela, todo o resto pode ser ignorado.

🟠 **Custo:** ~600 tokens permanentemente no system prompt de toda sessão.
🟠 **Benefício:** ativação garantida do pipeline.

**Por que a doc oficial não recomenda isso:** especulação razoável — porque viola o princípio da economia de contexto que ela mesma estabelece. A Anthropic descreve o sistema *como deveria ser* (lazy, econômico). O superpowers descobriu que *como é* não basta, e fez um trade-off explícito de tokens por confiabilidade.

## 3. Cross-skill protocol — como skills se conectam

🟠 Padrão consistente em todas as 14 skills do superpowers. A doc oficial 🔵 não fala em "skills se referenciando entre si" — trata cada skill como ilha. O superpowers padronizou três markers:

### `**REQUIRED BACKGROUND:**`

Pré-requisito conceitual. *"Você precisa entender X antes de aplicar este skill."*

```markdown
**REQUIRED BACKGROUND:** You MUST understand superpowers:test-driven-development
before using this skill. That skill defines the fundamental RED-GREEN-REFACTOR
cycle. This skill adapts TDD to documentation.
```

Não é "execute aquela skill agora". É "leia para ter o fundamento".

### `**REQUIRED SUB-SKILL:**`

Próximo passo obrigatório do pipeline. É o que conecta a cadeia.

```markdown
**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review
```

A skill termina apontando para a próxima. Pipeline emerge dessa concatenação.

### `**Related skills:**`

Cross-references informativas, sem obrigação. Para casos não óbvios.

### Sintaxe canônica: `superpowers:<nome>`

🟠 Sempre com prefixo do plugin. Skills com mesmo nome podem coexistir entre plugins; o namespace evita ambiguidade.

🟠 Regra explícita contra `@`:

> *"Why no @ links: `@` syntax force-loads files immediately, consuming 200k+ context before you need them."* (writing-skills/SKILL.md linhas 286-288)

🔵 A doc oficial não desencoraja `@` ativamente, só descreve a sintaxe. 🟠 O padrão observado: `@` é exceção, prosa textual ("see foo.md") é regra.

🔴 **Inconsistência interna do superpowers:** o próprio `writing-skills/SKILL.md` usa `@` em duas referências (`@graphviz-conventions.dot`, `@testing-skills-with-subagents.md`). Sintoma de tensão entre regra e pragmatismo.

## 4. Pipeline canônico (a cadeia que vira disciplina)

🟠 As 14 skills se organizam em um pipeline que o `using-superpowers` (bootstrap) força:

```
                         using-superpowers
                                │ (eager, via SessionStart)
                                ▼
        ┌───────────────────────┴───────────────────────┐
        ▼                                               ▼
   brainstorming                                  systematic-debugging
   (HARD-GATE: design                             (4 fases até root cause
    antes de código)                               antes de qualquer fix)
        │                                               │
        ▼                                               ▼
   using-git-worktrees ◄──── isolamento ────► verification-before-completion
        │                                               (run command before
        ▼                                                claiming success)
   writing-plans
   (bite-sized 2-5 min)
        │
        ├──► subagent-driven-development ──┐
        │            │                     │  (fresh subagent
        │            ▼                     │   per task +
        │     test-driven-development ◄────┤   two-stage review)
        │     (RED-GREEN-REFACTOR)         │
        │            ▲                     │
        └──► executing-plans ──────────────┘
                     │
                     ▼
            requesting-code-review ──► receiving-code-review
                     │
                     ▼
        finishing-a-development-branch
        (4 opções estruturadas)

       (transversais)
       dispatching-parallel-agents (qualquer ponto com N tasks independentes)
       writing-skills (fora do pipeline; ao criar/editar skills)
```

🟠 Esse pipeline é prescritivo. Cada skill termina com `**REQUIRED SUB-SKILL:**` apontando a próxima.

🔴 A doc oficial 🔵 não trata de pipelines. Trata cada skill como independente. O motivo da divergência (inferência): sem encadeamento explícito, o modelo pula etapas — vai direto para implementar sem brainstorm, escreve código sem teste, declara pronto sem verificar. O pipeline força gates que o modelo sozinho ignoraria.

---

# Parte III — Anatomia da pasta de uma skill

🟠 Em plugins maduros, uma skill é mais que `SKILL.md`. Catálogo dos seis tipos de arquivo observados nas 14 skills do superpowers:

## 3.1 `SKILL.md` (sempre)

🔵 Único obrigatório. Frontmatter + corpo. Limite empírico recomendado: ≤ 500 linhas no corpo. Se ultrapassar, dividir em arquivos auxiliares.

## 3.2 Reference docs lazy (sem `@`)

Documentos `.md` referenciados com prosa ("see foo.md"). Modelo decide se lê.

| Arquivo | Skill | Função |
|---|---|---|
| `anthropic-best-practices.md` | writing-skills | 🔵 Doc oficial pinada como referência |
| `persuasion-principles.md` | writing-skills | 🟣 Cialdini, Meincke et al. |
| `testing-anti-patterns.md` | test-driven-development | 🟠 Anti-patterns documentados |
| `condition-based-waiting.md` | systematic-debugging | 🟠 Técnica auxiliar |
| `defense-in-depth.md` | systematic-debugging | 🟠 Técnica auxiliar |
| `root-cause-tracing.md` | systematic-debugging | 🟠 Técnica auxiliar |
| `visual-companion.md` | brainstorming | 🟠 Modo opcional |

🟠 **Padrão:** referência opcional, extensiva, que importa em alguns casos.

## 3.3 Reference docs force-load (`@arquivo`)

Carregamento imediato ao chegar na linha. Custoso. Usar com moderação.

| Arquivo | Skill | Por que force-load |
|---|---|---|
| `testing-skills-with-subagents.md` | writing-skills | Toda execução da skill precisa da metodologia completa |
| `graphviz-conventions.dot` | writing-skills | Convenções aplicadas em diagramas gerados |

🟠 **Padrão:** material crítico que toda invocação precisa, e cuja ausência causa output errado.

## 3.4 Prompt templates (a inovação central)

🟠 Arquivos `.md` com placeholders que servem como **contrato** para subagentes. Não são lidos pelo modelo automaticamente — são *recipe files* que o controlador da skill preenche e dispatch via Task tool.

| Arquivo | Skill que usa | Para que |
|---|---|---|
| `code-reviewer.md` | requesting-code-review | Despachar reviewer geral |
| `implementer-prompt.md` | subagent-driven-development | Despachar implementador |
| `spec-reviewer-prompt.md` | subagent-driven-development | Reviewer de spec compliance |
| `code-quality-reviewer-prompt.md` | subagent-driven-development | Reviewer de qualidade |
| `spec-document-reviewer-prompt.md` | brainstorming | Reviewer do spec gerado |
| `plan-document-reviewer-prompt.md` | writing-plans | Reviewer do plano gerado |

**Anatomia** (vide `requesting-code-review/code-reviewer.md`):

```markdown
# Code Reviewer Prompt Template

```
Task tool (general-purpose):
  description: "Review code changes"
  prompt: |
    You are a Senior Code Reviewer ...

    ## What Was Implemented
    {DESCRIPTION}

    ## Requirements / Plan
    {PLAN_OR_REQUIREMENTS}

    ## Git Range to Review
    Base: {BASE_SHA}
    Head: {HEAD_SHA}

    [contrato: o que checar, formato de output, regras]
```

**Placeholders:**
- {DESCRIPTION} — brief summary of what was built
- {PLAN_OR_REQUIREMENTS} — what it should do
- {BASE_SHA} — starting commit
- {HEAD_SHA} — ending commit
```

🟠 **Características importantes do design observado:**

1. **Status enums explícitos** — `implementer-prompt.md` força retorno como um de quatro: `DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT`. Permite roteamento programático pelo controlador
2. **Self-review embutido** — template instrui o subagente a auto-revisar antes de reportar
3. **Escape hatches explícitos** — *"It is always OK to stop and say 'this is too hard for me'"* — reduz pressão para resultado ruim
4. **Output format estruturado** — reviewers retornam em formato fixo (Strengths / Issues por severidade / Recommendations / Assessment)
5. **Curadoria explícita do contexto** — placeholders forçam o controlador a coletar informação específica em vez de despejar histórico de sessão

## 3.5 Examples (código para adaptar)

| Arquivo | Skill | Função |
|---|---|---|
| `condition-based-waiting-example.ts` | systematic-debugging | TS funcional para copiar/adaptar |
| `examples/CLAUDE_MD_TESTING.md` | writing-skills | Worked example de pressure testing |

🟢 Convenção de engenharia: um exemplo excelente vale mais que cinco mediocres em linguagens diferentes.

## 3.6 Scripts (executados, não lidos)

🔵 Doc oficial: scripts são executados via Bash; apenas o stdout consome tokens.

| Arquivo | Skill | Para que |
|---|---|---|
| `find-polluter.sh` | systematic-debugging | Identificar teste que polui estado |
| `render-graphs.js` | writing-skills | Renderizar diagramas Graphviz |
| `scripts/server.cjs` | brainstorming | Servidor local do visual companion |
| `scripts/start-server.sh`, `stop-server.sh` | brainstorming | Bootstrap do servidor |

🟠 **Padrão:** operações determinísticas, repetitivas, ou com I/O específico. Mais barato que regenerar código toda vez.

## 3.7 Platform adaptation

🟠 Mapeamento de tools entre harnesses (resolve heterogeneidade do ecossistema).

| Arquivo | Skill | Função |
|---|---|---|
| `references/codex-tools.md` | using-superpowers | Mapeia tools Claude Code → Codex |
| `references/copilot-tools.md` | using-superpowers | Mapeia para Copilot CLI |
| `references/gemini-tools.md` | using-superpowers | Mapeia para Gemini CLI |

Skills usam nomes Claude Code (`TodoWrite`, `Task`, `Skill`) por convenção. Outros harnesses precisam tradução.

## 3.8 Test materials (rastros do desenvolvimento, opcional)

| Arquivo | Skill | Função |
|---|---|---|
| `test-academic.md` | systematic-debugging | Cenário acadêmico baseline |
| `test-pressure-{1,2,3}.md` | systematic-debugging | Pressure scenarios |
| `CREATION-LOG.md` | systematic-debugging | Log do RED-GREEN-REFACTOR |

🟠 Não consumidos em runtime. Servem como prova de proveniência ("a skill foi testada de fato") e referência para futuras edições.

## 3.9 Esqueleto consolidado

```
skills/<nome>/
├── SKILL.md                          [obrigatório]
│
├── <ref>.md                          [lazy — modelo decide se lê]
├── (com @<ref>.md no SKILL.md)       [force-load]
│
├── <papel>-prompt.md                 [🟠 template para subagente]
├── <example>.<ext>                   [código para adaptar]
├── <script>.{sh,js,cjs}              [executável — output consome tokens]
├── references/<harness>-tools.md     [🟠 platform adaptation]
└── test-*.md, CREATION-LOG.md        [histórico opcional]
```

---

# Parte IV — Delegação a subagentes (e a ausência de `agents/`)

🔴 **A divergência mais reveladora do superpowers**: ele não usa `agents/`. Vou explicar as duas opções, o que o superpowers escolheu, e por quê.

## 4.1 Approach A — Agents registrados (`agents/<nome>.md`)

🔵 Padrão da doc oficial. Existe nos plugins `feature-dev`, `code-review`, `mcp-server-dev` da Anthropic.

```markdown
---
name: code-architect
description: Designs feature architectures by analyzing existing codebase patterns
tools: Glob, Grep, LS, Read, NotebookRead, WebFetch, TodoWrite, WebSearch
model: sonnet
color: green
---

You are a senior software architect ...
```

O harness registra. Modelo invoca via Task tool com `subagent_type: "code-architect"`. Aparece no `subagent_type` autodescoberto.

**Características:**
- 🔵 Discoverability automática — modelo "vê" os agentes
- 🔵 Tools/model lock-in via frontmatter
- 🔵 Reuso fácil entre prompts (mesmo agente para vários usos)
- 🔵 Versionado com o plugin

## 4.2 Approach B — Prompt templates + agent genérico

🟠 Padrão exclusivo do superpowers. Templates `.md` dentro da pasta da skill com placeholders. Skill instrui o controlador a preencher e despachar via `subagent_type: "general-purpose"`.

```
Task(
  subagent_type: "general-purpose",
  description: "Implement Task 2",
  prompt: <conteúdo de implementer-prompt.md com placeholders preenchidos>
)
```

## 4.3 Por que o superpowers abandonou Approach A

🟠 Inferência baseada em filosofia documentada e comportamento observado:

1. **Curadoria explícita > implícita.** O superpowers tem regra clara (`subagent-driven-development/SKILL.md` linhas 10-12): *"They should never inherit your session's context or history — you construct exactly what they need."* Templates com placeholders **forçam** essa curadoria. Agents nomeados deixam o controlador improvisar o `prompt` a cada uso, e improviso degrada com a fadiga da sessão

2. **Especialização por uso, não por papel.** O `code-reviewer.md` em `requesting-code-review/` é diferente do `code-quality-reviewer-prompt.md` em `subagent-driven-development/`. Mesmo "papel" (reviewer), contextos distintos, contratos diferentes. Approach A obriga um único agente por papel; Approach B permite nuance

3. **Status enums no contrato.** Templates definem o protocolo de retorno (`DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT`) que o controlador despacha programaticamente. Em Approach A, o status é prosa livre — controlador precisa parsear

4. **Independência de harness.** `agents/` precisa ser registrado pelo harness para aparecer em `subagent_type`. Implementação varia entre Claude Code, Cursor, Codex, OpenCode, Gemini. `general-purpose` + prompt texto funciona em qualquer harness com Task tool. Multi-harness sério prefere o que tem cobertura universal

5. **Sem fantasmas no harness.** Plugins instalados via Approach A poluem a lista global de `subagent_type`. Approach B mantém a lógica interna ao plugin

6. **Versionamento acoplado à skill.** O template é versionado com a skill que o usa. Mudou o template, mudou a skill. Approach A separa as duas dimensões e abre janela para drift

7. **Self-review embutido.** Templates incluem instruções de self-review estruturadas que o agente deve executar antes de reportar. Em Approach A, isso fica disperso na descrição do agente

8. **Escape hatches explícitos.** Cada template tem permissão clara para escalar (`BLOCKED`, `NEEDS_CONTEXT`). Reduz tendência a forçar resultado ruim. Em Approach A, está implícito

## 4.4 Quando cada um faz sentido

| Cenário | Approach |
|---|---|
| Sub-agente verdadeiramente genérico, reutilizável em N contextos | A — registrar uma vez, usar em todo lugar |
| Workflow especializado com contexto curado por skill | B — template parametrizável |
| Multi-harness é requisito | B (mais portável) |
| Você precisa que o modelo descubra o agente sozinho | A |
| Você precisa de status enums no retorno | B |
| Plugin é zero-dependency e mantido por uma pessoa | B (menos superficie) |
| Plugin tem ecossistema de uso variado e múltiplos contribuidores | A pode fazer sentido |

## 4.5 Padrão híbrido

🟢 Nada impede combinar. Plugins maduros tendem a:
- `agents/` para sub-agentes verdadeiramente reutilizáveis (debugger geral, doc-writer)
- `skills/<nome>/<papel>-prompt.md` para workflows especializados que precisam contexto curado

O superpowers escolheu apenas o segundo. O `feature-dev` da Anthropic, apenas o primeiro. Ambos funcionam — mas o superpowers tem evals adversariais provando que o seu funciona.

---

# Parte V — Linguagem de bulletproofing

🟠 O superpowers usa linguagem deliberadamente coercitiva. Não é estilo — é técnica testada empiricamente. Base científica em 🟣 Meincke et al. (2025), University of Pennsylvania, com Cialdini como co-autor: testaram 7 princípios de persuasão em **N=28.000 conversas com LLMs**, compliance subiu de **33% para 72%** com técnicas combinadas.

🔴 A doc oficial 🔵 recomenda tom "professional" e mensurado. O superpowers escolheu autoridade explícita. Inferência: a Anthropic escreve para uma audiência ampla onde o tom importa; o superpowers otimiza para um único objetivo (compliance do modelo).

## 5.1 Os princípios aplicados

| Princípio (Cialdini) | Como aparece | Exemplo |
|---|---|---|
| **Authority** | Imperativo absoluto | "YOU MUST", "ABSOLUTELY MUST", "Never" |
| **Commitment** | Anúncio público | "Announce at start: 'I'm using the X skill'" |
| **Scarcity** | Urgência temporal | `<EXTREMELY_IMPORTANT>`, "BEFORE any response" |
| **Social proof** | Norma universal | "Every time", "X without Y = failure" |
| **Unity** | Linguagem coletiva | "your human partner" (não "the user") |
| **Reciprocity** | Evitado | (raramente útil em skills) |
| **Liking** | Proibido | Causa sycophancy, conflita com feedback honesto |

## 5.2 Padrões idiomáticos

🟠 Toda skill disciplinar tem essa estrutura. Não é ornamento — é defesa contra rationalization:

### Iron Law

```markdown
## The Iron Law

NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST

Write code before the test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete
```

🟠 Padrão: regra fundacional + lista explícita de loopholes proibidos. Não basta dizer "delete it" — tem que negar especificamente *cada* forma de não deletar.

### Tabela de Rationalizations

```markdown
| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Already manually tested" | Ad-hoc ≠ systematic. No record, can't re-run. |
```

🟠 Padrão: capturar verbatim a racionalização do modelo (vinda de pressure testing) e adicionar counter específico. Cada linha é um loophole fechado.

### Red Flags

```markdown
## Red Flags - STOP and Start Over

- Code before test
- Test passes immediately
- "I already manually tested it"
- "It's about spirit not ritual"
- "This is different because..."

**All of these mean: Delete code. Start over with TDD.**
```

🟠 Padrão: lista de pensamentos que indicam que o modelo está prestes a violar. Auto-checagem.

### "Spirit vs Letter"

```markdown
**Violating the letter of the rules is violating the spirit of the rules.**
```

🟠 Princípio fundacional para fechar a brecha do "estou seguindo o espírito mas não a letra". Aparece em quase toda skill disciplinar.

## 5.3 "your human partner"

🟠 Vocabulário deliberado. CLAUDE.md de contribuição é explícito: *"'your human partner' is deliberate, not interchangeable with 'the user'"*. Não é colorido — é técnica de unity (Cialdini), enquadrando a relação como colaborativa em vez de hierárquica. Afeta como o modelo trata feedback, como pede esclarecimento, como reporta problemas.

🔴 A doc oficial 🔵 não usa nem recomenda essa terminologia.

---

# Parte VI — Multi-harness é default

🟠 Um plugin sério **nasce** multi-harness. Ele não "também roda no Cursor" — ele tem cinco manifestos paralelos.

| Harness | Diretório / arquivo | Particularidade |
|---|---|---|
| **Claude Code** | `.claude-plugin/plugin.json` | Padrão; instalação via `/plugin install` |
| **Cursor** | `.cursor-plugin/plugin.json` + `hooks-cursor.json` | Output JSON é `additional_context` (snake_case) |
| **Codex (CLI/App)** | `.codex-plugin/plugin.json` | Frontmatter rico (`displayName`, `brandColor`, ícones) |
| **OpenCode** | `.opencode/plugins/<name>.js` + `package.json` | Instalação via `git+https://...` em `opencode.json` |
| **Gemini CLI** | `gemini-extension.json` (root) | Activation explícita de skills |
| **Factory Droid, Copilot CLI** | reusam `.claude-plugin/` | Marketplace próprio |

🔵 A doc oficial trata Claude Code como o caso primário. 🟠 O ecossistema real exige paridade entre harnesses — usuários instalam o mesmo plugin onde estão.

## 6.1 Polyglot hook

🟠 Padrão observado em `hooks/run-hook.cmd`: arquivo único que serve como bash script no Unix e como batch no Windows. Truque sintático com `: << 'CMDBLOCK'`. Resolve fricção sem manter dois arquivos.

## 6.2 Output de hook por harness

🟠 Um hook serve múltiplos harnesses detectando variável de ambiente:

```bash
if [ -n "${CURSOR_PLUGIN_ROOT:-}" ]; then
  printf '{"additional_context": "%s"}\n' "$ctx"     # Cursor (snake_case)
elif [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -z "${COPILOT_CLI:-}" ]; then
  printf '{"hookSpecificOutput": {...}}\n' "$ctx"     # Claude Code (nested)
else
  printf '{"additionalContext": "%s"}\n' "$ctx"       # SDK standard (Copilot CLI)
fi
```

🔴 Cada harness consume formato JSON diferente. Não há especificação cross-harness. Plugin maduro detecta e adapta.

## 6.3 Tool vocabulary

🟠 Skills são escritas em vocabulário Claude Code (`TodoWrite`, `Task`, `Skill` tool). Outros harnesses recebem mapeamento via `references/<harness>-tools.md`. O `using-superpowers` SKILL.md aponta para o mapeamento conforme o harness ativo.

---

# Parte VII — Onde a doc oficial diverge do padrão observado (com motivos inferidos)

🔴 Catálogo das principais divergências entre o que a doc oficial 🔵 recomenda e o que plugins maduros 🟠 fazem.

## 7.1 Description: triggers vs. "what + when"

| | Posição |
|---|---|
| 🔵 Doc oficial | Description deve incluir "what the Skill does AND when to use it" |
| 🟠 Padrão observado | Description é só "Use when [triggers]" |
| 🔴 Motivo inferido | Empírico: descrição que resume workflow vira atalho que o modelo segue ao invés de ler o SKILL.md completo |

## 7.2 `agents/` directory

| | Posição |
|---|---|
| 🔵 Doc oficial | `agents/<nome>.md` é uma das seções padrão |
| 🟠 Padrão observado | Não existe — substituído por prompt templates dentro de skills |
| 🔴 Motivo inferido | Curadoria explícita > implícita; especialização por uso > por papel; portabilidade entre harnesses |

## 7.3 `commands/` directory

| | Posição |
|---|---|
| 🔵 Doc oficial | Listado mas marcado como "legacy" |
| 🟠 Padrão observado | Removido completamente; slash commands vão em `skills/<nome>/SKILL.md` |
| 🔴 Motivo inferido | Dois caminhos para o mesmo conceito é confusão; skills/ é mais flexível |

## 7.4 Bootstrap eager via SessionStart

| | Posição |
|---|---|
| 🔵 Doc oficial | Skills são lazy; só metadata na inicialização |
| 🟠 Padrão observado | Uma skill especial (`using-superpowers`) é injetada eager no system prompt via hook |
| 🔴 Motivo inferido | Skills lazy podem ser ignoradas; bootstrap garante ativação. Custo de tokens é trade-off explícito |

## 7.5 Linguagem coercitiva

| | Posição |
|---|---|
| 🔵 Doc oficial | Tom professional, "consider", "may", "should" |
| 🟠 Padrão observado | "YOU MUST", `<EXTREMELY_IMPORTANT>`, "ABSOLUTELY", "No exceptions" |
| 🔴 Motivo inferido | Compliance dispara de 33% para 72% com técnicas Cialdini (Meincke et al. 2025); doc oficial otimiza tom genérico, plugin otimiza compliance específica |

## 7.6 Pipeline obrigatório entre skills

| | Posição |
|---|---|
| 🔵 Doc oficial | Skills são independentes |
| 🟠 Padrão observado | Cadeia obrigatória via `**REQUIRED SUB-SKILL:**` |
| 🔴 Motivo inferido | Sem encadeamento explícito o modelo pula etapas; gates força rigor |

## 7.7 Pressure testing

| | Posição |
|---|---|
| 🔵 Doc oficial | "Build evaluations first" — admite *"We do not currently provide a built-in way to run these evaluations"* |
| 🟠 Padrão observado | `evals/` próprio com cenários adversariais; metodologia RED-GREEN-REFACTOR para skills |
| 🔴 Motivo inferido | Anthropic não construiu o tooling; o ecossistema construiu o seu |

## 7.8 `@arquivo` force-load

| | Posição |
|---|---|
| 🔵 Doc oficial | Sintaxe documentada neutramente |
| 🟠 Padrão observado | Desencorajado ativamente — *"burns 200k+ context"* |
| 🔴 Motivo inferido | Custo empírico de contexto preempetivo |

## 7.9 "your human partner"

| | Posição |
|---|---|
| 🔵 Doc oficial | "the user" |
| 🟠 Padrão observado | "your human partner" — explicitamente "not interchangeable with 'the user'" |
| 🔴 Motivo inferido | Unity (Cialdini) — enquadramento colaborativo afeta tom de feedback e disposição para discordar |

## 7.10 Status enums em retornos de subagente

| | Posição |
|---|---|
| 🔵 Doc oficial | Não trata |
| 🟠 Padrão observado | `DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT` em todos os templates |
| 🔴 Motivo inferido | Permite roteamento programático em vez de parsing de prosa |

## 7.11 Worktree-based isolation

| | Posição |
|---|---|
| 🔵 Doc oficial | Não trata |
| 🟠 Padrão observado | Skill própria (`using-git-worktrees`) com protocolo de 4 passos |
| 🔴 Motivo inferido | Multi-agent + parallel work + PR iteration tornam isolamento essencial |

## 7.12 Self-review embedded em prompts

| | Posição |
|---|---|
| 🔵 Doc oficial | Não trata |
| 🟠 Padrão observado | Templates incluem checklist de self-review obrigatório antes de reportar |
| 🔴 Motivo inferido | Reduz iterações; subagente pega seus próprios bugs antes do controlador |

## 7.13 Multi-harness como first-class

| | Posição |
|---|---|
| 🔵 Doc oficial | Foco em Claude Code |
| 🟠 Padrão observado | 5+ manifestos paralelos, polyglot hooks, tool vocabulary mapping |
| 🔴 Motivo inferido | Usuários instalam o mesmo plugin em vários harnesses; fragmentação não é opção |

## 7.14 Test materials kept in skill folders

| | Posição |
|---|---|
| 🔵 Doc oficial | Não trata |
| 🟠 Padrão observado | `test-pressure-*.md`, `CREATION-LOG.md` ficam na pasta da skill |
| 🔴 Motivo inferido | Proveniência ("foi testada de fato") + referência para edições futuras |

---

# Parte VIII — Como construir seu plugin (decisões práticas)

🟠 Decision tree adotando padrões observados:

```
Você quer adicionar capacidade ao Claude Code.
│
├── É instrução textual que muda comportamento do modelo?
│   ├── SIM → SKILL
│   │   ├── Modelo invoca por contexto?
│   │   │   └── skills/<nome>/SKILL.md
│   │   │       (description: "Use when [só triggers]")
│   │   │
│   │   └── Usuário invoca via /comando?
│   │       └── skills/<nome>/SKILL.md
│   │           (mesma estrutura + argument-hint, allowed-tools)
│   │           NÃO use commands/ — formato deprecado
│   │
│   └── NÃO → continue
│
├── É um sub-agente?
│   ├── Reutilizável em múltiplos contextos sem curadoria especial?
│   │   └── agents/<nome>.md (Approach A)
│   │
│   └── Workflow especializado precisa contexto curado?
│       └── skills/<skill>/<papel>-prompt.md (Approach B, padrão observado)
│
├── É shell command em ciclo de vida?
│   └── hooks/hooks.json + script
│       (use polyglot wrapper se Windows também)
│       (output JSON adaptado por harness)
│
├── É ferramenta externa (API, DB)?
│   └── .mcp.json
│       (mantenha em plugin separado se zero-dependency é objetivo)
│
└── Sua skill precisa de:
    ├── Bootstrap eager?                  → hooks/session-start injetando o SKILL.md
    ├── Pré-requisito conceitual?         → **REQUIRED BACKGROUND:**
    ├── Próximo passo do pipeline?        → **REQUIRED SUB-SKILL:**
    ├── Cross-reference informativa?      → **Related skills:**
    ├── Doc opcional/extensa?             → reference.md (lazy)
    ├── Doc crítica para toda execução?   → @reference.md (force-load — usar com parcimônia)
    ├── Despachar subagente curado?       → <papel>-prompt.md com placeholders
    ├── Código exemplo?                   → example.<ext>
    ├── Operação determinística?          → script.{sh,js,py}
    ├── Suporte multi-harness?            → references/<harness>-tools.md
    └── Histórico de criação?             → test-*.md, CREATION-LOG.md (opcional)
```

## Princípios de redação de skill

🟠 Padrões observados, ordenados por importância empírica:

1. **Description é só trigger condition.** Terceira pessoa, "Use when...", sem resumo de workflow
2. **Iron Law no início.** Estabeleça a regra inviolável antes de qualquer detalhe
3. **Spirit vs Letter cedo.** Princípio fundacional fechando a brecha "estou seguindo o espírito"
4. **Tabela de Rationalizations.** Cada loophole explícito com counter específico
5. **Red Flags list.** Pensamentos que sinalizam violação iminente
6. **Status enums se retorna a controller.** Roteamento programático
7. **Self-review embutido se for template.** Reduz iterações
8. **Escape hatches.** "It is always OK to stop and say 'this is too hard'"
9. **Anúncio obrigatório.** "I'm using the [X] skill" (commitment de Cialdini)
10. **`@` só para o que é crítico em toda execução.** Resto, prosa lazy

## Princípios de pipeline

🟠 Se sua skill faz parte de um workflow:

1. Termine com `**REQUIRED SUB-SKILL:**` apontando próximo passo
2. Comece com `**REQUIRED BACKGROUND:**` se há pré-requisito
3. Use namespace canônico: `<plugin>:<skill-name>`
4. Não use `@<plugin>/<skill>/SKILL.md` — força-load destrói contexto
5. Cite `**Related skills:**` para cross-references não obrigatórias

## Multi-harness

🟠 Se o plugin é sério:

1. Manifestos paralelos: `.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/`, `.opencode/`, `gemini-extension.json`
2. Hook polyglot (`run-hook.cmd` com bash/batch dual)
3. Output JSON detectando harness via env vars
4. Tool vocabulary mapping em `references/<harness>-tools.md`
5. Symlink `AGENTS.md → CLAUDE.md` para single source

---

# Parte IX — Mapa de proveniência

## Sistema de plugins (substrato)

| Conceito | Origem |
|---|---|
| Estrutura `.claude-plugin/`, `skills/`, `hooks/`, `.mcp.json` | 🔵 Anthropic |
| Frontmatter YAML com `name`, `description` | 🔵 Anthropic |
| Limites 64/1024 chars | 🔵 Anthropic |
| Pipeline lazy de carregamento | 🔵 Anthropic |
| Sintaxe `@arquivo` | 🔵 Anthropic |
| Eventos de hook | 🔵 Anthropic |
| `${CLAUDE_PLUGIN_ROOT}` | 🔵 Anthropic |
| Marketplace via `/plugin install` | 🔵 Anthropic |
| `agents/<nome>.md` como mecanismo | 🔵 Anthropic |
| `commands/` (legacy) | 🔵 Anthropic |
| **Description "what + when"** | 🔵 Anthropic |

## Padrões observados em plugins maduros

| Conceito | Origem |
|---|---|
| Bootstrap via SessionStart injetando skill eager | 🟠 |
| **Description "só when, nunca what"** | 🟠 (contradiz 🔵) |
| Pipeline obrigatório via `**REQUIRED SUB-SKILL:**` | 🟠 |
| Prompt templates como contrato de subagente | 🟠 |
| Status enums em retornos | 🟠 |
| Self-review embutido em templates | 🟠 |
| Iron Law / Red Flags / Rationalization tables | 🟠 |
| Linguagem coercitiva (`<EXTREMELY-IMPORTANT>`) | 🟠 (base 🟣) |
| "your human partner" terminology | 🟠 |
| Polyglot hook (bash + batch em um arquivo) | 🟠 |
| Output JSON adaptado por harness | 🟠 |
| Multi-harness manifests paralelos | 🟠 |
| Symlink `AGENTS.md → CLAUDE.md` | 🟠 |
| `evals/` adversarial separado de `tests/` | 🟠 |
| Pressure testing com 3+ pressões combinadas | 🟠 |
| Worktree-based isolation com submodule guard | 🟠 |
| Two-stage review (spec compliance + code quality) | 🟠 |
| `references/<harness>-tools.md` para vocabulary mapping | 🟠 |

## Substrato cultural

| Conceito | Origem |
|---|---|
| TDD, RED-GREEN-REFACTOR | 🟢 Kent Beck |
| YAGNI, DRY | 🟢 XP / Pragmatic Programmer |
| Root cause analysis | 🟢 Toyota Production System |
| Code review como prática | 🟢 Engenharia de software |
| Git worktrees | 🟢 Feature do git |
| Subagentes / multi-agent | 🟢 IA agêntica padrão |

## Pesquisa acadêmica

| Citação | Aparece em | O que diz |
|---|---|---|
| 🟣 Cialdini, R. B. (2021). *Influence* | `persuasion-principles.md` | 7 princípios |
| 🟣 Meincke et al. (2025). *Call Me A Jerk*. UPenn | `persuasion-principles.md` | N=28k em LLMs; compliance 33% → 72% |

---

# Parte X — Limitações deste documento

Honestidade explícita sobre o que você está lendo:

**1. Inferência onde não há documentação.** Vários "motivos" da Parte VII são deduções razoáveis baseadas em comportamento observado, filosofia documentada, e contexto. Onde o time do superpowers não publicou explicação direta, indiquei como inferência.

**2. Um plugin como referência ≠ universal.** O superpowers é o caso mais maduro que tenho acesso direto, mas é um único caso. Outros plugins maduros podem fazer escolhas diferentes por motivos igualmente válidos. Padrões aqui são "o que funciona neste caso bem-testado", não "o que funciona em todo lugar".

**3. Documentação oficial não é desprezível, é apenas insuficiente.** A doc da Anthropic 🔵 é necessária para a especificação técnica (formato, eventos, instalação). Não é suficiente para construir um plugin que efetivamente afeta comportamento. As duas se complementam.

**4. Meu pré-treino tem viés institucional.** Fui treinado em material que provavelmente inclui fortemente a doc oficial da Anthropic. Em pontos de conflito com observação empírica, tive que ativamente desfazer pressuposições. Onde isto aconteceu, sinalizei na Parte VII.

**5. Métricas reportadas pelo superpowers não são reproduzíveis.** Frases como "100% compliance under maximum pressure" ou "95% first-time fix rate vs 40%" não vêm com metodologia detalhada nem dataset compartilhado. Vale como narrativa direcional, não como dado replicável.

**6. O ecossistema está em movimento.** O próprio superpowers migrou commands/ → skills/ recentemente, e tem `evals/` como substituição em curso de testes bash legados. Padrões consolidados hoje podem mudar.

**7. Se você for construir um plugin, tem dois caminhos.**
- **Conservador:** seguir doc oficial 🔵, aceitar riscos conhecidos (skills lazy podem ser ignoradas, description com workflow cria atalho, etc.)
- **Empírico:** seguir padrões observados 🟠, pagar tokens do bootstrap, aceitar linguagem coercitiva, beneficiar de compliance maior

Este documento sugere o segundo. Mas a sugestão é informada por *uma referência*, e referências evoluem.

---

# Plugins instalados nesta máquina

| Plugin | Versão | Marketplace | Caminho |
|---|---|---|---|
| `superpowers` | 5.1.0 | `superpowers-dev` (obra/superpowers) | `~/.claude/plugins/cache/superpowers-dev/superpowers/5.1.0` |

Marketplaces registrados:
- `claude-plugins-official` — `~/.claude/plugins/marketplaces/claude-plugins-official/`
- `superpowers-dev` — `~/.claude/plugins/marketplaces/superpowers-dev/`

Repo de desenvolvimento próprio: `/Users/lucasnatal/Developer/Plugins/lbn-plugins/`

Cópia local para estudo: `/Users/lucasnatal/Developer/Plugins/superpowers/`
