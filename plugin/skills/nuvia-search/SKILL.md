---
name: nuvia-search
description: "Busca e prospecção na Nuvia. Encontra empresas brasileiras na base de CNPJ (por CNPJ, setor/CNAE, UF/cidade, porte, situação cadastral, capital social ou sócio) e empresas na base global por firmográficos (setor/NAICS, porte, faturamento, localização, tech stack, intenção de compra); e localiza os decisores de uma empresa (CEO, C-level, diretores) por nome, domínio ou CNPJ, ou monta uma lista de decisores por recorte (cargo, setor, região). Use quando o usuário quer montar uma lista de empresas, achar uma empresa específica, descobrir quem decide numa conta, ou mapear a liderança de uma empresa — no Brasil ou fora."
---

# Buscar empresas e prospectar decisores

Cobre as três buscas da Nuvia: **empresas BR** (base de CNPJ, grátis), **empresas global** (firmográficos, paga) e **decisores** (pessoas).

**Antes de começar, leia a skill `nuvia` (Parte 1 — Fundamentos):** ordem de custo, lookups obrigatórios, paginação, curadoria e como falar sem expor termos internos da base. Os catálogos e mapeamentos ficam em `../nuvia/references/`.

> **Regra que vale para tudo aqui:** filtros de categoria/localização/cargo **não aceitam texto livre** — resolva o termo no catálogo (`lookup_*`) antes de buscar. E o objetivo é **curadoria, não volume**: refine até a lista ficar enxuta antes de paginar.

---

## A) Empresas brasileiras (base de CNPJ — 🆓 gratuita)

A base é gratuita e enorme. Cure o recorte com os filtros certos.

UF/CNAE/natureza/cidade/sócio só aceitam valores de catálogo: `lookup_brazil_filter_values(field, query)` com **uma palavra-chave limpa**.

⚠️ A chave do `filters` nem sempre é o nome do campo de catálogo. Mapeamentos confirmados (ex.: `brazil_cnae` → `cnae_principal`, `brazil_socio` → `nome_socio`) em `../nuvia/references/mapeamentos-br.md`. **Se um filtro "não filtrou" (total na casa dos milhões), a chave provavelmente está errada.** Campos enumerados (direto): `uf`, `porte`, `situacao_cadastral` (ver `../nuvia/references/catalogos-br.md`).

**Por CNPJ (match exato, ideal):**
```
search_brazil_companies(filters: { cnpjs: { values: ["<cnpj sem máscara>"] } }, page_size: 1)
```

**Por recorte (combine filtros para estreitar):**
```
search_brazil_companies(filters: {
  uf:                 { values: ["MG"] },
  cnae_principal:     { values: ["<value do lookup>"] },
  situacao_cadastral: { values: ["Ativa"] }
}, page_size: 50)
```
Combine UF + setor/CNAE + situação até a lista ficar enxuta. `filter_operator` é `and` por padrão.

**Evite `nome_empresa`:** é fuzzy e traz milhares. Use só como último recurso, combinado com UF/situação.

Casos especiais:
- **MEI:** não dá para filtrar antes da busca. Aproxime com `natureza_juridica = "Empresário (Individual)"` + `porte = "Microempresa - ME"`, e filtre `simples.mei_opcao == "Sim"` no resultado. Ver `mapeamentos-br.md`.
- **Geofiltro fino:** cidade respeita acento ("são paulo", não "sao paulo"); bairro é texto livre bagunçado — prefira `cep`. Ver `mapeamentos-br.md`.
- **Por sócio (pessoa → empresas):** use `nome_socio` (não `socio`), resolvendo em `brazil_socio` antes.

Entregar — tabela: **Empresa · CNPJ · Setor/CNAE · UF/Cidade · Situação · Porte**.

---

## B) Empresas na base global (firmográficos — 💳 CONSOME CRÉDITO)

> **Atenção:** `search_businesses` **consome créditos**. Avise o usuário antes de disparar, use `page_size` 10–20, **não repita buscas idênticas** e cure o recorte ao máximo antes de paginar. Para empresas brasileiras, prefira a busca BR (seção A, gratuita) sempre que possível.

Quando usar a base global:
- Prospecção de contas fora do Brasil.
- Obter firmográficos (porte, faturamento, setor, tech stack) e o identificador de uma empresa para depois buscar decisores.
- Cruzar/seguir tech stack ou intenção de compra.

Resolva os filtros antes: `lookup_filter_values(field, entity_type="businesses", query)`.
- Catálogo (precisa lookup): `naics_category`, `region_country_code`, `country`, `city_region_country`, `google_category`, `linkedin_category`, `company_tech_stack_tech`, `business_intent_topics`.
- Enumerado (direto): `company_size`, `company_revenue`, `company_age`, `number_of_locations`. Lista em `../nuvia/references/catalogos-global.md`.

```
search_businesses(filters: {
  naics_category:      { values: ["<value do lookup>"] },
  company_size:        { values: ["<faixa>"] },
  region_country_code: { values: ["us-ca"] }
}, page_size: 10)   # comece pequeno; pagine só se necessário
```

- **Intenção de compra / eventos:** `business_intent_topics {topics[], topic_intent_level}` e `events` ajudam a priorizar contas quentes.
- **Não use** "tem website/e-mail" como filtro de qualificação — ver `../nuvia/references/enriquecimento.md`.
- **NAICS round-trip:** o `naics` que volta numa empresa é o mesmo `value` do lookup `naics_category`. Pegue o de uma empresa-alvo conhecida para achar semelhantes.

Entregar — tabela: **Empresa · Domínio · Setor · Porte · Faturamento · Localização · LinkedIn**. Não exponha o identificador interno.

---

## C) Decisores de uma empresa (pessoas)

Objetivo: identificar as pessoas certas, gastando crédito só quando necessário e sem deixar ninguém passar batido.

**Passo 0 — De onde você parte?**
- **Tem domínio ou CNPJ?** Comece pelo lado gratuito (Passo 1) para ancorar a empresa antes de gastar crédito.
- **Tem só o nome?** Empresa BR: ache CNPJ/domínio com `search_brazil_companies` (grátis). Empresa global: `search_businesses` (custa) só se necessário.
- **Recorte amplo** ("CMOs de fintechs em SP")? Vá direto ao Passo 4, resolvendo o cargo no catálogo antes.

**Passo 1 — Ancorar a empresa de graça (quando der)**
- Tem o domínio: `link_global_to_brazil(domain)` (grátis) devolve cadastro BR e domínios.
- Tem o CNPJ: `search_brazil_companies(filters: { cnpjs: { values: ["<cnpj sem máscara>"] } }, page_size: 1)` — match exato, grátis, já traz domínios e razão social.

Guarde `domain` (de `dominios[]`) e a razão social — elevam muito o acerto na ponte para a base global.

**Passo 2 — Ponte para a base global (💳 consome crédito)**
```
link_brazil_to_global(cnpj, domain, company_name)  →  { business_id }
```
- **Sempre passe `domain` E `company_name`.** Sem o vínculo por CNPJ, o match cai para domínio e depois nome — informar os dois aumenta o acerto.
- `found: false`? Tente outro domínio de `dominios[]` antes de desistir. Sem o identificador da empresa não há como buscar decisores.
- Avise: "vou fazer a ponte para a base global da Nuvia para localizar a empresa — esse vínculo consome créditos". (A busca dos decisores em si é gratuita.)

**Passo 3 — Buscar os decisores (🆓 grátis)**
```
search_prospects(
  filters = { business_id: { values: ["<id da empresa>"] },
              job_level:   { values: ["c-suite","vice president","director"] } },
  page = 1, page_size = 50   # use 100 em empresa grande
)
```
Regras que definem a qualidade:
1. **Sem filtro por nome.** Para achar alguém específico: trave a empresa, estreite por `job_level`, **pagine até esgotar** (`total_results`) e varra o nome no resultado. Concluir "não está" sem esgotar é falso negativo.
2. **Estreite por nível para reduzir o N.** Só `c-suite` derruba uma empresa grande de centenas para dezenas. Faltou alguém? Alargue: `c-suite` → +`vice president`/`director` → +`partner`/`owner`.
3. `job_level` e `job_department` são **enumerados** (use direto). `job_title` é catálogo → resolva com `lookup_filter_values(entity_type="prospects", field="job_title")` antes.
4. **Não use** "tem e-mail"/"tem telefone" — corta recall e não prevê o enrich. A busca só identifica.
5. **Teto de cobertura:** nem todo diretor tem perfil público. Se alguém não aparece nem alargando até `partner/owner`, é ausência real na base, não bug. Registre como "sem pegada digital".

**Passo 4 — Busca por recorte (sem empresa específica)**
Para "decisores de tal cargo em tal setor/região": resolva `job_title` (catálogo, prospects) e a localização (`city_region_country`, ex. `"Sao Paulo, BR"`), combine com `job_level`, e refine até uma lista enxuta antes de paginar. A busca é gratuita, mas o objetivo é **qualidade, não volume**.

Entregar — tabela: **Nome · Cargo · Nível · Localização · LinkedIn**. Para o LinkedIn, prefira o **handle legível** de `linkedin_url_array` (ex.: `linkedin.com/in/nome-sobrenome`) — normalmente o segundo item; evite a URN opaca (`linkedin.com/in/ACoAA…`). Não mostre identificadores internos.

---

## Próximos passos comuns
- **Trazer os resultados para a Nuvia** (salvar como Pessoas/Empresas, opcionalmente numa Lista) → skill `nuvia`.
- **Enriquecer e-mail/telefone** dos contatos salvos → skill `nuvia` (consome crédito; leia os cuidados).
- **Entender quem é dono vs quem aparece publicamente** numa empresa BR → skill `nuvia-deep`.
