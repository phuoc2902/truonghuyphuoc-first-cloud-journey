---
title: "5.4. Xác thực người dùng với Amazon Cognito"
date: 2026-07-06
weight: 4
---

#### 1. Bài toán nghiệp vụ & Giải pháp kiến trúc

**Bài toán:** Quản lý thông tin định danh người dùng (Identity) là một trong những thành phần rủi ro nhất của một hệ thống ứng dụng. Nếu đội ngũ phát triển tự xây dựng tính năng đăng nhập/đăng ký, lưu trữ mật khẩu trực tiếp trong RDS, hệ thống sẽ đối mặt với các rủi ro bảo mật nghiêm trọng (như rò rỉ dữ liệu (Data Leak), tấn công Brute-force). Ngoài ra, việc tự code các tính năng như Quên mật khẩu, Xác thực qua Email (OTP), hay Đăng nhập bằng Google/Facebook mất rất nhiều thời gian.

**Giải pháp:** Sử dụng **Amazon Cognito User Pool** như một Identity Provider (Nhà cung cấp định danh) độc lập.
- **Cognito User Pool:** Hoạt động như một thư mục người dùng an toàn. Khi khách hàng đăng nhập thành công trên giao diện Frontend (Next.js), Cognito trả về một mã JWT (JSON Web Token).
- Frontend dùng token này gửi cho Backend (Spring Boot) để xác thực người dùng thay vì truyền mật khẩu trực tiếp. Mọi nghiệp vụ đăng nhập, quản lý mật khẩu, OTP đều do AWS gánh vác, đảm bảo tuân thủ các tiêu chuẩn bảo mật quốc tế.

Dưới đây là phần **Minh chứng triển khai** cấu hình Cognito và mã nguồn xác thực từ Frontend:

#### 2. Khởi tạo Amazon Cognito User Pool
1. Tại AWS Console, tìm kiếm dịch vụ **Cognito** và nhấn **Create user pool**.
2. Tại mục **Configure sign-in experience**:
   - Chọn phương thức đăng nhập bằng **Email** hoặc **Username**.
3. Tại mục **Configure security requirements**:
   - Chọn chính sách mật khẩu (Password policy).
   - Chọn **No MFA** (Nếu chỉ thử nghiệm) hoặc **Required MFA** để tăng độ bảo mật bằng mã xác nhận gửi về điện thoại/email.
4. Tại mục **Configure sign-up experience**:
   - Bật hoặc tắt tính năng tự đăng ký (Self-service sign-up).
5. Tại mục **Integrate your app**:
   - Điền **User Pool name**: `novatech-user-pool`.
   - Tại mục **Initial app client**: Thiết lập app client mới tên là `novatech-nextjs-client`.
   - Đảm bảo **TẮT** tùy chọn **Generate client secret** (Vì Next.js Client Side component không thể bảo mật Client Secret ở phía client).
6. Nhấn **Create user pool** và đợi vài giây. Sau khi hoàn thành, sao chép các thông tin **User Pool ID** và **Client ID**:

![Tổng quan Cognito User Pool](/images/5-Workshop/cognito_overview.png)

---

#### 3. Tích hợp AWS Amplify Auth ở phía Frontend (Next.js)
Để giao diện Next.js của NovaTech gọi trực tiếp tới Cognito kiểm tra đăng nhập, cài đặt thư viện AWS Amplify:

```bash
npm install aws-amplify
```

Cấu hình khởi tạo Amplify Auth trong dự án Frontend Next.js:

```javascript
import { Amplify } from 'aws-amplify';

Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: process.env.NEXT_PUBLIC_COGNITO_USER_POOL_ID,
      userPoolClientId: process.env.NEXT_PUBLIC_COGNITO_CLIENT_ID,
      signUpVerificationMethod: 'code' // Gửi mã OTP xác nhận về Email
    }
  }
});
```

---

#### 4. Gọi API xác thực từ mã nguồn Frontend
Đoạn mã xử lý gửi mã xác nhận OTP và đăng ký tài khoản mới lên Cognito từ Next.js:

```typescript
import { signUp, confirmSignUp } from 'aws-amplify/auth';

// 1. Đăng ký tài khoản mới lên Cognito User Pool
async function handleSignUp(email, password) {
  try {
    const { isSignUpComplete, userId, nextStep } = await signUp({
      username: email,
      password: password,
      options: {
        userAttributes: {
          email: email
        }
      }
    });
    console.log("Đăng ký thành công, bước tiếp theo:", nextStep.signUpStep);
  } catch (error) {
    console.error("Lỗi đăng ký Cognito:", error);
  }
}

// 2. Xác thực tài khoản bằng mã OTP gửi qua Email
async function handleConfirmSignUp(email, verificationCode) {
  try {
    const { isSignUpComplete, nextStep } = await confirmSignUp({
      username: email,
      confirmationCode: verificationCode
    });
    if (isSignUpComplete) {
      console.log("Tài khoản đã được kích hoạt thành công trên Cognito!");
    }
  } catch (error) {
    console.error("Lỗi xác minh mã OTP:", error);
  }
}
```

Khi chạy thử nghiệm giao diện đăng ký, bạn sẽ nhận được thông báo mã OTP gửi thành công và kích hoạt tài khoản Cognito:

![Xác thực OTP Cognito thành công](/images/5-Workshop/cognito_success_code.png)
