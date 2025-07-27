---
title: "Xây dựng API đơn giản với Flask, demo với Ngrok"
description: "Tuần vừa rồi anh teamlead seminar về Flask. Oh, sao mình thấy thằng này build API nhanh gọn, đơn giản thế, đúng tinh thần của Python."
date: 2021-01-14T01:03:00+07:00
slug: flask
image: flask.webp
categories: [Technical]
tags: [Python, Flask, Ngrok]
---

# Lời mở đầu

Team mình đợt này đang triển khai chương trình mỗi tuần một bài học, nghĩa là mỗi tuần một người trong team sẽ lên seminar cho mọi người về 1 kỹ thuật hoặc công nghệ gì đấy. Anh em vừa trải qua mấy tuần mới đi hết Kotlin. Tuần vừa rồi anh teamlead seminar về Flask. Oh, sao mình thấy thằng này build API nhanh gọn, đơn giản thế, đúng tinh thần của Python. Lâu nay mình chỉ code Front-end với Mobile, chưa đụng Back-end bao giờ (Thực ra là hồi Sinh viên làm PHP nhưng quên hết rồi :joy::joy:). Thế là mình cũng về thử bắt tay làm ngay một chú xem thế nào.

# Setup

## Basic

Về IDE, tối ưu nhất có lẽ là PyCharm. Nhưng cài nó cũng khá nặng nên mình tận dụng máy đang có sẵn Visual Studio Code, chỉ cần cài thêm extension Python là đủ để chiến.

Trước tiên là cài Python rồi.

* Nếu bạn dùng Windows thì link tải ở đây: [Download Python](https://www.python.org/downloads/).

* Còn mình dùng Ubuntu nên chỉ cần gõ:

```
$ sudo apt-get update
$ sudo apt-get install python3.9
```

## Option

Bạn có thể setup **Virtual environments** hoặc không, bạn có thể bỏ qua bước này. Virtual environments có tác dụng quản lý các dependencies của project. Ví dụ bạn có thể cài 1 lib với 2 version khác nhau cho 2 project khác nhau, thậm chí là Python version khác nhau. Mỗi project sẽ có 1 bộ Python libraries riêng biệt, không thằng nào ảnh hưởng đến thằng nào.

Python3 sử dụng module venv để tạo Virtual environments. Bạn có thể thực hiện lệnh sau để tạo folder cho project và venv folder luôn:

```
$ mkdir myproject
$ cd myproject
$ python3 -m venv venv
```

Trên Windows:

```
$ py -3 -m venv venv
```

Run environment của bạn:

```
$ . venv/bin/activate
```

## Flask

Sau đó là cài Flask:

```
$ pip install Flask
```

# Code thôi

## Hello World

Bắt tay vào để code 1 ứng dụng Flask đơn giản nhất nào. Tạo 1 file `hello.py` (tên gì cũng được, miễn là tránh `flask.py` để tránh conflict) và code những dòng đầu tiên:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

Giải thích một chút:

* `Flask(__name__)` có tác dụng tạo 1 instance của class Flask.
* `route()` thì sẽ chỉ cho chúng ta URL của API.

Run app luôn và ngay:

```
$ export FLASK_APP=hello.py
$ flask run
```

Check http://127.0.0.1:5000/ để xem app của bạn đã chạy rồi này.

Nhưng giờ mỗi lần bạn sửa code, refresh browser vẫn chưa thấy code được apply. Thế là thế nào nhỉ??? Đó là vì bạn cần bật Debug mode lên nữa.

```
$ export FLASK_ENV=development
$ flask run
```

Bây giờ thì thử sửa code và xem thành quả của bạn nào.

Bạn cũng có thể xem kết quả của mình trên thiết bị khác (ở đây mình muốn xem trên máy Android) bằng cách thay đổi host (địa chỉ host là IP của laptop/PC bạn đang dùng) và kết nối 2 máy tới cùng 1 mạng wifi:

```
$ flask run --host=192.168.xxx.xxx
```

## JSON

JSON là dạng dữ liệu mà mình hay dùng cho các API. Thử demo một chút JSON xem nào. Đầu tiên là cần import thêm `jsonify`:

```python
from flask import Flask, jsonify
```

Thay vì return `Hello, World!`, chúng ta sẽ return kết quả trả về dưới dạng JSON:

```python
@app.route('/')
def hello_world():
    return jsonify([
        {
            "id": 1,
            "title": "First Memory",
            "description": "This is first Memory"
        },
        {
            "id": 2,
            "title": "Second Memory",
            "description": "This is second Memory"
        },
        {
            "id": 3,
            "title": "Third Memory",
            "description": "This is third Memory"
        }
    ])
```

Nhưng mình thấy các field bị đảo lộn hết cả lên, vậy thì phải thêm một chút config:

```python
app.config['JSON_SORT_KEYS'] = False
```

Ok, có vẻ ổn hơn rồi đấy!

## Ngrok - demo app không cần deploy

Mình là dev Android nên muốn thử xem Flask có dùng làm API cho Retrofit trong Android được không. Thế mà config như trên, máy nhận được API rồi nhưng khi apply vào Retrofit thì vẫn không được. Không hiểu có phải do http không? Nhờ các cao nhân giải đáp giúp đoạn này.

Vậy nên mình sẽ giới thiện với các bạn [**Ngrok**](https://ngrok.com/), một công cụ giúp bạn nhanh chóng demo app Flask mà không cần deploy lên server.

Cài đặt ngrok:

```
$ pip install pyngrok
```

Run lệnh `$ ngrok --help` để chắc chắn rằng chúng ta đã cài đặt thành công.
Sửa code một chút để tạo Ngrok Tunnel nào:

```python
from pyngrok import ngrok
...
url = ngrok.connect(5000).public_url
print('Henzy Tunnel URL:', url)
```

Tiếp tục run `$ flask run` để xem kết quả. Check url được print ra và truy cập trên máy Android. Thử thay vào Url của Retrofit (nhớ dùng https) thì mình thấy nó chạy khá ổn. 

# Lời kết

Trên đây mới là hướng dẫn Quickstart. Nếu các bạn có hứng thú, mình sẽ viết tiếp các bài tiếp theo.

Cảm ơn các bạn đã đọc!

Tham khảo:

* https://flask.palletsprojects.com/en/1.1.x/quickstart/#quickstart
* https://blog.miguelgrinberg.com/post/access-localhost-from-your-phone-or-from-anywhere-in-the-world
