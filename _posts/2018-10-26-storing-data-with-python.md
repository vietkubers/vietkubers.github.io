---
layout: post
title: Python, storing Data using the JSON
date: 2018-10-26
categories: [python]
tags: [python]
---

Dữ liệu của một chương trình viết bằng [Python](https://www.python.org/) có thể chứa nhiều loại thông tin khác nhau, chẳng hạn như một game sẽ yêu cầu người chơi nhập vào username và chương trình này sẽ lưu lại điểm số sau mỗi lượt chơi, các tuỳ chọn thiết lập trong game... Dù thông tin ở dạng nào đi nữa thì chúng cũng sẽ được lưu trong các cấu trúc dữ liệu mà ngôn ngữ lập trình Python hỗ trợ như: danh sách (**lists**) hoặc từ điển (**dictionaries**). Khi người dùng đóng chương trình, những thông tin được tạo ra trong quá trình chạy chương trình (phải) được lưu lại. Một biện pháp đơn giản để thực hiện việc này là lưu trữ data bằng cách sử dụng mô-đun (module) **json**.  

Mô-đun **json** cho phép ***dump*** (từ này mình không biết nên Việt hóa như thế nào nữa? :D) một cấu trúc dữ liệu Python đơn giản vào một file và tải (load) dữ liệu từ file đó vào lần chạy tiếp theo của chương trình. Người phát triển có thể dùng **json** để chia sẻ dữ liệu giữa các chương trình Python khác nhau. Một ưu điểm vượt trội khác là kiểu dữ liệu **JSON** được thiết kế để không chỉ dùng riêng cho Python nên người phát triển có thể chia sẻ dữ liệu lưu trữ ở định dạng JSON với những người phát triển ngôn ngữ lập trình khác, nó rất hữu ích và khả chuyển (portable).

> JSON (JavaScript Object Notation): Ban đầu JSON được phát triển cho JavaScript. Tuy nhiên, nó đã trở nên phổ biến và được dùng bởi nhiều ngôn ngữ lập trình khác, trong đó có Python.


**Trong bài viết:**

<!-- MarkdownTOC -->
[1. json.dump() và json.load()](#1-json-dump-load)  
[2. Lưu trữ và đọc dữ liệu User-Generated](#2-saving-reading-user-generated-data)  
<!-- /MarkdownTOC -->

<a name="1-json-dump-load"><a/>
## 1. json.dump() và json.load()
Chương trình **number_writer.py** lưu trữ một tập hợp các số và chương trình **number_reader.py** đọc những con số này ngược trở lại bộ nhớ. Chương trình thứ nhất sẽ sử dụng `json.dump()` để lưu trữ tập hợp số và chương trình thứ hai sử dụng `json.load()`.  

Hàm `json.dump()` nhận 2 đối số (arguments):
* Dữ liệu lưu trữ
* Đối tượng file dùng để lưu trữ dữ liệu.  

*number_writer.py*
```python
import json

numbers = [2, 3, 5, 7, 11, 13]

filename = 'numbers.json'
with open(filename, 'w') as f_obj:
	json.dump(numbers, f_obj)
```
Hàm `json.dump()` sẽ lưu trữ danh sách `numbers = [2, 3, 5, 7, 11, 13]` vào tệp *numbers.json*. Chương trình phía trên không có output nào ra console, nhưng nó sẽ tạo ra tệp *numbers.json* lưu trữ danh sách numbers theo định dạng giống như Python:
```
$ python number_writer.py
$ cat numbers.json
[2, 3, 5, 7, 11, 13]
```

Tiếp theo chương trình thứ hai **number_reader.py** sử dụng `json.load()` để đọc danh sách numbers ngược trở lại bộ nhớ.  

*number_reader.py*
```python
import json

filename = 'numbers.json'
with open(filename) as f_obj:
	numbers = json.load(f_obj)

print(numbers)
```
Hàm `json.load()` đọc thông tin lưu trữ trong tệp *numbers.json* và lưu thông tin vào biến numbers, chương trình sẽ in ra danh sách các số giống như danh sách được tạo ở **number_writer.py**
```
$ python number_reader.py
[2, 3, 5, 7, 11, 13]
```
Nội dung trên đã trình bày một cách đơn giản để chia sẻ dữ liệu giữa 2 chương trình.

<a name="2-saving-reading-user-generated-data"><a/>
## 2. Lưu trữ và đọc dữ liệu User-Generated
Lưu dữ liệu bằng **json** thực sự hữu ích khi làm việc với dữ liệu do người dùng tạo ra (User-Generated Data), bởi vì nếu không lưu thông tin của người dùng bằng một cách nào đó sẽ dẫn đến sự mất mát thông tin khi chương trình ngừng chạy. Ví dụ dưới đây sẽ lưu tên của người dùng nhập vào ở lần chạy đầu tiên và sau đó nhớ tên của họ ở những lần chạy tiếp theo.  

*remember_me.py*
```python
import json

username = input("What's your name? ")

filename = 'username.json'
with open(filename, 'w') as f_obj:
	json.dump(username, f_obj)
	print("We'll remember you when you come back, " + username + "!")
```
Khi chạy chương trình trên, username sẽ được lưu vào file *username.json* và in ra màn hình thông báo rằng chương trình đã lưu tên người dùng cho lần chạy tiếp theo.
```
$ python3 remember_me.py
What's your name? TruongNH
We'll remember you when you come back, TruongNH!
$ cat username.json
"TruongNH"
```
Chương trình mới dưới đây sẽ in ra lời chào với người dùng mà username đã được lưu trữ trước đó.   

*greet_user.py*
```python
import json

filename = 'username.json'

with open(filename) as f_obj:
	username = json.load(f_obj)
	print("Welcome back, " + username + "!")
```
```
$ python3 greet_user.py
Welcome back, TruongNH!
```

Gộp hai chương trình trên vào một file mã nguồn, khi chạy chương trình *remember_me.py*, người dùng muốn lấy username từ bộ nhớ nếu có thể; do đó người phát triển sẽ dùng một khối `try`. Nếu file *username.json* không tồn tại, sẽ có khối `except` nhắc lệnh truyền một username vào và lưu tại file *username.json* dùng cho lần sau:

*remember_me.py*
```python
import json

# Load the username, if it has been stored previously.
# Otherwise, prompt for the username and store it

filename = 'username.json'
try:
	with open(filename) as f_obj:
		username = json.load(f_obj)
except FileNotFoundError:
	username = input("What's your name? ")
	with open(filename, 'w') as f_obj:
		json.dump(username, f_obj)
		print("We'll remember you when you come back, " + username + "!")
else:
	print("Welcome back, " + username + "!")
```
Nếu lần đầu chương trình trên được chạy: `$ python3 remember_me.py`
```
What's your name? Nguyen Hai Truong
We'll remember you when you come back, Nguyen Hai Truong!
```
Hoặc:
```
Welcome back, Nguyen Hai Truong!
```

*Nguồn: [Python Crash Course: A Hands-On, Project-Based Introduction to Programming](https://www.amazon.com/Python-Crash-Course-Hands-Project-Based/dp/1593276036)*
