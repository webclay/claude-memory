# Resend - Email Service Patterns

**Purpose:** Comprehensive guide for sending emails with Resend
**Last Updated:** 2025-01-12
**Official Docs:** https://resend.com/docs

---

## Overview

Resend is a modern email API built for developers, with first-class support for React Email templates and excellent deliverability.

**Key Features:**
- **React Email** - Build emails with React components
- **TypeScript Support** - Full type safety
- **Great Deliverability** - Built-in SPF/DKIM
- **Simple API** - Minimal setup, maximum reliability
- **Email Testing** - Preview emails before sending
- **Webhooks** - Track email events (delivered, opened, clicked, bounced)

---

## Installation

```bash
npm install resend
# or
pnpm add resend

# For React Email templates (optional but recommended)
npm install @react-email/components react-email
```

---

## Setup

### Environment Variables

```env
RESEND_API_KEY=re_...
```

Get your API key from: https://resend.com/api-keys

### Basic Client Setup

```typescript
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

export { resend };
```

---

## Sending Basic Emails

### Simple Text Email

```typescript
import { resend } from '@/lib/resend';

export async function sendWelcomeEmail(to: string, name: string) {
  try {
    const { data, error } = await resend.emails.send({
      from: 'Acme <onboarding@resend.dev>', // Use your verified domain
      to: [to],
      subject: 'Welcome to Acme!',
      text: `Hi ${name},\n\nWelcome to Acme! We're excited to have you on board.`,
    });

    if (error) {
      console.error('Email error:', error);
      return { success: false, error };
    }

    console.log('Email sent:', data?.id);
    return { success: true, id: data?.id };
  } catch (error) {
    console.error('Failed to send email:', error);
    return { success: false, error };
  }
}
```

### HTML Email

```typescript
import { resend } from '@/lib/resend';

export async function sendNewsletterEmail(to: string) {
  const { data, error } = await resend.emails.send({
    from: 'Newsletter <newsletter@yourdomain.com>',
    to: [to],
    subject: 'Weekly Newsletter - January 2025',
    html: `
      <!DOCTYPE html>
      <html>
        <head>
          <style>
            body { font-family: Arial, sans-serif; }
            .container { max-width: 600px; margin: 0 auto; }
            .header { background: #0070f3; color: white; padding: 20px; }
            .content { padding: 20px; }
            .button {
              background: #0070f3;
              color: white;
              padding: 12px 24px;
              text-decoration: none;
              border-radius: 5px;
              display: inline-block;
            }
          </style>
        </head>
        <body>
          <div class="container">
            <div class="header">
              <h1>Weekly Newsletter</h1>
            </div>
            <div class="content">
              <p>Hello!</p>
              <p>Here's what's new this week...</p>
              <a href="https://example.com" class="button">Read More</a>
            </div>
          </div>
        </body>
      </html>
    `,
  });

  return { success: !error, id: data?.id, error };
}
```

---

## React Email Templates

React Email provides beautiful, reusable email components.

### Install React Email

```bash
npm install @react-email/components
```

### Create Email Template

**emails/WelcomeEmail.tsx:**

```typescript
import {
  Body,
  Button,
  Container,
  Head,
  Heading,
  Html,
  Img,
  Link,
  Preview,
  Section,
  Text,
} from '@react-email/components';
import * as React from 'react';

interface WelcomeEmailProps {
  name: string;
  loginUrl: string;
}

export function WelcomeEmail({ name, loginUrl }: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Preview>Welcome to Acme - Get started today!</Preview>
      <Body style={main}>
        <Container style={container}>
          <Img
            src="https://your-domain.com/logo.png"
            width="48"
            height="48"
            alt="Acme Logo"
          />

          <Heading style={h1}>Welcome to Acme, {name}!</Heading>

          <Text style={text}>
            We're thrilled to have you on board. Get started by logging into your account.
          </Text>

          <Section style={buttonContainer}>
            <Button style={button} href={loginUrl}>
              Get Started
            </Button>
          </Section>

          <Text style={text}>
            If you have any questions, feel free to{' '}
            <Link href="mailto:support@acme.com" style={link}>
              contact our support team
            </Link>
            .
          </Text>

          <Text style={footer}>
            Acme Inc., 123 Street, San Francisco, CA 94102
          </Text>
        </Container>
      </Body>
    </Html>
  );
}

// Styles
const main = {
  backgroundColor: '#f6f9fc',
  fontFamily: '-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Ubuntu,sans-serif',
};

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  padding: '20px 0 48px',
  marginBottom: '64px',
};

const h1 = {
  color: '#333',
  fontSize: '24px',
  fontWeight: 'bold',
  margin: '40px 0',
  padding: '0',
};

const text = {
  color: '#333',
  fontSize: '16px',
  lineHeight: '26px',
};

const buttonContainer = {
  padding: '27px 0 27px',
};

const button = {
  backgroundColor: '#0070f3',
  borderRadius: '5px',
  color: '#fff',
  fontSize: '16px',
  fontWeight: 'bold',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'block',
  padding: '12px 20px',
};

const link = {
  color: '#0070f3',
  textDecoration: 'underline',
};

const footer = {
  color: '#8898aa',
  fontSize: '12px',
  lineHeight: '16px',
  marginTop: '32px',
};

export default WelcomeEmail;
```

### Send React Email Template

```typescript
import { resend } from '@/lib/resend';
import { WelcomeEmail } from '@/emails/WelcomeEmail';

export async function sendWelcome(to: string, name: string) {
  const { data, error } = await resend.emails.send({
    from: 'Acme <onboarding@yourdomain.com>',
    to: [to],
    subject: 'Welcome to Acme!',
    react: WelcomeEmail({ name, loginUrl: 'https://app.yourdomain.com/login' }),
  });

  return { success: !error, id: data?.id, error };
}
```

---

## Common Email Templates

### Password Reset Email

**emails/PasswordResetEmail.tsx:**

```typescript
import {
  Body,
  Button,
  Container,
  Head,
  Heading,
  Html,
  Preview,
  Section,
  Text,
} from '@react-email/components';

interface PasswordResetEmailProps {
  name: string;
  resetUrl: string;
}

export function PasswordResetEmail({ name, resetUrl }: PasswordResetEmailProps) {
  return (
    <Html>
      <Head />
      <Preview>Reset your password</Preview>
      <Body style={main}>
        <Container style={container}>
          <Heading>Password Reset Request</Heading>

          <Text>Hi {name},</Text>

          <Text>
            We received a request to reset your password. Click the button below to choose a new password.
          </Text>

          <Section style={buttonContainer}>
            <Button style={button} href={resetUrl}>
              Reset Password
            </Button>
          </Section>

          <Text>
            This link will expire in 1 hour. If you didn't request this, you can safely ignore this email.
          </Text>

          <Text style={footer}>
            For security reasons, this link can only be used once.
          </Text>
        </Container>
      </Body>
    </Html>
  );
}

const main = {
  backgroundColor: '#f6f9fc',
  fontFamily: '-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif',
};

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  padding: '20px 0 48px',
};

const buttonContainer = {
  padding: '27px 0',
};

const button = {
  backgroundColor: '#dc2626',
  borderRadius: '5px',
  color: '#fff',
  fontSize: '16px',
  fontWeight: 'bold',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'block',
  padding: '12px 20px',
};

const footer = {
  color: '#8898aa',
  fontSize: '12px',
  marginTop: '32px',
};

export default PasswordResetEmail;
```

### Email Verification (OTP)

**emails/EmailVerificationEmail.tsx:**

```typescript
import {
  Body,
  Container,
  Head,
  Heading,
  Html,
  Preview,
  Section,
  Text,
} from '@react-email/components';

interface EmailVerificationProps {
  name: string;
  otp: string;
}

export function EmailVerificationEmail({ name, otp }: EmailVerificationProps) {
  return (
    <Html>
      <Head />
      <Preview>Your verification code: {otp}</Preview>
      <Body style={main}>
        <Container style={container}>
          <Heading>Email Verification</Heading>

          <Text>Hi {name},</Text>

          <Text>
            Use the following verification code to complete your registration:
          </Text>

          <Section style={otpContainer}>
            <Text style={otpText}>{otp}</Text>
          </Section>

          <Text>
            This code will expire in 10 minutes.
          </Text>

          <Text style={footer}>
            If you didn't request this code, please ignore this email.
          </Text>
        </Container>
      </Body>
    </Html>
  );
}

const main = {
  backgroundColor: '#f6f9fc',
  fontFamily: '-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif',
};

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  padding: '20px 0 48px',
};

const otpContainer = {
  background: '#f4f4f4',
  borderRadius: '5px',
  margin: '32px 0',
  padding: '24px',
  textAlign: 'center' as const,
};

const otpText = {
  fontSize: '32px',
  fontWeight: 'bold',
  letterSpacing: '8px',
  margin: '0',
  fontFamily: 'monospace',
};

const footer = {
  color: '#8898aa',
  fontSize: '12px',
  marginTop: '32px',
};

export default EmailVerificationEmail;
```

---

## Advanced Features

### Sending to Multiple Recipients

```typescript
import { resend } from '@/lib/resend';

export async function sendBulkEmail(recipients: string[]) {
  const { data, error } = await resend.emails.send({
    from: 'Updates <updates@yourdomain.com>',
    to: recipients, // Array of emails
    subject: 'Important Update',
    html: '<p>This is an important update for all users.</p>',
  });

  return { success: !error, data, error };
}
```

### CC and BCC

```typescript
const { data, error } = await resend.emails.send({
  from: 'Sales <sales@yourdomain.com>',
  to: ['customer@example.com'],
  cc: ['manager@yourdomain.com'],
  bcc: ['archive@yourdomain.com'],
  subject: 'Quote Request',
  html: '<p>Here is your quote...</p>',
});
```

### Attachments

```typescript
import { resend } from '@/lib/resend';
import fs from 'fs';

export async function sendEmailWithAttachment(to: string) {
  const pdfBuffer = fs.readFileSync('invoice.pdf');

  const { data, error } = await resend.emails.send({
    from: 'Billing <billing@yourdomain.com>',
    to: [to],
    subject: 'Your Invoice',
    html: '<p>Please find your invoice attached.</p>',
    attachments: [
      {
        filename: 'invoice.pdf',
        content: pdfBuffer,
      },
    ],
  });

  return { success: !error, data, error };
}
```

### Reply-To Header

```typescript
const { data, error } = await resend.emails.send({
  from: 'Support <noreply@yourdomain.com>',
  replyTo: 'support@yourdomain.com', // Replies go here
  to: ['customer@example.com'],
  subject: 'Support Ticket #1234',
  html: '<p>Your support ticket has been updated.</p>',
});
```

### Custom Headers

```typescript
const { data, error } = await resend.emails.send({
  from: 'App <app@yourdomain.com>',
  to: ['user@example.com'],
  subject: 'Notification',
  html: '<p>You have a new notification.</p>',
  headers: {
    'X-Entity-Ref-ID': '123456789',
    'X-Custom-Header': 'custom-value',
  },
});
```

---

## Email Scheduling (Future)

Currently Resend doesn't support native scheduling, but you can implement it:

```typescript
// Using a job queue (e.g., Bull, BullMQ)
import { Queue } from 'bullmq';

const emailQueue = new Queue('emails');

export async function scheduleEmail(
  to: string,
  subject: string,
  html: string,
  sendAt: Date
) {
  const delay = sendAt.getTime() - Date.now();

  await emailQueue.add(
    'send-email',
    { to, subject, html },
    { delay }
  );
}

// Worker processes the queue
emailQueue.process('send-email', async (job) => {
  const { to, subject, html } = job.data;

  await resend.emails.send({
    from: 'Scheduled <scheduled@yourdomain.com>',
    to: [to],
    subject,
    html,
  });
});
```

---

## Email Testing & Previewing

### Preview React Email Templates Locally

```bash
# Install email dev server
npm install -D email

# Add script to package.json
"scripts": {
  "email:dev": "email dev"
}

# Run preview server
npm run email:dev
```

Visit `http://localhost:3000` to preview your email templates.

### Testing with Resend

Use Resend's test mode:

```typescript
// Send to test inbox (delivered@resend.dev)
const { data, error } = await resend.emails.send({
  from: 'Test <onboarding@resend.dev>',
  to: ['delivered@resend.dev'], // Resend test inbox
  subject: 'Test Email',
  html: '<p>This is a test</p>',
});
```

---

## Webhooks (Event Tracking)

Track email events like delivered, opened, clicked, bounced.

### Setup Webhook Endpoint

```typescript
// app/api/webhooks/resend/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { Webhook } from 'svix';

export async function POST(req: NextRequest) {
  const payload = await req.text();
  const headers = req.headers;

  const wh = new Webhook(process.env.RESEND_WEBHOOK_SECRET!);

  let event;

  try {
    event = wh.verify(payload, {
      'svix-id': headers.get('svix-id')!,
      'svix-timestamp': headers.get('svix-timestamp')!,
      'svix-signature': headers.get('svix-signature')!,
    });
  } catch (error) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  // Handle different event types
  switch (event.type) {
    case 'email.sent':
      console.log('Email sent:', event.data);
      // Update database: email sent
      break;

    case 'email.delivered':
      console.log('Email delivered:', event.data);
      // Update database: email delivered
      break;

    case 'email.opened':
      console.log('Email opened:', event.data);
      // Track analytics: email opened
      break;

    case 'email.clicked':
      console.log('Email clicked:', event.data);
      // Track analytics: link clicked
      break;

    case 'email.bounced':
      console.log('Email bounced:', event.data);
      // Update user: invalid email, mark as bounced
      break;

    case 'email.complained':
      console.log('Email marked as spam:', event.data);
      // Unsubscribe user, investigate
      break;

    default:
      console.log('Unknown event type:', event.type);
  }

  return NextResponse.json({ received: true });
}
```

---

## Error Handling

```typescript
import { resend } from '@/lib/resend';

export async function sendEmailWithErrorHandling(to: string, subject: string, html: string) {
  try {
    const { data, error } = await resend.emails.send({
      from: 'App <app@yourdomain.com>',
      to: [to],
      subject,
      html,
    });

    if (error) {
      // Log specific error
      console.error('Resend error:', error);

      // Handle specific error types
      if (error.message.includes('validation_error')) {
        return { success: false, error: 'Invalid email address' };
      }

      if (error.message.includes('missing_required_field')) {
        return { success: false, error: 'Missing required email field' };
      }

      return { success: false, error: 'Failed to send email' };
    }

    console.log('Email sent successfully:', data?.id);
    return { success: true, emailId: data?.id };

  } catch (error) {
    console.error('Unexpected error:', error);
    return { success: false, error: 'Unexpected error sending email' };
  }
}
```

---

## Rate Limiting & Best Practices

### 1. Batch Emails

```typescript
// Send multiple emails efficiently
export async function sendBatchEmails(emails: Array<{ to: string; subject: string; html: string }>) {
  const promises = emails.map(email =>
    resend.emails.send({
      from: 'Batch <batch@yourdomain.com>',
      to: [email.to],
      subject: email.subject,
      html: email.html,
    })
  );

  const results = await Promise.allSettled(promises);

  const successful = results.filter(r => r.status === 'fulfilled').length;
  const failed = results.filter(r => r.status === 'rejected').length;

  return { successful, failed, total: emails.length };
}
```

### 2. Unsubscribe Links

```typescript
// Include unsubscribe link in all marketing emails
const unsubscribeUrl = `https://yourdomain.com/unsubscribe?token=${userToken}`;

const { data, error } = await resend.emails.send({
  from: 'Newsletter <newsletter@yourdomain.com>',
  to: [email],
  subject: 'Weekly Newsletter',
  html: `
    <p>Your newsletter content...</p>
    <p><a href="${unsubscribeUrl}">Unsubscribe</a></p>
  `,
  headers: {
    'List-Unsubscribe': `<${unsubscribeUrl}>`,
  },
});
```

### 3. Verify Domain

For production, verify your sending domain:

1. Go to Resend Dashboard → Domains
2. Add your domain
3. Add DNS records (SPF, DKIM, DMARC)
4. Verify

```typescript
// Production sending from verified domain
const { data, error } = await resend.emails.send({
  from: 'Acme <noreply@acme.com>', // Your verified domain
  to: [customerEmail],
  subject: 'Order Confirmation',
  react: OrderConfirmationEmail({ order }),
});
```

---

## Server Actions Pattern (Next.js)

```typescript
'use server';

import { resend } from '@/lib/resend';
import { WelcomeEmail } from '@/emails/WelcomeEmail';

export async function sendWelcomeEmailAction(email: string, name: string) {
  try {
    const { data, error } = await resend.emails.send({
      from: 'Acme <onboarding@yourdomain.com>',
      to: [email],
      subject: 'Welcome to Acme!',
      react: WelcomeEmail({ name, loginUrl: 'https://app.acme.com/login' }),
    });

    if (error) {
      return { success: false, error: error.message };
    }

    return { success: true, emailId: data?.id };
  } catch (error) {
    console.error('Failed to send welcome email:', error);
    return { success: false, error: 'Failed to send email' };
  }
}
```

**Client component:**

```typescript
'use client';

import { sendWelcomeEmailAction } from './actions';

export function SignUpForm() {
  const handleSubmit = async (formData: FormData) => {
    const email = formData.get('email') as string;
    const name = formData.get('name') as string;

    const result = await sendWelcomeEmailAction(email, name);

    if (result.success) {
      alert('Welcome email sent!');
    } else {
      alert('Failed to send email: ' + result.error);
    }
  };

  return (
    <form action={handleSubmit}>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" required />
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

---

## Common Use Cases

### 1. Transactional Emails

```typescript
// Order confirmation
await resend.emails.send({
  from: 'Orders <orders@shop.com>',
  to: [customerEmail],
  subject: `Order #${orderId} Confirmed`,
  react: OrderConfirmationEmail({ order }),
});

// Payment receipt
await resend.emails.send({
  from: 'Billing <billing@shop.com>',
  to: [customerEmail],
  subject: 'Payment Receipt',
  react: PaymentReceiptEmail({ payment }),
});
```

### 2. Notification Emails

```typescript
// New comment notification
await resend.emails.send({
  from: 'Notifications <notifications@app.com>',
  to: [userEmail],
  subject: 'New comment on your post',
  react: NewCommentEmail({ comment }),
});
```

### 3. Marketing Emails

```typescript
// Product announcement
await resend.emails.send({
  from: 'Marketing <marketing@company.com>',
  to: subscriberList,
  subject: 'Introducing Our New Product',
  react: ProductAnnouncementEmail({ product }),
  headers: {
    'List-Unsubscribe': '<https://company.com/unsubscribe>',
  },
});
```

---

## Resources

- **Official Docs:** https://resend.com/docs
- **React Email Docs:** https://react.email
- **React Email Components:** https://react.email/docs/components
- **Email Examples:** https://react.email/examples
- **Dashboard:** https://resend.com/emails

---

*This guide covers Resend email service with React Email integration. For advanced features or specific use cases, consult the official documentation.*
