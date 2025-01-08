---
title: "Implement small http server"
date: 2023-09-26T18:40:23+09:00
draft: false
---

小さな http サーバを実装する

1. ソケットの作成，ポートのバインド

```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define PORT 9090

int main(){
    int server_fd;
    struct sockaddr_in address;

    // create socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if(server_fd == -1){
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    printf("Socket created successfully\n");

    // bind
    address.sin_family = AF_INET; // IPv4
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    if(bind(server_fd, (struct sockaddr *)&address, sizeof(address)) == -1){
        perror("Bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Bind to port %d successful.\n", PORT);

    close(server_fd);

    return 0;
}
```

```sh
lubuntu@lubuntu-virtualbox:~/linux_tutorial/httpserver$ ./server 
Socket created successfully
Bind to port 9090 successful.
```

何も変化はないがとりあえずエラーが出てないだけ OK

2. リッスンとアクセプト

具体的なレスポンスを確定し，その内容を送信する

```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>

#define PORT 9090

int main(){
    int server_fd, client_fd;
    struct sockaddr_in address;
    int addr_len = sizeof(address);
    char *response = "HTTP/1.1 200 OK\r\nContent-Length: 13\r\n\r\nHello, World!\n";

    // create socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if(server_fd == -1){
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    printf("Socket created successfully\n");

    // bind
    address.sin_family = AF_INET; // IPv4
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    if(bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0){
        perror("Bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Bind to port %d successful.\n", PORT);

    // listen, accept

    if(listen(server_fd, 3) < 0){
        perror("Listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Listening on port %d...\n", PORT);

    client_fd = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addr_len);
    if(client_fd < 0){
        perror("Accept failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Connection accepted\n");

    send(client_fd, response, strlen(response), 0);
    printf("Response sent to client\n");

    close(server_fd);
    close(client_fd);

    return 0;
}
```

```sh
lubuntu@lubuntu-virtualbox:~/linux_tutorial/httpserver$ ./server 
Socket created successfully
Bind to port 9090 successful.
Listening on port 9090...
Connection accepted
Response sent to client
```

別タブで下記

```sh
lubuntu@lubuntu-virtualbox:~$ curl http://localhost:9090
Hello, World!
```

うまく送信できてるっぽい

3. データの送受信

```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>

#define PORT 9090
#define BUFFER_SIZE 1024

int main(){
    int server_fd, client_fd;
    struct sockaddr_in address;
    int addr_len = sizeof(address);
    char buffer[1024] = {0};

    // create socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if(server_fd == -1){
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    printf("Socket created successfully\n");

    // bind
    address.sin_family = AF_INET; // IPv4
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    if(bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0){
        perror("Bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Bind to port %d successful.\n", PORT);

    // listen, accept

    if(listen(server_fd, 3) < 0){
        perror("Listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Listening on port %d...\n", PORT);

    while((client_fd = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addr_len)) >= 0){
        printf("Connection accepted.\n");

        int bytes_read = read(client_fd, buffer, BUFFER_SIZE - 1);
        if(bytes_read > 0){
            buffer[bytes_read] = '\0';
            printf("Received: %s\n", buffer);

            size_t response_size = strlen(buffer) + 100;
            char *response = (char *)malloc(response_size);

            if (response == NULL) {
                perror("Memory allocation failed");
                close(client_fd);
                continue;
            }

            int written = snprintf(response, BUFFER_SIZE,
                     "HTTP/1.1 200 OK\r\n"
                     "Content-Type: text/plain\r\n"
                     "Content-Length: %ld\r\n"
                     "\r\n"
                     "You sent:\n%s",
                     strlen(buffer) + 9,
                     buffer);
            
            if(written < 0 || written >= BUFFER_SIZE){
                perror("Failed to write response");
                free(response);
                close(client_fd);
                continue;
            }

            send(client_fd, response, strlen(response), 0);
            printf("Response sent to cliend");
        }
        close(client_fd);
    }
    close(server_fd);

    return 0;
}
```

```sh
lubuntu@lubuntu-virtualbox:~/linux_tutorial/httpserver$ ./server 
Socket created successfully
Bind to port 9090 successful.
Listening on port 9090...
Connection accepted.
Received: POST / HTTP/1.1
Host: localhost:9090
User-Agent: curl/7.81.0
Accept: */*
Content-Length: 22
Content-Type: application/x-www-form-urlencoded

This is a test request
Response sent to cliendConnection accepted.
```

```sh
lubuntu@lubuntu-virtualbox:~$ curl -d "This is a test request" http://localhost:9090
You sent:
POST / HTTP/1.1
Host: localhost:9090
User-Agent: curl/7.81.0
Accept: */*
Content-Length: 22
Content-Type: application/x-www-form-urlencoded

This is a test request
```

よさそう

ひとまずこれくらい
