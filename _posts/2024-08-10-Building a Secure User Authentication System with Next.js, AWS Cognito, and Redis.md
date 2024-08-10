---
title: Building a Secure User Authentication System with Next.js, AWS Cognito, and Redis
date: 2024-08-10 01:00:00 -0400
categories: "frontend"
header:
  teaser: "/assets/images/user-auth.svg"
tags:
  - React
  - nextjs
  - AWS
toc: true
toc_sticky: true
---

![user-auth](/assets/images/user-auth.svg)

In this article, I’ll walk you through a project where I developed a secure and efficient user authentication system using Next.js, AWS Cognito, and Redis. This setup not only handles user login and logout but also manages user sessions with a robust middleware system to protect sensitive routes.

# Overview of the Tech Stack
- Next.js: A powerful React framework that enables server-side rendering and static site generation.
- AWS Cognito: A fully managed service that provides user sign-up, sign-in, and access control.
- Redis: An in-memory data store used here for session management.

# Project Setup
The project is structured as a Next.js application, bootstrapped with create-next-app. Below are the steps to get started:

## 1. Environment Variables
You’ll need to set up a .env.local file at the root of your project directory:

```
AWS_REGION=<< value >>
COGNITO_USERPOOL_ID=<< value >>
COGNITO_CLIENT_ID=<< value >>
NEXT_PUBLIC_GOOGLE_CLIENT_ID=<< value >>
REDIS_HOST=<< value >>
REDIS_PASSWORD=<< value >>
REDIS_PORT=<< value >>
```

Replace the placeholders with actual values from your AWS Cognito setup and Redis instance.

## 2. Running the Development Server
Once the environment variables are in place, start the development server with:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Visit http://localhost:3000 in your browser to see the app in action.

# Session Management with Redis
In this project, Redis plays a crucial role in managing user sessions. Here's how it works:

- Session Storage: When a user logs in, their session data, including tokens, is stored in Redis. This allows for fast retrieval and checks across various parts of the application.
- Middleware Protection: A custom middleware is implemented to guard protected routes. It checks for a valid session in Redis and ensures that the access token has not expired.
- Token Refresh: If the access token is expired, the middleware will attempt to refresh it using the refresh token stored in Redis. If this process fails, the user is logged out and redirected to the sign-in page.

# Handling Expired Sessions
Sessions are automatically managed based on their expiration. If a session expires, the middleware will:

- Delete the session data from Redis.
- Redirect the user to the login page.

This ensures that users cannot access protected resources with an expired session, thereby enhancing security.

# Conclusion
This project demonstrates a robust implementation of user authentication and session management using Next.js, AWS Cognito, and Redis. The combination of these technologies offers a scalable and secure solution for managing user identities and sessions, making it suitable for modern web applications.

Feel free to explore the code in my [GitHub repository](https://github.com/hkim-dev/nextjs-user-auth) to see this system in action. I hope this article helps you understand the principles behind secure user session management and inspires you to implement similar solutions in your projects.