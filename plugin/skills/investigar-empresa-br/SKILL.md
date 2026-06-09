---
name: investigar-empresa-br
description: Investigação societária de empresa brasileira. Cruza o quadro de sócios e administradores (dados públicos da Receita) com a liderança que aparece publicamente, distinguindo LTDA (sócios = donos) de S.A. (sócios = diretoria). Use quando o usuário quer entender quem realmente manda numa empresa, mapear donos versus executivos, seguir holdings e grupo econômico, ou qualificar uma conta antes de abordar.
---

# Investigar empresa BR (sócios × identidade digital)

Cruzar o **quadro de sócios e administradores** (quem é sócio/administrador no papel, via base de CNPJ da Receita) com a **identidade digital** da empresa (quem aparece e atua publicamente). O cruzamento revela a estrutura real de poder, equity e decisão — e os buracos entre uma coisa e outra.

**Antes de começar, leia a skill `nuvia-fundamentos`** (ordem de custo, lookups, e como falar sem expor termos internos da base). Faça **todo o trabalho gratuito primeiro**; o único passo pago é a ponte para a base global (Fase 2) — gaste crédito só ali, quando for realmente cruzar com o lado digital.

## A leitura que muda tudo: LTDA vs S.A.

A natureza jurídica define **o que o quadro de sócios significa**. Leia-a **antes de qualquer coisa**:

- **LTDA** → o quadro de sócios mostra os **donos reais** (+ administradores). Mire o dono que também atua publicamente.
- **S.A.** → o quadro mostra **só a diretoria estatutária**. O dono/controlador (fundos, holdings, listco) **não aparece** — está no nível societário acima, normalmente fora do Brasil. A entrada comercial é o executivo (diretor estatutário = decisor real e abordável).

## Fluxo em 3 fases

**Fase 1 — Lado jurídico (grátis).** Tendo o CNPJ:
```
search_brazil_companies(filters: { cnpjs: { values: ["<cnpj>"] } }, page_size: 1)
```
Sem o CNPJ, ancore pelo domínio (grátis): `link_global_to_brazil(domain)`. Use o domínio institucional/histórico; se falhar, tente `.com.br`/raiz do site. Essa rota já devolve o cadastro inteiro com os sócios.
Leia `natureza_juridica`, `socios[]` (com PF e PJ), `dominios[]`.

**Fase 2 — Ponte para o global (consome crédito 💳), só quando for cruzar.** Este é o **único passo pago** do fluxo até aqui — confirme com o usuário antes de disparar.
```
link_brazil_to_global(cnpj, domain, company_name)  →  { business_id }
```
Sempre passe `domain` E `company_name`. `found:false` → tente outro domínio. Sem o identificador da empresa, não há fase digital.

**Fase 3 — Lado digital (grátis).**
```
search_prospects(filters: { business_id, job_level: ["c-suite","vice president","director"] }, page_size: 50/100)
```
Pagine até esgotar; alargue o nível progressivamente se faltar alguém. A busca é gratuita — o custo só volta no **enriquecimento** (`enrich_list`, abrir e-mail/telefone), que é um passo separado e à parte.

## Cruzar e classificar

Para cada nome, cruze presença no quadro de sócios × presença no digital e rotule em buckets (🟢 dono+digital, 🔵 dono silencioso, 🟡 executivo contratado, 🔗 sócio PJ/holding, ⚪ colaborador). O matching é heurístico por nome (não há chave comum entre as bases).

**Estes detalhes estão em `references/playbook-qsa-digital.md` — leia antes de cruzar:** a tabela completa de buckets, a árvore de decisão, as regras de normalização e níveis de confiança do match, os ruídos a descartar (ex-funcionários, inflação de senioridade, homônimos), e o teto de cobertura.

## Seguir o grupo econômico

Se houver **sócio PJ** no quadro (bucket 🔗), é um gancho de grupo: pegue o CNPJ da holding e rode `search_brazil_companies` nele para seguir a cadeia de controle e outras empresas do grupo.

## Buscar por uma pessoa (nome → empresas)

Para mapear a rede societária de alguém (todas as empresas em que consta como sócio): use `nome_socio` (não `socio`!), resolvendo o nome em `brazil_socio` antes. Detalhes em `nuvia-fundamentos/references/mapeamentos-br.md`.

## Entregar — visão unificada (sócios × digital)

Entregue uma **visão unificada**: um *outer join* de quadro de sócios (CNPJ) × contatos digitais (LinkedIn), cruzado por nome. **Ninguém é descartado:**
- sócio que **deu match** no digital → uma linha unificada com cargo digital + LinkedIn;
- sócio **sem pegada digital** → linha mesmo assim, com o LinkedIn em branco (bucket 🔵);
- contato digital que **não é sócio** → continua listado (🟡 executivo / ⚪ colaborador).

Tabela: **Nome · Papel no quadro de sócios · Cargo digital · LinkedIn · Bucket · Confiança do match**.

Sempre inclua os links (ver `nuvia-fundamentos`): o **site** e o **LinkedIn da empresa** no cabeçalho da empresa, e o **LinkedIn de cada contato** na linha dele — preferindo o handle legível do `linkedin_url_array` (não a URN `ACoAA…`).

Mais um parágrafo de leitura: quem é o alvo nº 1, onde está o controle, e (se S.A.) o aviso de que a decisão de compra pode subir para o controlador.
