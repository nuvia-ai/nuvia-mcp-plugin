---
name: enriquecer-contatos
description: Enriquece e-mail e telefone dos contatos de uma lista do CRM da Nuvia. CONSOME CRÉDITOS (e-mail = 1/contato, telefone = 10/contato). Use quando o usuário pede explicitamente para preencher contatos faltantes. Sempre confirma o custo estimado antes de disparar, porque o gasto é enfileirado de imediato e roda sobre todos os contatos elegíveis do recorte.
---

# Enriquecer contatos de uma lista

Esta operação **gasta créditos de verdade** sobre **todos** os contatos elegíveis do recorte. Trate com cuidado e confirmação explícita.

**Antes de qualquer coisa, leia `nuvia-fundamentos/references/enriquecimento.md`** — ele explica por que "tem e-mail" na busca ≠ enrich e a mecânica completa.

## ⚠️ Confirme antes de gastar

O enriquecimento gasta crédito de verdade e o gasto é enfileirado de imediato (não é simulação). Antes de chamar `enrich_list`, confirme com o usuário o recorte e o custo estimado.

## Usar `enrich_list`

Pré-requisito: os contatos precisam estar **numa lista** (use `salvar-no-crm` para materializar uma busca numa lista primeiro).

```
enrich_list(
  list_id,
  fields: ["email"] | ["phone_number"] | ["email","phone_number"],
  view_id?, filters?, exclude_row_ids?
)  →  { jobId, estimatedCredits, contactCount }
```

Fluxo seguro:
1. Confirme o recorte e quantos contatos serão enriquecidos.
2. **Calcule e mostre o custo:** e-mail = 1 crédito/contato, telefone = 10 créditos/contato. Telefone pesa 10× o e-mail.
3. **Peça confirmação explícita do usuário** antes de chamar. O gasto é enfileirado de imediato — **não é simulação**.
4. Acompanhe o job (assíncrono).
5. Para enxergar o resultado: a lista default não tem essas colunas. Adicione colunas `source=record` com `record_field: "email"`/`"phone_number"` (skill `gerenciar-listas`), depois `list_records`.

## Reduzir o gasto

Use `filters` / `exclude_row_ids` para enriquecer só o subconjunto que importa. Se o usuário só precisa de e-mail, não inclua telefone (10× mais caro). Não há rota de saldo de créditos — o cliente confere no painel da Nuvia.
