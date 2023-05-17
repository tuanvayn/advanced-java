# What are microservices? How do microservices communicate independently of each other?

## What are microservices

-   Microservice architecture is a distributed system, which is divided into different service units according to business to solve the shortcomings of single system performance.
-   Microservices is an architectural style, where a large software application consists of multiple service units. The service units in the system can be deployed separately, and the service units are loosely coupled to each other.

> The origin of the microservices concept：[Microservices](https://martinfowler.com/articles/microservices.html)

## How microservices communicate independently of each other

### synchronous

#### REST HTTP agreement

REST requests are one of the most commonly used means of communication in microservices and rely on the HTTP\HTTPS protocol. RESTFUL FEATURES ARE:

1. Each URI represents 1 resource
2. The client uses GET, POST, PUT, DELETE 4 verbs that indicate the operation mode to operate on server-side resources: GET is used to obtain resources, POST is used to create new resources (can also be used to update resources), PUT is used to update resources, and DELETE is used to delete resources
3. Manipulate resources by manipulating their representations
4. Resources are represented in XML or HTML
5. The interaction between the client and the server is stateless between requests, and each request from the client to the server must contain the information necessary to understand the request

For example, a server provides the following interface:

```java
@RestController
@RequestMapping("/communication")
public class RestControllerDemo {
    @GetMapping("/hello")
    public String s() {
        return "hello";
    }
}
```

Another service needs to call the interface, and the caller only needs to send a request according to the API document to get the return result.

```java
@RestController
@RequestMapping("/demo")
public class RestDemo{
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/hello2")
    public String s2() {
        String forObject = restTemplate.getForObject("http://localhost:9013/communication/hello", String.class);
        return forObject;
    }
}
```

In this way, communication between services can be achieved.

#### RPC TCP protocol

RPC (Remote Procedure Call) remote procedure call, simply understood as a node requesting a service provided by another node. Its workflow is like this

1. Execute the client call statement and pass parameters
2. Call the local system to send network messages
3. The message is delivered to the remote host
4. The server gets the message and gets the parameters
5. Execute remote procedures (services) based on invocation requests and parameters
6. When the execution process is complete, the result is returned to the server handle
7. The server handle returns the result, and the remote host's system network service is called to send the result
8. The message is passed back to localhost
9. The client handle receives messages by the local host's network service
10. The client receives the result data returned by the invocation statement

Take an example.

First of all, you need a server：

```java
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Method;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * RPC The interface and implementation class used by the server to register remote methods
 */
public class RPCServer {
    private static ExecutorService executor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

    private static final ConcurrentHashMap<String, Class> serviceRegister = new ConcurrentHashMap<>();

    /**
     * Register the method
     * @param service
     * @param impl
     */
    public void register(Class service, Class impl) {
        serviceRegister.put(service.getSimpleName(), impl);
    }

    /**
     * Start method
     * @param port
     */
    public void start(int port) {
        ServerSocket socket = null;
        try {
            socket = new ServerSocket();
            socket.bind(new InetSocketAddress(port));
            System.out.println("Service starts");
            System.out.println(serviceRegister);
            while (true) {
                executor.execute(new Task(socket.accept()));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (socket != null) {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private static class Task implements Runnable {
        Socket client = null;

        public Task(Socket client) {
            this.client = client;
        }

        @Override
        public void run() {
            ObjectInputStream input = null;
            ObjectOutputStream output = null;
            try {
                input = new ObjectInputStream(client.getInputStream());
                // Read what the other party wrote in order
                String serviceName = input.readUTF();
                String methodName = input.readUTF();
                Class<?>[] parameterTypes = (Class<?>[]) input.readObject();
                Object[] arguments = (Object[]) input.readObject();
                Class serviceClass = serviceRegister.get(serviceName);
                if (serviceClass == null) {
                    throw new ClassNotFoundException(serviceName + " Not found!");
                }
                Method method = serviceClass.getMethod(methodName, parameterTypes);
                Object result = method.invoke(serviceClass.newInstance(), arguments);

                output = new ObjectOutputStream(client.getOutputStream());
                output.writeObject(result);
            } catch (Exception e) {
                e.printStackTrace();

            } finally {
                try {
                    // I don't write output!=null here to close this logic
                    output.close();
                    input.close();
                    client.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }

}

```

The second is a client：

```java
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.net.InetSocketAddress;
import java.net.Socket;

/**
 * RPC client
 */
public class RPCclient<T> {
    /**
     * The parameters are sent to RPCServer through the dynamic proxy, and the RPCserver returns the result as the correct entity
     */
    public static <T> T getRemoteProxyObj(final Class<T> service, final InetSocketAddress addr) {

        return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[]{service}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                Socket socket = null;
                ObjectOutputStream out = null;
                ObjectInputStream input = null;
                try {
                    socket = new Socket();
                    socket.connect(addr);

                    // Send entity classes, parameters, to remote callers
                    out = new ObjectOutputStream(socket.getOutputStream());
                    out.writeUTF(service.getSimpleName());
                    out.writeUTF(method.getName());
                    out.writeObject(method.getParameterTypes());
                    out.writeObject(args);

                    input = new ObjectInputStream(socket.getInputStream());
                    return input.readObject();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    out.close();
                    input.close();
                    socket.close();
                }
                return null;
            }
        });

    }

}

```

Let's test the remote method again.。

```java
public interface Tinterface {
    String send(String msg);
}

public class TinterfaceImpl implements Tinterface {
    @Override
    public String send(String msg) {
        return "send message " + msg;
    }
}

```

The test code is as follows：

```java
import java.net.InetSocketAddress;


public class RunTest {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                RPCServer rpcServer = new RPCServer();
                rpcServer.register(Tinterface.class, TinterfaceImpl.class);
                rpcServer.start(10000);
            }
        }).start();
        Tinterface tinterface = RPCclient.getRemoteProxyObj(Tinterface.class, new InetSocketAddress("localhost", 10000));
        System.out.println(tinterface.send("rpc Test cases"));

    }
}

```

Output 'send message RPC test case'.

### asynchronous

#### Message middleware

Common message middleware is Kafka, ActiveMQ, RabbitMQ, RocketMQ, and common protocols are AMQP, MQTTP, STOMP, XMPP. The message queue is not expanded here, please go to the official website for how to use it.
