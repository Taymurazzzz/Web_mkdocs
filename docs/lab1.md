# Лабораторная работа 1
В этой работе предстоит научиться работь с веб-сокетами
## Задание 1
Реализовать клиентскую и серверную часть приложения. Клиент отправляет серверу сообщение «Hello, server», и оно должно отобразиться на стороне сервера. В ответ сервер отправляет клиенту сообщение «Hello, client», которое должно отобразиться у клиента.
### Код сервера
```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(('localhost', 8081))

while True:

    data, client_address = server_socket.recvfrom(1024)

    if data.decode() == "Hello, server":
        response = 'Hello, client'
        server_socket.sendto(response.encode(), client_address)

    else:
        response = "Bye"
        server_socket.sendto(response.encode(), client_address)
```
### Код клиента
```python
import socket

client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

message = 'Hello, server'

client_socket.sendto(message.encode(), ('localhost', 8081))

response, _ = client_socket.recvfrom(1024)
print(f'Ответ от сервера: {response.decode()}')

client_socket.close()
```
## Задание 2
Реализовать клиентскую и серверную часть приложения. Клиент запрашивает выполнение математической операции, параметры которой вводятся с клавиатуры. Сервер обрабатывает данные и возвращает результат клиенту.
Для моего варианта сервер должен решить квадратное уравнение.
### Код сервера
```python
import socket
import math

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(('localhost', 8081))
server_socket.listen(1)

while True:
    client_connection, client_address = server_socket.accept()

    request = client_connection.recv(1024).decode()
    eq_params = request.split(" ")
    a = int(eq_params[0])
    b = int(eq_params[1])
    c = int(eq_params[2])
    discr = b ** 2 - 4 * a * c
    if discr > 0:
        x1 = (-b + math.sqrt(discr)) / (2 * a)
        x2 = (-b - math.sqrt(discr)) / (2 * a)
        response = "x1 = %.2f \nx2 = %.2f" % (x1, x2)
        client_connection.sendall(response.encode())

    elif discr == 0:
        x = -b / (2 * a)
        response = "x = %.2f" % x
        client_connection.sendall(response.encode())

    else:
        response = "Корней нет"
        client_connection.sendall(response.encode())

    client_connection.close()
```
### Код клиента
```python
import socket

client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

client_socket.connect(('localhost', 8081))

params = input()

client_socket.sendall(bytes(params, "utf-8"))

response = client_socket.recv(1024)
print(f'Ответ от сервера: {response.decode()}')

client_socket.close()
```
## Задание 3
Реализовать серверную часть приложения. Клиент подключается к серверу, и в ответ получает HTTP-сообщение, содержащее HTML-страницу, которая сервер подгружает из файла index.html.
### Код сервера
```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(('localhost', 8082))
server_socket.listen(1)

while True:
    client_socket, client_address = server_socket.accept()
    print(f"Connection from {client_address}")

    request = client_socket.recv(1024).decode('utf-8')
    print(f"Request:\n{request}")

    with open("index.html", "r") as file:
        html_content = file.read()

    response = (
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html; charset=utf-8\r\n"
        f"Content-Length: {len(html_content)}\r\n"
        "Connection: close\r\n"
        "\r\n"
        f"{html_content}"
    )

    client_socket.sendall(response.encode('utf-8'))
    client_socket.close()
```
### Код странички
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f4f4f9;
            color: #333;
            margin: 0;
            padding: 0;
        }
        header {
            background-color: #007BFF;
            color: white;
            padding: 20px;
        }
        h1 {
            margin: 0;
        }
        main {
            padding: 20px;
        }
        footer {
            background-color: #333;
            color: white;
            padding: 10px;
            position: fixed;
            bottom: 0;
            width: 100%;
        }
        a {
            color: #007BFF;
            text-decoration: none;
        }
    </style>
</head>
<body>
    <header>
        <h1>Welcome to My Page!</h1>
    </header>
    <main>
        <p>Hello, world! This is a simple HTML welcome page.</p>
        <p>Feel free to explore and <a href="#">learn more about HTML</a>.</p>
    </main>
    <footer>
        <p>© 2025 Your Name. All rights reserved.</p>
    </footer>
</body>
</html>
```
## Задание 4
Реализовать двухпользовательский или многопользовательский чат. Для максимального количества баллов реализуйте многопользовательский чат.
### Код сервера
```python
import socket
import threading

clients = []


def handle_client(client_socket):
    clients.append(client_socket)
    while True:
        message = client_socket.recv(1024)
        broadcast(message, client_socket)


def broadcast(message, sender_socket):
    for client in clients:
        if client != sender_socket:
            client.send(message)


def start_server(host='localhost', port=8083):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(5)
    print(f"Server is running on {host}:{port}")

    while True:
        client_socket, client_address = server_socket.accept()
        thread = threading.Thread(target=handle_client, args=(client_socket,))
        thread.start()


if __name__ == "__main__":
    start_server()
```
### Код клиента
```python
import socket
import threading


def receive_messages(client_socket):
    while True:
        message = client_socket.recv(1024).decode('utf-8')
        print(message)


def start_client(host='localhost', port=8083):
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect((host, port))

    receive_thread = threading.Thread(target=receive_messages, args=(client_socket,))
    receive_thread.start()

    while True:
        message = input()
        if message.lower() == 'exit':
            client_socket.close()
            break
        client_socket.send(message.encode('utf-8'))


if __name__ == "__main__":
    start_client()
```
## Задание 5
Написать простой веб-сервер для обработки GET и POST HTTP-запросов с помощью библиотеки socket в Python.
Сервер должен:
* Принять и записать информацию о дисциплине и оценке по дисциплине.
* Отдать информацию обо всех оценках по дисциплинам в виде HTML-страницы.
## Код сервера
```python
import socket
import threading
import urllib.parse

grades = {}


def handle_client(client_socket):
    request = client_socket.recv(1024).decode('utf-8')

    headers = request.split('\r\n')
    first_line = headers[0]
    method, path, _ = first_line.split()

    if method == "POST" and path == "/grades":
        content = headers[-1]
        content = content.split('&')
        discipline = urllib.parse.unquote(content[0][11:])
        grade = urllib.parse.unquote(content[1][6:])
        grades[discipline] = grades.get(discipline, []) + [grade]

        print(grades)
        response = "HTTP/1.1 303 See Other\r\nLocation: /grades\r\n\r\n"
        client_socket.sendall(response.encode())

    elif method == "GET" and path == "/grades":
        response_body = """
        <html>
        <head><title>Оценки</title></head>
        <body>
        <h1>Оценки по дисциплинам</h1>
        <ul>
        """
        for discipline in grades:
            response_body += f"<li>{discipline}: {', '.join(map(str, grades.get(discipline, [])))}</li>"
        response_body += """
        </ul>
        <h2>Добавить оценку</h2>
        <form method="post" action="/grades">
            Дисциплина: <input type="text" name="discipline"><br>
            Оценка: <input type="text" name="grade"><br>
            <input type="submit" value="Добавить">
        </form>
        </body>
        </html>
        """

        response = (
            "HTTP/1.1 200 OK\r\n"
            "Content-Type: text/html; charset=utf-8\r\n"
            f"Content-Length: {len(response_body.encode())}\r\n"
            "Connection: close\r\n"
            "\r\n"
            f"{response_body}"
        )

        client_socket.sendall(response.encode('utf-8'))

    client_socket.close()


def start_server(host="localhost", port=8085):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(5)
    print(f"Сервер запущен на {host}:{port}")

    while True:
        client_sock, _ = server_socket.accept()
        handle_client(client_sock)



if __name__ == "__main__":
    start_server()
```
## Вывод
В этой лабораторной работе я научился пользоваться библиотекой `socket`, а также базово изучил как работают протоколы TCP, UDP и HTTP