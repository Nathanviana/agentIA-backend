Aqui está um modelo de **README** que descreve como configurar e usar seu ambiente com **N8N**, **Waha** e **ngrok**. Este arquivo orienta como rodar e configurar o ambiente Docker, incluindo detalhes sobre as URLs, variáveis de ambiente e como fazer o teste. Além do fluxo do agente no n8n.

---

# **README - Integração N8N, Waha e ngrok**

Este repositório configura um ambiente com **N8N**, **Waha** e **ngrok** usando **Docker Compose**. O objetivo é integrar o **Waha** para enviar mensagens via WhatsApp ao **N8N** usando webhooks expostos pelo **ngrok**.

## **Pré-requisitos**

Antes de iniciar, certifique-se de que você tenha o seguinte instalado:

* **Docker** e **Docker Compose**
* **Conta no ngrok** (para gerar o token ngrok)

## **Configuração**

1. **Clone este repositório** (se ainda não o fez):

   ```bash
   git clone https://github.com/Nathanviana/agentIA-backend.git
   ```

2. **Configure o arquivo `.env`**

   No diretório do projeto, crie um arquivo `.env` para definir suas variáveis de ambiente. Este arquivo deve conter:

   ```env
   URL=https://from-ngrok.ngrok-free.app # URL pública do ngrok para o N8N
   TIMEZONE=America/Sao_Paulo
   NGROK_TOKEN=<Seu_Token_Ngrok>  # Token do ngrok para autenticação
   ```

3. **Verifique o arquivo `ngrok.yml`**

   Certifique-se de que o arquivo `ngrok.yml` esteja configurado corretamente. Ele deve estar configurado para criar os túneis desejados para **N8N** e **Waha**. Aqui está um exemplo de como deve ser:

   ```yaml
   version: 2
   log_level: debug
   tunnels:
     n8n:
       proto: http
       addr: n8n:5678
       domain: # UR pública do ngrok para o N8N
     waha:
       proto: http
       addr: waha:3000
   ```

## **Rodando o Docker Compose**

1. **Inicie os contêineres**:

   Com o arquivo `.env` configurado, inicie os contêineres usando o **Docker Compose**:

   ```bash
   docker-compose up -d --build
   ```

2. **Acesse os serviços**:

   * **N8N** estará disponível em: <URL diponibilizada pelo ngrok>
   * **Waha** estará disponível em: <URL diponibilizada pelo ngrok>

   **ngrok** cria túneis para expor essas URLs publicamente, e você pode acessá-las diretamente.

## **Configuração do Webhook**

### **N8N**

* No **N8N**, o webhook será configurado automaticamente para responder a eventos enviados pelo **Waha**.
* **URL do Webhook** para o **N8N** (em caso de necessidade de personalização):

  ```yaml
  WEBHOOK_URL=https://mammoth-healthy-coral.ngrok-free.app/webhook/webhook
  ```

### **Waha**

* Para o **Waha**, você precisa configurar a variável de ambiente `WHATSAPP_HOOK_URL` para a URL do **N8N** exposta pelo **ngrok**. Como exemplo, se o **ngrok** gerar a URL `https://mammoth-healthy-coral.ngrok-free.app`, a configuração no **Waha** seria:

  ```yaml
  WHATSAPP_HOOK_URL=${URL}
  ```

* A configuração completa para o **Waha** no `docker-compose.yml` é:

  ```yaml
  waha:
    container_name: waha
    image: devlikeapro/waha:latest
    environment:
      WHATSAPP_HOOK_URL: https://mammoth-healthy-coral.ngrok-free.app/webhook/webhook  # URL do N8N
      WHATSAPP_DEFAULT_ENGINE: GOWS
      WHATSAPP_HOOK_EVENTS: message
    networks:
      - n8n-network
    volumes:
      - waha_sessions:/app/.sessions
      - waha_media:/app/.media
  ```

## **Testando a Configuração**

### **Verifique o ngrok**

Após rodar os contêineres, acesse os logs do **ngrok** para garantir que os túneis foram configurados corretamente:

```bash
docker logs ngrok
```

Você deve ver algo como:

```
Forwarding                    https://mammoth-healthy-coral.ngrok-free.app -> http://n8n:5678
Forwarding                    https://32a24120b792.ngrok-free.app -> http://waha:3000
```

Isso indica que o **ngrok** está corretamente redirecionando o tráfego para o **N8N** e o **Waha**.

### **Testar o Webhook no N8N**

Tente acessar o webhook do **N8N** diretamente com `curl` para verificar se ele responde corretamente:

```bash
curl -X GET https://mammoth-healthy-coral.ngrok-free.app/webhook/webhook
```

Se a URL do webhook estiver configurada corretamente no **Waha** e o fluxo de trabalho estiver ativo no **N8N**, você deve ver uma resposta bem-sucedida.

### **Testar o Webhook no Waha**

Envia uma mensagem ou evento de teste do **Waha** para verificar se o **N8N** está recebendo os webhooks corretamente.

## **Problemas Comuns e Soluções**

* **Timeout (ETIMEDOUT)**: Isso pode acontecer se a URL do webhook não estiver configurada corretamente ou se o **ngrok** não estiver gerando o túnel corretamente. Certifique-se de que os contêineres estão na mesma rede e que a URL do webhook no **Waha** aponta para a URL pública do **ngrok**.

* **404 - Webhook não registrado**: Certifique-se de que o fluxo de trabalho no **N8N** que contém o webhook esteja **ativo** e tenha sido executado ao menos uma vez.

## **Variáveis de Ambiente**

As variáveis de ambiente necessárias para configurar o projeto incluem:

* **`URL`**: URL pública do ngrok para o **N8N**.
* **`TIMEZONE`**: Fuso horário para os serviços.
* **`NGROK_TOKEN`**: Token de autenticação do ngrok.

## **Comandos Importantes**

* **Para parar os contêineres**:

  ```bash
  docker-compose down
  ```

* **Para rodar os contêineres novamente**:

  ```bash
  docker-compose up -d
  ```

* **Para visualizar os logs de um contêiner**:

  ```bash
  docker logs <nome_do_contêiner>
  ```

## **Conclusão**

Com essa configuração, você terá **N8N**, **Waha** e **ngrok** rodando juntos, com o **ngrok** expondo o **N8N** e o **Waha** publicamente. Siga os passos para configurar corretamente as URLs do webhook e garantir que o fluxo de trabalho esteja ativo para que tudo funcione conforme esperado.

Se tiver mais dúvidas ou problemas, fique à vontade para perguntar!
