---
name: nuvia-search
description: "Busca e prospecção na Nuvia. Encontra empresas brasileiras na base de CNPJ (por CNPJ, setor/CNAE, UF/cidade, porte, situação cadastral, capital social ou sócio) e empresas na base global por firmográficos (setor/NAICS, porte, faturamento, localização, tech stack, intenção de compra); e localiza os decisores de uma empresa (CEO, C-level, diretores) por nome, domínio ou CNPJ, ou monta uma lista de decisores por recorte (cargo, setor, região). Use quando o usuário quer montar uma lista de empresas, achar uma empresa específica, descobrir quem decide numa conta, ou mapear a liderança de uma empresa — no Brasil ou fora."
---

# Buscar empresas e prospectar decisores

Cobre as três buscas da Nuvia: **empresas BR** (base de CNPJ), **empresas global** (firmográficos) e **decisores** (pessoas). **Todas as buscas são gratuitas** — buscar e cruzar nunca consome crédito; o único custo na Nuvia é o enriquecimento de contato (skill `nuvia`).

**Antes de começar, leia a skill `nuvia` (Parte 1 — Fundamentos):** lookups obrigatórios, paginação, curadoria e como falar sem expor termos internos da base. Os catálogos e mapeamentos ficam em `../nuvia/references/`.

> **Regra que vale para tudo aqui:** filtros de categoria/localização/cargo **não aceitam texto livre** — resolva o termo no catálogo (`lookup_*`) antes de buscar. E o objetivo é **curadoria, não volume**: refine até a lista ficar enxuta antes de paginar.

## Qual base usar? Decida pelo filtro decisivo (não pela geografia)

O erro comum é escolher a base pelo país. Decida pelo **filtro que define o recorte** — e só **força** a global o filtro que **não tem equivalente na BR**. Um filtro que existe nas duas bases (localização, setor, receita) nunca força a global: para alvo brasileiro, fique na BR e use a global só se o decisivo for exclusivo dela.

| O filtro decisivo é… | Base | Campos |
|---|---|---|
| Cadastral / societário | **BR** (seção A) | CNPJ, situação cadastral, sócio, capital social, natureza jurídica |
| **Exclusivos da global** | **Global** (seção B) | nº de funcionários (`company_size`), tech stack (`company_tech_stack_tech`), intenção de compra (`business_intent_topics`), eventos (`events`) |
| Existe nas duas (não força a base) | a que o resto do recorte indicar | setor (CNAE↔NAICS), receita (`porte`↔`company_revenue`), localização — só muda a granularidade |
| Precisa dos dois (ex.: empresas BR *e* seu headcount) | comece numa, faça a **ponte** | `link_brazil_to_global` / `link_global_to_brazil` (seção C, passo 2) |

⚠️ **Quando o filtro decisivo é só da global e o alvo é o Brasil, sinalize o tradeoff — não troque calado.** A cobertura BR da base global é uma **fração** da base de CNPJ (validado: Rio tem ~15 mil empresas grandes na global vs +1 milhão de CNPJs ativos) — e fica mais rasa quanto menor a presença digital/internacional da empresa. Diga em uma linha e ofereça a alternativa:

> "Nº de funcionários só existe na base global, cuja cobertura BR é parcial — empresas sem presença digital forte podem faltar. Alternativa: base de CNPJ por `porte` (régua de **receita**, não de funcionários), cobertura quase completa, mas sem filtro fino por headcount."

Escolha a base que o filtro decisivo exige, sinalize, siga a entrega.

---

## A) Empresas brasileiras (base de CNPJ — 🆓 gratuita)

A base é gratuita e enorme. Cure o recorte com os filtros certos.

⚠️ **Não há filtro por nº de funcionários aqui.** O `porte` da Receita (`Microempresa - ME` · `Empresa de pequeno porte - EPP` · `Demais empresas`) é régua de **receita/faturamento**, não de headcount. Recorte por "+N funcionários" só na base global (seção B) — pesando o tradeoff de cobertura do bloco "Qual base usar?".

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

## B) Empresas na base global (firmográficos — 🆓 gratuita)

> A busca global é **gratuita**. Ainda assim, comece com `page_size` 10–20 e cure o recorte — qualidade vence volume. Para alvos brasileiros, use a global só quando o **filtro decisivo** for dela (ver bloco "Qual base usar?") — e sinalize o tradeoff de cobertura.

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

- **Nº de funcionários é por FAIXA, não número exato.** `company_size`: `1-10 · 11-50 · 51-200 · 201-500 · 501-1000 · 1001-5000 · 5001-10000 · 10001+`. Para "+N", pegue **todas** as faixas acima do corte. A faixa que *cruza* o corte é **parcial** — ex.: "+100" → use `51-200` e acima, mas o `51-200` mistura empresas de 51 a 99 (que não batem) com 100 a 200 (que batem). **Sinalize na entrega quem caiu na faixa de fronteira: precisa de conferência manual.**
- **Localização global é ruidosa.** `city_region_country` pode trazer matches fora do lugar (ex.: filtrar "Rio De Janeiro, BR" pode retornar uma empresa sediada nos EUA). **Confira `city_name`/`country_name` no resultado** antes de entregar — não confie só no filtro.
- **Intenção de compra / eventos:** `business_intent_topics {topics[], topic_intent_level}` e `events` ajudam a priorizar contas quentes.
- **Não use** "tem website/e-mail" como filtro de qualificação — ver `../nuvia/references/enriquecimento.md`.
- **NAICS round-trip:** o `naics` que volta numa empresa é o mesmo `value` do lookup `naics_category`. Pegue o de uma empresa-alvo conhecida para achar semelhantes.

Entregar — tabela: **Empresa · Domínio · Setor · Porte · Faturamento · Localização · LinkedIn**. Não exponha o identificador interno.

---

## C) Decisores de uma empresa (pessoas)

Objetivo: identificar as pessoas certas, sem deixar ninguém passar batido. Todo o fluxo (ancorar, ponte, buscar decisores) é **gratuito**.

**Passo 0 — De onde você parte?**
- **Tem domínio ou CNPJ?** Ancore a empresa pelo cadastro brasileiro (Passo 1).
- **Tem só o nome?** Empresa BR: ache CNPJ/domínio com `search_brazil_companies`. Empresa global: `search_businesses`.
- **Recorte amplo** ("CMOs de fintechs em SP")? Vá direto ao Passo 4, resolvendo o cargo no catálogo antes.

**Passo 1 — Ancorar a empresa (quando der)**
- Tem o domínio: `link_global_to_brazil(domain)` devolve cadastro BR e domínios.
- Tem o CNPJ: `search_brazil_companies(filters: { cnpjs: { values: ["<cnpj sem máscara>"] } }, page_size: 1)` — match exato, já traz domínios e razão social.

Guarde `domain` (de `dominios[]`) e a razão social — elevam muito o acerto na ponte para a base global.

**Passo 2 — Ponte para a base global**
```
link_brazil_to_global(cnpj, domain, company_name)  →  { business_id }
```
- **Sempre passe `domain` E `company_name`.** Sem o vínculo por CNPJ, o match cai para domínio e depois nome — informar os dois aumenta o acerto.
- `found: false`? Tente outro domínio de `dominios[]` antes de desistir. Sem o identificador da empresa não há como buscar decisores.

**Passo 3 — Buscar os decisores**
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
