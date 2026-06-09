# Referência das tools do MCP da Nuvia

Convenção: **O quê** · **Quando** · **Params** · **Retorna** · 💡 dica / ⚠️ armadilha.
Marcas: 💳 consome crédito · 🆓 grátis · ⏳ assíncrono (job).

## Identidade

### `whoami` 🆓
- **O quê.** Identidade autenticada na sessão.
- **Quando.** Sanity check ("estou na empresa certa?") e para obter o id da empresa antes de operar Pessoas/Empresas/Listas.
- **Params:** nenhum. **Retorna:** `{ userId, companyId, userName, companyName, scopes[] }`.

## Prospecção / Descoberta

### `search_prospects` 💳
- **O quê.** Pessoas/decisores na base global (perfil estilo LinkedIn).
- **Quando.** Achar decisores de uma empresa (ancorando pelo identificador da empresa) ou por recorte (cargo + país + setor).
- **Pré-requisito:** cargo/categoria/localização só aceitam catálogo → resolva com `lookup_filter_values(entity_type="prospects")` antes.
- **Params (`filters`/`exclude`):** `job_title {values[], include_related_job_titles?}`, `job_department {values[]}`, `job_level {values[]}` (owner|c-suite|vice president|director|manager|partner|senior non-managerial|non-managerial|junior), identificador da empresa, `company_name`, `company_size`, `company_revenue`, `country_code`, `company_country_code`, `region_country_code` (`us-ca`), `city_region_country` (`Sao Paulo, BR`), `naics_category`, `google_category`, `linkedin_category`, `total_experience_months`/`current_role_months {gte,lte}`, `page`, `page_size` (50, máx 100).
- **Retorna:** perfis (ver `campos-retornados.md`): `full_name`, `job_title`, níveis de cargo, links de LinkedIn, etc.
- ⚠️ **Sem filtro por nome.** Para achar alguém: trave o identificador da empresa, estreite por nível, pagine até o fim.
- ⚠️ **Não use** `has_email`/`has_phone_number` (corta recall, não prevê enrich — ver `enriquecimento.md`).

### `search_businesses` 💳
- **O quê.** Empresas na base global por firmográficos (setor/NAICS, porte, faturamento, tech stack, localização, intenção de compra).
- **Quando.** Prospecção de contas com foco global, ou para obter o identificador e firmográficos de uma empresa.
- **Pré-requisito:** resolva categorias/localização com `lookup_filter_values(entity_type="businesses")` antes.
- **Params (`filters`/`exclude`):** `company_name {values[]}`, identificador da empresa, `country_code`, `region_country_code`, `city_region_country`, `city_region`, `company_size`, `company_revenue`, `company_age`, `number_of_locations`, `naics_category`, `google_category`, `linkedin_category`, `company_tech_stack_tech`, `company_tech_stack_category`, `website_keywords {keywords[], operator}`, `business_intent_topics {topics[], topic_intent_level}`, `events {values[], last_occurrence}`, `is_public_company`, `page`, `page_size` (50, máx 100).
- **Retorna:** empresa global (ver `campos-retornados.md`).
- 💡 Refine os filtros até uma lista enxuta antes de paginar.

### `search_brazil_companies` 🆓
- **O quê.** Empresas brasileiras na base nacional de CNPJ (Receita Federal) — com quadro de sócios, CNAE, situação, domínios.
- **Quando.** Ponto de partida de qualquer investigação BR (grátis); ideal buscar por **`cnpjs`** (match exato).
- **Pré-requisito:** UF/CNAE/natureza/cidade só aceitam catálogo → `lookup_brazil_filter_values` antes.
- **Params (`filters`, formato `{values:[...]}` salvo indicado):** `cnpjs`, `nome_empresa` (fuzzy — evite), `dominio`, `uf`, `municipio`, `cnae_principal`, `setor_economico`, `natureza_juridica`, `porte`, `situacao_cadastral`, `capital_social_min`/`capital_social_max` (números), `nome_socio`, `bairro`, `cep`, `filter_operator` (and|or), `page`, `page_size` (máx 100).
- **Retorna:** empresa BR (ver `campos-retornados.md`): `cnpj`, `natureza_juridica`, `socios[]`, `dominios[]`, `linkedin_url`, CNAEs, endereço, bloco `simples` (flag MEI).
- 💡 `cnpjs` é match exato; `nome_empresa` é fuzzy e traz milhares — prefira CNPJ.

## Catálogos

### `lookup_filter_values` 🆓
- **O quê.** Catálogo de valores para a base global.
- **Params:** `field`, `entity_type` (`businesses`|`prospects`), `query`.
- Detalhes de quais campos precisam de lookup em `catalogos-global.md`.

### `lookup_brazil_filter_values` 🆓
- **O quê.** Catálogo de valores para a base BR.
- **Params:** `field`, `query`.
- Detalhes em `catalogos-br.md` e mapeamento campo→filtro em `mapeamentos-br.md`.

## Ponte BR ↔ Global

### `link_brazil_to_global` 💳
- **O quê.** Empresa BR (CNPJ) → identificador na base global.
- **Quando.** Já tem a empresa BR e quer prospectar decisores ou firmográficos globais.
- **Params:** `cnpj` (obrig.), `domain?`, `company_name?`.
- **Retorna:** `{ business_id, company_data }` ou `{ found: false }`.
- 💡 **Passe sempre `domain` E `company_name`** (do resultado BR) — eleva muito o acerto. `found:false` → tente outro domínio antes de desistir.

### `link_global_to_brazil` 🆓
- **O quê.** Empresa global (domínio) → dados cadastrais BR (CNPJ, razão social, sócios, CNAE, situação).
- **Quando.** Tem só o domínio e quer o CNPJ + sócios (grátis); ótima âncora quando falta o CNPJ.
- **Params:** `domain` (sem http/www).
- **Retorna:** `{ cnpj, company_data }` ou `{ found: false }`.
- 💡 Use o domínio **institucional/histórico**; se falhar, tente `.com.br`/raiz do site.

## Busca → Nuvia

### `save_search_results` ✏️ ⏳
- **O quê.** Materializa resultados de busca como registros na Nuvia (Pessoas/Empresas), opcionalmente numa Lista.
- **Params:** `ids[]` (até 2000) · `result_type` (`prospects`|`businesses`|`brazil_companies`) · `save_mode` (`contacts`|`list`) · `list_id?` **ou** `list_name?` (obrigatório se `save_mode=list`).
- **Retorna:** `{ job_id }` → acompanhe com `get_save_status`.

### `get_save_status` 🆓
- **Params:** `job_id`. **Retorna:** `{ status: pending|processing|completed|failed, result?, error? }`. O job expira em minutos — consulte logo após o save.

## Pessoas / Contatos (restrito à sua empresa)

Campos: `name`, `country_code` (DDI, só dígitos), `phone_number` (só dígitos), `email`, `job_title`, `seniority`, `department`, `city`, `region`, `country`, `linkedin_identifier`, `experience`, `skills`, `business` (empresa interna), `additional_attributes` (custom fields por slug; slug inválido é ignorado em silêncio; faz merge).

- `create_contact` ✏️ — 1 contato. Obrigatórios: `name`, `country_code`, `phone_number`.
- `create_contacts` ✏️ — lote (até 100). Retorna `{ total, created[], failed[{index, reason}] }` (falha parcial isolada).
- `update_contact` ✏️ — `id` + campos. `business: null` remove o vínculo de empresa. `additional_attributes` faz merge.
- `update_contacts` ✏️ — lote (até 100). `updates[{id, ...}]`.
- `list_contacts` 🆓 — lista/busca os contatos (Pessoas) da sua empresa na Nuvia (não é a base global de prospecção).
- `find_contacts_by_phone` 🆓 — `phones[]` (máx 50) → `{ contacts, phone_mapping }`. Normaliza DDI/máscara.
- `add_contact_note` ✏️ — `contact_id`, `content` (máx 5000).
- `list_contact_notes` 🆓 — `contact_id` → notas, mais recentes primeiro.

## Listas / registros

Uma lista agrupa **contatos OU empresas** (`object_type`, imutável após criar). Colunas têm `source`: `record` (espelha campo do contato/empresa) ou `custom` (valor avulso).

- `list_lists` 🆓 — `search?`, `object_type?`, `page`, `page_size` (máx 50) → `{ data[{id,name,object_type,status,columns,row_count}], total, has_more }`.
- `get_list` 🆓 — metadados + colunas com as **keys**. **Pré-requisito** de `add_records`/`update_record`.
- `create_list` ✏️ — `name`, `object_type` (`contact`|`business`), `description?`, `columns?`. ⚠️ Sem dedup por nome — confirme antes de criar de novo.
- `add_list_column` ✏️ — `list_id`, `columns[]` (máx 20), cada `{name, type?, description?, settings}`. `settings.source`: `custom` ou `record` (+ `record_field`). A `key` é gerada automaticamente (busque depois via `get_list`).
- `add_records` ✏️ — `table_id`, `records[{contact_id?|business_id?, data?}]` (máx 100). **Idempotente** (já presentes voltam em `skipped`).
- `list_records` 🆓 — `table_id`, `filters?`, `page`, `page_size` (máx 50), `sort`.
- `update_record` ✏️ — `row_id`, `data` (por key). MERGE. ⚠️ Editar coluna `source=record` de e-mail/telefone **altera o contato subjacente**.
- `update_records` ✏️ — lote (até 100).
- `enrich_list` ✏️ 💳 ⏳ — ver `enriquecimento.md` antes de usar.

## Campanhas (leitura)
- `list_campaigns` 🆓 — inventário de campanhas.
- `get_campaign` 🆓 — `id` → `{ id, name, status, channel_type, trigger_type, metrics, ... }`.

## Atendimento / Inbox (leitura)
- `list_conversations` 🆓 — `search?`, `status?` (OPEN|RESOLVED), `channel?`, `page`, `page_size` (máx 50).
- `list_messages` 🆓 — id da conversa → mensagens, mais recentes primeiro.
