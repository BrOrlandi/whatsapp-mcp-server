# Migrating to WhatsApp MCP Local

[🇧🇷 Português](migrar-para-local.md)

The server version of WhatsApp MCP is being retired in favour of
[WhatsApp MCP Local](https://github.com/BrOrlandi/whatsapp-mcp-local), which
also runs on a headless Linux server: it installs without `sudo`, runs as a
systemd service and connects WhatsApp with a QR code drawn in the terminal.

Your instance keeps working after this version. It just gets no new features.
Migrating is a recommendation, and you choose when.

## Before you migrate, check

| | Server | Local on a server |
|---|---|---|
| Where the MCP answers | a public `https` address, with a key | `http://127.0.0.1:47821/mcp`, on the server itself |
| Another computer | through the public address | through an SSH tunnel (`ssh -N -L 47821:127.0.0.1:47821 user@server`) |
| claude.ai in the browser and the Claude app on the phone | yes | not yet |
| WhatsApp numbers | several instances | one number |
| Webhooks | yes | yes, same format |
| `send_contact` (send a contact card) | yes | not yet |
| Parts to maintain | Evolution Go, two Postgres, RabbitMQ, Traefik | one program and wacli |

If you use WhatsApp from claude.ai or the phone, or need more than one number,
stay on the server version for now.

## 1. Write down what you will redo

- **Webhooks**: in **Configurações › Webhooks**, note each one's address and
  events. What your script receives is the same in both versions: the same
  headers (`X-WhatsApp-MCP-Event`, `X-WhatsApp-MCP-Delivery`,
  `X-WhatsApp-MCP-Signature`) and the same JSON. The only difference is that
  Local does not send `instance_id`. The signing secret changes, since it is only
  shown when the webhook is created.
- **AI tools**: note where the MCP is connected (Claude Code, agents, scripts).
  The address and the key will change.

## 2. Turn the server version off

On the server:

```sh
cd /opt/whatsapp-mcp
sudo docker compose -f docker-compose.yml -f deploy/docker-compose.public.yml down
sudo systemctl disable --now whatsapp-mcp-updater.path whatsapp-mcp-updater.service
sudo rm -f /usr/local/bin/whatsapp-mcp
```

The last command removes the old `whatsapp-mcp` command, which has the same name
as the new one.

On the phone, in **WhatsApp › Settings › Linked devices**, log the old device
out.

The data stays on disk (the Docker volumes and `/opt/whatsapp-mcp`). Delete it
only once Local is working:

```sh
cd /opt/whatsapp-mcp
sudo docker compose -f docker-compose.yml -f deploy/docker-compose.public.yml down -v
sudo rm -rf /opt/whatsapp-mcp /etc/whatsapp-mcp
```

## 3. Install Local and connect WhatsApp

As the user that will run the service (no `sudo`):

```sh
curl -fsSL https://raw.githubusercontent.com/BrOrlandi/whatsapp-mcp-local/main/install.sh | bash
whatsapp-mcp pair       # the QR code shows up in the terminal
whatsapp-mcp status     # "ready" once WhatsApp is connected
```

Can't scan the QR code? Connect with a code typed on the phone instead:
`whatsapp-mcp pair --phone '+1 555 010 0100'`.

To keep the service running after you leave SSH:

```sh
sudo loginctl enable-linger $USER
```

History is not copied from the server version. What the phone sends on
connecting arrives, and the `sync_history` tool asks for older chats.

The full guide, with Docker and voice-note transcription, is
[On a headless server](https://github.com/BrOrlandi/whatsapp-mcp-local/blob/main/docs/servidor.md)
(in Portuguese).

## 4. Reconnect everything

- **Webhooks**: recreate each one with the same address, and give your script
  the new signing secret.

  ```sh
  whatsapp-mcp webhooks add https://your-script.example.com/whatsapp --events message
  ```

- **AI tools**: `whatsapp-mcp config` shows how to connect Claude Code and the
  others. An agent running on the server itself uses
  `http://127.0.0.1:47821/mcp`. From another computer, open the SSH tunnel and
  use the same address there.
- **Panel**: `whatsapp-mcp open`, or, from another computer, the SSH tunnel and
  `http://127.0.0.1:47821/` in the browser.

## Questions

Open an issue on the [Local repository](https://github.com/BrOrlandi/whatsapp-mcp-local/issues).
