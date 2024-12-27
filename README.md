# terraform_scripts

Integrating Atlantis with Git to identify and automate Terraform changes involves setting up Atlantis as a pull request automation tool. Here's a step-by-step guide:

---

### **1. Deploy Atlantis**
Atlantis can be deployed on your infrastructure (e.g., on Kubernetes, ECS, or directly on a virtual machine).

#### Option 1: Deploy via Docker
Run the Atlantis container locally or on a server:
```bash
docker run -d \
  --name atlantis \
  -p 4141:4141 \
  -v /path/to/config:/etc/atlantis \
  runatlantis/atlantis
```

#### Option 2: Deploy via Helm (for Kubernetes)
If you use Kubernetes, you can deploy Atlantis using Helm:
```bash
helm repo add runatlantis https://runatlantis.github.io/helm-charts
helm install atlantis runatlantis/atlantis -f values.yaml
```

---

### **2. Configure Git Repository**
Atlantis needs to integrate with your Git repository (GitHub, GitLab, Bitbucket).

#### Set up Webhooks
1. Go to your Git repository's settings.
2. Add a webhook pointing to your Atlantis server URL: `http://<atlantis-server>/events`.
   - Content type: `application/json`
   - Events: Pull request events (e.g., Opened, Updated, Closed, Merged).

#### Add Atlantis YAML Config
Add a configuration file (`atlantis.yaml`) to the root of your Terraform repository:
```yaml
version: 3
projects:
  - dir: .
    workspace: default
    autoplan:
      when_modified: ["*.tf", "*.tfvars", "modules/**.tf"]
      enabled: true
    terraform_version: 1.5.5
```

---

### **3. Set up Atlantis Server**
Update the Atlantis server configuration (`server.yaml`) with details about your Git provider:

#### Example for GitHub:
```yaml
atlantis-url: http://<your-atlantis-server>
gh-user: <github-username>
gh-token: <personal-access-token>
repo-allowlist: github.com/<org>/*
```

#### Example for GitLab:
```yaml
atlantis-url: http://<your-atlantis-server>
gitlab-user: <gitlab-username>
gitlab-token: <personal-access-token>
repo-allowlist: gitlab.com/<org>/*
```

---

### **4. Open a Pull Request**
1. Push changes to your Terraform files to a feature branch.
2. Open a pull request (PR).
3. Atlantis will:
   - Detect changes in `.tf` files.
   - Comment on the PR with the `terraform plan` output.
   - Allow you to run `atlantis apply` by commenting `/atlantis apply` on the PR.

---

### **5. Automation with Terraform Workflows**
Atlantis automatically runs workflows on PRs based on your configuration. Common workflows include:

- **Plan**: Automatically runs `terraform plan` when a PR is opened or updated.
- **Apply**: Runs `terraform apply` when `/atlantis apply` is commented on the PR.

---

### **6. (Optional) Add CI/CD Integration**
For tighter integration:
- Trigger Atlantis via CI/CD pipelines when changes are detected.
- Use pipeline tools (e.g., Jenkins, GitHub Actions) to verify Atlantis workflows.

---
