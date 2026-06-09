---
name: pesquisar-empresas-br
description: "Pesquisa empresas brasileiras na base nacional de CNPJ (Receita Federal) por CNPJ, setor/CNAE, UF/cidade, porte, situação cadastral, capital social ou sócio. Gratuito. Use quando o usuário quer montar uma lista de empresas BR por recorte (ex.: empresas de energia ativas em MG, MEIs de tecnologia em SP), achar uma empresa por CNPJ, ou listar empresas de um sócio. Sempre resolve os filtros no catálogo antes de buscar."
---

# Pesquisar empresas brasileiras

A base de CNPJ é gratuita e enorme. O jogo é **curar o recorte** com os filtros certos até uma lista relevante — não trazer milhões de linhas.

**Antes de começar, leia a skill `nuvia-fundamentos`** (lookups obrigatórios, curadoria, linguagem).

## Regra central: resolva o filtro antes de buscar

UF/CNAE/natureza/cidade/sócio só aceitam valores de catálogo. Chutar gera busca vazia. Use `lookup_brazil_filter_values(field, query)` com **uma palavra-chave limpa**.

⚠️ A chave do `filters` nem sempre é o nome do campo de catálogo. Os mapeamentos confirmados (ex.: `brazil_cnae` → `cnae_principal`, `brazil_socio` → `nome_socio`) estão em `nuvia-fundamentos/references/mapeamentos-br.md`. **Se um filtro "não filtrou" (total na casa dos milhões), a chave provavelmente está errada.**

Campos enumerados (use direto): `uf`, `porte`, `situacao_cadastral`. Lista em `nuvia-fundamentos/references/catalogos-br.md`.

## Padrões de busca

**Por CNPJ (match exato, ideal):**
```
search_brazil_companies(filters: { cnpjs: { values: ["<cnpj sem máscara>"] } }, page_size: 1)
```

**Por recorte (combine filtros para estreitar):**
```
search_brazil_companies(filters: {
  uf: { values: ["MG"] },
  cnae_principal: { values: ["<value do lookup>"] },
  situacao_cadastral: { values: ["Ativa"] }
}, page_size: 50)
```
Combine UF + setor/CNAE + situação até a lista ficar enxuta. `filter_operator` é `and` por padrão.

**Evite `nome_empresa`:** é fuzzy e traz milhares (a empresa certa some no meio). Use só como último recurso, combinado com UF/situação.

## Casos especiais

- **MEI:** não dá para filtrar antes da busca. Aproxime com `natureza_juridica = "Empresário (Individual)"` + `porte = "Microempresa - ME"`, e filtre `simples.mei_opcao == "Sim"` no resultado. Ver `mapeamentos-br.md`.
- **Geofiltro fino:** cidade respeita acento ("são paulo", não "sao paulo"); bairro é texto livre bagunçado — prefira `cep`. Ver `mapeamentos-br.md`.
- **Por sócio (pessoa → empresas):** use `nome_socio` (não `socio`), resolvendo em `brazil_socio` antes.

## Entregar

Tabela: **Empresa · CNPJ · Setor/CNAE · UF/Cidade · Situação · Porte**. Se o usuário quiser prospectar decisores dessas empresas, encaminhe para `prospectar-decisores` ou `investigar-empresa-br`. Para trazer à Nuvia, use `salvar-na-nuvia`.
