---
name: salvar-no-crm
description: Materializa resultados de busca (empresas ou pessoas) como registros no CRM da Nuvia, opcionalmente dentro de uma lista. Use depois de uma busca curada, quando o usuário quer trabalhar os contatos/empresas dentro da Nuvia. Cuida do job assíncrono e confirma o import. É gratuito.
---

# Salvar resultados de busca no CRM

Transforma o que você achou nas buscas (`search_prospects`, `search_businesses`, `search_brazil_companies`) em registros do CRM. O import é assíncrono (job).

**Antes de começar, leia a skill `nuvia-fundamentos`** (linguagem, fluxo geral).

## Pré-requisito: ter uma busca curada

Só salve depois de refinar a busca para uma lista relevante. Salvar milhares de registros raramente ajuda. Pegue os identificadores (`ids[]`) dos resultados que você quer materializar — até 2000 por vez.

## Fluxo

```
1) save_search_results(
     ids:         ["<...>"],            # até 2000
     result_type: "prospects" | "businesses" | "brazil_companies",
     save_mode:   "contacts" | "list",
     list_id:     "<id de lista existente>"   # OU
     list_name:   "<nome de lista nova>"       # obrigatório se save_mode=list e não houver list_id
   )  →  { job_id }

2) get_save_status(job_id)  →  { status: pending|processing|completed|failed, result?, error? }
```

- `save_mode: "contacts"` cria só os registros. `save_mode: "list"` também adiciona a uma lista.
- O job **expira em minutos** — consulte `get_save_status` logo após o save, repetindo até `completed` ou `failed`.
- O mapeamento de campos é automático.

## Cuidado com listas

`create_list`/`list_name` **não tem dedup por nome** — passar um `list_name` novo cria outra lista. Se o usuário quer reusar uma lista existente, confirme o `list_id` (use a skill `gerenciar-listas` ou `list_lists`) antes de salvar.

## Depois de salvar

- Trabalhar os registros, adicionar colunas, atualizar → skill `gerenciar-listas`.
- Enriquecer e-mail/telefone → skill `enriquecer-contatos` (consome crédito; leia os cuidados).
- Editar contatos individualmente → skill `gerenciar-contatos`.
