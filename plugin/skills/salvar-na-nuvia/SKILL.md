---
name: salvar-na-nuvia
description: Materializa resultados de busca (empresas ou pessoas) como registros na Nuvia, opcionalmente dentro de uma Lista. Use depois de uma busca curada, quando o usuário quer trabalhar os contatos/empresas dentro da plataforma. Cuida do job assíncrono e confirma a importação. É gratuito.
---

# Salvar resultados de busca na Nuvia

Transforma o que você achou nas buscas (`search_prospects`, `search_businesses`, `search_brazil_companies`) em **registros** na Nuvia (Pessoas ou Empresas), opcionalmente dentro de uma **Lista**. A importação é assíncrona (job).

**Antes de começar, leia a skill `nuvia-fundamentos`** (linguagem, fluxo geral).

## Pré-requisito: ter uma busca curada

Só salve depois de refinar a busca para um recorte relevante. Salvar milhares de registros raramente ajuda. Pegue os identificadores (`ids[]`) dos resultados que você quer materializar — até 2000 por vez.

## Fluxo

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

- `save_mode: "contacts"` cria só os registros (em Pessoas/Empresas). `save_mode: "list"` também adiciona a uma Lista.
- O job **expira em minutos** — consulte `get_save_status` logo após o save, repetindo até `completed` ou `failed`.
- O mapeamento de campos é automático.
- **Salvou numa Lista? Devolva o link** para o usuário abrir: `https://app.nuvia.ai/lists/<list_id>` (use o `list_id` informado ou o retornado quando a Lista foi criada).

## Cuidado com Listas

Criar Lista **não tem deduplicação por nome** — passar um `list_name` novo cria outra Lista. Se o usuário quer reusar uma Lista existente, confirme o `list_id` (skill `gerenciar-listas` ou `list_lists`) antes de salvar. Lembre que **Listas são estáticas**: representam um recorte do momento em que foram criadas; novos registros que atenderiam aos mesmos critérios depois **não entram sozinhos**.

## Depois de salvar

- Trabalhar os registros, adicionar colunas, criar Vistas → skill `gerenciar-listas`.
- Enriquecer e-mail/telefone → skill `enriquecer-contatos` (consome crédito; leia os cuidados).
- Editar Pessoas/Empresas individualmente → skill `gerenciar-contatos`.

## Direcionar o usuário na plataforma

Se o usuário quiser fazer isso pela interface (ou entender o conceito de Listas/registros/Vistas), aponte a central de ajuda — ver os links em `nuvia-fundamentos/references/ajuda-nuvia.md`.
