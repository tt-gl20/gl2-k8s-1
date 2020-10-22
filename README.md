# gl2-k8s-1

### task:
1. Выполнить шаги что были на сессии
2. Настроиить config.json так чтоб контейнер запускался как демон
3. Настроить адреса бриджа и контейнера
4. Пробросить сеть внутрь контейнера, добавив в настроенный неймспейс
5. Проверить работоспособность: контейнер слушает порт и отвечает на запросы curl

## terminal session:

https://asciinema.org/a/367107

## commands:
```
 docker pull busybox
 docker run busybox
 docker ps -a
 mkdir rootfs
 docker export b96fb964b378 | tar xf - -C rootfs
 runc spec
 vi config.json
 
 sudo apt-get install bridge-utils
 sudo brctl addbr runc0
 sudo ip link set runc0 up                        
 sudo ip addr add 192.168.10.1/24 dev runc0
 sudo ip link add name veth-host type veth peer name veth-guest
 sudo ip link set veth-host up
 sudo brctl addif runc0 veth-host
 sudo ip netns add runc
 sudo ip link set veth-guest netns runc
 sudo ip netns exec runc ip link set veth-guest name eth0
 sudo ip netns exec runc ip addr add 192.168.10.101/24 dev eth0
 sudo ip netns exec runc ip link set eth0 up
 sudo ip netns exec runc ip route add default via 192.168.10.1
 
 sudo runc run demo
```

in other terminal session:
```
curl 192.168.10.101:8000
```


## content of config.json (parts)
```
      {
        "ociVersion": "1.0.1-dev",
        "process": {
                "terminal": false,
                "user": {
                        "uid": 0,
                        "gid": 0
                },
                "args": [
                        "sh","-c","while true ; do  echo -e 'HTTP/1.1 200 OK\n\n  tim  task V1.0' | nc -v -l -p 8000  ; done"
                ],
```

```
 "root": {
                "path": "rootfs",
                "readonly": false
            }
```

```
 {
      "type": "network",
      "path": "/var/run/netns/runc"
  },

```

thank's to : https://fstn.hateblo.jp/entry/2015/08/01/231302 , https://habr.com/ru/company/selectel/blog/316258/
