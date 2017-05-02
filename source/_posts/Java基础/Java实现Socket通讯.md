---
title: Java实现Socket通讯
date: 2017-04-10 21:46:55
tags: [Java基础]
---

## Socket 服务端

```java
/**
 * Socket服务端
 * @author NikoBelic
 * @create 2017/4/10 20:51
 */
public class SocketServer
{
    public static void main(String[] args) throws IOException
    {
        // 创建Socket服务端，绑定到本地8899端口
        ServerSocket serverSocket = new ServerSocket();
        serverSocket.bind(new InetSocketAddress("localhost", 8899));
        // 使用线程池异步处理业务逻辑（否则将不支持多客户端）
        ExecutorService threadPool = new ThreadPoolExecutor(0, 3,
                60L, TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(2));
        Socket socket;
        while (true)
        {
            // 接收客户端请求（阻塞方法）
            socket = serverSocket.accept();
            // 创建线程 处理业务逻辑
            threadPool.execute(new SocketTask(socket));
        }
    }
}

```

<!--more-->

## Socket 服务端业务逻辑处理

```java
/**
 * 业务逻辑处理线程
 * @author NikoBelic
 * @create 2017/4/10 21:16
 */
public class SocketTask implements Runnable
{
    private Socket socket;

    public SocketTask(Socket socket)
    {
        this.socket = socket;
    }

    @Override
    public void run()
    {
        InputStream in = null;
        OutputStream out = null;
        PrintWriter pw;
        BufferedReader reader;
        String clientStrs;
        try
        {
            // 从套接字中获取输入流（客户端传输的数据流）
            in = socket.getInputStream();
            reader = new BufferedReader(new InputStreamReader(in));
            // 从套接字中获取输出流（回传给客户端的数据流）
            out = socket.getOutputStream();
            pw = new PrintWriter(out);
            // 从输入流中读取数据（注意：readline是阻塞方法）
            while ((clientStrs = reader.readLine()) != null)
            {
                System.out.println("Server端接收到:" + clientStrs);
                // 给客户端回传的内容
                pw.println("我收到了:" + clientStrs);
                pw.flush();
            }
        } catch (IOException e)
        {
            e.printStackTrace();
        }finally
        {
            try
            {
                in.close();
                out.close();
                socket.close();
            } catch (IOException e)
            {
                e.printStackTrace();
            }
        }

    }
}

```

## Socket 客户端

```java
/**
 * Socket客户端
 * @author NikoBelic
 * @create 2017/4/10 21:02
 */
public class SocketClient
{
    public static void main(String[] args) throws IOException
    {
        // 创建套接字，与服务端进行通讯
        Socket socket = new Socket("localhost", 8899);
        // 从套接字中获得输出流（向服务端发送的数据流）
        OutputStream outputStream = socket.getOutputStream();
        PrintWriter pw = new PrintWriter(outputStream);
        // 从套接字中获得输入流（服务端回传给客户端的数据流）
        InputStream inputStream = socket.getInputStream();
        Scanner scanner = new Scanner(System.in);
        String input;
        while ((input = scanner.nextLine()) != "end")
        {
            // 向服务端发送数据流
            pw.println(input);
            pw.flush();

            // 读取端返回的处理结果
            System.out.println(new BufferedReader(new InputStreamReader(inputStream)).readLine());
        }
    }
}

```

## 测试

客户端Console

```commandline
你好啊
我收到了:你好啊
Java网络编程
我收到了:Java网络编程
不错哦！！！嘿嘿嘿！
我收到了:不错哦！！！嘿嘿嘿！
```

服务端Console
```commandline
Server端接收到:你好啊
Server端接收到:Java网络编程
Server端接收到:不错哦！！！嘿嘿嘿！
```