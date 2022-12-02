# Como construir una Aplicación de finanzas descentralizadas (DeFi dApp) "Regálame un café"

La tecnología Blockchain es increíble porque nos da la capacidad de programar el dinero utilizando código y software. Con unas pocas líneas de código, es posible construir todo tipo de aplicaciones y protocolos que pueden crear nuevas oportunidades para personas de todo el mundo.

**"Buy Me A Coffee"** es un popular sitio web que creadores, educadores, animadores y todo tipo de personas utilizan para crear una página de destino en la que cualquiera puede enviar alguna cantidad de dinero como agradecimiento por sus servicios. Sin embargo, para poder utilizarla, debes tener una cuenta bancaria y una tarjeta de crédito. ¡No todo el mundo tiene eso!
Una de las ventajas de las aplicaciones descentralizadas construidas sobre una cadena de bloques es que cualquier persona de todo el mundo puede acceder a la aplicación utilizando sólo un monedero de Ethereum, que cualquiera puede configurar gratuitamente en menos de 1 minuto. Vamos a ver cómo podemos utilizar esto en nuestro beneficio.

En este tutorial, vas a aprender a **desarrollar y desplegar un contrato inteligente descentralizado "Buy Me a Coffee"** que permite a los visitantes enviarte ETH (falsos) como propinas y dejar mensajes agradables, usando **Alchemy, Hardhat, Ethers.js, y Ethereum Goerli.**

Al final de este tutorial, aprenderás a hacer lo siguiente:

+ Utilizar el entorno de desarrollo de **Hardhat** para construir, probar y desplegar nuestro contrato inteligente.
+ Conectar tu billetera de MetaMask a la red de prueba **Goerli** utilizando un punto final **RPC de Alchemy.**
+ Obtener **Goerli** ETH gratis desde **goerlifaucet.com.**
+ Utilizar **Ethers.js** para interactuar con el contrato inteligente desplegado.
+ Construir un sitio web frontend para la aplicación descentralizada con **Replit.**

**Próximamente versión de video**


## Requisitos previos

Para realizar el resto de este tutorial, necesitas tener

+ npm (npx) versión 8.5.5
+ la versión 16.13.1 de node
+ Una cuenta de Alchemy [(¡regístrate aquí gratis!)]( https://alchemy.com/?a=roadtoweb3weektwo)

Lo siguiente no es necesario, pero es extremadamente útil:

+ cierta familiaridad con la [línea de comandos]( https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Command_line)
+ cierta familiaridad con [JavaScript]( https://www.codecademy.com/learn/introduction-to-javascript)

¡Ahora vamos a empezar a construir nuestro contrato inteligente!

## Codifica el contrato inteligente BuyMeACoffee.sol

Referencia en Github:
https://github.com/alchemyplatform/RTW3-Week2-BuyMeACoffee-Contracts

Si has utilizado herramientas como **OpenZeppelin Wizard** y **Remix** antes, entonces ya estás preparado para utilizar **Hardhat.**

**Hardhat** es similar porque es un entorno de desarrollo y una herramienta de codificación, pero es un poco más personalizable y se ejecuta desde la interfaz de línea de comandos de tu propio ordenador en lugar de una aplicación de navegador.

Vamos a utilizar **Hardhat** para:

+ generar la plantilla del proyecto
+ probar nuestro código de contrato inteligente
+ desplegar en la red de prueba Goerli

¡Vamos allá!

Abre tu terminal y crea un nuevo directorio.

``
mkdir BuyMeACoffee-contracts
cd BuyMeACoffee-contracts
``

Dentro de este directorio, queremos iniciar un nuevo proyecto npm (la configuración por defecto está bien):

``
npm init -y
``

Esto debería crear un archivo ``package.json`` se debe parecer a esto:

```
thatguyintech@albert BuyMeACoffee-contracts % npm init -y
Wrote to /Users/thatguyintech/Documents/co/videos/week2/BuyMeACoffee-contracts/package.json:

{
  "name": "buymeacoffee-contracts",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Instalar hardhat:

``npm install --save-dev hardhat``

Ahora creamos un proyecto de ejemplo:

``npx hardhat``

![IMG1](https://files.readme.io/886d2ce-npx_hardhat.png)

Presiona ``Enter`` para aceptar toda la configuración predeterminada e instalar las dependencias del proyecto de ejemplo:(``hardhat``,`` @nomiclabs/hardhat-waffle``, ``ethereum-waffle``, ``chai``, ``@nomiclabs/hardhat-ethers``,`` ethers``).

En caso de que algo vaya mal con la instalación de la dependencia, intenta reinstalar ejecutando este comando:

```
npm install --save-dev hardhat@^2.9.3 @nomiclabs/hardhat-waffle@^2.0.0 ethereum-waffle@^3.0.0 chai@^4.2.0 @nomiclabs/hardhat-ethers@^2.0.0 ethers@^5.0.0
```

El directorio de tu proyecto debería tener ahora un aspecto similar al siguiente (Utilizando [tree](https://www.cyberciti.biz/faq/linux-show-directory-structure-command-line/) para visualizarlo):

```
thatguyintech@albert BuyMeACoffee-contracts % tree -C -L 1
.
├── README.md
├── contracts
├── hardhat.config.js
├── node_modules
├── package-lock.json
├── package.json
├── scripts
└── test
```

Las carpetas y archivos importantes son:

+ ``contracts`` - carpeta donde se alojan tus contratos inteligentes
  + en este proyecto sólo crearemos una, para organizar nuestra lógica de ``BuyMeACoffee``
+ ``scripts`` - carpeta donde se alojan tus scripts javscript de Hardhat
  + escribiremos la lógica de ``deploy``
  + ejemplo de script de ``buy-coffee``
  + y un script de ``withdraw `` para cobrar nuestras propinas
+ ``hardhat.config.js`` - archivo de configuración con los ajustes para la versión de solidity y el despliegue

Ahora puedes usar cualquier editor de código para abrir la carpeta del proyecto. Me gusta usar VSCode.

![IMG2](https://files.readme.io/4e2aa36-vs.png)

Verás que hay una serie de archivos ya autogenerados a través de la herramienta de proyectos de ejemplo de Hardhat. Vamos a sustituirlos todos, empezando por el contrato ``Greeter.sol.``

1.Cambia el nombre del archivo del contrato a ``BuyMeACoffee.sol``
2.Sustituye el código del contrato por el siguiente:

```
//SPDX-License-Identifier: Unlicense

// contracts/BuyMeACoffee.sol
pragma solidity ^0.8.0;

// Switch this to your own contract address once deployed, for bookkeeping!
// Example Contract Address on Goerli: 0xDBa03676a2fBb6711CB652beF5B7416A53c1421D

contract BuyMeACoffee {
    // Event to emit when a Memo is created.
    event NewMemo(
        address indexed from,
        uint256 timestamp,
        string name,
        string message
    );
    
    // Memo struct.
    struct Memo {
        address from;
        uint256 timestamp;
        string name;
        string message;
    }
    
    // Address of contract deployer. Marked payable so that
    // we can withdraw to this address later.
    address payable owner;

    // List of all memos received from coffee purchases.
    Memo[] memos;

    constructor() {
        // Store the address of the deployer as a payable address.
        // When we withdraw funds, we'll withdraw here.
        owner = payable(msg.sender);
    }

    /**
     * @dev fetches all stored memos
     */
    function getMemos() public view returns (Memo[] memory) {
        return memos;
    }

    /**
     * @dev buy a coffee for owner (sends an ETH tip and leaves a memo)
     * @param _name name of the coffee purchaser
     * @param _message a nice message from the purchaser
     */
    function buyCoffee(string memory _name, string memory _message) public payable {
        // Must accept more than 0 ETH for a coffee.
        require(msg.value > 0, "can't buy coffee for free!");

        // Add the memo to storage!
        memos.push(Memo(
            msg.sender,
            block.timestamp,
            _name,
            _message
        ));

        // Emit a NewMemo event with details about the memo.
        emit NewMemo(
            msg.sender,
            block.timestamp,
            _name,
            _message
        );
    }

    /**
     * @dev send the entire balance stored in this contract to the owner
     */
    function withdrawTips() public {
        require(owner.send(address(this).balance));
    }
}

```


Tómate un tiempo para leer los comentarios de los contratos y ver si puedes entender lo que está pasando.

Voy a enumerar lo más destacado aquí:

+ Cuando desplegamos el contrato, el ``constructor`` guarda la dirección de la cartera que se encargó de desplegar dentro de una variable propietaria ``owner`` como dirección de pago ``payable``. Esto es útil para más adelante cuando queramos retirar las propinas recogidas por el contrato.
+ La función ``buyCoffee`` es la más importante del contrato. Acepta dos cadenas, un ``_name``, y un ``_message``, y también acepta ether debido al modificador “payable  ”. Utiliza las entradas ``_name`` y ``_message`` para crear una estructura Memo que se almacena en el blockchain. 
  + Cuando los visitantes llaman a la función ``buyCoffee``, deben enviar algo de ether debido a la declaración ``require(msg.value > 0)``. El ether se mantiene entonces en el ``balance`` del contrato hasta que se retira.
+ El array de ``memos`` contiene todos los structs de ``Memo`` generados por las compras de café.
+ Los eventos de registro ``NewMemo`` se emiten cada vez que se compra un café. Esto nos permite visualizar las nuevas compras de café desde nuestro sitio web.
+ ``withdrawTips`` es una función a la que cualquiera puede llamar, pero que sólo enviará dinero al que originalmente desplegó el contrato.
  + ``address(this).balance`` recupera el ether almacenado en el contrato
  + ``owner.send(...)`` es la sintaxis para crear una transacción de envío con ether
  + la declaración ``require(...)`` que envuelve todo está ahí para asegurar que si hay algún problema, la transacción se revierte y no se pierde nada
  + así es como obtenemos ``require(owner.send(address(this).balance))``

Armados con este código de contrato inteligente, ¡ahora podemos escribir un script para probar nuestra lógica!

## Crea un script buy-coffee.js para probar tu contrato

En la carpeta de ``scripts``, debería haber un script de ejemplo ``sample-script.js``. Vamos a cambiar el nombre de ese archivo a ``buy-coffee.js`` y a pegar el siguiente código:

```
const hre = require("hardhat");

// Returns the Ether balance of a given address.
async function getBalance(address) {
  const balanceBigInt = await hre.ethers.provider.getBalance(address);
  return hre.ethers.utils.formatEther(balanceBigInt);
}

// Logs the Ether balances for a list of addresses.
async function printBalances(addresses) {
  let idx = 0;
  for (const address of addresses) {
    console.log(`Address ${idx} balance: `, await getBalance(address));
    idx ++;
  }
}

// Logs the memos stored on-chain from coffee purchases.
async function printMemos(memos) {
  for (const memo of memos) {
    const timestamp = memo.timestamp;
    const tipper = memo.name;
    const tipperAddress = memo.from;
    const message = memo.message;
    console.log(`At ${timestamp}, ${tipper} (${tipperAddress}) said: "${message}"`);
  }
}

async function main() {
  // Get the example accounts we'll be working with.
  const [owner, tipper, tipper2, tipper3] = await hre.ethers.getSigners();

  // We get the contract to deploy.
  const BuyMeACoffee = await hre.ethers.getContractFactory("BuyMeACoffee");
  const buyMeACoffee = await BuyMeACoffee.deploy();

  // Deploy the contract.
  await buyMeACoffee.deployed();
  console.log("BuyMeACoffee deployed to:", buyMeACoffee.address);

  // Check balances before the coffee purchase.
  const addresses = [owner.address, tipper.address, buyMeACoffee.address];
  console.log("== start ==");
  await printBalances(addresses);

  // Buy the owner a few coffees.
  const tip = {value: hre.ethers.utils.parseEther("1")};
  await buyMeACoffee.connect(tipper).buyCoffee("Carolina", "You're the best!", tip);
  await buyMeACoffee.connect(tipper2).buyCoffee("Vitto", "Amazing teacher", tip);
  await buyMeACoffee.connect(tipper3).buyCoffee("Kay", "I love my Proof of Knowledge", tip);

  // Check balances after the coffee purchase.
  console.log("== bought coffee ==");
  await printBalances(addresses);

  // Withdraw.
  await buyMeACoffee.connect(owner).withdrawTips();

  // Check balances after withdrawal.
  console.log("== withdrawTips ==");
  await printBalances(addresses);

  // Check out the memos.
  console.log("== memos ==");
  const memos = await buyMeACoffee.getMemos();
  printMemos(memos);
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```
En este punto, el directorio de tu proyecto debería tener un aspecto similar al siguiente:

![IMG3](https://files.readme.io/66e9ef5-sample-script.js.png)

renombrar scripts/sample-script.js a scripts/buy-coffee.js y pegar el código de ejemplo

Tómate unos minutos para leer el código del script. Hay algunas funciones de utilidad definidas en la parte superior para hacer cosas como obtener los saldos de la cartera e imprimirlos.

La lógica principal del script está dentro de la función ``main()`` El código comentado muestra el flujo del script:

1.Obtenemos las cuentas de ejemplo con las que vamos a trabajar.
2.Obtenemos el contrato a desplegar.
3.Desplegar el contrato.
4.Comprobar los saldos antes de la compra de café.
5.Comprar al propietario algunos cafés.
6.Comprobar los saldos después de la compra de café.
7.Retirar.
8.Comprobar los saldos después de la retirada.
9.Comprobar los memos.

¡Este script prueba todas las funciones que implementamos en nuestro contrato inteligente! Es increíble.

También puedes notar que estamos haciendo llamadas interesantes como

+ ``hre.waffle.provider.getBalance``
+ ``hre.ethers.getContractFactory``
+ ``hre.ethers.utils.parseEther``
+ ``etc.``

En estas líneas de código aprovechamos el entorno de desarrollo de **Hardhat** (hre) junto con los plugins del SDK de **Ethers** y **Waffle** para acceder a la funcionalidad que nos permite leer los saldos de las cuentas de las carteras de blockchain, desplegar contratos y formatear los valores de la criptomoneda Ether.

No profundizaremos demasiado en ese código en este tutorial, pero puedes aprender más sobre ellos consultando la documentación de Hardhat y Ethers.js.

Basta de hablar. Ahora para la diversión, vamos a ejecutar el script:

```
npx hardhat run scripts/buy-coffee.js
```
Deberías ver la salida en tu terminal así:

```
thatguyintech@albert BuyMeACoffee-contracts % npx hardhat run scripts/buy-coffee.js
Compiled 1 Solidity file successfully
BuyMeACoffee deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
== start ==
Address 0 balance:  9999.99877086625
Address 1 balance:  10000.0
Address 2 balance:  0.0
== bought coffee ==
Address 0 balance:  9999.99877086625
Address 1 balance:  9998.999752902808629985
Address 2 balance:  3.0
== withdrawTips ==
Address 0 balance:  10002.998724967892122376
Address 1 balance:  9998.999752902808629985
Address 2 balance:  0.0
== memos ==
At 1652033688, Carolina (0x70997970C51812dc3A010C7d01b50e0d17dc79C8) said: "You're the best!"
At 1652033689, Vitto (0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC) said: "Amazing teacher"
At 1652033690, Kay (0x90F79bf6EB2c4f870365E785982E1f101E93b906) said: "I love my Proof of Knowledge"
```

Al comienzo del script (justo después del despliegue del contrato, ``deploy``), observa que la dirección ``0`` tiene ``9999,99877086625`` ETH. Esto se debe a que comenzó con 10k ETH como una de las direcciones precargadas de hardhat, pero tuvo que gastar una pequeña cantidad para desplegar a la blockchain local.

En el segundo paso ``== bought coffee ==``, la dirección 1 compra un café. Otros dos monederos que no se muestran TAMBIÉN compran cafés. En total, se compraron 3 cafés por una cantidad total de propina de ``3.0 ETH``. Se puede ver que la dirección 2 (que representa la dirección del contrato), mantiene ``3.0`` ETH.

Después de llamar a la función ``withdrawTips()`` en ``== withdrawTips ==``, el contrato vuelve a bajar a ``0`` ETH, y el desplegador original, también conocido como Dirección 0, ha ganado ahora algo de dinero y está sentado en ``10002,998724967892122376`` ETH.

¿¡Ya nos estamos divirtiendo!? ¿Te imaginas las propinas que vas a ganar? Yo sí.

¡Ahora vamos a implementar un script de despliegue aislado para mantener el despliegue real simple y también prepararnos para desplegar en la red de prueba de Goerli!

## Despliegue de su contrato inteligente BuyMeACoffe.sol en la red de prueba Ethereum Goerli utilizando Alchemy y MetaMask

Vamos a crear un nuevo archivo ``scripts/deploy.js`` que será súper simple, sólo para desplegar nuestro contrato en cualquier red que elijamos más adelante (elegiremos Goerli).

El archivo ``deploy.js`` debe tener este aspecto:

```
// scripts/deploy.js

const hre = require("hardhat");

async function main() {
  // We get the contract to deploy.
  const BuyMeACoffee = await hre.ethers.getContractFactory("BuyMeACoffee");
  const buyMeACoffee = await BuyMeACoffee.deploy();

  await buyMeACoffee.deployed();

  console.log("BuyMeACoffee deployed to:", buyMeACoffee.address);
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Para repasar la estructura del proyecto, ahora tenemos un contrato inteligente y dos scripts de hardware:

![](https://files.readme.io/3b56180-buyCoffee.png)

```
npx hardhat run scripts/deploy.js
```
Verás una sola línea impresa:

```
BuyMeACoffee deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```
Lo interesante es que, si lo ejecutas una y otra vez, verás la misma dirección de despliegue exacta cada vez:

```
thatguyintech@albert BuyMeACoffee-contracts % npx hardhat run scripts/deploy.js
BuyMeACoffee deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
thatguyintech@albert BuyMeACoffee-contracts % npx hardhat run scripts/deploy.js
BuyMeACoffee deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
thatguyintech@albert BuyMeACoffee-contracts % npx hardhat run scripts/deploy.js
BuyMeACoffee deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```
¿Por qué? Porque cuando ejecutas el script, la red por defecto que utiliza la herramienta Hardhat es una red de desarrollo local, justo en tu ordenador. Es rápido y determinista, y está muy bien para una rápida comprobación de la cordura.

Sin embargo, para poder desplegar una red de prueba que se ejecuta a través de Internet con nodos en todo el mundo, tenemos que cambiar nuestro archivo de configuración de Hardhat para darnos la opción.

Aquí es donde entra el archivo ``hardhat.config.json.``

Una palabra rápida de advertencia antes de sumergirse en:


> ### 📘¡LAS CONFIGURACIONES SON DIFÍCILES! ¡MANTÉN TUS SECRETOS A SALVO!
>  
> Hay todo tipo de pequeños detalles que pueden salir mal, y las cosas cambian todo el tiempo. Lo más peligroso son los valores secretos, por ejemplo, tu clave privada MetaMask y tu URL Alchemy.
>     
> Si algo no te funciona, comprueba el StackExchange de Ethereum, el Discord de Alchemy o busca tus errores en Google.
>     
> Y no compartas nunca tus secretos. ¡Tus claves, tus monedas!


Cuando abras tu archivo`` hardhat.config.js``, verá un ejemplo de código de despliegue. Bórralo y pega esta versión:

```
// hardhat.config.js

require("@nomiclabs/hardhat-ethers");
require("@nomiclabs/hardhat-waffle");
require("dotenv").config()

// You need to export an object to set up your config
// Go to https://hardhat.org/config/ to learn more

const GOERLI_URL = process.env.GOERLI_URL;
const PRIVATE_KEY = process.env.PRIVATE_KEY;

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
  solidity: "0.8.4",
  networks: {
    goerli: {
      url: GOERLI_URL,
      accounts: [PRIVATE_KEY]
    }
  }
};
```

Aquí ocurren un par de cosas:

+ Al importar ``hardhat-ethers``, ``hardhat-waffle``, y ``dotenv`` en la parte superior del archivo de configuración, todo nuestro proyecto **Hardhat** tendrá acceso a esas dependencias.
+ Sé que no hemos introducido ``dotenv`` todavía, es una herramienta importante de la que hablaremos en un momento.
+ ``process.env.GOERLI_URL`` y ``process.env.PRIVATE_KEY`` son la forma de acceder a las variables de entorno para utilizarlas en nuestro archivo de configuración sin exponer los valores secretos.
+ Dentro de ``modules.exports``, estamos usando la versión 0.8.4 del compilador solidity. Diferentes versiones del compilador soportan diferentes características y conjuntos de sintaxis, por lo que es importante hacer coincidir esta versión con la declaración pragma en la parte superior de nuestro contrato inteligente ``BuyMeACoffee.sol``.
  + Si vuelves a ese archivo puedes revisar que la declaración ``pragma solidity ^0.8.0``;. En este caso, aunque los números no coincidan exactamente, no pasa nada porque el símbolo de quilates ``^`` significa que cualquier versión que sea mayor o igual a ``0.8.0`` funcionará.
+ También en el ``modules.exports``, definimos una configuración de redes que contiene una configuración de red de prueba para ``goerli``.
Ahora, antes de que podamos hacer nuestro despliegue, necesitamos asegurarnos de que tenemos una última herramienta instalada, el módulo ``dotenv``. Como su nombre indica, ``dotenv`` nos ayuda a conectar un archivo ``.env`` con el resto de nuestro proyecto. Vamos a instalarlo.
Instala ``dotenv``:

```
npm install dotenv
```

Crea el archivo ``.env``

```
touch .env

```

Rellena el archivo ``.env`` con las variables que necesitamos:

```
GOERLI_URL=https://eth-goerli.alchemyapi.io/v2/<your api key>
GOERLI_API_KEY=<your api key>
PRIVATE_KEY=<your metamask api key>
```

Te darás cuenta de que no he revelado ninguno de mis secretos. Sí. La seguridad es lo primero. Sin embargo, está totalmente bien que pongas ese archivo, siempre y cuando también tengas un ``.gitignore`` que asegure que no empujes accidentalmente el archivo al control de versiones. Asegúrate de que el ``.env`` está listado en tu ``.gitignore``

```
node_modules
.env
coverage
coverage.json
typechain

#Hardhat files
cache
artifacts
```

Además, para obtener lo que necesitas para las variables de entorno, puedes utilizar los siguientes recursos:

+ ``GOERLI_URL`` - regístrate en una cuenta en [Alchemy](https://alchemy.com/?a=roadtoweb3weektwo), crea una aplicación Ethereum -> Goerli, y utiliza la URL HTTP
+ ``GOERLI_API_KEY`` - desde tu misma aplicación Alchemy Ethereum Goerli, puedes obtener la última parte de la URL, y esa será tu API KEY
+ ``PRIVATE_KEY`` - Sigue estas [instrucciones de MetaMask](https://metamask.zendesk.com/hc/en-us/articles/360015289632-How-to-Export-an-Account-Private-Key) para exportar tu clave privada.
Ahora, con ``dotenv`` instalado y su archivo ``.env`` completo, estamos CASI listos para desplegar en la red de pruebas de Goerli.

Lo último que tenemos que hacer es asegurarnos de que tienes algo de Goerli ETH. Este es el ether falso que le permite practicar haciendo cosas en la red de prueba Goerli, que es una especie de zona de práctica para la construcción de aplicaciones Ethereum. De esta manera no tienes que gastar dinero real en la red principal de Ethereum.

Ve a https://www.goerlifaucet.com e inicia sesión con tu cuenta de Alchemy para conseguir algo de ether de prueba gratis.

¡Ahora podemos desplegar!

Ejecuta el script de despliegue, esta vez añadiendo una bandera especial para usar la red Goerli:

```
npx hardhat run scripts/deploy.js --network goerli
```

Si te encuentras con algún error aquí, por ejemplo, el ``error HH8``, entonces te recomiendo que busques en Google y en Stack Overflow o en Ethereum Stackexchange para encontrar soluciones. Es común encontrarse con esos problemas cuando algo en tu ``hardhat.config.js``, ``.env``, o tu módulo ``dotenv`` no está configurado correctamente.

Si todo va bien, deberías poder ver tu dirección de contrato registrada en la consola después de unos segundos:

```
BuyMeACoffee deployed to: 0xDBa03676a2fBb6711CB652beF5B7416A53c1421D
```

🎉 ¡Felicidades! 🎉

Ya tienes un contrato desplegado en la testnet de Goerli. Puedes verlo en el explorador de blockchain Goerli etherscan pegando tu dirección aquí: https://goerli.etherscan.io/

![](https://files.readme.io/6e07f02-aeiou.png)
Antes de pasar a la parte de la web frontend (dapp) del tutorial, vamos a preparar un script más que querremos utilizar más adelante, el script ``withdraw.js.``

## Implementar un script de retirada (``withdraw``)

Más adelante, cuando publiquemos nuestro sitio web, necesitaremos una forma de recoger todos los increíbles consejos que nos dejan nuestros amigos y fans. Podemos escribir otro script hardhat para hacer precisamente eso.

Crea un archivo en ``scripts/withdraw.js``

```
// scripts/withdraw.js

const hre = require("hardhat");
const abi = require("../artifacts/contracts/BuyMeACoffee.sol/BuyMeACoffee.json");

async function getBalance(provider, address) {
  const balanceBigInt = await provider.getBalance(address);
  return hre.ethers.utils.formatEther(balanceBigInt);
}

async function main() {
  // Get the contract that has been deployed to Goerli.
  const contractAddress="0xDBa03676a2fBb6711CB652beF5B7416A53c1421D";
  const contractABI = abi.abi;

  // Get the node connection and wallet connection.
  const provider = new hre.ethers.providers.AlchemyProvider("goerli", process.env.GOERLI_API_KEY);

  // Ensure that signer is the SAME address as the original contract deployer,
  // or else this script will fail with an error.
  const signer = new hre.ethers.Wallet(process.env.PRIVATE_KEY, provider);

  // Instantiate connected contract.
  const buyMeACoffee = new hre.ethers.Contract(contractAddress, contractABI, signer);

  // Check starting balances.
  console.log("current balance of owner: ", await getBalance(provider, signer.address), "ETH");
  const contractBalance = await getBalance(provider, buyMeACoffee.address);
  console.log("current balance of contract: ", await getBalance(provider, buyMeACoffee.address), "ETH");

  // Withdraw funds if there are funds to withdraw.
  if (contractBalance !== "0.0") {
    console.log("withdrawing funds..")
    const withdrawTxn = await buyMeACoffee.withdrawTips();
    await withdrawTxn.wait();
  } else {
    console.log("no funds to withdraw!");
  }

  // Check ending balance.
  console.log("current balance of owner: ", await getBalance(provider, signer.address), "ETH");
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

La estructura de tu proyecto debería ser la siguiente:

![](https://files.readme.io/b7a9b34-contrar.png)

Ahora tenemos 1 contrato y 3 scripts de hardhat.

La parte más importante de este script es cuando llamamos a la función ``withdrawTips()`` para sacar dinero del saldo de nuestro contrato y enviarlo a la cartera del propietario:
```
// Withdraw funds if there are funds to withdraw.
  if (contractBalance !== "0.0") {
    console.log("withdrawing funds..")
    const withdrawTxn = await buyMeACoffee.withdrawTips();
    await withdrawTxn.wait();
  }
  
 ```

Si no hay fondos en el contrato, evitamos que se intente retirar para no gastar las tasas de gas innecesariamente.

Cuando ejecute el script, verá una salida como esta:

```
thatguyintech@albert BuyMeACoffee-contracts % npx hardhat run scripts/withdraw.js
current balance of owner:  0.039608085986833815 ETH
current balance of contract:  0.001 ETH
withdrawing funds..
current balance of owner:  0.040562731986622163 ETH
```
Ten en cuenta que esta vez no hemos añadido la bandera ``--network goerli``, y eso es porque nuestro script codifica la configuración de la red directamente dentro de la lógica:
```
const provider = new hre.ethers.providers.AlchemyProvider(
    "goerli",
    process.env.GOERLI_API_KEY
);

```

Genial, ¡ahora tenemos una forma de rescatar las propinas del contrato!

Pasemos a la parte dapp de este proyecto para poder compartir nuestra página de propinas con todos nuestros amigos :)

## Construir la dapp del sitio web Buy Me A Coffee con Replit y Ethers.js

Para esta parte del sitio web, con el fin de mantener las cosas simples y limpias, vamos a utilizar una herramienta increíble para hacer girar los proyectos de demostración rápidamente, llamado Replit IDE.

Visita el proyecto de ejemplo aquí, y haz un fork para crear tu propia copia para modificar: https://replit.com/@thatguyintech/BuyMeACoffee-Solidity-DeFi-Tipping-app

![](https://files.readme.io/c0057db-Fork_repl.png)

Haz clic en "Fork repl" en la parte superior derecha para hacer tu propia copia y seguir adelante.

También puedes ver el código completo del sitio web aquí: https://github.com/alchemyplatform/RTW3-Week2-BuyMeACoffee-Website

Después de bifurcar la réplica, deberías ser llevado a una página IDE donde puedes

+ Ver el código de una aplicación web ``Next.js``
+ Acceder a una consola, un shell de terminal y una vista previa del archivo README.md
+ Ver una versión de recarga en vivo de tu aplicación

Debería tener este aspecto:

![](https://files.readme.io/517c1cf-albert.png)

Esta parte del tutorial será rápida y divertida -- ¡vamos a actualizar un par de variables para que esté conectado al contrato inteligente que desplegamos en las partes anteriores del proyecto y para que muestre su propio nombre en el sitio web!

Primero vamos a tener todo conectado y funcionando, y luego te explicaré lo que pasa en cada parte.

Aquí están los cambios que tenemos que hacer:

1. Actualizar el ``contractAddress`` en ``pages/index.js``
2. Actualizar las cadenas de nombre para que sean su propio nombre en ``pages/index.js``
3. Asegurarse de que el ABI del contrato coincide con el tuyo en ``utils/BuyMeACoffee.json``

## Actualizar contractAddress en pages/index.js

Puedes ver que la variable contractAddress ya está llena con una dirección. Este es un contrato de ejemplo desplegado, que puedes utilizar, pero si lo haces... todas las propinas enviadas a tu sitio web irán a otra dirección :)

Puedes arreglar esto pegando tu dirección de cuando desplegamos el contrato inteligente ``BuyMeACoffee.sol`` antes.

![](https://files.readme.io/9f1ee86-buya.sol.png)

Cambia el contractAddress para que apunte a tu contrato ``BuyMeACoffee.sol`` desplegado en Goerli

## Actualiza las cadenas de nombre para que sean tu propio nombre en pages/index.js

Ahora mismo el sitio tiene mi nombre por todas partes. Encuentra todos los lugares que usan ``Albert`` y reemplázalo con tu nombre/perfil anon / dominio ENS, o lo que sea que quieras que la gente te llame.

Puedes hacer un ``cmd + F`` o ``ctrl + F`` para buscar todas las instancias de ``Albert`` para reemplazar.

![](https://files.readme.io/d03249f-theGuy.png)

Cambia el texto del nombre por tu propio nombre. 

## Asegúrate de que el contrato ABI coincide en utils/BuyMeACoffee.json

Esto es también una cosa clave para comprobar especialmente cuando haces cambios en tu contrato inteligente más adelante (después de este tutorial).

La ABI es la interfaz binaria de la aplicación, que no es más que una forma elegante de decirle a nuestro código frontend qué tipo de funciones están disponibles para llamar en el contrato inteligente. La ABI se genera dentro de un archivo json cuando se compila el contrato inteligente. Puedes encontrarlo en la carpeta del contrato inteligente en la ruta ``artifacts/contracts/BuyMeACoffee.sol/BuyMeACoffee.json``

Cada vez que cambie el código de tu contrato inteligente y vuelvas a desplegarlo, tu ABI cambiará también. Copia esto y pégalo en el archivo Replit: ``utils/BuyMeACoffee.json``

![](https://files.readme.io/1aaabe1-utils.png)

Ahora, si la aplicación no se está ejecutando, puedes ir a la shell y utilizar npm run dev para iniciar un servidor local para probar tus cambios. El sitio web debería cargar en unos segundos:

![](https://files.readme.io/9ea1320-connect_wallet.png)

Lo mejor de Replit es que una vez que tienes el sitio web, puedes volver a tu perfil, encontrar el enlace del proyecto Replit, y enviarlo a tus amigos para que visiten tu página de propinas.

Ahora vamos a dar una vuelta por la página web y el código. Ya puedes ver en la captura de pantalla de arriba que cuando visitas por primera vez la dapp, ésta comprobará si tienes MetaMask instalado y si tu billetera está conectada al sitio. La primera vez que lo visites, no estarás conectado, por lo que aparecerá un botón pidiéndote que ``conectes tu billetera.``
Después de hacer clic en ``Conectar tu billetera``, aparecerá una ventana de MetaMask que te preguntará si quieres confirmar la conexión firmando un mensaje. Esta firma del mensaje no requiere ninguna tasa o coste de gas.

Una vez completada la firma, el sitio web reconocerá tu conexión y podrás ver el formulario de café, así como cualquiera de los mensajes dejados anteriormente por otros visitantes.

![](https://files.readme.io/ae9a697-momo.png)

¡BOOM! Ya está. Eso es todo el proyecto. Tómate un segundo para darte una palmadita en la espalda y reflexionar sobre el viaje que has hecho :relajado:

Para recapitular:

+ Usamos Hardhat y Ethers.js para codificar, probar y desplegar un contrato inteligente personalizado en solidity.
+ Desplegamos el contrato inteligente en la red de prueba Goerli usando Alchemy y MetaMask.
+ Implementamos un script de retirada para permitirnos aceptar los frutos de nuestro trabajo.
+ Conectamos un sitio web frontend construido con Next.js, React, y Replit al contrato inteligente utilizando Ethers.js para cargar el contrato ABI.

¡Eso es mucho!

## Desafíos

Bien, ahora es el momento de la mejor parte. Te voy a dejar algunos retos para que los intentes por tu cuenta, ¡para ver si entiendes bien lo que has aprendido aquí! [(Para obtener algo de orientación, mira el vídeo de YouTube aquí)](https://www.youtube.com/watch?v=cxxKdJk55Lk&t=3886s).

1. Permite que tu contrato inteligente actualice la dirección de retirada.
2. Permite que tu contrato inteligente compreLargeCoffee por 0,003 ETH, y crea un botón en el sitio web del frontend que muestre un botón "Buy Large Coffee for 0.003ETH".

Una vez que hayas terminado con tu desafío, tuitea sobre él etiquetando a [@AlchemyLearn](https://twitter.com/AlchemyLearn) en Twitter y utilizando el hashtag [#roadtoweb3!](https://twitter.com/search?q=%23RoadToWeb3)

Y asegúrate de compartir tus reflexiones sobre este proyecto para ganar tu ficha de Prueba de Conocimiento (PoK): https://university.alchemy.com/discord
Nos vemos en el otro lado :heart:
