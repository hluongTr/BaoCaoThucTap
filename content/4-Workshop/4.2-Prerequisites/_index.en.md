---
title: "Environment preparation"
date: 2026-11-07
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

## Prerequisites

| Component | Preparation |
| --- | --- |
| Repository | The current CloudDocs version, including Back_end, FrontEnd and scripts |
| Tools | Git, Java 21, Maven, MySQL client and Node.js; AWS CLI if using command-line operations |
| Database | An export of the existing schema and authorized test data |
| AWS | Permission to create/manage resources in the assigned demo environment |
| Environment | S3/RDS/EC2 Region; Cognito's own Region and issuer |
| Domain | Intended HTTPS origin and exactly matching callback/logout URLs |

Do not assume S3 and Cognito use the same Region: existing settings may differ. The backend uses AWS_REGION for S3; COGNITO_ISSUER_URI must identify the actual User Pool.

## Check the local source version

Run these commands in the CloudDocs application repository, not in the report website directory:

~~~powershell
java -version
mvn -version
node --version
mvn -f Back_end/pom.xml package
node --test FrontEnd/tests/*.test.cjs
~~~

Build/test commands do not establish connectivity to actual AWS resources. Save results and the commit version; record passing test counts only after execution.

## Run a local reference version

1. Import the correct schema and required migrations as described in the RDS section.
2. Configure the Cognito public client/callback in FrontEnd/js/cognito-config.js and API_BASE in app.js.
3. Use scripts/start-local-backend.ps1, which prompts for the database password. If using another pool, update issuer/client/UserInfo together; the current script assigns specific development values.
4. Use scripts/start-frontend.ps1 or node scripts/serve-frontend.cjs and open the printed URL.
5. Check /api/health and sign in. Local execution still uses Cognito; there is no alternative local-password sign-in mechanism.

**Output:** verified environment, commit, schema and public configuration. Keep secrets out of the report.
