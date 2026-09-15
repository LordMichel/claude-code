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
3. **Disponibilidade por horário** — uma faixa por hora com **todas** as salas, cada uma
   com a lotação e uma cor: verde livre a faixa toda, âmbar livre só em parte (com "até
   HH:MM" ou "a partir de HH:MM"), vermelho ocupada, roxo pendente. Uma sala reservada
   fica vermelha na faixa em vez de simplesmente sumir da lista.
4. **Mais e menos reservadas** — barras por sala, em percentual e horas.
5. **Semana** — grade de segunda a sexta em percentual e horas, carregada sozinha ao abrir
   a página e revista a cada 20 min.

**Atualizar tudo**, no cabeçalho, refaz o painel inteiro ignorando todo cache: as dez
leituras do dia e as vinte e cinco da semana saem com `refresh: true`, a agenda do usuário
é relida e os dois retratos são regravados.

Cada bloco carrega um carimbo de sincronização — bolinha verde e `sincronizado às HH:MM`
quando o dado é seu e ao vivo, `dados de HH:MM` quando veio do retrato compartilhado,
âmbar quando está velho e pulsando enquanto busca.

## Reserva que a sala não confirmou

Uma reunião na agenda do usuário pode nomear uma sala sem que a sala tenha aceitado: caixas
de recurso do Exchange recusam automaticamente pedidos fora do horário de funcionamento,
acima da duração máxima ou além da janela de agendamento. O resultado é uma reunião que
existe para quem convidou e não existe para a sala — e o painel, que lê a agenda da sala,
mostra livre com razão.

A página cruza as duas leituras: quando um evento do próprio usuário nomeia uma sala e a
agenda daquela sala está livre no intervalo inteiro, aparece um aviso com o assunto, o
horário e a sala, sugerindo conferir a resposta da sala no Outlook. O cruzamento só usa
dado ao vivo, nunca o retrato compartilhado, que poderia estar velho.

## Rotinas de atualização

Enquanto a página estiver aberta e visível ela se atualiza sozinha, sem clique:

| Rotina | Intervalo | O que faz |
|---|---|---|
| Relógio e cartões | 30 s | redesenha "Agora", o marcador de horário atual e os carimbos |
| Ocupação do dia | 2 min | relê as cinco salas e regrava o retrato `cache/day-<data>` |
| Agenda do usuário | 10 min | relê os eventos que dão nome aos blocos |
| Semana | 20 min | relê os 25 pares sala-dia e regrava `cache/week-<segunda>` |

O relógio de 30 s só dispara o que já venceu, então uma aba aberta faz cerca de 11 chamadas
a cada 2,5 min. Com a aba escondida nada é buscado; ao voltar para ela, ao receber foco ou
ao reconectar à rede a atualização acontece na hora. O retrato compartilhado só é reescrito
quando o conteúdo muda, ou a cada 5 min para manter o horário do carimbo fresco.

## De onde vêm os dados

| Informação | Origem | Disponível para |
|---|---|---|
| Livre/ocupado por sala | `outlook_find_available_time` (Graph findMeetingTimes) | Quem tem o conector Microsoft 365 |
| Assunto e organizador | `outlook_calendar_search` na agenda do próprio usuário | Reuniões em que o usuário participa |
| Retrato compartilhado do dia | documento `cache/day-<data>` no armazenamento do artefato | Qualquer pessoa que abra a página |
| Retrato compartilhado da semana | documento `cache/week-<segunda>` no armazenamento do artefato | Qualquer pessoa que abra a página |

Cada consulta usa a credencial de quem está olhando, nunca a de quem publicou. Quem não tem
o conector não conseguiria ver nada, então a página guarda um retrato compactado da
ocupação do dia — uma string de 54 caracteres por sala, `f` livre, `b` ocupada, `t`
pendente, `u` sem dado — sempre que alguém com conector a carrega. Quem abre sem conector
vê esse retrato, com o horário em que foi tirado e um convite para conectar. O dado ao vivo
sempre tem precedência sobre o retrato; o retrato só preenche lacuna.

O retrato guarda apenas livre/ocupado. Assunto e organizador nunca são gravados: eles são
lidos ao vivo, por quem participa da reunião. O da semana guarda só dois números por
sala-dia: blocos ocupados e percentual.

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

## Conectar sem sair da página

O aviso do topo muda conforme o que falta, lendo `permissions.state("mcp:Microsoft 365")`
— que nunca dispara diálogo — e só perguntando no clique:

| Estado | O que aparece |
|---|---|
| `prompt` — conector na conta, página não autorizada | botão **Permitir acesso**, que chama `permissions.request(["mcp:Microsoft 365"])` e, ao ser concedido, refaz o `listTools` e carrega tudo sem recarregar |
| `denied` — recusado nesta sessão | explicação e botão de recarregar; um novo `request` não reabre o diálogo no mesmo carregamento |
| sem conector / `unavailable` | link **Abrir Conectores no claude.ai** mais os três passos |
| `needs_reauth` | mesmo link, com o texto de reconexão |

Não há como uma página adicionar um conector à conta de quem a abre: isso é ajuste de conta
no claude.ai. O botão cobre o consentimento; o link leva ao lugar certo para o resto. A
linha de status do cabeçalho é derivada do mesmo estado, para não contradizer o aviso.

`permissions` é embutido no runtime e **não** entra em `capabilities`.

## Capacidades declaradas

```json
{
  "mcp": {"servers": [{"server": "Microsoft 365",
          "tools": ["outlook_find_available_time", "outlook_calendar_search"]}]},
  "db":  {"rules": [{"path": "config", "read": "interact", "write": "admin"}]}
}
```

## Como foi testado

As respostas reais do Microsoft 365 para as cinco salas em 2026-09-15 foram capturadas
(`outlook_find_available_time`, fatias de 15 min, dois blocos de 27 candidatos por sala) e
usadas para alimentar a própria página em Chromium headless, com um `window.claude` de
teste que devolve exatamente aquelas respostas. Nada foi simulado: o que a API não devolveu
é ocupado, que é como a página infere.

Conferido contra o cálculo feito à parte a partir das mesmas respostas:

| Sala | Ocupada (horário local) | Grade | % do dia |
|---|---|---|---|
| Beacon | 11:00–12:00, 14:30–15:00 | 6 blocos | 11% · 1h30 |
| Cadeia do Frio | 13:30–14:00, 16:00–17:30 | 8 blocos | 15% · 2h |
| FicaFrio | 15:00–15:30 | 2 blocos | 4% · 30min |
| Super Easy | 14:30–15:30 | 4 blocos | 7% · 1h |
| Syos Easy | 14:00–14:30, 15:00–17:00 | 10 blocos | 19% · 2h30 |

O aviso de reserva não confirmada foi exercitado com os dois eventos reais da agenda do
usuário no dia: "Michel Teste" às 15:00 com a FicaFrio, que a sala aceitou e portanto não
gera aviso, e "TESTE" às 19:00 com a mesma sala, que a agenda da FicaFrio não registrou —
esse gera. O botão **Atualizar tudo** foi medido no mesmo arranjo: um clique produz
exatamente 10 leituras do dia e 25 da semana, todas com `refresh: true`.

Os quatro estados de conexão foram exercitados no mesmo arranjo: com consentimento
pendente a página faz zero chamadas e mostra o botão; o clique concedendo leva a 36
chamadas e aos dados completos; a recusa cai no aviso de recarregar; e sem conector
aparece o link para os Conectores.

Também conferidos: `Livres agora 4/5`, `Ocupação do dia 11% (7h30 de 67h30)`,
`Pico 15:00 com 3 salas`, a ordem dos cartões, as 14 faixas horárias e a gravação dos dois
retratos no banco.

Com o relógio virtual do navegador adiantado por 22 min simulados, as rotinas dispararam
como esperado: o dia a cada 2 a 2,5 min, a semana uma vez aos 20 min, e as escritas de
retrato acompanhando. E com `use('mcp')` devolvendo `null`, a página desenha tudo a partir
dos dois retratos, com os carimbos `dados de HH:MM` e zero chamadas ao conector.
