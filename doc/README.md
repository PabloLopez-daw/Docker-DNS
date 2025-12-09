# 🐳 Docker DNS: Servidor BIND9 Local

Este proyecto despliega un servidor DNS local utilizando **BIND9** en un contenedor **Docker**. El objetivo es simular un entorno de red con resolución de nombres propia, configurando una zona directa (`pablomoya.test`) y una zona inversa.

---

## 📋 Índice de Contenidos

1. [Requisitos Previos](#-requisitos-previos)
2. [Estructura del Proyecto](#-estructura-del-proyecto)
3. [Configuración del Servidor DNS](#-configuración-del-servidor-dns)
    - [Opciones Globales](#1-opciones-globales-namedconfoptions)
    - [Definición de Zonas](#2-definición-de-zonas-namedconflocal)
4. [Archivos de Zona](#-archivos-de-zona)
    - [Zona Directa](#1-zona-directa)
    - [Zona Inversa](#2-zona-inversa)
5. [Despliegue con Docker](#-despliegue-con-docker)
6. [Verificación y Pruebas](#-verificación-y-pruebas)

---

## 🛠 Requisitos Previos

Para "instalar" y ejecutar este servidor DNS, no necesitas instalar BIND9 directamente en tu ordenador. Solo necesitas tener instalado el entorno de contenedores.

### Lo que necesitas instalar:

1.  **Docker Desktop** (si usas Windows o Mac) o **Docker Engine** (si usas Linux).
    * *Windows/Mac:* Descárgalo desde [Docker Desktop](https://www.docker.com/products/docker-desktop).
    * *Linux:* Ejecuta `sudo apt install docker.io` (en Debian/Ubuntu) o sigue la [guía oficial](https://docs.docker.com/engine/install/).
2.  **Docker Compose**: Normalmente viene incluido con Docker Desktop.
    * Para verificar si lo tienes, abre una terminal y escribe: `docker-compose --version`.

---
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

## 8.Levantamos el servidor con un up -d
``` bash
    docker-compose up -d
```


## 9.Me da un error por conflicto de puertos he cambiado 5353 por 5454


## 10.Levantamos el servidor con un up -d y comprovamos que se ha levantado


## 11.Entramos en el contenedor para ejecutar los camandos para comprobar que va bien el DNS
``` bash
    docker exec -it dns-server bash
```

## 12.Ahora hacemos las comprobaciones de la sintaxis de los codigos , debe de dar 'OK'
``` bash
    named-checkconf /etc/bind/named.conf.options
    named-checkzone pablomoya.test /var/lib/bind/db.pablomoya.test
    named-checkzone 1.168.192.in-addr.arpa /var/lib/bind/db.192.168.1
```

## 13. Comprobamos los logs del docker en la terminal de windows
``` bash
    docker logs dns-server
```

## 14. Comprobacion de que el servidor DNS va correctamente 
``` bash
    nslookup debian.pablomoya.test 127.0.0.1
```

