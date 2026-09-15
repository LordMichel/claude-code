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
| Livre/ocupado por sala | `outlook_find_available_time` (Graph findMeetingTimes) | Quem tem o conector Microsoft 365 |
| Assunto e organizador | `outlook_calendar_search` na agenda do próprio usuário | Reuniões em que o usuário participa |
| Retrato compartilhado | documento `cache/day-<data>` no armazenamento do artefato | Qualquer pessoa que abra a página |

Cada consulta usa a credencial de quem está olhando, nunca a de quem publicou. Quem não tem
o conector não conseguiria ver nada, então a página guarda um retrato compactado da
ocupação do dia — uma string de 54 caracteres por sala, `f` livre, `b` ocupada, `t`
pendente, `u` sem dado — sempre que alguém com conector a carrega. Quem abre sem conector
vê esse retrato, com o horário em que foi tirado e um convite para conectar. O dado ao vivo
sempre tem precedência sobre o retrato; o retrato só preenche lacuna.

O retrato guarda apenas livre/ocupado. Assunto e organizador nunca são gravados: eles são
lidos ao vivo, por quem participa da reunião.

A janela do dia tem 54 fatias de 15 min e o limite da API é 50 candidatos por chamada, então
cada sala-dia é lida em duas chamadas. A grade da semana usa fatias de 30 min, uma chamada
por sala-dia.

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
