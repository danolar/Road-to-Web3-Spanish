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
