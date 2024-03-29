<h2>Подготовка серверов</h2>

В первую очередь, необходимо открыть 22 порт дляисходящих подключений от будущей Мастер-ноды

Подготавливаем окружение Мастер-ноды, либо того сервера, от куда будет произведен запуск kubespray:
```sh
yum install epel-release python-pip git
```

Далее, на всех нодах, в том числе и Мастер-ноде отключаем своп и файрвол:
```sh
swapoff -a 
firewall-cmd --state
systemctl stop firewalld
systemctl disable firewalld
```

От рута заходим на узлы и генерим ssh-ключи:
```sh
ssh-keygen -t rsa
```

Копируем сгенерированные ключи с каждого узла на Мастер-ноду, либо на тот сервер от куда будет произведен запуск kubespray:
```sh
ssh-copy-id root@host
```

Загружаем репозиторий Kubespray(в нашем случае, нужно позаботиться о том, как достать данный репо, ибо доступ к github закрыт):
```sh
git clone https://github.com/kubernetes-sigs/kubespray
```

Так же, необходимо позаботиться о том, как перетащить на сервер модули питона.
Если мы глянем в файл: requirements.txt, то увидим все необходимые зависимости, которые требуются для запуска Ansible:
```sh
ansible==3.4.0
ansible-base==2.10.15
cryptography==2.8
jinja2==2.11.3
netaddr==0.7.19
pbr==5.4.4
jmespath==0.9.5
ruamel.yaml==0.16.10
ruamel.yaml.clib==0.2.4
MarkupSafe==1.1.1
```

Установку пакетов производим с помощью pip:
```sh
pip install package_nam.py or package_name.whl
```

<h2>Подготовка Kubespray</h2>

Редактируем файл inventory:
```sh
cp -R kubespray/inventory/sample kubespray/inventory/local/inventory.ini
```

К примеру, у нас есть 4 ноды (1 мастер и 2 воркер-ноды)
```sh
kub-master-1 ansible_host=192.168.0.100(к примеру) ip=192.168.0.100(к примеру)
kub-node-1 ansible_host= ip=
kub-node-2 ansible_host= ip=

[kube-master]
kub-master-1

[etcd]
kub-master-1

[kube-node]
kub-node-1
kub-node-2

[k8s-cluster:children]
kube-master
kube-node
```

Редактируем group_vars/k8s-cluster/k8s-cluster.yml:
```sh
kube_network_plugin: flannel
kube_proxy_mode: iptables
cluster_name: (имя кластера)
```

Правим group_vars/k8s-cluster/k8s-net-flannel.yml, в зависимости от вашей подсети выставляем регекс, к примеру:
```sh
flannel_interface_regexp: ‘10\\.10\\.0\\.\\d{1,3}’
```
Создаем там файл kube-ingress.yml с содержимым:
```sh
node_labels:
  node-role.kubernetes.io/ingress: ""
node_taints:
  - "node-role.kubernetes.io/ingress=:NoSchedule"
```
Правим group_vars/k8s-cluster/addons.yml:
```sh
ingress_nginx_enabled: true
ingress_nginx_nodeselector:
  kubernetes.io/os: "linux"
ingress_nginx_namespace: "ingress-nginx"
ingress_nginx_insecure_port: 80
ingress_nginx_secure_port: 443
#путь до сертов
ingress_nginx_extra_args:
  - --default-ssl-certificate=
```

Запускем установку:
```sh
ansible-playbook -u root -k -i inventory/local/inventory.ini -b --diff inventory.ini
```
