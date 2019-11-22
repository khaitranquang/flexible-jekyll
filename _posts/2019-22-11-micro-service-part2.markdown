---
layout: post
title: Micro service Part 2 | API Gateway
date: 2019-11-12 22:32:20 +0300
description: Micro service - API Gateway
img: microservice.png
tags: [Architecture, Learning]
---
Trong phần 1, chúng ta đã tìm hiểu qua khái niệm cơ bản về Micro service, lợi ích cũng như hạn chế của nó, Trong phần này, chúng ta sẽ tìm hiểu về thành phần không thể thiếu trong kiến trúc Micro service là **API Gateway**

Come on baby!!!

## Mở đầu

Nhân kỳ này đang gần cuối kỳ 1 của mình tại trường. Mình cũng đã được học về môn `Phát triển phần mềm phân tán`. Qua đó cũng đã được tiến hành một bài tập nho nhỏ để xây dựng một hệ thống trên kiến trúc micro service. Bài tập đơn giản là làm một hệ thống quản lý công việc trong một công ty mới các cập bậc của Giám đốc, Trưởng phòng, Nhân viên trong các phòng ban khác nhau. 

Ok. Đề tài đã có. Chúng ta hoàn toàn có thể xây dựng trên một kiến trúc monolithic đơn giản mà. Nhưng không, mình đang học về hệ phân tán cơ mà, nên chúng ta sẽ cùng xây dựng đề tài trên theo kiến trúc micro service đang tìm hiểu :

Giả sử client yêu cầu lấy thông tin chi tiết công việc (task) của một nhân viên. Các thông tin cần đưa ra bao gồm:
* Thông tin chi tiết công việc (ID, Name, Description, Deadline...)
* Thông tin dự án mà công việc đó nằm trong
* Thông tin nhân viên đang thực hiện
* Các comment, nhận xét của các thành viên trong dự án
* Các file đính kèm nếu có

Ai chà, cũng kha khá thông tin đó chứ nhỉ. Nếu như trong kiến trúc một khối đơn giản thì client sẽ đơn giản chỉ cần gọi đến một API Endpoint để lấy được toàn bộ thông tin về:

`GET api.example.com/v1/task/tasKID`

thì server sẽ query, trả về toàn bộ các thông tin bên trên.

Nhưng trong micro service, chúng ta sẽ xây dựng nên các service "siêu nhỏ" như:
* ProjectTaskService - Thông tin dự án, công việc
* UserService - Thông tin nhân viên
* CommentService - Thông tin comment
* AttachmentService - Thông tin file

Và từ đây, vấn đề bắt đầu đặt ra là làm thế nào để client lấy được hết các thông tin trên?

![Api-Gateway-problem]({{site.baseurl}}/assets/img/api-gateway-intro.png)

Để giải quyết bài toàn liên kết giữa client và các micro services thì chúng ta có 2 giải pháp:

## Option 1: Client gọi thẳng đến Micro service

Quá đơn giản. Mỗi service có một endpoint riêng biệt (**http://serviceName.api.example.com**). Đã có sẵn rồi thì client chỉ việc gọi thôi mà :v

Nhưng rõ ràng việc gọi trực tiếp từ client như thế này là không ổn một tí nào. Chúng ta dễ dàng nhận ra các sự ngu học của cách làm này:
* Client sẽ tốn 4 request để có thể lấy được các thông tin
* Nếu các service con không đơn giản là dùng REST API mà lại là giao thức RPC, hoặc AMQP message thì clien phải tốn công cài đặt lời gọi của các phương thức khác nhau.
* Khi đó, client sẽ bị phình to, trở nên rối tung lên, gây khó bảo trì. 
* Cùng với đó, nếu trong mỗi service lại gọi đến các service khác nhau thì các service rõ ràng lại quá liên kết chặt chẽ với trong (Vi phạm tính độc lập của kiến trúc)

Kết lại, cách tiếp cận này hoàn toàn không ổn. Và ngạc nhiên chưa, giảng viên G đang dạy mình lại cho cả lớp làm cùng một đề tài và ghép với nhau theo cách như thế này. Ae hãy tưởng tượng việc các nhóm chạy loạn trong giờ để yêu cầu dữ liệu mày phải thế này, mày phải thế kia đi :)))

Và từ đó, chúng ta sẽ tiếp cận theo cách 2

## Option 2: API Gateway - Sự cứu rỗi 

Thay vì client phải tự giao tiếp với các micro service (như việc chạy tứ tung trong lớp để hỏi dữ liệu của các nhóm khác) thì client sẽ chỉ giao tiếp duy nhất với 1 bên là API Gateway. API Gateway sẽ có trách nhiện điều phối, gọi đến các service theo trình tự như nào, tổng hợp kết quả và trả ra cho client.

![Api-Gateway]({{site.baseurl}}/assets/img/api-gateway.png)

Ngoài ra, API Gateway có thể thực hiện một số công việc như xác thực (authentication), giám sát (monitoring), cân bằng tải (load balancing), caching, ...

Bên trong API Gateway sẽ thực hiện tất cả các công việc như gọi đến service nào, phương thức gọi là gì, tổng hợp kết quả ra sao, và cuối cùng sẽ trả ra các dữ liệu mà client cần. Anh em có thể tham khảo thêm về Netflix API

## Ưu điểm API Gateway

* Đơn giản hóa việc giao tiếp cho client: giảm số lượng request, response; đơn giản hóa code
* Đóng gói được ứng dụng thành một khối, che giấu đi sự tồn tại của các service bên trong

## Nhược điểm

Không có gì là toàn năng, việc xây dựng API Gateway cũng tồn tại một số nhược điểm như:
* Server dựng API Gateway phải đủ khỏe, chịu tải tốt
* Việc mọi giao tiếp đi qua Gateway có thể gây nghẽn cổ chai (bottleneck), cần đảm bảo tránh việc gói tin nghẽn, không được xử lý, hoặc xử lý quá lâu.
* Gateway phải liên tục được cập nhật thay đổi theo sự thay đổi của các micro service liên quan.

## Vấn đề trong triển khai thực tế

Một số vấn đề cần lưu tâm khi triển khai:

### Hiệu suất và khả năng mở rộng

Như đã nói ở trên, API Gateway phải là một server đủ lớn, có sự chịu tải cao, khả năng mở rộng tốt. Cần phối hợp các công nghệ bất đồng bộ, non-blocking để đảm bảo hiệu năng

### Định tuyến 

API Gateway cần xác định rõ địa chỉ micro service mà mình gọi đến: phương thức gọi (qua REST API hay là AMQP message...), địa chỉ gọi (domain), tính đồng bộ hay bất đồng bộ...

### Hỗ trợ nhiều cách thức giao tiếp

Micro service có thể gọi được qua nhiều cách khác nhau: 
* Bất đồng bộ: Message broker,..
* Đồng bộ: HTTP,...

Do đó, API Gateway phải hỗ trợ nhiều các giao tiếp khác nhau

### Service Discovery

Thường thì các micro service sẽ có một địa chỉ rõ ràng để gọi đến. Tuy nhiên trong một số trường hợp, các services con lại không tĩnh, thay đổi liên tục để phù hợp mục đích autoscaling và upgrade. Nên API Gateway cần một cơ chế để xác định được địa chỉ của các service động này.

### Xử lý lỗi

Tránh để một lời gọi bị treo khi một service lỗi. Cần có cơ chế xử lý, thông báo lỗi lại cho client.

Có thể trả dữ liệu trong cache có sẵn tùy theo trường hợp.

## Kết luận

Tuy có nhiều vấn đề, nhưng rõ ràng việc tiếp cận theo hướng sử dụng API Gateway là hoàn toàn cần thiết trong một hệ thống kiển trúc micro service. Nó đảm bảo cho client đơn giản, tăng tính thống nhất dữ liệu, che giấu đi được các thành phàn bến trong của hệ thống.

Phần tiếp theo sẽ về **Inter-Process Communication**