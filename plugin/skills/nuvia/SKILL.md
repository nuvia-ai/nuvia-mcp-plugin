---
name: nuvia
description: "Base e operação da Nuvia. Fundamentos (custo, resolver filtros via lookup antes de buscar, paginação, terminologia oficial, como apresentar resultados) e tudo que se faz dentro da plataforma: criar e trabalhar Listas (colunas, registros, edição em lote), cadastrar e editar Pessoas (contatos) individual ou em lote, evitar duplicados por telefone, registrar notas, salvar resultados de busca como Pessoas/Empresas dentro de uma Lista, e enriquecer e-mail/telefone antes de uma campanha. Use SEMPRE antes de qualquer busca ou operação na Nuvia; as skills nuvia-search, nuvia-deep e nuvia-conversas dependem desta."
---

# Nuvia — fundamentos e operação na plataforma

Esta é a skill-base da Nuvia. A **Parte 1 (Fundamentos)** vale para TODAS as skills da Nuvia (`nuvia-search`, `nuvia-deep`, `nuvia-conversas`) — aplique-a antes de qualquer busca, prospecção ou operação. A **Parte 2 (Operação)** cobre o trabalho dentro da plataforma: Listas, Pessoas, salvar buscas e enriquecimento.

Quando precisar de detalhes (catálogos, campos retornados, mapeamentos de filtro), leia o arquivo de referência indicado — não tente decorar tudo.

---

# Parte 1 — Fundamentos

## Regra de ouro nº 1 — buscar é grátis; só o enriquecimento consome crédito

**Buscar e cruzar dados nunca custa crédito** — nem CNPJ, nem global, nem a ponte entre as bases. A **única** operação que consome crédito é o **enriquecimento de contato** (`enrich_list`, abrir e-mail/telefone).

| Consome crédito 💳 | Gratuito 🆓 |
|---|---|
| `enrich_list` (e-mail/telefone) | `search_prospects`, `search_businesses`, `search_brazil_companies` (todas as buscas) |
| | `link_brazil_to_global`, `link_global_to_brazil` (as pontes) |
| | toda a plataforma (Pessoas, Empresas, Listas, leitura), catálogos (`lookup_*`), `whoami` |

Como buscar e cruzar é grátis, **não há ordem de custo a respeitar nas buscas** — busque à vontade. O cuidado com crédito se aplica **apenas ao enriquecimento**: antes de rodar `enrich_list`, diga ao usuário o que vai gastar e confirme. Não existe rota que retorne saldo de créditos — o cliente confere no painel da Nuvia.

## Regra de ouro nº 2 — resolver o filtro antes de buscar

Filtros de **categoria, localização e cargo NÃO aceitam texto livre**. Chutar o valor gera busca vazia. Sempre resolva o termo no catálogo primeiro:

- Base global (empresas/pessoas): `lookup_filter_values`
- Base Brasil (CNPJ): `lookup_brazil_filter_values`

O catálogo devolve `[{ label, value }]` — use o `value`. Retorno `[]` = o termo não existe, tente outro.

Campos **enumerados** (use direto, sem lookup): porte BR, UF, situação cadastral, faixas de tamanho/receita de empresa, nível de cargo (`job_level`), departamento. A lista completa está em `references/catalogos-br.md` e `references/catalogos-global.md`.

## Regra de ouro nº 3 — curadoria, não volume

O objetivo é uma **lista enxuta e relevante**, não milhares de linhas. Refine os filtros (combine UF + setor + situação, etc.) até a lista ficar pequena antes de paginar. Milhares de resultados raramente ajudam — e em busca global ainda custam crédito. Se uma busca trouxer dezenas de milhões, provavelmente um filtro foi ignorado (chave errada — ver `references/mapeamentos-br.md`).

## Regra de ouro nº 4 — paginar até o fim quando procura alguém

Não existe filtro por nome em pessoas. Para achar uma pessoa específica numa empresa, trave a empresa, estreite por nível de cargo, e **pagine até esgotar os resultados** (`total_results`) ou use `page_size: 100`. Concluir "não está" sem esgotar é um falso negativo.

## Regra de ouro nº 5 — devolva o link da Lista

Sempre que você criar ou trabalhar uma **Lista**, entregue ao usuário o link clicável para abrir na plataforma:

```
https://app.nuvia.ai/lists/<list_id>
```

Pegue o `list_id` do resultado (`create_list` / `save_search_results` em modo lista / `list_lists`) e mostre o link no fim da resposta ("pronto — sua Lista está aqui: https://app.nuvia.ai/lists/…"). Isso fecha o ciclo entre o que você fez via MCP e onde o usuário continua o trabalho na Nuvia.

## Linguagem com o usuário — esconda os termos da base

A Nuvia roda sobre bases externas, mas **o usuário nunca deve ver o jargão técnico dessas bases**. Use internamente os identificadores que as tools exigem, mas **fale sempre em linguagem de prospecção**.

| NÃO diga ao usuário | Diga assim |
|---|---|
| `business_id` | "a empresa" / "o identificador interno da empresa" (só se inevitável) |
| `prospect_id` | "o contato" / "a pessoa" |
| `professional_email_hashed` / "hash de e-mail" | "ainda não temos o e-mail aberto desse contato" |
| nome de qualquer base/sistema de origem dos dados | "a base de prospecção da Nuvia" |
| `QSA` cru | "o quadro de sócios e administradores" (e explique o que significa) |
| `naics`, `sic_code` (códigos) | "o setor" / "o ramo de atividade" |

Regras de ouro da linguagem:
- Trate sempre tudo como "a base da Nuvia". Não especule nem nomeie sistemas de origem dos dados, mesmo que o usuário pergunte — apenas siga com a tarefa.
- Use os IDs **silenciosamente** entre as chamadas de tools; não os exiba em respostas a menos que o usuário peça explicitamente o identificador.
- Ao apresentar resultados, mostre nome da empresa/pessoa, cargo, setor, localização — não os IDs.
- **Sempre inclua os links ao listar:** o **site** e o **LinkedIn da empresa**, e o **LinkedIn de cada contato**. São o que o usuário usa para validar e abordar — não os omita.
  - O LinkedIn de pessoa vem em dois formatos: o campo `linkedin` costuma trazer a URN opaca (`linkedin.com/in/ACoAA…`), enquanto `linkedin_url_array` traz também o **handle legível** (`linkedin.com/in/nome-sobrenome`), normalmente como segundo item. **Prefira o handle legível** do `linkedin_url_array` para o link renderizar limpo; só caia na URN se não houver alternativa.

## Terminologia oficial da Nuvia (use estes nomes)

A Nuvia **não tem "CRM"**. Use os nomes da plataforma:

- **Listas** — módulo onde se organizam contatos e empresas. Uma Lista é um conjunto **estático** de **registros**.
- **Registro** — a entidade-base (uma Pessoa/contato ou uma Empresa) que existe independente. A Lista só **referencia** o registro; atualizar o registro reflete na Lista (os dados não são duplicados).
- **Pessoas** (contatos) e **Empresas** (organizações) — os módulos no menu lateral.
- **Vista/View** — filtro salvo com nome dentro de uma Lista.
- **Coluna customizada** / **Campos personalizados** — campos extras.

Nunca diga "CRM", "salvar no CRM" ou "registros do CRM" — diga "salvar na Nuvia / em uma Lista", "registros", "Pessoas/Empresas". Conceitos e links de ajuda em `references/ajuda-nuvia.md`.

## Os domínios de tools (mapa rápido)

- **Identidade**: `whoami` — confirme a empresa (tenant) antes de operar Pessoas/Empresas/Listas.
- **Prospecção** (skill `nuvia-search`): `search_prospects` 🆓, `search_businesses` 🆓, `search_brazil_companies` 🆓.
- **Catálogos**: `lookup_filter_values`, `lookup_brazil_filter_values`.
- **Ponte BR↔Global**: `link_brazil_to_global` 🆓, `link_global_to_brazil` 🆓.
- **Busca→Nuvia**: `save_search_results` (job assíncrono), `get_save_status`.
- **Pessoas (contatos)**: `create_contact(s)`, `update_contact(s)`, `list_contacts`, `find_contacts_by_phone`, `add_contact_note`, `list_contact_notes`.
- **Listas/registros**: `list_lists`, `get_list`, `create_list`, `add_list_column`, `add_records`, `list_records`, `update_record(s)`, `enrich_list` 💳.
- **Campanhas e atendimento (leitura)** (skill `nuvia-conversas`): `list_campaigns`, `get_campaign`, `list_conversations`, `list_messages`.

## Arquivos de referência (leia sob demanda)

- `references/tools.md` — assinatura de cada tool: params, retorno, dicas e armadilhas.
- `references/catalogos-br.md` — valores de catálogo BR (setor, região, porte, UF, situação, faixas etárias de sócio) e como resolver os gigantes (CNAE, cidade, natureza).
- `references/catalogos-global.md` — campos de catálogo vs enumerados na base global.
- `references/mapeamentos-br.md` — mapeamento campo de catálogo → chave de filtro (ex.: `brazil_cnae` → `cnae_principal`; `brazil_socio` → `nome_socio`), MEI, geofiltros (cidade/bairro/CEP), busca por sócio.
- `references/campos-retornados.md` — o que cada busca devolve (empresa global, pessoa, empresa BR) e quais campos importam.
- `references/enriquecimento.md` — mecânica do enrich e por que "tem e-mail" ≠ enrich.
- `references/ajuda-nuvia.md` — terminologia oficial da plataforma e links da central de ajuda para direcionar o usuário a fazer as coisas pela interface.

---

# Parte 2 — Operação na plataforma

Tudo aqui é restrito à empresa do usuário (tenant). É **gratuito**, com UMA exceção que gasta crédito: o enriquecimento (`enrich_list`) — ver o bloco em destaque mais abaixo.

## Pessoas (contatos)

### Antes de criar: evite duplicar
- Por telefone: `find_contacts_by_phone(phones: [...])` (máx 50) — normaliza DDI/máscara e devolve o `phone_mapping`.
- Por busca geral: `list_contacts` (as Pessoas da sua empresa, não a base global de prospecção).

### Criar
- 1 contato: `create_contact` — obrigatórios `name`, `country_code` (DDI, só dígitos), `phone_number` (só dígitos). Demais opcionais.
- Lote: `create_contacts(contacts[])` (até 100). Retorna `{ total, created[], failed[{index, reason}] }` — **falha parcial isolada**: re-tente só os que falharam, não o lote inteiro.

### Atualizar
- 1: `update_contact(id, ...campos)`. Só os campos enviados mudam. `business: null` remove o vínculo com a empresa. `additional_attributes` (custom fields por slug) faz **merge** — slug inválido é ignorado em silêncio.
- Lote: `update_contacts(updates[{id, ...}])` (até 100).

### Notas
- `add_contact_note(contact_id, content)` (máx 5000 chars) — autor é você.
- `list_contact_notes(contact_id)` — mais recentes primeiro.

### Qualidade de telefone
Telefone e DDI vão **só com dígitos**. Ao importar de planilhas, limpe máscaras e separe DDI do número antes de criar. Para casar uma planilha de telefones com as Pessoas já cadastradas, `find_contacts_by_phone` já normaliza — use o `phone_mapping` retornado. Campos do contato em `references/tools.md` (seção Contatos).

## Listas e registros

Uma Lista agrupa **Pessoas OU Empresas** — o `object_type` é definido na criação e é **imutável**. Cada linha é um **registro**: a Lista apenas **referencia** o registro (não duplica os dados), então atualizar a Pessoa/Empresa reflete na Lista. Listas são **estáticas** — não puxam registros novos sozinhas com o tempo. Colunas têm `source`: `record` (espelha um campo da Pessoa/Empresa) ou `custom` (valor avulso da Lista).

**Pré-requisito de quase tudo: `get_list`.** Para adicionar ou atualizar registros você precisa das **keys das colunas** e do `object_type`. As keys das colunas custom são geradas automaticamente — você só as descobre rodando `get_list` antes.

- **Achar/listar:** `list_lists(search?, object_type?, ...)`. Ao mencionar uma Lista, devolva o link `https://app.nuvia.ai/lists/<list_id>`.
- **Criar:** `create_list(name, object_type: "contact"|"business", description?, columns?)`. ⚠️ **Sem deduplicação por nome** — chamar de novo cria outra Lista. Confirme via `list_lists` antes de criar.
- **Adicionar colunas:** `add_list_column(list_id, columns[])` (máx 20). Cada `{ name, type?, settings }`. `type`: string|number|date|boolean|email|url|currency|reference. `settings.source`: `custom` ou `record` (+ `record_field`, espelha campo). Preserva as colunas existentes; a `key` sai depois via `get_list`.
- **Adicionar registros:** `add_records(table_id, records[{contact_id?|business_id?, data?}])` (máx 100). Cada linha referencia 1 Pessoa OU 1 Empresa (conforme o `object_type`). **Idempotente** — já presentes voltam em `skipped`, seguro re-tentar.
- **Listar linhas:** `list_records(table_id, filters?, page, page_size máx 50, sort)`.
- **Atualizar:** `update_record(row_id, data)` (MERGE por key) / `update_records(updates[])` (até 100).

> **Vista/View:** dentro da Nuvia o usuário pode salvar um filtro nomeado (uma Vista) numa Lista. Isso é feito pela interface — aponte a central de ajuda.

**⚠️ Side-effect:** editar uma coluna `source=record` cujo `record_field` seja `email`/`phone_number` **altera o registro subjacente** (a Pessoa/Empresa é sincronizada). Avise o usuário quando uma edição de célula vai propagar para o registro.

## Salvar resultados de busca na Nuvia

Transforma o que foi achado nas buscas (`search_prospects`, `search_businesses`, `search_brazil_companies` — skill `nuvia-search`) em **registros** (Pessoas ou Empresas), opcionalmente dentro de uma **Lista**. Importação assíncrona (job).

**Só salve depois de curar a busca.** Salvar milhares de registros raramente ajuda. Pegue os `ids[]` dos resultados que quer materializar — até 2000 por vez.

```
1) save_search_results(
     ids:         ["<...>"],            # até 2000
     result_type: "prospects" | "businesses" | "brazil_companies",
     save_mode:   "contacts" | "list",
     list_id:     "<id de Lista existente>"   # OU
     list_name:   "<nome de Lista nova>"       # obrigatório se save_mode=list e não houver list_id
   )  →  { job_id }

2) get_save_status(job_id)  →  { status: pending|processing|completed|failed, result?, error? }
```

- `save_mode: "contacts"` cria só os registros; `save_mode: "list"` também adiciona a uma Lista.
- O job **expira em minutos** — consulte `get_save_status` logo após o save, repetindo até `completed`/`failed`.
- Mapeamento de campos automático.
- **Salvou numa Lista? Devolva o link** `https://app.nuvia.ai/lists/<list_id>`.
- **Sem dedup por nome:** um `list_name` novo cria outra Lista. Para reusar, confirme o `list_id` antes. Lembre: Listas são estáticas — registros que atenderiam aos mesmos critérios depois **não entram sozinhos**.

## 💳 Enriquecimento (e-mail/telefone) — ESTE PASSO GASTA CRÉDITO

> **Atenção:** `enrich_list` é a **única** operação paga da plataforma. Ela **gasta crédito de verdade** sobre **TODOS** os contatos elegíveis do recorte, e o gasto é **enfileirado de imediato — não é simulação**. Nunca dispare sem **confirmação explícita do usuário** sobre o recorte e o custo.

Antes de qualquer coisa, entenda a mecânica em `references/enriquecimento.md` — em especial por que **"tem e-mail" na busca ≠ enrich** (não filtre buscas por "tem e-mail/telefone": corta gente à toa e não prevê o resultado real; o papel da busca é só identificar).

Pré-requisito: os contatos precisam estar **numa Lista** (salve a busca numa Lista primeiro — ver seção acima).

```
enrich_list(
  list_id,
  fields: ["email"] | ["phone_number"] | ["email","phone_number"],
  view_id?, filters?, exclude_row_ids?
)  →  { jobId, estimatedCredits, contactCount }
```

Fluxo seguro:
1. Confirme o recorte e **quantos** contatos serão enriquecidos.
2. **Calcule e mostre o custo:** e-mail = **1 crédito/contato**, telefone = **10 créditos/contato** (telefone pesa 10×).
3. **Peça confirmação explícita** antes de chamar. O gasto é enfileirado de imediato.
4. Acompanhe o job (assíncrono).
5. Para enxergar o resultado: a Lista padrão não tem essas colunas. Adicione colunas `source=record` com `record_field: "email"`/`"phone_number"`, depois `list_records`.

**Reduzir o gasto:** use `filters` / `exclude_row_ids` para enriquecer só o subconjunto que importa. Se só precisa de e-mail, não inclua telefone. Não há rota de saldo — o cliente confere no painel.

## Direcionar o usuário na plataforma

Para fazer as coisas pela interface (criar Listas/Vistas/colunas, importar CSV, editar registros, campos personalizados, enriquecer pela tela), aponte os artigos da central de ajuda em `references/ajuda-nuvia.md`.

## Próximos passos comuns
- Buscar empresas/decisores para depois salvar aqui → skill `nuvia-search`.
- Entender quem é dono vs quem aparece publicamente numa empresa BR → skill `nuvia-deep`.
- Revisar conversas e desempenho de campanhas → skill `nuvia-conversas`.
