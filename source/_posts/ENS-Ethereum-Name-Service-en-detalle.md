---
title: Tutorial. ENS en detalle.
date: 2017-10-08 00:47:24
tags: ENS,Ethereum,Tutorial
---





#### Carlos Buendía (@buendiadas)

http://buendia.io


# ENS: Ethereum name Service

![hello](https://cdn-images-1.medium.com/max/1600/1*U-48MhnGYYpJkxZObAw20w.png)



## Motivación

Si bien la web necesitaba un estandar para la resolución de nombres legibles en lugar del acceso a IP's, la web3 (o aquello a lo que aspira conseguir Ethereum) necesitará un protocolo similar. En mayor medida si cabe, ya si bien que una IPv4 puede ser incluso legible, una dirección en Hexadecimal como la que encontramos en la figura anterior puede ser una pesadilla. Entre otros motivos, este problema facilita la ejecución de ataques de [phising ](http://buendia.io/htmlposts/tokensale.html#22), ya que un cambio de dirección por el atacante pasará inadvertido por cualquier persona (incluso en muchos casos el dueño de la dirección).

Además de las direcciones, podemos comenzar a asociar una multitud de registros nuevos, y, de la misma manera que DNS albergaba diferentes registros (MX,A etc..) comenzar a dirigir peticiones de Swarm, DNS, DNI (para ID's), o multitud de nuevas aplicaciones.

------

## Arquitectura

De la misma manera que dividimos la arquitectura de DNS en tres diferentes componentes, dividimos la arquitectura de ENS en otros 3:

1. [**ENS** ](https://github.com/ethereum/ens/blob/master/contracts/ENS.sol): Un registro único para todos los dominios de Ethereum. Un Smart contract muy simplón: mapping de los nombres registrados a dos direcciones:

   - La dirección del owner
   - La dirección del resolver
   - TTL

   Comparando con la arquitectura DNS, sustituiría la función de los Nameservers.

2. [**El resolver**](https://github.com/ethereum/ens/blob/master/contracts/PublicResolver.sol) Otro mapping, en este caso desde el dominio seleccionado a los diferentes recursos asociados (namespace) .

3. [**Contratos de registro**]: Diseñados para gestionar dominios jerarquicamente inferiores .

------

## Arquitectura

![architecture](https://cdn-images-1.medium.com/max/1600/1*WfSYnum96UqchRJj83xZXA.png)



------



## ENS Registry

```javascript
contract AbstractENS {
    function owner(bytes32 node) constant returns(address);
    function resolver(bytes32 node) constant returns(address);
    function ttl(bytes32 node) constant returns(uint64);
    function setOwner(bytes32 node, address owner);
    function setSubnodeOwner(bytes32 node, bytes32 label, address owner);
    function setResolver(bytes32 node, address resolver);
    function setTTL(bytes32 node, uint64 ttl);

    event NewOwner(bytes32 indexed node, bytes32 indexed label, address owner);
    event Transfer(bytes32 indexed node, address owner);
    event NewResolver(bytes32 indexed node, address resolver);
    event NewTTL(bytes32 indexed node, uint64 ttl);
}
```

------

## ENS Registry

Básicamente:

```c
    struct Record {
        address owner;
        address resolver;
        uint64 ttl;
    }

    mapping(bytes32=>Record) records;
```

## Namehash

¿Cómo mantenemos cierto grado de privacidad sobre los nodos/Keys/dominios, manteniendo un URI?

```javascript
var labels = name.split('.')

    for(var i = labels.length - 1; i >= 0; i--) {
      var labelSha = sha3(labels[i])
      node = sha3(new Buffer(node + labelSha, 'hex'))
    }
```

Básicamente, haciendo un Keccak hash recursivo de cada "label", separada por los puntos.

Definición: [EIP137](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-137.md)

## Namehash: Implementación

```sh
npm install eth-ens-namehash -S
```



```javascript
var namehash = require('eth-ens-namehash')
var hash = namehash('buendia.eth')
// '0x3b54251663d00d80dfd97b41b67522dd6e85564dec4161e78630a8772bf3be97'
```



------

## Registrars

- Contrato con capacidad de registrar subdominios de un registro
- Fija las normas para registrar estos subdominios
- Owner de ENS al deployer del contrato (actualmente una multisig)
- Por jerarquía, el registro principal actualmente es '.eth', un 'auctionRegistrar'

------

## Ejemplo: [FIFS Registrar](https://github.com/ethereum/ens/blob/master/contracts/FIFSRegistrar.sol)

```javascript
import './AbstractENS.sol';
/**
 * A registrar that allocates subdomains to the first person to claim them.
 */
contract FIFSRegistrar {
    AbstractENS ens;
    bytes32 rootNode;
  //...
    function FIFSRegistrar(AbstractENS ensAddr, bytes32 node) {
        ens = ensAddr;
        rootNode = node;
    }
    function register(bytes32 subnode, address owner) only_owner(subnode) {
        ens.setSubnodeOwner(rootNode, subnode, owner);
    }
}
```

------

## Ejemplo práctico FIFS (I)

En la terminal 2:

```sh
$ testrpc
```

En la terminal 1:

```sh
$ git clone https://github.com/ethereum/ens.git
$ truffle migrate --network dev.fifs
$ truffle console
```

Almacena las claves migradas (yourENSDeployedAddress, yourFIFSDeployedAddress)

------

## Ejemplo práctico FIFS (II)

```javascript
const namehash = require('./node_modules/eth-ens-namehash');
const ens = ENS.at(yourENSDeployedAddres);
const fifs= FIFSRegistrar.at(yourFIFSDeployedAddress);
const tld= 'eth'; // This is by default TLD, probably opened in a future
const default_account = web3.eth.accounts[0];

let node = namehash('eth');
//'0x93cdeb708b7545dc668eb9280176169d1c33cfd8ed6f04690a0bcc88a93fc4ae'

ens.owner(''); //Owner of the ENS Contract
// default_account
ens.owner(node);
// FIFS_Registrar
```

------

## Ejemplo práctico FIFS (III)

```javascript
let subnode='buendiadas'; //I want to register my name!
let complete_node=subnode + '.' + tld; //buendiadas.eth
let curr_account= web3.eth.accounts[1]; //My account!

let tx = fifs.register(web3.sha3(subnode),curr_account);
let new_owner=ens.owner(namehash(complete_node);
new_owner
// My Account!
```

------

## Resolver

- Acceso al NameSpace del dominio
- Cada tipo de registro (Recuerda A,MX, etc…) se representa como una nueva interfaz.
- Actualmente:

| Record type       | Function(s) | Interface ID | Defined in                               |
| ----------------- | ----------- | ------------ | ---------------------------------------- |
| Ethereum address  | addr        | 0x3b3b57de   | [EIP137](https://github.com/ethereum/EIPs/issues/137) |
| ENS Name          | name        | 0x691f3431   | [EIP181](https://github.com/ethereum/EIPs/issues/181) |
| ABI specification | ABI         | 0x2203ab56   | [EIP205](https://github.com/ethereum/EIPs/pull/205) |
| Public key        | pubkey      | 0xc8690233   | [EIP619](https://github.com/ethereum/EIPs/pull/619) |

------

## Ejemplo práctico resolver

Vamos a llenar el campo address (interfaz 0x3b3b57de)!

```javascript
const res_addres= PublicResolver.address;
ens.setResolver(namehash(complete_node), res.address,{from:curr_account});//Only allowed from curr_account!

res.setAddr(namehash(complete_node), web3.eth.accounts[2], {from: curr_account});
getAddr('myname.test')
```

------

# The end

![img](https://s3.amazonaws.com/codementor_content/2015-Aug-week2/cookie.jpg)
