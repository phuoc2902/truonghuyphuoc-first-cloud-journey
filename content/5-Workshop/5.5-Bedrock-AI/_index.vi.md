---
title: "5.5. Tích hợp Trợ lý ảo AI Chatbot (Amazon Bedrock)"
date: 2026-07-06
weight: 5
---

#### 1. Bài toán nghiệp vụ & Giải pháp kiến trúc

**Bài toán:** Điểm khác biệt cốt lõi (Unique Selling Point) của dự án NovaTech so với các trang web bán hàng truyền thống là **Khả năng tư vấn thông minh**. Khách hàng mua thiết bị công nghệ thường bối rối trước hàng ngàn thông số (Core i5 hay i7, RAM 8GB hay 16GB). Nếu không có nhân viên tư vấn trực tiếp 24/7, khách hàng rất dễ bỏ đi (drop-off).

**Giải pháp:** Tích hợp **AI Chatbot** đóng vai trò như một nhân viên Sale chuyên nghiệp trực 24/7.
- Sử dụng mô hình ngôn ngữ lớn (LLM) **Claude 3 Haiku/Sonnet** qua dịch vụ **Amazon Bedrock**.
- Áp dụng kỹ thuật **RAG (Retrieval-Augmented Generation)**: Chatbot không tự "bịa" ra thông số, mà nó đọc trực tiếp các tệp dữ liệu tri thức (specs của các laptop đang bán) từ S3. Điều này đảm bảo AI tư vấn chính xác 100% về giá cả, thông số máy có trong cửa hàng, giúp tăng tỷ lệ chốt đơn (Conversion Rate).

Dưới đây là phần **Minh chứng triển khai** cấu hình Amazon Bedrock và mã nguồn gọi AI từ Backend:

#### 2. Yêu cầu quyền truy cập Mô hình trên Amazon Bedrock
Mặc định, các mô hình Generative AI trên Bedrock bị tắt vì lý do chi phí và pháp lý. Bạn cần kích hoạt thủ công:
1. Đăng nhập vào AWS Console, tìm kiếm dịch vụ **Amazon Bedrock**.
2. Ở thanh menu bên trái, cuộn xuống chọn mục **Model access**.
3. Nhấn **Manage model access** ở góc phải.
4. Tìm mô hình **Anthropic Claude 3** (ví dụ: Claude 3 Haiku) và tích chọn.
5. Nhấn **Request model access** và đợi hệ thống phê duyệt (thường mất từ 1 - 5 phút). Trạng thái sẽ chuyển thành **Access granted**:

![Yêu cầu quyền truy cập Claude 3 trên Bedrock](/images/5-Workshop/bedrock_model_access.png)

---

#### 3. Cài đặt thư viện Amazon Bedrock SDK trong Spring Boot
Thêm dependency của Bedrock Runtime client vào file `pom.xml` của dự án Backend để giao tiếp với AI model:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>bedrockruntime</artifactId>
    <version>2.25.15</version>
</dependency>
```

---

#### 4. Mã nguồn kết nối Amazon Bedrock sinh câu trả lời
Dưới đây là mã nguồn Service trong Spring Boot gọi mô hình Claude 3 Haiku từ Amazon Bedrock:

```java
package com.novatech.nova_tech_be.modules.chatbot.service.impl;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import software.amazon.awssdk.core.SdkBytes;
import software.amazon.awssdk.services.bedrockruntime.BedrockRuntimeClient;
import software.amazon.awssdk.services.bedrockruntime.model.InvokeModelRequest;
import software.amazon.awssdk.services.bedrockruntime.model.InvokeModelResponse;
import org.json.JSONObject;

import java.nio.charset.StandardCharsets;

@Service
public class BedrockChatServiceImpl {

    private final BedrockRuntimeClient bedrockRuntimeClient;

    @Value("${AWS_REGION:ap-southeast-1}")
    private String region;

    public BedrockChatServiceImpl(BedrockRuntimeClient bedrockRuntimeClient) {
        this.bedrockRuntimeClient = bedrockRuntimeClient;
    }

    public String generateResponse(String userPrompt, String contextData) {
        // Cấu hình ID mô hình Anthropic Claude 3 Haiku
        String modelId = "anthropic.claude-3-haiku-20240307-v1:0";

        // Thiết lập Prompt chứa dữ liệu tri thức (RAG Context) từ S3 gửi tới AI
        String systemPrompt = "Bạn là trợ lý ảo tư vấn laptop chuyên nghiệp của NovaTech. " 
            + "Hãy trả lời câu hỏi dựa trên thông tin sản phẩm sau đây:\n" + contextData;

        // Chuẩn bị payload JSON gửi cho Claude 3
        JSONObject requestBody = new JSONObject();
        requestBody.put("anthropic_version", "bedrock-2023-05-31");
        requestBody.put("max_tokens", 1000);
        requestBody.put("system", systemPrompt);
        
        JSONObject message = new JSONObject();
        message.put("role", "user");
        message.put("content", userPrompt);
        
        requestBody.put("messages", new Object[]{message});

        // Tạo yêu cầu Invoke Model gửi lên AWS Bedrock
        InvokeModelRequest request = InvokeModelRequest.builder()
                .modelId(modelId)
                .contentType("application/json")
                .accept("application/json")
                .body(SdkBytes.fromUtf8String(requestBody.toString()))
                .build();

        InvokeModelResponse response = bedrockRuntimeClient.invokeModel(request);
        
        // Đọc kết quả phản hồi từ JSON trả về
        String responseBody = response.body().asString(StandardCharsets.UTF_8);
        JSONObject jsonResponse = new JSONObject(responseBody);
        
        return jsonResponse.getJSONArray("content")
                .getJSONObject(0)
                .getString("text");
    }
}
```

---

#### 5. Kiểm thử Chatbot trên giao diện ứng dụng Web
Sau khi hoàn thành tích hợp API, mở trang web NovaTech và nhấn vào biểu tượng Chatbot ở góc phải màn hình. Thử nhập câu hỏi như: *"Tư vấn cho tôi dòng máy laptop cấu hình cao để lập trình dưới 20 triệu"*, AI Chatbot sẽ trả lời thông tin chính xác dựa trên cơ sở dữ liệu đã chuẩn bị.
