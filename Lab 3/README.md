# DevOps Lab 3: Monitoring, Logging & DevSecOps

**Student:** Meenakshi | **Roll No.:** 2301730200 | **Email:** meenakship0805@gmail.com
**Course:** DevOps and Automation (ENSP461)
**Environment:** Ubuntu 24.04 LTS, Docker, Ansible, Terraform, ELK, Prometheus, Grafana, GitHub Actions

A Flask app with Prometheus metrics, JSON logs and feature flags, deployed with Ansible and Terraform, monitored with Prometheus/Grafana, logged to ELK, and checked by security gates on every push.

| Path | Purpose | Task |
|---|---|---|
| `app/` | Flask app: metrics at `/metrics`, JSON logs, feature flags | all |
| `ansible/` | Playbook + `webapp` role: install Docker, build and run the app | 1 |
| `terraform/` | Docker network, N app containers, nginx load balancer | 2 |
| `logging/` | Elasticsearch, Logstash, Kibana; app logs sent with the GELF driver | 3 |
| `security/Jenkinsfile`, `.github/workflows/devsecops.yml` | gitleaks, bandit, pip-audit, trivy config, trivy image | 4 |
| `monitoring/` | Prometheus, Grafana, node-exporter, alert rules, `load_test.sh` | 5 |
| `tests/` | 13 pytest tests for feature flags, metrics and endpoints | 6 |

## Tests

```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -r app/requirements.txt pytest
pytest -v
```

## Task 1: Ansible

```bash
sudo apt install -y ansible
ansible-galaxy collection install community.docker
cd ansible
ansible-playbook -i inventory.ini site.yml --syntax-check
ansible-playbook -i inventory.ini site.yml -K      # run twice; second run shows changed=0
curl localhost:5001/health
```

## Task 2: Terraform

```bash
sudo snap install terraform --classic
cd terraform
terraform init && terraform validate && terraform plan
terraform apply -auto-approve                      # 2 app containers + nginx on :8090
terraform apply -auto-approve -var replicas=4      # scale out
terraform destroy -auto-approve
```

## Task 3: ELK logging

```bash
sudo sysctl -w vm.max_map_count=262144
cd logging && docker compose up -d --build
for i in $(seq 50); do curl -s localhost:5002/ >/dev/null; curl -s localhost:5002/error >/dev/null; done
# Kibana: http://localhost:5601 -> Data View "lab3-logs-*" -> Discover
```

## Task 4: DevSecOps

Runs automatically on every push (see the **Actions** tab). Locally:

```bash
pip install bandit pip-audit
bandit -r app -ll
pip-audit -r app/requirements.txt
```

## Task 5: Monitoring

```bash
cd monitoring && docker compose up -d --build
bash load_test.sh
# Prometheus: http://localhost:9090/targets   Grafana: http://localhost:3000 (admin/admin)
```

## Task 6: Feature flags

`FEATURE_NEW_UI` and `FEATURE_DISCOUNT` (`true`, `1`, `yes`, `on` = on).

```bash
docker build -t lab3-app app
docker run -d -p 5003:5000 -e FEATURE_NEW_UI=true lab3-app && curl localhost:5003
```
