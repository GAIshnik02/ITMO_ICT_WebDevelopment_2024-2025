# Лабораторная работа №1
<b>Выполнил:</b> Захматов Юрий Дмитриевич

<b>Группа:</b> K3341 

---

## Задание 1:

---
<b>Содержание:</b> Реализовать клиентскую и серверную часть приложения. Клиент отправляет серверу сообщение «Hello, server», и оно должно отобразиться на стороне сервера. В ответ сервер отправляет клиенту сообщение «Hello, client», которое должно отобразиться у клиента.

<b>Требования:</b> Обязательно использовать библиотеку socket.
Реализовать с помощью протокола UDP.

---
<b>Выполнение: </b>

1. Создаем скрипт для сервера: 
~~~python
import socket


def run():
    # Создание UDP сокета
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # Задаем параметры
    server_address = ("localhost", 8080)
    server_socket.bind(server_address)

    print(f"Server started on {server_address[0]}:{server_address[1]}")
    print("Awaiting connection...")

    try:
        while True:
            # Получаем запрос клиента

            data, client_address = server_socket.recvfrom(1024)
            message = data.decode("utf-8")

            print(f"Received message from {client_address} : {message}")

            # отправляем ответ клиенту
            response = "Hello, client"
            server_socket.sendto(response.encode("utf-8"), client_address)
            print(f"Sent response to {client_address}")
    except Exception:
        print("\nServer shutting down...")
    finally:
        server_socket.close()


if __name__ == "__main__":
    run()
~~~
---
2. Создаем скрипт для клиента
~~~python
import socket

def run():
    #Создаем сокет для клиента UDP
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    try:
        #Подключаемся к сокету
        client_socket.connect(('localhost', 8080))

        #Отправляем сообщение
        client_socket.sendall(b'Hello, server')

        #Получаем ответ
        response = client_socket.recv(1024)
        print(f"Response from server: {response.decode('utf-8')}")
    except Exception as e:
        print(e)
    finally:
        client_socket.close()

if __name__ == '__main__':
    run()
~~~
---
3. Запускаем сервер
<img src="screenshots\Screenshot_1.jpeg.jpg">
---
4. Запускаем скрипт
![img.png](screenshots/img.png)
---
5. Скрин с сервера
![img.png](screenshots/img1.png)

## Задание 2:

---
<b>Содержание:</b> Реализовать клиентскую и серверную часть приложения. Клиент запрашивает выполнение математической операции, параметры которой вводятся с клавиатуры. Сервер обрабатывает данные и возвращает результат клиенту.

<b>Требования:</b> Обязательно использовать библиотеку socket.
Реализовать с помощью протокола TCP.

<b>Вариант: </b> 8 (Поиск площади параллелограмма)

---
<b>Выполнение: </b>

1. Создаем скрипт для сервера
~~~python
import socket

def calculate(base, height):
    return base * height

def run():
    # Создание TCP сокета
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Задаем параметры
    server_address = ("localhost", 8080)
    server_socket.bind(server_address)
    server_socket.listen(1)

    print(f"Server started on {server_address[0]}:{server_address[1]}")
    print("Awaiting connection...")

    while True:
        # Принимаем подключение клиента
        client_socket, addr = server_socket.accept()

        print(f"Connected by: {addr}")

        try:
            # Получаем данные от клиента
            data = client_socket.recv(1024).decode("utf-8")
            print(f"Received data: {data}")

            # Преобразуем данные
            base, height = map(float, data.split(","))

            # Считаем
            area = calculate(base,height)

            # Отправляем данные
            client_socket.send(str(area).encode("utf-8"))

            print(f"Sent result to client: {area}")

        except Exception as e:
            error_msg = f"Error: {e}"
            client_socket.send(error_msg.encode("utf-8"))
            print(error_msg)

        finally:
            client_socket.close()

if __name__ == "__main__":
    run()
~~~
---
2. Создаем скрипт для клиента
~~~python
import socket


def run():
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect(('localhost', 8080))

    try:
        base = float(input("Введите основание параллелограмма: "))
        height = float(input("Введите высоту параллелограмма: "))

        data = f"{base},{height}"
        client_socket.send(data.encode('utf-8'))

        result = client_socket.recv(1024).decode('utf-8')
        print(f"Площадь параллелограмма: {result}")

    except Exception as e:
        print(f"Ошибка: {e}")

    finally:
        client_socket.close()


if __name__ == "__main__":
    run()
~~~
---
3. Лог сервера (клиент запускался 2 раза)![img_1.png](screenshots%2Fimg_1.png)
---
4. Лог клиента![img_2.png](screenshots%2Fimg_2.png)


## Задание 3:

---
<b>Содержание:</b> Реализовать серверную часть приложения. Клиент подключается к серверу, и в ответ получает HTTP-сообщение, содержащее HTML-страницу, которая сервер подгружает из файла index.html

<b>Требования:</b> Обязательно использовать библиотеку socket.

---
<b>Выполнение: </b>

1. Создаем скрипт для сервера
~~~python
import os
import socket

HOST = "localhost"
PORT = 8080
MAX_CONNECTIONS = 3
BUFFER_SIZE = 1024


def load_html_file(filename):
    if os.path.exists(filename):
        with open(filename, 'r', encoding="utf-8") as file:
            return file.read()
    else:
        return None



def run():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind((HOST, PORT))
    server.listen(MAX_CONNECTIONS)
    print("Server is running on http://localhost:8080")

    while True:
        client, _ = server.accept()
        client.recv(BUFFER_SIZE)

        html = load_html_file("index.html")
        if html is not None:
            html_bytes = html.encode("utf-8")
            response = (
                        "HTTP/1.1 200 OK\r\n"
                        "Content-Type: text/html; charset=utf-8\r\n"
                        f"Content-Length: {len(html_bytes)}\r\n"
                        "\r\n"

            ).encode('utf-8') + html_bytes
        else:
            response = (
                "HTTP/1.1 404 Not Found\r\n"
                "Content-Type: text/html; charset=utf-8\r\n"
                "\r\n"
                "<h1>404 Not Found</h1>"
            ).encode('utf-8')
        client.sendall(response)
        client.close()

if __name__ == "__main__":
    run()
~~~
---
2. Проверяем что все работает
![img_3.png](screenshots%2Fimg_3.png)


## Задание 4

---
<b>Содержание:</b> Реализовать многопользовательский чат. Для максимального количества баллов реализуйте многопользовательский чат.

<b>Требования:</b> Обязательно использовать библиотеку socket.
Для многопользовательского чата необходимо использовать библиотеку threading.

---
<b>Выполнение: </b>

1. Скрипт сервера
~~~python
import socket
import threading

HOST = 'localhost'
PORT = 8080


class ChatServer:
    def __init__(self, host=HOST, port=PORT):
        self.host = host
        self.port = port
        self.clients = [] # Кто подключен
        self.nicknames = [] # Никнеймы
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    def broadcast(self, message, sender_client=None):
        """Отправка сообщения всем клиентам кроме отправителя"""
        for client in self.clients:
            if client != sender_client:
                try:
                    client.send(message)
                except:
                    # Если отправка не удалась, удаляем клиента
                    self.remove_client(client)

    def remove_client(self, client):
        """Удаление клиента из чата"""
        if client in self.clients:
            index = self.clients.index(client)
            self.clients.remove(client)
            nickname = self.nicknames[index]
            self.nicknames.remove(nickname)

            broadcast_message = f"{nickname} покинул чат!".encode('utf-8')
            self.broadcast(broadcast_message)
            print(f"{nickname} отключился")

    def handle_client(self, client):
        """Обработка сообщений от клиента"""
        while True:
            try:
                message = client.recv(1024)
                if message:
                    self.broadcast(message, client)
                else:
                    self.remove_client(client)
                    break
            except:
                self.remove_client(client)
                break

    def start_server(self):
        """Запуск сервера"""
        self.server_socket.bind((self.host, self.port))
        self.server_socket.listen()
        print(f"Сервер чата запущен на {self.host}:{self.port}")

        while True:
            client, address = self.server_socket.accept()
            print(f"Новое подключение от {str(address)}")

            # Запрос ника у клиента
            client.send("NICK".encode('utf-8'))
            nickname = client.recv(1024).decode('utf-8')

            self.nicknames.append(nickname)
            self.clients.append(client)

            print(f"Никнейм клиента: {nickname}")
            broadcast_message = f"{nickname} присоединился к чату!".encode('utf-8')
            self.broadcast(broadcast_message)
            client.send("Подключение к серверу успешно!".encode('utf-8'))

            # Запуск потока для обработки клиента
            thread = threading.Thread(target=self.handle_client, args=(client,), daemon=True)
            thread.start()


if __name__ == "__main__":
    server = ChatServer()
    server.start_server()
~~~
---
2. Скрипт для клиента:
~~~python
import socket
import threading


HOST = 'localhost'
PORT = 8080

class ChatClient:
    def __init__(self, host=HOST, port=PORT):
        self.host = host
        self.port = port
        self.client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.nickname = None

    def receive_messages(self):
        """Получение сообщений от сервера"""
        while True:
            try:
                message = self.client_socket.recv(1024).decode('utf-8')
                if message == "NICK":
                    self.client_socket.send(self.nickname.encode('utf-8'))
                else:
                    print(message)
            except:
                print("Произошла ошибка!")
                self.client_socket.close()
                break

    def send_messages(self):
        """Отправка сообщений на сервер"""
        while True:
            try:
                message = input()
                if message.lower() == 'exit':
                    break
                formatted_message = f"{self.nickname}: {message}"
                self.client_socket.send(formatted_message.encode('utf-8'))
            except:
                print("Ошибка отправки сообщения!")
                break

    def start_client(self):
        """Запуск клиента"""
        try:
            self.client_socket.connect((self.host, self.port))

            # Получение ника
            self.nickname = input("Введите ваш никнейм: ")

            # Запуск потока для получения сообщений
            receive_thread = threading.Thread(target=self.receive_messages, daemon=True)
            receive_thread.start()

            print("Подключение успешно! Начинайте общение (для выхода введите 'exit')")
            print("-" * 50)

            # Основной поток для отправки сообщений
            self.send_messages()

        except Exception as e:
            print(f"Ошибка подключения: {e}")
        finally:
            self.client_socket.close()


if __name__ == "__main__":
    client = ChatClient()
    client.start_client()
~~~
---
3. Пример работы (слева клиент 1, справа клиент 2, снизу сервер)

![img_4.png](screenshots%2Fimg_4.png)


## Задание 5

---
<b>Содержание:</b> Написать простой веб-сервер для обработки GET и POST HTTP-запросов с помощью библиотеки socket в Python.

<b>Требования:</b> Сервер должен:
1. Принять и записать информацию о дисциплине и оценке по дисциплине.
2. Отдать информацию обо всех оценках по дисциплинам в виде HTML-страницы.
---
<b>Выполнение: </b>


