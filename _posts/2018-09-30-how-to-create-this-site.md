---
layout: post
title: Cách tạo một website cá nhân với Github Pages
date: 2018-09-30
categories: [tutorials]
mathjax: true
tags: [tutorials]
---

Bài viết này sẽ hướng dẫn các bạn cách xây dựng một website cá nhân bằng [Github Pages](https://pages.github.com). Với những bạn kỹ sư phần mềm hoặc lập trình viên đã quen làm việc với github thì mình tin rằng các bạn sẽ chỉ mất khoảng 30 phút để tạo một website cho riêng mình, còn những bạn học ngành khác thì có thể sẽ mất nhiều thời gian hơn... chắc khoảng một giờ đồng hồ gì đó :)

**Trong bài viết:**

<!-- MarkdownTOC -->
- [1. Môi trường phát triển](#1-moi-truong-phat-trien) 
	- [1.1. Server website](#server-website)  
	- [1.2. Hệ điều hành](#he-dieu-hanh)  
	- [1.3. Trình soạn thảo](#trinh-soan-thao)  
- [2. Các tính năng bổ sung của website](#2-cac-tinh-nang-bo-sung-cua-website)  
	- [2.1. Một số cú pháp markdown cơ bản](#mot-so-cu-phap-markdown-co-ban)  
	- [2.2. Mục lục với markdown TOC](#muc-luc-voi-markdown-toc)  
	- [2.3. Comments dưới mỗi bài viết](#comments-duoi-moi-bai-viet)  
	- [2.4. Phân tích dữ liệu trang web](#phan-tich-du-lieu-trang-web)  
- [3. Tạo một bài viết mới](#3-tao-mot-bai-viet-moi)  
<!-- /MarkdownTOC -->

<a name="1-moi-truong-phat-trien"><a/>
## 1. Môi trường phát triển

<a name="server-website"><a/>
### 1.1. Server website

Website cá nhân của mình là: https://truongnh1992.github.io hoạt động với tên miền `github.io` - tên miền mặc định của [Github Pages](https://pages.github.com). Mình nghĩ rằng nếu bạn học ngành IT thì chắc hẳn sẽ quen với việc sử dụng [git](https://git-scm.com) để quản lý các phiên bản mã nguồn (source codes) của các project và https://github.com là một trang rất nổi tiếng để lưu trữ và quản lý source codes đó.  

Toàn bộ mã nguồn của website này đều được công bố [tại đây](https://github.com/truongnh1992/truongnh1992.github.io). Website được tạo trên nền tảng [Jekyll](https://jekyllrb.com) hỗ trợ web tĩnh, [markdown](https://en.wikipedia.org/wiki/Markdown) và có khả năng tích hợp HTML, CSS.


<a name="he-dieu-hanh"><a/>
### 1.2. Hệ điều hành

Hệ điều hành mình sử dụng là [GNU/Linux](https://www.debian.org/releases/stable/amd64/ch01s02.html.vi) (Debian, Fedora, Ubuntu) và MacOS.

<a name="chuong-trinh-soan-thao"><a/>
### 1.3. Chương trình soạn thảo

Editor được khuyên dùng [Sublime Text](https://www.sublimetext.com/3) và [vim](https://www.vim.org).

<a name="2-cac-tinh-nang-bo-sung-cua-website"><a/>
## 2. Các tính năng bổ sung của website

<a name="mot-so-cu-phap-markdown-co-ban"><a/> 
### 2.1. Một số cú pháp markdown cơ bản  

Dưới đây là một số cú pháp thường dùng, các bạn có thể xem thêm các cú pháp nâng cao [tại đây](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).  

* **In nghiêng**
```
*Nguyễn Hải Trường*
```
*Nguyễn Hải Trường*

* **In đậm**
```
**Đại học Bách Khoa Hà Nội**
```
**Đại học Bách Khoa Hà Nội**

* **Chèn link**
```
[My Tech Blog](https://truongnh1992.github.io)
```
[My Tech Blog](https://truongnh1992.github.io)

* **Chèn hình ảnh vào bài viết**
```
![My workingspace](../img/workingspace.jpg)
```
![My workingspace](/static/img/workingspace.jpg)

* **Nhúng code vào bài viết**  
Để hiển thị được source codes với syntax highlighting của mỗi ngôn ngữ lập trình, ta chèn đoạn code trong cặp dấu:
````
```programming language
Enter your source codes here
```
````
Ví dụ:  
````
```java
public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello, World");
    }
}
```
````
Ta được:
```java
public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello, World");
    }
}
```

<a name="muc-luc-voi-markdown-toc"><a/>
### 2.2. Mục lục với markdown TOC

Trong bài viết này, để đánh mục lục, mình cài gói [Markdown TOC](https://github.com/jonschlinkert/markdown-toc) cho Sublime Text hoặc có thể viết với cú pháp sau:

```
<!-- MarkdownTOC -->
- [1. Môi trường phát triển](#1-moi-truong-phat-trien) 
	- [1.1. Server website](#server-website)  
	- [1.2. Hệ điều hành](#he-dieu-hanh)  
	- [1.3. Trình soạn thảo](#trinh-soan-thao)  
- [2. Các tính năng bổ sung của website](#2-cac-tinh-nang-bo-sung-cua-website)  
	- [2.1. Một số cú pháp markdown cơ bản](#mot-so-cu-phap-markdown-co-ban)  
	- [2.2. Mục lục với markdown TOC](#muc-luc-voi-markdown-toc)  
	- [2.3. Comments dưới mỗi bài viết](#comments-duoi-moi-bai-viet)  
	- [2.4. Phân tích dữ liệu trang web](#phan-tich-du-lieu-trang-web)  
- [3. Tạo một bài viết mới](#3-tao-mot-bai-viet-moi)  
<!-- /MarkdownTOC -->

<a name="1-moi-truong-phat-trien"><a/>
## 1. Môi trường phát triển

<a name="server-website"><a/>
### 1.1. Server website
...
```

<a name="comments-duoi-moi-bai-viet"><a/>
### 2.3. Comments dưới mỗi bài viết

Nền tảng Jekyll hỗ trợ comments dùng công cụ [Disqus](https://disqus.com) và [Facebook](https://developers.facebook.com/docs/plugins/comments) comments.
Đăng kí tài khoản [Disqus](https://disqus.com) và thêm Disqus shortname của bạn vào tham số `disqus` trong file `_config.yml`. Và giờ đây, website của bạn đã hỗ trợ tính năng comments dưới mỗi bài viết rồi.

<a name="phan-tich-du-lieu-trang-web"><a/>
### 2.4. Phân tích dữ liệu trang web

Để thống kê tổng số lượt truy cập, số lượt truy cập theo vùng địa lý, theo độ tuổi,... giá trị trang web... các bạn có thể dùng công cụ [Google Analytics](https://marketingplatform.google.com/about/analytics/). Tích hợp tính năng này vào website bằng cách thêm Google Tracking ID của bạn vào `google_analytics` trong file `_config.yml`.
	
<a name="3-tao-mot-bai-viet-moi"><a/>
## 3. Tạo một bài viết mới

Mã nguồn của [bài viết này](https://raw.githubusercontent.com/truongnh1992/truongnh1992.github.io/master/_posts/2018-09-30-how-to-create-this-site.md) nằm trong thư mục [\_posts](https://github.com/truongnh1992/truongnh1992.github.io/tree/master/_posts).

Tương tự, để tạo mới một bài viết bất kì, các bạn tạo một file có tên theo cú pháp `yyyy-mm-dd-ten-bai-viet.md` trong thư mục `_posts`, và phần khai báo như sau:  
```
---
layout: post
title: Cách tạo một website cá nhân với Github Pages
date: 2018-09-30
tags: blog github website github-page
---
```

### Kết
Trên đây mình đã tóm lược các bước để xây dựng website cá nhân với Github Pages, website dạng này rất thích hợp để viết Tech Blog. Hi vọng bài viết sẽ giúp ích được mọi người :)
