---
title: "5.6. Configure Non-AWS Application Server & PayOS Payment Gateway"
date: 2026-07-06
weight: 6
---

#### 1. Business Case & Architecture Solution

**Business Case 1 (Application Server):** While AWS is a powerful cloud platform, running an always-on application server on Amazon EC2 can incur high and unpredictable costs for an MVP (Minimum Viable Product).
**Solution 1:** Use **DigitalOcean Droplet (Ubuntu)**. This is a VPS solution with fixed-cost pricing that's easy to manage. This server handles all the business logic (Spring Boot) and securely connects to AWS services.

**Business Case 2 (Online Payment):** An e-commerce experience is incomplete without a payment gateway. In Vietnam, bank transfer via QR Code (VietQR) is the most popular method.
**Solution 2:** Integrate **PayOS**. This platform provides an API to generate dynamic bank transfer QR codes and a Webhook system to automatically notify the backend when a customer successfully transfers money, enabling the system to automatically mark the order as "Paid".

Below is the **Implementation Proof** for deploying on the DigitalOcean server and integrating PayOS:

---

#### 2. Deploy Spring Boot Backend on DigitalOcean (Ubuntu Droplet)

1. **SSH Connection**:
   - Rent a virtual machine (Droplet) running **Ubuntu LTS** on DigitalOcean.
   - Allocate a static IP (Reserved IP) and SSH connect from your Terminal:
     ```bash
     ssh root@your_droplet_ip
     ```

2. **Install Java Runtime Environment**:
     ```bash
     sudo apt update
     sudo apt install openjdk-17-jdk -y
     ```

3. **Configure Systemd Service** to run and monitor the Java JAR application in the background:
   - Create a systemd service file: `/etc/systemd/system/novatech-backend.service`
   - Service configuration details:
     ```ini
     [Unit]
     Description=NovaTech Backend API Service
     After=network.target

     [Service]
     User=root
     WorkingDirectory=/var/www/nova-tech-be
     ExecStart=/usr/bin/java -jar nova-tech-be-0.0.1-SNAPSHOT.jar \
       --spring.profiles.active=prod
     SuccessExitStatus=143
     Restart=always
     RestartSec=10

     [Install]
     WantedBy=multi-user.target
     ```
   - Reload and start the background service:
     ```bash
     sudo systemctl daemon-reload
     sudo systemctl enable novatech-backend.service
     sudo systemctl start novatech-backend.service
     ```

---

#### 3. Configure Nginx Reverse Proxy & SSL (Let's Encrypt)
The Spring Boot backend app runs internally on port `8080`. We configure **Nginx** as a Reverse Proxy to route public traffic from port `80`/`443` to `8080` and install HTTPS security.

1. **Install Nginx**:
   ```bash
   sudo apt install nginx -y
   ```
2. **Setup Reverse Proxy Configuration**:
   - Edit the default server block in `/etc/nginx/sites-available/default`:
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
   - Restart Nginx: `sudo systemctl restart nginx`

3. **Install Let's Encrypt SSL via Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   sudo certbot --nginx -d api.novatech-laptop.id.vn
   ```
   *Certbot will automatically request the SSL certificates and configure Nginx to redirect HTTP traffic to HTTPS.*

---

#### 4. Integrate PayOS Payment Gateway
To allow clients to pay for their laptop orders via QR Codes, the backend integrates the **PayOS Java SDK**.

1. **Add Dependency in pom.xml**:
   ```xml
   <dependency>
       <groupId>vn.payos</groupId>
       <artifactId>payos-java</artifactId>
       <version>1.0.1</version>
   </dependency>
   ```

2. **Spring Boot Payment Process (PayOsPaymentStrategy.java)**:
   Below is the source code for [PayOsPaymentStrategy.java](file:///Users/tranthikieuoanh/Documents/FCJ-Workshop/nova-tech/nova-tech-be/src/main/java/com/novatech/nova_tech_be/modules/payment/service/impl/strategy/PayOsPaymentStrategy.java) handling payment link generation:

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
               long amount = payment.getAmount().longValue(); // VND Amount

               String dynamicReturnUrl = returnUrl + "?orderCode=" + payment.getOrderCode();
               String dynamicCancelUrl = cancelUrl + "?orderCode=" + payment.getOrderCode();

               // Create PayOS payment request object
               CreatePaymentLinkRequest payosRequest = CreatePaymentLinkRequest.builder()
                       .orderCode(payosOrderCode)
                       .amount(amount)
                       .description(payment.getDescription())
                       .returnUrl(dynamicReturnUrl)
                       .cancelUrl(dynamicCancelUrl)
                       .build();

               // Call SDK to generate checkout URL and QR code
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
               log.error("Failed to create PayOS link: {}", e.getMessage());
               throw new RuntimeException("PayOS Gateway Error: " + e.getMessage(), e);
           }
       }

       @Override
       public PaymentMethod getMethod() {
           return PaymentMethod.PAYOS;
       }
   }
   ```
