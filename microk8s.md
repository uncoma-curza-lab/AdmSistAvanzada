# MICROK8S Instalación y Configuración Inicial en Ubuntu Server 22.04

---

## Configuración del sistema

```bash
sudo nano /etc/issue
sudo nano /etc/hosts
sudo hostnamectl set-hostname microk
sudo sh -c "apt update; apt upgrade -y; apt autoclean -y; apt autoremove -y; reboot"
```

---

## Instalación
Detalles en la página oficial de [MicroK8s](https://microk8s.io/#install-microk8s).

```bash
sudo snap install microk8s --classic
sudo usermod -a -G microk8s $USER
mkdir .kube
sudo chown -R $USER ~/.kube
newgrp microk8s
microk8s status --wait-ready
microk8s kubectl get all -A
```

---

## Alias para kubectl

```bash
nano .bash_aliases
    alias kubectl='microk8s kubectl'
source .bashrc
kubectl get all -A
```

---

## Crear un namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: prod
  labels:
    app: MyBigWebApp
```

y

```bash
kubectl apply -f ns.yaml
```

---

## lanzar un pod por línea de comandos

```bash
kubectl run pod-example --image=nginx:stable-alpine --port=80 -n prod
```

### por archivo YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
  namespace: prod
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
```

---

## Logs con sidecar

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ejemplo
spec:
  containers:
  - name: ejemplo
    image: busybox
    args:
      - /bin/sh
      - -c
      - >
        while true;
        do
        echo "$(date)" >> /var/log/ejemplo.log;
        sleep 1;
        done
    volumeMounts:
      - name: varlog
        mountPath: /var/log
  - name: sidecar
    image: busybox
    args: [/bin/sh, -c, 'tail -f /var/log/ejemplo.log']
    volumeMounts:
      - name: varlog
        mountPath: /var/log
  volumes:
    - name: varlog
      emptyDir: {}
```

## Logs con kibana

- Habilitar un "Ingress": [Addon Ingress](https://microk8s.io/docs/addon-ingress)
- Configurar un “Ingress” y accedar al mismo (habrá que dar de alta la URL en el servidor DNS) [Kibana](http://kibana.microk.local)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: kube-system
  name: kibana
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: "kibana.microk.local"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kibana-logging
            port:
              number: 5601
```
