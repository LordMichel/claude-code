# Salas SyOS — ocupação e reserva das salas de reunião (Artifact)

Página única (HTML + CSS + JS) publicada como Artifact do claude.ai. O foco é a
**visibilidade** que o Teams não dá: as cinco salas lado a lado, quando cada uma está
ocupada e — quando a reunião é do próprio usuário — por quem.

Artifact: https://claude.ai/artifact/JKi9B4bk89hhQ85378zAHW

## Abas

1. **Ocupação das salas** (padrão)
   - Linha do tempo de 7h às 19h30, de 15 em 15 min, para as cinco salas, com marcador
     do horário atual e navegação por dia.
   - Indicadores do dia: ocupação, horas reservadas, salas livres agora, horário de pico.
   - **Reservas do dia**: cada bloco ocupado vira uma linha com horário, sala e nome da
     reunião + organizador quando visível para quem está olhando.
   - Uso por sala (barras) e grade semanal de segunda a sexta, carregada sob demanda.
2. **Reservar** — formulário com as 5 regras da política aplicadas ao vivo.

## De onde vêm os dados

| Informação | Origem | Disponível para |
|---|---|---|
| Livre/ocupado por sala | `outlook_find_available_time` (Graph findMeetingTimes) | Todos |
| Assunto e organizador | `outlook_calendar_search` na agenda do próprio usuário | Reuniões em que o usuário participa |
| Criação da reserva | `outlook_create_event` | Requer `Calendars.ReadWrite` |

Um bloco ocupado só é nomeado quando a reunião está na agenda de quem abre a página. Para
que todos vejam todos os nomes, o administrador precisa liberar leitura no calendário de
cada sala:

```powershell
Set-MailboxFolderPermission -Identity "Sala Cadeia do Frio:\Calendar" -User Default -AccessRights Reviewer
```

## Salas

| Sala | Capacidade | Caixa de recurso |
|---|---|---|
| Beacon | 4 | descoberta |
| Cadeia do Frio | 6 | confirmada |
| FicaFrio | 4 | a descobrir |
| Super Easy | 6 | a descobrir |
| Syos Easy | 8 | a descobrir |

Os endereços das caixas de recurso têm sufixo gerado pelo Microsoft 365
(`SalaCadeiadoFrio…@syos.com`) e não são adivinháveis, mas ninguém precisa procurá-los.
Ao abrir a página, ela varre as reuniões do próprio usuário atrás de salas convidadas e
grava o que encontrar em `discovered/rooms`, um documento que qualquer pessoa da
organização pode escrever. A partir daí a sala vale para todos os usuários da página.
A varredura roda no máximo uma vez a cada seis horas por pessoa; o botão
**Descobrir agora** força a execução.

A configuração em `config/rooms` (só administradores escrevem) tem precedência sobre o
que foi descoberto, então um endereço errado pode ser corrigido lá.

## Capacidades declaradas

```json
{
  "mcp": {"servers": [{"server": "Microsoft 365",
          "tools": ["outlook_find_available_time", "outlook_calendar_search", "outlook_create_event"]}]},
  "db":  {"rules": [{"path": "config", "read": "interact", "write": "admin"}]}
}
```

- `config/rooms` — nome, capacidade e e-mail de cada sala. Leitura para todos, escrita só
  para quem pode editar o Artifact.
- `discovered/rooms` — endereços aprendidos automaticamente. Herda a regra padrão da raiz,
  então qualquer pessoa da organização que abra a página pode contribuir.
