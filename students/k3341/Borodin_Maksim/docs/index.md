# Лабораторная работа №1: Работа с сокетами

**Студент:** Бородин Максим  
**Группа:** K3341  
**Дата:** 24 сентября 2025 г.

## Цель работы

Изучить основы работы с сокетами в Python, реализовать клиент-серверные приложения с использованием протоколов UDP и TCP, создать простой HTTP-сервер и многопользовательский чат.

## Задание 1: UDP-сокеты

### Условие
Реализовать клиентскую и серверную часть приложения. Клиент отправляет серверу сообщение «Hello, server», и оно должно отобразиться на стороне сервера. В ответ сервер отправляет клиенту сообщение «Hello, client», которое должно отобразиться у клиента.

**Требования:**
- Использовать библиотеку `socket`
- Реализовать с помощью протокола UDP

### Реализация

#### Серверная часть (server.py)
```python
import socket

HOST = 'localhost'
PORT = 8080

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as server_socket:
    server_socket.bind((HOST, PORT))
    print(f"UDP сервер запущен на {HOST}:{PORT}...")
    while True:
        data, addr = server_socket.recvfrom(1024)
        msg = data.decode('utf-8', errors='ignore')
        print(f"Получено от {addr}: {msg}")
        server_socket.sendto(b"Hello, client", addr)
        server_socket.close()
```

#### Клиентская часть (client.py)
```python
import socket

HOST = 'localhost'
PORT = 8080

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as client_socket:
    client_socket.settimeout(2.0)
    client_socket.sendto(b"Hello, server", (HOST, PORT))
    data, addr = client_socket.recvfrom(1024)
    print(f"Ответ от сервера: {data.decode('utf-8', errors='ignore')}")
```

### Результат работы
*[Вставить скриншот работы UDP клиента и сервера]*

---

## Задание 2: TCP-сокеты для математических операций

### Условие
Реализовать клиентскую и серверную часть приложения. Клиент запрашивает выполнение математической операции, параметры которой вводятся с клавиатуры. Сервер обрабатывает данные и возвращает результат клиенту.

**Варианты операций:**
1. Теорема Пифагора
2. Решение квадратного уравнения  
3. Поиск площади трапеции
4. Поиск площади параллелограмма

**Требования:**
- Использовать библиотеку `socket`
- Реализовать с помощью протокола TCP

### Реализация

#### Серверная часть (server.py)
```python
import socket
import math

HOST = 'localhost'
PORT = 8080

def parse_floats(parts, n):
    try:
        vals = [float(x) for x in parts[1:1+n]]
        if len(vals) != n:
            return None, "Ошибка: неправильное число аргументов."
        return vals, None
    except ValueError:
        return None, "Ошибка: параметры должны быть числами."

def handle_request(req: str) -> str:
    parts = req.strip().split()
    if not parts:
        return "Ошибка: пустой запрос."
    op = parts[0].upper()

    if op == 'PYTH':
        vals, err = parse_floats(parts, 2)
        if err: return err
        a, b = vals
        if a <= 0 or b <= 0:
            return "Ошибка: a и b должны быть > 0."
        c = math.hypot(a, b)
        return f"Гипотенуза: {c:.6g}"

    elif op == 'QUAD':
        vals, err = parse_floats(parts, 3)
        if err: return err
        a, b, c = vals
        if a == 0.0:
            if b == 0.0:
                return "Нет решений (a=0 и b=0)."
            x = -c / b
            return f"Линейное уравнение, x = {x:.6g}"
        D = b*b - 4*a*c
        if D > 0:
            sqrtD = math.sqrt(D)
            x1 = (-b + sqrtD) / (2*a)
            x2 = (-b - sqrtD) / (2*a)
            return f"Два корня: x1 = {x1:.6g}, x2 = {x2:.6g}"
        elif D == 0:
            x = -b / (2*a)
            return f"Один корень: x = {x:.6g}"
        else:
            return "Нет вещественных корней."

    elif op == 'TRAP':
        vals, err = parse_floats(parts, 3)
        if err: return err
        a, b, h = vals
        if a <= 0 or b <= 0 or h <= 0:
            return "Ошибка: a, b, h должны быть > 0."
        area = (a + b) / 2 * h
        return f"Площадь трапеции: {area:.6g}"

    elif op == 'PARA':
        vals, err = parse_floats(parts, 2)
        if err: return err
        a, h = vals
        if a <= 0 or h <= 0:
            return "Ошибка: a и h должны быть > 0."
        area = a * h
        return f"Площадь параллелограмма: {area:.6g}"

    else:
        return "Ошибка: неизвестная операция."

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind((HOST, PORT))
server_socket.listen(5)
print(f"TCP сервер запущен на {HOST}:{PORT}...")

while True:
    conn, addr = server_socket.accept()
    print("Клиент подключился:", addr)
    data = conn.recv(1024)
    if not data:
        conn.close()
        continue
    req = data.decode('utf-8', errors='ignore')
    print("Запрос:", req.strip())
    resp = handle_request(req)
    conn.sendall((resp + "\n").encode('utf-8'))
    conn.close()
```

#### Клиентская часть (client.py)
```python
import socket
import sys

HOST = 'localhost'
PORT = 8080

def require_pos(x, name):
    if x <= 0:
        print(f"Ошибка: {name} должно быть > 0.")
        sys.exit(1)

# Определение варианта по номеру в журнале
try:
    n = int(input("Введите ваш номер в журнале (целое > 0): ").strip())
    if n <= 0:
        print("Ошибка: номер должен быть > 0.")
        sys.exit(1)
except ValueError:
    print("Ошибка: номер должен быть целым числом.")
    sys.exit(1)

variant = ((n - 1) % 4) + 1
print(f"Ваш вариант: {variant}")

# Ввод параметров в зависимости от варианта
try:
    if variant == 1:
        # Теорема Пифагора
        a = float(input("a (катет 1): "))
        b = float(input("b (катет 2): "))
        require_pos(a, "a")
        require_pos(b, "b")
        msg = f"PYTH {a} {b}\n"

    elif variant == 2:
        # Квадратное уравнение
        a = float(input("a: "))
        b = float(input("b: "))
        c = float(input("c: "))
        msg = f"QUAD {a} {b} {c}\n"

    elif variant == 3:
        # Площадь трапеции
        a = float(input("a (основание 1): "))
        b = float(input("b (основание 2): "))
        h = float(input("h (высота): "))
        require_pos(a, "a")
        require_pos(b, "b")
        require_pos(h, "h")
        msg = f"TRAP {a} {b} {h}\n"

    else:  # variant == 4
        # Площадь параллелограмма
        a = float(input("a (сторона/основание): "))
        h = float(input("h (высота): "))
        require_pos(a, "a")
        require_pos(h, "h")
        msg = f"PARA {a} {h}\n"

except ValueError:
    print("Ошибка: параметры должны быть числами.")
    sys.exit(1)

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))
sock.sendall(msg.encode('utf-8'))

data = sock.recv(1024)
print("Ответ сервера:", data.decode('utf-8', errors='ignore').strip())

sock.close()
```

### Результат работы
*[Вставить скриншот работы TCP клиента и сервера с примерами вычислений]*

---

## Задание 3: HTTP-сервер

### Условие
Реализовать серверную часть приложения. Клиент подключается к серверу, и в ответ получает HTTP-сообщение, содержащее HTML-страницу, которая сервер подгружает из файла `index.html`.

**Требования:**
- Использовать библиотеку `socket`

### Реализация

#### Серверная часть (server.py)
```python
import socket
from pathlib import Path

HOST = "0.0.0.0"
PORT = 8080
FILE = Path("index.html")

def build_response(body: bytes, status: str = "200 OK", content_type="text/html; charset=utf-8") -> bytes:
    headers = [
        f"HTTP/1.1 {status}",
        f"Content-Type: {content_type}",
        f"Content-Length: {len(body)}",
        "Connection: close",
    ]
    head = ("\r\n".join(headers) + "\r\n\r\n").encode("utf-8")
    return head + body

def main():
    srv = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    srv.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    srv.bind((HOST, PORT))
    srv.listen(5)
    print(f"HTTP сервер запущен на http://localhost:{PORT}")

    try:
        while True:
            conn, addr = srv.accept()
            try:
                _ = conn.recv(1024)

                if FILE.exists():
                    body = FILE.read_bytes()
                    resp = build_response(body, "200 OK")
                else:
                    body = b"404 Not Found: index.html not found"
                    resp = build_response(body, "404 Not Found", content_type="text/plain; charset=utf-8")

                conn.sendall(resp)
            finally:
                conn.close()
    except KeyboardInterrupt:
        print("\nОстановка сервера...")
    finally:
        srv.close()

if __name__ == "__main__":
    main()
```

#### HTML-страница (Index.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My html page</title>
</head>
<body>
    <p>
        Today is a beautiful day. We go swimming and fishing.
    </p>
    
    <p>
         Hello there. How are you?
    </p>
</body>
</html>
```

### Результат работы
*[Вставить скриншот HTTP-сервера в браузере]*

---

## Задание 4: Многопользовательский чат

### Условие
Реализовать многопользовательский чат с использованием библиотеки `threading`.

**Требования:**
- Использовать библиотеку `socket`
- Использовать библиотеку `threading`
- Протокол TCP

### Реализация

#### Серверная часть (chat_server.py)
```python
import socket
import threading

HOST = "0.0.0.0"
PORT = 9090
BACKLOG = 20
ENC = "utf-8"
MAX_LINE = 800

clients = {}           # socket -> name
clients_lock = threading.Lock()

def broadcast(message: str, exclude_sock=None):
    """Отправить сообщение всем клиентам, кроме exclude_sock (если задан)."""
    data = message.encode(ENC, errors="ignore")
    dead = []
    with clients_lock:
        for sock in list(clients.keys()):
            if sock is exclude_sock:
                continue
            try:
                sock.sendall(data)
            except OSError:
                dead.append(sock)
        # подчистим умершие соединения
        for sock in dead:
            try:
                name = clients.pop(sock)
            except KeyError:
                name = None
            try:
                sock.close()
            except OSError:
                pass
            if name:
                # уведомим остальных, что клиент отвалился
                msg = f"* {name} left the chat\n"
                for s in list(clients.keys()):
                    try:
                        s.sendall(msg.encode(ENC))
                    except OSError:
                        pass

def safe_line(s: str) -> str:
    """Обрезаем переносы строк и длину, чтобы не ломать терминалы/формат."""
    s = s.replace("\r", "").replace("\n", "")
    if len(s) > MAX_LINE:
        s = s[:MAX_LINE]
    return s

def handle_client(conn: socket.socket, addr):
    conn.settimeout(300)
    buf = ""
    name = None
    try:
        # читаем первую строку как имя
        while "\n" not in buf:
            chunk = conn.recv(1024)
            if not chunk:
                raise ConnectionError("client disconnected before sending name")
            buf += chunk.decode(ENC, errors="ignore")
        name, buf = buf.split("\n", 1)
        name = safe_line(name).strip() or f"user@{addr[0]}:{addr[1]}"

        # регистрируем клиента
        with clients_lock:
            clients[conn] = name
        conn.sendall(f"* Welcome, {name}! Type /quit to exit\n".encode(ENC))
        broadcast(f"* {name} joined the chat\n", exclude_sock=conn)

        # основной цикл чтения строк
        while True:
            if "\n" in buf:
                line, buf = buf.split("\n", 1)
            else:
                data = conn.recv(1024)
                if not data:
                    break
                buf += data.decode(ENC, errors="ignore")
                if "\n" not in buf:
                    continue
                line, buf = buf.split("\n", 1)

            text = safe_line(line)
            if not text:
                continue
            if text == "/quit":
                break

            broadcast(f"[{name}] {text}\n", exclude_sock=conn)

    except (ConnectionError, OSError, socket.timeout):
        pass
    finally:
        # снятие регистрации, закрытие и уведомление остальных
        with clients_lock:
            user = clients.pop(conn, None)
        try:
            conn.close()
        except OSError:
            pass
        if user:
            broadcast(f"* {user} left the chat\n")

def main():
    srv = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    srv.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    srv.bind((HOST, PORT))
    srv.listen(BACKLOG)
    print(f"Chat server listening on {HOST}:{PORT} ...")

    try:
        while True:
            conn, addr = srv.accept()
            t = threading.Thread(target=handle_client, args=(conn, addr), daemon=True)
            t.start()
    except KeyboardInterrupt:
        print("\nStopping server...")
    finally:
        with clients_lock:
            for s in list(clients.keys()):
                try:
                    s.close()
                except OSError:
                    pass
            clients.clear()
        try:
            srv.close()
        except OSError:
            pass

if __name__ == "__main__":
    main()
```

#### Клиентская часть (chat_client.py)
```python
import socket
import threading
import sys

HOST = "127.0.0.1"
PORT = 9090
ENC = "utf-8"
MAX_LINE = 800

def recv_loop(sock: socket.socket):
    try:
        while True:
            data = sock.recv(1024)
            if not data:
                print("\n[disconnected]")
                break
            try:
                print(data.decode(ENC), end="")
            except UnicodeDecodeError:
                pass
    except OSError:
        pass
    finally:
        try:
            sock.close()
        except OSError:
            pass

def safe_line(s: str) -> str:
    s = s.replace("\r", " ").replace("\n", " ")
    return s[:MAX_LINE]

def main():
    name = input("Введите ваш ник: ").strip()
    if not name:
        print("Ник не может быть пустым.")
        sys.exit(1)

    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        sock.connect((HOST, PORT))
    except OSError as e:
        print(f"Не удалось подключиться: {e}")
        sys.exit(1)

    # запускаем поток-приёмник
    t = threading.Thread(target=recv_loop, args=(sock,), daemon=True)
    t.start()

    try:
        # первая строка — ник
        sock.sendall((safe_line(name) + "\n").encode(ENC))
        # далее — сообщения пользователя
        while True:
            try:
                line = input()
            except (EOFError, KeyboardInterrupt):
                line = "/quit"
            line = line.strip()
            if not line:
                continue
            line = safe_line(line)
            sock.sendall((line + "\n").encode(ENC))
            if line == "/quit":
                break
    except OSError:
        pass
    finally:
        try:
            sock.close()
        except OSError:
            pass

if __name__ == "__main__":
    main()
```

### Результат работы
*[Вставить скриншот работы многопользовательского чата с несколькими клиентами]*

---

## Задание 5: Веб-сервер для обработки GET и POST запросов

### Условие
Написать простой веб-сервер для обработки GET и POST HTTP-запросов с помощью библиотеки `socket` в Python.

**Функциональность:**
1. Принять и записать информацию о дисциплине и оценке по дисциплине
2. Отдать информацию обо всех оценках по дисциплинам в виде HTML-страницы

### Реализация

#### Серверная часть (server.py)
```python
import socket
import sys
import signal
import re
from urllib.parse import parse_qs, urlparse

class SimpleHTTPServer:
    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.grades = {}  # словарь для хранения оценок {дисциплина: оценка}
        self.running = True
        
    def start_server(self):
        server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        server_socket.bind((self.host, self.port))
        server_socket.listen(5)
        
        print(f"Сервер запущен на {self.host}:{self.port}")
        print("Нажмите Ctrl+C для остановки сервера")
        
        while self.running:
            try:
                server_socket.settimeout(1.0)
                client_socket, address = server_socket.accept()
                print(f"Подключение от {address}")
                
                try:
                    self.handle_request(client_socket)
                except Exception as e:
                    print(f"Ошибка при обработке запроса: {e}")
                finally:
                    client_socket.close()
            except socket.timeout:
                continue
            except Exception as e:
                if self.running:
                    print(f"Ошибка сервера: {e}")
        
        server_socket.close()
        print("Сервер остановлен")
    
    def handle_request(self, client_socket):
        request_data = client_socket.recv(1024).decode('utf-8')
        if not request_data:
            return
            
        print("Получен запрос:")
        print(request_data)
        
        lines = request_data.split('\n')
        request_line = lines[0]
        method, path, version = request_line.split()
        
        if method == 'GET':
            self.handle_get(client_socket, path)
        elif method == 'POST':
            body_start = request_data.find('\r\n\r\n')
            if body_start != -1:
                body = request_data[body_start + 4:]
                self.handle_post(client_socket, body)
        else:
            self.send_error_response(client_socket, 405, "Method Not Allowed")
    
    def handle_get(self, client_socket, path):
        if path == '/':
            self.send_main_page(client_socket)
        else:
            self.send_error_response(client_socket, 404, "Not Found")
    
    def handle_post(self, client_socket, body):
        try:
            form_data = parse_qs(body)
            subject = form_data.get('subject', [''])[0].strip()
            grade = form_data.get('grade', [''])[0].strip()
            
            validation_error = self.validate_input(subject, grade)
            if validation_error:
                self.send_error_response(client_socket, 400, validation_error)
                return
            
            self.grades[subject] = grade
            print(f"Добавлена оценка: {subject} - {grade}")
            
            self.send_redirect_response(client_socket, '/')
        except Exception as e:
            print(f"Ошибка при обработке POST: {e}")
            self.send_error_response(client_socket, 500, "Internal Server Error")
    
    def validate_input(self, subject, grade):
        if not subject:
            return "Дисциплина не может быть пустой"
        
        if len(subject) > 100:
            return "Название дисциплины слишком длинное (максимум 100 символов)"
        
        if not re.match(r'^[а-яА-Яa-zA-Z0-9\s\-\.]+$', subject):
            return "Название дисциплины содержит недопустимые символы"
        
        if not grade:
            return "Оценка не может быть пустой"
        
        try:
            grade_num = int(grade)
            if grade_num < 1 or grade_num > 5:
                return "Оценка должна быть от 1 до 5"
        except ValueError:
            return "Оценка должна быть числом"
        
        return None
    
    def send_main_page(self, client_socket):
        html_content = self.render_template()
        
        response = f"""HTTP/1.1 200 OK\r
Content-Type: text/html; charset=utf-8\r
Content-Length: {len(html_content.encode('utf-8'))}\r
\r
{html_content}"""
        
        client_socket.send(response.encode('utf-8'))
    
    def render_template(self):
        try:
            with open('template.html', 'r', encoding='utf-8') as file:
                template = file.read()
        except FileNotFoundError:
            template = '<html><body><h1>шаблон не найден</h1>{{grades}}</body></html>'

        grades_html = ''
        if self.grades:
            for subject, grade in self.grades.items():
                grades_html += f'<div class="grade-item"><strong>{subject}</strong>: {grade}</div>'
        else:
            grades_html = '<p>Оценки пока не добавлены</p>'

        return template.replace('{{grades}}', grades_html)
    
    def send_redirect_response(self, client_socket, location):
        response = f"""HTTP/1.1 302 Found\r
Location: {location}\r
Content-Length: 0\r
\r
"""
        client_socket.send(response.encode('utf-8'))
    
    def send_error_response(self, client_socket, status_code, message):
        html_content = f"""
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Ошибка {status_code}</title>
</head>
<body>
    <h1>Ошибка {status_code}</h1>
    <p>{message}</p>
    <a href="/">Вернуться на главную</a>
</body>
</html>"""
        
        response = f"""HTTP/1.1 {status_code} {message}\r
Content-Type: text/html; charset=utf-8\r
Content-Length: {len(html_content.encode('utf-8'))}\r
\r
{html_content}"""
        
        client_socket.send(response.encode('utf-8'))

def signal_handler(signum, frame):
    print("\nПолучен сигнал остановки...")
    global server
    if 'server' in globals():
        server.running = False

def main():
    if len(sys.argv) != 3:
        print("Использование: python server.py <host> <port>")
        print("Пример: python server.py localhost 8080")
        sys.exit(1)
    
    host = sys.argv[1]
    port = int(sys.argv[2])
    
    global server
    server = SimpleHTTPServer(host, port)
    
    signal.signal(signal.SIGINT, signal_handler)
    signal.signal(signal.SIGTERM, signal_handler)
    
    try:
        server.start_server()
    except KeyboardInterrupt:
        print("\nСервер остановлен")

if __name__ == "__main__":
    main()
```

#### HTML-шаблон (template.html)
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>система оценок</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        .form-container { background: #f5f5f5; padding: 20px; border-radius: 5px; margin-bottom: 30px; }
        .grades-container { background: #e8f4f8; padding: 20px; border-radius: 5px; }
        input, button { padding: 8px; margin: 5px; }
        button { background: #007bff; color: white; border: none; border-radius: 3px; cursor: pointer; }
        button:hover { background: #0056b3; }
        .grade-item { background: white; padding: 10px; margin: 5px 0; border-radius: 3px; }
    </style>
</head>
<body>
    <h1>система учета оценок</h1>
    
    <div class="form-container">
        <h2>добавить новую оценку</h2>
        <form method="POST" action="/">
            <div>
                <label>дисциплина:</label><br>
                <input type="text" name="subject" required placeholder="введите название дисциплины">
            </div>
            <div>
                <label>оценка:</label><br>
                <input type="text" name="grade" required placeholder="введите оценку">
            </div>
            <button type="submit">добавить оценку</button>
        </form>
    </div>
    
    <div class="grades-container">
        <h2>список оценок</h2>
        {{grades}}
    </div>
</body>
</html>
```

### Результат работы
*[Вставить скриншот веб-интерфейса для добавления и просмотра оценок]*

---

## Выводы

В ходе выполнения лабораторной работы были изучены основы работы с сокетами в Python:

1. **UDP-сокеты** - реализован простой обмен сообщениями между клиентом и сервером
2. **TCP-сокеты** - создан калькулятор для математических операций с обработкой различных вариантов заданий
3. **HTTP-сервер** - разработан базовый веб-сервер для отдачи статических HTML-страниц
4. **Многопользовательский чат** - реализован с использованием потоков для одновременной обработки множества клиентов
5. **Веб-приложение** - создан полноценный веб-сервер с обработкой GET и POST запросов, формами и валидацией данных

Все задания успешно выполнены с использованием библиотеки `socket` и соблюдением требований по протоколам. Многопользовательский чат и веб-сервер демонстрируют использование потоков (`threading`) для обработки множественных подключений.

Полученные навыки работы с сокетами являются фундаментальными для понимания сетевого программирования и разработки распределенных приложений.
