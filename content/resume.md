---
title: "Resume"
description: "Resume of Yann Labour, CTO and full-stack engineer: CapStellar, Diadeis/SGSco, Laravel, applied AI, DevOps, and 18 years building and running software."
layout: "single"
showToc: true
tocOpen: true
showReadingTime: false
showShareButtons: false
editPost:
  disabled: true
---

<a class="hero-link primary resume-download" href="/Yann_Labour-Resume.pdf">download pdf &darr;</a>

<p class="contact-line"><a href="mailto:yann@sugarsplashes.com">yann@sugarsplashes.com</a> &middot; {{< phone >}}</p>

## Summary

CTO and full-stack engineer with 18 years across web platforms, DevOps, and system architecture. At CapStellar (a crypto/VASP) I lead a small engineering team and own architecture, infrastructure, security, and applied AI. I re-platformed a Node/Python monolith to Laravel microservices, shipped multi-exchange (Binance/Kraken) crypto products, and built an AI support agent that resolves about 1 in 3 tickets end-to-end. Specialist in Laravel, automation, and AI-driven operating leverage. Based in Mauritius (UTC+4, full EMEA overlap), working remotely; open to relocation (Europe / UAE).

## Experience

### Chief Technology Officer (CTO) — CapStellar, Flow Corporation
*July 2025 – Present · Remote / Mauritius*

Virtual Asset Service Provider (VASP) — crypto portfolio automation.

- Re-architected the platform (V2 to V3) from a Node.js/Python monolith to a Laravel microservices architecture across multiple production environments
- Shipped core product end-to-end: automated multi-exchange (Binance + Kraken) index-rebalancing engine, zero-downtime Binance-to-Kraken migration, Earn/yield, and prepaid-wallet fee settlement
- Built a Claude-based AI support agent (RAG + secure tool integration) that resolves ~31% of tickets end-to-end with zero human touch, letting the support team absorb a 4x volume surge without added headcount
- Designed ~75 automation workflows and established continuous delivery (CI/CD, feature flags, event sourcing), shipping 200+ production merges per month with a three-person team
- Built a strategy backtesting simulator over historical market data — configurable indices, entry/exit rules and capital management, surfacing Sharpe, drawdown and trade-level analytics, with an AI natural-language strategy mode
- Built the data platform and reporting pipelines powering operational and revenue dashboards for company leadership
- Owned technical remediation of an external penetration test and CIS/ANSSI configuration audit; co-authored ICT operational-resilience governance under the EU DORA regulation as interim ICT security lead
- Embedded automated security scanning in the delivery pipeline: SAST, dependency (SCA), secrets, container-image and infrastructure-as-code checks on every pull request
- Integrated ComplyAdvantage for KYC and identity-document verification in client onboarding
- Sole owner of infrastructure: a ~20-server multi-provider fleet (DigitalOcean, Hetzner, IONOS) managed via Laravel Forge plus serverless workloads on Laravel Cloud, a multi-region API with automated failover (Cloudflare Load Balancer, warm standby, MySQL read-replica routing), and a Tailscale zero-trust network
- Built a single-pane-of-glass operations console monitoring the entire estate — server vitals, health checks, deploys, CI, uptime and security findings — with automated incident detection and Slack alerting

**Tech:** Laravel, API Platform, Python, Node.js, Vue, Inertia, React (platform stack), Claude/LLM, RAG, MCP, n8n, PostgreSQL, MongoDB, BigQuery, Redis, Laravel Forge, Laravel Cloud, Docker, CI/CD, Cloudflare (LB/WAF/Tunnels), Tailscale, gRPC, Aikido (SAST/SCA/secrets/container/IaC), ComplyAdvantage

### Software Engineering Manager — Diadeis / SGSco
*May 2024 – June 2025 · Remote · Software Engineering team / Global*

- Rebuilt color profile system with Data Science team, creating industry-leading tool that strengthened company's market position
- Developed client-facing search application integrating with Data Science team's API
- Created automated file delivery solution saving US$45K/month globally
- Built Adobe Illustrator/InDesign extension for automated solutions and content injection
- Implemented critical automated quality check systems at group level
- Added vulnerability scanning to the Azure deployment path: infrastructure-as-code analysis and container-image checks before release

**Tech:** Laravel, Node.js, Vue.js, Inertia.js, .NET Core, JS/Adobe ExtendScript, Azure SQL, MariaDB, Redis, Postgres, MongoDB, Terraform, Azure DevOps, GitHub Actions, Azure App Service, Container Apps, Docker

### Development Team Lead — Diadeis / SGSco
*May 2021 – April 2024 · Remote · Data Science team / North America*

### Development Team Lead — Diadeis / SGSco
*June 2019 – April 2021 · Mauritius · Operations / Mauritius site*

### Co-founder and Lead Developer — Sugarsplashes Ltd
*September 2015 – May 2019 · Mauritius*

- Built a GIS platform for ARS Réunion (regional health agency): island-wide tracking of mosquito-borne disease and vector-control operations, Leaflet mapping over spatial data. Year-long engagement renewed for a second year; still in production at geolav.re
- Partnered with Barclays Incubator startup to develop CX field proof-of-concept
- Client-facing role for pitches, proposals and estimates

**Tech:** Laravel, Vue.js, Leaflet, GIS / spatial data, MySQL, Git, Docker, Ubuntu

### Lead Developer, IT Manager, Web Developer — Earlier roles
*August 2008 – August 2015 · Mauritius / La Réunion*

- TF Ltd (Lead Developer, team of 4): Laravel/MongoDB transport platform for a startup, microservices and REST API, server configuration and deployment
- Activeline – Harel Mallac (Head of Web Department): technical lead on web projects, feasibility studies and costing, systems administration
- Evolution Prépresse (IT Manager, team of 3): built an intranet project-management and workflow application; InDesign/Illustrator automation cutting production time by up to 50%; ran LDAP/DNS/DHCP, MySQL replication and a virtualised Linux estate
- Université de La Réunion (Web Developer, then Engineer): BabelWEB European e-learning project, and ORI-OAI / ESUP Portail / Nuxeo deployments plus data-exchange tooling at the TICE unit

**Tech:** Laravel, Yii, AngularJS, MySQL, MongoDB, PostgreSQL, XML/XSLT, Chef, Linux, Mac OS X Server

## Education

### Data Science BootCamp — Le Wagon (2023)
Finals project "Deep Dive" — marine debris detection using transfer learning with VGG19. Classified underwater images into 17 pollution categories with 60% accuracy.
[GitHub](https://github.com/yayann/deep_dive)

### Master (MSc) Computer Science — Université de la Réunion (2009)
Information and Communication Sciences and Technologies

## Skills

| Domain | Technologies |
|--------|-------------|
| **Development** | PHP: Laravel & ecosystem (Horizon, Nova, Passport, Scout) · Python (FastAPI) · JavaScript: Vue.js, Inertia, Node.js, jQuery, Adobe ExtendScript · TailwindCSS, Bootstrap · REST/gRPC APIs, SPAs, background jobs · SQS, Beanstalk |
| **AI & Automation** | Claude / Anthropic API · RAG, vector search (PGVector) · Model Context Protocol (MCP) tools · n8n workflow automation · Agent evaluation, confidence scoring, human-in-the-loop |
| **Data** | MySQL, MongoDB, PostgreSQL, SQL Server, BigQuery, Redis |
| **Data Science** | Python (Pandas, NumPy, Scikit-Learn, TensorFlow) · Machine Learning fundamentals · Data processing and analysis |
| **Cloud & DevOps** | Multi-provider VPS fleet: DigitalOcean, Hetzner, IONOS, AWS (Laravel Forge) · PaaS / serverless: Laravel Cloud, Cloudflare Pages · Azure (App Service, Container Apps, Functions, DevOps) · Multi-region HA: Cloudflare Load Balancing, warm standby, DB read-replica routing · Docker · CI/CD (GitHub Actions, Azure DevOps), zero-downtime deploys · Terraform |
| **Security & Domain** | Application security, penetration-test remediation (OWASP, CIS/ANSSI) · Pipeline security automation: SAST, SCA, secrets, container-image and IaC scanning (Aikido) · KYC / identity-document verification integration (ComplyAdvantage) · Encryption & key management · Digital assets / crypto (CeFi), Fintech · VASP & regulatory transition (MiCA, DORA) |
| **Systems** | Linux fleet administration (Ubuntu, Debian) · Nginx, Apache, PHP-FPM, Octane/FrankenPHP · Zero-trust networking: Tailscale, Cloudflare Access, Tunnels, WAF · Monitoring & observability: health checks, heartbeats, uptime/cron monitoring, incident alerting · Microservices architecture |

## Languages

- **English** — Fluent
- **French** — Native
- **Spanish** — Basic
