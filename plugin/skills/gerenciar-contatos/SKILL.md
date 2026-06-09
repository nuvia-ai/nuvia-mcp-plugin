---
name: gerenciar-contatos
description: Cria, edita, lista e anota Pessoas (contatos) na Nuvia, e reconcilia telefones com a base. Use quando o usuário quer cadastrar contatos (individual ou em lote), atualizar dados, conferir se um contato já existe, dedupar por telefone, ou registrar notas. Tudo gratuito e dentro da empresa do usuário.
---

# Gerenciar Pessoas (contatos) na Nuvia

Operações de contato são gratuitas e restritas à empresa do usuário (tenant).

**Antes de começar, leia a skill `nuvia-fundamentos`** (linguagem, fluxo). A lista de campos do contato está em `nuvia-fundamentos/references/tools.md` (seção Contatos).

## Antes de criar: evite duplicar

Confira se o contato já existe:
- Por telefone: `find_contacts_by_phone(phones: [...])` (máx 50) — normaliza DDI/máscara e devolve o mapeamento.
- Por busca geral: `list_contacts` (busca as Pessoas da sua empresa, não a base global de prospecção).

## Criar

- 1 contato: `create_contact` — obrigatórios `name`, `country_code` (DDI, só dígitos), `phone_number` (só dígitos). Demais opcionais.
- Lote: `create_contacts(contacts[])` (até 100). Retorna `{ total, created[], failed[{index, reason}] }` — **falha parcial isolada**: re-tente só os que falharam, não o lote inteiro.

## Atualizar

- 1: `update_contact(id, ...campos)`. Só os campos enviados mudam. `business: null` remove o vínculo com a empresa. `additional_attributes` (custom fields por slug) faz **merge** — slug inválido é ignorado em silêncio.
- Lote: `update_contacts(updates[{id, ...}])` (até 100).

## Notas

- `add_contact_note(contact_id, content)` (máx 5000 chars) — autor é você.
- `list_contact_notes(contact_id)` — mais recentes primeiro.

## Dica de qualidade

Telefone e DDI vão **só com dígitos**. Ao importar de planilhas, limpe máscaras e separe DDI do número antes de criar. Para casar uma planilha de telefones com as Pessoas já cadastradas, `find_contacts_by_phone` já normaliza — use o `phone_mapping` retornado para saber qual número casou.

## Importação em massa pela plataforma

Para grandes importações (ex.: CSV), o usuário pode usar a própria interface da Nuvia. Aponte os artigos de ajuda em `nuvia-fundamentos/references/ajuda-nuvia.md` (importar via CSV, editar registros, campos personalizados).
