# DevSecOps Workshop — Parte 2 (Práctico)

Este repo acompaña el taller práctico: pipeline en GitHub + Kubernetes local (kind/minikube), demostrando herramientas de seguridad con escenarios **break/fix**.

:P

:P

## Estructura
```
apps/
  00-safe-baseline/       # app sin fallos (Node.js)
  10-secrets-leak/        # secretos filtrados (.env + hardcodeado)
  20-sast-bugs/           # Python/Flask con patrones inseguros (SAST)
  30-sca-vuln/            # deps vulnerables (Node.js)
  40-iac-misconfig/       # manifiestos K8s inseguros
  50-container-bad/       # Dockerfile inseguro
  60-dast-target/         # endpoints inseguros (DAST)
charts/demo-app/          # Helm chart de despliegue
k8s/                      # Manifiestos sueltos (inseguros)
.github/workflows/        # pipeline CI/CD
.semgrep/                 # regla de ejemplo
gitleaks.toml             # config opcional para secretos
```

## Requisitos
- Docker o Podman
- kind **o** minikube, kubectl, Helm
- GitHub (Actions habilitado). Recomendado: **self-hosted runner** en el mismo host que el clúster.

## Uso rápido
1. Crea un clúster local (ej. kind):
   ```bash
   cat > kind.yaml <<'EOF'
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
     - role: control-plane
       extraPortMappings:
         - containerPort: 30080
           hostPort: 30080
   EOF
   kind create cluster --name devsecops --config kind.yaml --wait 120s
   ```

2. (Opcional) Instala self-hosted runner: GitHub → Settings → Actions → Runners.

3. Sube este repo a GitHub y habilita Actions. El workflow construye por defecto `apps/00-safe-baseline`.
   - Para probar escenarios, edita `APP_DIR` en variables del workflow o modifica el job **build** para apuntar a otra carpeta (ej. `apps/10-secrets-leak`).

4. Despliegue: el pipeline cargará la imagen local al clúster kind y hará `helm upgrade --install demo charts/demo-app`. Accede en `http://localhost:30080`.

## Escenarios sugeridos
- **Secrets**: usa `apps/10-secrets-leak` → gitleaks debe fallar → corrige (mueve secretos a GitHub Secrets, limpia el repo) → vuelve a correr.
- **SAST**: `apps/20-sast-bugs` → Semgrep marca `yaml.load` inseguro → usa `yaml.safe_load`.
- **SCA (deps)**: `apps/30-sca-vuln` → Trivy fs detecta CVEs → actualiza dependencias.
- **IaC**: `apps/40-iac-misconfig` → Checkov debe fallar → agrega `securityContext`/`resources`.
- **Imagen**: `apps/50-container-bad` → Trivy image detecta HIGH/CRITICAL → mejora Dockerfile.
- **DAST**: `apps/60-dast-target` → ZAP detecta cabeceras faltantes → añade Helmet u otros.

> **Nota**: El pipeline está pensado para **self-hosted** runners para acceder al clúster local. Si usas hosted runners de GitHub, necesitarás publicar la imagen (GHCR) y exponer acceso a tu clúster o simular despliegue.
