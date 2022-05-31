# STOMP
## simple broker 사용 시 work flow
<img src="./../img/stomp-flow.png">

```java
@Configuration
@EnableWebSocketMessageBroker
@RequiredArgsConstructor
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Value("${origins}")
    private String origins;

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // SockJS을 연결할 endpoint url
        registry.addEndpoint("/stomp")
                .setAllowedOrigins(origins)
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // 클라이언트가 메세지를 보낼 때의 url prefix
        registry.setApplicationDestinationPrefixes("/chat")
        // 클라이언트가 subscribe 할 때 붙는 url prefix
        // 메세지 브로커는 url처럼 이름이 붙는데 메세지 브로커 이름이 subs라 보면 된다.
                .enableSimpleBroker("/subs");
    }
}
```
* WebSocketMessageBrokerConfigurer를 구현하여 STOMP를 사용할 수 있는데 registerStompEndpoints()의 addEndpoint()를 통해 어느 url로 SockJS를 연결할 지 정하고, configureMessageBroker()의 setApplicationDestinationPrefixes()를 통해 클라이언트가 메세지를 보낼 때의 url prefix와 enableSimpleBroker()를 통해 메세지 브로커의 이름을 정한다.
```java
@Controller
public class ChatController {

    // topic
    @MessageMapping("/room")
    // /subs 메세지 브로커에 room 토픽으로 메세지를 내림
    // 메세지가 들어올 때는 /chat/room으로 들어오지만 메세지 브로커에는 /subs/room으로 메세지를 내린다.(정확히는 브로커 채널에)
    @SendTo("/subs/room")
    public String message(String msg) {
        return msg;
    }
}
```
```typescript
import { useEffect, useRef, useState } from "react";
import SockJS from "sockjs-client";
import Stomp from "stompjs";

function ChatForm() {
    const stomp = useRef<Stomp.Client>();
    const [receivedMsg, setReceivedMsg] = useState<string[]>([]);
    const [msg, setMsg] = useState("");
    useEffect(() => {
        const ws = new SockJS("http://localhost:8080/stomp");
        stomp.current = Stomp.over(ws);
        stomp.current.connect({}, () => {
            // /subs/ 브로커의 room 토픽을 susbscribe
            stomp.current?.subscribe("/subs/room", (message) => {
                setReceivedMsg(receivedMsg => [message.body, ...receivedMsg]);
            });
        });
        return () => {
            stomp.current?.disconnect(() => {});
        }
    }, []);
    const onSubmit = (event: React.FormEvent<HTMLFormElement>) => {
        event.preventDefault();
        // 메세지는 /chat/room 으로 보낸다.
        stomp.current?.send("/chat/room", {}, msg);
        setMsg("");
    }
    const onChange = (event: React.ChangeEvent<HTMLInputElement>) => {
        setMsg(event.target.value);
    }

    return (
        <div>
            <ul>
                {receivedMsg.map((msg, idx) => <li key={idx}>{msg}</li>)}
            </ul>
            <form onSubmit={onSubmit}>
                <input type="text" placeholder='메세지 입력' onChange={onChange} value={msg} required/>
                <button>보내기</button>
            </form>
        </div>
    );
}

export default ChatForm;
```
* 구독
  1. 클라이언트는 서버와 SockJS연결을 한다.
  2. 연결이 완료되면 클라이언트는 서버에게 원하는 topic에 대한 subscribe frame을 보내서 해당 토픽에 대한 subscribe를 한다.
* 메세지 전달
  1. 클라이언트는 특정 topic에 대한 메세지를 보낸다.
  2. 서버에서는 해당 topic을 처리하는 메세지 핸들러가 해당 메세지를 받아 ApplicationDestinationPrefix를 메세지 브로커의 이름으로 바꾼 후 브로커 채널에 내려보낸다.
  3. 해당 이름을 가진 메세지 브로커가 topic을 subscribe 하고있는 클라이언트에게 응답을 보낸다.
## 외부 메세지 브로커 사용 시 work flow(RabbitMQ)
<img src="./../img/message-flow-broker-relay.png">

```java
@Configuration
@EnableWebSocketMessageBroker
@RequiredArgsConstructor
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Value("${origins}")
    private String origins;

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/stomp")
                .setAllowedOrigins(origins)
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry
                .setPathMatcher(new AntPathMatcher("."))
                .setApplicationDestinationPrefixes("/chat")
                .enableStompBrokerRelay("/queue", "/topic", "/exchange", "/amq/queue")
                .setRelayHost("localhost")
                .setRelayPort(61613)
                .setClientLogin("guest")
                .setClientPasscode("guest");
    }
}
```
* STOMP destination
  <img style="margin-left:30px" src="./../img/exchange.png">

  1. __/exchange__ : /exchange/{exchange_name}/{routing_key} {exchange_name}으로 subscribe frame을 보낸다. 새로 큐를 생성하며 {routing_key}로 바인딩 된다. `/exchange/chat.exchange/room.1`로 subscribe frame 을 보내면 새로 큐가 생성되면서 위와 같이 exchange와 routing key가 바인딩 된다.
  <img src="./../img/queue.png">
  <img src="./../img/queue_not_created.png">

  2. __/queue__ : /queue/{queue_name} 으로 subscribe frame을 보낸다. 현재 rabbiMQ가 관리하는 큐에 대해 subscribe를 하고 큐를 새로 생성하지는 않는다. exchange는 default exchange를 사용한다. `/queue/chat.queue`로 subscribe frame을 보내면 위의 이미지 처럼 큐가 새로 생성되지 않고 이미 존재하는 queue였던 chat.queue를 subscribe하게 된다.

  <img style="margin-left:30px" src="./../img/topic.png">

  3. __/topic__ : /topic/{routing_key} 으로 subscribe frame을 보낸다. 큐를 새로 생성하며 exchange는 amq.topic을 사용하고 {routing_key}로 바인딩 된다. 위의 이미지는 `/topic/room.1`로 subscribe frame을 보냈다.
  4. __/amq/queue__ : /amq/queue/{queue_name} 으로 subscribe frame을 보낸다. 돌려보니깐 /queue/{queue_name}랑 똑같이 동작한다. 레퍼런스를 보니깐 STOMP gateway 바깥에서 만들어진 queue를 구독한다는 걸 보니 /queue/{queue_name}의 경우에는 서버에서 직접 만들어진 queue를 구독할 때 쓰는 것 같고, /amq/queue/{queue_name}는 서버 외부에서 만들어진 queue(위의 /topic 이나 /exchange 같은걸로 만들어진거)를 구독할 때 쓰이는 것 같다.
```java
@Component
@Slf4j
public class MessageListenerLog implements MessageListener {

    @Override
    public void onMessage(Message message) {
        log.info("received message : {}", message);
    }
}
```
```java
@Configuration
@RequiredArgsConstructor
public class RabbitConfig {

    private static final String chatQueueName = "chat.queue";

    private static final String chatExchangeName = "chat.exchange";

    private static final String routingKey = "room.*";

    private final MessageListener messageListener;

    //Queue 등록
    @Bean
    public Queue queue(){
        return new Queue(chatQueueName, true);
    }

    //Exchange 등록
    @Bean
    public TopicExchange exchange(){
        return new TopicExchange(chatExchangeName);
    }

    //Exchange 와 Queue 바인딩
    @Bean
    public Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(routingKey);
    }

    @Bean
    public RabbitTemplate rabbitTemplate(){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory());
        rabbitTemplate.setMessageConverter(jsonMessageConverter()); // 객체 전달 시 필요
        return rabbitTemplate;
    }

    // 여러 큐의 메세지를 받아와야하는 메세지 리스너는 MessageListenerContainer 를 통해 등록하는 것이 좋다.
    @Bean
    public SimpleMessageListenerContainer container(){
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory()); // connectionFactory 의 설정대로 rabbitmq 와 연결됨
        container.setQueueNames(chatQueueName); // 메세지 리스너가 해당하는 큐를 listen 함, 여러 큐 지정 가능
        container.setMessageListener(messageListener); // 메세지 리스너 등록
        return container;
    }

    @Bean
    public ConnectionFactory connectionFactory(){
        CachingConnectionFactory factory = new CachingConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("guest");
        factory.setPassword("guest");
        return factory;
    }

    @Bean
    public Jackson2JsonMessageConverter jsonMessageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```
```java
@Controller
@RequiredArgsConstructor
@Slf4j
public class ChatController {

    private static final String chatQueueName = "chat.queue";

    private static final String chatExchangeName = "chat.exchange";

    private static final String routingKey = "room.*";


    private final RabbitTemplate rabbitTemplate;

    @MessageMapping("room.{roomId}")
    public void message(String msg, @DestinationVariable String roomId) {
        rabbitTemplate.convertAndSend(chatExchangeName, "room." + roomId, msg);
    }

//     각각 다른 큐의 메세지를 listen 하는 메세지 리스너를 등록할 때는 아래와 같은 방법으로 등록함
//    @RabbitListener(bindings = @QueueBinding(
//            exchange = @Exchange(name = chatExchangeName, type = ExchangeTypes.TOPIC),
//            value = @Queue(name = chatQueueName),
//            key = routingKey
//    ))
//    public void messageLog(String message){
//        log.info("received message : {}", message);
//    }
}
```
```typescript
import { useEffect, useRef, useState } from "react";
import SockJS from "sockjs-client";
import Stomp from "stompjs";
import styled from "styled-components";

const InputPanel = styled.div`
    position: fixed;
    left: 0%;
    right: 0%;
    bottom: 0%;
`;

const MsgContainer = styled.div`
    padding-bottom: 10px;
`

function ChatForm() {
    const stomp = useRef<Stomp.Client>();
    const [receivedMsg, setReceivedMsg] = useState<string[]>([]);
    const [msg, setMsg] = useState("");
    useEffect(() => {
        const ws = new SockJS("http://localhost:8080/stomp");
        stomp.current = Stomp.over(ws);
        stomp.current.connect({}, () => {
            stomp.current?.subscribe(`/exchange/chat.exchange/room.1`, (message) => {
                setReceivedMsg(receivedMsg => [...receivedMsg, message.body]);
            });
        });
        return () => {
            stomp.current?.disconnect(() => {});
        }
    }, []);
    useEffect(() => {
        window.scrollBy({top: 100});
    }, [receivedMsg])
    const onSubmit = (event: React.FormEvent<HTMLFormElement>) => {
        event.preventDefault();
        stomp.current?.send(`/chat/room.1`, {}, msg);
        setMsg("");
    }
    const onChange = (event: React.ChangeEvent<HTMLInputElement>) => {
        setMsg(event.target.value);
    }

    return (
        <div>
            <MsgContainer>
                {receivedMsg.map((msg, idx) => 
                <div key={idx}>
                    <span>{msg}</span>
                </div>)}
            </MsgContainer>
            <InputPanel>
                <form onSubmit={onSubmit}>
                    <input type="text" placeholder='메세지 입력' onChange={onChange} value={msg} required/>
                    <button>보내기</button>
                </form>
            </InputPanel>
        </div>
    );
}

export default ChatForm;
```
* 구독
  1. 클라이언트는 서버와 SockJS연결을 한다.
  2. 연결이 완료되면 클라이언트는 서버에게 subscribe frame을 보낸다.(구독)
* 메세지 전달
  1. 클라이언트는 특정 routing key로 메세지를 보낸다.
  2. 서버에서는 해당 routing key를 처리하는 메세지 핸들러가 해당 메세지를 받아 ApplicationDestinationPrefix를 exchange 이름으로 바꾼 후 브로커 채널에 내려보낸다.
  3. 메세지 브로커는 binding에 따라 해당 메세지를 queue에 넣는다.(정확히는 메세지전체를 넣는게 아니라 참조만 넣는다.)
  4. queue에 저장된 메세지는 해당 queue를 구독하는 consumer에게 전달된다.