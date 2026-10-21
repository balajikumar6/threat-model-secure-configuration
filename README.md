# Threat Model and Secure Configuration

**Track:** Cyber Security  
**Level:** Beginner-friendly  
**Estimated time:** 45–60 minutes  

---

## Target Scenario: Small B2B SaaS Application

To make the analysis concrete and practical, this threat model evaluates a standard **Multi-Tenant Note & Document Collaboration SaaS** (`Notely App`) consisting of a Single Page Application (React), a REST/GraphQL API gateway (Node.js/Express), a managed PostgreSQL database, and external third-party integrations (Stripe, SendGrid, and AWS S3).

---

## 1. System Architecture & Data Flow Diagram (DFD)

The ASCII architecture diagram below maps the actors, data stores, external dependencies, and trust boundaries:

```text
================================== UNTRUSTED ZONE ==================================
                            [ Browser / Mobile Client ]
                                         |
                                         | HTTPS / TLS 1.3
                                         v
-------------------------------- TRUST BOUNDARY 1 (Internet / Perimeter) -----------
                           [ Cloudflare CDN & WAF / Reverse Proxy ]
                                         |
                                         | Reverse Proxy (mTLS / Internal VPC)
                                         v
================================ APPLICATION DMZ ===================================
                              [ API Gateway / Web Server ]
                              - Authentication Middleware
                              - Input Validation & Rate Limiter
                              - Business Logic & RBAC
                                 /       |       \
            Internal SQL / TCP  /        |        \   HTTPS / SDK
        ----------------------------     |         ---------------------------
       |                                 |                                   |
       v                                 v                                   v
----------------- TRUST BOUNDARY 2 -----------               --------------- TRUST BOUNDARY 3 -
[ Managed PostgreSQL ]             [ Local File Cache / S3 ]       [ Third-Party APIs ]
- Users & Hash PII                 - Uploaded Document Blobs       - Stripe (Billing)
- Tenant Document Tables           - Presigned URL Generator       - SendGrid (Email)
- Audit & Auth Logs                                                - Auth0 / OAuth IdP
================================ DATA PERSISTENCE ZONE =============================
