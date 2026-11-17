# Migrar para o WhatsApp MCP Local

[🇺🇸 English](migrating-to-local.md)

A versão servidor do WhatsApp MCP vai ser descontinuada em favor do
[WhatsApp MCP Local](https://github.com/BrOrlandi/whatsapp-mcp-local), que
também roda num servidor Linux sem tela: instala sem `sudo`, roda como serviço
do systemd e conecta o WhatsApp com um QR code desenhado no próprio terminal.

A sua instância continua funcionando depois desta versão. Ela só não recebe
novidades. Migrar é uma recomendação, e você escolhe quando.

## Antes de migrar, confira

| | Servidor | Local num servidor |
|---|---|---|
| Onde o MCP responde | um endereço `https` público, com chave | `http://127.0.0.1:47821/mcp`, só no próprio servidor |
| Outro computador | pelo endereço público | por um túnel SSH (`ssh -N -L 47821:127.0.0.1:47821 usuario@servidor`) |
| claude.ai no navegador e o app do Claude no celular | sim | ainda não |
| Números de WhatsApp | várias instâncias | um número |
| Webhooks | sim | sim, no mesmo formato |
| `send_contact` (enviar cartão de contato) | sim | ainda não |
| Peças para manter | Evolution Go, dois Postgres, RabbitMQ, Traefik | um programa e o wacli |

Se você usa o WhatsApp pelo claude.ai ou pelo celular, ou precisa de mais de um
número, continue na versão servidor por enquanto.

## 1. Anote o que vai refazer

- **Webhooks**: em **Configurações › Webhooks**, anote o endereço de cada um e
  os eventos marcados. O formato do que o seu script recebe é o mesmo nas duas
  versões: os mesmos cabeçalhos (`X-WhatsApp-MCP-Event`, `X-WhatsApp-MCP-Delivery`,
  `X-WhatsApp-MCP-Signature`) e o mesmo JSON. A única diferença é que o Local não
  manda `instance_id`. A chave de assinatura muda, porque ela só aparece quando o
  webhook é criado.
- **Ferramentas de IA**: anote onde o MCP está ligado (Claude Code, agentes,
  scripts). O endereço e a chave vão mudar.

## 2. Desligue a versão servidor

No servidor:

```sh
cd /opt/whatsapp-mcp
sudo docker compose -f docker-compose.yml -f deploy/docker-compose.public.yml down
sudo systemctl disable --now whatsapp-mcp-updater.path whatsapp-mcp-updater.service
sudo rm -f /usr/local/bin/whatsapp-mcp
```

O último comando tira o comando `whatsapp-mcp` antigo, que tem o mesmo nome do
novo.

No celular, em **WhatsApp › Configurações › Dispositivos conectados**, desconecte
o aparelho antigo.

Os dados continuam no disco (os volumes do Docker e `/opt/whatsapp-mcp`). Só
apague depois de conferir que o Local está funcionando:

```sh
cd /opt/whatsapp-mcp
sudo docker compose -f docker-compose.yml -f deploy/docker-compose.public.yml down -v
sudo rm -rf /opt/whatsapp-mcp /etc/whatsapp-mcp
```

## 3. Instale o Local e conecte o WhatsApp

Com o usuário que vai rodar o serviço (sem `sudo`):

```sh
curl -fsSL https://raw.githubusercontent.com/BrOrlandi/whatsapp-mcp-local/main/install.sh | bash
whatsapp-mcp pair       # o QR code aparece no terminal
whatsapp-mcp status     # "pronto" quando o WhatsApp estiver conectado
```

Sem acesso à câmera para ler o QR code, conecte com um código digitado no
celular: `whatsapp-mcp pair --phone '+55 11 91234-5678'`.

Para o serviço continuar rodando depois que você sair do SSH:

```sh
sudo loginctl enable-linger $USER
```

O histórico não é copiado da versão servidor. Chega o que o celular manda ao
conectar, e a ferramenta `sync_history` pede mais conversas antigas.

O guia completo, com Docker e transcrição de áudio, é o
[Num servidor sem tela](https://github.com/BrOrlandi/whatsapp-mcp-local/blob/main/docs/servidor.md).

## 4. Refaça as ligações

- **Webhooks**: recrie cada um com o mesmo endereço, e troque a chave de
  assinatura no seu script pela nova.

  ```sh
  whatsapp-mcp webhooks add https://seu-script.exemplo.com/whatsapp --events message
  ```

- **Ferramentas de IA**: `whatsapp-mcp config` mostra como ligar o Claude Code e
  as outras. Um agente que roda no próprio servidor usa
  `http://127.0.0.1:47821/mcp`. De outro computador, abra o túnel SSH e use o
  mesmo endereço nele.
- **Painel**: `whatsapp-mcp open`, ou, de outro computador, o túnel SSH e
  `http://127.0.0.1:47821/` no navegador.

## Dúvidas

Abra uma issue no [repositório do Local](https://github.com/BrOrlandi/whatsapp-mcp-local/issues).
