## web-socket
# 1. Introdução
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
