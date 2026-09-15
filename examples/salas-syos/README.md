# Salas SyOS — visibilidade das salas de reunião (Artifact)

Página única (HTML + CSS + JS) publicada como Artifact do claude.ai. Responde numa tela só
o que o Teams não mostra: **qual sala está livre agora, até que horas, e quem reservou**.
Não faz agendamento — reservar continua no Teams ou no Outlook, adicionando a sala como
local da reunião.

Artifact: https://claude.ai/artifact/JKi9B4bk89hhQ85378zAHW

## Seções

1. **Agora** — um cartão por sala: livre ou ocupada, até que horas, e o nome da reunião
   com o organizador quando visível.
2. **Ocupação do dia** — linha do tempo de 7h às 19h30 de 15 em 15 min para as cinco
   salas, marcador do horário atual, navegação por dia e indicadores (ocupação, horas
   reservadas, livres agora, horário de pico). Clicar num bloco livre abre o Outlook já
   com a sala e o horário.
3. **Reservas do dia** — cada bloco ocupado como uma linha: horário, sala, reunião e
   organizador.
4. **Uso por sala** — barras de ocupação do dia.
5. **Semana** — grade de segunda a sexta, carregada sob demanda.

## De onde vêm os dados

| Informação | Origem | Disponível para |
|---|---|---|
| Livre/ocupado por sala | `outlook_find_available_time` (Graph findMeetingTimes, granularidade de 15 min) | Todos |
| Assunto e organizador | `outlook_calendar_search` na agenda do próprio usuário | Reuniões em que o usuário participa |

Um bloco ocupado só é nomeado quando a reunião está na agenda de quem abre a página; os
demais aparecem como reservados, sem nome. Para que todos vejam todos os nomes, o
administrador libera leitura no calendário de cada sala:

```powershell
Set-MailboxFolderPermission -Identity "Sala Cadeia do Frio:\Calendar" -User Default -AccessRights Reviewer
```

A agenda da sala não é legível pela API de eventos com a permissão padrão — apenas
livre/ocupado. Isso foi verificado: ler o próprio calendário funciona, ler o da sala
retorna `ErrorItemNotFound`.

## Salas

| Sala | Lugares | Caixa de recurso |
|---|---|---|
| Beacon | 4 | desconhecida |
| Cadeia do Frio | 6 | `SalaCadeiadoFrio…@syos.com` |
| FicaFrio | 4 | desconhecida |
| Super Easy | 6 | `SalaPaulo…@syos.com` |
| Syos Easy | 8 | `SalaBugs…@syos.com` |

Cada endereço cadastrado veio de um convite real observado na agenda do usuário. Nenhum
foi deduzido: um palpite que resolve no Exchange e devolve livre/ocupado ainda assim não
prova ser a sala certa, e mostrar isso como agenda da sala seria inventar dado.

Os apelidos vêm de nomes antigos das salas, então não correspondem ao nome atual nem podem
ser adivinhados. Sem o endereço não há consulta a fazer: a sala aparece como **Desconhecida**,
nunca como livre, e a página explica o porquê acima da linha do tempo.

A página aprende sozinha: ao abrir, varre as reuniões do próprio usuário, identifica a sala
pelo campo de local do evento e grava o endereço em `discovered/rooms`, que qualquer pessoa
da organização pode escrever. A varredura roda no máximo uma vez a cada seis horas por
pessoa; **Descobrir agora** força a execução. A configuração em `config/rooms` (só
administradores) tem precedência.

## Capacidades declaradas

```json
{
  "mcp": {"servers": [{"server": "Microsoft 365",
          "tools": ["outlook_find_available_time", "outlook_calendar_search"]}]},
  "db":  {"rules": [{"path": "config", "read": "interact", "write": "admin"}]}
}
```
