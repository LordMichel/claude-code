# Salas SyOS — reserva de salas de reunião (Artifact)

Página única (HTML + CSS + JS) publicada como Artifact do claude.ai para reservar as
quatro salas de reunião da SyOS (Beacon, Cadeia do Frio, Fica Frio, Super Easy)
seguindo a Política Interna de Uso das Salas de Reunião.

Artifact publicado: https://claude.ai/artifact/JKi9B4bk89hhQ85378zAHW

## O que a página faz

- Aplica as 5 regras da política antes de enviar: reserva pelo Teams (o convite vai
  para a caixa de recurso da sala), antecedência mínima de 30 min, assunto obrigatório
  (uso corporativo), capacidade da sala, sem recorrência.
- Consulta a agenda de cada sala no dia escolhido (faixa de 15 em 15 min, 7h–19h30)
  usando o conector Microsoft 365 do próprio usuário (`outlook_find_available_time`).
- Um clique em "Reservar sala" cria o evento no Outlook/Teams do usuário
  (`outlook_create_event`) com a sala como recurso e os participantes convidados.
- Se o conector não puder criar eventos, abre o Outlook/Teams já preenchido.

## Capacidades declaradas na publicação

```json
{
  "mcp": {"servers": [{"server": "Microsoft 365", "tools": ["outlook_create_event", "outlook_find_available_time"]}]},
  "db":  {"rules": [{"path": "config", "read": "interact", "write": "admin"}]}
}
```

- `mcp`: chamadas feitas com as credenciais de quem está vendo a página (cada pessoa
  precisa ter o conector Microsoft 365 adicionado no claude.ai).
- `db`: documento `config/rooms` com capacidade e e-mail (caixa de recurso) de cada
  sala; leitura para todos, escrita só para quem pode editar o Artifact.

## Pré-requisitos no Microsoft 365

- Para criar reservas pela página, o conector do claude.ai precisa da permissão
  delegada `Calendars.ReadWrite` (hoje o tenant concede apenas leitura). Sem ela, a
  página informa e oferece o Outlook preenchido.
- Cada sala precisa existir como caixa de recurso (room mailbox) no Exchange Online com
  processamento automático (aceita se livre, recusa se ocupada).

## Configuração inicial

Abrir a página como administrador → "Configurar salas" → informar capacidade e e-mail
de cada sala → "Salvar para todos".
