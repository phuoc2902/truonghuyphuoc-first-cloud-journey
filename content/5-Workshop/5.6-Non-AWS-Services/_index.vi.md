---
title: "5.6. Cấu hình máy chủ ứng dụng (DigitalOcean) & Cổng thanh toán PayOS"
date: 2026-07-06
weight: 6
---

#### 1. Bài toán nghiệp vụ & Giải pháp kiến trúc

**Bài toán 1 (Máy chủ Ứng dụng):** Mặc dù AWS là nền tảng Cloud mạnh mẽ, nhưng việc chạy liên tục một máy chủ ứng dụng (App Server) trên Amazon EC2 có thể phát sinh chi phí cao và khó kiểm soát cho một dự án MVP (Minimum Viable Product).
**Giải pháp 1:** Sử dụng **DigitalOcean Droplet (Ubuntu)**. Đây là giải pháp VPS với mức giá cố định (Fixed-cost), dễ quản trị. Máy chủ này đóng vai trò xử lý toàn bộ logic nghiệp vụ (Spring Boot) và kết nối an toàn với các dịch vụ AWS. 

**Bài toán 2 (Thanh toán trực tuyến):** Một trải nghiệm thương mại điện tử sẽ bị đứt gãy nếu không có bước thanh toán. Ở Việt Nam, thanh toán qua mã QR (VietQR) là phổ biến nhất.
**Giải pháp 2:** Tích hợp **PayOS**. Nền tảng này cung cấp API tạo mã QR chuyển khoản và hệ thống Webhook tự động thông báo khi khách hàng chuyển tiền thành công, giúp hệ thống tự động gạch nợ và đổi trạng thái Đơn hàng thành "Đã thanh toán".

Dưới đây là phần **Minh chứng triển khai** thực tế trên máy chủ DigitalOcean và PayOS:

---

#### 2. Triển khai Backend Spring Boot trên DigitalOcean (Droplet Ubuntu)

1. **Khởi tạo và kết nối SSH**:
   - Thuê máy ảo (Droplet) chạy hệ điều hành **Ubuntu LTS** trên DigitalOcean.
   - Trỏ một địa chỉ IP tĩnh (Elastic/Reserved IP) và kết nối SSH từ Terminal:
     ```bash
     ssh root@your_droplet_ip
     ```

2. **Cài đặt Java Runtime Environment**:
     ```bash
     sudo apt update
     sudo apt install openjdk-17-jdk -y
     ```

3. **Chạy ứng dụng Spring Boot Backend ngầm với Systemd**:
   Bạn upload file `nova-tech-be.jar` lên Server. Thay vì chạy lệnh `java -jar` trực tiếp (dễ bị tắt khi đóng Terminal), ta tạo một service để hệ điều hành tự động chạy lại nếu server khởi động lại.
   
   Tạo file `/etc/systemd/system/novatech.service`:
   ```ini
   [Unit]
   Description=NovaTech Spring Boot Application
   After=network.target

   [Service]
   User=root
   ExecStart=/usr/bin/java -jar /var/www/novatech/nova-tech-be.jar
   SuccessExitStatus=143
   Restart=always
   RestartSec=10
   EnvironmentFile=/var/www/novatech/.env

   [Install]
   WantedBy=multi-user.target
   ```
   Lưu file và khởi động dịch vụ:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start novatech
   sudo systemctl enable novatech
   ```

---

#### 3. Cấu hình Nginx Reverse Proxy & SSL (Let's Encrypt)
Máy chủ Spring Boot chạy ngầm ở cổng `8080`. Ta sử dụng **Nginx** làm máy chủ đảo ngược (Reverse Proxy) để chuyển hướng các yêu cầu cổng `80`/`443` công cộng về ứng dụng và gán chứng chỉ SSL bảo mật HTTPS.

1. **Cài đặt Nginx**:
   ```bash
   sudo apt install nginx -y
   ```
2. **Cấu hình Reverse Proxy**:
   - Chỉnh sửa cấu hình block server trong `/etc/nginx/sites-available/default`:
     ```nginx
     server {
         listen 80;
         server_name api.novatech-laptop.id.vn;

         location / {
             proxy_pass http://127.0.0.1:8080;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
         }
     }
     ```
   - Khởi động lại Nginx: `sudo systemctl restart nginx`

3. **Cấu hình chứng chỉ bảo mật SSL miễn phí bằng Certbot (Let's Encrypt)**:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   sudo certbot --nginx -d api.novatech-laptop.id.vn
   ```
   *Certbot sẽ tự động đăng ký SSL và tự cấu hình chuyển hướng HTTPS vào file cấu hình Nginx.*

---

#### 4. Tích hợp cổng thanh toán PayOS
Để người dùng thanh toán tiền mua laptop tiện lợi qua QR Code, dự án tích hợp thư viện **PayOS Java SDK** để giao tiếp với hệ thống PayOS.

1. **Thêm dependency vào file pom.xml**:
   ```xml
   <dependency>
       <groupId>vn.payos</groupId>
       <artifactId>payos-java</artifactId>
       <version>1.0.1</version>
   </dependency>
   ```

2. **Mã nguồn xử lý giao dịch thanh toán (PayOsPaymentStrategy.java)**:
   Dưới đây là mã nguồn chiến lược thanh toán [PayOsPaymentStrategy.java](file:///Users/tranthikieuoanh/Documents/FCJ-Workshop/nova-tech/nova-tech-be/src/main/java/com/novatech/nova_tech_be/modules/payment/service/impl/strategy/PayOsPaymentStrategy.java) gọi API tạo liên kết thanh toán:

   ```java
   package com.novatech.nova_tech_be.modules.payment.service.impl.strategy;

   import com.novatech.nova_tech_be.modules.payment.dto.response.PaymentResponse;
   import com.novatech.nova_tech_be.modules.payment.entity.Payment;
   import com.novatech.nova_tech_be.modules.payment.entity.PaymentMethod;
   import com.novatech.nova_tech_be.modules.payment.repository.PaymentRepository;
   import com.novatech.nova_tech_be.modules.payment.service.interfaces.PaymentStrategy;
   import lombok.RequiredArgsConstructor;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Component;
   import vn.payos.PayOS;
   import vn.payos.model.v2.paymentRequests.CreatePaymentLinkRequest;
   import vn.payos.model.v2.paymentRequests.CreatePaymentLinkResponse;

   @Slf4j
   @Component
   @RequiredArgsConstructor
   public class PayOsPaymentStrategy implements PaymentStrategy {

       private final PayOS payOS;
       private final PaymentRepository paymentRepository;

       @Value("${PAYOS_RETURN_URL}")
       private String returnUrl;

       @Value("${PAYOS_CANCEL_URL}")
       private String cancelUrl;

       @Override
       public PaymentResponse processPayment(Payment payment) {
           log.info("Processing PayOS payment for order: {}", payment.getOrderCode());
           try {
               Long payosOrderCode = generateUniquePayosOrderCode();
               long amount = payment.getAmount().longValue(); // Đơn vị VND

               String dynamicReturnUrl = returnUrl + "?orderCode=" + payment.getOrderCode();
               String dynamicCancelUrl = cancelUrl + "?orderCode=" + payment.getOrderCode();

               // Tạo cấu trúc yêu cầu thanh toán gửi lên PayOS
               CreatePaymentLinkRequest payosRequest = CreatePaymentLinkRequest.builder()
                       .orderCode(payosOrderCode)
                       .amount(amount)
                       .description(payment.getDescription())
                       .returnUrl(dynamicReturnUrl)
                       .cancelUrl(dynamicCancelUrl)
                       .build();

               // Gọi SDK PayOS sinh link thanh toán QR Code
               CreatePaymentLinkResponse payosResponse = payOS.paymentRequests().create(payosRequest);

               payment.setPayosOrderCode(payosOrderCode);
               payment.setCheckoutUrl(payosResponse.getCheckoutUrl());
               payment.setQrCode(payosResponse.getQrCode());
               paymentRepository.save(payment);

               return PaymentResponse.builder()
                       .id(payment.getId())
                       .paymentCode(payment.getPaymentCode())
                       .checkoutUrl(payment.getCheckoutUrl())
                       .qrCode(payment.getQrCode())
                       .build();
           } catch (Exception e) {
               log.error("Lỗi khi tạo PayOS link: {}", e.getMessage());
               throw new RuntimeException("Lỗi cổng thanh toán PayOS: " + e.getMessage(), e);
           }
       }

       @Override
       public PaymentMethod getMethod() {
           return PaymentMethod.PAYOS;
       }
   }
   ```
