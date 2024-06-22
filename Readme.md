> forked from [KostyaEsmukov/smtp_to_telegram](https://github.com/KostyaEsmukov/smtp_to_telegram)

# Utilisation au ResEl

Ce programme est utilisé pour envoyer des messages dans un canal Telegram en se faisant passer pour un serveur SMTP. Il est utilisé pour envoyer des messages d'alerte depuis UniFi dans un canal Telegram de monitoring au ResEl (lien vers le canal disponible sur #ResEl).

Il est déployé sur la machine virtuelle `ubnt` (du controlleur UniFi) et est configuré pour écouter sur le port 2525 (sur la loopback uniquement). Les messages envoyés à cette adresse sont ensuite transmis au canal Telegram par le biais d'un bot.

# SMTP to Telegram

[![Docker Hub](https://img.shields.io/docker/pulls/kostyaesmukov/smtp_to_telegram.svg?style=flat-square)][Docker Hub]
[![Go Report Card](https://goreportcard.com/badge/github.com/KostyaEsmukov/smtp_to_telegram?style=flat-square)][Go Report Card]
[![License](https://img.shields.io/github/license/KostyaEsmukov/smtp_to_telegram.svg?style=flat-square)][License]

[Docker Hub]:      https://hub.docker.com/r/kostyaesmukov/smtp_to_telegram
[Go Report Card]:  https://goreportcard.com/report/github.com/KostyaEsmukov/smtp_to_telegram
[License]:         https://github.com/KostyaEsmukov/smtp_to_telegram/blob/master/LICENSE

`smtp_to_telegram` is a small program which listens for SMTP and sends
all incoming Email messages to Telegram.

Say you have a software which can send Email notifications via SMTP.
You may use `smtp_to_telegram` as an SMTP server so
the notification mail would be sent to the chosen Telegram chats.

## Getting started

1. Create a new Telegram bot: <https://core.telegram.org/bots#creating-a-new-bot>.
2. Open that bot account in the Telegram account which should receive
    the messages, press `/start`.
3. Retrieve a chat id with `curl https://api.telegram.org/bot<BOT_TOKEN>/getUpdates`.
4. Repeat steps 2 and 3 for each Telegram account which should receive the messages.

## Usage

> For chats that are inside threads you need to set `ST_TELEGRAM_CHAT_IDS` with the chat id and the thread id separated by an underscore. For example: `-213213089_4` and set `ST_TELEGRAM_IS_THREAD` to true.

Assuming that your Email-sending software is running in docker as well,
you may use `smtp_to_telegram:2525` as the target SMTP address.
No TLS or authentication is required.

The default Telegram message format is:

```ini
From: {from}\\nTo: {to}\\nSubject: {subject}\\n\\n{body}\\n\\n{attachments_details}
```

### Inside a docker container

```bash
docker run \
    --name smtp_to_telegram \
    -e ST_TELEGRAM_CHAT_IDS=<CHAT_ID1>,<CHAT_ID2> \
    -e ST_TELEGRAM_IS_THREAD=<true/false> \
    -e ST_TELEGRAM_BOT_TOKEN=<BOT_TOKEN> \
    kostyaesmukov/smtp_to_telegram
```

A custom format might be specified as well:

```bash
docker run \
    --name smtp_to_telegram \
    -e ST_TELEGRAM_CHAT_IDS=<CHAT_ID1>,<CHAT_ID2> \
    -e ST_TELEGRAM_BOT_TOKEN=<BOT_TOKEN> \
    -e ST_TELEGRAM_MESSAGE_TEMPLATE="Subject: {subject}\\n\\n{body}" \
    kostyaesmukov/smtp_to_telegram
```

### As a standalone service (without docker)

#### Build from source

```bash
GOPROXY=direct
CGO_ENABLED=0 GOOS=linux go build \
        -ldflags "-s -w \
            -X main.Version=${ST_VERSION:-UNKNOWN_RELEASE}" \
        -a -o smtp_to_telegram
```

#### Run

This example use custom format (optional).

```bash
smtp_to_telegram \ 
    --smtp-listen "127.0.0.1:2525"\
    --telegram-chat-ids <CHAT_ID1>,<CHAT_ID2> \
    --telegram-is-thread <true/false> \
    --telegram-bot-token <BOT_TOKEN> \
    --message-template "Subject: {subject}\\n\\n{body}"
```

You can create a systemd service to run `smtp_to_telegram` as a daemon.

```ini
[Unit]
Description=SMTP to Telegram for Unifi alerting
After=network.target

[Service]
Type=simple
Restart=always
ExecStart=/srv/smtp_to_telegram/smtp_to_telegram --smtp-listen "127.0.0.1:2525" --telegram-chat-ids <CHAT_ID> --telegram-is-thread <true/false> --telegram-bot-token <BOT_TOKEN> --message-template "**{subject}** \\n\\n{body}"

[Install]
WantedBy=multi-user.target
```
