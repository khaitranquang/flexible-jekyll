---
layout: post
title: Homework1 | CyStack Internship 2018
date: 2018-03-18 23:32:20 +0300
description: Homework week 1
img: cystack.png
tags: [CyStack, Homework]
---
Đây là phần đầu tiên trong loạt bài tìm hiểu về microservice của mình. Cũng như các bài viết khác được gắn tag `Learning` thì bài viết này được xây dựng trong quá trình học tập, như là một công cụ ghi chép những kiến thức mình học và tìm hiểu được. Nếu có gì sai sót mong người đọc nhẹ tay :v

Loạt bài sử dụng nguồn tham khảo chính là cuốn sách `Microservice From Design to Deployment - Chris Richardson with Floyd Smith` cùng một số nguồn tài liệu khác.

Và giờ cùng tìm hiểu xem microservice là gì nào! Let's go !!!

## Kiến trúc Monolithic

Đây là kiểu kiến trúc một khối truyền thống. Hình sau miêu tả một ứng dụng cơ bản theo kiểu kiến trúc này:

![Monolithic]({{site.baseurl}}/assets/img/3tier.png)

(*Nguồn ảnh: https://www.jinfonet.com/resources/bi-defined/3-tier-architecture-complete-overview*)

Chẳng có gì xa lạ với sinh viên như mình cả. Hầu như tất cả các bài tập lớn bài tập nhỏ trên lớp thì mình đều phát triển theo kiểu kiến trúc này.
Với ứng dụng web thì tầng trên cùng là tầng trình diễn là các web UI do người dùng tương tác. Client có thể xây dựng bằng rất nhiều các framework khác nhau (React, Vue, ...).
Từ client sẽ gọi lên phía server back end thông qua REST API, thực hiện truy vấn database rồi trả về cho client hiển thị kết quả lên.

Một mô hình rất cơ bản với mọi ứng dụng bất kể mobile, web, hay desktop.

Ưu điểm của kiến trúc này là dễ phát triển, đơn giản (Quá dễ làm với sinh viên như mình :v)

Tuy nhiên nó lại bộc lộ một số khuyết điểm khi ứng dụng dần lớn ra. Tưởng tượng, bạn mới ra trường hoặc thực tập, được kế thừa lại một dự án monolithic cũ, cả một cục code hàng tiệu dòng ném vào mồm, các module chồng chéo :v Rip, bạn chính thức toang :v Bất kể code có được đẹp đến đâu thì cũng không tránh khỏi rối rắm.
Hoặc ứng dụng bạn làm bất ngờ một ngày đẹp trời nào đó tạo thành một cơn sốt, hàng trăm nghìn thậm chí hàng triệu người dùng :)) Và rồi bạn muốn mở rộng phục vụ được số lượng khách hàng, bạn lại phải deploy lại toàn bộ hàng triệu dòng code, khách hàng bị gián đoạn, kêu gào... Và thế là lại không ai dùng ứng dụng của bạn do bảo trì quá lâu, hay bị gián đoạn. Bạn lại RIP tập 2.

Do đó, kiến trúc microservice được ra đời như là một sự khắc phục những nhược điểm của kiến trúc monolithic truyền thống.


## Microservice - Chia nhỏ vấn đề phức tạp

Ý tưởng của microservice đơn giản là sự chia nhỏ ứng dụng lớn thành các service siêu nhỏ. Mỗi service có thể đảm nhận một nhiệm vụ riêng biệt nào đó, có thể coi như một ứng dụng con, có server riêng, có database riêng. Các service này giao tiếp với nhau thông qua mạng (Gửi nhận message qua HTTP hoặc sử dụng MessageQueue)

Để dễ hình dung, anh em có thể xem hình dưới biểu diễn kiến trúc cơ bản tại Cystack Platform (https://app.cystack.net)

![Cystack Microservice]({{site.baseurl}}/assets/img/cystackmicroservice.jpg)

Cystack là nền tảng bảo mật bảo vệ website với 4 app chính là:

* Scanning - Detect vulnerabilities
* Monitoring - Giám sát website
* Responding - Detect and remove malwares in web servers
* Protecting - Như là một firewall bảo vệ website người dùng

Nếu tất cả các ứng dụng trên được xây dựng trong cùng một khối thì thật là chằng chịt. Do đó, ứng dụng được chia thành 4 apps con đảm nhận các thành phần chức năng khác nhau, với một database riêng biêt. 
Người dùng, client chỉ giao tiếp với API Gateway. API Gateway gọi đến các app con thông qua REST API.
Từ đó, ứng dụng trở nên dễ bảo trì hơn.
Hay sau này, bên Cystack phát triển thêm WhiteHub (Sàn bug bounty - Quy tụ hacker mũ trắng :v) thì 4 apps con kia sẽ gộp dưới một app con khác gọi là WEB Protection, WhiteHub trở thành một microservice ngang hàng với WEB Protection. Cực kỳ dễ mở rộng app phải không nào :D

## Ưu nhược điểm của Microservice
### Ưu điểm
* Dễ nâng cấp, scale
* Tách biệt giữa các module. Khi sửa chữa bảo trì thì chỉ một module cần bảo trì, các module khác vẫn hoạt động bình thường.
* Đa dạng được các ngôn ngữ lập trình sử dụng: Các module tùy thích lựa chọn công nghệ, không bị gò bó bởi bất cứ một ngôn ngữ nào cả.
* Quá trình phát triển các module có thể tiến hành song song, không phải đợi chờ nhau. (Tất nhiên vẫn có những module service cần được xây dựng trước :v)

### Nhược điểm
Tất nhiên, không có gì gọi là hoàn hảo cả. Microservice tồn tại một số nhược điểm thấy rõ như:
* Quá trình phát triển trở nên phức tạp hơn, nhiều server nên quản lý phức tạp hơn. 
* Các module giao tiếp với nhau qua mạng nên tốc độ chậm hơn so với monolithic.
* Mỗi service có một database riêng nên tính đồng nhất trong dữ liệu trở thành một vấn đề quan trọng.
* Các service còn phải đảm bảo về vấn đề xác thực phân quyền giữa các service với nhau...


## Tạm kết
Microservice trở thành một xu thế tất yếu, nó khắc phục được những điểm yếu mà kiến trúc một khối truyền thống mắc phải. Tuy nhiên, chi phí phát triển, độ phức tạp của ứng dụng cần được xem xét trước khi sử dụng microservice.
Các module bất kể trong kiến trúc một khối hay microservice cũng nên được chia tách, đảm bảo sự riêng biệt để có thể dễ bảo trì, nâng cấp sau này.

Bài tiếp theo sẽ là về API Gateway trong Microservice.
