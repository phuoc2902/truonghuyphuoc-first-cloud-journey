---
title: "5.4. Configure Amazon Cognito Authentication"
date: 2026-07-06
weight: 4
---

#### 1. Business Case & Architecture Solution

**The Problem:** Managing user identities is one of the highest-risk components of any application. If the development team builds custom login/signup features and stores passwords directly in RDS, the system faces severe security risks (such as Data Leaks and Brute-force attacks). Additionally, coding features like "Forgot Password," Email Verification (OTP), and Social Logins from scratch is highly time-consuming.

**The Solution:** Use **Amazon Cognito User Pool** as an independent Identity Provider.
- **Cognito User Pool:** Acts as a secure user directory. When a customer successfully logs in via the Frontend (Next.js), Cognito returns a JWT (JSON Web Token).
- The Frontend uses this token to authenticate with the Backend (Spring Boot) instead of transmitting raw passwords. AWS handles all login flows, password management, and OTP verification, ensuring compliance with international security standards.

Below is the **Implementation Proof** detailing Cognito configuration and Frontend authentication code:

#### 2. Create Amazon Cognito User Pool
1. In the AWS Console, search for **Cognito** and click **Create user pool**.
2. Under **Configure sign-in experience**:
   - Choose login options such as **Email** or **Username**.
3. Under **Configure security requirements**:
   - Set the password policy.
   - Choose **No MFA** (for development) or **Required MFA** to secure logins via OTP codes sent to emails or phone numbers.
4. Under **Configure sign-up experience**:
   - Enable or disable self-service sign-up.
5. Under **Integrate your app**:
   - Enter a **User pool name**: `novatech-user-pool`.
   - Under **Initial app client**: Set up a new app client named `novatech-nextjs-client`.
   - Ensure the **Generate client secret** option is **OFF** (Next.js Client Side components cannot securely store secrets).
6. Click **Create user pool**. Once completed, copy the **User Pool ID** and **Client ID**:

![Cognito User Pool Overview](/images/5-Workshop/cognito_overview.png)

---

#### 3. Integrate AWS Amplify Auth in Frontend (Next.js)
To allow the Next.js frontend to verify logins directly with Cognito, install the AWS Amplify library:

```bash
npm install aws-amplify
```

Initialize Amplify Auth in your Next.js project:

```javascript
import { Amplify } from 'aws-amplify';

Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: process.env.NEXT_PUBLIC_COGNITO_USER_POOL_ID,
      userPoolClientId: process.env.NEXT_PUBLIC_COGNITO_CLIENT_ID,
      signUpVerificationMethod: 'code' // Send OTP code to Email
    }
  }
});
```

---

#### 4. Call Auth API from Frontend Source Code
The following snippet handles OTP verification and registering new accounts to Cognito from Next.js:

```typescript
import { signUp, confirmSignUp } from 'aws-amplify/auth';

// 1. Register a new account to the Cognito User Pool
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
    console.log("Signup successful, next step:", nextStep.signUpStep);
  } catch (error) {
    console.error("Cognito signup error:", error);
  }
}

// 2. Verify account with OTP sent via Email
async function handleConfirmSignUp(email, verificationCode) {
  try {
    const { isSignUpComplete, nextStep } = await confirmSignUp({
      username: email,
      confirmationCode: verificationCode
    });
    if (isSignUpComplete) {
      console.log("Account successfully activated on Cognito!");
    }
  } catch (error) {
    console.error("OTP verification error:", error);
  }
}
```

When testing the signup UI, you will receive a successful OTP notification activating the Cognito account:

![Cognito OTP Verification Success](/images/5-Workshop/cognito_success_code.png)
