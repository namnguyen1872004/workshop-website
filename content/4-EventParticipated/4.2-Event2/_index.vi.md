---
title: "Event 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---


# Bài thu hoạch Hội thảo Công nghệ Cloud & Phát triển Phần mềm hiện đại

### Mục Đích Của Sự Kiện

- Chia sẻ các giải pháp bảo mật nâng cao kết hợp AWS WAF và Học máy (Machine Learning).
- Giới thiệu phương pháp phát triển game đa người chơi qua kiến trúc Serverless WebSockets trên AWS.
- Hướng dẫn xây dựng cơ sở tri thức đồ thị (Graph Knowledge Base) với AWS Neptune phục vụ cho GraphRAG.
- Cung cấp các best practices về công nghệ đóng gói Container (Docker) và kỹ năng cộng tác đội ngũ hiệu quả.
- Định hướng lộ trình phát triển nghề nghiệp thực tế từ IT Helpdesk dịch chuyển sang Cloud-DevOps.

### Danh Sách Diễn Giả

- **Bảo Huỳnh** - Expert về Containerization Technology (Docker)
- **Lê Hoàng Gia Đại** - Expert về An ninh mạng & AI/ML tại AWS
- **Nguyễn Quốc Bảo** - Cloud & Game Developer Specialist
- **Trương Phước** - Team Work & Agile Coach
- **Việt Phát** - AI & Graph Database Engineer
- **Vinh Trần** - Senior Sysadmin & Cloud-DevOps Engineer

### Nội Dung Nổi Bật

#### Docker - A Containerization Technology (Bảo Huỳnh)
- **Khái niệm cốt lõi:** Bản chất của công nghệ đóng gói (Containerization) và cách Docker giải quyết bài toán "chạy được trên máy tôi nhưng lỗi trên server".
- **Kiến trúc Docker:** Cách tối ưu hóa Dockerfile, quản lý Images, Containers và Volumes để lưu trữ dữ liệu bền vững.
- **Tối ưu Development Workflow:** Ứng dụng Docker trong môi trường lập trình cục bộ giúp đồng bộ hóa môi trường giữa các thành viên trong dự án.

#### Combining AWS WAF with Machine Learning for Cyber Attack Detection (Lê Hoàng Gia Đại)
- **Hạn chế của WAF truyền thống:** Tại sao các bộ quy tắc (Ruleset) cố định dựa trên pattern không còn đủ để chống lại các cuộc tấn công mạng tinh vi hiện nay.
- **Tích hợp Học máy (ML):** Phương pháp kết hợp AWS WAF với các mô hình ML để phân tích hành vi traffic, phát hiện bất thường (Anomaly Detection) và ngăn chặn mã độc thời gian thực.
- **Kiến trúc an toàn bảo mật:** Thiết kế hạ tầng phòng thủ thông minh, giảm thiểu tỷ lệ báo động giả (False Positives) trên môi trường AWS.

#### Multiplayer in the Cloud: Connecting Godot Clients with AWS WebSockets (Nguyễn Quốc Bảo)
- **Bài toán kết nối thời gian thực:** Thách thức khi xây dựng hạ tầng mạng cho game multiplayer (đa người chơi) yêu cầu độ trễ thấp.
- **Giải pháp AWS WebSockets:** Sử dụng Amazon API Gateway (WebSocket API) kết hợp với AWS Lambda để quản lý kết nối đồng thời từ client game làm bằng Godot Engine.
- **Đồng bộ trạng thái:** Mô hình lưu trữ và truyền tải dữ liệu game một cách tối ưu chi phí dựa trên kiến trúc Serverless.

#### Cách làm việc nhóm hiệu quả (Trương Phước)
- **Rào cản trong teamwork:** Phân tích những nguyên nhân phổ biến dẫn đến xung đột, thiếu gắn kết và giảm hiệu suất khi làm việc nhóm trong các dự án công nghệ.
- **Chiến lược cải tiến:** Các phương pháp cốt lõi để xây dựng lòng tin, tối ưu hóa quy trình giao tiếp và phân rã công việc một cách khoa học.
- **Ứng dụng mô hình Agile/Scrum:** Cách áp dụng các buổi họp ngắn (Standup), đánh giá (Review) giúp team thích ứng nhanh với các thay đổi của dự án.

#### AWS Neptune for Building a Graph Knowledge Base for GraphRAG (Việt Phát)
- **Sự trỗi dậy của GraphRAG:** Tại sao RAG truyền thống dựa trên Vector Search gặp giới hạn về mặt ngữ cảnh kết nối, và cách GraphRAG giải quyết bằng cách kết hợp cơ sở dữ liệu đồ thị.
- **Ứng dụng AWS Neptune:** Phương pháp thiết kế thực thể (Nodes) và mối quan hệ (Edges) trên dịch vụ quản trị đồ thị AWS Neptune để lưu trữ tri thức.
- **Quy trình truy vấn nâng cao:** Cách trích xuất thông tin có cấu trúc đồ thị đưa vào LLM giúp câu trả lời AI chính xác, sâu sắc và giữ trọn vẹn ngữ cảnh của hệ thống lớn.

#### Từ IT Helpdesk lên Senior Sysadmin: Hành trình tự học & Dịch chuyển sang Cloud-DevOps (Vinh Trần)
- **Câu chuyện thực tế:** Hành trình vượt qua vùng an toàn, từ một kỹ sư IT Helpdesk xử lý sự cố phần cứng/hệ điều hành cơ bản vươn lên vị trí Senior Sysadmin.
- **Lộ trình tự học:** Các chứng chỉ chuyên môn cốt lõi, kỹ năng tự nghiên cứu các công nghệ then chốt cần tập trung để bứt phá.
- **Dịch chuyển sang Cloud-DevOps:** Bản đồ kỹ năng (Skill roadmap) toàn diện bao gồm: Infrastructure as Code (IaC), CI/CD pipelines, Automation scripting và tư duy Cloud-native tại doanh nghiệp.

### Những Gì Học Được

#### Tư Duy Thiết Kế
- **Data-Relationship First Approach:** Hiểu được tầm quan trọng của các mối quan hệ dữ liệu trong Graph Database (AWS Neptune) để phục vụ cho các bài toán AI phức tạp như GraphRAG, thay vì chỉ phụ thuộc vào cơ sở dữ liệu quan hệ truyền thống.
- **Tư duy phòng thủ chủ động (Proactive Security):** Chuyển dịch từ việc cấu hình luật bảo mật bằng tay sang tự động hóa phát hiện bằng trí tuệ nhân tạo (AWS WAF + ML).
- **Trọng tâm kỹ năng mềm (People-centric):** Nhận ra công nghệ chỉ là công cụ, việc tối ưu hóa cách giao tiếp và quy trình làm việc nhóm mới là đòn bẩy cốt lõi giúp dự án đi đến thành công.

#### Kiến Trúc Kỹ Thuật
- Nắm vững cách thiết lập kết nối hai chiều (Bi-directional) thông qua WebSockets trên AWS để giải quyết các bài toán thời gian thực như đồng bộ hóa game engine (Godot).
- Hiểu sâu về cấu trúc cô lập môi trường của Docker, biết cách build image tinh gọn giúp tối ưu hóa dung lượng và tăng tốc độ deploy hệ thống.
- Làm quen với các mô hình triển khai CI/CD và kiến trúc tự động hóa – chìa khóa vàng để dịch chuyển từ Sysadmin truyền thống lên Cloud-DevOps.

#### Lộ trình & Phát Triển Bản Thân
- Có được một lộ trình thăng tiến nghề nghiệp rõ ràng, thực tế từ kinh nghiệm xương máu của người đi trước, giúp định hướng kế hoạch tự học hiệu quả.
- Hiểu rõ phương pháp "chia để trị" và áp dụng tinh thần Agile vào quản lý công việc cá nhân lẫn công việc nhóm.

### Ứng Dụng Vào Công Việc

- **Ứng dụng Docker:** Triển khai Docker hóa toàn bộ môi trường phát triển của project hiện tại nhằm đảm bảo tính đồng nhất giữa Local và Production.
- **Tối ưu hóa GraphRAG:** Nghiên cứu tích hợp cơ sở dữ liệu đồ thị AWS Neptune vào trợ lý AI học thuật của nhóm để nâng cao khả năng trả lời các câu hỏi mang tính tổng hợp, liên kết thông tin.
- **Thử nghiệm WebSockets:** Xây dựng một mô hình nhỏ (POC) kết nối serverless qua WebSockets để phục vụ các tính năng tương tác thời gian thực của ứng dụng Web/App.
- **Cải tiến bảo mật:** Đề xuất và cấu hình thêm các tầng bảo mật tự động thông minh bằng cách tích hợp ML vào tường lửa bảo vệ hệ thống hạ tầng của nhóm.
- **Nâng cao hiệu suất Teamwork:** Áp dụng ngay các kỹ thuật giao tiếp và tổ chức cuộc họp theo hướng Agile để giải quyết triệt để tình trạng phân chia task chồng chéo trong nhóm.

### Trải nghiệm trong event

Tham gia chuỗi sự kiện công nghệ này là một trải nghiệm vô cùng quý giá, giúp tôi không chỉ mở rộng kiến thức kỹ thuật chuyên sâu về Cloud-AI mà còn nhận được những bài học đắt giá về kỹ năng con người và lộ trình sự nghiệp. Một số trải nghiệm nổi bật:

#### Học hỏi từ các góc nhìn thực chiến đa dạng
- Sự kiện kết hợp hài hòa giữa các chủ đề kỹ thuật nặng (AWS Neptune GraphRAG, WAF+ML) cùng với các góc nhìn rất gần gũi với sinh viên như kỹ năng làm việc nhóm hay hành trình tự học dịch chuyển sự nghiệp.
- Các diễn giả chia sẻ trực quan thông qua demo chạy thực tế (như kết nối game Godot với AWS WebSockets), giúp những khái niệm trừu tượng trở nên vô cùng sống động.

#### Định hướng sự nghiệp rõ ràng
- Phiên chia sẻ của anh Vinh Trần đã truyền rất nhiều động lực cho tôi. Việc nhìn thấy một lộ trình rõ ràng từ IT Helpdesk lên Senior Sysadmin và Cloud-DevOps giúp tôi có thêm niềm tin vào con đường tự học công nghệ đám mây.

#### Bài học rút ra
- Đóng gói (Docker) và Bảo mật (WAF+ML) là hai yếu tố cốt lõi không thể thiếu khi xây dựng bất kỳ ứng dụng hiện đại nào trên môi trường Cloud.
- Công nghệ thay đổi rất nhanh (như việc chuyển dịch từ RAG sang GraphRAG), đòi hỏi kỹ sư công nghệ phải luôn giữ vững tinh thần tự học hỏi không ngừng và khả năng thích nghi cao.

#### Một số hình ảnh khi tham gia sự kiện
![Event 2](/workshop-website/images/event2.png)