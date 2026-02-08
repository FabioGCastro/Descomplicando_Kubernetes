# Descomplicando o Kubernetes - Day 2: Expert Mode 🚀

Este documento resume os principais conceitos e manifestos apresentados no Day-2, focando na menor unidade do cluster: o **Pod**.

---

## 1. O que é um Pod?
O Pod é a menor unidade dentro de um cluster Kubernetes. 
- **Conceito:** Uma "caixa" que contém um ou mais containers.
- **Compartilhamento:** Containers no mesmo Pod compartilham rede (IP, localhost), namespaces e volumes.
- **Ciclo de Vida:** São efêmeros. Se um Pod morre, um novo é criado (geralmente por um Controller).

---

## 2. Comandos Essenciais (Cheat Sheet)

| Ação | Comando |
| :--- | :--- |
| **Criar/Atualizar** | `kubectl apply -f arquivo.yaml` |
| **Listar Pods** | `kubectl get pods` |
| **Detalhes Técnicos** | `kubectl describe pod <nome>` |
| **Ver Logs** | `kubectl logs -f <nome>` |
| **Acessar Terminal** | `kubectl exec -it <nome> -- bash` |
| **Remover Pod** | `kubectl delete pod <nome>` |

---

## 3. Manifestos YAML Principais

### A. Pod Simples (`pod.yaml`)
Cria um Pod com um único container Nginx.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: giropops
  labels:
    run: giropops
spec:
  containers:
  - name: giropops
    image: nginx
    ports:
    - containerPort: 80 


B. Pod com Limites de Recursos (pod-limitado.yaml)
Define o quanto de CPU e Memória o container tem garantido (requests) e o máximo que pode usar (limits).

apiVersion: v1
kind: Pod
metadata:
  name: pod-limitado
spec:
  containers:
  - name: girus
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "300m"
      limits:
        memory: "128Mi"
        cpu: "500m"

C. Pod Multi-container com Volume EmptyDir (pod-emptydir.yaml)
Demonstra dois containers compartilhando o mesmo volume temporário.

apiVersion: v1
kind: Pod
metadata:
  name: giropops-vols
spec:
  containers:
  - name: container-1
    image: ubuntu
    args: ["sleep", "infinity"]
    volumeMounts:
    - name: armazenamento-comum
      mountPath: /dados
  - name: container-2
    image: alpine
    args: ["sleep", "infinity"]
    volumeMounts:
    - name: armazenamento-comum
      mountPath: /shared-data
  volumes:
  - name: armazenamento-comum
    emptyDir:
      sizeLimit: 256Mi

4. Notas Importantes de "Expert"
Strict Decoding: O Kubernetes diferencia maiúsculas de minúsculas. O erro comum sizelimit deve ser sempre corrigido para sizeLimit.

Unidades de CPU: 500m (milliCPUs) é o mesmo que 0.5. Significa metade de um núcleo de CPU.

Unidades de Memória: Use Mi (Mebibytes) e Gi (Gibibytes). O Kubernetes prefere o sistema binário.

Exec vs Attach:

attach: Conecta à console do processo principal.

exec: Cria um novo processo (como o bash ou sh) dentro do container existente.
