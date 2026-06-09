---
name: gerenciar-listas
description: Cria e trabalha Listas na Nuvia — Listas de Pessoas OU Empresas, com colunas próprias, registros e edição em lote. Use quando o usuário quer organizar contatos/empresas em Listas, adicionar colunas (customizadas ou espelhando campos), inserir registros, listar/filtrar linhas, ou atualizar células. Gratuito.
---

# Gerenciar Listas na Nuvia

Uma Lista agrupa **Pessoas OU Empresas** — o `object_type` é definido na criação e é **imutável**. Cada linha é um **registro**: a Lista apenas **referencia** o registro (não duplica os dados), então atualizar a Pessoa/Empresa reflete na Lista. Listas são **estáticas** — não puxam registros novos sozinhas com o tempo. Colunas têm `source`: `record` (espelha um campo da Pessoa/Empresa) ou `custom` (coluna customizada, valor avulso da Lista).

**Antes de começar, leia a skill `nuvia-fundamentos`** (linguagem e terminologia). Detalhes das tools em `nuvia-fundamentos/references/tools.md` (seção Listas).

## Pré-requisito de quase tudo: `get_list`

Para adicionar ou atualizar registros você precisa das **keys das colunas** e do `object_type`. Sempre rode `get_list` antes — as keys das colunas custom são geradas automaticamente e você só as descobre aqui.

## Operações

- **Achar/listar Listas:** `list_lists(search?, object_type?, ...)`. Ao mencionar uma Lista ao usuário, devolva o link: `https://app.nuvia.ai/lists/<list_id>`.
- **Criar:** `create_list(name, object_type: "contact"|"business", description?, columns?)`. ⚠️ **Sem deduplicação por nome** — chamar de novo cria outra Lista. Confirme com o usuário se a Lista já existe (via `list_lists`) antes de criar.
- **Adicionar colunas:** `add_list_column(list_id, columns[])` (máx 20). Cada `{ name, type?, settings }`. `type`: string|number|date|boolean|email|url|currency|reference. `settings.source`: `custom` (coluna customizada) ou `record` (+ `record_field`, espelha campo da Pessoa/Empresa). Preserva as colunas existentes. A `key` sai depois via `get_list`.
- **Adicionar registros:** `add_records(table_id, records[{contact_id?|business_id?, data?}])` (máx 100). Cada linha referencia 1 Pessoa OU 1 Empresa (conforme o `object_type`). **Idempotente** — já presentes voltam em `skipped`, seguro re-tentar.
- **Listar linhas:** `list_records(table_id, filters?, page, page_size máx 50, sort)`.
- **Atualizar:** `update_record(row_id, data)` (MERGE por key) / `update_records(updates[])` (até 100).

> **Vista/View:** dentro da Nuvia, o usuário pode salvar um filtro nomeado (uma Vista) numa Lista para segmentar sem criar outra Lista. Isso é feito pela interface — aponte a central de ajuda.

## ⚠️ Side-effect importante

Editar uma coluna `source=record` cujo `record_field` seja `email`/`phone_number` **altera o registro subjacente** (a Pessoa/Empresa é sincronizada). Não muda o vínculo do registro. Avise o usuário quando uma edição de célula vai propagar para o registro.

## Visualizar resultado de enriquecimento

A Lista padrão não tem colunas de e-mail/telefone. Para enxergar o que o enriquecimento preencheu: `add_list_column` com `source=record` + `record_field: "email"`/`"phone_number"`, depois `list_records`.

## Direcionar o usuário na plataforma

Para fazer isso pela interface (criar Listas, Vistas, colunas, importar CSV), aponte os artigos em `nuvia-fundamentos/references/ajuda-nuvia.md`.
