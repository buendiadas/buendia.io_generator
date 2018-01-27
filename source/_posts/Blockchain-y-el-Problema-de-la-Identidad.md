---
title: Blockchain y el Problema de la Identidad
date: 2016-05-09 21:25:16
tags: Identidad,Ethereum,Blockchain
---

Uno de los grandes problemas aún no resueltos desde la creación de Internet está asociado a la creación de una identidad digital única en la red. La cuestión envuelve infinidad de variables técnicas, regulatorias y sociales. Sin embargo, alcanzar una solución óptima resultaría de gran importancia especialmente para el sector financiero, en el cual se eliminarían los enormes costes asociados los procesos tradicionales de Know Your Customer (KYC) exigidos en cumplimiento de la normativa de Prevención de Blanqueo de Capitales.
A la hora de diseñar una solución, cabe plantearse las siguientes preguntas:
¿Quién verifica que una persona es quien dice ser quién es?
¿Quién almacena la información asociada a esta persona?
¿Cómo hacer que dicha información sea verificable y válida para que una entidad financiera permita a un usuario realizar operaciones cumpliendo las políticas relativas a Prevención de Blanqueo de Capitales (AML, por sus siglas en inglés) de forma ágil?

El pasado mes de Abril, parte del equipo de Grant Thornton, en colaboración con otros compañeros, presentamos una [propuesta a este problema](https://github.com/oraclize/dapp-proof-of-identity.git) utilizando blockchain en uno de los Hackathons asociados a esta tecnología más importantes a nivel internacional, que daría respuesta a muchas de las preguntas mencionadas. La solución planteada tuvo un gran impacto en la comunidad, siendo incluso mencionado por Vitalik Butterin (creador de Ethereum) durante el evento Consensus2016 que se está celebrando esta semana.
blockathon_1
El equipo que desarrolló la prueba de concepto “Hidentity”.

## La solución planteada
La mayoría de las soluciones basadas en blockchain actualmente disponibles (Bitcoin, Ethereum o Eris, entre otras), funcionan de manera pseudoanónima. Esto quiere decir que cualquier participante de la red tendrá un identificador asociado (su clave pública), que sin embargo no estará asociado a ninguna identidad física.
Este pseudoanonimato ofrece muchas ventajas en multitud de casos de uso (por ejemplo, para casos de IoT). Sin embargo resulta un inconveniente para muchos otros, en los cuales es necesaria la asociación con la identidad física de la persona. Este es por ejemplo el caso de aquellas operativas relativas al sector financiero.
Con esta Prueba de Concepto se abordó esta problemática, alcanzando una solución que permitiría actuar sobre la cadena de bloques con una identidad verificada.
A continuación se muestra la arquitectura de la solución desarrollada, cuyos procesos  serán posteriormente explicados.

![Diagrama Identidad Digital](/images/id_architecture.png)

### Validación de la identidad
Para la realización de este proceso, en lugar de reinventar la rueda, se utilizaron los estándares ya utilizados en la actualidad: el certificado o firma digital asociado a los documentos nacionales de identificación (DNI) suministrados por las autoridades públicas de cada jurisdicción.
En este caso, y dada la iniciativa de Estonia a crear un estándar de identificación global, se realizó la lectura de un DNI de esta procedencia.
e-residency
### Verificación de la identidad suministrada
La lectura del DNI proporcionará una firma en formato RSA. Esta firma será procesada a través de un Smart Contract (“on-chain”, es decir, dentro de la cadena de bloques en cuestión), asociando automáticamente una dirección de dicha blockchain a la identidad de la persona. De esta manera, se verificará de manera irrefutable (e inmutable) la pertenencia de la dirección firmada a esta identidad.
Una vez se ha firmado la dirección pública, el usuario podrá utilizarla con asociación directa a su identidad, por lo que este proceso solo deberá realizarse una vez.

### Almacenamiento de variables asociadas a dicha identidad
Hecho esto, la persona dispondrá de una identidad digital almacenada en la blockchain o cadena de bloques, pudiendo asociar diferentes datos a ella, y de esta manera haciéndolos públicos e íntegros desde cualquier punto en la blockchain, de forma que cualquier entidad o persona (según el uso que se dé a esta solución) pueda acceder a ellos. Esto será fácilmente realizable a través del uso de Smart Contracts.
De esta manera, haciendo uso de la firma digital y la blockchain, logramos desarrollar una verificación segura, pública e interoperable de asociar la identidad de una persona a una clave pública de la blockchain, facilitando la trazabilidad de transacciones y permitiendo agilizar el cumplimiento de políticas de AML/KYC. De hecho, este tipo de soluciones habilitarían una consolidación de este tipo de procesos con los exigidos por la normativa FATCA (Foreign Account Tax Compliance Act), cuestión que será abordada en posteriores artículos.
El código desarrollado se encuentra actualmente abierto al público en [Github](https://github.com/oraclize/dapp-proof-of-identity.git), subido por Thomas Bertani (en la foto, el primero a la derecha). Por otro lado, el contrato se encuentra subido a la red de pruebas “Morden” de Ethereum, pudiendo observar su almacenamiento en el siguiente enlace.
