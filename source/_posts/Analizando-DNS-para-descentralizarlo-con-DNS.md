---
title: Analizando DNS para después descentralizarlo con ENS
date: 2017-10-07 00:47:24
tags: ENS,Ethereum,Tutorial
---


## Intro

En el siguiente post, observaremos en detalle el servicio de registro de nombre de Ethereum (ENS) y haremos un pequeño tutorial de cómo utilizar Ethereum Name Service para registrar tu propio dominio y asociar variables al mismo.

*Como siempre, comenzaremos explorando los fundamentos (en este caso, los protocolos actualmente utilizados en la web) para comprender el racional de arquitectura, posteiormente analizaremos el estandard ENS y finalmente realizaremos ejercicio sobre la red actual*.

*Esto ayudará a entender los requisitos necesarios para futuras necesidades de ENS y funcionalidades requeridas por ENS, de modo que podamos reemplazar completamente el servicio*.

*Para aquellos que sepan cómo funciona DNS y quieran ir al grano con ENS, pueden pasar directamente al punto [ENS](#ENS:Ethereum-Name-Service)*

### Precedente: DNS (Domain Name System)

[DNS (Domain Name System), RFC 1035](https://www.ietf.org/rfc/rfc1035.txt) es uno de los servicios pilares de la web bien conocido por todos, cuyo objetivo básico es el de proveer de un servicio de nombres para el recursos. Como se verá en puntos posteriores, su funcionamiento tiene una estructura jerárquica y distrubuida (no descentralizada), ya que su base de datos se encuentra repartida entre numerosos servidores (Domain Servers).

Fundamentalmente, su funcionamiento puede separarse en tres funcionalidades principales:

- Arboles de dominio
- Resolvers
- Namespaces
- Name Servers

Que exploraremos en los puntos posteriores.

### Namespaces

El estandar DNS define una estructura de arbol en el cual, cada rama representa un dominio separado. De esta manera. Cada uno de estos dominios tiene asociados una serie de recursos (invisibles para el usuario), de manera que según la funcionalidad que una query haga a esta dirección tendrá acceso a un [recurso](http://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml) diferente. La mayor parte de los lectores que haya configurado un servidor de nombres le serán familiares la mayor parte de estos [recursos](http://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml), que incluyen algunos como los siguientes:

| TYPE  | Value | Meaning                                | Reference                                | Template | Registration Date |
| ----- | ----- | -------------------------------------- | ---------------------------------------- | -------- | ----------------- |
| A     | 1     | a host address                         | [[RFC1035](http://www.iana.org/go/rfc1035)] |          |                   |
| NS    | 2     | an authoritative name server           | [[RFC1035](http://www.iana.org/go/rfc1035)] |          |                   |
| MD    | 3     | a mail destination (OBSOLETE - use MX) | [[RFC1035](http://www.iana.org/go/rfc1035)] |          |                   |
| MF    | 4     | a mail forwarder (OBSOLETE - use MX)   | [[RFC1035](http://www.iana.org/go/rfc1035)] |          |                   |
| CNAME | 5     | the canonical name for an alias        | [[RFC1035](http://www.iana.org/go/rfc1035)] |          |                   |

… Y así un gran listado de ellos

Un ejemplo para poder acceder a los recursos asociados a un registro determinado es utilizando el comando ``$host`` (Unix):

```sh
  $host buendia.io

  buendia.io has address 192.30.252.154
```

En el que comprobamos la dirección IPv4 del host de la página ( registro tipo A). O utilizando la opción -a, observando el resto de registros del NameSpace.

```sh
  $host -a buendia.io
  ;; ANSWER SECTION:
  buendia.io.		1642	IN	MX	20 eforward5.registrar-servers.com.
  buendia.io.		1642	IN	MX	10 eforward2.registrar-servers.com.
  buendia.io.		1642	IN	MX	15 eforward4.registrar-servers.com.
  buendia.io.		1642	IN	MX	10 eforward1.registrar-servers.com.
  buendia.io.		1642	IN	MX	10 eforward3.registrar-servers.com.
  buendia.io.		753	IN	A	192.30.252.153
  buendia.io.		753	IN	A	192.30.252.154
```



En el cual podemos comprobar, por ejemplo, que el dominio [buendia.io](http://buendia.io) tiene:

- Registros tipo "A" (IPv4) del host a IP's: ```sh 192.30.252.153```, ```sh 192.30.252.154```

- Registros tipo "MX" (Mail Exchange) al servidor: eforward5.registrar-servers.com

### Name Servers

Aplicación en un servidor que contiene la información sobre el arbol, la cual responde a las queries para aquellos dominios de los que tiene autoridad (estructura jerárquica) y reenvía las peticiones a otros dominios de los que no tiene autoridad a otro resolver, lo cual recursivamente permite acceder a todos los dominios en un árbol.

Podemos observar donde se encuentra el Nameserver al que estamos accediendo a través del resolver haciendo un ``$nslookup``:

```shell
$nslookup buendia.io

Server:		192.168.0.1
Address:	192.168.0.1#53

Non-authoritative answer:
Name:	buendia.io
Address: 192.30.252.154
Name:	buendia.io
Address: 192.30.252.153
```

Lo cual nos devuelve la dirección del NameServer de autoridad en mi red local (Mi router), y la respuesta obtenido.

### Resolvers

```
                 Local Host                        |  Foreign
                                                   |
    +---------+               +----------+         |  +--------+
    |         | user queries  |          |queries  |  |        |
    |  User   |-------------->|          |---------|->|Foreign |
    | Program |               | Resolver |         |  |  Name  |
    |         |<--------------|          |<--------|--| Server |
    |         | user responses|          |responses|  |        |
    +---------+               +----------+         |  +--------+
                                |     A            |
                cache additions |     | references |
                                V     |            |
                              +----------+         |
                              |  cache   |         |
                              +----------+         |
```

¿A quíen debo preguntar como usuario para saber donde acceder al recurso? Los resolvers se encargan de esta tarea. Un resolver tiene acceso al menos a **un servidor DNS**.

En este caso, en UNIX, puedes hacer:

```shell
$ cat /etc/resolv.conf
```

Lo cual devolverá algo parecido a:

```nameserver 192.168.0.1```

### Jerarquía

A partir de esta información, encontramos una estructura de comunicaciones entre NameServers y resolvers jerárquica, en la cual, cada una de las ramas *"padre"*, conoce a sus hijos.

![hierarchy](http://player.slideplayer.com/16/5263622/data/images/img4.jpg)



Y bien, ¿Quiénes son esos Root Name servers que tienen autoridad sobre todo servidor? Podemos encontrarlos en [IANA (Internet Assigned Numbers Authority)](https://www.iana.org/domains/root/servers):

| HOSTNAME           | IP ADDRESSES                      | MANAGER                                 |
| ------------------ | --------------------------------- | --------------------------------------- |
| a.root-servers.net | 198.41.0.4, 2001:503:ba3e::2:30   | VeriSign, Inc.                          |
| b.root-servers.net | 192.228.79.201, 2001:500:200::b   | University of Southern California (ISI) |
| c.root-servers.net | 192.33.4.12, 2001:500:2::c        | Cogent Communications                   |
| d.root-servers.net | 199.7.91.13, 2001:500:2d::d       | University of Maryland                  |
| e.root-servers.net | 192.203.230.10, 2001:500:a8::e    | NASA (Ames Research Center)             |
| f.root-servers.net | 192.5.5.241, 2001:500:2f::f       | Internet Systems Consortium, Inc.       |
| g.root-servers.net | 192.112.36.4, 2001:500:12::d0d    | US Department of Defense (NIC)          |
| h.root-servers.net | 198.97.190.53, 2001:500:1::53     | US Army (Research Lab)                  |
| i.root-servers.net | 192.36.148.17, 2001:7fe::53       | Netnod                                  |
| j.root-servers.net | 192.58.128.30, 2001:503:c27::2:30 | VeriSign, Inc.                          |
| k.root-servers.net | 193.0.14.129, 2001:7fd::1         | RIPE NCC                                |
| l.root-servers.net | 199.7.83.42, 2001:500:9f::42      | ICANN                                   |
| m.root-servers.net | 202.12.27.33, 2001:dc3::35        | WIDE Project                            |



![Comunicaciones](http://player.slideplayer.com/16/5263622/data/images/img9.jpg)

[Ejemplo de propagación de nombres](https://dnschecker.org/#A/buendia.io)

Pero bien, ya hemos hablado suficiente de DNS, y todos lo conocemos bien, **¿Por qué no empezamos a hablar de ENS?**
