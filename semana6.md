# Cómo construir una Dapp de staking

Bienvenido a la semana 6 de la serie R2W3. En este tutorial, aprenderás a desarrollar una simple dApp de staking con interés acumulado, slashing, y funcionalidad de depósito/retiro.

Durante las últimas 5 semanas de R2W3, hemos aprendido un montón de diferentes primicias de Solidity y Javascript que nos dan los bloques de construcción básicos para el desarrollo Web3.

Hemos aprendido a utilizar Hardhat desde cero, construir nuestros propios frontends, e incluso escribir Solidity.

Si bien todas estas habilidades son muy valiosas para los desarrolladores que buscan construir una base sólida, también hay herramientas que ayudan a abstraer algunas de las complejidades de la configuración del entorno y las dependencias que permiten a los constructores jugar un poco más fácil.

Una de estas herramientas que recomendamos es [Scaffold-eth]!(http://scaffoldeth.io/)

En su esencia, Scaffold-eth proporciona una pila lista para usar para la creación rápida de prototipos en Ethereum, dando a los desarrolladores acceso a herramientas de última generación para aprender y lanzar rápidamente una dApp basada en Ethereum.

Usando Scaffold-eth y Alchemy, puedes sintetizar y desplegar fácilmente código en la blockchain.

En este tutorial, usaremos el código base [del Reto #1](https://speedrunethereum.com/challenge/decentralized-staking) de [SpeedRunEthereum](https://speedrunethereum.com/) y trabajaremos juntos para construir una simple dApp de staking.

Si no estás familiarizado con el staking de cripto, se resume mejor como el proceso de bloquear/depositar tenencias de cripto en un protocolo DeFi o contrato inteligente para ganar intereses.

El staking de cripto se ha convertido en una piedra angular de muchos protocolos DeFi y permite a los desarrolladores crear productos financieros derivados y complejos.

Aunque la mayoría de los contratos de staking DeFi son extremadamente complejos, vamos a trabajar a través de uno de los más básicos para que podamos aprender los conceptos clave.

Juntos, aprenderemos los siguientes bloques de construcción para el staking:

1. Construir con Scaffold-Eth
 * Hacking frontends
 * Creación de "backends" Solidity
2. Transferencia de ETH de billetereas a contratos inteligentes y viceversa
3. Utilización de modificadores Solidity

Empecemos por entender cómo funciona Scaffold-Eth.

> Proximamente vídeo guiado en español.

Cuando hayas terminado con tu proyecto, envíalo aquí: https://university.alchemy.com/discord

## 1. Descargar Scaffold-Eth

En este tutorial, vamos a utilizar el entorno de desarrollo Scaffold-Eth para elaborar nuestros contratos inteligentes y armar nuestra interfaz de usuario frontend.

>📘¡Antes de empezar, quiero transmitir algunos detalles importantes a tener en cuenta!
>
>Scaffold-Eth es impresionante en la abstracción de las configuraciones del entorno y las dependencias frontales, lo que lo convierte en una herramienta poderosa.
>
>Si bien hay un montón de funcionalidades que son manejadas por Scaffold-Eth de forma automática, es importante sumergirse en el código para entender cómo algunas de estas características se generan cuando se tiene una comprensión más sólida del entorno de desarrollo en su conjunto.


Vamos a empezar por la bifurcación del repositorio de código base del [Desafío # 1](https://speedrunethereum.com/challenge/decentralized-staking) de [SpeedRunEthereum:](https://speedrunethereum.com/)


```
git clone https://github.com/scaffold-eth/scaffold-eth-challenges.git challenge-1-decentralized-staking

cd challenge-1-decentralized-staking

git checkout challenge-1-decentralized-staking

yarn install
```

Si has seguido el proceso correctamente, podrás ver una nueva carpeta titulada ``challenge-1-decentralized-staking`` en tu directorio de archivos base.

Después de ejecutar los comandos anteriores, nos encontramos con una gran carpeta llena de archivos.

Incluso antes de pasar al código, deberíamos familiarizarnos sobre dónde se almacenan los archivos clave en Scaffold-Eth para saber dónde centrarnos.

![](https://files.readme.io/b03d067-staking.png)
![](https://files.readme.io/b08c3d4-react-app.png)

En este tutorial, trabajaremos principalmente con ``Staker.sol`` y ``App.jsx``.

## 2. Configura tu entorno

A continuación, necesitarás tres terminales separadas para los siguientes tres comandos:

Inicia tu frontend React:

``yarn start``

Inicia tu backend Hardhat:

``yarn chain``

Compila, despliega y publica todos los contratos en tu archivo packages/contracts:

``yarn deploy``

>📘Cada vez que actualices tus contratos, ejecuta ``yarn deploy --reset`` para "refrescar" tus contratos en Scaffold-Eth.

Muy bien. Ahora deberías poder ver la interfaz de usuario de este repositorio en ``http://localhost:3000/``

## 3. Familiarízate con Scaffold-Eth

Aunque sé que te mueres por empezar con el código, ¡sólo hay que ocuparse de unos pocos detalles adicionales!

En nuestra vista por defecto, tenemos dos pestañas- ``Staker UI`` & ``Debug Contracts.``

![](https://files.readme.io/3dac8b8-stakerUI.png)

Muevete entre una y otra en tu frontend para echar un vistazo a las diferentes características.

``Staker UI`` contiene todos los componentes del frontend con los que interactuaremos principalmente.

Si hace clic en los botones proporcionados, te darás cuenta de que la mayoría de ellos no están conectados todavía y te encontrarás inmediatamente con errores.

> **📘 Echa un vistazo a ``Staker UI``. ¡Te darás cuenta de que es dolorosamente un espartano! Eso es lo que vamos a eliminar.**

Cualquier interacción en la cadena de Ethereum requiere testnet ETH, necesitarás testnet ETH local para empezar a hackear.

Primero, coge la dirección de tu cartera localhost.

Haz clic en el botón "Copiar" en la esquina superior derecha.
![](https://files.readme.io/c477349-0xf0xf.png)

A continuación, dirígete a la esquina inferior izquierda. Aquí podrás acceder al facucet "grifo" local.

* Copia y pegua la dirección en el campo abierto o clic en el icono "Billetera".
* Pega tu dirección en la vista ampliada
* Envíate ETH de prueba

![](https://files.readme.io/fae220d-lc_faucet.png)

![](https://files.readme.io/0c5349f-17...png)

Después de recargar tu monedero local, ¡podrás interactuar con tus contratos!

¡La segunda pestaña, ``Debug Contracts``, es otra visualización del frontend que contiene uno de los superpoderes de Scaffold-Eth!

Una vez que despliegues tus contratos (``yarn deploy``) y lo configures para leer los datos del contrato correctamente, generará automáticamente una interfaz de usuario básica que te permitirá interactuar con las funciones de tu contrato.

En el ejemplo de abajo, podemos leer y escribir información a través de nuestro contrato inteligente simplemente introduciendo parámetros y haciendo clic en "Enviar". Con Scaffold-Eth, no necesitamos usar sólo comandos CLI y tenemos una forma más intuitiva de crear prototipos.

>📘 Si quieres almacenar y ver una variable en el ``Debug Contracts`` tab, ¡asegúrate de establecer la variable como pública!

¡Impresionante!

Ahora que estamos familiarizados con Scaffold-Eth, podemos sumergirnos profundamente en el código.

## 4. Sumérgete en Solidity

Mirando nuestro archivo ``Staker.sol``, vemos que tenemos [un archivo](https://github.com/scaffold-eth/scaffold-eth/blob/challenge-1-decentralized-staking/packages/hardhat/contracts/Staker.sol) Solidity bastante vacío con un montón de comentarios que dictan lo que hay que rellenar.

Dado que el tutorial Road to Web3 (R2W3) se desvía del Desafío nº 1 de Scaffold-Eth, podemos seguir adelante e ignorar los comentarios y empezar con el siguiente código.

```
pragma solidity >=0.6.0 <0.7.0;

import "hardhat/console.sol";
import "./ExampleExternalContract.sol";

contract Staker {
  ExampleExternalContract public exampleExternalContract;
  
  constructor(address exampleExternalContractAddress) public {
      exampleExternalContract = ExampleExternalContract(exampleExternalContractAddress);
  }
  
}

```

### Parámetros del proyecto

Antes de escribir el código de nuestro contrato inteligente, repasemos cómo esperamos que funcione nuestra dApp de staking.

1. Para simplificar, sólo esperamos que un único usuario interactúe con nuestra dApp de Staking.
2. Tenemos que ser capaces de depositar y retirar del contrato ``Staker``
  * Hacer stake es una acción de un solo uso, lo que significa que una vez que depositamos no podemos volver a hacerlo de nuevo.
  * Los retiros del contrato eliminan todo el saldo principal y cualquier interés acumulado
3. El contrato Staker tiene una tasa de pago de intereses de 0,1 ETH por cada segundo que el ETH depositado es elegible para el devenir de intereses
4. Al desplegar el contrato, el contrato ``Staker`` debe comenzar con 2 contadores de tiempo. El primer plazo debe establecerse en 2 minutos y el segundo en 4 minutos
  * El plazo de 2 minutos dicta el periodo en el que el usuario de staker puede depositar fondos. (Entre t=0 minutos y t=2 minutos, el usuario puede depositar)
  * Todos los bloqueos que tengan lugar entre el depósito de fondos y el plazo de 2 minutos son válidos para la acumulación de intereses.
  * Una vez transcurrido el plazo de 2 minutos para retirar fondos, el usuario puede retirar la totalidad del saldo principal y los intereses devengados hasta que se cumpla el plazo de 4 minutos.
  * Una vez transcurrido el plazo adicional de 2 minutos para retiradas, el usuario no podrá retirar sus fondos desde que se agotó el plazo.

5. Si a un usuario le quedan fondos, tenemos una última función a la que podemos llamar para "bloquear" los fondos en un contrato externo que ya está preinstalado en nuestro entorno Scaffold-Eth, ``ExampleExternalContract.sol``

> 📘Si bien los parámetros de deposito enumerados anteriormente pueden parecer un poco enredados, muchas dApps de staking de la vida real presentan primitivas similares en las que los usuarios tienen un período limitado para depósitos y retiros.
>
> Además, muchas dApps desincentivan el capital "improductivo" que se queda en el banco una vez finalizados los periodos de stake.
>
> A veces, el protocolo DeFi puede incluso absorber los depósitos pendientes después de que finalice un periodo de espera similar al último parámetro que indicamos en nuestro tutorial.

### Mapeo de solidity

En nuestro contrato inteligente, necesitaremos dos mapeos que nos ayuden a almacenar algunos datos.

En particular, necesitamos algo para hacer un seguimiento de:

1. cuánto ETH se deposita en el contrato
2. la hora a la que se produjo el depósito

Podemos conseguirlo con

```
mapping(address => uint256) public balances; 
mapping(address => uint256) public depositTimestamps;
```

## Variables públicas

De acuerdo con las [directrices descritas anteriormente](https://docs.alchemy.com/docs/how-to-build-a-staking-dapp#project-parameters), también necesitaremos un puñado de variables diferentes.

```
uint256 public constant rewardRatePerSecond = 0.1 ether; 
uint256 public withdrawalDeadline = block.timestamp + 120 seconds; 
uint256 public claimDeadline = block.timestamp + 240 seconds; 
uint256 public currentBlock = 0;
```

La tasa de recompensa establece la tasa de interés para el desembolso de ETH sobre la cantidad principal depositada.

Los plazos de retirada y reclamación nos ayudan a establecer los plazos para que comience/finalice la mecánica de apuesta.

Y, por último, tenemos una variable que utilizamos para guardar el bloque actual.

>📘 Usamos ``block.timestamp`` + XXX segundos para crear plazos exactamente XXX segundos después de que se inicie nuestro contrato. Definitivamente es un poco "ingenuo" como mecanismo de temporización; ¿se te ocurre alguna forma mejor de implementarlo para que sea más generalizable, por ejemplo?

### Eventos

Aunque no enviaremos eventos a nuestro frontend, deberíamos asegurarnos de emitirlos en partes clave de nuestro contrato para garantizar que mantenemos las mejores prácticas de programación.

```
event Stake(address indexed sender, uint256 amount); 
event Received(address, uint); 
event Execute(address indexed sender, uint256 amount);
```
Ahora que ya tenemos los parámetros/variables clave, podemos pasar a crear las funciones específicas que utilizaremos en nuestro tutorial.


### Funciones de tiempo READ ONLY

Como se indica en los parámetros del proyecto, muchas de las diferentes funcionalidades de la dApp de stake están sujetas a "bloqueos de tiempo" que permiten/prohíben ciertas acciones en determinados momentos.

Aquí tenemos dos funciones diferentes que rigen el inicio y el final de la ventana de retirada.

```
  function withdrawalTimeLeft() public view returns (uint256 withdrawalTimeLeft) {
    if( block.timestamp >= withdrawalDeadline) {
      return (0);
    } else {
      return (withdrawalDeadline - block.timestamp);
    }
  }

  function claimPeriodLeft() public view returns (uint256 claimPeriodLeft) {
    if( block.timestamp >= claimDeadline) {
      return (0);
    } else {
      return (claimDeadline - block.timestamp);
    }
  }
  
```

Ambas funciones tienen un diseño muy familiar.

Ambas incluyen una declaración if -> else estándar.

El condicional simplemente comprueba si la hora actual es mayor o menor que los plazos dictados en la [ección de variables públicas.](https://docs.alchemy.com/docs/how-to-build-a-staking-dapp#public-variables)

Si el tiempo actual es mayor que los plazos preestablecidos, sabemos que el plazo ha pasado y devolvemos 0 para significar que se ha producido un "cambio de estado".

En caso contrario, simplemente devolvemos el tiempo restante antes de que se alcance la fecha límite.

### Modificadores

Para ver un ejemplo más detallado de un modificador, eche un vistazo a [Solidity By Example.](https://solidity-by-example.org/function-modifier)

A grandes rasgos, los modificadores de Solidity son piezas de código que pueden ejecutarse antes y/o después de una llamada a una función.

Aunque tienen muchos propósitos diferentes, uno de los casos de uso más comunes y básicos es para restringir el acceso a ciertas funciones si no se cumplen completamente determinadas condiciones.

En este tutorial, utilizaremos precisamente modificadores para ayudar a bloquear funciones clave que dictan nuestras funcionalidades de participación, retirada y repatriación.

Estos son los tres modificadores que utilizamos:

```

  modifier withdrawalDeadlineReached( bool requireReached ) {
    uint256 timeRemaining = withdrawalTimeLeft();
    if( requireReached ) {
      require(timeRemaining == 0, "Withdrawal period is not reached yet");
    } else {
      require(timeRemaining > 0, "Withdrawal period has been reached");
    }
    _;
  }

  modifier claimDeadlineReached( bool requireReached ) {
    uint256 timeRemaining = claimPeriodLeft();
    if( requireReached ) {
      require(timeRemaining == 0, "Claim deadline is not reached yet");
    } else {
      require(timeRemaining > 0, "Claim deadline has been reached");
    }
    _;
  }

  modifier notCompleted() {
    bool completed = exampleExternalContract.completed();
    require(!completed, "Stake already completed!");
    _;
  }
  
```

Los modificadores ``withdrawalDeadlineReached(bool requireReached)`` & ``claimDeadlineReached(bool requireReached)`` aceptan ambos un parámetro booleano y comprueban si sus respectivos plazos son **verdaderos** o **falsos**.

El modificador ``notCompleted()`` funciona de forma similar, pero en realidad es un poco más complejo aunque contiene menos líneas de código.

En realidad llama a una función ``completed()`` de un contrato externo fuera de ``Staker`` y comprueba si devuelve **true** o **false** para confirmar si se ha cambiado esa bandera.

Ahora, vamos a implementar los modificadores que acabamos de crear en el siguiente par de funciones para el acceso.

### Función de depósito/apuesta

En nuestra función de apuesta, usamos los modificadores creados anteriormente estableciendo los parámetros dentro de ``withdrawalDeadlineReached()`` a false y ``claimDeadlineReached()`` a false ya que no queremos que ninguna de las fechas límite haya pasado todavía.

```
  // Función stake para que un usuario pueda apostar ETH en nuestro contrato
  
  function stake() public payable withdrawalDeadlineReached(false) claimDeadlineReached(false) {
    balances[msg.sender] = balances[msg.sender] + msg.value;
    depositTimestamps[msg.sender] = block.timestamp;
    emit Stake(msg.sender, msg.value);
  }
  
```

El resto de la función es bastante estándar en un escenario típico de "depósito" en el que nuestra asignación de saldo se actualiza para incluir el dinero enviado.

También establecemos nuestra marca de tiempo de depósito con la hora actual del depósito para que podamos acceder a ese valor almacenado para los cálculos de interés más tarde.


### Función de retirada

En nuestra función de retiro, usamos nuevamente los modificadores creados anteriormente pero esta vez queremos que ``withdrawalDeadlineReached()`` sea true y ``claimDeadlineReached()`` sea false.

Este conjunto de modificadores/parámetros significa que estamos en el punto óptimo para la ventana de retirada, ya que es el momento para que la retirada tenga lugar sin ninguna penalización y además obtenemos intereses.



```
  /*
  Withdraw function for a user to remove their staked ETH inclusive
  of both the principle balance and any accrued interest
  */
  
  function withdraw() public withdrawalDeadlineReached(true) claimDeadlineReached(false) notCompleted{
    require(balances[msg.sender] > 0, "You have no balance to withdraw!");
    uint256 individualBalance = balances[msg.sender];
    uint256 indBalanceRewards = individualBalance + ((block.timestamp-depositTimestamps[msg.sender])*rewardRatePerSecond);
    balances[msg.sender] = 0;

    // Transfer all ETH via call! (not transfer) cc: https://solidity-by-example.org/sending-ether
    (bool sent, bytes memory data) = msg.sender.call{value: indBalanceRewards}("");
    require(sent, "RIP; withdrawal failed :( ");
  }
  
```

El resto de la función realiza algunos pasos importantes.

1. Comprueba que la persona que intenta retirar ETH realmente tiene una apuesta distinta de cero.
2. Calcula la cantidad de ETH adeudada en concepto de intereses tomando el número de segundos transcurridos desde el depósito hasta la retirada y multiplicándolo por nuestra constante de interés.
3. Establece el saldo de ETH depositado por el usuario en 0 para que no pueda producirse una doble contabilidad.
4. Transfiere el ETH del contrato inteligente de vuelta a la cartera del usuario.


### Ejecutar la función de distribución

Aquí, queremos que ``claimDeadlineReached()`` sea true ya que la distribución de fondos improductivos sólo puede ocurrir después de la marca de 4 minutos.

Del mismo modo, queremos que ``notCompleted`` sea true ya que esta dApp sólo está diseñada para un único uso.

```
    /*
  Allows any user to repatriate "unproductive" funds that are left in the staking contract
  past the defined withdrawal period
  */
  
  function execute() public claimDeadlineReached(true) notCompleted {
    uint256 contractBalance = address(this).balance;
    exampleExternalContract.complete{value: address(this).balance}();
  }
```

El resto de funciones:

1. Toma el saldo actual de ETH en el contrato ``Staker``
2. Envía el ETH al ``exampleExternalContract`` de la repo

Si has seguido en Solidity hasta ahora, tu ``Staker.sol`` debería tener el siguiente aspecto:

**``Staker.sol``**

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.4;

import "hardhat/console.sol";
import "./ExampleExternalContract.sol";

contract Staker {

  ExampleExternalContract public exampleExternalContract;

  mapping(address => uint256) public balances;
  mapping(address => uint256) public depositTimestamps;

  uint256 public constant rewardRatePerSecond = 0.1 ether;
  uint256 public withdrawalDeadline = block.timestamp + 120 seconds;
  uint256 public claimDeadline = block.timestamp + 240 seconds;
  uint256 public currentBlock = 0;

  // Events
  event Stake(address indexed sender, uint256 amount);
  event Received(address, uint);
  event Execute(address indexed sender, uint256 amount);

  // Modifiers
  /*
  Checks if the withdrawal period has been reached or not
  */
  modifier withdrawalDeadlineReached( bool requireReached ) {
    uint256 timeRemaining = withdrawalTimeLeft();
    if( requireReached ) {
      require(timeRemaining == 0, "Withdrawal period is not reached yet");
    } else {
      require(timeRemaining > 0, "Withdrawal period has been reached");
    }
    _;
  }

  /*
  Checks if the claim period has ended or not
  */
  modifier claimDeadlineReached( bool requireReached ) {
    uint256 timeRemaining = claimPeriodLeft();
    if( requireReached ) {
      require(timeRemaining == 0, "Claim deadline is not reached yet");
    } else {
      require(timeRemaining > 0, "Claim deadline has been reached");
    }
    _;
  }

  /*
  Requires that the contract only be completed once!
  */
  modifier notCompleted() {
    bool completed = exampleExternalContract.completed();
    require(!completed, "Stake already completed!");
    _;
  }

  constructor(address exampleExternalContractAddress){
      exampleExternalContract = ExampleExternalContract(exampleExternalContractAddress);
  }

  // Stake function for a user to stake ETH in our contract
  function stake() public payable withdrawalDeadlineReached(false) claimDeadlineReached(false){
    balances[msg.sender] = balances[msg.sender] + msg.value;
    depositTimestamps[msg.sender] = block.timestamp;
    emit Stake(msg.sender, msg.value);
  }

  /*
  Withdraw function for a user to remove their staked ETH inclusive
  of both principal and any accrued interest
  */
  function withdraw() public withdrawalDeadlineReached(true) claimDeadlineReached(false) notCompleted{
    require(balances[msg.sender] > 0, "You have no balance to withdraw!");
    uint256 individualBalance = balances[msg.sender];
    uint256 indBalanceRewards = individualBalance + ((block.timestamp-depositTimestamps[msg.sender])*rewardRatePerSecond);
    balances[msg.sender] = 0;

    // Transfer all ETH via call! (not transfer) cc: https://solidity-by-example.org/sending-ether
    (bool sent, bytes memory data) = msg.sender.call{value: indBalanceRewards}("");
    require(sent, "RIP; withdrawal failed :( ");
  }

  /*
  Allows any user to repatriate "unproductive" funds that are left in the staking contract
  past the defined withdrawal period
  */
  function execute() public claimDeadlineReached(true) notCompleted {
    uint256 contractBalance = address(this).balance;
    exampleExternalContract.complete{value: address(this).balance}();
  }

  /*
  READ-ONLY function to calculate the time remaining before the minimum staking period has passed
  */
  function withdrawalTimeLeft() public view returns (uint256 withdrawalTimeLeft) {
    if( block.timestamp >= withdrawalDeadline) {
      return (0);
    } else {
      return (withdrawalDeadline - block.timestamp);
    }
  }

  /*
  READ-ONLY function to calculate the time remaining before the minimum staking period has passed
  */
  function claimPeriodLeft() public view returns (uint256 claimPeriodLeft) {
    if( block.timestamp >= claimDeadline) {
      return (0);
    } else {
      return (claimDeadline - block.timestamp);
    }
  }

  /*
  Time to "kill-time" on our local testnet
  */
  function killTime() public {
    currentBlock = block.timestamp;
  }

  /*
  \Function for our smart contract to receive ETH
  cc: https://docs.soliditylang.org/en/latest/contracts.html#receive-ether-function
  */
  receive() external payable {
      emit Received(msg.sender, msg.value);
  }

}

```


## 5. Incursión en el Frontend

¡Impresionante! Acabamos de pasar por un montón de Solidity. Cuando se trata de pantallas frontend, Scaffold-Eth trata de mantener las cosas agradables y simples. ¡Contiene un montón de diferentes componentes react que dan a los usuarios soluciones de bajo código para UIs impresionantes! Te animo a jugar con los diferentes componentes disponibles para ti, pero, mientras tanto, vamos a aprender más en el lado del guerrero.

Mirando nuestro archivo ``App.jsx``, específicamente en [el bloque de código alrededor del enlace 573](https://github.com/scaffold-eth/scaffold-eth/blob/challenge-1-decentralized-staking/packages/react-app/src/App.jsx#L573), vemos un bloque de código utilizado para capturar eventos emitidos desde nuestros contratos Solidity y mostrarlos como una lista.

Efectivamente, nos permite registrar las diferentes acciones iniciadas desde nuestros contratos inteligentes, analizar la información almacenada, y luego permitir visualmente a los usuarios de la dApp ver su historial en la cadena.

Si bien vamos a practicar las buenas practicas de programación mediante la emisión de eventos en nuestro contrato Solidity, esta vez, vamos a ignorar los eventos en nuestro frontend para simplificar, así que vamos a eliminar ese bloque de código por completo.

Si miras en tu pestaña ``Staker UI``, notarás que la caja de eventos ha sido borrada.

### Modificaciones del frontend

En su mayor parte, gran parte del código de la interfaz seguirá siendo el mismo que el predeterminado. En el paso anterior, ya hemos eliminado el componente react de eventos.

Nuestro objetivo final será tener una interfaz de usuario agradable y sencilla que se parezca a la siguiente:

![](https://files.readme.io/5ff7eec-apy.png)


Ten en cuenta que el fronend por defecto esta faltando e incluso algunos elementos visuales que estan en el repo por defecto

### Tasa de recompensa por segundo
Para construir esta parte, vamos a introducir la siguiente pieza de codigo bajo el bloque del contrato de Staker:

```
  <div style={{ padding: 8, marginTop: 16 }}>
    <div>Reward Rate Per Second:</div>
    <Balance balance={rewardRatePerSecond} fontSize={64} /> ETH
  </div>
  
```

Aqui, tomamos ventaja del componente react del balance Scaffold-Eth'spara ayudarnos con el formato!

### Elementos de interfaz de usuario

Para construir la parte final de la visual de los componentes, estamos usando las siguientes 2 secciones de código:

```
   // ** keep track of a variable from the contract in the local React state:
  const claimPeriodLeft = useContractReader(readContracts, "Staker", "claimPeriodLeft");
  console.log("⏳ Claim Period Left:", claimPeriodLeft);

  const withdrawalTimeLeft = useContractReader(readContracts, "Staker", "withdrawalTimeLeft");
  console.log("⏳ Withdrawal Time Left:", withdrawalTimeLeft);
  
```
```
<div style={{ padding: 8, marginTop: 16, fontWeight: "bold" }}>
  <div>Claim Period Left:</div>
  {claimPeriodLeft && humanizeDuration(claimPeriodLeft.toNumber() * 1000)}
</div>

<div style={{ padding: 8, marginTop: 16, fontWeight: "bold"}}>
  <div>Withdrawal Period Left:</div>
  {withdrawalTimeLeft && humanizeDuration(withdrawalTimeLeft.toNumber() * 1000)}
</div>

```
Mientras la segunda seccion de codigo es familiar y se asemeja a lo que hemos visto anteriormente, la primera seccion de codigo es algo unica. 
No es que llamemos a claimPeriodLeft y ``withdrawalTimeLeft`` para acceder a los valores de variable en el frontend.

Sin embargo, primero debemos leer los valores desde el contrato inteligente. El primer fragmento de código maneja esta lógica.

### Misc. Edita otros componentes de la interfaz de usuario existentes

Ahora que has visto 2 ejemplos diferentes de componentes de interfaz de usuario frontend, ¡puedes averiguar cómo hacer los cambios restantes en el frontend para que se vea como el ejemplo proporcionado anteriormente!

Aquí sólo tendrás que ajustar unos cuantos parámetros, ¡así que no lo pienses demasiado!

### Freestyling on the UI

Mientras que Scaffold-Eth tiene un montón de componentes por defecto que permiten a los usuarios simplemente aprovechar las soluciones de "bajo código", también da a los usuarios acceso a las bibliotecas frontend más grandes.

Por defecto, tiene un gancho en Ant Design react components (https://ant.design/components/overview/) que permite a cualquiera extraer componentes de allí.

En nuestro frontend de ejemplo, vemos "líneas" que dividen cada gran bloque de componentes visuales.

Recrea estas líneas explorando las diferentes opciones disponibles en Ant Design

>📘 Si quieres una pista, ¡echa un vistazo en [Ant Dividers](https://ant.design/components/divider/)!
>
>¡No olvides que necesitamos importar el componente que planeamos usar en la parte superior de nuestro archivo ``App.jsx``!
>
>``import { Alert, Button, Col, Menu, Row, List, Divider } from "antd";``

Si has seguido el código del frontend, tu App.jsx debería tener el siguiente aspecto:

App.jsx

```
import WalletConnectProvider from "@walletconnect/web3-provider";
//import Torus from "@toruslabs/torus-embed"
import WalletLink from "walletlink";
import { Alert, Button, Col, Menu, Row, List, Divider } from "antd";
import "antd/dist/antd.css";
import React, { useCallback, useEffect, useState } from "react";
import { BrowserRouter, Link, Route, Switch } from "react-router-dom";
import Web3Modal from "web3modal";
import "./App.css";
import { Account, Address, Balance, Contract, Faucet, GasGauge, Header, Ramp, ThemeSwitch } from "./components";
import { INFURA_ID, NETWORK, NETWORKS } from "./constants";
import { Transactor } from "./helpers";
import {
  useBalance,
  useContractLoader,
  useContractReader,
  useGasPrice,
  useOnBlock,
  useUserProviderAndSigner,
} from "eth-hooks";
import { useEventListener } from "eth-hooks/events/useEventListener";
import { useExchangeEthPrice } from "eth-hooks/dapps/dex";
// import Hints from "./Hints";
import { ExampleUI, Hints, Subgraph } from "./views";

import { useContractConfig } from "./hooks";
import Portis from "@portis/web3";
import Fortmatic from "fortmatic";
import Authereum from "authereum";
import humanizeDuration from "humanize-duration";

const { ethers } = require("ethers");
/*
    Welcome to 🏗 scaffold-eth !

    Code:
    https://github.com/austintgriffith/scaffold-eth

    Support:
    https://t.me/joinchat/KByvmRe5wkR-8F_zz6AjpA
    or DM @austingriffith on Twitter or Telegram

    You should get your own Infura.io ID and put it in `constants.js`
    (this is your connection to the main Ethereum network for ENS etc.)


    🌏 EXTERNAL CONTRACTS:
    You can also bring in contract artifacts in `constants.js`
    (and then use the `useExternalContractLoader()` hook!)
*/

/// 📡 What chain are your contracts deployed to?
const targetNetwork = NETWORKS.localhost; // <------- select your target frontend network (localhost, rinkeby, xdai, mainnet)

// 😬 Sorry for all the console logging
const DEBUG = true;
const NETWORKCHECK = true;

// 🛰 providers
if (DEBUG) console.log("📡 Connecting to Mainnet Ethereum");
// const mainnetProvider = getDefaultProvider("mainnet", { infura: INFURA_ID, etherscan: ETHERSCAN_KEY, quorum: 1 });
// const mainnetProvider = new InfuraProvider("mainnet",INFURA_ID);
//
// attempt to connect to our own scaffold eth rpc and if that fails fall back to infura...
// Using StaticJsonRpcProvider as the chainId won't change see https://github.com/ethers-io/ethers.js/issues/901
const scaffoldEthProvider = navigator.onLine
  ? new ethers.providers.StaticJsonRpcProvider("https://rpc.scaffoldeth.io:48544")
  : null;
const poktMainnetProvider = navigator.onLine
  ? new ethers.providers.StaticJsonRpcProvider(
      "https://eth-mainnet.gateway.pokt.network/v1/lb/611156b4a585a20035148406",
    )
  : null;
const mainnetInfura = navigator.onLine
  ? new ethers.providers.StaticJsonRpcProvider("https://mainnet.infura.io/v3/" + INFURA_ID)
  : null;
// ( ⚠️ Getting "failed to meet quorum" errors? Check your INFURA_ID

// 🏠 Your local provider is usually pointed at your local blockchain
const localProviderUrl = targetNetwork.rpcUrl;
// as you deploy to other networks you can set REACT_APP_PROVIDER=https://dai.poa.network in packages/react-app/.env
const localProviderUrlFromEnv = process.env.REACT_APP_PROVIDER ? process.env.REACT_APP_PROVIDER : localProviderUrl;
if (DEBUG) console.log("🏠 Connecting to provider:", localProviderUrlFromEnv);
const localProvider = new ethers.providers.StaticJsonRpcProvider(localProviderUrlFromEnv);

// 🔭 block explorer URL
const blockExplorer = targetNetwork.blockExplorer;

// Coinbase walletLink init
const walletLink = new WalletLink({
  appName: "coinbase",
});

// WalletLink provider
const walletLinkProvider = walletLink.makeWeb3Provider(`https://mainnet.infura.io/v3/${INFURA_ID}`, 1);

// Portis ID: 6255fb2b-58c8-433b-a2c9-62098c05ddc9
/*
  Web3 modal helps us "connect" external wallets:
*/
const web3Modal = new Web3Modal({
  network: "mainnet", // Optional. If using WalletConnect on xDai, change network to "xdai" and add RPC info below for xDai chain.
  cacheProvider: true, // optional
  theme: "light", // optional. Change to "dark" for a dark theme.
  providerOptions: {
    walletconnect: {
      package: WalletConnectProvider, // required
      options: {
        bridge: "https://polygon.bridge.walletconnect.org",
        infuraId: INFURA_ID,
        rpc: {
          1: `https://mainnet.infura.io/v3/${INFURA_ID}`, // mainnet // For more WalletConnect providers: https://docs.walletconnect.org/quick-start/dapps/web3-provider#required
          42: `https://kovan.infura.io/v3/${INFURA_ID}`,
          100: "https://dai.poa.network", // xDai
        },
      },
    },
    portis: {
      display: {
        logo: "https://user-images.githubusercontent.com/9419140/128913641-d025bc0c-e059-42de-a57b-422f196867ce.png",
        name: "Portis",
        description: "Connect to Portis App",
      },
      package: Portis,
      options: {
        id: "6255fb2b-58c8-433b-a2c9-62098c05ddc9",
      },
    },
    fortmatic: {
      package: Fortmatic, // required
      options: {
        key: "pk_live_5A7C91B2FC585A17", // required
      },
    },
    // torus: {
    //   package: Torus,
    //   options: {
    //     networkParams: {
    //       host: "https://localhost:8545", // optional
    //       chainId: 1337, // optional
    //       networkId: 1337 // optional
    //     },
    //     config: {
    //       buildEnv: "development" // optional
    //     },
    //   },
    // },
    "custom-walletlink": {
      display: {
        logo: "https://play-lh.googleusercontent.com/PjoJoG27miSglVBXoXrxBSLveV6e3EeBPpNY55aiUUBM9Q1RCETKCOqdOkX2ZydqVf0",
        name: "Coinbase",
        description: "Connect to Coinbase Wallet (not Coinbase App)",
      },
      package: walletLinkProvider,
      connector: async (provider, _options) => {
        await provider.enable();
        return provider;
      },
    },
    authereum: {
      package: Authereum, // required
    },
  },
});

function App(props) {
  const mainnetProvider =
    poktMainnetProvider && poktMainnetProvider._isProvider
      ? poktMainnetProvider
      : scaffoldEthProvider && scaffoldEthProvider._network
      ? scaffoldEthProvider
      : mainnetInfura;

  const [injectedProvider, setInjectedProvider] = useState();
  const [address, setAddress] = useState();

  const logoutOfWeb3Modal = async () => {
    await web3Modal.clearCachedProvider();
    if (injectedProvider && injectedProvider.provider && typeof injectedProvider.provider.disconnect == "function") {
      await injectedProvider.provider.disconnect();
    }
    setTimeout(() => {
      window.location.reload();
    }, 1);
  };

  /* 💵 This hook will get the price of ETH from 🦄 Uniswap: */
  const price = useExchangeEthPrice(targetNetwork, mainnetProvider);

  /* 🔥 This hook will get the price of Gas from ⛽️ EtherGasStation */
  const gasPrice = useGasPrice(targetNetwork, "fast");
  // Use your injected provider from 🦊 Metamask or if you don't have it then instantly generate a 🔥 burner wallet.
  const userProviderAndSigner = useUserProviderAndSigner(injectedProvider, localProvider);
  const userSigner = userProviderAndSigner.signer;

  useEffect(() => {
    async function getAddress() {
      if (userSigner) {
        const newAddress = await userSigner.getAddress();
        setAddress(newAddress);
      }
    }
    getAddress();
  }, [userSigner]);

  // You can warn the user if you would like them to be on a specific network
  const localChainId = localProvider && localProvider._network && localProvider._network.chainId;
  const selectedChainId =
    userSigner && userSigner.provider && userSigner.provider._network && userSigner.provider._network.chainId;

  // For more hooks, check out 🔗eth-hooks at: https://www.npmjs.com/package/eth-hooks

  // The transactor wraps transactions and provides notificiations
  const tx = Transactor(userSigner, gasPrice);

  // Faucet Tx can be used to send funds from the faucet
  const faucetTx = Transactor(localProvider, gasPrice);

  // 🏗 scaffold-eth is full of handy hooks like this one to get your balance:
  const yourLocalBalance = useBalance(localProvider, address);

  // Just plug in different 🛰 providers to get your balance on different chains:
  const yourMainnetBalance = useBalance(mainnetProvider, address);

  const contractConfig = useContractConfig();

  // Load in your local 📝 contract and read a value from it:
  const readContracts = useContractLoader(localProvider, contractConfig);

  // If you want to make 🔐 write transactions to your contracts, use the userSigner:
  const writeContracts = useContractLoader(userSigner, contractConfig, localChainId);

  // EXTERNAL CONTRACT EXAMPLE:
  //
  // If you want to bring in the mainnet DAI contract it would look like:
  const mainnetContracts = useContractLoader(mainnetProvider, contractConfig);

  // If you want to call a function on a new block
  useOnBlock(mainnetProvider, () => {
    console.log(`⛓ A new mainnet block is here: ${mainnetProvider._lastBlockNumber}`);
  });

  // Then read your DAI balance like:
  const myMainnetDAIBalance = useContractReader(mainnetContracts, "DAI", "balanceOf", [
    "0x34aA3F359A9D614239015126635CE7732c18fDF3",
  ]);

  //keep track of contract balance to know how much has been staked total:
  const stakerContractBalance = useBalance(
    localProvider,
    readContracts && readContracts.Staker ? readContracts.Staker.address : null,
  );
  if (DEBUG) console.log("💵 stakerContractBalance", stakerContractBalance);

  const rewardRatePerSecond = useContractReader(readContracts, "Staker", "rewardRatePerSecond");
  console.log("💵 Reward Rate:", rewardRatePerSecond);

  // ** keep track of a variable from the contract in the local React state:
  const balanceStaked = useContractReader(readContracts, "Staker", "balances", [address]);
  console.log("💸 balanceStaked:", balanceStaked);

  // ** 📟 Listen for broadcast events
  const stakeEvents = useEventListener(readContracts, "Staker", "Stake", localProvider, 1);
  console.log("📟 stake events:", stakeEvents);

  const receiveEvents = useEventListener(readContracts, "Staker", "Received", localProvider, 1);
  console.log("📟 receive events:", receiveEvents);

  // ** keep track of a variable from the contract in the local React state:
  const claimPeriodLeft = useContractReader(readContracts, "Staker", "claimPeriodLeft");
  console.log("⏳ Claim Period Left:", claimPeriodLeft);

  const withdrawalTimeLeft = useContractReader(readContracts, "Staker", "withdrawalTimeLeft");
  console.log("⏳ Withdrawal Time Left:", withdrawalTimeLeft);


  // ** Listen for when the contract has been 'completed'
  const complete = useContractReader(readContracts, "ExampleExternalContract", "completed");
  console.log("✅ complete:", complete);

  const exampleExternalContractBalance = useBalance(
    localProvider,
    readContracts && readContracts.ExampleExternalContract ? readContracts.ExampleExternalContract.address : null,
  );
  if (DEBUG) console.log("💵 exampleExternalContractBalance", exampleExternalContractBalance);

  let completeDisplay = "";
  if (complete) {
    completeDisplay = (
      <div style={{padding: 64, backgroundColor: "#eeffef", fontWeight: "bold", color: "rgba(0, 0, 0, 0.85)" }} >
        -- 💀 Staking App Fund Repatriation Executed 🪦 --
        <Balance balance={exampleExternalContractBalance} fontSize={32} /> ETH locked!
      </div>
    );
  }

  /*
  const addressFromENS = useResolveName(mainnetProvider, "austingriffith.eth");
  console.log("🏷 Resolved austingriffith.eth as:", addressFromENS)
  */

  //
  // 🧫 DEBUG 👨🏻‍🔬
  //
  useEffect(() => {
    if (
      DEBUG &&
      mainnetProvider &&
      address &&
      selectedChainId &&
      yourLocalBalance &&
      yourMainnetBalance &&
      readContracts &&
      writeContracts &&
      mainnetContracts
    ) {
      console.log("_____________________________________ 🏗 scaffold-eth _____________________________________");
      console.log("🌎 mainnetProvider", mainnetProvider);
      console.log("🏠 localChainId", localChainId);
      console.log("👩‍💼 selected address:", address);
      console.log("🕵🏻‍♂️ selectedChainId:", selectedChainId);
      console.log("💵 yourLocalBalance", yourLocalBalance ? ethers.utils.formatEther(yourLocalBalance) : "...");
      console.log("💵 yourMainnetBalance", yourMainnetBalance ? ethers.utils.formatEther(yourMainnetBalance) : "...");
      console.log("📝 readContracts", readContracts);
      console.log("🌍 DAI contract on mainnet:", mainnetContracts);
      console.log("💵 yourMainnetDAIBalance", myMainnetDAIBalance);
      console.log("🔐 writeContracts", writeContracts);
    }
  }, [
    mainnetProvider,
    address,
    selectedChainId,
    yourLocalBalance,
    yourMainnetBalance,
    readContracts,
    writeContracts,
    mainnetContracts,
  ]);

  let networkDisplay = "";
  if (NETWORKCHECK && localChainId && selectedChainId && localChainId !== selectedChainId) {
    const networkSelected = NETWORK(selectedChainId);
    const networkLocal = NETWORK(localChainId);
    if (selectedChainId === 1337 && localChainId === 31337) {
      networkDisplay = (
        <div style={{ zIndex: 2, position: "absolute", right: 0, top: 60, padding: 16 }}>
          <Alert
            message="⚠️ Wrong Network ID"
            description={
              <div>
                You have <b>chain id 1337</b> for localhost and you need to change it to <b>31337</b> to work with
                HardHat.
                <div>(MetaMask -&gt; Settings -&gt; Networks -&gt; Chain ID -&gt; 31337)</div>
              </div>
            }
            type="error"
            closable={false}
          />
        </div>
      );
    } else {
      networkDisplay = (
        <div style={{ zIndex: 2, position: "absolute", right: 0, top: 60, padding: 16 }}>
          <Alert
            message="⚠️ Wrong Network"
            description={
              <div>
                You have <b>{networkSelected && networkSelected.name}</b> selected and you need to be on{" "}
                <Button
                  onClick={async () => {
                    const ethereum = window.ethereum;
                    const data = [
                      {
                        chainId: "0x" + targetNetwork.chainId.toString(16),
                        chainName: targetNetwork.name,
                        nativeCurrency: targetNetwork.nativeCurrency,
                        rpcUrls: [targetNetwork.rpcUrl],
                        blockExplorerUrls: [targetNetwork.blockExplorer],
                      },
                    ];
                    console.log("data", data);

                    let switchTx;
                    // https://docs.metamask.io/guide/rpc-api.html#other-rpc-methods
                    try {
                      switchTx = await ethereum.request({
                        method: "wallet_switchEthereumChain",
                        params: [{ chainId: data[0].chainId }],
                      });
                    } catch (switchError) {
                      // not checking specific error code, because maybe we're not using MetaMask
                      try {
                        switchTx = await ethereum.request({
                          method: "wallet_addEthereumChain",
                          params: data,
                        });
                      } catch (addError) {
                        // handle "add" error
                      }
                    }

                    if (switchTx) {
                      console.log(switchTx);
                    }
                  }}
                >
                  <b>{networkLocal && networkLocal.name}</b>
                </Button>
              </div>
            }
            type="error"
            closable={false}
          />
        </div>
      );
    }
  } else {
    networkDisplay = (
      <div style={{ zIndex: -1, position: "absolute", right: 154, top: 28, padding: 16, color: targetNetwork.color }}>
        {targetNetwork.name}
      </div>
    );
  }

  const loadWeb3Modal = useCallback(async () => {
    const provider = await web3Modal.connect();
    setInjectedProvider(new ethers.providers.Web3Provider(provider));

    provider.on("chainChanged", chainId => {
      console.log(`chain changed to ${chainId}! updating providers`);
      setInjectedProvider(new ethers.providers.Web3Provider(provider));
    });

    provider.on("accountsChanged", () => {
      console.log(`account changed!`);
      setInjectedProvider(new ethers.providers.Web3Provider(provider));
    });

    // Subscribe to session disconnection
    provider.on("disconnect", (code, reason) => {
      console.log(code, reason);
      logoutOfWeb3Modal();
    });
  }, [setInjectedProvider]);

  useEffect(() => {
    if (web3Modal.cachedProvider) {
      loadWeb3Modal();
    }
  }, [loadWeb3Modal]);

  const [route, setRoute] = useState();
  useEffect(() => {
    setRoute(window.location.pathname);
  }, [setRoute]);

  let faucetHint = "";
  const faucetAvailable = localProvider && localProvider.connection && targetNetwork.name.indexOf("local") !== -1;

  const [faucetClicked, setFaucetClicked] = useState(false);
  if (
    !faucetClicked &&
    localProvider &&
    localProvider._network &&
    localProvider._network.chainId === 31337 &&
    yourLocalBalance &&
    ethers.utils.formatEther(yourLocalBalance) <= 0
  ) {
    faucetHint = (
      <div style={{ padding: 16 }}>
        <Button
          type="primary"
          onClick={() => {
            faucetTx({
              to: address,
              value: ethers.utils.parseEther("0.01"),
            });
            setFaucetClicked(true);
          }}
        >
          💰 Grab funds from the faucet ⛽️
        </Button>
      </div>
    );
  }

  return (
    <div className="App">
      {/* ✏️ Edit the header and change the title to your project name */}
      <Header />
      {networkDisplay}
      <BrowserRouter>
        <Menu style={{ textAlign: "center" }} selectedKeys={[route]} mode="horizontal">
          <Menu.Item key="/">
            <Link
              onClick={() => {
                setRoute("/");
              }}
              to="/"
            >
              Staker UI
            </Link>
          </Menu.Item>
          <Menu.Item key="/contracts">
            <Link
              onClick={() => {
                setRoute("/contracts");
              }}
              to="/contracts"
            >
              Debug Contracts
            </Link>
          </Menu.Item>
        </Menu>

        <Switch>
          <Route exact path="/">
            {completeDisplay}

            <div style={{ padding: 8, marginTop: 16 }}>
              <div>Staker Contract:</div>
              <Address value={readContracts && readContracts.Staker && readContracts.Staker.address} />
            </div>

            <Divider />

            <div style={{ padding: 8, marginTop: 16 }}>
              <div>Reward Rate Per Second:</div>
              <Balance balance={rewardRatePerSecond} fontSize={64} /> ETH
            </div>

            <Divider />

            <div style={{ padding: 8, marginTop: 16, fontWeight: "bold" }}>
              <div>Claim Period Left:</div>
              {claimPeriodLeft && humanizeDuration(claimPeriodLeft.toNumber() * 1000)}
            </div>

            <div style={{ padding: 8, marginTop: 16, fontWeight: "bold"}}>
              <div>Withdrawal Period Left:</div>
              {withdrawalTimeLeft && humanizeDuration(withdrawalTimeLeft.toNumber() * 1000)}
            </div>

            <Divider />

            <div style={{ padding: 8, fontWeight: "bold"}}>
              <div>Total Available ETH in Contract:</div>
              <Balance balance={stakerContractBalance} fontSize={64} />
            </div>

            <Divider />

            <div style={{ padding: 8,fontWeight: "bold" }}>
              <div>ETH Locked 🔒 in Staker Contract:</div>
              <Balance balance={balanceStaked} fontSize={64} />
            </div>

            <div style={{ padding: 8 }}>
              <Button
                type={"default"}
                onClick={() => {
                  tx(writeContracts.Staker.execute());
                }}
              >
                📡 Execute!
              </Button>
            </div>

            <div style={{ padding: 8 }}>
              <Button
                type={"default"}
                onClick={() => {
                  tx(writeContracts.Staker.withdraw());
                }}
              >
                🏧 Withdraw
              </Button>
            </div>

            <div style={{ padding: 8 }}>
              <Button
                type={balanceStaked ? "success" : "primary"}
                onClick={() => {
                  tx(writeContracts.Staker.stake({ value: ethers.utils.parseEther("0.5") }));
                }}
              >
                🥩 Stake 0.5 ether!
              </Button>
            </div>

            {/*
                🎛 this scaffolding is full of commonly used components
                this <Contract/> component will automatically parse your ABI
                and give you a form to interact with it locally
            */}

            {/* uncomment for a second contract:
            <Contract
              name="SecondContract"
              signer={userProvider.getSigner()}
              provider={localProvider}
              address={address}
              blockExplorer={blockExplorer}
              contractConfig={contractConfig}
            />
            */}
          </Route>
          <Route path="/contracts">
            <Contract
              name="Staker"
              signer={userSigner}
              provider={localProvider}
              address={address}
              blockExplorer={blockExplorer}
              contractConfig={contractConfig}
            />
            <Contract
              name="ExampleExternalContract"
              signer={userSigner}
              provider={localProvider}
              address={address}
              blockExplorer={blockExplorer}
              contractConfig={contractConfig}
            />
          </Route>
        </Switch>
      </BrowserRouter>

      <ThemeSwitch />

      {/* 👨‍💼 Your account is in the top right with a wallet at connect options */}
      <div style={{ position: "fixed", textAlign: "right", right: 0, top: 0, padding: 10 }}>
        <Account
          address={address}
          localProvider={localProvider}
          userSigner={userSigner}
          mainnetProvider={mainnetProvider}
          price={price}
          web3Modal={web3Modal}
          loadWeb3Modal={loadWeb3Modal}
          logoutOfWeb3Modal={logoutOfWeb3Modal}
          blockExplorer={blockExplorer}
        />
        {faucetHint}
      </div>

      <div style={{ marginTop: 32, opacity: 0.5 }}>
        {/* Add your address here */}
        Created by <Address value={"Your...address"} ensProvider={mainnetProvider} fontSize={16} />
      </div>

      <div style={{ marginTop: 32, opacity: 0.5 }}>
        <a target="_blank" style={{ padding: 32, color: "#000" }} href="https://github.com/scaffold-eth/scaffold-eth">
          🍴 Fork me!
        </a>
      </div>

      {/* 🗺 Extra UI like gas price, eth price, faucet, and support: */}
      <div style={{ position: "fixed", textAlign: "left", left: 0, bottom: 20, padding: 10 }}>
        <Row align="middle" gutter={[4, 4]}>
          <Col span={8}>
            <Ramp price={price} address={address} networks={NETWORKS} />
          </Col>

          <Col span={8} style={{ textAlign: "center", opacity: 0.8 }}>
            <GasGauge gasPrice={gasPrice} />
          </Col>
          <Col span={8} style={{ textAlign: "center", opacity: 1 }}>
            <Button
              onClick={() => {
                window.open("https://t.me/joinchat/KByvmRe5wkR-8F_zz6AjpA");
              }}
              size="large"
              shape="round"
            >
              <span style={{ marginRight: 8 }} role="img" aria-label="support">
                💬
              </span>
              Support
            </Button>
          </Col>
        </Row>

        <Row align="middle" gutter={[4, 4]}>
          <Col span={24}>
            {
              /*  if the local provider has a signer, let's show the faucet:  */
              faucetAvailable ? (
                <Faucet localProvider={localProvider} price={price} ensProvider={mainnetProvider} />
              ) : (
                ""
              )
            }
          </Col>
        </Row>
      </div>
    </div>
  );
}

export default App;

```

¡Impresionante!

Hemos trabajado juntos en un montón de nuevos componentes, tanto en términos de entornos de desarrollo, Solidity, y código frontend.

Comprueba que tu dApp funciona como esperabas.

 1. ¿La dApp cuenta con depositos de un solo uso?
 2. ¿Se respetan las condiciones de retirada/repatriación de fondos? Pulsa ``yarn deploy --reset`` varias veces para comprobar cada ventana de tiempo.

### Hora del desafío
Bien, ahora es el momento de la mejor parte. Voy a dejarte con unos cuantos retos de extensión para que los pruebes por tu cuenta, ¡para ver si entiendes completamente lo que has aprendido aquí!

1. Actualiza el mecanismo de intereses en el contrato de ``Staker.sol`` para que recibas una cantidad "no lineal" de ETH basada en los bloques entre el depósito y la retirada

> 📘¡Sugiero implementar una función exponencial básica!

2. Permitir a los usuarios depositar cualquier cantidad arbitraria de ETH en el contrato inteligente, no sólo 0,5 ETH.

3. En lugar de utilizar el contrato vainilla ``ExampleExternalContract``, implementa una función en Staker.sol que te permita recuperar el ETH bloqueado en ``ExampleExternalContract`` y volver a depositarlo en el contrato ``Staker``.

* Asegúrate de poner en la "lista blanca" sólo una dirección para llamar a esta nueva función y controlar su uso.
* Asegúrese de crear lógica/eliminar código existente para garantizar que los usuarios puedan interactuar con el contrato ``Staker`` una y otra vez. Queremos ser capaces de hacer ping-pong desde Staker -> ``ExampleExternalContract`` repetidamente.

Una vez que hayas terminado con el desafío, tuitea sobre él etiquetando a [@AlchemyPlatform](https://twitter.com/AlchemyPlatform) en Twitter y al autor [@crypt0zeke](https://twitter.com/crypt0zeke) usando el hashtag [#roadtoweb3.](https://twitter.com/search?q=%23RoadToWeb3)

ADEMÁS, canjea tu desafío por una Prueba de Conocimiento NFT enviando este formulario aquí: https://forms.gle/uYhMS3brabNg3RVs7

Paz y amor. ¡Feliz desarrollo! 🏗️🚀
![image](https://user-images.githubusercontent.com/48574383/231923343-dd909d2c-960b-4ae5-aa65-3308d9c5297b.png)
