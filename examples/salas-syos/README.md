# Salas SyOS — visibilidade das salas de reunião (Artifact)

Página única (HTML + CSS + JS) publicada como Artifact do claude.ai. Responde numa tela só
o que o Teams não mostra: **qual sala está livre agora, até que horas, e quem reservou**.
Não faz agendamento — reservar continua no Teams ou no Outlook, adicionando a sala como
local da reunião.

Artifact: https://claude.ai/artifact/JKi9B4bk89hhQ85378zAHW

## Seções

1. **Agora** — um cartão por sala com livre ou ocupada, até que horas, quantas horas
   disponíveis e o próximo horário livre. Filtro por livres/ocupadas e ordenação que
   coloca primeiro quem tem a maior janela livre e por último quem fica ocupada por mais
   tempo.
2. **Ocupação do dia** — linha do tempo das 7h às 20h30, de 15 em 15 min, com marcador do
   horário atual e navegação por dia. Indicadores em percentual e em horas.
3. **Disponibilidade por horário** — uma faixa por hora com quantas e quais salas estão
   livres; salas livres só em parte da faixa aparecem com contorno tracejado e o horário
   em que ficam ocupadas.
4. **Mais e menos reservadas** — barras por sala, em percentual e horas.
5. **Semana** — grade de segunda a sexta em percentual e horas, carregada sob demanda.

## De onde vêm os dados

| Informação | Origem | Disponível para |
|---|---|---|
| Livre/ocupado por sala | `outlook_find_available_time` (Graph findMeetingTimes) | Todos |
| Assunto e organizador | `outlook_calendar_search` na agenda do próprio usuário | Reuniões em que o usuário participa |

Um bloco ocupado só é nomeado quando a reunião está na agenda de quem abre a página; os
demais aparecem como reservados, sem nome. A página não depende disso: a organização dos
dados é por horário e por sala, não por quem reservou.

A janela do dia tem 54 fatias de 15 min e o limite da API é 50 candidatos por chamada, então
cada sala-dia é lida em duas chamadas. A grade da semana usa fatias de 30 min, uma chamada
por sala-dia. Para que todos vejam todos os nomes, o
administrador libera leitura no calendário de cada sala:

```powershell
Set-MailboxFolderPermission -Identity "Sala Cadeia do Frio:\Calendar" -User Default -AccessRights Reviewer
```

A agenda da sala não é legível pela API de eventos com a permissão padrão — apenas
livre/ocupado. Isso foi verificado: ler o próprio calendário funciona, ler o da sala
retorna `ErrorItemNotFound`.

## Salas

As cinco salas do escritório de São Paulo, com o endereço da caixa de recurso lido de um
convite real na agenda do usuário:

| Sala | Lugares | Caixa de recurso |
|---|---|---|
| Beacon | 4 | `SalaBeacon@syos.com` |
| Cadeia do Frio | 6 | `SalaCadeiadoFriofc7570d9…@syos.com` |
| FicaFrio | 4 | `SaladeReunioOperaes86cd8dfa…@syos.com` |
| Super Easy | 6 | `SalaPaulo3916ca4f…@syos.com` |
| Syos Easy | 8 | `SalaBugs8bf36d62…@syos.com` |

Três apelidos vêm de nomes antigos das salas: a Super Easy era Sala Paulo, a Syos Easy era
Sala Bugs e a FicaFrio era Sala de Reunião Operações. Por isso não há como derivar o
endereço do nome atual, e nenhum foi deduzido — cada um apareceu como participante de uma
reunião real.

Sem endereço não há consulta a fazer, e a sala aparece como **Desconhecida**, nunca como
livre. A página aprende sozinha: ao abrir, varre as reuniões do próprio usuário, identifica
a sala pelo campo de local do evento e grava o endereço em `discovered/rooms`, que qualquer
pessoa da organização pode escrever. A varredura roda no máximo uma vez a cada seis horas
por pessoa; **Descobrir agora** força a execução. A configuração em `config/rooms` (só
administradores) tem precedência.

## Capacidades declaradas

```json
{
  "mcp": {"servers": [{"server": "Microsoft 365",
          "tools": ["outlook_find_available_time", "outlook_calendar_search"]}]},
  "db":  {"rules": [{"path": "config", "read": "interact", "write": "admin"}]}
}
```
