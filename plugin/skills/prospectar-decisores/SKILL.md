---
name: prospectar-decisores
description: Encontra decisores e contatos-chave de uma empresa específica usando o MCP da Nuvia. Use quando o usuário quer achar o CEO, diretores, C-level ou qualquer decisor de uma empresa (por nome, domínio ou CNPJ), ou montar uma lista de decisores por recorte (cargo + setor + região). Aplica a ordem de custo correta (grátis primeiro), resolve a empresa, e pagina até encontrar.
---

# Prospectar decisores de uma empresa

Objetivo: identificar as pessoas certas (decisores) numa empresa, gastando crédito só quando necessário e sem deixar ninguém passar batido.

**Antes de começar, leia a skill `nuvia-fundamentos`** (ordem de custo, lookups, paginação e — importante — como falar sem expor termos internos da base).

## Passo 0 — De onde você está partindo?

- **Tem o domínio ou o CNPJ?** Comece pelo lado gratuito (próximo passo) para ancorar a empresa antes de gastar crédito.
- **Tem só o nome da empresa?** Para uma empresa brasileira, ache o CNPJ/domínio com `search_brazil_companies` (grátis). Para uma empresa global, use `search_businesses` (custa crédito) só se necessário.
- **É um recorte amplo** (ex.: "CMOs de fintechs em SP")? Vá direto à busca por recorte (Passo 3), resolvendo o cargo no catálogo antes.

## Passo 1 — Ancorar a empresa de graça (quando der)

Se você tem o domínio: `link_global_to_brazil(domain)` (grátis) devolve o cadastro BR e os domínios.
Se você tem o CNPJ: `search_brazil_companies(filters: { cnpjs: { values: ["<cnpj sem máscara>"] } }, page_size: 1)` — match exato, grátis, e já traz domínios e razão social que você vai reusar.

Guarde `domain` (de `dominios[]`) e a razão social — eles elevam muito o acerto na ponte para a base global.

## Passo 2 — Ponte para a base global (consome crédito)

```
link_brazil_to_global(cnpj, domain, company_name)  →  { business_id }
```

- **Sempre passe `domain` E `company_name`.** Sem o vínculo por CNPJ, o match cai para domínio e depois nome — informar os dois aumenta o acerto.
- `found: false`? Tente outro domínio de `dominios[]` antes de desistir. Sem o identificador da empresa não há como buscar os decisores.
- Avise o usuário: "vou fazer a ponte para a base global da Nuvia para localizar a empresa — esse vínculo consome créditos". (A busca dos decisores em si, no passo seguinte, é gratuita.)

## Passo 3 — Buscar os decisores (grátis)

```
search_prospects(
  filters = { business_id: { values: ["<id da empresa>"] },
              job_level:   { values: ["c-suite","vice president","director"] } },
  page = 1, page_size = 50   # use 100 em empresa grande
)
```

Regras que definem a qualidade:

1. **Sem filtro por nome.** Para achar uma pessoa específica: trave a empresa, estreite por `job_level`, **pagine até esgotar** (`total_results`) e varra o nome no resultado. Concluir "não está" sem esgotar é um falso negativo.
2. **Estreite por nível para reduzir o N.** Só `c-suite` derruba uma empresa grande de centenas para dezenas. Faltou alguém? Alargue progressivamente: `c-suite` → +`vice president`/`director` → +`partner`/`owner`.
3. `job_level` e `job_department` são **enumerados** (use direto). `job_title` é catálogo → resolva com `lookup_filter_values(entity_type="prospects", field="job_title")` antes de filtrar por cargo específico.
4. **Não use** "tem e-mail"/"tem telefone" — corta recall e não prevê o enrich. A busca só identifica.
5. **Teto de cobertura:** nem todo diretor tem perfil público. Se alguém não aparece nem alargando até `partner/owner`, é ausência real na base, não bug. Registre como "sem pegada digital".

## Passo 4 — Busca por recorte (sem empresa específica)

Para "decisores de tal cargo em tal setor/região": resolva `job_title` (catálogo, prospects) e a localização (`city_region_country`, ex. `"Sao Paulo, BR"`), combine com `job_level`, e refine até uma lista enxuta antes de paginar. A busca é gratuita, mas o objetivo é **qualidade, não volume**: uma lista de milhares de contatos pouco relevantes é um fracasso, não um sucesso — refine os filtros até o recorte ficar enxuto.

## Passo 5 — Entregar

Apresente uma tabela limpa: **Nome · Cargo · Nível · Localização · LinkedIn**. Para o LinkedIn, prefira o **handle legível** de `linkedin_url_array` (ex: `linkedin.com/in/nome-sobrenome`) — normalmente o segundo item; se o array só tiver um item, é ele mesmo. Evite a URN opaca (`linkedin.com/in/ACoAA…`). Não mostre identificadores internos. Se o usuário quiser trabalhar esses contatos dentro da Nuvia, encaminhe para a skill `salvar-na-nuvia`.

## Próximos passos comuns
- Cruzar quem é dono vs quem aparece publicamente → skill `investigar-empresa-br`.
- Materializar os contatos na Nuvia → skill `salvar-na-nuvia`.
