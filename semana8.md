# Cómo construir un generador de números aleatorios para la blockchain

[Post original por Ori Pomerantz](https://blog.logrocket.com/build-random-number-generator-blockchain/)

![](https://blog.logrocket.com/wp-content/uploads/2022/06/build-random-number-generator-blockchain-nocdn.png)

La propia estructura de una cadena de bloques se basa en el determinismo. En un ecosistema blockchain, el estado de cada red es público; la transparencia es total. Si se conoce el estado y la entrada, se puede calcular la salida. El determinismo está relacionado con el consenso, que es lo que permite verificar el progreso de una cadena de bloques. Sin este determinismo, la verificación independiente del progreso de la cadena de bloques sería imposible, ya que la cadena de bloques dejaría de estar descentralizada.

En la mayoría de los casos de uso, los números aleatorios no pueden conocerse hasta que se utilizan realmente. Esto significa que los fundamentos mismos de una cadena de bloques, la transparencia y el consenso, dificultan bastante la generación de números aleatorios.

En este artículo, veremos cómo superar las restricciones para generar números aleatorios para una cadena de bloques. Veremos cómo construir y probar un contrato Solidity para un juego de apuestas de casino que utiliza números aleatorios. También discutiremos algunas estrategias para prevenir el abuso en un juego de apuestas blockchain.

Ten en cuenta que después de [The Merge](https://ethereum.org/en/upgrades/merge/) puede haber una fuente de aleatoriedad en el propio EVM; sin embargo, incluso si se implementa el EIP 4499, [la aleatoriedad seguirá estando lejos de ser perfecta.](https://eips.ethereum.org/EIPS/eip-4399#security-considerations)

**Temas:**

+ Casos de uso de la aleatoriedad
  + NFTs
  + Juegos

+ El protocolo de confirmación/revelación
+ Tutorial del juego de apuestas
  + Configuración del contrato
  + Transacciones con un rollup
  + Probando el contrato

+ Prevención de abusos de las apuestas
  + Protegerse de un "never reveal"
  + Protegerse contra el "frontrunning"

## Casos de uso de la aleatoriedad

Para algunos fines, como el muestreo estadístico, puede ser suficiente utilizar [números pseudoaleatorios](https://en.wikipedia.org/wiki/Pseudorandomness), que parecen aleatorios pero que en realidad han sido generados por un proceso determinista. Sin embargo, hay algunos casos en los que un número aparentemente aleatorio que puede predecirse no es suficiente.

Veamos algunos ejemplos.

### NFTs

Muchos proyectos de NFT, como [OptiPunks](https://www.optipunks.com/), [Optimistic Bunnies](https://www.optimisticbunnies.com/) y [Optimistic Loogies](https://optimistic.loogies.io/), asignan aleatoriamente atributos a sus NFT cuando se acuñan. Como algunos atributos son más valiosos que otros, el resultado del minteo debe permanecer desconocido para el acuñador hasta después del minteo.

### Juegos
Muchos juegos se basan en la aleatoriedad, ya sea para tomar decisiones o para generar información que se supone oculta al jugador. Sin aleatoriedad, los juegos de blockchain se limitarían a aquellos en los que toda la información es conocida por todos los jugadores, como el ajedrez o las damas.

## El protocolo commit/reveal
Entonces, ¿cómo generamos números aleatorios en la cadena de bloques, que es totalmente transparente? Recuerde que "no hay secretos en la cadena de bloques".

La respuesta está en las tres últimas palabras: "en la cadena de bloques". Para generar números aleatorios, utilizaremos un número secreto que una parte de la interacción tiene y la otra no. Sin embargo, nos aseguraremos de que el número secreto no esté en la cadena de bloques.

El protocolo commit/reveal permite que dos o más personas lleguen a un valor aleatorio mutuamente acordado utilizando una [función hash criptográfica](https://en.wikipedia.org/wiki/Cryptographic_hash_function). Veamos cómo funciona:

1. El sideA genera un número aleatorio, ``randomA``
2. El sideA envía un mensaje con el hash de ese número, ``hash(randomA)``. Este
compromete al SideA con el valor de ``randomA``, porque mientras que nadie puede adivinar el valor de ``randomA``, una vez que el SideA lo proporciona todo el mundo puede comprobar que su valor es correcto.
3. El sideB envía un mensaje con otro número aleatorio, ``randomB``
4. El sideA revela el valor de ``randomA`` en un tercer mensaje
5. Ambas partes aceptan que el número aleatorio es ``randomA ^ randomB``, el or ``exclusivo (XOR)`` de los dos valores.

La ventaja de XOR aquí es que se determina por igual por ambos lados, por lo que ninguno puede elegir un valor "aleatorio" ventajoso.

## Tutorial de juego de apuestas de casino

Para ver cómo se puede utilizar un generador de números aleatorios en un juego real de blockchain, revisaremos el código de Casino.sol, un juego de apuestas de casino. Casino.sol está escrito en Solidity y utiliza el esquema commit/reveal; se puede acceder a él en [GitHub.](https://github.com/qbzzt/qbzzt.github.io/tree/master/LogRocket/20220615-random)

### Configuración del contrato

Recorramos el código de apuestas Casino.sol; está en este archivo de [GitHub.](https://github.com/qbzzt/qbzzt.github.io/blob/master/LogRocket/20220615-random/contracts/Casino.sol)

Primero, especificamos la licencia y la versión de Solidity:

```
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;
```

A continuación, definimos un contrato llamado ``Casino.`` [Los contratos Solidity](https://blog.logrocket.com/writing-smart-contracts-solidity/) son algo similares a los objetos de otros lenguajes de programación.
```
contract Casino {
```
Ahora, creamos un [struct](https://docs.soliditylang.org/en/v0.8.14/structure-of-a-contract.html#struct-types), ``ProposedBet``, donde almacenaremos información sobre las apuestas propuestas:

```
struct ProposedBet {
   address sideA;
   uint value;
   uint placedAt;
   bool accepted;  
 }    // struct ProposedBet
 ```
Esta estructura no incluye el commitment, el valor ``hash(randomA)``, porque ese valor se utiliza como clave para localizar la ``ProposedBet``. Sin embargo, contiene los siguientes campos:

Campo |Tipo |Finalidad
---|---|---
sideA| address |la [dirección](https://docs.soliditylang.org/en/v0.8.14/types.html#address) que propone la apuesta
value| integer |el tamaño de la apuesta en Wei, la denominación más pequeña de Ether
placedAt| entero |la fecha y hora de la propuesta ["timestamp"](https://en.wikipedia.org/wiki/Unix_time)
accepted |Booleano| si la propuesta ha sido aceptada

Nota: el campo ``placedAt`` no se utiliza en este ejemplo, pero más adelante explicaré por qué es importante tener en cuenta esta información.

A continuación, creamos una estructura ``AcceptedBet`` para almacenar la información extra después de que la apuesta sea aceptada.

Una diferencia interesante aquí es que ``sideB`` nos proporciona ``randomB`` directamente, en lugar de un hash.
```
 struct AcceptedBet {
   address sideB;
   uint acceptedAt;
   uint randomB;
 }   // struct AcceptedBet
 
 ```
Estos son los [mapeos](https://docs.soliditylang.org/en/v0.8.14/types.html#mapping-types) que almacenan las apuestas propuestas y aceptadas:

```
 // Proposed bets, keyed by the commitment value
 mapping(uint => ProposedBet) public proposedBet;
 // Accepted bets, also keyed by commitment value
 mapping(uint => AcceptedBet) public acceptedBet;
 
 ```
 
A continuación, creamos un evento, ``BetProposed``. Los [eventos](https://docs.soliditylang.org/en/v0.8.14/abi-spec.html#events) son el mecanismo estándar utilizado en los contratos inteligentes Solidity para enviar mensajes al mundo exterior. Este evento le dice al mundo que un usuario (en este caso, ``sideA``) está proponiendo una apuesta y por cuánto.

 ```
 event BetProposed (
   uint indexed _commitment,
   uint value
 );
 
  ```
  
Ahora, creamos otro evento, ``BetAccepted``. Este evento le dice al mundo (y específicamente al ``sideA``, que propuso la apuesta), que es hora de revelar ``randomA``. No hay forma de enviar un mensaje desde la blockchain sólo a un usuario específico.

 ```
event BetAccepted (
   uint indexed _commitment,
   address indexed _sideA
 );
 
  ```
A continuación, creamos un evento, ``BetSettled``. Este evento se emite cuando se liquida la apuesta.

 ```
event BetSettled (
   uint indexed _commitment,
   address winner,
   address loser,
   uint value   
 );
 
  ```
Ahora, creamos una función ``proposeBet``. El commitment es el único parámetro de esta función.

Todo lo demás (el valor de la apuesta y la identidad del ``sideA``) está disponible como parte de la transacción.

Observa que esta función es ``payable``. Esto significa que puede aceptar Ether como pago.

```
// Called by sideA to start the process
 function proposeBet(uint _commitment) external payable {
```

La mayoría de las funciones llamadas externamente, como ``proposedBet`` que se muestra aquí, comienzan con un montón de declaraciones [``require``](https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions).


```
require(proposedBet[_commitment].value == 0,
     "there is already a bet on that commitment");
   require(msg.value > 0,
     "you need to actually bet something");
```

Cuando escribimos un contrato inteligente debemos asumir que se intentará llamar a la función de forma maliciosa. Esta suposición nos llevará a establecer protecciones.

En el código anterior, tenemos dos condiciones:

1. Si ya hay una apuesta en el commitment, rechazar esta. De lo contrario, la gente podría tratar de usarlo para sobrescribir las apuestas existentes, lo que haría que la cantidad que ``sideA`` puso se quedara atascada en el contrato para siempre
2. Si la apuesta es por 0 Wei, rechazarla

Si no se cumple ninguna de estas dos condiciones, escribimos la información en ``proposedBet.``

Debido a la forma en que funciona el almacenamiento Ethereum, no necesitamos crear una nueva estructura, llenarla, y luego asignarla al mapeo. En su lugar, ya existe una estructura para cada valor de commitment, rellenada con ceros - sólo tenemos que modificarla.
```
 proposedBet[_commitment].sideA = msg.sender;
 proposedBet[_commitment].value = msg.value;
 proposedBet[_commitment].placedAt = block.timestamp;
 // accepted is false by default
   
```

Ahora, le decimos al mundo sobre la apuesta propuesta y la cantidad:
```
   emit BetProposed(_commitment, msg.value);
 }  // function proposeBet
 
```

Necesitamos dos parámetros para saber qué acepta el usuario: el commitment y el valor aleatorio del usuario.
```
 // Called by sideB to continue
 function acceptBet(uint _commitment, uint _random) external payable {
 
```

En el siguiente código, comprobamos tres posibles problemas antes de aceptar la apuesta:

1. Si la apuesta ya ha sido aceptada por alguien, no puede ser aceptada de nuevo
2. Si la dirección de ``sideA`` es cero, significa que nadie ha hecho realmente la apuesta
3. ``sideB`` necesita apostar la misma cantidad que ``sideA``

```
   require(!proposedBet[_commitment].accepted,
     "Bet has already been accepted");
   require(proposedBet[_commitment].sideA != address(0),
     "Nobody made that bet");
   require(msg.value == proposedBet[_commitment].value,
     "Need to bet the same amount as sideA");
     
```


Si se han cumplido todos los requisitos, creamos la nueva ``AcceptedBet``, marcamos en la`` proposedBet ``que ha sido aceptada y emitimos un mensaje ``BetAccepted.``
```
   acceptedBet[_commitment].sideB = msg.sender;
   acceptedBet[_commitment].acceptedAt = block.timestamp;
   acceptedBet[_commitment].randomB = _random;
   proposedBet[_commitment].accepted = true;
   emit BetAccepted(_commitment, proposedBet[_commitment].sideA);
 }   // function acceptBet
 
```

¡La siguiente función es la gran ``reveal``!

``sideA`` revela ``randomA``, y podemos ver quién ganó:
```
// Called by sideA to reveal their random value and conclude the bet
 function reveal(uint _random) external {
 
 ```
 
No necesitamos el propio commitment como parámetro, porque podemos derivarlo de ``randomA.``

```
   uint _commitment = uint256(keccak256(abi.encodePacked(_random)));

```

Para reducir el riesgo de enviar accidentalmente ETH a direcciones donde se atascará, Solidity sólo nos permite enviarlo a direcciones del tipo[`` address payable``](https://docs.soliditylang.org/en/latest/types.html#address)
```
   address payable _sideA = payable(msg.sender);
   address payable _sideB = payable(acceptedBet[_commitment].sideB);
```

El valor aleatorio acordado es un XOR de los dos valores aleatorios, como se explica a continuación:

```
     uint _agreedRandom = _random ^ acceptedBet[_commitment].randomB;

```

Vamos a utilizar el valor de la apuesta en múltiples lugares dentro del contrato, así que por brevedad y legibilidad, crearemos otra variable, ``_value``, para contenerlo.
```
   uint _value = proposedBet[_commitment].value;
   
```

Hay dos casos en los que ese ``proposedBet[_commitment].sideA == msg.sender`` no es igual al commitment.

1. El usuario no ha realizado la apuesta
2. El valor proporcionado como ``_random`` es incorrecto. En este caso, ``_commitment`` será un valor diferente y, por lo tanto, la apuesta propuesta en ese lugar no tendrá el valor correcto para ``sideA.``

```
   require(proposedBet[_commitment].sideA == msg.sender,
     "Not a bet you placed or wrong value");
 ```
```    
   require(proposedBet[_commitment].accepted,
     "Bet has not been accepted yet");
```

La función anterior ``proposedBet[_commitment].accepted`` sólo tendrá sentido después de que la apuesta haya sido aceptada.

A continuación, utilizamos el pedazo menos significativo del valor para decidir el ganador:
```
 // Pay and emit an event
   if (_agreedRandom % 2 == 0) {
   
```  

Aquí, le damos al ganador la apuesta y emitimos un mensaje para decirle al mundo que la apuesta se ha resuelto.

```
   // sideA wins
     _sideA.transfer(2*_value);
     emit BetSettled(_commitment, _sideA, _sideB, _value);
   } else {
     // sideB wins
     _sideB.transfer(2*_value);
     emit BetSettled(_commitment, _sideB, _sideA, _value);     
   }
   
```

Ahora, borraremos el almacén de apuestas, que ya no es necesario.

Cualquiera puede mirar atrás en la blockchain y ver cuál era el commitment y el valor revelado de la apuesta. El propósito de borrar estos datos es cobrar la devolución del gas por limpiar el almacenamiento que ya no se necesita.
```
  // Cleanup
   delete proposedBet[_commitment];
   delete acceptedBet[_commitment];

```
Por último, tenemos el final de la función y el contrato:
```
 }  // function reveal
}   // contract Casino
```

## Transacciones con un rollup

En el momento de escribir esto, la forma más económica de realizar transacciones en Ethereum es utilizar un [rollup](https://ethereum.org/en/layer-2/).

Básicamente, un rollup es un blockchain que escribe todas las transacciones en Ethereum, pero ejecuta el procesamiento en otro lugar donde es más barato. Recuerda, cualquiera puede verificar el estado del blockchain, porque Ethereum no es censurable.

La raíz de estado se publica entonces en la Capa 1, y hay garantías ([matemáticas](https://ethereum.org/en/developers/docs/scaling/zk-rollups/) o (económicas)[https://ethereum.org/en/developers/docs/scaling/optimistic-rollups/]) de que es el valor correcto. Utilizando la raíz de estado, es posible probar cualquier parte del estado - por ejemplo, probar la propiedad de algo.

Este mecanismo significa que el procesamiento (que puede realizarse en el rollup, o Capa 2) es muy barato, y los datos de la transacción (que deben almacenarse en Ethereum, o Capa 1) son, en comparación, muy caros. Por ejemplo, en el momento de escribir esto, el gas de la Capa 1 cuesta 20.000 veces más que el de la Capa 2 en el [rollup](https://www.optimism.io/) que yo utilizo. Puedes consultar [aquí la relación actual entre los precios del gas de Capa 1 y Capa 2.](https://public-grafana.optimism.io/d/9hkhMxn7z/public-dashboard?orgId=1&refresh=5m)

Por esta razón, ``reveal`` sólo toma ``randomA.``

Podría haber escrito el juego Casino.sol para obtener también el valor del commitment, y entonces podría distinguir entre valores incorrectos y apuestas que no existen. Sin embargo, en un rollup, esto aumentaría significativamente el coste de la transacción.

## Probando el contrato

[``casino-test.js``](https://github.com/qbzzt/qbzzt.github.io/blob/master/LogRocket/20220615-random/test/casino-test.js) es el código JavaScript que prueba el contrato de Casino.sol. Es repetitivo, así que sólo explicaré las partes interesantes.

La función hash del paquete ethers (``ethers.utils.keccak256``)[https://docs.ethers.org/v5/api/utils/hashing/#utils-keccak256] acepta una cadena que contiene un número hexadecimal. Este número no es convertido a 256bits si es más pequeño, así que por ejemplo ``0x01``, ``0x0001``, y ``0x000001`` todos hacen hash a valores diferentes. Para crear un hash que sea idéntico al producido en Solidity, necesitaríamos un número de 64 caracteres, aunque sea ``0x00..00``. El uso de la función hash aquí es una forma sencilla de asegurarse de que el valor que generamos es 32bytes.

```
const valA = ethers.utils.keccak256(0xBAD060A7)
```
Queremos comprobar los dos resultados posibles: una victoria de ``sideA`` y una victoria de ``sideB.``

Si el valor que envía ``sideB`` es el mismo que el hash de ``sideA``, el resultado es cero (cualquier número xor sí mismo es cero), y por lo tanto ``sideB`` pierde.

```
const hashA = ethers.utils.keccak256(valA)
const valBwin = ethers.utils.keccak256(0x600D60A7)
const valBlose = ethers.utils.keccak256(0xBAD060A7)
```
Cuando se utiliza el Hardhat EVM para pruebas locales, la razón de reversión se proporciona como un objeto [``Buffer``](https://nodejs.org/api/buffer.html) dentro de la traza de pila. Cuando nos conectamos a una blockchain real, la obtenemos en el campo ``reason``.

Esta función nos permite ignorar esta diferencia en el resto del código.
```
// Chai's expect(<operation>).to.be.revertedWith behaves
// strangely, so I'm implementing that functionality myself
// with try/catch
const interpretErr = err => {
 if (err.reason)
   return err.reason
 else
   return err.stackTrace[0].message.value.toString('ascii')
}
```

A continuación se muestra la forma estándar de utilizar la [biblioteca de pruebas Chai](https://www.chaijs.com/). ``Describimos`` un trozo de código con una serie de declaraciones ``it`` para denotar las acciones que deberían ocurrir.

```
describe("Casino", async () => {
 it("Not allow you to propose a zero Wei bet", async () => {
 
```
 
Aquí está el mecanismo estándar de Ethers para [crear una nueva instancia de un contrato:](https://docs.ethers.io/v5/api/contract/contract-factory/#ContractFactory--creating)

```
   f = await ethers.getContractFactory("Casino")
   c = await f.deploy()
```   
   
Por defecto, las transacciones tienen un ``value`` (cantidad de Wei adjunta) de cero.

```
  try {
     tx = await c.proposeBet(hashA)
```     

La llamada a la función ``tx.wait()`` devuelve un objeto ``Promise``. La expresión ``await <Promise>`` hace una pausa hasta que la promesa se resuelve, y luego continúa (si la promesa se resuelve con éxito) o lanza un error (si la promesa termina con un error).
  
```
     rcpt = await tx.wait()
```
  
Si no hay error, significa que se aceptó una apuesta Wei cero. Esto significa que el código falló la prueba.
  
```
  // If we get here, it's a fail
     expect("this").to.equal("fail")
  
```

Aquí capturamos el error y verificamos que el error coincide con el que esperaríamos del contrato Casino.sol.

Si ejecutamos utilizando el EVM Hardhat, el Buffer que obtenemos incluye algunos otros caracteres, por lo que es más fácil simplemente coincidir para asegurarnos de que vemos la cadena de error en lugar de comprobar la igualdad.
  
```
   } catch(err) {
     expect(interpretErr(err)).to
       .match(/you need to actually bet something/)       
   }
 })   // it "Not allow you to bet zero Wei"
  
```
Las otras condiciones de error, como ésta, son bastante similares:
```
 it("Not allow you to accept a bet that doesn't exist", async () => {
   .
   .
   .
 })   // it "Not allow you to accept a bet that doesn't exist"
```

Para cambiar el comportamiento por defecto de la interacción del contrato (por ejemplo, para adjuntar un pago a la transacción), [añadimos un hash de anulación como parámetro extra](https://docs.ethers.io/v5/api/contract/contract/#Contract-functionsCall). En este caso, enviamos 10Wei para probar si se acepta este tipo de apuesta:

```
   it("Allow you to propose and accept bets", async () => {
   f = await ethers.getContractFactory("Casino")
   c = await f.deploy()
   tx = await c.proposeBet(hashA, {value: 10})
```

Si una transacción tiene éxito, obtenemos el recibo cuando se resuelve la promesa de ``tx.wait().``

Entre otras cosas, ese recibo tiene todos los eventos emitidos. En este caso, esperamos tener un evento: ``BetProposed.``

Por supuesto, en código a nivel de producción también comprobaríamos que los parámetros emitidos son correctos.
```
   rcpt = await tx.wait()
   expect(rcpt.events[0].event).to.equal("BetProposed")
   tx = await c.acceptBet(hashA, valBwin, {value: 10})
   rcpt = await tx.wait()
   expect(rcpt.events[0].event).to.equal("BetAccepted")     
 })   // it "Allow you to accept a bet"   
 ```
 
A veces necesitamos tener unas cuantas operaciones con éxito para llegar al fallo que queremos probar, como por ejemplo un intento de aceptar una apuesta que ya ha sido aceptada:

```
it("Not allow you to accept an already accepted bet", async () => {
   f = await ethers.getContractFactory("Casino")
   c = await f.deploy()
   tx = await c.proposeBet(hashA, {value: 10})
   rcpt = await tx.wait()
   expect(rcpt.events[0].event).to.equal("BetProposed")
   tx = await c.acceptBet(hashA, valBwin, {value: 10})
   rcpt = await tx.wait()
   expect(rcpt.events[0].event).to.equal("BetAccepted")

```   
   
En este ejemplo, si la apuesta ya había sido aceptada, la transacción se revertirá, pero seguirá en la blockchain. Esto significa que si ``sideA`` revela prematuramente, cualquier otro puede aceptar la apuesta con un valor ganador.
```
    try {
     tx = await c.acceptBet(hashA, valBwin, {value: 10})
     rcpt = await tx.wait()  
     expect("this").to.equal("fail")     
   } catch (err) {
       expect(interpretErr(err)).to
         .match(/Bet has already been accepted/)
   }         
 })   // it "Not allow you to accept an already accepted bet"
 ```
 ```
 it("Not allow you to accept with the wrong amount", async () => {
     .
     .
     .
 })   // it "Not allow you to accept with the wrong amount"  
 it("Not allow you to reveal with wrong value", async () => {
     .
     .
     .
 })   // it "Not allow you to accept an already accepted bet"
 it("Not allow you to reveal before bet is accepted", async () => {
     .
     .
     .
 })   // it "Not allow you to reveal before bet is accepted" 
 ```
 
Hasta ahora hemos utilizado una única dirección para todo. Sin embargo, para comprobar una apuesta entre dos usuarios necesitamos tener dos direcciones de usuario.

Utilizaremos [``ethers.getSigners()``](https://hardhat.org/guides/waffle-testing.html#setting-up) de Hardhat para obtener un array de firmantes; todas las direcciones derivan del mismo mnemónico. A continuación, utilizaremos el método [``Contract.connect``](https://docs.ethers.io/v5/api/contract/contract/#Contract-connect) para obtener un objeto contrato que pase por uno de esos firmantes.

```
 it("Work all the way through (B wins)", async () => {
   signer = await ethers.getSigners()
   f = await ethers.getContractFactory("Casino")
   cA = await f.deploy()
   cB = cA.connect(signer[1])
```   
   
En este sistema, Ether se utiliza tanto como el activo que se apuesta y como la moneda utilizada para pagar las transacciones. Como resultado, el cambio en el balance de ``sideA`` es parcialmente el resultado de pagar por la transacción ``reveal.``

Para ver cómo ha cambiado el saldo debido a la apuesta, nos fijamos en el ``sideB.``

Comprobamos el ``preBalanceB:``

```
   .
   .
   .
   // A sends the transaction, so the change due to the
   // bet will only be clearly visible in B
   preBalanceB = await ethers.provider.getBalance(signer[1].address)   
   
 ```
Y compáralo con el ``postBalanceB:``

```
   tx = await cA.reveal(valA)
   rcpt = await tx.wait()
   expect(rcpt.events[0].event).to.equal("BetSettled")
   postBalanceB = await ethers.provider.getBalance(signer[1].address)       
   deltaB = postBalanceB.sub(preBalanceB)
   expect(deltaB.toNumber()).to.equal(2e10)  
 })   // it "Work all the way through (B wins)"
 it("Work all the way through (A wins)", async () => {
     .
     .
     .
     .
   expect(deltaB.toNumber()).to.equal(0)
 })   // it "Work all the way through (A wins)"
})     // describe("Casino")
```

## Evitar abusos en el juego de apuestas

Cuando escribes un contrato inteligente debes considerar cómo los usuarios hostiles podrían intentar abusar de él y luego implementar estrategias para prevenir esas acciones.

### Protegerse de una revelación imposible

Dado que no hay nada en el contrato que obligue a la ``parte A`` a revelar el número aleatorio, una ``parte A`` rencorosa y perdedora podría evitar emitir la transacción de ``revelación`` e impedir que la ``parte B`` cobre la apuesta.

Afortunadamente, este problema tiene una solución sencilla: Mantener una marca de tiempo de cuando ``sideB`` aceptó la apuesta. Si ha pasado un tiempo predefinido desde la marca de tiempo, y el ``sideA`` no ha respondido con una ``reveal`` válida, deja que el ``sideB`` emita una transacción de ``forfeit`` para cobrar la apuesta.

Esta es la razón de mantener un registro del momento en el que se llama a una función, el campo ``placedAt`` creado anteriormente.

## Protección contra el frontrunning

Las transacciones de Ethereum no se ejecutan inmediatamente. En su lugar, se colocan en una entidad llamada mempool, y los mineros (o proponentes de bloques después de la fusión) eligen qué transacciones colocar en el bloque que envían.

Normalmente, las transacciones elegidas son las que aceptan pagar más gas y, por tanto, proporcionan más beneficios.

Tan pronto como ``sideA`` ve la transacción ``acceptBet`` de ``sideB`` en el mempool, con un valor aleatorio que haría perder a ``sideA``, ``sideA`` puede emitir una transacción ``acceptBet`` diferente (posiblemente desde una dirección diferente).

Si la transacción ``acceptBet``de ``sideA`` da más gas al minero, podemos esperar que el minero ejecute su transacción primero. De esta forma, ``sideA`` podría retirarse de la apuesta en lugar de perderla.

Esta estrategia, llamada [frontrunning](https://consensys.github.io/smart-contract-best-practices/attacks/frontrunning/), es posible gracias a la estructura descentralizada de Ethereum y a la asimetría de información entre ``sideA`` y ``sideB`` después de que ``sideB`` envíe la transacción acceptBet.

No podemos abordar la descentralización; el mempool tiene que estar disponible, al menos para los mineros (y los stakers después de la Fusión), para que la red no sea censurable.

Sin embargo, podemos evitar el frontrunning eliminando la asimetría.

Cuando ``sideB`` envía la transacción ``acceptBet``, ``sideA`` ya conoce ``randomA``y ``randomB``, y por tanto puede ver quién ha ganado. Sin embargo, el ``sideB`` no tiene ni idea hasta la ``reveal``.

Si ``acceptBet`` de ``sideB`` sólo revela ``hash(randomB)``, entonces ``sideA`` tampoco sabe quién ganó, haciendo inútil ejecutar la transacción por adelantado. Entonces, una vez que la aceptación de la apuesta por parte de ``sideB`` forma parte de la cadena de bloques, tanto ``sideA ``como ``sideB`` pueden emitir transacciones de revelación.

Una vez que una de las partes emite una ``reveal``, la otra sabe quién ha ganado, pero si añadimos transacciones de ``pérdida``, no hay ninguna ventaja en negarse a revelar más allá del pequeño coste de la transacción en sí.

Un problema potencial que hay que tener en cuenta es que el ``sideB`` podría hacer exactamente el mismo commitment que el ``sideA``. Entonces, cuando el ``sideA`` revela, el ``sideB`` puede revelar el mismo número. El XOR de un número consigo mismo es siempre cero. Sin embargo, debido a la forma en que está escrito este juego en particular, en este escenario el ``sideB`` simplemente se estaría asegurando de que el ``sideA`` gane.

## Conclusión

En este artículo, hemos revisado un juego de apuestas de casino del contrato Solidity línea por línea para demostrar cómo construir un generador de números aleatorios para la blockchain. Crear números aleatorios en una máquina determinista no es trivial, pero descargando la tarea a los usuarios conseguimos una solución bastante buena.

También revisamos varias estrategias para evitar abusos o acciones hostiles en la quiniela de la blockchain.

Mientras ambas partes tengan interés en que el resultado sea aleatorio, podemos estar seguros del resultado.
