# Catálogos da base Brasil (`lookup_brazil_filter_values`)

Resolva o termo livre → `value` de catálogo antes de filtrar. Para os catálogos gigantes, passe uma `query` com **uma palavra-chave limpa** ("limitada" funciona; "limitada sociedade" volta vazio).

## Enumerados — use DIRETO, sem lookup

### `uf` (siglas maiúsculas)
`AC AL AP AM BA CE DF ES GO MA MT MS MG PA PB PR PE PI RJ RN RS RO RR SC SP SE TO`

### `porte`
`Microempresa - ME` · `Empresa de pequeno porte - EPP` · `Demais empresas`

### `situacao_cadastral`
`Ativa` · `Baixada` · `Inapta` · `Suspensa` · `Nula`

### `brazil_faixa_etaria_socio`
`21 a 30 anos` · `31 a 40 anos` · `41 a 50 anos` · `51 a 60 anos` · `61 a 70 anos` · `71 a 80 anos` · `Maiores de 80 anos` · `Não se aplica`

### `brazil_setor` (completo)
Comércio · Outras atividades de serviços · Serviços · Administração/saúde/educação públicas · Indústrias de transformação · Transporte/armazenagem/correio · Construção · Informação e comunicação · Agropecuária · Atividades financeiras/seguros · Atividades imobiliárias · Eletricidade e gás/água/esgoto · Indústrias extrativas · Não definido

### `brazil_region` (completo)
Sudeste · Sul · Nordeste · Centro-Oeste · Norte

## Catálogo gigante — resolva por `query`

São finitos só na teoria; têm milhares de valores. Sempre faça lookup:
`brazil_cnae`, `brazil_cnae_codigo`, `brazil_cnae_secundarios_codigo`, `brazil_city`, `brazil_bairro`, `brazil_cep`, `brazil_ddd`, `brazil_natureza_juridica`, `brazil_mesorregiao`, `brazil_microrregiao`, `brazil_socio`, `brazil_qualificacao_socio`, `brazil_identificador`, `brazil_situacao_cadastral_motivo`.

Exemplos do que volta:

**`brazil_natureza_juridica`** (query "limitada"): Sociedade Empresária Limitada · Sociedade Simples Limitada · EIRELI (Empresária) · EIRELI (Simples).

**`brazil_cnae`** (query "energia"): Geração de energia elétrica · Distribuição de energia elétrica · Transmissão de energia elétrica · Comércio atacadista de energia elétrica · etc.

> Os campos de catálogo nem sempre viram a chave de filtro óbvia. Confira `mapeamentos-br.md` antes de montar o `filters` (ex.: `brazil_cnae` → `cnae_principal`).
