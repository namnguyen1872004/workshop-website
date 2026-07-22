---
title: "Event 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---


# Bài thu hoạch Hội thảo Công nghệ và Trải nghiệm Hackathon

### Mục Đích Của Sự Kiện

- Chia sẻ tư duy tối ưu hóa ngữ cảnh (Context) giúp AI hoạt động hiệu quả trong thực tế.
- Giới thiệu bộ giải pháp Amazon Quick giúp khám phá dữ liệu và xây dựng workflow bằng ngôn ngữ tự nhiên.
- Hướng dẫn xây dựng nền tảng hạ tầng vững chắc, bảo mật và tối ưu chi phí với Amazon CloudFront.
- Chia sẻ trải nghiệm thực tế từ hành trình 36 giờ giành giải tại cuộc thi LotusHacks với sản phẩm UTMorpho.
- Phân tích chuyên sâu về tính bất định (Non-Determinism) của LLM và kiến trúc hệ thống Multi-Agent ứng dụng trong doanh nghiệp.

### Danh Sách Diễn Giả

- **Tinh Truong** - Speaker về Context & AI Brain Concept
- **Anh Pham** - Speaker về Amazon Quick Ecosystem
- **Thinh Nguyen** - Expert về Amazon CloudFront & Infrastructure
- **Team VIB** - Đội ngũ phát triển dự án UTMorpho tại LotusHacks
- **Duc Dao** - Expert về Cơ chế hoạt động của LLM
- **Vy Lam** - Expert về Hệ thống Enterprise Multi-Agent

### Nội Dung Nổi Bật

#### Context Is Everything: Making AI Actually Work for You (Tinh Truong)
- **Tầm quan trọng của Ngữ cảnh:** Lý do tại sao AI thường thất bại nếu thiếu context và định nghĩa bản chất của "ngữ cảnh" trong vận hành AI.
- **Sự tiến hóa của AI:** Xu hướng chuyển dịch từ kỹ thuật viết prompt đơn thuần sang cơ chế lưu trữ bộ nhớ (Khái niệm "Bộ não AI thứ hai" - Second AI Brain).
- **Tối ưu kết quả:** Tư duy cốt lõi và các mẹo thực tế giúp tận dụng tốt context để cho ra kết quả AI chính xác nhất.
- **Góc nhìn nghề nghiệp:** Định hướng và lời khuyên giúp sinh viên bắt đầu xây dựng ứng dụng với AI.

#### Friendly AI Assistant with Amazon Quick (Anh Pham)
- **Quick Chat Agent:** Trợ lý AI hỗ trợ khám phá dữ liệu và phân tích chuyên sâu các thông tin chi tiết (insights).
- **Quick Flows:** Tạo lập các quy trình làm việc thông minh trực tiếp bằng ngôn ngữ tự nhiên mà không cần viết code (No-code).
- **Quick Spaces:** Không gian cộng tác chia sẻ, giúp chuyển đổi các phát hiện của cá nhân thành tri thức chung của toàn đội ngũ.
- **Quick Sight:** Xây dựng các bảng điều khiển (dashboards) và báo cáo trực quan từ dữ liệu thô thông qua câu lệnh ngôn ngữ tự nhiên.

#### From Edge To Origin: CloudFront as Your Foundation (Thinh Nguyen)
- **Giải pháp toàn diện:** Ứng dụng Amazon CloudFront cho mọi loại workload từ phân phối nội dung tĩnh đến động.
- **Tối ưu chi phí:** Các chiến lược quản lý và cắt giảm chi phí hạ tầng hiệu quả với CloudFront.
- **Bảo mật chuyên sâu:** Các tính năng bảo mật tích hợp tại vùng biên (Edge location).
- **Độ tin cậy và Hiệu năng:** Nâng cao trải nghiệm người dùng cuối nhờ khả năng tăng tốc độ truy cập và đảm bảo hệ thống luôn sẵn sàng.

#### 36 hrs with LotusHacks – Building UTMorpho from Idea to Reality (Team VIB)
- **Động lực tham gia:** Lý do Team VIB quyết định thử sức tại sân chơi LotusHacks.
- **Hành trình Brainstorming:** Quá trình đi từ con số 0 đến khi hình thành ý tưởng, xác định bài toán cốt lõi để định hình nên sản phẩm UTMorpho.
- **Áp lực thời gian:** Trải nghiệm thực tế trong 36 giờ sprint phát triển sản phẩm liên tục.
- **Bước chuyển mình:** Những thách thức, thất bại gặp phải trên đường đua và cách đội ngũ xoay chuyển tình thế để hoàn thiện bản Demo sản phẩm.

#### Non-Determinism of "Deterministic" LLM Settings (Duc Dao)
- **Cơ chế chọn Token:** Cách các mô hình ngôn ngữ lớn (LLM) tính toán và lựa chọn token tiếp theo trong chuỗi văn bản.
- **Lầm tưởng phổ biến:** Giả định cho rằng việc thiết lập thuộc tính `Temperature=0` sẽ đảm bảo 100% kết quả đầu ra luôn cố định (deterministic).
- **Thực tế vận hành:** Các kỹ thuật tối ưu hóa trong quá trình Inference (Suy luận) làm thay đổi tính cố định này.
- **Tác động & Giải pháp:** Các ảnh hưởng thực tế đến ứng dụng doanh nghiệp và chiến lược giảm thiểu tính bất định của mô hình.

#### Enterprise-Grade Multi-Agent System: Credit Scoring Case Study (Vy Lam)
- **Bài toán hệ thống:** Sự bất đối xứng về mặt cấu trúc dữ liệu giữa hệ thống ngân hàng truyền thống và dữ liệu của các startup.
- **Single Agent vs Multi-Agent:** Khi nào nên dùng một Agent độc lập và khi nào bắt buộc phải chuyển dịch sang mô hình Multi-Agent.
- **Virtual Credit Committee:** Sơ đồ kiến trúc (Blueprint) của một Hội đồng tín dụng ảo vận hành bằng AI.
- **Hành lang an toàn:** Thiết lập các Guardrails bảo vệ và đảm bảo tính tuân thủ (Compliance) trong tài chính.
- **Hiệu quả đầu tư:** Đo lường chỉ số ROI trong vận hành và lộ trình triển khai thực tế cho doanh nghiệp.

### Những Gì Học Được

#### Tư Duy Thiết Kế
- **Context-first Mindset:** Khi làm việc với AI, dữ liệu cấu trúc và ngữ cảnh xung quanh quan trọng hơn rất nhiều so với việc chỉ tối ưu câu lệnh (prompt) đơn thuần.
- **Tư duy No-code/Low-code:** Sử dụng các công cụ thông minh như Amazon Quick giúp rút ngắn thời gian từ ý tưởng đến sản phẩm, tối ưu hóa khả năng làm việc nhóm.
- **Hạ tầng hướng Edge:** Luôn cân nhắc việc đưa nội dung và các lớp bảo mật ra gần người dùng nhất (sử dụng CDN/CloudFront) thay vì chỉ tập trung vào máy chủ gốc (Origin).

#### Kiến Trúc Kỹ Thuật
- Hiểu rõ cơ chế chọn token của LLM để không bị phụ thuộc hoàn toàn vào thiết lập `Temperature=0`, từ đó chuẩn bị các phương án xử lý dữ liệu đầu ra linh hoạt hơn.
- Nắm vững kiến trúc phối hợp Multi-Agent (Nhiều tác nhân thông minh), biết cách phân rã một quy trình nghiệp vụ phức tạp thành các vai trò chuyên biệt của từng Agent.
- Cách thiết lập hệ thống cảnh báo, rào chắn bảo mật (Guardrails) khi ứng dụng Generative AI vào các ngành có tính tuân thủ cao như Tài chính - Ngân hàng.

#### Chiến Lược Phát Triển Dự Án (Từ Hackathon đến Doanh nghiệp)
- Kỹ năng quản lý thời gian và quản trị rủi ro dưới áp lực cực lớn (bài học từ Sprint 36 giờ của dự án UTMorpho).
- Biết cách định hình bài toán từ thực tế, chấp nhận thử nghiệm sai và sửa sai nhanh (Fail-fast, Learn-faster).
- Chiến lược hiện đại hóa quy trình doanh nghiệp bằng cách kết hợp AI với các công cụ Dashboard tự động để tối ưu ROI.

### Ứng Dụng Vào Công Việc

- **Xây dựng "Second Brain":** Áp dụng tư duy quản lý ngữ cảnh để thiết kế hệ thống quản lý tri thức cá nhân/đội ngũ tốt hơn khi làm việc với AI.
- **Thử nghiệm Amazon Quick:** Tích hợp thử nghiệm các tính năng Quick Flows và Quick Sight để tự động hóa việc làm báo cáo và tối ưu hóa workflow của nhóm.
- **Cấu hình CloudFront:** Áp dụng các bộ quy chuẩn bảo mật và tối ưu hóa chi phí của Amazon CloudFront cho các dự án Web/App đang triển khai.
- **Xử lý LLM Output:** Cải tiến lại hệ thống Prompt Engineering và bổ sung cơ chế kiểm định đầu ra để giảm thiểu lỗi bất định từ LLM.
- **Nghiên cứu cấu trúc Multi-Agent:** Thử nghiệm xây dựng một hệ thống Agent mini phối hợp giải quyết các task tự động hóa quy trình nội bộ.

### Trải nghiệm trong event

Tham gia chuỗi workshop và chia sẻ công nghệ này là một trải nghiệm cực kỳ giá trị, mang lại cái nhìn vừa sâu sắc về mặt kỹ thuật, vừa thực tế thông qua các câu chuyện làm sản phẩm. Một số trải nghiệm nổi bật:

#### Học hỏi từ các diễn giả giàu kinh nghiệm thực chiến
- Các diễn giả đã bóc tách các khái niệm học thuật phức tạp (như cơ chế chọn token của LLM hay kiến trúc Multi-Agent) thành những ví dụ rất trực quan, dễ hiểu.
- Những chia sẻ về lỗi thực tế (Inference optimizations ảnh hưởng đến Determinism) giúp tôi tránh được những lầm tưởng tai hại khi thiết kế hệ thống AI sau này.

#### Truyền cảm hứng từ trải nghiệm Hackathon thực tế
- Phiên chia sẻ của Team VIB về 36 giờ tại LotusHacks mang lại bầu không khí vô cùng sôi động. Qua đó, tôi học được cách giữ cái đầu lạnh để vượt qua các thất bại, áp lực thời gian và cách tinh gọn tính năng để tạo ra một bản Demo mượt mà nhất.

#### Tiếp cận các công cụ AWS hiện đại
- Việc tận mắt chứng kiến demo của hệ sinh thái Amazon Quick và kiến trúc an toàn của CloudFront giúp tôi thấy rõ bức tranh hạ tầng hiện đại: Nhanh hơn, bảo mật hơn và ít phải can thiệp bằng code hơn.

#### Bài học rút ra
- AI chỉ thực sự mạnh khi đặt trong một hệ sinh thái có ngữ cảnh (Context) tốt và được bảo vệ bởi các hành lang an toàn (Guardrails).
- Mọi giải pháp công nghệ đều cần hướng tới đích đến cuối cùng là giải quyết bài toán cốt lõi của bài học/doanh nghiệp và mang lại giá trị ROI thực tế.

#### Một số hình ảnh khi tham gia sự kiện
![Event 1](/workshop-website/images/event1.png)
![Event 11](/workshop-website/images/event11.jpg)
