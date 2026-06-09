---
name: inspecionar-atendimento
description: Inspeciona conversas de atendimento e campanhas no CRM da Nuvia (somente leitura). Use quando o usuário quer revisar conversas do inbox, ler o histórico de mensagens de um contato, ou inventariar campanhas e ver métricas/detalhes de uma campanha. Gratuito.
---

# Inspecionar atendimento e campanhas

Tudo aqui é **somente leitura** e gratuito.

**Antes de começar, leia a skill `nuvia-fundamentos`** (linguagem). Detalhes das tools em `nuvia-fundamentos/references/tools.md`.

## Conversas (inbox)

```
list_conversations(search?, status? "OPEN"|"RESOLVED", channel?, page, page_size máx 50)
  →  { data[{ contato, inbox, agente, status, último resumo }], total, has_more }

list_messages(<id da conversa>)   # mensagens, mais recentes primeiro
```

Fluxo típico: `list_conversations` para achar a conversa (filtre por `status`/`channel`) → `list_messages` com o id para ler o histórico. `channel` aceita valores como `LINKEDIN`, `WAP_AUTO_CLOSER`, etc.

## Campanhas

```
list_campaigns()           # inventário, para achar o id
get_campaign(id)  →  { id, name, status, channel_type, trigger_type, metrics, ... }
```

## Entregar

Resuma em linguagem de negócio: estado das conversas (abertas/resolvidas), o que o cliente pediu, próximos passos sugeridos; para campanhas, o funil e as métricas relevantes. Não é uma skill de escrita — não há rotas para enviar mensagem ou alterar campanha pelo MCP.
