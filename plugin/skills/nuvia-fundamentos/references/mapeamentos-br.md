# Mapeamentos, MEI, geofiltros e busca por sócio (base BR)

## Campo de catálogo → chave no `filters`

A chave usada em `search_brazil_companies` **nem sempre** é o nome do campo de catálogo sem o prefixo `brazil_`. Se um filtro "não filtrou" (total estourou para dezenas de milhões), a chave provavelmente está errada e foi ignorada.

| Campo de catálogo (`lookup_brazil_filter_values`) | Chave no `filters` |
|---|---|
| `brazil_cnae` | `cnae_principal` |
| `brazil_city` | `municipio` |
| `brazil_socio` | `nome_socio` ⚠️ (NÃO `socio`) |
| `brazil_bairro` | `bairro` |
| `brazil_cep` | `cep` |
| `brazil_uf` | `uf` |
| `brazil_porte` | `porte` |
| `brazil_situacao_cadastral` | `situacao_cadastral` |
| `brazil_natureza_juridica` | `natureza_juridica` |
| `brazil_setor` | `setor_economico` |

## MEI — não é filtro, é flag de saída

Não existe `porte = MEI`. **Não dá para filtrar MEI antes da busca.** Cada empresa retornada traz o bloco `simples`:

```json
"simples": { "mei_opcao": "Sim"|"Não"|null, "mei_opcao_data": <ts>, "mei_exclusao_data": <ts|null>, "simples_opcao": "Sim"|"Não"|null }
```

- **Aproximação (pré-busca):** `natureza_juridica = "Empresário (Individual)"` + `porte = "Microempresa - ME"` — pega o universo certo, mas inclui empresários individuais não-MEI.
- **Exato (pós-busca):** traga o lote e filtre por `simples.mei_opcao == "Sim"` no seu lado. 100% preciso.

## Geofiltros (formato `{values:[...]}`)

| Chave | Catálogo p/ resolver | Observação |
|---|---|---|
| `municipio` | `brazil_city` | ⚠️ respeite o acento — `"sao paulo"` → vazio; `"são paulo"` → 6,2M |
| `bairro` | `brazil_bairro` | texto livre e bagunçado (variações, typos) |
| `cep` | `brazil_cep` | mais confiável que bairro para recorte fino |
| `uf` | enumerado | siglas maiúsculas |

⚠️ **Bairro** é texto livre na Receita: para "Itaim Bibi" existem `ITAIM BIBI`, ` ITAIM BIBI` (espaço inicial), `VILA ITAIM BIBI`, typos, etc. Para boa cobertura, inclua os principais variantes em `values` **ou** use `cep`.

## Busca por sócio (nome de pessoa → empresas)

Busca todas as empresas em que uma pessoa consta no quadro de sócios e administradores (dado público da Receita).

- **Chave:** `nome_socio` (`{values:["NOME COMPLETO"]}`). ⚠️ É `nome_socio`, NÃO `socio` (passar `socio` é ignorado em silêncio e retorna a base inteira).
- **Catálogo:** `brazil_socio`.

**Fluxo:**
1. `lookup_brazil_filter_values(brazil_socio, query="sobrenome forte")` → confirma o nome exato.
2. `search_brazil_companies(filters: { nome_socio: { values: ["NOME COMPLETO"] } })`.

Cada empresa traz `socios[]` com: `nome`, `cnpj_cpf` (**CPF mascarado**, ex. `***442017**`), `faixa_etaria`, `qualificacao`, `entrada_sociedade_data`, `identificador` (PF/PJ).

⚠️ **Homônimos:** o match é por nome → pode somar pessoas diferentes. Use o CPF mascarado para desambiguar (mesmo trecho `***NNNNNN**` = mesma pessoa). Nomes parciais/apelidos podem voltar vazio — tente o nome completo registrado ou um sobrenome distintivo.
