# Design: Unificar skills da Nuvia (10 → 4)

**Data:** 2026-06-09
**Objetivo:** Consolidar as 10 skills atuais do plugin em 4 skills com prefixo `nuvia-`, sem perder cobertura de ativação nem os avisos de custo/segurança.

## Motivação

Hoje há variantes da mesma tarefa em skills separadas (busca BR vs global; gestão fragmentada em listas/contatos/enriquecer/salvar). O usuário quer skills mais unificadas: uma única que sabe fazer a tarefa inteira, em vez de uma por variante.

## Estrutura final (4 skills)

| Comando | Conteúdo | Origem (skills fundidas) |
|---------|----------|--------------------------|
| **`/nuvia`** | Fundamentos + operação na plataforma: Listas, Pessoas (contatos), enriquecimento e salvar resultados de busca. Dona das `references/` compartilhadas. | `nuvia-fundamentos` + `gerenciar-listas` + `gerenciar-contatos` + `enriquecer-contatos` + `salvar-na-nuvia` (5→1) |
| **`/nuvia-search`** | Buscar empresas (BR + global) e prospectar decisores. | `pesquisar-empresas-br` + `pesquisar-empresas-global` + `prospectar-decisores` (3→1) |
| **`/nuvia-deep`** | Investigação societária BR (sócios × identidade digital, via BigQuery). | `investigar-empresa-br` (renomeada) |
| **`/nuvia-conversas`** | Atendimento e campanhas (somente leitura). | `inspecionar-atendimento` (renomeada) |

Mudança real: **2 fusões** (`/nuvia` de 5→1, `/nuvia-search` de 3→1) + **2 renomeações** (`/nuvia-deep`, `/nuvia-conversas`).

## Decisões de arquitetura

### `nuvia-fundamentos` é absorvida por `/nuvia`
A base de conhecimento compartilhada (regras de custo, lookup-antes-de-buscar, paginação, terminologia, glossário) passa a viver dentro da skill `/nuvia`. Os 7 arquivos de `nuvia-fundamentos/references/` movem para `nuvia/references/`:
`ajuda-nuvia.md`, `campos-retornados.md`, `catalogos-br.md`, `catalogos-global.md`, `enriquecimento.md`, `mapeamentos-br.md`, `tools.md`.

As outras 3 skills continuam consumindo esses fundamentos via path explícito `nuvia/references/...` e instrução "leia a skill `nuvia` primeiro".

### Pasta vs nome da skill
O nome no frontmatter (`name:`) deve bater com a pasta. Skills:
- `plugin/skills/nuvia/` → `name: nuvia`
- `plugin/skills/nuvia-search/` → `name: nuvia-search`
- `plugin/skills/nuvia-deep/` → `name: nuvia-deep`
- `plugin/skills/nuvia-conversas/` → `name: nuvia-conversas`

## Os 4 cuidados críticos no merge

### 1. Avisos de custo/segurança ficam inline e em destaque
Ao fundir operações pagas com gratuitas, o aviso NÃO pode virar uma linha perdida. Manter em destaque, na seção que executa a ação:
- **`/nuvia`**: enriquecimento "gasta créditos de verdade sobre TODOS os contatos elegíveis do recorte — confirmação explícita antes" fica em bloco próprio, não enterrado entre listas/contatos (que são grátis).
- **`/nuvia-search`**: "busca global (`search_prospects`/`search_businesses`) consome créditos; BR (`search_brazil_companies`) é grátis" fica no topo da seção de busca global.
- **`/nuvia-deep`**: o único passo pago é a ponte para a base global — manter o aviso "gaste crédito só ali".

### 2. Descriptions = união dos gatilhos das originais
A `description` é o que faz a skill ativar. Cada description fundida precisa carregar TODOS os gatilhos concretos das skills originais. Não generalizar para "gerencie a Nuvia".

- **`/nuvia`** deve cobrir: cadastrar contato (individual/lote), atualizar dados, conferir se contato existe, evitar duplicado por telefone, registrar nota, criar Lista, adicionar coluna, inserir registros, filtrar linhas, editar em lote, enriquecer e-mail/telefone antes de campanha, salvar resultado de busca como Pessoas/Empresas dentro de Lista.
- **`/nuvia-search`** deve cobrir: empresas BR por CNPJ/CNAE/UF/porte/situação/capital/sócio; empresas global por NAICS/porte/faturamento/localização/tech stack/intenção; decisores (CEO/C-level) por nome/domínio/CNPJ ou por recorte.
- **`/nuvia-deep`** e **`/nuvia-conversas`**: manter as descriptions atuais (só ajustar o nome se citado).

### 3. Reestruturar, não concatenar
SKILL.md fundido deve ser enxuto e rotear para `references/`. Não colar os 5 (ou 3) arquivos em sequência. Seções com propósito claro; o conhecimento pesado fica nas referências já existentes.

### 4. Cross-references entre skills
- `/nuvia-search` aponta: "para salvar os resultados na plataforma, veja `/nuvia`" (fluxo busca→salvar não fica órfão).
- `/nuvia-search`, `/nuvia-deep`, `/nuvia-conversas` apontam: "leia a skill `nuvia` primeiro" + paths `nuvia/references/...`.

## Reescrita de referências cruzadas (path)
Toda menção a `nuvia-fundamentos` e `nuvia-fundamentos/references/...` deve virar `nuvia` e `nuvia/references/...`. Ocorrências conhecidas:
- `pesquisar-empresas-br/SKILL.md`: linhas 10, 16, 18 (vira `/nuvia-search`)
- `prospectar-decisores/SKILL.md`: linha 10 (vira `/nuvia-search`)
- `inspecionar-atendimento/SKILL.md`: linha 10 (vira `/nuvia-conversas`)
- `investigar-empresa-br/SKILL.md`: linhas 10, 52, 63 (vira `/nuvia-deep`)
- a referência interna `references/playbook-qsa-digital.md` em `investigar-empresa-br` permanece relativa (não muda).

## O que NÃO muda
- `marketplace.json`: não lista skills individualmente (descoberta automática pela pasta). Não há índice manual de skills para editar.
- `references/` de `investigar-empresa-br` (`playbook-qsa-digital.md`) acompanha a skill renomeada para `/nuvia-deep`.
- Conteúdo técnico das referências de fundamentos (só muda o path da pasta, não o conteúdo).

## Plano de arquivos
**Criar:**
- `plugin/skills/nuvia/SKILL.md` (+ mover `references/` de fundamentos)
- `plugin/skills/nuvia-search/SKILL.md`

**Renomear pasta + ajustar SKILL.md:**
- `investigar-empresa-br/` → `nuvia-deep/` (mantém `references/playbook-qsa-digital.md`)
- `inspecionar-atendimento/` → `nuvia-conversas/`

**Apagar (conteúdo absorvido):**
- `nuvia-fundamentos/`, `gerenciar-listas/`, `gerenciar-contatos/`, `enriquecer-contatos/`, `salvar-na-nuvia/`
- `pesquisar-empresas-br/`, `pesquisar-empresas-global/`, `prospectar-decisores/`

## Verificação
- 4 pastas em `plugin/skills/`, cada `name:` batendo com a pasta.
- Nenhuma menção remanescente a `nuvia-fundamentos`, `gerenciar-`, `pesquisar-empresas-`, `prospectar-decisores`, `inspecionar-atendimento`, `investigar-empresa-br` em nenhum SKILL.md.
- Avisos de custo presentes e em destaque em `/nuvia` (enriquecer) e `/nuvia-search` (busca global).
- Descriptions fundidas contêm a união dos gatilhos.
- `grep` por paths `nuvia/references/` resolve para arquivos existentes.
