# Deployment, Hosting, DNS and HTTPS: Theory and Implementation

> **TV2 report contribution.** This draft documents a managed PaaS deployment using Render Web Service and Neon PostgreSQL. Replace placeholders with verified facts and original screenshots. Keep tests as “Not run” until executed. Do not claim unperformed infrastructure tasks or invent measurements.

## Abstract

Deployment releases an application into an environment that users can reach. This chapter explains deployment, hosting, DNS and HTTPS, then describes a managed hosting procedure and a verification plan. The demonstration uses Render for the web runtime and Neon for PostgreSQL. Render supplies the public service hostname and manages TLS for that hostname. This is a PaaS deployment, not direct Linux VPS, Nginx, Certbot or registrar DNS administration. A test matrix covers deployment, restart, logs, HTTPS, persistence and performance; the team must fill results from its own evidence.

**Keywords:** deployment, hosting, PaaS, DNS, HTTPS, TLS, Render, PostgreSQL, Neon.

## 1. Introduction

A web application becomes useful when users can reach it and its supporting services work. Moving code from a development computer to a public environment requires compatible dependencies, environment configuration, persistent storage, network access and operational verification. A release should be traceable to a source revision, and its behavior should be checked after deployment.

The project uses managed hosting. Render builds and runs the web service from the team's connected source repository. Neon provides PostgreSQL. The team configures the source branch, build/start commands, application variables, database connection and health route. The provider operates the underlying host and public TLS layer.

This chapter explains the concepts, records the deployment procedure and evaluates it using repeatable checks. Complete these fields before submission:

- Application URL: <verified URL>
- Source branch and full commit SHA: <branch> / <SHA>
- Deployment date/timezone: <date/timezone>
- Testers: <names>

## 2. Theoretical background

### 2.1 Deployment

Deployment releases a defined application version into a target runtime. A repeatable release identifies the source revision, installs dependencies, builds or packages the application, supplies configuration, starts the process and checks behavior. Recording the commit and result supports diagnosis and rollback.

A deployment can be manual or automated. Manual steps may be suitable for a small coursework demo, but they should remain explicit and repeatable. Continuous integration and delivery automate build, test and release activities. Automation reduces variation, but does not replace reviewing logs, authorization and data persistence.

A hosted Node.js application must follow its runtime contract: install from the lockfile, use the declared start command, read environment variables and listen on the port/interface required by the host. Hard-coded local paths and embedded credentials make an application difficult to move and unsafe to share.

### 2.2 Hosting models

Hosting provides compute, network and related services that let software run and be reached. The project considers shared hosting, a virtual private server (VPS)/Infrastructure as a Service (IaaS), and managed Platform as a Service (PaaS).

| Criterion | Shared hosting | VPS / IaaS | Managed PaaS used |
|---|---|---|---|
| Operating system | Provider controlled | Team administers VM | Provider operates host/platform |
| Deployment | Provider-specific | Team installs runtime | Connect source and configure build/start |
| Proxy/process management | Provider-dependent | Team configures/maintains | Mostly platform-managed |
| Database | Bundled or restricted | Team-managed or separate | Neon managed PostgreSQL |
| Domain/TLS | Provider-dependent | Team configures | Provider hostname and managed TLS |
| Control | Low | High | Moderate at app level, limited host access |
| Operations burden | Low, with constraints | Highest; patching, firewall, services | Lower infrastructure burden, quotas/provider dependency |
| Fit | Depends on runtime support | Strong server-admin learning | Fast public coursework demonstration |

A VPS offers operating-system access and network control, while making the team responsible for updates, firewall, process supervision, reverse proxy, certificates and monitoring. PaaS reduces that operational burden and supports source-based releases, but abstracts the host and imposes platform limits. The team selected PaaS to publish the demo without maintaining a server. This demonstrates managed deployment, not direct Linux administration.

### 2.3 Domain names and DNS

The Domain Name System is a distributed, hierarchical naming system that maps human-readable names to records clients use to locate services. A and AAAA records associate names with IP addresses; CNAME records alias a name to another. DNS answers can be cached, so updates may take time to appear consistently.

A registrar manages domain registration; a DNS hosting provider publishes records. These may be the same or different organizations. Domain registration alone does not deploy an application: the owner must connect the name to a service and configure the required records.

The current demo uses a provider hostname ending in onrender.com. Render supplies and manages DNS for that hostname. The group has not demonstrated purchasing a custom domain or editing registrar records. If a custom domain is required later, add it to the service, configure the provider-prescribed DNS records, verify resolution and check its certificate. Until there is evidence, describe this as future work.

### 2.4 HTTPS and TLS

HTTPS is HTTP protected by Transport Layer Security (TLS). TLS establishes cryptographic protection and uses certificates to authenticate the server hostname. Certificate validation helps the browser determine whether it is communicating with the intended service. Encryption in transit reduces the risk of interception or modification.

HTTPS does not secure application logic by itself. Authorization flaws, weak credentials, exposed secrets, unsafe queries and missing backups remain possible. A managed platform may terminate TLS at its public edge and forward traffic to the application runtime. The report should describe that architecture accurately and should not claim end-to-end encryption without evidence.

For this project, the provider supplies TLS for the onrender.com hostname and manages certificate renewal and HTTP-to-HTTPS behavior. The team verifies the public HTTPS URL and browser certificate. It does not manually issue certificates, configure Nginx TLS directives or run Certbot. A custom domain requires separate setup and verification.

### 2.5 Configuration and persistent data

Web instances can restart or be replaced. Local files on an instance should not be considered durable unless persistent storage is configured. The project stores relational records in Neon PostgreSQL. Its connection URI is supplied to the service as a secret variable instead of being embedded in source code.

Environment settings include the database URI, JWT signing secret, allowed browser origins, runtime mode and port. Separating configuration from source lets the same code run in multiple environments. Secrets should be restricted, rotated if exposed, and excluded from Git history and screenshots. A runtime database account should have only required permissions; if a broad owner role is used, state that limitation and a hardening opportunity.

## 3. System architecture

    Browser --HTTPS--> Render-managed public edge/TLS
                            |
                            v
                     Node.js + Express
                            |
                      TLS database URI
                            v
                    Neon PostgreSQL

The source repository is connected to Render. A release selects a branch and commit, installs dependencies, starts the web process and publishes a service URL. The endpoint GET /api/health checks readiness according to the application's implementation. Insert its actual response contract from a successful captured request.

**Figure 1.** Actual deployment architecture. Insert the team's exported diagram and verify every label against the live service.

**Figure 2.** Render deployment event/runtime log showing source commit and startup. Redact account information and secrets.

## 4. Practical deployment procedure

### 4.1 Prerequisites

The operator needs access to the source repository, Render service and Neon project; compatible Node.js/npm for local checks; and a browser. Confirm Node version and commands from the application repository. Never copy a production URI, JWT secret or access token into this report or screenshots.

### 4.2 Prepare Neon PostgreSQL

1. Open the team's Neon project and select the intended branch/database.
2. Record region and plan observed at submission time.
3. Apply the approved schema/migration and confirm expected tables, keys, constraints and indexes.
4. Copy the TLS connection URI; use the pooled endpoint if the application is configured for it.
5. Store the URI in Render as secret DATABASE_URL.
6. Confirm connectivity through the health route and an application workflow.

**Evidence:** screenshot the schema/table list with credentials hidden.

### 4.3 Configure and deploy Render

1. Connect Render to the team's Git repository and choose the intended branch. Private repositories work when access is granted.
2. Create/open a Node.js Web Service.
3. Set install/build command to npm ci and start command to npm start if they match package.json and the lockfile.
4. Configure variables required by the app. The discussed names include NODE_ENV, HOST, DATABASE_URL, JWT_SECRET and CORS_ORIGINS; verify exact names against the application. The service must listen on the port Render assigns.
5. Set /api/health as health path only if its response is suitable.
6. Deploy and inspect build/runtime logs for dependency, configuration and DB errors.
7. Record URL, branch, commit SHA, time and instance plan.

For a local checkout, from the application root:

    node --version
    npm --version
    npm ci
    npm start

Local startup needs valid development configuration. Use an approved development database and .env.example. Inspect scripts with npm run and document only commands that exist. Do not publish production credentials.

### 4.4 Verify deployment

Confirm the dashboard event references the intended commit; the homepage loads over HTTPS; GET /api/health returns its documented healthy response; core workflows work with throwaway accounts; records are isolated and persist after restart; logs have no fatal errors or secrets; and the browser accepts the hostname certificate.

Example request; replace the placeholder:

    $base = 'https://<verified-render-hostname>'
    Invoke-WebRequest "$base/api/health" -Method Get

## 5. System testing and evidence

This is a test plan. Keep status as Not run until execution and evidence are recorded. Store original screenshots in the final repository and cite exact filenames.

| ID | Area/procedure | Expected result | Actual result/status | Evidence |
|---|---|---|---|---|
| DEP-01 Deploy | Compare dashboard event with intended branch/commit | Deploy succeeds; SHA matches | <Not run/pass/fail, date> | evidence/01-render-deploy.png |
| DEP-02 Startup/logs | Review runtime log after release | Process starts on assigned port; no fatal error | <record> | evidence/02-runtime-log.png |
| DEP-03 Health/DB | Request GET /api/health | Healthy HTTP response and database check | <status/body> | evidence/03-health.png |
| DEP-04 HTTPS | Open URL, inspect certificate, test HTTP redirect if enabled | Valid hostname cert; redirect behavior recorded | <record> | evidence/04-https.png |
| DEP-05 Restart | Restart from dashboard, request health again | Service healthy; recovery time recorded | <record> | evidence/05-restart.png |
| DEP-06 Logs | Filter around deploy/request/restart timestamps | Events found; no secrets in logs | <findings> | evidence/06-logs.png |
| DEP-07 Persistence | Create test record, restart service, retrieve it | Record remains in PostgreSQL | <record> | evidence/07-persistence.png |
| APP-01 Auth | Register/login with test account; try invalid credentials | Valid access works; invalid is rejected safely | <record> | evidence/08-auth.png |
| APP-02 Isolation | Attempt cross-account access with two test users | Users cannot access each other's data | <record> | evidence/09-isolation.png |
| APP-03 CRUD | Create/read/update/delete a test item | Operations yield expected state | <record> | evidence/10-crud.png |
| PERF-01 Warm | Make documented requests when service is awake | Status and latency recorded | <n, median, p95> | evidence/11-performance.png |
| PERF-02 Cold | After idle, time first and later request | Cold start separated from warm behavior | <times/state> | evidence/12-cold-start.png |

### 5.1 Performance measurement

Use a small repeatable sample suitable for a course demo. Record endpoint, tool/version, request count, network context, timestamp, service state (awake/after idle), status codes and latency. Report median and p95 when the sample supports it. Separate the first request after idle from warm requests. One request is not a benchmark; a small sample does not demonstrate production capacity or uptime.

| Observation | Conditions | Result | Interpretation |
|---|---|---|---|
| First request after idle | <endpoint/tool/date/state> | <ms/status> | <cold-start effect> |
| Warm median | <n/tool/version> | <ms> | <cautious interpretation> |
| Warm p95 | <n/tool/version> | <ms> | <cautious interpretation> |
| Failures/timeouts | <count/total> | <result> | <interpretation> |

### 5.2 Restart and log analysis

Compare deployed commit with intended source. Record whether startup logs show the expected version and whether health succeeds after release. For restart, note the dashboard action, start/end times, first healthy response and whether the instance was active or waking. A test record remaining after restart supports database persistence; it does not prove backup recovery or high availability.

Build logs describe dependency installation; runtime logs show startup and application output. Locate events by timestamp and commit, explain errors and resolutions, and exclude credentials, authorization headers, database URIs and personal data. If the app lacks useful log detail, report an observability limitation instead of inventing events.

### 5.3 Screenshot analysis

Insert original, legible, numbered screenshots for deploy/commit, startup logs, health response, certificate, Neon schema, restart/persistence, workflow and performance. Explain what each figure establishes. Redact credentials, emails, tokens and irrelevant account information. Screenshots support analysis; they do not replace it.

## 6. Results, limitations and security

Complete this section after tests. Summarize passed, failed and unrun cases; discuss restart, persistence, latency and errors from the actual matrix. Reference figure numbers and captions.

The PaaS delegates host operating system, public hostname DNS, TLS issuance/renewal and process management to Render. The project does not evidence team-operated Ubuntu, firewall rules, Nginx, PM2/systemd, Certbot or registrar DNS. If the rubric expects self-managed infrastructure, state this scope gap and clarify with the instructor or add a separate lab with its own evidence.

Free or low-cost tiers may have quotas, idle behavior and plan changes. Record observed conditions and date; do not claim production availability from a course demo. Protect DB credentials and JWT secrets. A restricted runtime DB role, backup/restore, monitoring, failure recovery and custom domain are possible future work.

## 7. Conclusion

Deployment combines a repeatable release process with runtime configuration, persistent storage, public routing and verification. A VPS grants direct control with greater maintenance responsibility. PaaS reduces server operations while limiting low-level access. DNS maps names to service destinations, and HTTPS uses TLS to authenticate and protect network traffic.

The project demonstrates a managed deployment: Render runs the web service and supplies a public hostname and managed TLS; Neon stores PostgreSQL data. Evidence must establish the deployed revision, health/database connectivity, core workflow, restart/persistence, logs, HTTPS and performance observations. The report should state both demonstrated capabilities and the limits of provider-managed infrastructure.

## References

Use the Faculty-required citation style if supplied. Verify access dates and versions before submission.

[1] P. Mell and T. Grance, “The NIST Definition of Cloud Computing,” NIST SP 800-145, 2011. https://doi.org/10.6028/NIST.SP.800-145.

[2] P. Mockapetris, “Domain Names—Concepts and Facilities,” RFC 1034, IETF, 1987. https://www.rfc-editor.org/rfc/rfc1034.

[3] E. Rescorla, “The Transport Layer Security (TLS) Protocol Version 1.3,” RFC 8446, IETF, 2018. https://www.rfc-editor.org/rfc/rfc8446.

[4] Render, “Web Services.” https://render.com/docs/web-services.

[5] Render, “Deploy for Free.” https://render.com/docs/free.

[6] Render, “Fully Managed TLS Certificates.” https://render.com/docs/tls.

[7] Render, “Environment Variables and Secrets.” https://render.com/docs/configure-environment-variables.

[8] Render, “Health Checks.” https://render.com/docs/health-checks.

[9] Render, “Custom Domains.” https://render.com/docs/custom-domains.

[10] Neon, “Connection Pooling.” https://neon.com/docs/connect/connection-pooling.

[11] Neon, “Plans and Usage.” https://neon.com/docs/introduction/plans.

[12] Node.js, “Environment Variables.” https://nodejs.org/api/environment_variables.html.

[13] The Twelve-Factor App, “III. Config.” https://12factor.net/config.

[14] J. Humble and D. Farley, Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation. Addison-Wesley, 2010. https://martinfowler.com/books/continuousDelivery.html.

[15] PostgreSQL Global Development Group, “PostgreSQL Documentation.” https://www.postgresql.org/docs/current/.
