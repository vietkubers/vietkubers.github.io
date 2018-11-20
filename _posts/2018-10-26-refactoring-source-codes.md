---
layout: post
title: Refactoring source codes (Tái cấu trúc mã nguồn)
date: 2018-10-26
categories: [python]
tags: [python]
---

***Refactoring*** được Việt hoá có nghĩa là: *Tái cấu trúc*. Vậy trong kĩ thuật lập trình, *tái cấu trúc mã nguồn* là gì?  

Để trả lời câu hỏi trên ta lấy ví dụ sau:

***remember_me.py***
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
Chương trình trên sẽ được chạy thành công, nếu lần đầu chương trình được chạy, ta được:
```
What's your name? Nguyen Hai Truong
We'll remember you when you come back, Nguyen Hai Truong!
```
Hoặc:
```
Welcome back, Nguyen Hai Truong!
```
Thử thay đổi mã nguồn trên một chút bằng cách chuyển toàn bộ logic của chương trình vào hàm `greet_user()`:

***remember_me_v2.py***
```python
import json

def greet_user():
	"""Greet the user by name"""
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

greet_user()
```
Chương trình nhìn có vẻ "sáng sủa" và gọn gàng hơn. Hàm `greet_user()` làm nhiều hơn là chỉ thực hiện "greeting the user" - nó cũng truy xuất username được lưu trữ nếu người dùng đã tồn tại và yêu cầu nhập username mới nếu người dùng không tồn tại.

**Vậy tái cấu trúc là công việc cải thiện mã nguồn (source codes) bằng cách định nghĩa các hàm thực hiện một chức năng cụ thể, giúp cho codes "sạch sẽ" hơn, dễ dàng đọc hiểu và mở rộng (triển khai) hơn.**

Bây giờ hãy **refactor** hàm `greet_user()` để nó không phải thực hiện quá nhiều chức năng cùng lúc:

***remember_me_v3.py***
```python
import json

def get_stored_username():
	"""Get stored username if available"""
	filename = 'username.json'
	try:
		with open(filename) as f_obj:
			username = json.load(f_obj)
	except FileNotFoundError:
		return None
	else:
		return username

def greet_user():
	"""Greet the user by name"""
	username = get_stored_username()
	if username:
		print("Welcome back, " + username + "!")
	else:
		username = input("What's your name? ")
		filename = 'username.json'
		with open(filename, 'w') as f_obj:
			json.dump(username, f_obj)
			print("We'll remember you when you come back, " + username + "!")

greet_user()
```

Hàm `get_stored_username()` có một mục đích rõ ràng: hàm này sẽ truy xuất username và trả về username nếu nó tìm thấy. Trong trường hợp file *username.json* không tồn tại, hàm trả về giá trị `None`.  

Đoạn code trên vẫn chưa thực sự tối ưu, chương trình nên có thêm hàm `get_new_username()`

***remember_me_v4.py***
```python
import json

def get_stored_username():
	"""Get stored username if available"""
	filename = 'username.json'
	try:
		with open(filename) as f_obj:
			username = json.load(f_obj)
	except FileNotFoundError:
		return None
	else:
		return username

def get_new_username():
	"""Prompt for a username"""
	username = input("What's your name? ")
	filename = 'username.json'
	with open(filename, 'w') as f_obj:
		json.dump(username, f_obj)
	return username

def greet_user():
	"""Greet the user by name"""
	username = get_stored_username()
	if username:
		print("Welcome back, " + username + "!")
	else:
		username = get_new_username()
		print("We'll remember you when you come back, " + username + "!")

greet_user()
```

*Nguồn: [Python Crash Course: A Hands-On, Project-Based Introduction to Programming](https://www.amazon.com/Python-Crash-Course-Hands-Project-Based/dp/1593276036)*
