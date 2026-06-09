---
name: pesquisar-empresas-global
description: Pesquisa empresas na base global por firmográficos — setor/NAICS, porte, faturamento, localização, tech stack, intenção de compra. Consome créditos. Use quando o usuário quer prospectar contas com foco global/fora do Brasil, ou obter os firmográficos e o identificador de uma empresa. Resolve categorias no catálogo antes de buscar e cura o recorte para não desperdiçar crédito.
---

# Pesquisar empresas na base global

Esta busca **consome créditos**. Cure o recorte ao máximo antes de paginar — milhares de resultados não ajudam e custam.

**Antes de começar, leia a skill `nuvia-fundamentos`** (ordem de custo, lookups, linguagem). Para empresas brasileiras, prefira a skill `pesquisar-empresas-br` (gratuita) sempre que possível.

## Quando usar a base global

- Prospecção de contas fora do Brasil.
- Obter firmográficos (porte, faturamento, setor, tech stack) e o identificador de uma empresa para depois buscar decisores.
- Cruzar/seguir tech stack ou intenção de compra.

## Resolva os filtros antes de buscar

Categorias e localização só aceitam catálogo: `lookup_filter_values(field, entity_type="businesses", query)`.
- Catálogo (precisa lookup): `naics_category`, `region_country_code`, `country`, `city_region_country`, `google_category`, `linkedin_category`, `company_tech_stack_tech`, `business_intent_topics`.
- Enumerado (direto): `company_size`, `company_revenue`, `company_age`, `number_of_locations`.

Lista completa em `nuvia-fundamentos/references/catalogos-global.md`.

## Padrão de busca

```
search_businesses(filters: {
  naics_category: { values: ["<value do lookup>"] },
  company_size:   { values: ["<faixa>"] },
  region_country_code: { values: ["us-ca"] }
}, page_size: 10)   # comece pequeno; pagine só se necessário
```

- Avise o usuário que a busca consome créditos antes de disparar.
- Use `page_size` 10–20 e não repita buscas idênticas.
- **Intenção de compra / eventos:** `business_intent_topics {topics[], topic_intent_level}` e `events` ajudam a priorizar contas quentes.
- **Não use** "tem website/e-mail" como filtro de qualificação de contato — ver `nuvia-fundamentos/references/enriquecimento.md`.

## NAICS round-trip

O `naics` (código) que volta numa empresa é o mesmo `value` do lookup `naics_category`. Pegue o de uma empresa-alvo conhecida e use para achar semelhantes.

## Entregar

Tabela: **Empresa · Domínio · Setor · Porte · Faturamento · Localização · LinkedIn**. Não exponha o identificador interno. Para achar os decisores dessa empresa, encaminhe para `prospectar-decisores` (você já tem o identificador). Para trazer à Nuvia, use `salvar-na-nuvia`.
