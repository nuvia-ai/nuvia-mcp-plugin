# Playbook: cruzar quadro de sócios × identidade digital

## Buckets de classificação

Para cada nome, cruze presença no quadro de sócios (QSA) × presença no digital:

| Bucket | Sócio? | Digital? | Leitura | Ação comercial |
|--------|:----:|:--------:|---------|----------------|
| 🟢 Dono + digital | ✅ | ✅ | Decisor real, ativo publicamente | Alvo nº 1 — aborda direto |
| 🔵 Dono silencioso | ✅ | ❌ | Sócio sem pegada digital (investidor, familiar, sócio PJ) | Mapear por fora; pode ser quem assina |
| 🟡 Executivo contratado | ❌ | ✅ | C-level/diretor sem equity local (ou equity na holding) | Decisor operacional; entrada técnica |
| 🔗 Sócio PJ (holding) | ✅ (PJ) | — | Pessoa jurídica sócia → gancho de grupo | Seguir o CNPJ: controle, outras empresas |
| ⚪ Colaborador digital | ❌ | ✅ (não-liderança) | Funcionário comum | Contexto, não decisor |

## Natureza jurídica — o que o quadro entrega

| | LTDA (Soc. Empresária Limitada) | S.A. (Sociedade Anônima) |
|---|---|---|
| código típico | `2062` | `2054` (fechada) / `2046` (aberta) |
| O quadro mostra… | os **sócios reais** (donos) + administradores | **só a diretoria estatutária** |
| Acionistas/controle | aparecem (cotistas PF e PJ) | **NÃO aparecem** — ficam na holding/listco |
| Sócio PJ | gancho de grupo econômico | raríssimo; controle é externo |
| Match com digital | bom para **donos** | bom para **C-level executivo**, nunca para dono |

> Regra mental: em LTDA o quadro é o cap table (aproximado); em S.A. é o organograma da diretoria. Quem controla a S.A. de verdade não está no quadro — está acima, normalmente fora do Brasil.

## Árvore de decisão

```
1. search_brazil_companies(cnpj) → ler natureza_juridica
   ├─ LTDA → quadro = sócios reais. Marcar PF (donos) e PJ (holdings).
   └─ S.A. → quadro = diretoria. Avisar: controlador NÃO está aqui.
2. Tem sócio PJ? → bucket 🔗; opcionalmente search_brazil_companies no CNPJ da holding.
3. link_brazil_to_global(cnpj, domain, razão social) → identificador global
   └─ found:false? → tentar outro domínio/nome. Sem ele, não há fase digital.
4. search_prospects(empresa, job_level=[c-suite,vice president,director])  ← grátis
5. Cruzar (matching abaixo) e classificar (buckets acima).
6. Entregar tabela UNIFICADA (outer join, ninguém descartado):
   Nome | Papel no quadro | Cargo digital | LinkedIn | Bucket | Confiança.
```

> O cruzamento é um **outer join** por nome: sócio com match vira linha unificada (com LinkedIn); sócio sem pegada digital entra mesmo assim (LinkedIn em branco, 🔵); contato digital sem ser sócio continua listado (🟡/⚪). Nunca dropar um lado por não casar com o outro.

## Matching de nomes (heurístico — não há chave comum)

1. **Normalizar:** upper, remover acentos, remover títulos/sufixos (JR, NETO, FILHO, PhD, CFA), colapsar espaços.
2. **Tokenizar** e comparar conjuntos (ordem não importa).
3. **Match forte:** primeiro nome **e** último sobrenome batem. Nomes do meio que somem no LinkedIn são esperados.
4. **Match parcial:** só um lado bate → confirme por cargo/empresa.
5. **CPF mascarado** só desambigua dentro do quadro de sócios (homônimos); o LinkedIn não expõe CPF, então nunca fecha match sozinho.
6. **LinkedIn para exibir:** prefira o **handle legível** do `linkedin_url_array` (ex: `linkedin.com/in/nome-sobrenome`) — costuma ser o segundo item. O campo `linkedin` e o primeiro item do array trazem a URN opaca (`linkedin.com/in/ACoAA…`); só use-a se não houver handle legível.

| Confiança | Critério |
|-----------|----------|
| Exato | tokens normalizados idênticos |
| Forte | 1º nome + último sobrenome batem; só meio/sufixo divergem |
| Parcial | bate parte do nome **+** confirma por cargo/empresa |
| Fraco/descartar | só 1º nome OU só sobrenome, sem reforço de cargo |

## Ruídos a descartar / sinalizar

- **Contas não-humanas:** bots/QA. Descartar.
- **Ex-funcionários defasados:** perfil ainda atrelado mas histórico já mostra cargo novo em outra empresa. Sinalizar como stale.
- **Inflação de senioridade:** nível às vezes marca como c-suite quem é assistente/secretária. Conferir pelo cargo real.
- **Geografia fora do esperado:** contractors no exterior. Não confundir com liderança local.
- **Homônimos:** nome comum + sobrenome curto → exigir match por cargo/empresa.
- **Teto de cobertura:** nem todo sócio/diretor tem perfil atrelado à empresa (conselheiros discretos, nomeações recentes, perfis acadêmicos, pessoas de outra entidade do grupo). Mesmo esgotando páginas e alargando o nível, podem não surgir. Registre como "sem pegada digital" (🔵) em vez de inventar correspondência. Num caso real o teto foi ~69% (9 de 13 diretores).

## Síntese por natureza jurídica

| Natureza | Quadro entrega | Cruzamento revela | Limite |
|----------|-------------|-------------------|--------|
| LTDA | Donos (PF) + holdings (PJ) | Quem é dono **e** atua; founders sem equity local | Holding no exterior corta a trilha de controle |
| S.A. | Diretoria estatutária | Mapa do C-level executivo (alta taxa de match) | Controlador é invisível (fica na listco/holding) |

Para prospecção: em LTDA, mire o 🟢 dono+digital e investigue 🔗 holding para o grupo; em S.A., a entrada é o 🟡 executivo, ciente de que a compra pode subir para o controlador.
