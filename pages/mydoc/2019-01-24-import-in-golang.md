---
title: Import trong Golang
tags: [go]
keywords: go
last_updated: January 24, 2019
summary: "Import in Golang"
sidebar: mydoc_sidebar
permalink: 2019-01-24-import-in-golang.html
folder: mydoc
---


Khi vọc source codes Kubernetes, bạn có thể thấy những dòng import như sau:
```go
import (
	"fmt"
	
	"k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/kubernetes/pkg/master/ports"
	"k8s.io/kubernetes/test/e2e/framework"
	imageutils "k8s.io/kubernetes/test/utils/image"

	. "github.com/onsi/ginkgo"
	. "github.com/onsi/gomega"
	_ "k8s.io/kubernetes/pkg/api/testapi"
)
```
Những "ký hiệu" như **imageutils**, **.** , **_** sẽ khiến ta bỡ ngỡ khi lần đầu tiếp xúc với Go, vậy chúng là gì và chức năng như nào thì ta cùng tìm hiểu nhé.

### 1. Simple import

Một trong những chương trình đầu tiên khi ta học một ngôn ngữ là print ra 1 chuỗi như sau:
```go
import "fmt"

func main() {
    fmt.Println("Hello world!")
}
```
Khi import fmt package, mọi structs và functions trong package đó sẽ được sử dụng với *fmt.* như fmt.Println ở trên.

### 2. Multiple package import

Để import nhiều package, thay vì gõ nhiều lần import từng package, ta wrap các package lại với *import()*. 
Ví dụ:
```go
import (
    "fmt"
    "bytes"
)
```

### 3. Import alias

Alias cung cấp cho dev một các viết ngắn gọn và dễ nhớ khi dùng package.
Hãy xem xét một đoạn import sau:
```go
import(
    "math/rand"
    "crypto/rand"
)
```
Việc call rand dễ gây ra conflict giữa 2 packages, vì thế trong case này dùng alias là rất hợp lý
```go
import(
    "math/rand"
    crand "crypto/rand"
)
```
### 4. Dot import

Với những dev đã từng dùng Python thì có thể hình dung dot import như sau:

Go's import "os" == Python's import os

Go's import . "os" == Python's from os import *

Format thường dùng là PackageName "." exported_identifier. 
Khi sử dụng dot import thì ta có thể call các exported identifier mà không cần gọi package name

*Note*: exported_identifier: variable/function/... của package mà được exported (public)*

*Nhược điểm*: Khó đọc code cho dev vì không biết được func, struct được call từ đâu. Đa số ý kiến từ community là hạn chế dùng thằng này

Ref: https://golang.org/ref/spec#Import_declarations

### 5. Blank import
- Hạn chế việc optimize tự động của Go đối với những unused import. Tức là khi dùng blank import thì bạn vẫn có thể run chương trình khi chưa dùng package đã import.
- Việc sử dụng Blank import đồng nghĩa với việc tạo ra các package-level variables và execute function *init* của package đó. Theo định nghĩa của Golang.org thì việc sử dụng như này là "import a package solely for its side-effects (initialization)". 

Ref: 
- https://golang.org/ref/spec#Import_declarations
- https://stackoverflow.com/questions/21220077/what-does-an-underscore-in-front-of-an-import-statement-mean-in-golang

Ví dụ:

```go
package main

import (
	_ "fmt"
)
func main() {
}

```
Chương trình vẫn run bình thường mà không auto remove unused import.

### 6. Coding convention trong import

- Import nên được nhóm theo group, cách nhau bởi 1 blank line
- Các package standard luôn đặt trên top, dưới là các third party packages
- Ví dụ:

```go
package main

import (
	"fmt"
	"hash/adler32"
	"os"

	"appengine/foo"
	"appengine/user"

        "github.com/foo/bar"
	"rsc.io/goversion/version"
)
```

### 7. Use case

Để hình dung rõ hơn về việc sử dụng import trong Go, mình sẽ giới thiệu với các bạn một ví dụ nhỏ.
Đây là một commit trong sourcecode của k8s, đã được merged. Các bạn có thể tham khảo tại [PR](https://github.com/kubernetes/kubernetes/pull/72014).
- Problem: Việc mình cần làm là move *pkg/scheduler/algorithmprovider/defaults/compatibility_test.go* sang *pkg/scheduler/api*
- Solution: 
	- Step 1. Mình tạo thêm package compatibility để tránh lỗi *import cycle* và để tổ chức test case gọn gàng dễ hình hơn.
	
	- Step 2. Sau khi move file *compatibility_test.go* sang package mới, chạy thử test case, mình gặp phải lỗi *Predicate type not found for MatchNodeSelector*. Dò code trong file test thì khác nhiều chỗ call tới predicate type này. Vậy mình sẽ phải dò xem "MatchNodeSelector" từ đâu mà ra. Sau một hồi tìm kiếm thì mình thấy các predicates và priorities được init tại *pkg/scheduler/algorithmprovider/defaults/register_predicates.go*. Vậy mình sẽ cần phải import thằng này vào.
	
	- Step 3. Trên đoạn import, mình thêm "k8s.io/kubernetes/pkg/scheduler/algorithmprovider/defaults". Nhưng mình lại gặp lỗi "Unused import...". Oh, đến đây mình có thể vận dụng kiến thức trên rồi đây. Ở đây mình sẽ dùng Blank import, vì được một công đôi việc.
	
	- Step 4. Sau khi sửa thành _ "k8s.io/kubernetes/pkg/scheduler/algorithmprovider/defaults", tất cả các test case đã Passed.

### 8. Refer

- https://github.com/golang/go/wiki/CodeReviewComments#import-dot
- https://scene-si.org/2018/01/25/go-tips-and-tricks-almost-everything-about-imports/

Author: [huynq](https://github.com/huynq0911)
