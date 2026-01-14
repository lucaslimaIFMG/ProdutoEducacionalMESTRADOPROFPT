
1. Visão Geral

O projeto TeddyBot consiste em uma API desenvolvida em ASP.NET Core que implementa um bot conversacional baseado em uma máquina de estados finitos, modelada por meio de um grafo de conversação. O bot integra-se à WAHA API para envio de mensagens e utiliza um webhook para receber eventos de mensagens dos usuários. Os estados e transições da conversa são definidos de forma declarativa no arquivo appsettings.json, permitindo alterar
o comportamento do bot sem recompilar a aplicação.

2. Objetivos do Projeto
Implementar um bot conversacional baseado em fundamentos teóricos de Ciência da Computação; Permitir a alteração de fluxos conversacionais via configuração; Garantir portabilidade e reprodutibilidade por meio de containers Docker; Facilitar uso acadêmico, institucional e experimental.



3. Arquitetura Geral
A solução é composta pelos seguintes componentes principais:
3.1 ASP.NET Core Web API
Responsável por expor o endpoint de webhook eCentraliza a lógica de orquestração da aplicação.
 3.2 ConversationService
Camada de serviço que processa a lógica da conversação. Interpreta as mensagens recebidas e determina as transições de estado.


3.3 ConversationGraph
Estrutura que representa o grafo de estados da conversa. Contém nós (estados) e transições possíveis.


3.4 WAHA API (WahaWebhookMessage.cs)
Responsável pela comunicação com o WhatsApp. Encaminha mensagens recebidas para o webhook da API. Envia respostas geradas pelo bot aos usuários.


3.5 IMemoryCache
Utilizado para armazenar temporariamente o estado da conversa de cada usuário. Permite múltiplas sessões simultâneas sem conflito.


3.6 appsettings.json
Arquivo central de configuração do fluxo conversacional. Define estados, transições e mensagens.


3.7  Docker e Docker Compose
Responsáveis pela criação e orquestração dos containers. Garantem execução padronizada em qualquer ambiente.
4. Modelagem da Máquina de Estados
O bot é implementado como uma Máquina de Estados Finitos, onde: Cada estado é representado por um ConversationNode; As transições ocorrem a partir: Da entrada do usuário; Ou de uma transição automática (Next); O estado atual do usuário é armazenado em um objeto ConversationState.
5. Definição do Grafo no appsettings.json
Os estados e transições são definidos na seção ConversationGraph do arquivo appsettings.json, carregada na inicialização da aplicação:
```csharp
var conversationGraph = builder.Configuration
    .GetSection("ConversationGraph")
    .Get<ConversationGraph>();

if (conversationGraph != null)
{
    builder.Services.AddSingleton(conversationGraph);
    builder.Services.AddScoped<IConversationService, ConversationService>();
}
````

6. Gerenciamento de Estado
O estado de cada usuário é armazenado temporariamente em memória utilizando IMemoryCache.
Vantagens Suporte a múltiplos usuários simultâneos;  Baixa latência; Simplicidade para ambientes de desenvolvimento e testes. Para ambientes de produção com alta escala, recomenda-se avaliar soluções distribuídas (Redis, por exemplo).

7. Integração com a WAHA API
A WAHA API é responsável por: Enviar mensagens ao usuário via WhatsApp; Encaminhar eventos de mensagens para o webhook configurado para manter sessões do WhatsApp ativas. A API Jornada Heroica atua como intermediária, processando a lógica antes de responder ao usuário.

8. Webhook
Endpoint Principal
POST /webhook
Esse endpoint recebe os eventos enviados pela WAHA API e os encaminha para o serviço de conversação.

9. Execução do Projeto
Pré-requisitos Antes de iniciar, certifique-se de possuir:
Docker 20 ou superior 
Docker Compose v2 ou superior
Acesso à internet (para download de imagens Docker)

10. Docker Compose
O projeto utiliza o seguinte arquivo docker-compose.yml:
Segue abaixo o código contido no arquivo Docker

```yaml
services:
    chatbot:
        restart: unless-stopped
        build:
            context: .
            dockerfile: ./src/JornadaHeroica.Api/Dockerfile
        ports:
            - 8998:8080
        environment:
            WAHA_API_URL: http://waha:3000

    waha:
        image: devlikeapro/waha
        pull_policy: never
        restart: unless-stopped
        ports:
            - 3627:3000
        environment:
            WHATSAPP_HOOK_URL: http://chatbot:8080/webhook
            WHATSAPP_HOOK_EVENTS: message
            WHATSAPP_HOOK_RETRIES_ATTEMPTS: 1
        volumes:
            - /tbf/ifmg-waha:/app/.sessions
```

---

11. Build e Deploy
Comandos para execução:

docker compose build
docker compose up -d

12. Verificação dos Serviços
Após a inicialização: cole os link no navegador 
API Jornada Heroica:
 http://localhost:8998
WAHA API:
 http://localhost:3627

14. Fluxo de Execução em Produção
O usuário envia uma mensagem via WhatsApp;


A WAHA (WahaWebhookMessage.cs ) recebe a mensagem;


A WAHA  (WahaWebhookMessage.cs )dispara o webhook (WebhookController.cs);


A API Jornada Heroica processa a mensagem;


A resposta é enviada ao usuário pela WAHA API.










