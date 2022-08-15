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

    

