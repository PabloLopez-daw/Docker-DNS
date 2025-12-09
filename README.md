# Docker-DNS
Recuperacion DNS


## 1.Primero creamos Primero el git ignore y metemos los archivos temporales para ignorarlos 
``` bash
    echo ".DS_Store" > .gitignore
```

## 2.Creamos la siguiente estructura de carpetas 
``` bash 
    mkdir config
    mkdir zones
    mkdir doc
```

## 3.Creamos el archivo named.conf.options dentro de config le añadimos el siguiente codigo
``` bash
    acl confiables {
        192.168.0.0/16;  // Tu red local típica
        172.16.0.0/12;   // Redes internas de Docker
        127.0.0.0/8;     // Localhost
    };

    options {
        directory "/var/cache/bind";

        listen-on { any; };
        listen-on-v6 { none; }; 

        allow-transfer { none; }; 
        
        recursion yes;           
        allow-recursion { confiables; }; 

        dnssec-validation yes;   
    };
```

## 4.Creamos el archivo named.conf.local donde configuramos las zonas, dentro de config le añadimos el siguiente codigo
``` bash
    zone "pablomoya.test" {
        type master;
        file "/var/lib/bind/db.pablomoya.test";
    };

    zone "1.168.192.in-addr.arpa" {
        type master;
        file "/var/lib/bind/db.192.168.1";
    };
```

## 5.Ahora haremos el archivo de la zona directa (db.pablomoya.test) y lo configuramos así
``` bash
    $TTL    86400
    @       IN      SOA     ns.pablomoya.test. admin.pablomoya.test. (
                                1         
                            3600         
                            1800         
                            604800        
                            86400 )       
    ;
    @       IN      NS      ns.pablomoya.test.
    @       IN      A       192.168.1.50    
    ns      IN      A       192.168.1.50    
    debian  IN      A       192.168.1.50    
    www     IN      CNAME   debian
```

## 6.Ahora haremos el archivo de la zona inversa (db.192.168.1) y lo configuramos así
``` bash
    $TTL    86400
    @       IN      SOA     ns.pablomoya.test. admin.pablomoya.test. (
                                1        
                            3600         
                            1800         
                            604800        
                            86400 )       
    ;
    @       IN      NS      ns.pablomoya.test.
    50      IN      PTR     debian.pablomoya.test.
``` 

## 7.Creamos en la raiz del proyecto el docker-composer.yml y lo configuramos de la siguiente manera 
``` bash
version: '3.8'

services:
  bind9:
    image: ubuntu/bind9:9.16-20.04_beta
    container_name: dns-server
    platform: linux/amd64 
    environment:
      - BIND9_USER=root
      - TZ=Europe/Madrid
    ports:
      - "5353:53/tcp"
      - "5353:53/udp"
    volumes:
      - ./config/named.conf.options:/etc/bind/named.conf.options
      - ./config/named.conf.local:/etc/bind/named.conf.local
      - ./zones:/var/lib/bind
    command: ["-g", "-4"] 
    restart: unless-stopped
```
