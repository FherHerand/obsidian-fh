---
tags:
  - openshift
  - kubernetes
  - troubleshooting
  - sysadmin
  - runbook
  - cheatsheet
  - linux
aliases:
  - OpenShift CheatSheet
---

# Revisión, Mantenimiento y Troubleshooting

Guía técnica para tareas de diagnóstico de pods, inspección de recursos en nodos, ciclo de vida del cluster, certificados y administración de ETCD.

> [!link] Notas Relacionadas
> - Almacenamiento distribuido: [[Rook-Ceph - Operación y Solución de Problemas]]
> - Middleware y aplicaciones: [[Cloud Pak y ACE - Diagnóstico y Mantenimiento]]

---

## 1. Diagnóstico y Gestión de Pods

### 1.1. Identificar y Contar Pods en Estado `NotReady`
Comandos para detectar pods cuyos contenedores no están listos (incluso si la fase general indica `Running`):

```bash
# Listar pods NotReady en todos los namespaces
oc get pods --all-namespaces | grep -Pv '\s+([1-9]+[\d]*)\/\1\s+' | grep -v Completed

oc get pods -A --field-selector=status.phase!=Succeeded | awk '{split($3, a, "/"); if (a[1] != a[2] || $4 != "Running") print $0}'

# Monitoreo continuo (cada 5 segundos)
watch -n5 "oc get pods --all-namespaces | grep -Pv '\s+([1-9]+[\d]*)\/\1\s+' | grep -v Completed"

# Contar cantidad total de pods en NotReady
watch -n5 "oc get pods --all-namespaces | grep -Pv '\s+([1-9]+[\d]*)\/\1\s+' | grep -v Completed | wc -l"
```

### 1.2. Eliminación de Pods Problemáticos
```bash
# Eliminar pods NotReady del namespace actual
oc delete pods $(oc get pods --no-headers | grep -Pv '\s+([1-9]+[\d]*)\/\1\s+' | grep -v Completed | awk '{print $1}')

# Eliminar pods NotReady en un namespace específico
NAMESPACE="mi-namespace"
oc delete pods -n $NAMESPACE $(oc get pods -n $NAMESPACE --no-headers | grep -Pv '\s+([1-9]+[\d]*)\/\1\s+' | grep -v Completed | awk '{print $1}')

# Eliminar pods con Error (excluyendo Completed, Running, Init y Terminating) en TODOS los namespaces
oc get pods --all-namespaces -o wide --no-headers | grep -Pv '\s+([1-9]+[\d]*)\/\1\s+' | grep -v "Completed\|Running\|Init\|Terminating" | awk '{print $1 " " $2}' | while read ns pod; do
  oc delete pod -n $ns $pod
done
```

### 1.3. Métricas de Reinicios y Trazas
```bash
# Listar pods ordenados por contador de reinicios (restarts)
oc get pods --all-namespaces --sort-by='.status.containerStatuses[0].restartCount'

# Logs de pods del Core de OpenShift
oc logs -n openshift-console -l app=console -f
oc logs -n openshift-authentication -l app=oauth-openshift -f

# Recorrer e imprimir logs de todos los pods en un namespace específico
NAMESPACE="apic"
PODS=$(oc get pods -n $NAMESPACE --no-headers | grep -v Completed | awk '{print $1}')
for pod in $PODS; do
  echo "=== POD: $pod ==="
  oc logs -n $NAMESPACE --all-containers $pod
done
```

---

## 2. Inspección y Consumo de Recursos en Nodos

### 2.1. Visualizar Requests y Limits
```bash
# Vista rápida por nodo
NODES=$(oc get nodes -o jsonpath='{.items[*].metadata.name}')
for n in $NODES; do
  echo "---------------- NODO: $n ----------------"
  oc describe node $n | grep "Resource" -A 3
done

# Vista tabular formateada (CPU y Memoria)
printf "  node\t\t\t\t\tcpu requests\t\tcpu limits\t\tmemory requests\t\tmemory limits\n"
printf "  --------\t\t\t\t--------\t\t--------\t\t--------\t\t--------\n"
for n in $(kubectl get nodes -o jsonpath='{.items[*].metadata.name}'); do
  export node=$n
  kubectl describe node $n | grep "Resource" -A 3 | xargs -n16 sh -c 'printf "  %-30s\t" $node && printf "%-7s " $7 && echo -n "$8" && printf "\t\t" && printf "%-7s " $9 && echo -n "${10}" && printf "\t\t" && printf "%-7s " ${12} && echo -n "${13}" && printf "\t\t" && printf "%-7s " ${14} && echo "${15}"'
done
```

### 2.2. Comparativa Completa: Requests, Usable (Allocatable) y Uso Real
```bash
NODES=$(oc get nodes -o jsonpath='{.items[*].metadata.name}')
for n in $NODES; do
  echo "==================== NODO: $n ===================="
  printf "  Total Requests y Limits:\n"
  oc describe node $n | grep "Resource" -A 3 | xargs -n16 sh -c 'echo "    CPU requests: $7 $8" && echo "    CPU limits: $9 ${10}" && echo "    Memory requests: ${12} ${13}" && echo "    Memory limits: ${14} ${15}"'

  printf "\n  Total Usable (Allocatable):\n"
  oc get node $n --no-headers -o custom-columns=NODE:metadata.name,Allocated-CPU:status.allocatable.cpu,Allocated-RAM:status.allocatable.memory,Allocated-Storage:status.allocatable.ephemeral-storage \
  | xargs -n4 sh -c 'printf "    CPU: $1\n    Memory: $2\n    Storage: $3\n"'

  printf "\n  Total Uso Actual:\n"
  oc adm top nodes | grep $n | xargs -n5 sh -c 'echo "    CPU: $1 ($2)" && echo "    Memory: $3 ($4)"'
done
```

---

## 3. Redes y Validación de Proxy

### 3.1. Estado de Proxy del Cluster
```bash
# Inspección de recurso Proxy global
oc get proxy/cluster -o jsonpath='{.status}'

# Inspección a nivel de nodo (SSH / Bastion)
sudo systemctl show kubelet | grep -i proxy
cat /etc/mco/proxy.env
```

### 3.2. Script de Validación de Salida a Internet por Nodos
```bash
NODES=$(oc get nodes --no-headers -o wide | awk '{print $6}')
PROXY_URL="<ip>:<puerto>"
HTTPS_HTTP_URLS="quay.io cdn01.quay.io cdn02.quay.io cdn03.quay.io access.redhat.com registry.connect.redhat.com registry.redhat.io api.openshift.com mirror.openshift.com storage.googleapis.com/openshift-release rhcos.mirror.openshift.com"
HTTPS_URLS_ONLY="cp.icr.io icr.io registry-1.docker.io"

for n in $NODES; do
  echo "----------------- NODE: $n -----------------"
  for u in $HTTPS_HTTP_URLS; do
    echo "URL: $u"
    ssh core@$n "echo -n '443: ' && curl -s -x $PROXY_URL -I https://$u | head -n1 && echo -n '80:  ' && curl -s -x $PROXY_URL -I http://$u | head -n1"
  done
  for u in $HTTPS_URLS_ONLY; do
    echo "URL: $u"
    ssh core@$n "echo -n '443: ' && curl -s -x $PROXY_URL -I https://$u | head -n1"
  done
done
```

---

## 4. Mantenimiento y Actualizaciones del Cluster

### 4.1. Trazas del Daemon de MCO y Bloqueos de Drenado (Drain)
```bash
# Localizar nodo en mantenimiento o SchedulingDisabled
NODE=$(oc get nodes | grep 'SchedulingDisabled\|NotReady' -m1 | awk '{print $1}')
echo "Analizando nodo: $NODE"

# Trazas del daemon de Machine Config en el nodo
POD_DAEMON=$(oc get pod -n openshift-machine-config-operator -o wide | grep $NODE | grep daemon | awk '{print $1}')
oc logs -n openshift-machine-config-operator -c machine-config-daemon $POD_DAEMON -f

# Trazas del controlador de MCO
POD_CTRL=$(oc get pod -n openshift-machine-config-operator | grep controller | awk '{print $1}')
oc logs -n openshift-machine-config-operator $POD_CTRL -f

# Si un PodDisruptionBudget (PDB) detiene el drenado del nodo:
oc patch pdb <pdb_name> -n <pdb_namespace> --type=merge -p '{"spec":{"minAvailable": 0}}'

# Eliminar manualmente pods pegados en un nodo específico
NAMESPACE="mi-namespace"
PODS=$(oc get pod -n $NAMESPACE -o wide | grep $NODE | awk '{print $1}')
oc delete pods -n $NAMESPACE $PODS
```

### 4.2. Recuperar Nodo con Fallo "Unexpected on-disk state"
Si el daemon del nodo se marca degradado tras un reinicio incompleto:
```bash
# Ejecutar dentro del nodo afectado:
rm /etc/machine-config-daemon/currentconfig
sudo touch /run/machine-config-daemon-force
```

### 4.3. Liberación de Imágenes por Disk Pressure en Nodos
```bash
# Limpiar contenedores parados, pods muertos e imágenes huérfanas en todos los nodos
NODES=$(oc get nodes -o wide --no-headers | awk '{print $6}')
for n in $NODES; do
  ssh core@$n "sudo crictl rm \$(sudo crictl ps -q --state Exited)"
  ssh core@$n "sudo crictl rmp \$(sudo crictl pods -q -s NotReady)"
  ssh core@$n "sudo crictl rmi --prune"
done
```

---

## 5. Secretos, Pull-Secrets y Finalizers

### 5.1. Gestión del Pull-Secret Global
```bash
# Extraer pull-secret actual a texto plano / JSON
oc get secret pull-secret -n openshift-config --template='{{index .data ".dockerconfigjson" | base64decode}}' > pull-secret.txt
cat pull-secret.txt | jq . > pull-secret.json

# Convertir de JSON con formato a formato de una línea
cat pull-secret.json | jq . -c > pull-secret-min.json

# Actualizar el secret en el cluster
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=/path/to/pull-secret.json
```

### 5.2. Desbloquear Namespaces y Recursos con Finalizers
```bash
# Listar recursos con finalizers bloqueando un namespace
oc api-resources --verbs=list --namespaced -o name | xargs -n 1 oc get --show-kind --ignore-not-found -n <namespace>

# Vaciar finalizers en un tipo de recurso (CRD) específico
CRD="<nombre_del_recurso>"
for res in $(oc get $CRD -o jsonpath='{.items[*].metadata.name}'); do
  oc get $CRD "$res" -o name | xargs -I {} oc patch {} --type merge -p '{"metadata":{"finalizers": []}}'
done
```

---

## 6. Certificados, ETCD y Recuperación de Emergencia

### 6.1. Inspección de Vigencia de Certificados
```bash
# Validar certificado del Endpoint API
echo | openssl s_client -connect api.<cluster-name>.<base-domain>:6443 2>/dev/null | openssl x509 -noout -dates -subject
echo | openssl s_client -connect api-int.<cluster-name>.<base-domain>:6443 2>/dev/null | openssl x509 -noout -dates -subject

# Inspeccionar certificados dentro de Secrets o ConfigMaps
oc get secret <secret-name> -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -issuer -dates -subject
oc get configmap <cm-name> -o jsonpath='{.data.ca\.crt}' | openssl x509 -noout -issuer -dates -subject

# Extraer fechas de certificados de Ignition (Master / Worker)
oc extract -n openshift-machine-api secret/master-user-data --keys=userData --to=- | jq -r '.ignition.security.tls.certificateAuthorities[0].source' | cut -d, -f2 | base64 -d | openssl x509 -dates -noout
```

### 6.2. Salud del Cluster ETCD
```bash
# Identificar pod etcd activo
ETCD=$(oc get pods -n openshift-etcd -l app=etcd --field-selector="status.phase==Running" -o jsonpath="{.items[0].metadata.name}")

# Validar miembros, salud y estado
oc exec -n openshift-etcd $ETCD -c etcdctl -- etcdctl member list -w table
oc exec -n openshift-etcd $ETCD -c etcdctl -- etcdctl endpoint health -w table
oc exec -n openshift-etcd $ETCD -c etcdctl -- etcdctl endpoint status -w table
```

### 6.3. Kubeconfig de Respaldo Local en Nodos Master
Si la API externa no responde o los certificados de balanceador expiraron, ingresar por SSH a un Master:
```bash
sudo su
# Usar el kubeconfig que apunta al balanceador interno o local
export KUBECONFIG=/etc/kubernetes/static-pod-resources/kube-apiserver-certs/secrets/node-kubeconfigs/lb-int.kubeconfig
oc get nodes
```