# Plugin Nuvia para Claude Code

Skills que ensinam o Claude a usar o **MCP da Nuvia** com as melhores práticas de prospecção, Listas e atendimento — e que já trazem o conector da Nuvia configurado.

O que vem dentro:

| Skill | Para quê |
|---|---|
| `nuvia` | Base de conhecimento (custo, lookups, paginação, linguagem) **e** operação na plataforma: Listas, Pessoas (contatos), salvar buscas e enriquecimento (e-mail/telefone). As outras skills dependem desta. |
| `nuvia-search` | Buscar empresas (BR por CNPJ/setor/UF/sócio e global por firmográficos) e prospectar decisores (CEO/C-level) de uma empresa ou recorte. |
| `nuvia-deep` | Investigação societária BR — cruzar quadro de sócios × identidade digital (quem manda de verdade). |
| `nuvia-conversas` | Ler conversas do inbox e desempenho de campanhas (somente leitura). |

---

## Instalação

### Opção A — Claude Code (recomendado): plugin via marketplace

No Claude Code, adicione o marketplace e instale o plugin:

```
/plugin marketplace add nuvia-ai/nuvia-mcp-plugin
/plugin install nuvia-mcp@nuvia
```

Isso instala as skills **e** configura o conector do MCP da Nuvia automaticamente. Na primeira vez, autentique-se com:

```
/mcp
```

e siga o login (OAuth) do servidor `nuvia`. Depois, é só pedir em linguagem natural — ex.: *"ache os decisores da empresa X"*, *"investigue quem manda na empresa do CNPJ Y"*.

Para atualizar quando sair uma versão nova:

```
/plugin marketplace update nuvia
```

### Opção B — claude.ai: conectar o MCP manualmente

No **claude.ai**, acesse **Conectores → Adicionar conector personalizado** e informe a URL:

```
https://api.nuvia.ai/mcp
```

Autentique-se quando solicitado. As ferramentas do MCP ficam disponíveis no chat. (As skills deste plugin são feitas para o Claude Code; no claude.ai você usa o conector diretamente.)

---

## As ferramentas do MCP (referência rápida)

**Prospecção:** `search_businesses`, `search_prospects`, `search_brazil_companies`, `lookup_filter_values`, `lookup_brazil_filter_values`, `link_brazil_to_global`, `link_global_to_brazil`.
**Salvar na Nuvia:** `save_search_results`, `get_save_status`.
**Contatos:** `list_contacts`, `find_contacts_by_phone`, `create_contact(s)`, `update_contact(s)`, `add_contact_note`, `list_contact_notes`.
**Listas e registros:** `list_lists`, `get_list`, `create_list`, `add_list_column`, `list_records`, `add_records`, `update_record(s)`, `enrich_list`.
**Atendimento e campanhas (leitura):** `list_conversations`, `list_messages`, `list_campaigns`, `get_campaign`.

> **Custo:** buscar e cruzar é **tudo gratuito** — `search_prospects`, `search_businesses`, `search_brazil_companies` e as pontes (`link_brazil_to_global`, `link_global_to_brazil`), além de toda a operação de Pessoas/Empresas/Listas. A **única** operação que consome crédito é o enriquecimento de contato (`enrich_list`, abrir e-mail/telefone). Não há rota de saldo — confira no painel da Nuvia.

---

## Desenvolvimento

Estrutura:

```
.claude-plugin/marketplace.json   # catálogo do marketplace (raiz do repo)
plugin/
  .claude-plugin/plugin.json       # manifesto do plugin
  .mcp.json                        # conector do MCP da Nuvia (http)
  skills/<nome>/SKILL.md           # uma skill por caso de uso
```

Testar localmente sem instalar:

```
claude --plugin-dir ./plugin
```

Validar antes de publicar:

```
claude plugin validate .
claude plugin validate ./plugin
```

Suporte: developers@nuvia.ai · https://nuvia.ai
