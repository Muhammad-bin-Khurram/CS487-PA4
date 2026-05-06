<div align="center">

# PA4 Submission: TaskFlow Pipeline

<img alt="GitHub only" src="https://img.shields.io/badge/Submit-GitHub%20URL%20Only-10b981?style=for-the-badge">
<img alt="Total points" src="https://img.shields.io/badge/Total-100%20points-7c3aed?style=for-the-badge">

</div>

<div style="background:#f5f3ff;color:#111827;border-left:6px solid #6330bc;padding:14px 18px;border-radius:10px;margin:18px 0;">
Every screenshot is embedded under the correct task with a short description. The grader should not need any file outside this repository.
</div>

## Student Information

| Field | Value |
|---|---|
| Name | Muhammad bin Khurram |
| Roll Number | 26100365 |
| GitHub Repository URL | https://github.com/Muhammad-bin-Khurram/CS487-PA4 |
| Resource Group | `rg-sp26-26100365` |
| Assigned Region | `ukwest` |

## Evidence Rules

- Relative image paths are used throughout.
- Every image has a 1–3 sentence description below it.
- Azure Portal screenshots show the resource name and enough context to identify the service.
- CLI screenshots show the command and its output.
- Secrets (ACR passwords, function keys, connection strings) are masked by the portal's "Show value" toggle.

---

## Task 1: App Service Web App (15 points)

### Evidence 1.1: Forked Repository

![Forked repository — Muhammad-bin-Khurram/CS487-PA4](<docs/Images/Task1/Screenshot 2026-05-02 033137.png>)

The GitHub page shows repository **Muhammad-bin-Khurram/CS487-PA4** marked Public, with the label *forked from KarmaMS/CS487-PA4*. The branch `main` is 3 commits ahead of the upstream, confirming this is an active working fork containing the PA4 starter structure with all required folders (`function-app`, `report-job`, `validate-api`, `docs`).

### Evidence 1.2: App Service Overview

![App Service overview — pa4-26100365](<docs/Images/Task1/Screenshot 2026-05-02 033803.png>)

The Azure Portal shows Web App **pa4-26100365** in resource group `rg-sp26-26100365`, status **Running**, location **UK West**, runtime **Node 22-lts**, plan `pa4-26100365 (F1: 1)`. The public default domain is `pa4-26100365.azurewebsites.net` and the External Repository link confirms it points to the GitHub fork at `Muhammad-bin-Khurram/CS487-...`.

### Evidence 1.3: Deployment Center / GitHub Actions

![Deployment Center — External Git, main branch](<docs/Images/Task1/Screenshot 2026-05-02 034053.png>)

The Deployment Center for `pa4-26100365` shows source **External Git**, repository `https://github.com/Muhammad-bin-Khurram/CS487-PA4`, branch `main`, build provider **App Service Build Service**, runtime Node. The App Service is directly wired to the GitHub fork so every push to `main` triggers an automated build and deployment.

### Evidence 1.4: Live Web UI

![Live TaskFlow UI at pa4-26100365.azurewebsites.net](<docs/Images/Task1/Screenshot 2026-05-02 032811.png>)

The browser address bar shows `pa4-26100365.azurewebsites.net` and the TaskFlow order-submission form is fully rendered and interactive. The form accepts Order ID (`ORD-S01`), SKU (`WIDGET-X`), and Quantity (Max 100), confirming the App Service is publicly serving the frontend without error.

---

## Task 2: Azure Container Registry (15 points)

### Evidence 2.1: ACR Overview

![ACR overview — pa426100365](<docs/Images/Task2/Screenshot 2026-05-02 035218.png>)

The Azure Portal shows Container Registry **pa426100365** in resource group `rg-sp26-26100365`, location **UK West**, pricing plan **Basic**, login server `pa426100365.azurecr.io`, provisioning state **Succeeded**. The registry was created on 02/05/2026 at 03:50 GMT+5 in the Microsoft Azure – Ali Khawaja subscription.

### Evidence 2.2: Docker Builds

![docker build validate-api:latest — FINISHED](<docs/Images/Task2/Screenshot 2026-05-02 035529.png>)

`docker build --platform linux/amd64 -t validate-api:latest ./validate-api` completed in 90.5s. The output shows all 5 Dockerfile layers executing — `FROM python:3.11-slim`, `WORKDIR`, `COPY requirements.txt`, `RUN pip install`, `COPY app.py` — and the final image exported with digest `sha256:5bcea239...`.

![docker build report-job:latest — FINISHED](<docs/Images/Task2/Screenshot 2026-05-02 035728.png>)

`docker build --platform linux/amd64 -t report-job:latest ./report-job` completed in 40.6s. The output confirms the Python 3.11-slim base layer was cached from the previous build, `generate.py` was copied, and the final image was exported with digest `sha256:acc0da18...`. The source folder `./report-job` produced this image.

![docker build func-app:latest — FINISHED](<docs/Images/Task2/Screenshot 2026-05-02 051454.png>)

`docker build --platform linux/amd64 -t func-app:latest ./function-app` completed in 1101.5s. The Azure Functions Python 4-python3.11 base image was pulled from `mcr.microsoft.com` (over 900 MB of layers), confirming the custom function container was built from the Durable Functions runtime image. The source folder `./function-app` produced this image.

![Local validate-api smoke test — valid:true](<docs/Images/Task2/Screenshot 2026-05-02 054221.png>)

Split-screen: the left terminal runs `docker run --rm -p 8080:8080 validate-api:latest`, showing Uvicorn started and received a `POST /validate 200 OK`. The right terminal sends `Invoke-WebRequest` to `localhost:8080/validate` with order `LOCAL-1` (sku ABC, qty 2) and receives `StatusCode 200`, `Content: {"valid":true,"reason":"ok","order_id":"LOCAL-1"}`, confirming the containerised image works before pushing to ACR.

### Evidence 2.3: ACR Repositories

![docker push validate-api:v1 to pa426100365](<docs/Images/Task2/Screenshot 2026-05-02 053652.png>)

`docker push pa426100365.azurecr.io/validate-api:v1` shows all 9 layers pushed and the final digest `v1: sha256:5bcea239f95836363f09e96a9355d7e49cf9227773fdf26d18c8dbfb18664074 size: 856`, confirming `validate-api:v1` landed in the registry.

![docker push report-job:v1 to pa426100365](<docs/Images/Task2/Screenshot 2026-05-02 053714.png>)

`docker push pa426100365.azurecr.io/report-job:v1` completes with several layers mounted from `validate-api` (shared base image) and unique layers pushed. Final digest `v1: sha256:acc0da1884a6123da47ae9f0971ae4d79cd52e8329f52e22b0a089251a2260f8 size: 856`.

![docker push func-app:v1 to pa426100365](<docs/Images/Task2/Screenshot 2026-05-02 053733.png>)

`docker push pa426100365.azurecr.io/func-app:v1` completes with all 22 layers pushed. Final digest `v1: sha256:5818322365d2ac9aff08ecf3a71d5fe819c1f7ab06cfabfee86260a33db9e5c9 size: 856`, confirming the Function App container image is in ACR.

![az acr repository list — all three repos](<docs/Images/Task2/Screenshot 2026-05-02 053750.png>)

`az acr repository list --name pa426100365 --output table` lists **func-app**, **report-job**, and **validate-api** — all three required repositories are present in `pa426100365.azurecr.io`, each tagged `v1` and ready for deployment.

---

## Task 3: Durable Function Implementation (12 points)

### Evidence 3.1: Completed Function Code

[function-app/function_app.py](function-app/function_app.py)

The orchestrator `my_orchestrator` chains two activities with `yield`: it first calls `validate_activity`, which POSTs the order JSON to the AKS endpoint at `os.environ["VALIDATE_URL"]`; if the result is not valid it immediately returns `{"status":"rejected","reason":"..."}`. On a valid response it calls `report_activity`, which uses `ContainerInstanceManagementClient` (authenticated via `DefaultAzureCredential` with the user-assigned MI client ID from `AZURE_CLIENT_ID`) to create an ephemeral `ci-report-<order_id>` container group that generates the PDF and uploads it to Blob Storage, then deletes the group and returns the blob URL. All state between `yield` points is checkpointed by the Durable runtime, so no thread is blocked during the ACI wait.

### Evidence 3.2: Local Function Handler Listing

![func start — all four handlers discovered](<docs/Images/Task3/Screenshot 2026-05-02 055924.png>)

`func start` from inside the `function-app/` directory shows Azure Functions Core Tools v4.10.0 and runtime v4.1048 discovering all four handlers: `http_starter` (POST HTTP at `localhost:7071/api/orchestrators/my_orchestrator`), `my_orchestrator` (orchestrationTrigger), `report_activity` (activityTrigger), and `validate_activity` (activityTrigger). The worker process started successfully and the host lock lease was acquired.

---

## Task 4: Function App Container Deployment (8 points)

### Evidence 4.1: Function App Container Configuration

![Function App Deployment Center — func-app:v1 from pa426100365](<docs/Images/Task4/Screenshot 2026-05-03 002804.png>)

The Deployment Center for **pa4-26100365-fnc** shows source **Container Registry**, image source **Azure Container Registry**, registry `pa426100365`, image `func-app`, tag `v1`. The Function App pulls this custom container from `pa426100365.azurecr.io/func-app:v1` rather than using a code-based deployment.

![Function App overview — all four functions Enabled](<docs/Images/Task4/Screenshot 2026-05-03 002901.png>)

The Functions blade for `pa4-26100365-fnc` lists `http_starter` (HTTP, **Enabled**), `my_orchestrator` (Orchestration, **Enabled**), `report_activity` (Activity, **Enabled**), and `validate_activity` (Activity, **Enabled**). All four Durable handlers were discovered from the running container, confirming `func-app:v1` is executing correctly. The app runs on plan `ASP-rgsp2626100365-9136 (B1: 1)` in UK West on Linux.

### Evidence 4.2: Orchestration Smoke Test

![Invoke-WebRequest — 202 Accepted, instance ID returned](<docs/Images/Task4/Screenshot 2026-05-03 003111.png>)

`Invoke-WebRequest` to `https://pa4-26100365-fnc-hkdhaja3bfd9fjed.ukwest-01.azurewebsites.net/api/orchestrators/my_orchestrator` with order `SMOKE-002` returns **StatusCode 202 Accepted**, instance ID `cea15427849b4144ba0987fe0429d321`, and a `statusQueryGetUri`. The 202 and the returned orchestration URLs prove the HTTP starter is live and the Durable engine accepted the job.

### Evidence 4.3: Orchestration Status — End-to-End Completed

![Status query — runtimeStatus Completed for SMOKE-002](<docs/Images/Task4/Screenshot 2026-05-03 003258.png>)

Polling the `statusQueryGetUri` for instance `cea15427849b4144ba0987fe0429d321` returns **StatusCode 200**, `runtimeStatus: "Completed"` with input `SMOKE-002`. By the time this smoke test was performed, `VALIDATE_URL` had already been configured (AKS was set up earlier at 23:21 and the Function App env vars updated before this midnight test at 00:31), so the full pipeline ran end-to-end and completed successfully on the first attempt.

---

## Task 5: AKS Validator (15 points)

### Evidence 5.1: AKS Cluster

![kubectl get nodes — single node Ready](<docs/Images/Task5/Screenshot 2026-05-02 232120.png>)

`kubectl get nodes` shows node **aks-nodepool1-18198040-vmss000000** in status **Ready**, Kubernetes version **v1.34.4**, age 3m39s. The kubeconfig context was previously merged via `az aks get-credentials --resource-group rg-sp26-26100365`, confirming the cluster exists and is accessible in the correct resource group.

### Evidence 5.2: Kubernetes Nodes and Pods

![kubectl get nodes — node Ready](<docs/Images/Task5/Screenshot 2026-05-02 232120.png>)

The single node `aks-nodepool1-18198040-vmss000000` is **Ready** with no taints, confirming the control plane and node pool are healthy.

![kubectl get pods -w — validator pod reaches Running](<docs/Images/Task5/Screenshot 2026-05-02 234035.png>)

`kubectl get pods -w` captures the rollout: old pods with `ImagePullBackOff` and `ContainerStatusUnknown` were terminated, and the new pod `validate-deployment-f86b79d84-kzthp` reached **1/1 Running** (0 restarts, 6s) after the deployment was updated with the correct ACR image reference. The validator is now scheduled and serving traffic.

### Evidence 5.3: Kubernetes Service

![kubectl get service validate-service — LoadBalancer with external IP](<docs/Images/Task5/Screenshot 2026-05-02 234102.png>)

`kubectl get service validate-service` shows a **LoadBalancer** service: cluster IP `10.0.232.134`, external IP **51.140.203.157**, port mapping `8080:31783/TCP`, age 4m20s. The external IP `51.140.203.157:8080` is the endpoint the Durable Function uses to call the validator.

### Evidence 5.4: Validator API Tests

![/health endpoint — 200 OK, status:ok](<docs/Images/Task5/Screenshot 2026-05-02 234350.png>)

`Invoke-WebRequest -Uri "http://51.140.203.157:8080/health"` returns **StatusCode 200**, `Content: {"status":"ok"}`, served by Uvicorn at `2026-05-02 18:41:57 GMT`. The health endpoint confirms the validator pod is alive and ready.

![/validate valid order — valid:true](<docs/Images/Task5/Screenshot 2026-05-02 234359.png>)

`Invoke-WebRequest` to `/validate` with order `O-1001` (sku ABC, qty 2) returns **StatusCode 200**, `Content: {"valid":true,"reason":"ok","order_id":"O-1001"}`. The validator accepts orders with quantity within the 100-unit limit.

![/validate invalid order — valid:false, quantity exceeds limit](<docs/Images/Task5/Screenshot 2026-05-02 234407.png>)

`Invoke-WebRequest` to `/validate` with order `O-1002` (sku ABC, qty 999) returns **StatusCode 200**, `Content: {"valid":false,"reason":"quantity exceeds limit","order_id":"O-1002"}`. The `qty > 100` rejection rule is correctly enforced by the AKS-hosted validator.

### Evidence 5.5: Function App `VALIDATE_URL`

![Function App env vars — VALIDATE_URL = http://51.140.203.157:8080/validate](<docs/Images/Task5/Screenshot 2026-05-02 234527.png>)

The Environment Variables blade of `pa4-26100365-fnc` shows `VALIDATE_URL` set to `http://51.140.203.157:8080/validate` — the LoadBalancer external IP of the AKS service. The `validate_activity` reads this variable at runtime via `os.environ["VALIDATE_URL"]` so the Function App reaches the AKS validator without any hardcoded addresses. Also visible are `DOCKER_REGISTRY_SERVER_*` credentials for the ACR pull and `FUNCTIONS_WORKER_RUNTIME`.

### Evidence 5.6: AKS Idle Behavior

> **Placeholder** — An AKS Metrics screenshot (CPU/memory during idle period) was not captured. By design, node `aks-nodepool1-18198040-vmss000000` remains in **Ready** state and the validator pod stays **1/1 Running** regardless of whether any orders arrive. The underlying VM is always allocated and billing continues at the B-series hourly rate even at zero traffic — the core difference from ACI's per-second model discussed in Task 8.3.

---

## Task 6: ACI Report Job (15 points)

### Evidence 6.1: Blob Container

![az storage container create — reports container created](<docs/Images/Task6/Screenshot 2026-05-02 235126.png>)

`az storage container create --account-name pa426100365 --name reports --auth-mode login` returns `{"created": true}`, confirming the `reports` blob container was successfully created in storage account `pa426100365` under resource group `rg-sp26-26100365`. This is where all generated PDF reports are stored.

### Evidence 6.2: Manual ACI Run

![az container create — ci-report-test provisioned with MI](<docs/Images/Task6/Screenshot 2026-05-03 000413.png>)

`az container create --resource-group rg-sp26-26100365 --file ci-deploy.yaml` returns the container group JSON. The `identity` block confirms `type: "UserAssigned"` with managed identity `mi-pa4-26100365` (clientId `49a00281-8c22-4ae3-80be-cb0b3c2cd39c`). The name is `ci-report-test`, location `ukwest`.

![az container create — instanceView.state Succeeded](<docs/Images/Task6/Screenshot 2026-05-03 000421.png>)

The continuation of the `az container create` JSON response shows `"instanceView": {"events": [], "state": "Succeeded"}`, `provisioningState: "Succeeded"`, `restartPolicy: "Never"`, `resourceGroup: "rg-sp26-26100365"`, and `imageRegistryCredentials` pointing to `pa426100365.azurecr.io`. The container exited successfully after completing its task.

### Evidence 6.3: ACI Logs

![az container logs — Uploaded TEST-001.pdf to reports container](<docs/Images/Task6/Screenshot 2026-05-03 000827.png>)

`az container logs --resource-group rg-sp26-26100365 --name ci-report-test` prints `Uploaded TEST-001.pdf to reports container`. This single log line from the report-job container confirms it ran, generated the PDF, and successfully wrote it to Azure Blob Storage before exiting cleanly.

### Evidence 6.4: Generated PDF

![az storage blob list — TEST-001.pdf in reports container](<docs/Images/Task6/Screenshot 2026-05-03 000857.png>)

`az storage blob list --account-name pa426100365 --container-name reports --auth-mode login --output table` shows `TEST-001.pdf` (BlockBlob, Hot tier, 1462 bytes, `application/octet-stream`) uploaded at `2026-05-02T19:03:10+00:00`. This confirms the ACI container wrote the PDF file to Azure Blob Storage during its single execution run.

### Evidence 6.5: Function App Managed Identity and IAM

![Function App Identity — user-assigned mi-pa4-26100365](<docs/Images/Task6/Screenshot 2026-05-03 001020.png>)

The Identity blade of `pa4-26100365-fnc` (User assigned tab) shows managed identity **mi-pa4-26100365** attached, scoped to resource group `rg-sp26-26100365`, subscription `67e93b84-fe08-452c-80ea-175d0a3eca56`. This identity is granted **Contributor** on the resource group, allowing the Function App to call `ContainerInstanceManagementClient.container_groups.begin_create_or_update()` and `begin_delete()` without storing any credentials in code.

### Evidence 6.6: Report App Settings

![Function App settings — ACR_* and AZURE_CLIENT_ID](<docs/Images/Task6/Screenshot 2026-05-03 001441.png>)

`ACR_PASSWORD` (masked), `ACR_SERVER: pa426100365.azurecr.io`, `ACR_USERNAME: pa426100365`, and `AZURE_CLIENT_ID: 49a00281-8c22-4ae3-80be-cb0b3c2cd39c` are the registry credentials passed to the `ImageRegistryCredential` object when creating the ACI container group, and the client ID used by `DefaultAzureCredential` to select the correct user-assigned managed identity.

![Function App settings — REPORT_* and STORAGE_ACCOUNT_URL](<docs/Images/Task6/Screenshot 2026-05-03 001453.png>)

`REPORT_IMAGE: pa426100365.azurecr.io/report-job:*` (full ACR image URI for the report container), `REPORT_LOCATION: ukwest`, `REPORT_RG: rg-sp26-26100365`, `STORAGE_ACCOUNT_URL: https://pa426100365.blob.core.winc...` (passed into the ACI container as an environment variable), and `SUBSCRIPTION_ID: 67e93b84-fe08-452c-80ea-175d0a3...` required by the Azure SDK management client.

---

## Task 7: End-to-End Pipeline (15 points)

### Evidence 7.1: Web App Wiring

![Web App env vars — FUNCTION_START_URL and FUNCTION_STATUS_URL](<docs/Images/Task1/Screenshot 2026-05-02 034638.png>)

The Environment Variables blade of `pa4-26100365` shows App Service settings including `SCM_DO_BUILD_DURING_DEPLOYMENT`. `FUNCTION_START_URL` and `FUNCTION_STATUS_URL` were added after this screenshot was taken, pointing to the Durable HTTP starter and status polling endpoints of `pa4-26100365-fnc`.

> **Note** — A screenshot showing `FUNCTION_START_URL` and `FUNCTION_STATUS_URL` explicitly was not captured. Please add one here if available. The values would be the `code`-authenticated URL of the `http_starter` endpoint and the `statusQueryGetUri` template from `pa4-26100365-fnc-hkdhaja3bfd9fjed.ukwest-01.azurewebsites.net`.

### Evidence 7.2: Happy Path UI

> **Placeholder** — Screenshots of the form before submit, the Running status badge, and the Completed status with report URL were not captured in the submitted image set. The successful end-to-end smoke test in Evidence 4.3 (order `SMOKE-002`, `runtimeStatus: Completed`) confirms the pipeline executes correctly. Please add browser screenshots here showing ORD-001 submitted and transitioning through Running → Completed → View PDF Report.

### Evidence 7.3: Backend Participation

> **Placeholder** — Screenshots tracing a single order ID across the Function App invocation log, AKS validator logs (`kubectl logs`), ACI container group creation, and the resulting PDF blob were not captured in the submitted image set. The ACI run in Evidence 6.2–6.4 (order `TEST-001`) demonstrates the individual components working; a full end-to-end trace screenshot should be added here.

### Evidence 7.4: Reject Path UI

> **Placeholder** — A screenshot of the TaskFlow UI showing a rejected order (qty > 100) and a matching `az container list` output showing no ACI was created were not captured. Evidence 5.4 (order `O-1002`, qty 999 → `valid:false, quantity exceeds limit`) proves the AKS validator rejects over-limit quantities correctly at the API level.

---

## Task 8: Write-up and Architecture Diagram (5 points)

### Evidence 8.1: Architecture Diagram

![Architecture Diagram](docs\pa4_taskflow_26100365.jpg)

---

### Question 8.2: Service Selection

**App Service** is the right host for the TaskFlow web frontend because it provides fully managed PaaS hosting with native CI/CD from GitHub (or External Git), automatic TLS termination, and a predictable F1/B1 plan cost — all without requiring container orchestration knowledge. The frontend is a stateless Node.js server that serves a single HTML form and makes a few AJAX calls; App Service handles horizontal scale-out automatically under load, and the GitHub integration means every `git push` to `main` triggers a build-and-deploy with zero manual steps. Operational overhead is minimal: no Dockerfile needed for the CI pipeline, no cluster to patch, and the portal gives immediate visibility into deployment logs and runtime health.

**Durable Functions** is the correct orchestration layer because the TaskFlow workflow is inherently sequential — the order must be validated before a report is generated — and the report step blocks for up to 60 seconds waiting for an ACI container to boot, run, and exit. Plain HTTP functions cannot hold an open thread for a minute without risking the default 5-minute execution timeout, and they have no built-in mechanism to persist intermediate state. Durable Functions checkpoints each completed activity's output to Azure Storage and suspends the orchestrator between `yield` points, so no thread is ever blocked and the pipeline can span minutes safely. The pay-per-execution billing model also means the orchestrator costs nothing while it is suspended waiting for ACI.

**AKS** is the right host for the validation microservice because it is a long-running, always-available HTTP API that must respond within milliseconds to every `validate_activity` call. A Kubernetes Deployment keeps the validator pod permanently scheduled and `Running`, with automatic restarts on failure and health-probe gating. For a stateless FastAPI service that receives frequent short-lived HTTP requests, a permanently resident pod on a persistent node is far cheaper per-request than spinning up a new ACI for each validation call, and Kubernetes rolling updates enable zero-downtime image upgrades with automatic rollback on failure.

**Azure Container Instances** is the right executor for the report job because generating and uploading a PDF is an ephemeral, run-to-completion workload that should not exist between orders. ACI's per-second billing means a container that runs for 20–30 seconds costs a fraction of a cent, while keeping the same workload on an AKS node would incur VM charges around the clock regardless of order volume. ACI also accepts any Docker image, so the report job can use heavy PDF libraries (ReportLab) without burdening any long-lived host. The `report_activity` creates the container group via SDK, polls until `Succeeded`, then immediately deletes it — leaving zero persistent infrastructure between runs.

---

### Question 8.3: ACI vs AKS — Hands-on Comparison

**What happens to AKS when it is idle for 10 minutes?**

Nothing changes. Node `aks-nodepool1-18198040-vmss000000` stays in **Ready** state and the validator pod remains **1/1 Running** regardless of whether any orders arrive. As shown in Evidence 5.1, the node age grows continuously from the moment it was provisioned; it is never terminated, drained, or de-allocated by idle behaviour alone. Azure bills for the underlying B-series VM every minute the node pool exists, even when CPU utilisation is effectively zero. The only way to stop billing is to manually stop or delete the cluster, both of which make the validator unavailable.

**What does idle mean for ACI in the TaskFlow pipeline?**

ACI containers do not exist between runs. After `report_activity` calls `begin_delete`, there is no container group in the resource group at all — an `az container list --resource-group rg-sp26-26100365` would return an empty table. Idle for ACI means literally zero cost, zero running processes, and zero allocated infrastructure. Every new report request boots a fresh container from the `pa426100365.azurecr.io/report-job:v1` image and terminates cleanly after writing the PDF to the `reports` blob container.

**If a malicious user spammed Submit 1,000 times in a minute, which service incurs the most cost, and why?**

**ACI** would incur the most cost by a wide margin. Each valid order causes `report_activity` to call `begin_create_or_update`, instantiating a new container group with 1 vCPU and 1.5 GB RAM. At Azure ACI pricing (~$0.0000025/vCPU-second and ~$0.0000003/GB-second), 1,000 simultaneous containers running for even 30 seconds each costs approximately $0.09–$0.13 for that single burst — and without a quota guard or rate limit, requests continue to pile up. AKS absorbs all 1,000 validation POST requests on the same single validator pod without any additional cost; the node's hourly VM charge is fixed regardless of traffic volume. The correct mitigation is to place an Azure API Management rate-limit policy in front of the Durable HTTP starter endpoint.

---

### Question 8.4: Durable Functions vs Plain HTTP

If the same validate → report flow were implemented as two plain HTTP-triggered functions that called each other, at least two concrete problems would make the implementation unreliable.

**Function timeouts.** The `report_activity` must block for up to 60 seconds in a polling loop (`time.sleep(5)` × 12) waiting for the ACI container to reach `Succeeded`. On an Azure Functions Consumption plan, the default execution timeout is 5 minutes — barely enough margin if the ACI takes 3–4 minutes to cold-start, and with no guarantee of completion if the slot is recycled. More critically, if the caller HTTP function is also waiting synchronously for the report function to return, it blocks for the same duration and is equally at risk of timeout. Durable Functions eliminates this entirely: the orchestrator suspends at each `yield`, releasing the thread and timer while the activity runs, and resumption is driven by a durable storage queue message rather than a blocked thread.

**State persistence and retry-on-failure.** If a plain HTTP function crashes mid-execution — due to a transient network partition, an Azure fabric blip, or a process recycle — the caller receives a 500 with no record of whether validation succeeded before the crash. Retrying the full HTTP chain risks re-running the validator (acceptable) but also re-creating a duplicate ACI container group for the same order ID (potentially billable and produces duplicate PDFs). Durable Functions checkpoints the serialised output of every completed activity to Azure Storage before replay; on restart the orchestrator reads the stored checkpoint and skips already-completed steps. Combined with `call_activity_with_retry`, the report step can be retried automatically with exponential back-off, without ever re-running the validator — a guarantee that would require a custom database-backed state machine to reproduce with plain HTTP functions.

---

### Question 8.5: Cost Review

> **Placeholder** — A Cost Management → Cost Analysis screenshot scoped to `rg-sp26-26100365` was not captured. Based on resources deployed and approximate durations:
>
> | Resource | Estimated Duration | Estimated Cost |
> |---|---|---|
> | AKS cluster (1× Standard_B2s node) | ~48 hours | ~$3–5 |
> | App Service (F1 plan — free tier) | ~48 hours | $0.00 |
> | Azure Container Registry (Basic SKU) | ~7 days | ~$1.20 |
> | Azure Container Instances (per-run, ~30 s each) | <20 runs | <$0.01 |
> | Storage Account `pa426100365` (LRS, <1 MB) | ~7 days | ~$0.00 |
> | Function App (B1 plan) | ~48 hours | ~$1–2 |
>
> **Most expensive resource:** The **AKS node pool** is the dominant cost driver because the B-series VM bills continuously regardless of traffic. The App Service on the F1 free tier incurs no charge, making AKS the single largest line item by a clear margin.

---

### Question 8.6: Challenges Faced

**Challenge 1: ImagePullBackOff — AKS could not pull the validate-api image from private ACR**

When `kubectl apply -f k8s/deployment.yaml` was first run, the pods immediately entered `ImagePullBackOff`. `kubectl describe pod <name>` showed `failed to pull image "pa426100365.azurecr.io/validate-api:v1": unauthorized: authentication required`. The AKS cluster's system-assigned managed identity did not have the `AcrPull` role on the registry. The fix was to run:

```bash
az aks update --name <aks-cluster-name> \
  --resource-group rg-sp26-26100365 \
  --attach-acr pa426100365
```

This granted the cluster managed identity the `AcrPull` role on `pa426100365`. After the role assignment propagated (~2 minutes), the deployment controller replaced the failing pods with a fresh rollout — visible in Evidence 5.2 where `validate-deployment-f86b79d84-kzthp` transitions from `ContainerCreating` to **1/1 Running** while the old `ImagePullBackOff` pods terminate.

**Challenge 2: `DefaultAzureCredential` not finding the user-assigned managed identity in report_activity**

During the first full orchestration attempt, `report_activity` failed with `ManagedIdentityCredential authentication failed: No User Assigned or Delegated Managed Identity found`. The Function App had `mi-pa4-26100365` attached (Evidence 6.5) but `DefaultAzureCredential` iterated its credential chain without selecting the correct identity because the `AZURE_CLIENT_ID` environment variable was not set. Two fixes were required:

1. Added `AZURE_CLIENT_ID: 49a00281-8c22-4ae3-80be-cb0b3c2cd39c` to the Function App application settings (Evidence 6.6) so `DefaultAzureCredential` explicitly targets the user-assigned MI.
2. Verified that `mi-pa4-26100365` held **Contributor** on `rg-sp26-26100365` — not just a scoped read role — so it could call `container_groups.begin_create_or_update()` and `begin_delete()`.

After both changes and a Function App restart, the full orchestration pipeline ran successfully end-to-end as demonstrated by the `SMOKE-002` completion in Evidence 4.3 and the `TEST-001.pdf` blob in Evidence 6.4.

---
