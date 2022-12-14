# Pod
**pod** konusuyla ilgili dosyalara buradan erişebilirsiniz.
***
Imperative yöntemle pod oluşturma.

```
$ kubectl run "pod_ismi" --image="image_ismi" --restart=Never

Ör: kubectl run firstpod --image=nginx --restart=Never
```
***
Bir objenin ayrıntılı özelliklerini görmek. 
```
$ kubectl describe "obje_tipi" "obje_ismi"

Ör: kubectl describe pods firstpod
```
k get pods -o wide (ayrintili bilgi)
k describe pods firstpod (firstpod un tum bilgilerini alabiliri)


***
Bir pod objesinin loglarını görüntüleme. _(-f opsiyonu çıktıya yapışmanızı ve anlık olarak üretilen logları görmenizi sağlar)_
```
$ kubectl logs "pod_ismi"

Ör: kubectl logs firstpod
Ör: kubectl logs -f firstpod (canli olarak izlemeyi saglar,shell loglara yapisir ve degisimleri gösterir, -f follow)
```
***
Pod'da komut çalıştırma. _(Eğer pod içerisinde birden fazla container varsa **-c "container_ismi"** opsiyonu ile komutun çalıştırılması istenilen container belirtilebilir)_
```
$ kubectl exec "pod_ismi" -- "komut"

k exec firstpod -- hostname

Ör: kubectl exec firstpod -- printenv
Ör: kubectl exec firstpod -c container1 --printenv
```
***
Pod'a shell bağlantısı oluşturma. _(Eğer pod içerisinde birden fazla container varsa **-c "container_ismi"** opsiyonu ile komutun çalıştırılması istenilen container belirtilebilir)_
```
$ kubectl exec -it "pod_ismi" -- "shell_konumu-yada-shel_komutu"

Ör: kubectl exec -it firstpod -- /bin/sh
kubectl exec -it firstpod -- sh  

Ör: kubectl exec -it firstpod -c container1 -- /bin/sh
```
***
Bir Kubernetes objesini silme. 
```
$ kubectl delete "obje_tipi" "obje_ismi"

Ör: kubectl delete pods firstpod
```
***
Json ya da Yaml formatında hazırlanmış bir konfigurasyon dosyası aracılığıyla yeni obje oluşturma. 
```
$ kubectl apply -f "dosya_yolu/dosya_ismi"

Ör: kubectl apply -f ./pod1.yaml
```
***
Oluşturulan objeyi dosya kullanarak silme
```
$ kubectl delete -f "dosya_yolu/dosya_ismi"

Ör: kubectl delete -f ./pod1.yaml
```
***
Label ve port konfigurasyonlarını da ekleyerek bir pod oluşturma. 
```
$ kubectl run "pod_ismi" --image="image_ismi" --port="port_numarası" --labels"anahtar:değer_eşlenikleri" --restart=Never

Ör: kubectl run secondpod --image=nginx --port=80 --labels="app=front-end,team=developer" --restart=Never
```
***
Cluster'da bulunan bir Kubernetes objesini varsayılan text editörü ile açarak özelliklerini güncelleme. Hemen degisikligi uygular.
```
$ kubectl edit "obje_tipi" "obje_ismi"

Ör: kubectl edit pods firstpod
```
***
kubectl komutları sonucu oluşan çıktıyı tek seferlik görmek yerine değişiklikleri izleyebilme imkanına **-w** opsiyonu ile kavuşuruz _(linux dünyasındaki **watch** komutunun yaptığı işin bir benzerini **-w** opsiyonu sağlar)_
```
$ kubectl "komut" -w

Ör: kubectl get pods -w
```
***
kubectl'in çalıştırıldığı bilgisayar üzerinden cluster'da bulunan bir objeye tünel açılarak bağlantı kontrolü yapılması "detaylarına network konusunda geleceğiz"
```
$ kubectl port-forward "obje_tipi"/"obje_ismi" "local_port":"hedef_port"

Ör: kubectl port-forward pod/multicontainer 8080:80
```

Note to Self:

Pod yasam dongusu:
yaml --> etcd (node bilgisi eklenir sched) <--> sched --> api -->   node (etcd deki node)--> 
                                                                    kubelet (image ceker, container olusturulur )
                                                                    k-proxy

    1. Pod lifecycle: Pending, Creating, ImagePullBackOff, Runing, Succeeded, Failed, CrashLoopBackOff
    
    Pending: yeni bir kod olusturuldu, etcd uzerinde fakat daha objeler olusturulmadi. Sched Node bilgisini etcd gonderir.
    2. Creating yaml dosyasindaki bilgiler etcd uzerinden schedule tarafindan cekildi fakat daha olusmadi. 
    3. ImagePullBackOff: image cekilme asamasinda sikinti var, register hatasai, int baglanti hatasi, image yazim hatasi
    4. Running
    5. RestartPolicy = Never | Always | on-failure

Commands:
    allias kubectl=k
    k apply -f podlife1.yaml
    kubectl get pods -w

Multi Container:
    - 2-tier basit uygulama.  
    - Ayni containerda birden fazla uygulama calisabilir fakat istenen tek uygulama calistirmaktir.
    - Izolasyon icin ayri containerlar daha mantikli. Scale yapilmasi gerektiginde birden cok uygulama calisan containerda tek uygulamaya bagli scale yapamayiz. Genisleme icin farkli containerda calistirmak daha mantiklali olur.
    - Bir pod icinde birden fazla container calistirilabilir. Ama yukardaki ayni sebeblerle tek pod tek container uygulanir.
    - Peki multi container tek port nerde kullanacagiz:
        .. Birbirine bagimli birden fazla uygulama varsa her biri ayri containerda fakat tek podda calistirilmasi mantikli olur.
        .. Ana uygulamaya bagimli, ayni network uzerinde izolasyon olmaksizin calisma ihtiyaci olan container ihtiyacini karsilar. Ayni volumea baglanirlar
        .. Ayni pod uzerinde calisan containerlar ayni worker nodda olusur (kubelet ayni)
        .. Containerlar ayri unitedir fakat podun lifecycle icinde yonetililrler. Containerlar birlikte olusur ve birlikte silinir.
        .. Containerlar localhosttan birbirlerine ulasabilir. Sanki ayni hosttaymis gibi davranirlar.
        .. Tek volume olusturarak buna baglanabiliriz.
        
        - k exec -it multicontainer -c webcontainer -- sh
        - apt update
        - apt install net-tools
        -
Init Container:
    1. Uygulama container baslatilmadan önce ilk olarak init container calisir.
    2. Init container yapmasi gereken islemi tamamlar ve kapanir.
    3. uygulama Container init container kapandiktan sonra calismaya baslar
    4. init container islemlerini tamamlamadan uygulama container baslatilmaz.
    5. Yaml dosyasinda containerin altinda degildir. Ayri bir object olarak initcontainer olarak olustturulur.
    - kubectl apply -f initcontainerpod
    - kubectl logs -f initcontainerpod -c initcontainer
    - watch -n 2 kubectl describe pod initcontainer
    - 


Labels:
    - alias k=kubectl
    - k get pods -l "app" (labelinda app olanlari listele)
    - k get pods -l "app=firstapp" (app anahtari firstapp olanlari listele)
    - k get pods -l "app=firstapp,tier=frontend" --show-labels (app ve tier bu degerlere sahip olanlar. Burda , and anlaminda)
    - k get pods -l "app=firstapp,tier!=frontend" --show-labels 
    - k get pods -l 'app in (firstapp)'
    - k get pods -l 'app in (firstapp, secondapp)' (burda virgul or anlaminda oldu)
    - k get pods -l '!app' (app olmayanlari listele)
    - k get pods -l "app in (firstapp), tier notin (frontend)" --show-labels
    - k label pods pod9 app=thirdapp (bir tane daha label eklenir)
    - k label --overwrite pods pod9 team
=team3 (team labelini degistirir)
    - k label pods --all foo=bar (tum podlara label ekler)
    - k label nodes minikube hddtype=ssd (bu label minikube uzerinde belirtilmeden container olusturulamaz)

Annotations:
    - labellar gibi metadataya bilgi ekleyip, cikarma degistirmeye imkan verir.
    - label hassa bilgilerdi ve tetikleyecegi hususlar olabilir.
    - annotationslar daha basit ve genel bilgileri icerir.
    - olusturma kurallari label lar gibidir.
    - labeldan fafrkli olarak veri kisminda tum karakterler kullanilabilir.

    - k apply -f podannotation.yaml
    - k annotate pods annotationpod foo=bar (annotation ekler)
    - k annotate pods annotationpod foo- (foo lari siler)
    - 
Namespace:
    - Firmada tum ekiplerin erisip islem yapabilecegi bir dosya sistemi kurmada fileserver da yönetilemeyecek cok buyuk sikintilar yasanabilir.
    - Her ekibe ayri bir klasor tahsis ederek sikintilarin cogunu ortadan kaldiririz.
    - Her ekibe proje bazli klasorde olsturabiliriz.
    - K8s cluster i file server olarak dusunursek, namespace de bu fileserver icindeki klasorler olarak dusunebiliriz.
    - Kaynak adlarin namespace icinde benzersiz olmasi gerekir. Namespaceler birbiri icine yerlestirilemez. Her Kubernetes kaynagi yalnizca bir namespace icinde olabilir.
    - Namespace cluster kaynaklarini birden cok kullanici arasinda bölmeye imkan verir.
    - Varsayilan olarak 4 namespace oludur:
        1. default
        2. kube-node-lease
        3. kube-public
        4. kube-system
    - aksi belirtilmedikce kubectl komutlari default namespace de calisir.

    - kubectl get pods --namespace-system
    - k get pods --all-namespaces
    - k create namespace app1 (app1 namespaceini olustur)
    - k exec -it namespacepod -n development --sh (development namespace inde calistir)
    - k config set-context --current --namespace=devolepment (varsayilan development namespace olur)
    - k delete namespaces development (development namespace ini siler)
