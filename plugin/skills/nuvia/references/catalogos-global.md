# Catálogos da base global (`lookup_filter_values`)

`lookup_filter_values(field, entity_type, query)` — `entity_type` é `businesses` (default) ou `prospects`.
Retorna `[{ label, value }]`. Use o `value`. `[]` = termo inexistente.

## Campos de CATÁLOGO (precisam de lookup antes de filtrar)

- `naics_category` (→ código NAICS)
- `region_country_code` (ex.: `us-ca`)
- `country` (ISO-2)
- `job_title` (entity_type=prospects)
- `city_region_country` (cobre o Brasil também)
- `city_region` (predominantemente EUA)
- `google_category`
- `linkedin_category`
- `company_tech_stack_tech`
- `business_intent_topics`

## Campos ENUMERADOS (use direto, SEM lookup)

- `company_size` (faixas de nº de funcionários)
- `company_revenue` (faixas de faturamento)
- `job_level` — owner · c-suite · vice president · director · manager · partner · senior non-managerial · non-managerial · junior
- `job_department`
- `company_age`
- `number_of_locations`

## Dicas

- **NAICS round-trip:** o `naics` (int) que volta no resultado de uma busca é o mesmo `value` de `lookup_filter_values(field=naics_category)`. Útil para achar empresas semelhantes a uma que você já encontrou.
- **Tamanho/receita na mesma régua:** `number_of_employees_range` e `yearly_revenue_range` retornados usam as mesmas faixas dos filtros `company_size`/`company_revenue` — dá para ler e filtrar com o mesmo vocabulário.
