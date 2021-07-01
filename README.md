## web socket
# 1. Visão Geral
Este post mostra como implementar um servidor WebSocket em Java usando a anotação @ServerEndpoint. Um aplicativo de servidor WebSocket pode ser implementado no Tomcat 7 ou superior, ou em qualquer outro contêiner de servlet Java EE que suporte WebSockets. Existem dois pacotes para programação WebSocket:

- javax.websocket - APIs comuns ao cliente e ao servidor;
- javax.websocket.server - APIs usadas apenas por aplicativos do lado do servidor.

WebSocket é uma tecnologia para estabelecer um canal de comunicação full-duplex persistente, de baixa latência e em uma única conexão http para troca de dados em tempo real entre um endpoint de servidor (Java, .NET, PHP etc.) e um cliente (HTML5 / JavaScript , iOS).

- O protocolo WebSocket é um padrão proposto pela IETF conforme descrito em RFC6455;
- O protocolo WebSocket define 2 novos esquemas de URI: ws e wss para criptografia TLS;
- Os URIs do servidor WebSocket suportam parâmetros de consulta como em: ws://hostname:8080/AppContext/endpoint?UserId=12345&location=London;
- A API Java WebSocket é executada em containers Servlet como Tomcat, JBoss e Websphere;
- Consulte JSR 356 da Oracle para obter detalhes de especificação da API Java para WebSocket.
- O W3C mantém a especificação da API WebSocket JavaScript e define a interface WebSocket;
- Os navegadores compatíveis com HTML5 fornecem uma implementação da especificação para permitir que os clientes se conectem a um servidor WebSocket e enviem e recebam dados (IE10 +, Chrome 16+, Firefox 11+, Safari 6+).

Para obter um exemplo de como criar um cliente WebSocket em JavaScript e HTML 5, consulte a postagem abaixo:

# 2. Criar um cliente WebSocket em JavaScript

### 2.1. Resumo
- Implementação de WebSocket Server em Java;
- Rodando o WebSocket Server no Tomcat;
- Testar o endpoint do servidor WebSocket a partir do navegador da Web;
- Envio automático de notificações para clientes WebSocket;
- Monitoramento do tráfego do WebSocket com as ferramentas do desenvolvedor Chrome.

### 2.2. Implementação de WebSocket Server em Java

Abaixo está o código-fonte Java para a implementação do terminal do servidor WebSocket. Na linha 10, a anotação @ServerEndpoint é usada para decorar uma classe que implementa um endpoint de servidor WebSocket. Mais quatro anotações de método são usadas para decorar manipuladores de eventos para conexões de cliente WebSocket.

```
@OnOpen - Chamado quando um cliente se conecta
@OnClose - Chamado quando uma conexão do cliente é fechada
@OnMessage - Chamado quando uma mensagem é recebida pelo cliente
@OnError - Chamado quando ocorre um erro para este endpoint
Esses quatro métodos são chamados pelo contêiner.
```

```
import java.io.IOException;
import java.util.List;
import java.util.Map;
import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;
@ServerEndpoint("/endpoint")
public class MyWebSocket {
    
    private static PushTimeService pst;
    @OnOpen
    public void onOpen(Session session) {
        System.out.println("onOpen::" + session.getId());        
    }
    @OnClose
    public void onClose(Session session) {
        System.out.println("onClose::" +  session.getId());
    }
    
    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("onMessage::From=" + session.getId() + " Message=" + message);
        
        try {
            session.getBasicRemote().sendText("Hello Client " + session.getId() + "!");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    @OnError
    public void onError(Throwable t) {
        System.out.println("onError::" + t.getMessage());
    }
}
```

Esta configuração permite executar o aplicativo do servidor WebSocket no Tomcat de dentro do Eclipse. A imagem acima mostra a estrutura básica do projeto dinâmico da web no Project Explorer. Observe o arquivo tomcat-websocket.jar na biblioteca de tempo de execução do servidor Apache Tomcat v9.0. Ele contém a API Java para suporte WebSocket.

# 3. Executando o Servidor WebSocket no Tomcat
Para exportar o aplicativo do servidor WebSocket para que ele possa ser implantado em um servidor Tomcat externo, consulte a postagem abaixo sobre a criação de um arquivo WAR:

Eclipse Neon - Exportar Projeto Dinâmico da Web para Arquivo WAR
Aqui, o aplicativo da web é executado de dentro do Eclipse usando a configuração de projeto da web dinâmica mencionada anteriormente. Nenhuma mudança na configuração do Tomcat é necessária para executar um ponto de extremidade do servidor WebSocket. O número da porta será o mesmo usado para conexões por meio do protocolo http, por exemplo, 8080.

# 4. Executando à Aplicação
Se não houver erros, o servidor WebSocket está funcionando e pronto para aceitar conexões de clientes. O URL do endpoint do servidor WebSocket para acessá-lo de uma máquina local seria:

```
ws://127.0.0.1:8080 WebSocketServer/endpoint
```

# 5. Testando o endpoint do servidor WebSocket no navegador da web
O código-fonte JavaScript fornecido a seguir cria uma classe chamada WebSocketClient que implementa um cliente WebSocket básico.

```
class WebSocketClient {
    
    constructor(protocol, hostname, port, endpoint) {
        
        this.webSocket = null;
        
        this.protocol = protocol;
        this.hostname = hostname;
        this.port     = port;
        this.endpoint = endpoint;
    }
    
    getServerUrl() {
        return this.protocol + "://" + this.hostname + ":" + this.port + this.endpoint;
    }
    
    connect() {
        try {
            this.webSocket = new WebSocket(this.getServerUrl());
            
            // 
            // Implement WebSocket event handlers!
            //
            this.webSocket.onopen = function(event) {
                console.log('onopen::' + JSON.stringify(event, null, 4));
            }
            
            this.webSocket.onmessage = function(event) {
                var msg = event.data;
                console.log('onmessage::' + JSON.stringify(msg, null, 4));
            }
            this.webSocket.onclose = function(event) {
                console.log('onclose::' + JSON.stringify(event, null, 4));                
            }
            this.webSocket.onerror = function(event) {
                console.log('onerror::' + JSON.stringify(event, null, 4));
            }
            
        } catch (exception) {
            console.error(exception);
        }
    }
    
    getStatus() {
        return this.webSocket.readyState;
    }
    send(message) {
        
        if (this.webSocket.readyState == WebSocket.OPEN) {
            this.webSocket.send(message);
            
        } else {
            console.error('webSocket is not open. readyState=' + this.webSocket.readyState);
        }
    }
    disconnect() {
        if (this.webSocket.readyState == WebSocket.OPEN) {
            this.webSocket.close();
            
        } else {
            console.error('webSocket is not open. readyState=' + this.webSocket.readyState);
        }
    }
}
```

O código JavaScript pode ser executado com o Chrome (ou qualquer outro navegador), usando suas ferramentas de desenvolvedor integradas. Abra uma nova janela no Chrome, copie o código do cliente JavaScript WebSocket acima e cole-o no console fornecido pelas ferramentas de desenvolvedor do Chrome.

A classe WebSocketClient agora foi definida na página atual. No console, digite o código JavaScript abaixo para criar um novo objeto dessa classe.

```
var client = new WebSocketClient('ws','127.0.0.1',8080,'/WebSocketServer/endpoint');
```

Em seguida, chame o método de conexão para abrir uma nova conexão com o ponto de extremidade do servidor WebSocket. O navegador chamará o manipulador de eventos onopen assim que a conexão for estabelecida.

```
client.connect();
```

Neste ponto, o contêiner que está hospedando o endpoint do servidor WebSocket chamará o método onOpen da implementação do servidor. Neste exemplo, o servidor simplesmente imprime o Id da sessão, que é 0 aqui.

- Veja o método onOpen da classe MyWebSocket.java da seção 1;
- Consulte a documentação do Oracle para obter detalhes sobre javax.websocket.Session.

No lado do cliente, execute o método send (String message) usando o JavaScript abaixo para enviar uma nova mensagem ao servidor.

```
client.send ('Olá servidor!');
```

O servidor retorna uma mensagem de saudação ao cliente, aqui "Hello Client 0!" Consulte a linha 34 da classe MyWebSocket.java da seção 1. Ao mesmo tempo, o contêiner que está hospedando o endpoint do servidor WebSocket chamará o método onMessage da implementação do servidor. O servidor simplesmente imprime a mensagem do cliente no console.

# 6. Envio automático de notificações para clientes WebSocket
Nesta seção, a classe MyWebSocket.java é alterada um pouco para que os clientes possam se inscrever em um serviço de notificação que enviará periodicamente a hora atual do sistema do servidor. Os clientes que desejam receber essas notificações precisam fornecer um parâmetro de string de consulta de URL, conforme mostrado abaixo:

```
ws://127.0.0.1:8080/WebSocketServer/endpoint?push=TIME
```
Novo código foi adicionado ao método onOpen entre as linhas 21 e 32, conforme mostrado abaixo. A classe PushTimeService mantém uma coleção de objetos de Sessão ativos e envia periodicamente o horário atual do servidor para todos os clientes que se inscreveram.

```
import java.io.IOException;
import java.util.List;
import java.util.Map;
import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;
@ServerEndpoint("/endpoint")
public class MyWebSocket {
    @OnOpen
    public void onOpen(Session session) {
        System.out.println("onOpen::" + session.getId());
        
        ///////////////////////////////////////////////////////////////////////////// 
        // Access request parameters from URL query String.
        // If a client subscribes, add Session to PushTimeService. 
        //
        Map<String, List<String>> params = session.getRequestParameterMap();
        
        if (params.get("push") != null && (params.get("push").get(0).equals("TIME"))) {
            
          PushTimeService.initialize();
          PushTimeService.add(session);
        }
        /////////////////////////////////////////////////////////////////////////////
    }
    @OnClose
    public void onClose(Session session) {
        System.out.println("onClose::" +  session.getId());
    }
    
    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("onMessage::From=" + session.getId() + " Message=" + message);
        
        try {
            session.getBasicRemote().sendText("Hello Client " + session.getId() + "!");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    @OnError
    public void onError(Throwable t) {
        System.out.println("onError::" + t.getMessage());
    }
}
```

No cenário a seguir, 2 clientes WebSocket foram iniciados em navegadores diferentes usando a classe WebSocketClient JavaScript da seção 3. Ambos se inscreveram para receber a hora do sistema do servidor.

A classe PushTimeService envia a hora do sistema do servidor para ambos os clientes, a cada 10 segundos.

Abordagens semelhantes podem ser usadas para enviar uma variedade de dados orientados a eventos do servidor para os clientes em tempo real. Uma vez que a conexão WebSocket do cliente reside na camada de interface do usuário, os WebSockets são uma boa solução (alternativa à pesquisa) para conduzir componentes de interface do usuário em tempo real, como cotações da bolsa ou notificações

# 7. Monitorando o tráfego do WebSocket com as ferramentas do desenvolvedor do Chrome
As Ferramentas para Desenvolvedores do Chrome fornecem meios para o monitoramento básico do tráfego do WebSocket. Use a guia Rede e filtre o tráfego com o botão WS para mostrar apenas WebSockets. A guia Cabeçalhos mostra a solicitação http do cliente e a resposta do servidor. Os parâmetros da string de consulta estão listados na parte inferior.

Observe que o servidor responde com uma atualização de conexão no cabeçalho de resposta para atualizar a conexão http para uma conexão WebSocket persistente. Consulte o mecanismo de atualização de protocolo para obter detalhes sobre este processo. A guia Frames mostra todas as mensagens WebSocket enviadas e recebidas. Clique em um quadro para ver seu conteúdo.

A guia Timig mostra há quanto tempo a conexão WebSocket está aberta. No exemplo abaixo, ele está aberto há 4,4 minutos.
