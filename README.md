# Bot de Clima no Telegram com n8n

Chatbot para Telegram que recebe uma cidade, consulta a API gratuita da OpenWeather e responde com a temperatura atual em graus Celsius.

Exemplo de resposta:

```text
🌤️ A temperatura em Belo Horizonte é de 25°C.
```

## Arquivos

- `workflow-chatbot-telegram.json`: workflow exportável do n8n.
- `docker-compose.yml`: opção para rodar o n8n localmente com Docker.
- `.env.example`: modelo das variáveis usadas no projeto.

## Variáveis e credenciais

O projeto espera estas variáveis:

```env
OPENWEATHER_API_KEY=sua_chave_da_openweather
TELEGRAM_BOT_TOKEN=token_do_botfather
WEBHOOK_URL=https://sua-url-publica-https/
```

Importante: não coloque tokens reais no repositório. O workflow usa `OPENWEATHER_API_KEY` por expressão no nó `Consultar OpenWeather`. O `TELEGRAM_BOT_TOKEN` deve ser usado para criar uma credencial do tipo Telegram no n8n.

Se o n8n estiver rodando localmente, o Telegram Trigger precisa de uma URL pública HTTPS em `WEBHOOK_URL`, como uma URL de ngrok, Cloudflare Tunnel ou um domínio com proxy/reverse proxy configurado.

## Como importar no n8n

1. Abra o n8n.
2. Vá em `Workflows`.
3. Clique em `Import from File`.
4. Selecione `workflow-chatbot-telegram.json`.
5. Abra o nó `Telegram Trigger` e selecione ou crie a credencial Telegram.
6. Abra os nós `Enviar clima` e `Enviar erro` e selecione a mesma credencial Telegram.
7. Confirme que a variável `OPENWEATHER_API_KEY` está disponível no ambiente em que o n8n roda.
8. Salve e ative o workflow.

As credenciais não ficam salvas no arquivo `workflow-chatbot-telegram.json` de propósito. Depois de importar, selecione a credencial Telegram nos nós indicados acima.

## Rodando com Docker

Crie um arquivo `.env` a partir do `.env.example` e preencha os valores reais localmente:

```bash
OPENWEATHER_API_KEY=sua_chave_real
TELEGRAM_BOT_TOKEN=seu_token_real
```

Depois suba o n8n:

```bash
docker compose up -d
```

O n8n ficará disponível em `http://localhost:5678`.

Para o Telegram Trigger funcionar localmente, publique o n8n em uma URL HTTPS e configure no `.env`:

```bash
WEBHOOK_URL=https://sua-url-publica-https/
```

Depois reinicie o Docker:

```bash
docker compose down
docker compose up -d
```

## Rodando direto no Windows CMD

Se você inicia o n8n direto no `cmd`, defina as variáveis no mesmo terminal antes de rodar o n8n. O arquivo `.env` não é carregado automaticamente nesse modo.

```bat
set "OPENWEATHER_API_KEY=sua_chave_real"
set "TELEGRAM_BOT_TOKEN=seu_token_real"
set "WEBHOOK_URL=https://sua-url-publica-https/"
set "N8N_PROXY_HOPS=1"

n8n start
```

Se você usa `npx`, troque o último comando:

```bat
npx n8n start
```

Depois de alterar `WEBHOOK_URL`, pare o n8n, rode os comandos acima de novo, abra o workflow, salve, desative e ative novamente para registrar a webhook atual no Telegram.

## Como testar

Envie mensagens para o seu bot no formato:

```text
Cidade,UF,BR
```

Sugestões de teste:

- `São Paulo,SP,BR`
- `Belo Horizonte,MG,BR`
- `Recife,PE,BR`
- `CidadeInexistente,ZZ,BR`

Para uma cidade válida, o bot responde com a temperatura atual. Para uma cidade inválida ou resposta incompleta da API, ele responde:

```text
❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).
```

## Como o workflow funciona

1. `Telegram Trigger` recebe mensagens de texto.
2. `Capturar e formatar entrada` salva o `chatId`, o texto original, a variável `queue` com a cidade normalizada e `weatherQuery` para consulta na API.
3. `Consultar OpenWeather` chama `https://api.openweathermap.org/data/2.5/weather` com `q`, `units=metric`, `lang=pt_br` e `appid={{ $env.OPENWEATHER_API_KEY }}`.
4. `Preparar mensagem` valida o status HTTP, extrai `main.temp` e monta a mensagem.
5. `Resposta válida?` separa sucesso e erro.
6. `Enviar clima` ou `Enviar erro` responde ao usuário no Telegram.

O nó `Capturar e formatar entrada` aceita mensagens que contenham o padrão `Cidade,UF,BR`, como `Qual o clima em São Paulo,SP,BR?`. Ele extrai `São Paulo,SP,BR` para `queue` e envia uma versão compatível com a OpenWeather no campo `weatherQuery`, por exemplo `São Paulo,BR`.

Observação: a OpenWeather recomenda chamadas por coordenadas e mantém a chamada por nome de cidade por compatibilidade. Para este desafio, o workflow usa o endpoint pedido, `/data/2.5/weather`, e envia a cidade no parâmetro `q`.

## Checklist antes da entrega

- Workflow importado no n8n.
- Credencial Telegram configurada nos três nós Telegram.
- `OPENWEATHER_API_KEY` configurada no ambiente do n8n.
- Teste realizado com pelo menos 3 cidades válidas.
- Teste realizado com 1 cidade inexistente.
- Nenhum token real salvo no JSON, README ou arquivos versionados.
  
- Working!
<img width="394" height="226" alt="image" src="https://github.com/user-attachments/assets/aab32fb9-d590-48c9-a010-229380ecfcd5" />

## Problemas comuns

### O n8n não recebe mensagens do Telegram

Verifique estes pontos:

1. O workflow está salvo e ativo.
2. O `Telegram Trigger` está usando a credencial correta do bot.
3. O n8n está acessível por uma URL pública HTTPS configurada em `WEBHOOK_URL`.
4. Depois de mudar `WEBHOOK_URL`, reinicie o n8n e desative/ative o workflow para registrar o webhook novamente no Telegram.
5. Não teste o mesmo bot ao mesmo tempo em modo teste e produção. O Telegram aceita apenas um webhook por bot.
6. Se estiver rodando direto no Windows CMD, defina `WEBHOOK_URL` com `set` antes de executar `n8n start`.
