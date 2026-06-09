# Campos retornados pelas buscas

Envelope comum das buscas: `{ data: [...], total_results, page }`. Pagine enquanto `data` tiver o tamanho do `page_size`; `total_results` é o teto.

> **Lembre-se da linguagem:** estes nomes de campo são internos. Ao falar com o usuário, traduza (`business_id` → "a empresa"; `professional_email_hashed` → "ainda não temos o e-mail aberto"). Nunca exponha os IDs sem o usuário pedir.

## Empresa global (`search_businesses` / `link_brazil_to_global.company_data`)

| Campo | Observação |
|---|---|
| `business_id` | Identificador global da empresa. É o que entra em `search_prospects`. **Não mostre ao usuário.** |
| `name` | Nome comercial (não é a razão social do RFB). |
| `domain` | Domínio canônico. |
| `country_name` / `city_name` / `region` | Minúsculo; cidade/região podem vir `null`. |
| `number_of_employees_range` | Faixa (mesma de `company_size`). |
| `yearly_revenue_range` | Faixa (mesma de `company_revenue`). |
| `business_description` | Bio livre. |
| `naics` / `naics_description` | Código + descrição do setor. |
| `linkedin_profile` | Página da empresa no LinkedIn. |

**NÃO vem aqui** (é base BR): CNPJ, razão social, sócios, CNAE, capital social, endereço, situação cadastral. Isso só em `search_brazil_companies` / `link_global_to_brazil`.

## Pessoa / prospect (`search_prospects`)

| Campo | Observação |
|---|---|
| `prospect_id` | Identificador do perfil. **Não mostre.** |
| `professional_email_hashed` | Só **hash**, nunca o e-mail aberto. Pode `null`. E-mail real vem do enrich externo. |
| `full_name` (e `first_name`/`last_name`) | Nome de exibição; é o que se casa com o quadro de sócios. |
| `country_name` / `region_name` / `city` | Localização (minúsculo). |
| `linkedin_url_array` | `[id interno, slug público]` — use o slug `[1]` para abordar. |
| `experience` / `skills` | Histórico de cargos / skills (podem `null`). |
| `company_name` / `company_website` / `company_linkedin` | Empresa atual. |
| `business_id` | Mesmo identificador da empresa — é o elo. |
| `job_title` | Cargo atual (texto livre, pode ser grande). |
| `job_level_array` / `job_level_main` | Nível normalizado — casa com o filtro `job_level`. |
| `job_department_array` / `job_department_main` | Departamento normalizado. |
| `job_seniority_level` | Vocabulário diferente do `job_level` (cxo/vp/senior). Para filtrar use `job_level`; para ler, ambos ajudam. |

## Empresa BR (`search_brazil_companies` / `link_global_to_brazil.company_data`)

`cnpj`, `natureza_juridica` (código + nome), `socios[]` (quadro de sócios e administradores), `dominios[]`, `linkedin_url`, CNAE principal e secundários, endereço, `situacao_cadastral`, `capital_social`, bloco `simples` (flag MEI — ver `mapeamentos-br.md`).

`socios[]` = `{ nome, cnpj_cpf (CPF mascarado), faixa_etaria, qualificacao (Sócio/Administrador/Diretor…), entrada_sociedade_data, identificador (PF/PJ) }`.

## Defensividade

`null` é comum: cidade/região (empresa), skills/cargo/hash de e-mail (pessoa). Trate sempre como possivelmente ausente.
