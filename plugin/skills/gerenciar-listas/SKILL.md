---
name: gerenciar-listas
description: Cria e trabalha listas/tabelas do CRM da Nuvia — listas de contatos OU empresas, com colunas próprias, registros e edição em lote. Use quando o usuário quer organizar contatos/empresas em tabelas, adicionar colunas (custom ou espelhando campos), inserir registros, listar/filtrar linhas, ou atualizar células. Gratuito.
---

# Gerenciar listas e tabelas do CRM

Uma lista agrupa **contatos OU empresas** — o `object_type` é definido na criação e é **imutável**. Colunas têm `source`: `record` (espelha um campo do contato/empresa) ou `custom` (valor avulso da lista).

**Antes de começar, leia a skill `nuvia-fundamentos`** (linguagem). Detalhes das tools em `nuvia-fundamentos/references/tools.md` (seção Listas).

## Pré-requisito de quase tudo: `get_list`

Para adicionar ou atualizar registros você precisa das **keys das colunas** e do `object_type`. Sempre rode `get_list` antes — as keys das colunas custom são geradas automaticamente e você só as descobre aqui.

## Operações

- **Achar/listar listas:** `list_lists(search?, object_type?, ...)`.
- **Criar:** `create_list(name, object_type: "contact"|"business", description?, columns?)`. ⚠️ **Sem dedup por nome** — chamar de novo cria outra lista. Confirme com o usuário se a lista já existe (via `list_lists`) antes de criar.
- **Adicionar colunas:** `add_list_column(list_id, columns[])` (máx 20). Cada `{ name, type?, settings }`. `type`: string|number|date|boolean|email|url|currency|reference. `settings.source`: `custom` ou `record` (+ `record_field`). Preserva as colunas existentes. A `key` sai depois via `get_list`.
- **Adicionar registros:** `add_records(table_id, records[{contact_id?|business_id?, data?}])` (máx 100). Cada linha referencia 1 contato OU 1 empresa (conforme o `object_type`). **Idempotente** — já presentes voltam em `skipped`, seguro re-tentar.
- **Listar linhas:** `list_records(table_id, filters?, page, page_size máx 50, sort)`.
- **Atualizar:** `update_record(row_id, data)` (MERGE por key) / `update_records(updates[])` (até 100).

## ⚠️ Side-effect importante

Editar uma coluna `source=record` cujo `record_field` seja `email`/`phone_number` **altera o contato subjacente** (sincroniza). Não muda o vínculo do registro. Avise o usuário quando uma edição de célula vai propagar para o contato.

## Visualizar resultado de enriquecimento

A lista default não tem colunas de e-mail/telefone. Para enxergar o que o enrich preencheu: `add_list_column` com `source=record` + `record_field: "email"`/`"phone_number"`, depois `list_records`.
