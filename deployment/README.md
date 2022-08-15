# Deployment
**deployment** konusuyla ilgili dosyalara buradan erişebilirsiniz.
***
Imperative yöntemle Deployment oluşturma.

```
$ kubectl create deployment "deployment_ismi" --image="image_ismi" --replicas="replika_adeti"

Ör: kubectl create deployment firstdeployment --image=nginx:latest --replicas=2
```
***
Deployment listeleme.

```
$ kubectl get deployment

get deployment -w

```
***
Deployment silme.

```
$ kubectl delete deployment "deployment_ismi"

Ör: kubectl delete deployment firstdeployment
```
***
Deployment içerisindeki imajı güncelleme.

```
$ kubectl set image deployment/"deployment_ismi" "container_ismi"="yeni_imaj"  

Ör: kubectl set image deployment/firstdeployment nginx=httpd:alpine
```
***
Deployment replika sayısını değiştirme "Scaling"

```
$ kubectl scale deployment "deployment_ismi" --replicas="istenilen_replika_adeti"

Ör: kubectl scale deployment firstdeployment --replicas=5
```
***
Deployment yapılan son değişikliğin geri alınması.

```
$ kubectl rollout undo deployment "deployment_ismi"

Ör: kubectl rollout undo deployment firstdeployment
```
***
Deployment yapılan değişikliklerin kaydının tutulması için "--record" opsiyonu kullanılır. 

```
Ör: kubectl set image deployment rolldeployment nginx=httpd:alpine --record=true 
```
***
Deployment yapılan değişikliklerin listelenmesi.

```
$ kubectl rollout history deployment "deployment_ismi"

Ör: kubectl rollout history deployment rolldeployment
```
***
Deployment yapılan değişikliklerin izlenmesi.

```
$ kubectl rollout status deployment "deployment_ismi"

Ör: kubectl rollout status deployment rolldeployment 
```
***
Deployment üstünde yapılan değişikliklerin durdurulması.

```
$ kubectl rollout pause deployment "deployment_ismi"

Ör: ➜ kubectl rollout pause deployment rolldeployment
```
***
Durdurulan rollout'un devam ettirilmesi. 

```
$ kubectl rollout resume deployment "deployment_ismi"

Ör: ➜ kubectl rollout resume deployment rolldeployment
```
***

Note to Self:

DEPLOYMENT

    * Kube scedule pod daki containerda bir sikinti varsa, restart policy ye göre hareket eder. 
    * Eger pod seviyesinde bir sikinti varsa bunu kube schedule cozemez.Pod terminate olur. Bu noktada Node lar önune load balancer koyarak kismen cözulur. Ama podlarla ilgili yeni duzenleme yapmak cok zahmetli olur. Bunu deploymentla duzenleyebiliriz.
    * deployment desire pod ile mevcut podu kiyaslayarak duzenleme yapar. Deployment controlur sistemi devamli kontrol eder. Current ile desire devamli kontrol eder.
    * Sadece desire bolumunde image de yapacagimiz degisiklikle tum sistemi yonetebiliriz. Özellikle yeni versiyonu yayimlarken tek bir komutla sistemi guncelleyebiliriz. Ayni zamanda istedigimizde önceki versiyona dönebiliriz.

    - kubectl get pods 
    - kubectl delete pods podsname (deployment icinde olusturdugumuz pod sileriz fakat desire kacsa ayni podu deployment tekrar olusturur.)
    - k apply -f deploymenttemplate.yaml (deployment olusturma)

ReplicaSet:
    * Replica Set in amaci herhangi bir zamanda calisan kararli bir Replika Pod setini surdurmektedir. Genellikle belirli sayida özdes Pod'un kullanilabilirligini garanti etmek icin kullanilir.
    * Deployment objesi olusturdugunda REplicaset olusturur ve podlari bu Replica set olusturur. Guncelleme yaptigimizda yeni bir ReplicaSet olusturulu ve eski Replica Set eski podlari silerken yeni Replica set Desire a göre Podslari ayarlar.
    * strategy
        type: Rollingupdate

    tek seferde en fazla   maxUnavailable kadar podu sil. Her seferde bu kadar podu silerek desire ulas. Sayi yerine yuzde de girilebilir.

    tek seferde enfazla maxSurge:2 kadar pod calisiyor olsun.

    Boylece sistemde hic kesinti olmadan devam eder.







    - k get replicaset
    - kubectl apply -f deployrolling.yaml
    - kebectl edit deployment rolldeployment
    - kubectl set image deployment rolldeployment nginx=httpd:alpine --record=true

Kubernetes Ag Yapisi:

* K8s kurulumunda pod'lara ip dagitmasi icin bir IP adres araligi (--pod-network-cidr) belirlenir.
* K8s de her popd bu cidr blogundan atanacak uniq ip ye sahip olur.
* Ayni cluster icerisindeki tum podlar varsayilan olarak birbirleriyle herhangi bir kisitlama olmadan ve NAT yani network adress translation olmadan haberlesebilirler.
* Container Network Interface (CNI):
    - Linux containerlarda ag yapilandirmak icin eklentiler yazilabilmesini saglayan spesifikasyonlari belirler.
    - CNI yalnizca containerlarin ag baglantisiyla ve containerlar silindikten sonra ayrilan kaynaklarin  kaldirilmasiyla ilgilenir.
    - CNI pluginlerin listesine ulasmak icin:
        https://github.com/containernetworking/cni
        https://kubernetes.io/docs/concepts/cluster-administration/networking/

