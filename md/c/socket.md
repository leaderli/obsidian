# socket

编写一个简单色server服务，可以用来调试测试各种网络请求

```java
package com.leaderli.demo;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class SockeServerTest {
    public static void main(String[] args) throws IOException {

        ServerSocket ss = new ServerSocket(3333);
        while (true) {
            Socket client = ss.accept();
            BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
            PrintWriter out = new PrintWriter(client.getOutputStream());
            out.print("HTTP/1.1 200 \r\n");
            out.print("\r\n");
            String line;
            while ((line = in.readLine()) != null) {
                if (line.length() == 0)
                    break;
                out.print(line + "\r\n");
            }

            out.close();
            in.close();
            client.close();
        }
    }

}

```

下面是一个简单的http请求的测试

```java
package com.leaderli.demo;

import org.springframework.boot.web.client.RestTemplateBuilder;
import sun.net.www.http.HttpClient;

import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.HashMap;
import java.util.Map;

public class RestTemplateTest {

    public static void main(String[] args) throws IOException {
        URL url = new URL("http://localhost:3333");
        HttpURLConnection con = (HttpURLConnection) url.openConnection();
        con.setRequestMethod("GET");

        con.setDoOutput(true);
        DataOutputStream out = new DataOutputStream(con.getOutputStream());
        out.writeBytes("a=1");


        out.flush();
        int status = con.getResponseCode();
        System.out.println("status:"+status);
        BufferedReader in = new BufferedReader(
                new InputStreamReader(con.getInputStream()));
        String inputLine;
        StringBuffer content = new StringBuffer();
        while ((inputLine = in.readLine()) != null) {
            content.append(inputLine);
        }
        System.out.println("content:\r\n"+content);
        in.close();
        out.close();
        con.disconnect();
    }
}

```

上述代码输出

> GET / HTTP/1.1User-Agent: Java/1.8.0_281Host: localhost:3333Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2Connection: keep-alive

当使用浏览器访问时 `http://localhost:3333`
> GET / HTTP/1.1
Host: localhost:3333
Connection: keep-alive
Cache-Control: max-age=0
sec-ch-ua: " Not;A Brand";v="99", "Google Chrome";v="91", "Chromium";v="91"
sec-ch-ua-mobile: ?0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) > Chrome/91.0.4472.164 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7

当使用命令行访问时 `curl http://localhost:3333`

>  GET / HTTP/1.1
Host: localhost:3333
User-Agent: curl/7.77.0
Accept: */*

所以当我调试一些被封装的比较深的工具类时，可以模拟一个服务端，然后观察这些工具类实际发送请求时的细节。