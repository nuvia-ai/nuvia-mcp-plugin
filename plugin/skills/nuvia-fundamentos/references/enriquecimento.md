# Enriquecimento de e-mail/telefone — mecânica e cuidados

## "Tem e-mail" na busca ≠ enrich (são coisas diferentes)

| | Sinal na **busca** (`search_*`) | **Enriquecimento** |
|---|---|---|
| O que entrega | apenas um indicador de presença (hash) — não o e-mail | o e-mail/telefone reais |
| Filtros | `has_email`/`has_phone_number` filtram sobre esse indicador | — |
| Uso | identificar a pessoa | obter o contato acionável |

**Consequência:** filtrar a busca por "tem e-mail" corta gente que o enriquecimento resolveria, baseado num sinal sem relação com o resultado final. **Não use esses filtros.** O papel da busca é identificar; o contato acionável vem depois.

## Como `enrich_list` funciona

1. Roda sobre uma **lista do CRM** (não sobre resultados de busca soltos). Fluxo: `save_search_results` → lista → `enrich_list`.
2. **Assíncrono e por contato.** Custo: e-mail = 1 crédito · telefone = 10 créditos, por contato. Roda sobre **todos** os elegíveis do recorte. Retorna `{ jobId, estimatedCredits, contactCount }` — o gasto é enfileirado, **não é simulação**.
3. Grava o resultado **no contato** e nas colunas da lista que espelham `record_field` email/phone. Para enxergar: `add_list_column` com `source=record` + `record_field: "email"`/`"phone_number"`, depois `list_records`.

## Regra de ouro

**Sempre confirme com o usuário o custo estimado antes de disparar.** Telefone pesa 10× o e-mail — se o usuário só precisa de e-mail, não inclua telefone. Use `filters`/`exclude_row_ids` para enriquecer só o subconjunto que importa. Não há rota de saldo de créditos — o cliente confere no painel da Nuvia.
