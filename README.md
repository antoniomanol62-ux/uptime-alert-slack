# uptime-alert-slack

Monitor de disponibilidade de sites em Bash: verifica cada URL, tenta de novo antes de declarar falha e avisa no Slack quando o site realmente caiu. Containerizado, com pipeline de CI no GitHub Actions.

## O problema que resolve

Checar manualmente se um site está no ar é lento e não escala: alguém precisa lembrar, abrir o navegador e conferir cada URL. Pior, uma falha passageira gera falso alarme. Este projeto automatiza a checagem, confirma a falha com retry e manda o alerta direto no canal do time.

## Como funciona

1. Para cada URL da lista, o script faz uma requisição com `curl` e lê o código HTTP.
2. Se não for 200, tenta novamente (até 3 tentativas, com pausa de 2s entre elas).
3. Se todas falharem, registra o alerta no log e envia mensagem ao Slack via webhook.
4. Se a variável `SLACK_WEBHOOK_URL` não estiver definida, o script só registra em log e não quebra.

## Stack

- **Bash + curl**: verificação HTTP (`health_check.sh`)
- **Docker**: containeriza o script
- **Docker Compose**: sobe o health check (em loop, com log em volume) junto com Prometheus e Grafana. A integração das métricas ainda está no Roadmap
- **GitHub Actions**: builda a imagem a cada push na `main`

## Como rodar

```bash
git clone https://github.com/antoniomanol62-ux/uptime-alert-slack.git
cd uptime-alert-slack

cp .env.example .env
# edite o .env e cole a URL do seu webhook do Slack

docker compose up -d
docker compose logs -f health-check
```

O `.env` deve ter uma linha por variável, no formato `NOME=valor`, sem aspas e sem espaços em volta do `=`:

- `SLACK_WEBHOOK_URL`: URL do webhook do Slack (opcional; sem ela o monitor só registra em log)
- `SITES`: URLs a monitorar, separadas por espaço (sem ela, o monitor usa `https://example.com`)

O health check roda em loop (uma rodada por minuto) e grava o log em `./logs/`.

## Estrutura

- `health_check.sh`: checagem HTTP, retry e alerta no Slack
- `Dockerfile`: containeriza o script
- `docker-compose.yml`: sobe o health check, Prometheus e Grafana, com restart automático
- `.env.example`: modelo da configuração (o `.env` real fica fora do Git)
- `.github/workflows/`: pipeline de CI

## Roadmap
- [x] Rodar o health check como serviço no Compose, em loop contínuo
- [x] Ler a URL do webhook de um `.env` (com `.env.example`)
- [x] Persistir o log em volume
- [x] Tratar redirects (`curl -L`) e definir timeout (`--max-time`)
- [x] Receber a lista de sites por variável de ambiente (`SITES`)
- [ ] Avisar no log quando o webhook não estiver configurado ou quando o padrão `example.com` for usado
- [ ] Evitar alerta repetido a cada minuto enquanto o site continua fora do ar
- [ ] Expor métricas para o Prometheus e montar um dashboard no Grafana


## Sobre o projeto

Construído como prática de DevOps: containerização, automação de pipeline e alertas, evoluindo uma peça de cada vez.
