# What is Kustomize?
- Kustomize is a **Kubernetes native configuration management tool** that allows to customize Kubernetes YAML manifests without actually modifying the original files.
- It is similar to Ansible which is a configuration management tool for OS (Linux).
- Instead of duplicating the YAML manifests for each environment (DEV, STAGE, PROD, etc.), Kustomize allow us to define overlays (small code snippets) on top of a common base.
- Kustomize follows a **template-free** approach, meaning it does not use any placeholders just like we have in HELM, instead it directly works with the standard YAML manifet files.

---

# Why Kustomize?
- Consider a scenario:
    - Let's say we have to create an <code>nginx-deployment</code> for three environments - DEV, STAGE and PROD.
    - The condition is, we need to have 2 replicas for DEV, 3 for STAGE and 5 for PROD.
    - Now if we don't use Kustomize, we will have to write three <code>nginx-deployment.yaml</code> manifest for each of the available environment.
    - Here, the entire nginx-configuration will be same, only the difference will be the number of replicas.
    - This is feasible and manageble, but if we have 30-40 microservices/manifests, then management of each of them is error prone.
    - To tackle this, we can make use of Kustomize.
    - With Kustomize will follow the below structure:
        - Create a <code>base</code> folder consisting of <code>nginx-deployment.yaml</code> manifest file (or any other files that need to be managed by Kustomize) and <code>Kustomization.yaml</code>.
        - <code>Kustomization.yaml</code> will include the resources that will be managed by Kustomize.
        - Create an <code>overlays</code> folder consisting of three different folders based on the environments (DEV, STAGE and PROD).
        - Each of this environment folder will consists of <code>Kustomization.yaml<code> file which include the path of <code>base</code> folder and the patches which will be used to customize the parent YAML manifests.

---

# Difference with HELM
- Kustomize is a **Kubernetes native configuration management tool** and is used for that purpose only, plus it is easy to understand and implement.
- HELM can also be used as a configuration management tool just like Kustomize, but HELM comes with multiple other feature.
    - HELM is basically a **Kubernetes Package Manager** which is use to install, upgrade, remove and publish the packages (HELM Charts) for others use.
    - HELM works with template, does not directly manipulate the parent YAML manifests.
    - HELM is a bit hard to understand and manage for smaller projects.
- We can go with any of the Kustomize or HELM depending on the requirement and need.

---

# Transformers in Kustomize
- Transformers in Kustomize are the **build-in functions** that automatically modifies the Kubernetes YAML manifests based on the configuration defined in <code>Kustomization.yaml</code> file.
- For example, transformers can:
    - Add labels or annotations to all the resources.
    - Update image tags in Deployments.
    - Modify namespace fields across all objects.
    - Set replica counts for Deployments or StatefulSets.
    - Adjust resource names (prefix, suffix) to avoid conflicts.
- There are total around 10-15 transformers available by default to use by Kustomize.

## Example
```yaml
commonLabels:
    app: my-app
    env: staging

namePrefix: dev-

nameSuffix: -prod
```

---

# Patches
- Patches in Kustomize are the partial/small YAML snippets that modifies the specific portion or fields of Kubernetes YAML manifest without actually coping or rewriting the whole file.
- Patches allow to ovverride certain parts (like replica counts, environment variables, image tags, etc.) in a clean, targeted way.

## Types of Patches
### Strategic Merge Patch
- Most widely and easy to use.
- Here, we create partial YAML snippet that specifies only the fields we want to update.
- Example:
```yaml
# patch-replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: my-app
spec:
    replicas: 5
```
In kustomization.yaml:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
    - ../base

patches:
    - path: patch-replicas.yaml
```

### JSON 9602 Patch
- More fine-grained and powerful.
- Here we define a JSON list of operations (add, remove, replace, etc.)
- Useful for very specific updates or when Strategic Merge doesn’t behave the way we want.
- Example:
```yaml
- op: add
    path: /metadata/labels/app-type
    value: web-serve
```
In kustomization.yaml:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
    - ../base

patches:
    - path: add-annotation.yaml
        target:
            group: apps
            version: v1
            kind: Deployment
            name: web-app
```

---

# Generators
- In Kustomize, Generators are special tools that generate **Kubernetes resources (like ConfigMaps and Secrets) from files, literals or environment variables - instead of manually writing the YAML for them**.

## Types of Generators
### ConfigMap Generator
- It creates a **ConfigMap** from files, literals, or environment variables.
```yaml
configMapGenerator:
  - name: app-config
    literals:
        - ENV=production
        - LOG_LEVEL=debug

configMapGenerator:
  - name: app-config
    files:
        - config.properties
```

### SecretGenerator
- It creates a **Secret** from files, literals, or environment variables.
```yaml
secretGenerator:
  - name: db-secret
    literals:
      - username=admin
      - password=secret123

secretGenerator:
  - name: tls-secret
    files:
      - tls.crt
      - tls.key
```
