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
