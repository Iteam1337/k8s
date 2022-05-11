# Så här konfigurerar du Kubernetes på Safesprings molntjänst i Sverige

## Sätt upp servrarna i Safespring

Det här steget kan man göra med en anann leverantör

1. Beställ konto på Safespring. Hälsa från oss så blir vi glada.
2. Skapa en ny security group som du döper tex till 'public' med följande portar öppna:

![image](https://user-images.githubusercontent.com/395843/153294252-9f639c32-6c53-4438-8324-170b49b65bca.png)

3. Launch Instance - Välj Ubuntu 20.04 och en nod med lokal lagring, t ex lb.large.1d. Skapa tre stycken och glöm inte att att välja din security group.
![image](https://user-images.githubusercontent.com/395843/153294835-3be4f2e7-67f8-4188-88c1-1d2f5a089152.png)

## Installera Microk8s

1. `ssh ubuntu@ip-adressen till dina noder`
2. `sudo snap install microk8s --classic`
3. Lägg till din användare i microk8s gruppen så att du slipper köra sudo för varje kommando:

    ```bash
    sudo usermod -a -G microk8s ubuntu
    sudo chown -f -R ubuntu ~/.kube
    newgrp microk8s
    ```

4. Från en av noderna: 

    ```bash
    microk8s add-node
    ```

Nu får du instruktioner att köra på en av de andra noderna. T ex `microk8s join xxx:25000`
Här kan du välja om du ska ha en master-nod och flera worker noder. Vi brukar välja att varje nod är både worker och master, välj då översta alternativet.

5. Lägg till de add-ons du vill använda:

    ```bash
    microk8s enable dns ingress helm3 registry storage
    ```

6. Lägg till externa IP som IP.100 i denna så att du kan nå klustret utifrån utan varningar. Upprepa för varje nod.

    ```bash
    sudo vim /var/snap/microk8s/current/certs/csr.conf.template
    sudo /snap/bin/microk8s refresh-certs
    ```

7. Nu är du redo att ansluta till dina noder. Så här hämtar du config till servrarna:

    ```bash
    microk8s config
    ```

   Lägg in detta i din lokala ~/kube/config

8. Så ska du nu kunna komma åt noderna:

    ```bash
    kubectl get nodes
    ```

## Cert manager

För att aktivera automatisk TLS i ditt kluster så kör du cert-manager. Så här installerar du cert-manager:

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.1/cert-manager.yaml
```

OBS! Redigera emailadress i [k8s/letsencrypt.yaml](k8s/letsencrypt.yaml)

```bash
kubectl apply -f k8s/letsencrypt.yaml
```

## Testa allt tillsammans

```bash
kubectl apply -f k8s/web.yaml
```


## Longhorn distribuerad lagring

För att köra något som kräver lagring på noderna så behöver du först installera en distribuerad lagring. Vi använder ofta Longhorn men det finns olika. Longhorn har inbyggd backup till S3 vilket är väldigt praktiskt. Så här installerar du Longhorn:

```bash
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --set csi.kubeletRootDir=/var/snap/microk8s/common/var/lib/kubelet --set defaultSettings.defaultDataPath=/data
```

## Skaffold



## IPv6

För att aktivera IPv6 så behöver man fortfarande pilla en del med interna filer på klustret men följer man den här guiden (hoppa över brandväggsreglerna) så fungerar det:
https://discuss.kubernetes.io/t/microk8s-ipv6-dualstack-how-to/14507

