---
title: "Authentication vs. Authorization"
date: 2023-11-09 10:32:28 -0400
categories: "web_dev"
header:
  teaser: "/assets/images/auth.png"
tags:
    - authentication
    - authorization
toc: true
---

![auths](/assets/images/auth.png)

*source: https://www.pedromonjo.com/2018/11/authentication-authorisation-aec.html*

# Background

The terms are not new not only to anyone who has some experience with web application development but also to anyone who has ever logged in. The two terms sound basically identical - at least to us non-native English speakers. In the colloquial sense, they might be interchangeable. However, the differences between them are as clear as ever in the context of Cybersecurity. After coming across these terms once again while reading about OAuth, I would like to clarify the differences between them in this post once and for all.

# Authentication vs. Authorization

According to the Oxford Dictionary, the term authentication is defined as the process or action of verifying the identity of a user or process in computing. In simple terms, it is **the process of confirming that users are who they claim to be**.

On the other hand, authorization is a **process by which a server determines if the client has permission to use a resource or access a file**. You can imagine users asking to be authorized to access certain resources on the Internet. 

Their differences can be summarized as follows:

| Authentication | Authorization |
| --- | --- |
| Determines whether users are who they claim to be | Determines what users can and cannot access |
| Challenges the user to validate credentials (for example, through passwords, answers to security questions, or facial recognition) | Verifies whether access is allowed through policies and rules |
| Usually done before authorization | Usually done after successful authentication |
| Generally, transmits info through an ID Token | Generally, transmits info through an Access Token |
| Generally governed by the OpenID Connect (OIDC) protocol | Generally governed by the OAuth 2.0 framework |

The primary purpose of an ID token is to provide information about the authenticated user. It usually contains the user’s unique identifier, name, email, and other profile information. An access token, however, is used to authorize the client application and make requests to resource servers (APIs) on behalf of the authenticated user. 

## OAuth

In the context of authentication and authorization, [OAuth](https://oauth.net/2/) is primarily an authorization framework and does not handle user authentication itself. Let’s walk through a simple scenario to find out How it handles user authentication and authorization.

- A user wants to access a service or resource on a client application (e.g., a mobile app, or a web app).
- The client application redirects the user to the authorization server to request permission.
- The user interacts with the server and authenticates themselves. The authentication method can vary, including username/password, multi-factor authentication, or authentication via a third-party identity provider.
- The server issues an access token and the client application includes the token in its requests to the resource server (API).

By separating the concerns of authentication and authorization, OAuth 2.0 provides a robust framework that enhances security and scalability. It allows the authentication process to happen independently of the authorization process, providing flexibility and enabling seamless integration with various identity providers.

#### references:
- https://www.bu.edu/tech/about/security-resources/bestpractice/auth/
- https://oauth.net/2/access-tokens/
- https://www.okta.com/identity-101/authentication-vs-authorization/