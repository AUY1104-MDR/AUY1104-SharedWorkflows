# AUY1104-SharedWorkflows

Repositorio central de la Evaluación Sumativa 2 (AUY1104 — Ciclo de Vida del Software II).

Concentra dos cosas:
1. **Workflows reutilizables** de GitHub Actions usados por los demás repos del alumno.
2. **Infraestructura como código (Terraform)** para provisionar la EC2 + k3s en AWS Learner Lab.

No despliega aplicaciones por sí mismo; sirve como librería de pipelines y como motor de provisión.

## Contenido

| Ruta | Propósito |
|---|---|
| `.github/workflows/deploy-api.yaml` | Workflow reutilizable (`workflow_call`). Hace `npm install` + `npm test`, build + push a Docker Hub, y deploy por SSH al nodo k3s. Lo invoca `AUY1104-SharedClient/.github/workflows/client.yaml`. |
| `.github/workflows/validate-api.yaml` | Workflow reutilizable de validación CI (tests + build Docker sin publicar). Disponible para ramas de trabajo. |
| `.github/workflows/ea2-provision-k8s-sandbox.yaml` | Workflow reutilizable que aplica Terraform y deja k3s instalado en una EC2. Usa secrets AWS + SSH. |
| `.github/workflows/ea2-lab-dispatch-main.yml` | Workflow disparador (`workflow_dispatch`) que llama a `ea2-provision-k8s-sandbox.yaml` desde este mismo repo. |
| `infra/ea2-sandbox-vm/` | Módulo Terraform: VPC default + Security Group + EC2 Ubuntu 22.04 + key pair. |
| `script/setup-kubectl-k3s-lab.sh` | Script local para configurar `kubectl` apuntando al k3s recién provisionado. |

## ¿Quién usa qué?

```
AUY1104-SharedClient/client.yaml
        │ uses
        ▼
AUY1104-SharedWorkflows/.github/workflows/deploy-api.yaml@main   ← workflow reutilizable

AUY1104-SharedWorkflows/.github/workflows/ea2-lab-dispatch-main.yml
        │ uses
        ▼
AUY1104-SharedWorkflows/.github/workflows/ea2-provision-k8s-sandbox.yaml@main
        │
        ▼
infra/ea2-sandbox-vm (Terraform → EC2 + k3s)
```

## Secrets/Variables esperados

A nivel de organización:
- `DOCKER_USERNAME`, `DOCKER_PASSWORD`
- `EA2_SSH_PRIVATE_KEY`
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`

A nivel de repo cliente (variables):
- `K3S_SERVER_PUBLIC_IP`

## Provisión del cluster (flujo manual)

1. Ir a la pestaña **Actions** de este repo.
2. Ejecutar `Ej K8S desde alumno` (`ea2-lab-dispatch-main.yml`) con los parámetros por defecto.
3. Al finalizar, copiar la IP pública del resumen al campo `vars.K3S_SERVER_PUBLIC_IP` de los repos `AUY1104-SharedApache`, `AUY1104-SharedClient` y `AUY1104-SharedNginx`.
4. Para destruir: `terraform destroy` desde el módulo, o terminar la EC2 desde AWS.


