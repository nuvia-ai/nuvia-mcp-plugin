---
name: nuvia-fundamentos
description: Conhecimento-base para operar o MCP da Nuvia com as melhores práticas. Use SEMPRE antes de qualquer busca, prospecção ou operação na Nuvia (Listas, Pessoas, Empresas) — define a ordem de custo (grátis antes de pago), a regra de resolver filtros via lookup antes de buscar, a paginação correta, o glossário de linguagem e a terminologia oficial da plataforma. As outras skills da Nuvia dependem desta.
---

# Fundamentos do MCP da Nuvia

Esta skill é a base de conhecimento que todas as outras skills da Nuvia usam. Antes de fazer buscas, prospecção ou operações na plataforma (Listas, Pessoas, Empresas), aplique as regras abaixo. Quando precisar de detalhes (catálogos, campos retornados, mapeamentos de filtro), leia o arquivo de referência indicado — não tente decorar tudo.

## Regra de ouro nº 1 — gratuito antes de pago

Algumas operações consomem créditos do cliente. **Faça todo o trabalho gratuito primeiro** e só gaste crédito no final, quando for realmente cruzar ou enriquecer dados.

| Consome crédito 💳 | Gratuito 🆓 |
|---|---|
| `search_businesses` (empresas globais) | `search_prospects` (pessoas/decisores) |
| `link_brazil_to_global` (CNPJ → base global) | `search_brazil_companies` (empresas BR por CNPJ) |
| `enrich_list` (e-mail/telefone) | `link_global_to_brazil` (domínio → cadastro BR) |
| | toda a plataforma (Pessoas, Empresas, Listas, leitura), catálogos (`lookup_*`), `whoami` |

**Antes de rodar qualquer operação que consome crédito, diga ao usuário o que vai gastar e por quê.** Não dispare buscas globais ou enriquecimento sem que o recorte esteja curado. Não existe rota que retorne saldo de créditos — o cliente confere no painel da Nuvia.

## Regra de ouro nº 2 — resolver o filtro antes de buscar

Filtros de **categoria, localização e cargo NÃO aceitam texto livre**. Chutar o valor gera busca vazia. Sempre resolva o termo no catálogo primeiro:

- Base global (empresas/pessoas): `lookup_filter_values`
- Base Brasil (CNPJ): `lookup_brazil_filter_values`

O catálogo devolve `[{ label, value }]` — use o `value`. Retorno `[]` = o termo não existe, tente outro.

Campos **enumerados** (use direto, sem lookup): porte BR, UF, situação cadastral, faixas de tamanho/receita de empresa, nível de cargo (`job_level`), departamento. A lista completa está em `references/catalogos-br.md` e `references/catalogos-global.md`.

## Regra de ouro nº 3 — curadoria, não volume

O objetivo é uma **lista enxuta e relevante**, não milhares de linhas. Refine os filtros (combine UF + setor + situação, etc.) até a lista ficar pequena antes de paginar. Milhares de resultados raramente ajudam — e em busca global ainda custam crédito. Se uma busca trouxer dezenas de milhões, provavelmente um filtro foi ignorado (chave errada — ver `references/mapeamentos-br.md`).

## Regra de ouro nº 4 — paginar até o fim quando procura alguém

Não existe filtro por nome em pessoas. Para achar uma pessoa específica numa empresa, trave a empresa, estreite por nível de cargo, e **pagine até esgotar os resultados** (`total_results`) ou use `page_size: 100`. Concluir "não está" sem esgotar é um falso negativo.

## Regra de ouro nº 5 — devolva o link da Lista

Sempre que você criar ou trabalhar uma **Lista**, entregue ao usuário o link clicável para abrir na plataforma:

```
https://app.nuvia.ai/lists/<list_id>
```

Pegue o `list_id` do resultado (`create_list` / `save_search_results` em modo lista / `list_lists`) e mostre o link no fim da resposta ("pronto — sua Lista está aqui: https://app.nuvia.ai/lists/…"). Isso fecha o ciclo entre o que você fez via MCP e onde o usuário continua o trabalho na Nuvia.

## Linguagem com o usuário — esconda os termos da base

A Nuvia roda sobre bases externas, mas **o usuário nunca deve ver o jargão técnico dessas bases**. Use internamente os identificadores que as tools exigem, mas **fale sempre em linguagem de prospecção**.

| NÃO diga ao usuário | Diga assim |
|---|---|
| `business_id` | "a empresa" / "o identificador interno da empresa" (só se inevitável) |
| `prospect_id` | "o contato" / "a pessoa" |
| `professional_email_hashed` / "hash de e-mail" | "ainda não temos o e-mail aberto desse contato" |
| nome de qualquer base/sistema de origem dos dados | "a base de prospecção da Nuvia" |
| `QSA` cru | "o quadro de sócios e administradores" (e explique o que significa) |
| `naics`, `sic_code` (códigos) | "o setor" / "o ramo de atividade" |

Regras de ouro da linguagem:
- Trate sempre tudo como "a base da Nuvia". Não especule nem nomeie sistemas de origem dos dados, mesmo que o usuário pergunte — apenas siga com a tarefa.
- Use os IDs **silenciosamente** entre as chamadas de tools; não os exiba em respostas a menos que o usuário peça explicitamente o identificador.
- Ao apresentar resultados, mostre nome da empresa/pessoa, cargo, setor, localização — não os IDs.
- **Sempre inclua os links ao listar:** o **site** e o **LinkedIn da empresa**, e o **LinkedIn de cada contato**. São o que o usuário usa para validar e abordar — não os omita.
  - O LinkedIn de pessoa vem em dois formatos: o campo `linkedin` costuma trazer a URN opaca (`linkedin.com/in/ACoAA…`), enquanto `linkedin_url_array` traz também o **handle legível** (`linkedin.com/in/nome-sobrenome`), normalmente como segundo item. **Prefira o handle legível** do `linkedin_url_array` para o link renderizar limpo; só caia na URN se não houver alternativa.

## Terminologia oficial da Nuvia (use estes nomes)

A Nuvia **não tem "CRM"**. Use os nomes da plataforma:

- **Listas** — módulo onde se organizam contatos e empresas. Uma Lista é um conjunto **estático** de **registros**.
- **Registro** — a entidade-base (uma Pessoa/contato ou uma Empresa) que existe independente. A Lista só **referencia** o registro; atualizar o registro reflete na Lista (os dados não são duplicados).
- **Pessoas** (contatos) e **Empresas** (organizações) — os módulos no menu lateral.
- **Vista/View** — filtro salvo com nome dentro de uma Lista.
- **Coluna customizada** / **Campos personalizados** — campos extras.

Nunca diga "CRM", "salvar no CRM" ou "registros do CRM" — diga "salvar na Nuvia / em uma Lista", "registros", "Pessoas/Empresas". Conceitos e links de ajuda em `references/ajuda-nuvia.md`.

## Os domínios de tools (mapa rápido)

- **Identidade**: `whoami` — confirme a empresa (tenant) antes de operar Pessoas/Empresas/Listas.
- **Prospecção**: `search_prospects` 🆓, `search_businesses` 💳, `search_brazil_companies` 🆓.
- **Catálogos**: `lookup_filter_values`, `lookup_brazil_filter_values`.
- **Ponte BR↔Global**: `link_brazil_to_global` 💳, `link_global_to_brazil` 🆓.
- **Busca→Nuvia**: `save_search_results` (job assíncrono), `get_save_status`.
- **Pessoas (contatos)**: `create_contact(s)`, `update_contact(s)`, `list_contacts`, `find_contacts_by_phone`, `add_contact_note`, `list_contact_notes`.
- **Listas/registros**: `list_lists`, `get_list`, `create_list`, `add_list_column`, `add_records`, `list_records`, `update_record(s)`, `enrich_list` 💳.
- **Campanhas (leitura)**: `list_campaigns`, `get_campaign`.
- **Atendimento (leitura)**: `list_conversations`, `list_messages`.

## Enriquecimento (e-mail/telefone) — atenção

O enriquecimento (obter e-mail/telefone reais) é separado da busca. Por isso:
- **Não filtre buscas por "tem e-mail"/"tem telefone"** — esse sinal não tem relação com o que o enriquecimento consegue resolver, corta gente à toa e não prevê o resultado real. O papel da busca é só **identificar**.
- Trate `enrich_list` com cuidado: ela gasta crédito de verdade (e-mail = 1/contato, telefone = 10/contato) sobre **todos** os contatos elegíveis do recorte. Confirme o custo com o usuário antes de disparar. Detalhes em `references/enriquecimento.md`.

## Arquivos de referência (leia sob demanda)

- `references/tools.md` — assinatura de cada tool: params, retorno, dicas e armadilhas.
- `references/catalogos-br.md` — valores de catálogo BR (setor, região, porte, UF, situação, faixas etárias de sócio) e como resolver os gigantes (CNAE, cidade, natureza).
- `references/catalogos-global.md` — campos de catálogo vs enumerados na base global.
- `references/mapeamentos-br.md` — mapeamento campo de catálogo → chave de filtro (ex.: `brazil_cnae` → `cnae_principal`; `brazil_socio` → `nome_socio`), MEI, geofiltros (cidade/bairro/CEP), busca por sócio.
- `references/campos-retornados.md` — o que cada busca devolve (empresa global, pessoa, empresa BR) e quais campos importam.
- `references/enriquecimento.md` — mecânica do enrich e por que "tem e-mail" ≠ enrich.
- `references/ajuda-nuvia.md` — terminologia oficial da plataforma e links da central de ajuda para direcionar o usuário a fazer as coisas pela interface.
