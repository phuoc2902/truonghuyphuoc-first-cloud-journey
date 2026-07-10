---
title: "5.5. Integrate AI Chatbot (Amazon Bedrock)"
date: 2026-07-06
weight: 5
---

#### 1. Business Case & Architecture Solution

**The Problem:** The core Unique Selling Point of NovaTech compared to traditional e-commerce sites is its **Smart Consultation Capability**. Customers buying tech devices are often confused by thousands of technical specs (Core i5 vs i7, 8GB vs 16GB RAM). Without a 24/7 sales representative, customers are highly likely to drop off.

**The Solution:** Integrate an **AI Chatbot** to act as a professional 24/7 sales representative.
- Use Large Language Models (LLM) **Claude 3 Haiku/Sonnet** via **Amazon Bedrock**.
- Apply **RAG (Retrieval-Augmented Generation)** techniques: The chatbot does not "hallucinate" specifications; it directly reads knowledge data files (specs of available laptops) from S3. This ensures the AI provides 100% accurate advice on prices and specs available in the store, boosting the Conversion Rate.

Below is the **Implementation Proof** detailing Amazon Bedrock configuration and the Backend AI connection code:

#### 2. Request Model Access on Amazon Bedrock
By default, generative AI models on Bedrock are disabled due to licensing and billing precautions. You must enable them manually:
1. Open the AWS Console, search for **Amazon Bedrock**.
2. From the left navigation menu, scroll down and click **Model access**.
3. Click **Manage model access** on the top right.
4. Locate **Anthropic Claude 3** models (e.g., Claude 3 Haiku) and check their boxes.
5. Click **Request model access** and wait for approval (takes 1 - 5 minutes). The status should change to **Access granted**:

![Request Bedrock Model Access](/images/5-Workshop/bedrock_model_access.png)

---

#### 3. Install Amazon Bedrock SDK in Spring Boot
Add the Bedrock Runtime client dependency in your backend `pom.xml` to communicate with the AI model:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>bedrockruntime</artifactId>
    <version>2.25.15</version>
</dependency>
```

---

#### 4. Source Code to Generate AI Responses
Below is the Spring Boot Service code calling the Claude 3 Haiku model from Amazon Bedrock:

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
        // Configure Anthropic Claude 3 Haiku Model ID
        String modelId = "anthropic.claude-3-haiku-20240307-v1:0";

        // Setup System Prompt embedding RAG Context from S3 to instruct the AI
        String systemPrompt = "You are a professional laptop consultant for NovaTech. " 
            + "Please answer questions based on the following product information:\n" + contextData;

        // Prepare JSON payload sent to Claude 3
        JSONObject requestBody = new JSONObject();
        requestBody.put("anthropic_version", "bedrock-2023-05-31");
        requestBody.put("max_tokens", 1000);
        requestBody.put("system", systemPrompt);
        
        JSONObject message = new JSONObject();
        message.put("role", "user");
        message.put("content", userPrompt);
        
        requestBody.put("messages", new Object[]{message});

        // Trigger Model Invocation to AWS Bedrock
        InvokeModelRequest request = InvokeModelRequest.builder()
                .modelId(modelId)
                .contentType("application/json")
                .accept("application/json")
                .body(SdkBytes.fromUtf8String(requestBody.toString()))
                .build();

        InvokeModelResponse response = bedrockRuntimeClient.invokeModel(request);
        
        // Parse the text reply from JSON response
        String responseBody = response.body().asString(StandardCharsets.UTF_8);
        JSONObject jsonResponse = new JSONObject(responseBody);
        
        return jsonResponse.getJSONArray("content")
                .getJSONObject(0)
                .getString("text");
    }
}
```

---

#### 5. Test Chatbot on Web Interface
After completing API integration, open the NovaTech website and click on the Chatbot icon in the bottom right corner. Try asking a question like: *"Suggest a high-performance programming laptop under $1000"*, and the AI Chatbot will provide accurate details based on the provided data context.
