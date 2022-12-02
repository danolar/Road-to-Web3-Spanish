# Como hacer NFTs con metadatos On-chain con Hardhat y Javascript

Al crear NFTs, es una buena práctica **almacenar los metadatos en almacenes de objetos centralizados** o en soluciones descentralizadas como **IPFS**, para evitar las enormes tarifas de gas derivadas del almacenamiento de grandes cantidades de datos, como imágenes y objetos JSON, directamente en la cadena.

**Sin embargo, hay un problema con esto:**

No almacenar tus metadatos en la cadena de bloques hará imposible interactuar con ellos desde tu contrato inteligente, ya que la cadena de bloques **no puede comunicarse con "el mundo externo".**

Si queremos actualizar nuestros metadatos directamente desde nuestro contrato inteligente tendremos que almacenarlos en la cadena, pero ¿qué pasa con las tarifas de gas?

Por suerte, las cadenas L2 como Polygon están aquí para ayudar, **reduciendo drásticamente los costes de gas**, e introduciendo una serie de ventajas que permiten a los desarrolladores ampliar las funcionalidades de sus aplicaciones.

En este tutorial, vas a aprender a crear **los fundamentos de un juego de blockchain**, desarrollar un NFT totalmente dinámico con metadatos en la cadena que cambian en función de tus interacciones con ella, y desplegarla en **Polygon Mumbai** para reducir las tarifas de gas.

**Más concretamente, aprenderás**

+ Cómo almacenar metadatos de NFTs en cadena
+ Qué es Polygon y por qué es importante para bajar las tasas de gas.
+ Cómo desplegar en Polygon Mumbai
+ Cómo procesar y almacenar cadena de imágenes SVG y objetos JSON
+ Cómo modificar sus metadatos en función de las interacciones con los NFT

**Próximamente el video tutorial:**

Antes de profundizar en nuestro código y desarrollar nuestro contrato inteligente de NFTs dinámicos, debemos entender brevemente un par de cosas:

+ **Qué es Polygon.**
+ **Por qué vamos a utilizarlo.**

Además, asegúrate de tener una cuenta de Alchemy [aquí](https://alchemy.com/?a=RTW3-Week-3), la necesitaremos más adelante para el desafío.

¡Empecemos!

## Polygon PoS - Menos gastos de gas y transacciones más rápidas

Polygon es **una plataforma de escalamiento descentralizada compatible con EVM** que permite a los desarrolladores construir DApps escalables y fáciles de usar con bajas tarifas de transacción sin sacrificar la seguridad.

Pertenece a un grupo de cadenas descritas como **cadenas de segunda capa (L2)**, lo que significa que están construidas sobre Ethereum para resolver algunos de los problemas que lo caracterizan, aunque dependen de él para funcionar.

Como todos sabemos, Ethereum no es ni rápido ni barato, y el despliegue de contratos inteligentes en él puede llegar a ser muy costoso, ahí es donde las soluciones L2 como **Polygon** u **Optimism**, entran en juego.

Polygon por ejemplo viene con 2 ventajas principales

+ **Transacciones más rápidas** (65.000 tx/segundos frente a ~14)
+ Aproximadamente ~10.000 **veces menos costos de gas** por transacción que Ethereum

La segunda es exactamente la razón por la que estamos desplegando nuestro contrato inteligente NFTs con metadatos en la cadena en Polygon. Si por un lado, al almacenar nuestros metadatos en Ethereum podemos esperar cientos de dólares por transacción, **en Polygon no costará más de un par de centavos.**

Si quieres profundizar en cómo Polygon y otras cadenas L2 reducen los costes de las transacciones, acelerando la velocidad de las mismas, te sugiero que vayas a ver [esta guía.](https://www.one37pm.com/nft/what-are-layer-2-solutions-and-why-are-they-important)

Ahora que tenemos una breve comprensión de lo que las soluciones L2 traen a la mesa, y por qué el uso de Polygon, en este caso, es un cambio de juego - Vamos a empezar por la configuración de nuestra cartera para conectarse con Polygon Mumbai, y obtener un poco de Matic libre que más tarde necesitaremos para pagar nuestras tarifas de Gas.

## Añade Polygon Mumbai a tu billetera de Metamask

En primer lugar, vamos a **añadir Polygon Mumbai a nuestro billetera de Metamask.**

Dirígete a [mumbai.polygonscan.com](https://mumbai.polygonscan.com/) y desplázate hasta el final de la página. Verás el botón **"Add Polygon Network"**, haz clic en él y confirma que quieres añadirlo a Metamask:

![](https://files.readme.io/f3d1895-mumbpoly.png)

Tan sencillo como eso, ¡ya tendrás Polygon Mumbai añadido a tu billetera de Metamask! 🎉

Ahora que tenemos a Polygon Mumbai conectado a nuestra extensión de Metamask, tendremos que **conseguir algún MATIC de prueba** para pagar las tasas de Gas.

## Consigue MATIC gratis para desplegar tu contrato inteligente de NFTs

Conseguir MATIC de prueba es súper sencillo, sólo tienes que navegar a uno de las siguientes fuentes:

+ [mumbaifaucet.com](https://mumbaifaucet.com/)
+ [faucet.polygon.technology](https://faucet.polygon.technology/)

Copia la dirección de la billetera en la barra de texto y haz clic en **"Send Me MATIC"**:

![](https://files.readme.io/3247953-SendMatic.png)

Después de 10-20 segundos verás que el MATIC aparece en el billetera de Metamask.

Podrás obtener hasta 1 MATIC cada 24 horas sin iniciar sesión, o 5 con una [cuenta de Alchemy](https://alchemy.com/?a=RTW3-Week-3).

**¡Genial!** Ahora que nuestra billetera está lista para funcionar, es hora de crear nuestro proyecto, desarrollar el contrato inteligente dinámico NFTs, ¡y empezar a interactuar con él!

## Cómo hacer NFTs con metadatos en la cadena - Configuración del proyecto

Abre el terminal, crea una nueva carpeta llamada "ChainBattled" e **instala Hardhat** ejecutando el siguiente comando:

```
yarn add hardhat
```

A continuación, inicia Hardhat para crear las plantillas del proyecto:

```
npx hardhat init
```
Selecciona "Crear un proyecto de muestra básico" y confirme todas las opciones:

![](https://files.readme.io/fb720fa-sampleProj.png)

Ahora necesitaremos instalar el paquete **OpenZeppelin** para tener acceso al estándar de [contratos inteligentes ERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721) que usaremos como plantilla para construir nuestro contrato inteligente de NFTs.

Instala la biblioteca de contratos inteligentes OpenZeppelin:

```
yarn add @openzeppelin/contracts
```

¡¡¡Increíble!!! Ahora hemos instalado todo lo que necesitaremos para hacer NFTs con metadatos en la cadena 🔥.

Limpiemos y modifiquemos los paquetes de desarrollo de nuestro proyecto y creemos el contrato inteligente NFTs dinámico.

Primero, sin embargo, tendremos que modificar **el archivo hardhat.config.js** para conectar con Polygon Mumbai y polygon scan - lo necesitaremos más adelante para verificar el código.

## Modificar el archivo hardhat.config.js

Vamos a abrir el proyecto en VSCode o en tu editor de texto favorito y **elimina el contrato inteligente Greeter.sol**, dentro de la carpeta "contract", y el **script "test-deploy.js"**, dentro de la carpeta "scripts".

El siguiente paso es conectar Hardhat a Polygon Mumbai. Abre el archivo **hardhat.config.js** contenido en la raíz de tu proyecto y dentro del objeto module.exports, copia el siguiente código:

```
module.exports = {
  solidity: "0.8.10",
  networks: {
    mumbai: {
      url: process.env.TESTNET_RPC,
      accounts: [process.env.PRIVATE_KEY]
    },
  },
};
```

Cuando despleguemos nuestro contrato inteligente, también querremos verificarlo usando mumbai.polygonscan, para ello necesitaremos proporcionar a Hardhat una clave API de etherscan o, en este caso, de **Polygon scan**.

La clave de la API de Polygonscan la obtendremos más adelante, por el momento, basta con añadir el siguiente código en el archivo hardhat.config.js:

```
etherscan: {
    apiKey: process.env.POLYGONSCAN_API_KEY
  }
```

En este punto **tu archivo hardhat.config.js deberia verse de la siguiente manera:** 

```
require("dotenv").config();
require("@nomiclabs/hardhat-waffle");
require("@nomiclabs/hardhat-etherscan");

module.exports = {
  solidity: "0.8.10",
  networks: {
    mumbai: {
      url: process.env.TESTNET_RPC,
      accounts: [process.env.PRIVATE_KEY]
    },
  },
  etherscan: {
    apiKey: process.env.POLYGONSCAN_API_KEY
  }
};

```

Ahora que nuestro archivo de configuración está listo, ¡empecemos a desarrollar el contrato inteligente!

## NFT con metadatos en la cadena: Desarrollar el contrato inteligente

En la carpeta de contratos, crea un nuevo archivo y llámalo "ChainBattles.sol".

Como siempre, tendremos que especificar el **SPDX-Licence-Identifier, el pragma**, e importar un par de librerías de **OpenZeppelin** que utilizaremos como base de nuestro contrato inteligente:


```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/utils/Base64.sol";

```

En este caso, estamos importando:

+ **El contrato ERC721URIStorage** que será utilizado como base de nuestro contrato inteligente ERC721
+ **La librería counters.sol**, se encargará de manejar y almacenar nuestros tokenIDs
+ **La librería string.sol** para implementar la función "toString()", que convierte los datos en cadenas - secuencias de caracteres
+ **La librería Base64** que, como hemos visto anteriormente, nos ayudará a manejar datos en base64 como nuestros SVGs en cadena

A continuación, vamos a iniciar el contrato.

## Iniciar el contrato inteligente

En primer lugar, tendremos que crear un nuevo contrato que **herede de la extensión ERC721URIStorage** que hemos importado de OpenZeppelin. Podemos hacerlo utilizando la palabra clave "is":


```
contract ChainBattles is ERC721URIStorage {}
```

Dentro del contrato, **incia la libreria de Strings and Counters:**

```
contract ChainBattles is ERC721URIStorage {
    using Strings for uint256;
    using Counters for Counters.Counter; 
}
```

En este caso "usar Strings para uint256" significa que estamos asociando todos los métodos dentro de la biblioteca "Strings" al tipo uint256. Puedes aprender más sobre cómo [asociar bibliotecas a tipos aquí.](https://forum.openzeppelin.com/t/what-does-this-mean-using-strings-for-uint256-in-erc721-contracts/7964)

Lo mismo se aplica a la línea "using Counters for Counters.Counter" - [puedes leer más sobre esto en el foro de OpenZeppelin.](https://forum.openzeppelin.com/t/what-does-this-mean-using-strings-for-uint256-in-erc721-contracts/7964)

Ahora que hemos inicializado nuestras bibliotecas, declaramos una nueva función tokenIds que necesitaremos para almacenar nuestros IDs de NFT:

```
contract ChainBattles is ERC721URIStorage {
    using Strings for uint256;
    using Counters for Counters.Counter; 
    Counters.Counter private _tokenIds;
}
```

La última variable global que debemos declarar es el [mapping](https://www.tutorialspoint.com/solidity/solidity_mappings.htm) tokenIdToLevels, que utilizaremos para almacenar el nivel de un NFT asociado a su tokenId:

```
mapping(uint256 => uint256) public tokenIdToLevels;
```

El mapeo **vinculará un uint256, el NFTId, a otro uint256**, el nivel del NFT.

A continuación, tendremos que declarar la función constructora de nuestro contrato inteligente:


```
constructor() ERC721 ("Chain Battles", "CBTLS"){
}
```

En este punto tu código debería verse así: 

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/utils/Base64.sol";

contract ChainBattles is ERC721URIStorage  {
    using Strings for uint256;
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    mapping(uint256 => uint256) public tokenIdToLevels;

    constructor() ERC721 ("Chain Battles", "CBTLS"){
    }
}

```

Ahora que tenemos la base de nuestro contrato inteligente NFT, necesitaremos implementar 4 funciones diferentes:

+ **generateCharacter:** para generar y actualizar la imagen SVG de nuestro NFT
+ **getLevels:** para obtener el nivel actual de un NFT
+ **getTokenURI:** para obtener el TokenURI de un NFT
+ **mint:** para mintear - por supuesto
+ **train:** para entrenar un NFT y aumentar su nivel

Empecemos por la primera función que tomará un SVG, lo convertirá en datos Base64, y lo guardará en la cadena - pero antes, deberíamos dedicar un par de palabras a entender por qué los SVGs pueden usarse para guardar imágenes en la cadena y cuál es su relación con los datos Base64.

## ¿Qué son los SVG y por qué son importantes?

Un archivo SVG, abreviatura de archivo gráfico vectorial escalable, es un tipo de archivo gráfico estándar utilizado para representar imágenes bidimensionales en Internet. A diferencia de otros formatos de archivo de imagen populares, el formato SVG almacena las imágenes como vectores, que es un tipo de gráfico compuesto por puntos, líneas, curvas y formas basadas en fórmulas matemáticas.

Los archivos SVG están escritos en XML, un lenguaje de marcado utilizado para almacenar y transferir información digital. El código XML de un archivo SVG especifica todas las formas, colores y texto que componen la imagen:

```
<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">
   <style>.base { fill: white; font-family: serif; font-size: 14px; }</style>
   <rect width="100%" height="100%" fill="black" />
   <text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">Warrior</text>
   <text x="50%" y="50%" class="base" dominant-baseline="middle" text-anchor="middle">Levels: getLevels(tokenId)</text>
 </svg>
```

Lo bueno de los SVG es que pueden ser

+ **Modificados fácilmente y generados mediante código**
+ **Convertirse fácilmente en datos Base64**

Ahora, podrías preguntarte por qué queremos convertir los archivos SVGs en datos Base64, la respuesta es muy simple:

Puedes mostrar imágenes Base64 en el navegador sin necesidad de un proveedor de alojamiento.

Tomemos esta imagen como ejemplo:

![](https://files.readme.io/3df2833-Screenshot_2022-05-17_at_03.34.25.png)

Copiando y pegando el siguiente código en la barra de URL de su navegador se mostrará la misma imagen:

```
data:image/svg+xml;base64,IDxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiBwcmVzZXJ2ZUFzcGVjdFJhdGlvPSJ4TWluWU1pbiBtZWV0IiB2aWV3Qm94PSIwIDAgMzUwIDM1MCI+CiAgICAgICAgPHN0eWxlPi5iYXNlIHsgZmlsbDogd2hpdGU7IGZvbnQtZmFtaWx5OiBzZXJpZjsgZm9udC1zaXplOiAxNHB4OyB9PC9zdHlsZT4KICAgICAgICA8cmVjdCB3aWR0aD0iMTAwJSIgaGVpZ2h0PSIxMDAlIiBmaWxsPSJibGFjayIgLz4KICAgICAgICA8dGV4dCB4PSI1MCUiIHk9IjQwJSIgY2xhc3M9ImJhc2UiIGRvbWluYW50LWJhc2VsaW5lPSJtaWRkbGUiIHRleHQtYW5jaG9yPSJtaWRkbGUiPldhcnJpb3I8L3RleHQ+CiAgICAgICAgPHRleHQgeD0iNTAlIiB5PSI1MCUiIGNsYXNzPSJiYXNlIiBkb21pbmFudC1iYXNlbGluZT0ibWlkZGxlIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIj5MZXZlbHM6IGdldExldmVscyh0b2tlbklkKTwvdGV4dD4KICAgICAgICA8L3N2Zz4=
```
Como puedes notar el código que has pegado no es una URL común, de hecho se compone de dos partes:

+ **Directivas de datos** - indican al navegador cómo manejar los datos ( data:image/svg+xml;base64,)
+ **Los datos Base64** - contienen los datos reales

Esto es útil porque, aunque Solidity no es capaz de manejar imágenes, sí es capaz de manejar cadenas y los SVGs no son otra cosa que secuencias de etiquetas y cadenas que podemos recuperar fácilmente en tiempo de ejecución, además, convertir todo a base64, nos permitirá almacenar nuestras imágenes en cadena sin necesidad de almacenar objetos.

Ahora que hemos explicado por qué los SVGs son importantes, vamos a aprender cómo generar nuestros propios SVGs en la cadena y convertirlos en datos Base64.

## Crear la función generateCharacter para crear la imagen SVG

Necesitaremos una función que **genere la imagen NFT en la cadena**, utilizando algún código SVG, teniendo en cuenta el nivel de la NFT.

Hacer esto en Solidity es un poco complicado, así que vamos a copiar primero el siguiente código, y luego iremos viendo las diferentes partes del mismo:

```
function generateCharacter(uint256 tokenId) public returns(string memory){

    bytes memory svg = abi.encodePacked(
        '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
        '<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>',
        '<rect width="100%" height="100%" fill="black" />',
        '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">',"Warrior",'</text>',
        '<text x="50%" y="50%" class="base" dominant-baseline="middle" text-anchor="middle">', "Levels: ",getLevels(tokenId),'</text>',
        '</svg>'
    );
    return string(
        abi.encodePacked(
            "data:image/svg+xml;base64,",
            Base64.encode(svg)
        )    
    );
}

```

Lo primero que debes notar es el tipo **"bytes"**, un array de tamaño dinámico de hasta 32 bytes donde puedes almacenar cadenas, y enteros.

Si quieres profundizar en Bytes, por otro lado, te sugiero que vayas a leer esta guía de [Jean Cvllr](https://jeancvllr.medium.com/?source=post_page-----9d88fdb22676--------------------------------).

En este caso, lo estamos usando para almacenar el código SVG que representa la imagen de nuestro NFT, transformado en un array de bytes gracias a la función abi.encodePacked() que toma una o más variables y las codifica en abi.

Puedes leer más sobre [el objeto global Solidity abi y la función encode en la documentación de Solidity.](https://docs.soliditylang.org/en/latest/abi-spec.html#non-standard-packed-mode)

Como puedes notar el código SVG toma el valor de retorno de una función getLevels() y lo usa para rellenar la propiedad "Levels:" - implementaremos esta función más adelante, pero toma nota de que **puedes usar funciones y variables para cambiar dinámicamente tus SVGs.**

Como hemos visto antes, para visualizar una imagen en el navegador necesitaremos tener la versión base64 de la misma, no la versión en bytes - además, necesitaremos anteponer la cadena "data:image/svg+xml;base64," para especificar al navegador que la cadena Base64 es una imagen SVG y cómo abrirla.

Para ello, en el código anterior, estamos devolviendo la versión codificada de nuestro SVG convertido en Base64 usando Base64.encode() con la cadena de especificación del navegador precargada, usando la función ``abi.encodePacked().``

Ahora que hemos implementado la función para generar nuestra imagen, necesitamos implementar una función para **obtener el nivel de nuestros NFTs.**

## Crear la función getLevels para obtener el nivel de los NFT

Para obtener el nivel de nuestro NFT, tendremos que utilizar **el mapeo tokenIdToLevels** que hemos declarado en nuestro contrato inteligente, pasando a la función el tokenId del que queremos obtener el nivel:

```
function getLevels(uint256 tokenId) public view returns (string memory) {
    uint256 levels = tokenIdToLevels[tokenId];
    return levels.toString();
}
```

Como puedes ver esto fue bastante sencillo, lo único que hay que notar es la función **toString()**, que viene de [la librería de Strings de OpenZeppelin](https://docs.openzeppelin.com/contracts/3.x/api/utils#Strings), y transforma nuestro nivel, que es un uint256, en una cadena - que luego será utilizada por la función generateCharacter como hemos visto antes.

A continuación, necesitaremos crear la función **getTokenURI** para generar y recuperar nuestro NFT TokenURI.

## Crear la función getTokenURI para generar el tokenURI

La función getTokenURI necesitará un parámetro, el tokenId, y lo utilizará para generar la imagen, y construir el nombre del NFT.

Como siempre, veamos primero el código, y repasemos las diferentes partes del mismo:

```
function getTokenURI(uint256 tokenId) public returns (string memory){
    bytes memory dataURI = abi.encodePacked(
        '{',
            '"name": "Chain Battles #', tokenId.toString(), '",',
            '"description": "Battles on chain",',
            '"image": "', generateCharacter(tokenId), '"',
        '}'
    );
    return string(
        abi.encodePacked(
            "data:application/json;base64,",
            Base64.encode(dataURI)
        )
    );
}

```

Lo primero que hay que notar es que estamos usando de nuevo la función **abi.encodePacked**, esta vez sin embargo, **para crear un objeto JSON.**

Si no sabes lo [que son los objetos JSON](https://www.w3schools.com/js/js_json_objects.asp), digamos brevemente que son una serie de pares de claves y valores, el nombre de una propiedad y el valor de la misma.

El valor de "nombre" en este caso es: "Chain Battles" más "#" y el tokenId toString(), el valor de la propiedad "image", por otro lado, es **el valor devuelto por la función generateCharacter().**

Finalmente, devolvemos una cadena que contiene el array de bytes que representa la versión codificada en Base64 del dataURI con las instrucciones de los datos JSON - **de forma similar a lo que hicimos con nuestra imagen SVG.**

Ahora que hemos creado nuestro **getTokenURI**, necesitaremos crear una función para acuñar nuestros NFTs e inicializar nuestras variables - ¡vamos a ver cómo!

## Crear la función Mint para crear el NFT con metadatos en la cadena

La función mint en este caso tendrá 3 objetivos:

+ **Crear un nuevo NFT.**
+ **Iniciar el valor del nivel.**
+ **Establecer el URI del token.**


Copia el siguiente código:

```
function mint() public {
    _tokenIds.increment();
    uint256 newItemId = _tokenIds.current();
    _safeMint(msg.sender, newItemId);
    tokenIdToLevels[newItemId] = 0;
    _setTokenURI(newItemId, getTokenURI(newItemId));
}
```
Como siempre, primero incrementamos el valor de nuestra variable _tokenIds, y almacenamos su valor actual en una nueva variable uint256, en este caso, "newItemId".

A continuación, llamamos a la función _safeMint() de la biblioteca [OpenZeppelin ERC721](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721), pasando la variable msg.sender, y el id actual.

Luego creamos un nuevo elemento en el mapeo tokenIdToLevels y asignamos su valor a 0, esto significa que nuestros NFTs/personajes comenzarán desde el nivel.

Como última cosa, **establecemos el URI del token pasando el newItemId y el valor de retorno de getTokenURI().**

Esto minteará un NFT cuyos metadatos, incluyendo la imagen, se almacenan completamente en la cadena 🔥.

Esto también significa que podremos **actualizar los metadatos directamente desde el contrato inteligente,** ¡vamos a ver cómo crear una función para entrenar a nuestros NFTs y que suban de nivel!

## Crear la función de entrenar para subir el nivel de los NFTs

Como decíamos, ahora que los metadatos de nuestros NFTs están completamente on-chain, podremos interactuar con ellos directamente desde el contrato inteligente.

Digamos que queremos subir el nivel de nuestros NFTs después de un entrenamiento intensivo, para ello, tendremos que crear una función de entrenamiento que:

+ Se asegure de que el NFT entrenado existe y que eres su propietario.
+ Aumente el nivel de tu NFT en 1.
+ Actualice el token URI para reflejar el entrenamiento.

```
function train(uint256 tokenId) public {
    require(_exists(tokenId), "Please use an existing token");
    require(ownerOf(tokenId) == msg.sender, "You must own this token to train it");
    uint256 currentLevel = tokenIdToLevels[tokenId];
    tokenIdToLevels[tokenId] = currentLevel + 1;
    _setTokenURI(tokenId, getTokenURI(tokenId));
}

```
Como puedes notar, usando la función require(), estamos comprobando dos cosas:

+ Si el token existe, usando la función [_exists()](https://docs.openzeppelin.com/contracts/2.x/api/token/erc721#ERC721-_exists-uint256-) del estándar ERC721,
+ Si el propietario del NFT es el *msg.sender* (el billetera que llama a la función).

Una vez superadas ambas comprobaciones, obtenemos el nivel actual del NFT del mapeo, y lo incrementamos en uno.

Por último, llamamos a la función _setTokenURI pasando el tokenId, y el valor de retorno de getTokenURI().

Al llamar a la función de tren ahora se incrementará el nivel de la NFT y esto se reflejará automáticamente en la imagen.

**Felicidades.** Acabas de terminar de escribir el contrato inteligente para NFTs con metadatos en la cadena. 🏆

El siguiente paso es desplegar el contrato inteligente en **Polygon Mumbai** e interactuar con él a través de [Polygonscan.](https://mumbai.polygonscan.com/) Para ello, necesitaremos coger nuestra clave de [Alchemy](https://alchemy.com/?a=RTW3-Week-3) y Polygonscan.

## Desplegar los NFT con el contrato inteligente de metadatos en cadena

En primer lugar, vamos a crear un nuevo archivo ``.env`` en la carpeta raíz de nuestro proyecto, y añadir las siguientes variables:

```
TESTNET_RPC=""
PRIVATE_KEY=""
POLYGONSCAN_API_KEY=""
```

A continuación, navega a [alchemy.com](https://alchemy.com/?a=RTW3-Week-3) y crea una nueva aplicación Polygon Mumbai:

Haz clic en la aplicación recién creada, copia la URL HTTP de la API y pega el valor de la API como **"TESTNET_RPC"** en el archivo .env que creamos anteriormente.

Abre tu cartera Metamask, haz clic en el menú de tres puntos > detalles de la cuenta > y copia-pega tu clave privada como valor **"PRIVATE_KEY"** en el ``.env.``

Por último, entra en [polygonscan.com](https://polygonscan.com/), y crea una nueva cuenta:

![](https://files.readme.io/4950cd8-polyscan.png)

Una vez que haya iniciado la sesión, vaya al **menú de perfil** y has clic en Claves API:

![](https://files.readme.io/1ae7acc-api_pol.png)

A continuación, haz clic en "Añadir" y dale un nombre a tu aplicación:

![](https://files.readme.io/24d5821-add.png)

Ahora copia y pega el Token Api-Key como valor **"POLYGONSCANAPI_KEY"** en el ``.env.``

Un último paso antes de desplegar nuestro Smart contract, necesitaremos **crear el script de despliegue.**

## Crear el script de despliegue

El script de despliegue, como has aprendido [en la lección anterior](https://docs.alchemy.com/alchemy/road-to-web3/weekly-learning-challenges/2.-how-to-build-buy-me-a-coffee-defi-dapp), se utiliza para, como su nombre indica, decirle a Hardhat cómo desplegar el contrato inteligente en la blockchain especificada.

_Nuestro script de despliegue, en este caso, es bastante sencillo:_

```
const main = async () => {
  try {
    const nftContractFactory = await hre.ethers.getContractFactory(
      "ChainBattles"
    );
    const nftContract = await nftContractFactory.deploy();
    await nftContract.deployed();

    console.log("Contract deployed to:", nftContract.address);
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};
  
main();

```

Llamamos a **get.contractFactory**, pasando el nombre de nuestro contrato inteligente, **desde Hardhat ethers**. Luego llamamos a la función deploy() y esperamos a que se despliegue, registrando la dirección.

Todo está envuelto en **un bloque try{} catch{}** para capturar cualquier error que pueda ocurrir e imprimirlo para fines de depuración.

Ahora que nuestro script de despliegue está listo, es el momento de compilar y desplegar nuestro contrato inteligente NFT dinámico en Polygon Mumbai.

## Compilar y desplegar el contrato inteligente

Para compilar el contrato inteligente simplemente ejecuta el siguiente comando, en la terminal dentro del proyecto:

```
npx hardhat compile
```


Si todo va como se espera, verás tu **contrato inteligente compilado dentro de la carpeta de “artifacts”.**

Ahora, vamos a desplegar el contrato inteligente en la cadena Polygon Mumbai en ejecución:

```
npx hardhat run scripts/deploy.js --network mumbai
```

Espera 10-15 segundos y deberías ver la dirección de tu contrato inteligente impresa en tu terminal

![](https://files.readme.io/856d094-smart_contract.png)

Increíble, ¡acabas de desplegar tu primer contrato inteligente en Polygon Mumbai! A continuación, tendremos que verificar el código de nuestro contrato inteligente para interactuar con él a través de Polygon Scan.

## Comprueba tu contrato inteligente en Polygon Scan

Copia la dirección del contrato inteligente recién desplegado, dirígete a [mumbai.polygonscan.com](https://mumbai.polygonscan.com/) y **pega la dirección del contrato inteligente en la barra de búsqueda.**

Una vez en la página de tu contrato inteligente, haz clic en la pestaña "Contrato".

Verás que el código del contrato no es legible:

![](https://files.readme.io/e86495b-concode.png)

**Esto es porque aún no hemos verificado nuestro código.**

Para verificar nuestro contrato inteligente tendremos que volver a nuestro proyecto y, en el terminal, ejecutar el siguiente código:

```
npx hardhat verify --network mumbai YOUR_SMARTCONTRACT_ADDRESS
```

![](https://files.readme.io/64fcc79-verify_code.png)

A veces puede aparecer el error "failed to send contract verification request" - simplemente inténtalo de nuevo y debería pasar.

Esto utilizará la clave API de escaneo de polygon que añadimos en el archivo hardhat.config.js para verificar el código del contrato inteligente. Ahora podremos interactuar con él a través de Polygon scan, vamos a probarlo.

## Interactúa con tu Smart Contract a través de Polygon Scan

Ahora que el Smart Contract ha sido verificado, mumbai.polygonscan.com mostrará un pequeño tick verde cerca de él:

![](https://files.readme.io/80d594c-consource.png)
Para interactuar con él, y mintear el primer NFT, haz **clic en el botón "Write Contract" bajo la pestaña "contract"**, y haz clic en "connect to Web3"

![](https://files.readme.io/556ed26-connect_web3.png)
A continuación, busca la función "mint" y haz clic en Write:

![](https://files.readme.io/a8d336c-write.png)
Esto abrirá una ventana emergente de Metamask que le pedirá que pague las tasas de gas, haz clic en el botón de firmar.

¡Enhorabuena! Acabas de mintear tu primera NFT dinámica - pasemos a OpenSea Testnet para verla en directo.

## Vea su NFT dinámico en OpenSea

Copia la dirección del contrato inteligente, entra a [testnet.opensea.com](https://testnets.opensea.io/) y pégala en la barra de búsqueda:

![](https://files.readme.io/c2bfe83-chain_jotter.png)
Si todo ha funcionado como se esperaba, ahora deberías ver tu NFT mostrándose en OpenSea, con su imagen dinámica, el título y la descripción.

No es nada nuevo hasta ahora, ya construimos una colección de NFT en [la primera lección](https://docs.alchemy.com/alchemy/road-to-web3/weekly-learning-challenges/1.-how-to-develop-an-nft-smart-contract-erc721-with-alchemy), lo que es genial aquí es que ahora podemos actualizar la imagen en tiempo real.

Volvamos a Polygon scan.

## Actualizar la imagen NFT dinámica Entrenando el NFT

Navega de nuevo a [mumbai.polygonscan.com](https://mumbai.polygonscan.com/), haz **clic en la pestaña de contrato > Escribir contrato** y busca la función "entrenar".

Introduce el ID de tu NFT - "1" en este caso, ya que sólo hemos minteado uno, y haz **clic en Escribir:**

![](https://files.readme.io/5caa7b7-nft_id.png)
A continuación, vuelve a [testnets.opensea.com](https://testnets.opensea.io/) y actualiza la página:

![](https://files.readme.io/f6bea78-chain_bottles.png)

¡Como puedes ver la imagen de tu NFT acaba de cambiar reflejando el nuevo nivel!

**Enhorabuena, tu NFT acaba de subir de nivel. ¡Bien hecho!** 🎉

¡Ahora es el momento de pasar al siguiente nivel con el reto de esta semana!

## El reto de esta semana

De momento sólo estamos almacenando el nivel de nuestros NFTs, ¿por qué no almacenar más?

**Sustituye la asignación actual tokenIdToLevels[] por una estructura que almacene:**

+ **Nivel**
+ **Velocidad**
+ **Fuerza**
+ **Vida**

Puedes leer más sobre los structs [en esta guía.](https://www.tutorialspoint.com/solidity/solidity_structs.htm)

Una vez que hayas creado el struct, inicializa las estadísticas en la función mint(), para ello puede que quieras mirar [la generación de pseudo números en Solidity.](https://blog.finxter.com/how-to-generate-random-numbers-in-solidity/)

Una vez que hayas completado el proyecto, comparte una reflexión sobre tu experiencia para ganar una Prueba de Conocimiento NFT en [el discord de la Universidad de Alchemy.](https://university.alchemy.com/discord)
