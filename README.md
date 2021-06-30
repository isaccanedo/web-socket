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
