# Cómo crear un mercado NFT desde cero

Bienvenido a la semana 7 de la serie Road To Web3. Este tutorial te enseña a construir tu propio mercado de NFT desde cero: frontend, almacenamiento de datos y contratos inteligentes.

Tanto si ordenas por número de usuarios como por volúmenes, los marketplaces de NFT son algunas de las empresas más grandes de Web3. Por ejemplo:

* En enero de 2022, Opensea, el mayor mercado de NFT de Ethereum vendió ~2,5 millones de NFT y negoció 5.000 millones de dólares en volumen.
* En mayo de 2022, Magic Eden, el mayor Mercado de NFT de Solana tuvo ~11,3 millones de transacciones y 200 millones de dólares en volumen.

Tal escala solo se logra con grandes contratos inteligentes e infraestructura escalable. Así que si eres un desarrollador web3 buscando mejorar tus habilidades de desarrollo web3, sigue este tutorial sobre la construcción de un mercado NFT usando [Alchemy](https://www.alchemy.com/), IPFS, Hardhat y ethers.js.

Algunas cosas a tener en cuenta:

* El enfoque de este tutorial será la construcción del contrato inteligente y no la construcción del frontend. Sin embargo, [el código frontend para un mercado NFT](https://github.com/OMGWINNING/NFT-Marketplace-Tutorial) está disponible en GitHub.
* No hay backend o base de datos involucrados en este tutorial. Un backend y una base de datos sólo serán necesarios cuando empieces a archivar datos e integrar funciones de registro o inicio de sesión.

## Paso 0: Crea una cuenta en Alchemy y crea una nueva aplicación

Si aún no lo has hecho, regístrate para obtener una [cuenta gratuita de Alchemy.](https://dashboard.alchemyapi.io/signup/?a=dabb74c129)

A continuación, puedes crear una nueva aplicación y crear claves API desde el panel de control de la aplicación.

Echa un vistazo a este vídeo sobre cómo crear una aplicación (ingles):

https://youtu.be/tfggWxfG9o0

O sigue los pasos escritos a continuación:

1. Navega hasta el botón "crear app" en la pestaña "Apps".

![](https://files.readme.io/034cc1e-pol.png)
Ejemplo de panel de control

Rellena los detalles en la ventana emergente para obtener tu nueva clave. Para este tutorial, debes elegir **"Ethereum"** como cadena y **"Goerli"** como red de prueba.

![](https://files.readme.io/b948240-geroli.png)

También puedes obtener claves API existentes pasando el ratón por encima de "Apps" y seleccionando una.

Aquí puedes "Ver Clave", así como "Editar App" para colocar dominios específicos en la lista blanca, ver varias herramientas para desarrolladores y ver analíticas.

## Paso 1: Configurar tu cartera MetaMask para el desarrollo

Si ya tienes MetaMask con una dirección Goerli y al menos 0.1 Goerli ETH en ella, salta al Paso 2.

Si no tienes una dirección Goerli, [conecta MetaMask a la red Goerli](https://docs.alchemy.com/docs/how-to-add-alchemy-rpc-endpoints-to-metamask), y luego [utiliza un grifo Goerli para solicitar Goerli ETH](https://goerlifaucet.com/). Necesitarás Goerli ETH para desplegar contratos inteligentes y subir NFTs a tu mercado NFT.

Asegúrate de añadir los siguientes detalles cuando añadas una nueva red:

> **Nombre de la Red:** Red de prueba Goerli
>
> **URL base RPC:** https://eth-goerli.g.alchemy.com/v2/{INSERT YOUR API KEY}
>
> **ID de cadena:** 5
>
> **URL del explorador de bloques:** https://goerli.etherscan.io/
>
> **Símbolo (opcional):** ETH

## Paso 2: Configurar el repositorio

Para hacerlo más fácil, hemos subido el código base al siguiente [repositorio de GitHub](https://github.com/alchemyplatform/RTW3-Week7-NFT-Marketplace). Este código tiene todo el frontend escrito, pero no tiene un contrato inteligente o integraciones con frontend.

Para clonar el repositorio, ejecuta los siguientes comandos en el símbolo del sistema:
```
git clone https://github.com/alchemyplatform/RTW3-Week7-NFT-Marketplace.git
cd RTW3-Week7-NFT-Marketplace
npm install
npm start

```

> ##📘 Nota
>
> El repositorio de GitHub anterior es el repositorio base sobre el que debes construir.
>
> Hay un repositorio GitHub diferente con [el código final de NFT Marketplace.](https://github.com/OMGWINNING/NFT-Marketplace-Tutorial)
>
> Consúltalo si te quedas atascado siguiendo el tutorial.

## Paso 3: Configura tus variables de entorno y la configuración de Hardhat

Crea un nuevo archivo .env en la raíz del proyecto que está justo dentro de la carpeta RTW3-Week7-NFT-Marketplace, y añade:

* La URL de la API de Alchemy que creaste en el Paso 1
* La clave privada del monedero MetaMask que utilizarás para el desarrollo.

Cuando hayas terminado, tu archivo ``.env``  debería tener este aspecto

```
REACT_APP_ALCHEMY_API_URL="<TU_API_URL>"
REACT_APP_PRIVATE_KEY="<TU_CLAVE_PRIVADA>"
```

Si no está ya instalado, instala dotenv en tu carpeta raíz:

```
npm install dotenv --save
```

dotenv te ayuda a gestionar las variables de entorno que se mencionan en el archivo .env, facilitando que tu proyecto pueda acceder a ellas.

>##**❗️Advertencia**
>
> No envíes una aplicación de producción con claves secretas en el archivo .env. Este tutorial muestra cómo subir a IPFS a través de su cliente react directamente solamente como una demostración.
>
> Cuando estés listo para producción, debes refactorizar tu aplicación para subir archivos IPFS usando un servicio backend.
>
> Lee esto para más contexto sobre las [variables de entorno de React.](https://create-react-app.dev/docs/adding-custom-environment-variables/#adding-development-environment-variables-in-env)

En tu directorio de inicio, asegúrate de que el siguiente código se añade a tu archivo hardhat.config.js:

```
require("@nomiclabs/hardhat-waffle");
require("@nomiclabs/hardhat-ethers");
const fs = require('fs');
// const infuraId = fs.readFileSync(".infuraid").toString().trim() || "";
require('dotenv').config();

task("accounts", "Prints the list of accounts", async (taskArgs, hre) => {
  const accounts = await hre.ethers.getSigners();

  for (const account of accounts) {
    console.log(account.address);
  }
});

module.exports = {
  defaultNetwork: "hardhat",
  networks: {
    hardhat: {
      chainId: 1337
    },
    goerli: {
      url: process.env.REACT_APP_ALCHEMY_API_URL,
      accounts: [ process.env.REACT_APP_PRIVATE_KEY ]
    }
  },
  solidity: {
    version: "0.8.4",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  }
};
```

>##**📘Nota**
>
>Puede que tengas problemas para hacer que ``process.env`` funcione en la configuración hardhat incluso después de instalar dotenv. En ese caso, simplemente pega la URL de goerli y la clave privada directamente en esta configuración. Asegúrese de no enviarlo a GitHub.

## Paso 4: Utiliza Piñata para subir tus datos a IPFS

Si no tienes una cuenta Piñata, [regístrate para obtener una cuenta Piñata gratuita](https://pinata.cloud/signup) y verifica tu correo electrónico.

### Crea tu clave API de Piñata

Para crear tu clave de Piñata:

* Dirigete a https://pinata.cloud/keys
* Selecciona el botón "Nueva clave" en la parte superior
* Activa el widget Admin
* Asigna un nombre a la clave

![](https://static.slab.com/prod/uploads/7adb25ff/posts/images/J__0NjUkj_6BObi1Q4Q3eRe6.png)

Aparecerá una ventana emergente con la información de tu API. Cópiala en un lugar seguro.

Asegúrate de guardar tu clave API y el secreto en un lugar seguro](https://static.slab.com/prod/uploads/7adb25ff/posts/images/to1HORepBqC2D350oKtfQJlh.png)
Ahora que la clave de Piñata está configurada, añádela a tu proyecto para poder utilizarla.

Añade tu clave API y el secreto para que el archivo .env tenga ahora el siguiente aspecto:

```
REACT_APP_ALCHEMY_API_URL="<TU_API_URL>"
REACT_APP_PRIVATE_KEY="<TU_CLAVE_PRIVADA>"
REACT_APP_PINATA_KEY="<TU_PINATA_KEY>"
REACT_APP_PINATA_SECRET="<SU_PINATA_SECRET>"
```

## Paso 5: Comprender los requisitos

A continuación se muestra el mercado NFT que vas a hacer al final de este tutorial.

Hemos elegido perros para este mercado. ¡Sé libre de cambiar a cualquier otra foto que te guste!


![](https://files.readme.io/e4520c5-dog.png)

Antes de que podamos sumergirnos profundamente en la escritura de código, vamos a repasar páginas adicionales para entender el conjunto de características que necesitamos, tanto desde una perspectiva de frontend como de contrato inteligente.

## Lista tu página NFT

![](https://files.readme.io/23374cc-mar.png)

Para cualquier artista o creador, esta es la página en la que puede poner a la venta su NFT en el mercado.

Como puedes ver, esta página recoge los siguientes atributos de la NFT:

* Nombre del NFT
* Descripción
* Precio (en ETH)
* Imagen del NFT

Una vez completado, este contenido se subirá al mercado de NFTs.

Para ello, necesitamos lo siguiente:

**Contrato inteligente** | **Frontend**|
---|---
|función ``createToken()``||
|**Entrada**|Script que hace lo siguiente:|
|una URL IPFS que tenga metadatos|Toma entradas de todos los detalles relevantes del NFT|
|el precio de venta de la NFT|Sube la imagen de la NFT a IPFS|
||Sube los metadatos del NFT con un enlace de imagen a IPFS|
|**¿Qué hace?**|Envía el enlace IPFS y el precio a la función ``createToken()`` en el contrato inteligente|
|Asigna un ``_tokenId`` a su NFT|* Notificar al usuario que la carga se ha realizado correctamente|
|Guarda los datos correspondientes en el contrato de mercado||
| *Emite un evento Listing Success una vez realizado|Puedes encontrar la implementación en ``src/contracts/SellNFT.js``|
|Vea la implementación [aquí.](https://docs.alchemy.com/docs/how-to-build-an-nft-marketplace-from-scratch#createtoken-and-createlistedtoken)||	


## Página de inicio del mercado

![](https://files.readme.io/b0e0f55-user_prof.png)
Ejemplo de página de inicio del mercado NFT

Esta es la página de inicio del mercado donde se listan todos los NFTs.
Para que esto suceda, necesitamos:

Contrato inteligente | Frontend
---|---
función getAllNFTs()|Obtiene todos los NFT en venta utilizando la función ``getAllNFTs()`` del contrato inteligente.
**Input**|
None|Muéstrelos en una cuadrícula
**Output**|* Permitir a los usuarios hacer clic en un NFT individual para ver más detalles
Una lista de todos los NFTs actualmente a la venta con sus metadatos| Puedes encontrar la implementación en ``src/components/Marketplace.js`` , ``src/components/NFTPage.js`` y ``src/components/NFTTile.js``


## Página de perfil de usuario

![](https://files.readme.io/d875c85-profile-page_1.png)
Página de perfil

Se trata de un perfil de usuario en marketplace que muestra:

* Dirección del monedero del usuario
* Datos sobre los NFT que son propiedad del usuario
* Una vista en cuadrícula de todos esos NFTs con detalles

Para conseguir esto, necesitamos:

Contrato inteligente| Frontend
---|---
Función ``getMyNFTs()`` que devuelve todos los NFTs que un usuario ha vendido en el pasado.| Obtener los datos utilizando ``getMyNFTs()`` del contrato inteligente.
   |	Analizar los datos para obtener números agregados y estadísticas
 |* Mostrar los datos en el formato anterior

## Página NFT individual

![](https://files.readme.io/f0808c0-individual-nft-page.png)
Página de aterrizaje para un NFT individual en el mercado de NFTs.

Si haces clic en cualquier NFT en la página del mercado o desde la página Perfil, esta es la página que verán los visitantes:

* Metadatos de la NFT
* Un botón "Comprar este NFT" que permite a otro usuario comprar el NFT.

Para conseguir esto, necesitamos:

|Smart Contract | Frontend|
|---|---|
|Algunas funciones:|Script que hace lo siguiente:|
|1. Una función tokenURI que devuelva el tokenURI para un tokenId. Luego obtenemos los metadatos para ese tokenURI.|  Obtiene el tokenURI usando el método tokenURI|
|| Obtiene los datos de ese tokenURI IPFS usando axios|
|| Muestra los datos|
|2. Una función ``executeSale()`` que ayuda a realizar las comprobaciones necesarias y transfiere la propiedad cuando un usuario hace clic en el botón "Comprar este NFT". La implementación se puede encontrar más abajo => (#executesale)| Además, llama a la función ``executeSale()`` cuando se pulsa el botón "Comprar esta NFT|

 
Ahora tienes una comprensión completa de las características necesarias para construir un mercado NFT.

¡Sigamos adelante! :tada:

## Paso 6: Escribir el Contrato Inteligente
¡Empecemos a construir un mercado NFT! Si te confundes, consulta el [Contrato Inteligente terminado.](https://github.com/OMGWINNING/NFT-Marketplace-Tutorial/blob/master/contracts/NFTMarketplace.sol)

### Añadir Importaciones

Hay un archivo ``NFTMarketplace.sol`` en tu carpeta de contratos.

Añade las siguientes importaciones en la parte superior de este archivo y añade una clase vacía con un constructor:

```
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

//Console functions to help debug the smart contract just like in Javascript
import "hardhat/console.sol";
//OpenZeppelin's NFT Standard Contracts. We will extend functions from this in our implementation
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract NFTMarketplace is ERC721URIStorage {
    constructor() ERC721("NFTMarketplace", "NFTM") {
        owner = payable(msg.sender);
    }
}
```
El código se explica en los comentarios.

### Añadir variables globales

Añade las siguientes Variables Globales en la parte superior de tu Contrato Inteligente dentro de la declaración de la clase:

```
    using Counters for Counters.Counter;
    //_tokenIds variable has the most recent minted tokenId
    Counters.Counter private _tokenIds;
    //Keeps track of the number of items sold on the marketplace
    Counters.Counter private _itemsSold;
    //owner is the contract address that created the smart contract
    address payable owner;
    //The fee charged by the marketplace to be allowed to list an NFT
    uint256 listPrice = 0.01 ether;

    //The structure to store info about a listed token
    struct ListedToken {
        uint256 tokenId;
        address payable owner;
        address payable seller;
        uint256 price;
        bool currentlyListed;
    }

    //the event emitted when a token is successfully listed
    event TokenListedSuccess (
        uint256 indexed tokenId,
        address owner,
        address seller,
        uint256 price,
        bool currentlyListed
    );

    //This mapping maps tokenId to token info and is helpful when retrieving details about a tokenId
    mapping(uint256 => ListedToken) private idToListedToken;

```

+ **``_tokenIds``**: Este es el último ID de token que corresponde a un NFT minteado con este contrato inteligente. tokenIDs mapea a ``tokenURI`` que es la URL que contiene los metadatos del NFT correspondiente

+ **``_itemsSold``**: Es un recuento del número de artículos vendidos en el mercado
+ **``owner``:** Es el propietario del contrato inteligente. Es la única dirección que puede emitir una solicitud de retirada.
+ **``listPrice``:** El precio (en ETH) que cualquier usuario necesita pagar para listar su NFT en el marketplace.
+ **``ListedToken``**: Una estructura de solidity (similar a un objeto Javascript) que dicta el formato en el que se almacenan los datos de un NFT
+ **``TokenListedSuccess``**: Evento emitido cuando un token es listado con éxito
+ **``idToListedToken``**: Es la asignación de todos los tokenId existentes al token NFT correspondiente.

## createToken y createListedToken

Esta función convierte un tokenURI (URL con metadatos) en un NFT real en la cadena, con detalles almacenados en el contrato inteligente. Esto es útil para la página **List your NFT.**

Añade las siguientes funciones dentro de tu clase de contrato justo debajo de tu declaración de variable Global:

```
    //The first time a token is created, it is listed here
    function createToken(string memory tokenURI, uint256 price) public payable returns (uint) {
        //Increment the tokenId counter, which is keeping track of the number of minted NFTs
        _tokenIds.increment();
        uint256 newTokenId = _tokenIds.current();

        //Mint the NFT with tokenId newTokenId to the address who called createToken
        _safeMint(msg.sender, newTokenId);

        //Map the tokenId to the tokenURI (which is an IPFS URL with the NFT metadata)
        _setTokenURI(newTokenId, tokenURI);

        //Helper function to update Global variables and emit an event
        createListedToken(newTokenId, price);

        return newTokenId;
    }

    function createListedToken(uint256 tokenId, uint256 price) private {
        //Make sure the sender sent enough ETH to pay for listing
        require(msg.value == listPrice, "Hopefully sending the correct price");
        //Just sanity check
        require(price > 0, "Make sure the price isn't negative");

        //Update the mapping of tokenId's to Token details, useful for retrieval functions
        idToListedToken[tokenId] = ListedToken(
            tokenId,
            payable(address(this)),
            payable(msg.sender),
            price,
            true
        );

        _transfer(msg.sender, address(this), tokenId);
        //Emit the event for successful transfer. The frontend parses this message and updates the end user
        emit TokenListedSuccess(
            tokenId,
            address(this),
            msg.sender,
            price,
            true
        );
    }
 ```
 
La relevancia de cada línea de código se menciona en los comentarios. Tómate 2 minutos para repasarlo.

### getAllNFTs

Esta función devuelve todos los NFT "activos" (actualmente a la venta) en el mercado. Es útil para la ** página de inicio del mercado.**

Añade la siguiente función en la clase de contrato, justo debajo de la función ``createListedToken:``

```
    //This will return all the NFTs currently listed to be sold on the marketplace
    function getAllNFTs() public view returns (ListedToken[] memory) {
        uint nftCount = _tokenIds.current();
        ListedToken[] memory tokens = new ListedToken[](nftCount);
        uint currentIndex = 0;

        //at the moment currentlyListed is true for all, if it becomes false in the future we will 
        //filter out currentlyListed == false over here
        for(uint i=0;i<nftCount;i++)
        {
            uint currentId = i + 1;
            ListedToken storage currentItem = idToListedToken[currentId];
            tokens[currentIndex] = currentItem;
            currentIndex += 1;
        }
        //the array 'tokens' has the list of all NFTs in the marketplace
        return tokens;
    }
    
```
 
La relevancia de cada línea de código se menciona en los comentarios.

### getMyNFTs

Esta función devuelve todas las NFT "activas" (actualmente a la venta) en el mercado, que posee el usuario que ha iniciado sesión. Esto es útil para la **página de perfil.**

Añade la siguiente función en su clase de contrato, justo debajo de la función ``getAllNFTs:``

```
       //Returns all the NFTs that the current user is owner or seller in
    function getMyNFTs() public view returns (ListedToken[] memory) {
        uint totalItemCount = _tokenIds.current();
        uint itemCount = 0;
        uint currentIndex = 0;
        
        //Important to get a count of all the NFTs that belong to the user before we can make an array for them
        for(uint i=0; i < totalItemCount; i++)
        {
            if(idToListedToken[i+1].owner == msg.sender || idToListedToken[i+1].seller == msg.sender){
                itemCount += 1;
            }
        }

        //Once you have the count of relevant NFTs, create an array then store all the NFTs in it
        ListedToken[] memory items = new ListedToken[](itemCount);
        for(uint i=0; i < totalItemCount; i++) {
            if(idToListedToken[i+1].owner == msg.sender || idToListedToken[i+1].seller == msg.sender) {
                uint currentId = i+1;
                ListedToken storage currentItem = idToListedToken[currentId];
                items[currentIndex] = currentItem;
                currentIndex += 1;
            }
        }
        return items;
    }
```

La relevancia de cada línea de código se menciona en los comentarios.

**executeSale**

Cuando un usuario hace clic en "Comprar esta NFT" en la página de perfil, se activa la función ``executeSale.``

Si el usuario ha pagado suficiente ETH igual al precio de la NFT, la NFT se transfiere a la nueva dirección y el producto de la venta se envía al vendedor.

Añade la siguiente función a tu contrato inteligente:
```
    function executeSale(uint256 tokenId) public payable {
        uint price = idToListedToken[tokenId].price;
        address seller = idToListedToken[tokenId].seller;
        require(msg.value == price, "Please submit the asking price in order to complete the purchase");

        //update the details of the token
        idToListedToken[tokenId].currentlyListed = true;
        idToListedToken[tokenId].seller = payable(msg.sender);
        _itemsSold.increment();

        //Actually transfer the token to the new owner
        _transfer(address(this), msg.sender, tokenId);
        //approve the marketplace to sell NFTs on your behalf
        approve(address(this), tokenId);

        //Transfer the listing fee to the marketplace creator
        payable(owner).transfer(listPrice);
        //Transfer the proceeds from the sale to the seller of the NFT
        payable(seller).transfer(msg.value);
    }
```
**Otras funciones de ayuda**

A continuación se presentan otras funciones de ayuda, que son buenas para tener en los contratos inteligentes para la prueba y sería útil si decides ampliar con más funcionalidades.

Siéntete libre de añadirlas en cualquier parte de tu clase:

```
    function updateListPrice(uint256 _listPrice) public payable {
        require(owner == msg.sender, "Only owner can update listing price");
        listPrice = _listPrice;
    }

    function getListPrice() public view returns (uint256) {
        return listPrice;
    }

    function getLatestIdToListedToken() public view returns (ListedToken memory) {
        uint256 currentTokenId = _tokenIds.current();
        return idToListedToken[currentTokenId];
    }

    function getListedTokenForId(uint256 tokenId) public view returns (ListedToken memory) {
        return idToListedToken[tokenId];
    }

    function getCurrentToken() public view returns (uint256) {
        return _tokenIds.current();
    }
 ```  
Después de hacer todo lo anterior, a continuación se muestra cómo deberia ser el contrato inteligente:

```
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "hardhat/console.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract NFTMarketplace is ERC721URIStorage {

    using Counters for Counters.Counter;
    //_tokenIds variable has the most recent minted tokenId
    Counters.Counter private _tokenIds;
    //Keeps track of the number of items sold on the marketplace
    Counters.Counter private _itemsSold;
    //owner is the contract address that created the smart contract
    address payable owner;
    //The fee charged by the marketplace to be allowed to list an NFT
    uint256 listPrice = 0.01 ether;

    //The structure to store info about a listed token
    struct ListedToken {
        uint256 tokenId;
        address payable owner;
        address payable seller;
        uint256 price;
        bool currentlyListed;
    }

    //the event emitted when a token is successfully listed
    event TokenListedSuccess (
        uint256 indexed tokenId,
        address owner,
        address seller,
        uint256 price,
        bool currentlyListed
    );

    //This mapping maps tokenId to token info and is helpful when retrieving details about a tokenId
    mapping(uint256 => ListedToken) private idToListedToken;

    constructor() ERC721("NFTMarketplace", "NFTM") {
        owner = payable(msg.sender);
    }

    function updateListPrice(uint256 _listPrice) public payable {
        require(owner == msg.sender, "Only owner can update listing price");
        listPrice = _listPrice;
    }

    function getListPrice() public view returns (uint256) {
        return listPrice;
    }

    function getLatestIdToListedToken() public view returns (ListedToken memory) {
        uint256 currentTokenId = _tokenIds.current();
        return idToListedToken[currentTokenId];
    }

    function getListedTokenForId(uint256 tokenId) public view returns (ListedToken memory) {
        return idToListedToken[tokenId];
    }

    function getCurrentToken() public view returns (uint256) {
        return _tokenIds.current();
    }

    //The first time a token is created, it is listed here
    function createToken(string memory tokenURI, uint256 price) public payable returns (uint) {
        //Increment the tokenId counter, which is keeping track of the number of minted NFTs
        _tokenIds.increment();
        uint256 newTokenId = _tokenIds.current();

        //Mint the NFT with tokenId newTokenId to the address who called createToken
        _safeMint(msg.sender, newTokenId);

        //Map the tokenId to the tokenURI (which is an IPFS URL with the NFT metadata)
        _setTokenURI(newTokenId, tokenURI);

        //Helper function to update Global variables and emit an event
        createListedToken(newTokenId, price);

        return newTokenId;
    }

    function createListedToken(uint256 tokenId, uint256 price) private {
        //Make sure the sender sent enough ETH to pay for listing
        require(msg.value == listPrice, "Hopefully sending the correct price");
        //Just sanity check
        require(price > 0, "Make sure the price isn't negative");

        //Update the mapping of tokenId's to Token details, useful for retrieval functions
        idToListedToken[tokenId] = ListedToken(
            tokenId,
            payable(address(this)),
            payable(msg.sender),
            price,
            true
        );

        _transfer(msg.sender, address(this), tokenId);
        //Emit the event for successful transfer. The frontend parses this message and updates the end user
        emit TokenListedSuccess(
            tokenId,
            address(this),
            msg.sender,
            price,
            true
        );
    }
    
    //This will return all the NFTs currently listed to be sold on the marketplace
    function getAllNFTs() public view returns (ListedToken[] memory) {
        uint nftCount = _tokenIds.current();
        ListedToken[] memory tokens = new ListedToken[](nftCount);
        uint currentIndex = 0;

        //at the moment currentlyListed is true for all, if it becomes false in the future we will 
        //filter out currentlyListed == false over here
        for(uint i=0;i<nftCount;i++)
        {
            uint currentId = i + 1;
            ListedToken storage currentItem = idToListedToken[currentId];
            tokens[currentIndex] = currentItem;
            currentIndex += 1;
        }
        //the array 'tokens' has the list of all NFTs in the marketplace
        return tokens;
    }
    
    //Returns all the NFTs that the current user is owner or seller in
    function getMyNFTs() public view returns (ListedToken[] memory) {
        uint totalItemCount = _tokenIds.current();
        uint itemCount = 0;
        uint currentIndex = 0;
        
        //Important to get a count of all the NFTs that belong to the user before we can make an array for them
        for(uint i=0; i < totalItemCount; i++)
        {
            if(idToListedToken[i+1].owner == msg.sender || idToListedToken[i+1].seller == msg.sender){
                itemCount += 1;
            }
        }

        //Once you have the count of relevant NFTs, create an array then store all the NFTs in it
        ListedToken[] memory items = new ListedToken[](itemCount);
        for(uint i=0; i < totalItemCount; i++) {
            if(idToListedToken[i+1].owner == msg.sender || idToListedToken[i+1].seller == msg.sender) {
                uint currentId = i+1;
                ListedToken storage currentItem = idToListedToken[currentId];
                items[currentIndex] = currentItem;
                currentIndex += 1;
            }
        }
        return items;
    }

    function executeSale(uint256 tokenId) public payable {
        uint price = idToListedToken[tokenId].price;
        address seller = idToListedToken[tokenId].seller;
        require(msg.value == price, "Please submit the asking price in order to complete the purchase");

        //update the details of the token
        idToListedToken[tokenId].currentlyListed = true;
        idToListedToken[tokenId].seller = payable(msg.sender);
        _itemsSold.increment();

        //Actually transfer the token to the new owner
        _transfer(address(this), msg.sender, tokenId);
        //approve the marketplace to sell NFTs on your behalf
        approve(address(this), tokenId);

        //Transfer the listing fee to the marketplace creator
        payable(owner).transfer(listPrice);
        //Transfer the proceeds from the sale to the seller of the NFT
        payable(seller).transfer(msg.value);
    }

    //We might add a resell token function in the future
    //In that case, tokens won't be listed by default but users can send a request to actually list a token
    //Currently NFTs are listed by default
}
```

## Paso 7: Desplegar el contrato inteligente en Goerli

¡Buen trabajo codificando ese enorme contrato inteligente! Eres increíble! :sparkling_heart:

Ahora tenemos que desplegar el contrato. Alchemy recomienda la [red de pruebas Goerli](https://www.alchemy.com/overviews/goerli-faucet)

Hay un script llamado ``deploy.js`` dentro de la carpeta ``scripts/``. En ese archivo, pega este código:

```
const { ethers } = require("hardhat");
const hre = require("hardhat");
const fs = require("fs");

async function main() {
  //get the signer that we will use to deploy
  const [deployer] = await ethers.getSigners();
  
  //Get the NFTMarketplace smart contract object and deploy it
  const Marketplace = await hre.ethers.getContractFactory("NFTMarketplace");
  const marketplace = await Marketplace.deploy();

  await marketplace.deployed();
  
  //Pull the address and ABI out while you deploy, since that will be key in interacting with the smart contract later
  const data = {
    address: marketplace.address,
    abi: JSON.parse(marketplace.interface.format('json'))
  }

  //This writes the ABI and address to the marketplace.json
  //This data is then used by frontend files to connect with the smart contract
  fs.writeFileSync('./src/Marketplace.json', JSON.stringify(data))
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
  
```  

Pulse Guardar.

A continuación, abra el símbolo del sistema y ejecute el siguiente comando:

```
npx hardhat run --network goerli scripts/deploy.js
```

>❗️Asegúrate de haber actualizado tu hardhat.config.js según el paso 3 para poder desplegar el contrato inteligente.

Si no ves ningún error o advertencia, ¡tu contrato inteligente se ha desplegado correctamente!

Deberías poder ver la dirección en la que se desplegó y la ABI del contrato inteligente en ``src/Marketplace.json.``

## Paso 8: Añadir las funciones para subir los metadatos NFT a Piñata

En tu directorio home, en el archivo vacío llamado ``pinata.js`` añade este código:

```
//require('dotenv').config();
const key = process.env.REACT_APP_PINATA_KEY;
const secret = process.env.REACT_APP_PINATA_SECRET;

const axios = require('axios');
const FormData = require('form-data');

export const uploadJSONToIPFS = async(JSONBody) => {
    const url = `https://api.pinata.cloud/pinning/pinJSONToIPFS`;
    //making axios POST request to Pinata ⬇️
    return axios 
        .post(url, JSONBody, {
            headers: {
                pinata_api_key: key,
                pinata_secret_api_key: secret,
            }
        })
        .then(function (response) {
           return {
               success: true,
               pinataURL: "https://gateway.pinata.cloud/ipfs/" + response.data.IpfsHash
           };
        })
        .catch(function (error) {
            console.log(error)
            return {
                success: false,
                message: error.message,
            }

    });
};

export const uploadFileToIPFS = async(file) => {
    const url = `https://api.pinata.cloud/pinning/pinFileToIPFS`;
    //making axios POST request to Pinata ⬇️
    
    let data = new FormData();
    data.append('file', file);

    const metadata = JSON.stringify({
        name: 'testname',
        keyvalues: {
            exampleKey: 'exampleValue'
        }
    });
    data.append('pinataMetadata', metadata);

    //pinataOptions are optional
    const pinataOptions = JSON.stringify({
        cidVersion: 0,
        customPinPolicy: {
            regions: [
                {
                    id: 'FRA1',
                    desiredReplicationCount: 1
                },
                {
                    id: 'NYC1',
                    desiredReplicationCount: 2
                }
            ]
        }
    });
    data.append('pinataOptions', pinataOptions);

    return axios 
        .post(url, data, {
            maxBodyLength: 'Infinity',
            headers: {
                'Content-Type': `multipart/form-data; boundary=${data._boundary}`,
                pinata_api_key: key,
                pinata_secret_api_key: secret,
            }
        })
        .then(function (response) {
            console.log("image uploaded", response.data.IpfsHash)
            return {
               success: true,
               pinataURL: "https://gateway.pinata.cloud/ipfs/" + response.data.IpfsHash
           };
        })
        .catch(function (error) {
            console.log(error)
            return {
                success: false,
                message: error.message,
            }

    });
};

```

Las dos funciones son:

1. ``uploadFileToIPFS()``

Esta función carga el archivo de imagen NFT en IPFS y devuelve una URL IPFS que puede consultarse para obtener la imagen.

2.``uploadJSONToIPFS(JSON)``

Esta función toma como entrada el JSON completo que se va a cargar y lo carga en IPFS. El valor devuelto por la función es un URI IPFS que puede ser consultado para obtener los metadatos. Este URI es muy útil cuando queremos recuperar la información de metadatos NFT más tarde.

## Paso 9: Integrar el Frontend con el Smart Contract
Para que la plataforma funcione a la perfección, integra el frontend con funciones del contrato inteligente.

>**📘 Una nota sobre el frontend**
>
>Construir el frontend para esto es una tarea enorme. Aunque nos encantaría enseñarlo todo aquí en este mismo tutorial a nuestros devs, no queremos abrumarte.
>
>Por lo tanto, el repositorio de Github tiene todo el código frontend con componentes separados para cada página por separado.
>
>Cada componente frontend como ``src/components/SellNFT.js`` por ejemplo,
>
>1. Tiene una función que crea un proveedor, un firmante y un objeto de contrato.
>2. Obtiene datos relevantes del contrato inteligente
>3. Obtiene datos relevantes de IPFS a través de Axios
>4. tiene un retorno donde devuelve el JSX/HTML para la página
>
>Aunque en este tutorial nos saltamos hablar del punto 4, seguimos cubriendo los puntos 1, 2 y 3. Publicaremos un futuro tutorial sobre el punto 4 y mantendremos esta página actualizada.

**src/components/SellNFT.js**

La integración más importante será en ``src/components/SellNFT.js`` donde hacemos 3 pasos:

1. Subir la imagen a IPFS
2. Subir los metadatos con una imagen a IPFS
3. Enviar los metadatos tokenURI y el precio al contrato inteligente

Añade el siguiente código a tu archivo ``src/components/SellNFT.js`` justo después de las declaraciones de variables de estado en la parte superior:

```
    //This function uploads the NFT image to IPFS
    async function OnChangeFile(e) {
        var file = e.target.files[0];
        //check for file extension
        try {
            //upload the file to IPFS
            const response = await uploadFileToIPFS(file);
            if(response.success === true) {
                console.log("Uploaded image to Pinata: ", response.pinataURL)
                setFileURL(response.pinataURL);
            }
        }
        catch(e) {
            console.log("Error during file upload", e);
        }
    }

    //This function uploads the metadata to IPDS
    async function uploadMetadataToIPFS() {
        const {name, description, price} = formParams;
        //Make sure that none of the fields are empty
        if( !name || !description || !price || !fileURL)
            return;

        const nftJSON = {
            name, description, price, image: fileURL
        }

        try {
            //upload the metadata JSON to IPFS
            const response = await uploadJSONToIPFS(nftJSON);
            if(response.success === true){
                console.log("Uploaded JSON to Pinata: ", response)
                return response.pinataURL;
            }
        }
        catch(e) {
            console.log("error uploading JSON metadata:", e)
        }
    }

    async function listNFT(e) {
        e.preventDefault();

        //Upload data to IPFS
        try {
            const metadataURL = await uploadMetadataToIPFS();
            //After adding your Hardhat network to your metamask, this code will get providers and signers
            const provider = new ethers.providers.Web3Provider(window.ethereum);
            const signer = provider.getSigner();
            updateMessage("Please wait.. uploading (upto 5 mins)")

            //Pull the deployed contract instance
            let contract = new ethers.Contract(Marketplace.address, Marketplace.abi, signer)

            //massage the params to be sent to the create NFT request
            const price = ethers.utils.parseUnits(formParams.price, 'ether')
            let listingPrice = await contract.getListPrice()
            listingPrice = listingPrice.toString()

            //actually create the NFT
            let transaction = await contract.createToken(metadataURL, price, { value: listingPrice })
            await transaction.wait()

            alert("Successfully listed your NFT!");
            updateMessage("");
            updateFormParams({ name: '', description: '', price: ''});
            window.location.replace("/")
        }
        catch(e) {
            alert( "Upload error"+e )
        }
    }
```
**src/components/Marketplace.js**

Aquí sólo necesitamos extraer todos los NFTs del contrato inteligente.

Añade esto a tu archivo justo después de las declaraciones de variables de estado en la parte superior y antes del retorno:

```
const [dataFetched, updateFetched] = useState(false);

async function getAllNFTs() {
    const ethers = require("ethers");
    //After adding your Hardhat network to your metamask, this code will get providers and signers
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const signer = provider.getSigner();
    //Pull the deployed contract instance
    let contract = new ethers.Contract(MarketplaceJSON.address, MarketplaceJSON.abi, signer)
    //create an NFT Token
    let transaction = await contract.getAllNFTs()

    //Fetch all the details of every NFT from the contract and display
    const items = await Promise.all(transaction.map(async i => {
        const tokenURI = await contract.tokenURI(i.tokenId);
        let meta = await axios.get(tokenURI);
        meta = meta.data;

        let price = ethers.utils.formatUnits(i.price.toString(), 'ether');
        let item = {
            price,
            tokenId: i.tokenId.toNumber(),
            seller: i.seller,
            owner: i.owner,
            image: meta.image,
            name: meta.name,
            description: meta.description,
        }
        return item;
    }))

    updateFetched(true);
    updateData(items);
}

if(!dataFetched)
    getAllNFTs();
    
```

**src/components/Profile.js**
Añade el siguiente código que extrae todos los NFTs que el usuario conectado posee:

```
      const [dataFetched, updateFetched] = useState(false);

    async function getNFTData(tokenId) {
        const ethers = require("ethers");
        let sumPrice = 0;

        //After adding your Hardhat network to your metamask, this code will get providers and signers
        const provider = new ethers.providers.Web3Provider(window.ethereum);
        const signer = provider.getSigner();
        const addr = await signer.getAddress();

        //Pull the deployed contract instance
        let contract = new ethers.Contract(MarketplaceJSON.address, MarketplaceJSON.abi, signer)

        //create an NFT Token
        let transaction = await contract.getMyNFTs()

        /*
        * Below function takes the metadata from tokenURI and the data returned by getMyNFTs() contract function
        * and creates an object of information that is to be displayed
        */
        
        const items = await Promise.all(transaction.map(async i => {
            const tokenURI = await contract.tokenURI(i.tokenId);
            let meta = await axios.get(tokenURI);
            meta = meta.data;

            let price = ethers.utils.formatUnits(i.price.toString(), 'ether');
            let item = {
                price,
                tokenId: i.tokenId.toNumber(),
                seller: i.seller,
                owner: i.owner,
                image: meta.image,
                name: meta.name,
                description: meta.description,
            }
            sumPrice += Number(price);
            return item;
        }))

        updateData(items);
        updateFetched(true);
        updateAddress(addr);
        updateTotalPrice(sumPrice.toPrecision(3));
    }

    const params = useParams();
    const tokenId = params.tokenId;
    if(!dataFetched)
        getNFTData(tokenId);
        
```        
**src/componentes/NFTPage.js**

Esta es la página individual para cada NFT, que sirve para dos funcionalidades:

1. mostrar todos los datos de una NFT en particular
2. permitir a cualquier usuario comprarla con un botón "Comprar este NFT"

Así que pega las siguientes dos funciones en tu código:

```
async function getNFTData(tokenId) {
    const ethers = require("ethers");
    //After adding your Hardhat network to your metamask, this code will get providers and signers
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const signer = provider.getSigner();
    //Pull the deployed contract instance
    let contract = new ethers.Contract(MarketplaceJSON.address, MarketplaceJSON.abi, signer)
    //create an NFT Token
    const tokenURI = await contract.tokenURI(tokenId);
    const listedToken = await contract.getListedTokenForId(tokenId);
    let meta = await axios.get(tokenURI);
    meta = meta.data;
    console.log(listedToken);

    let item = {
        price: meta.price,
        tokenId: tokenId,
        seller: listedToken.seller,
        owner: listedToken.owner,
        image: meta.image,
        name: meta.name,
        description: meta.description,
    }
    console.log(item);
    updateData(item);
}

async function buyNFT(tokenId) {
    try {
        const ethers = require("ethers");
        //After adding your Hardhat network to your metamask, this code will get providers and signers
        const provider = new ethers.providers.Web3Provider(window.ethereum);
        const signer = provider.getSigner();
        //Pull the deployed contract instance
        let contract = new ethers.Contract(MarketplaceJSON.address, MarketplaceJSON.abi, signer);
        const salePrice = ethers.utils.parseUnits(data.price, 'ether')
        let transaction = await contract.executeSale(tokenId, {value:salePrice});
        await transaction.wait();

        alert('You successfully bought the NFT!');
    }
    catch(e) {
        alert("Upload Error"+e)
    }
}

```

## Paso 10: Prueba tu código
Cuando pulses el comando``npm start`` en el terminal, el marketplace debería abrirse en tu localhost y tendrá el siguiente aspecto:


![](https://files.readme.io/6ba380b-topnf.png)

>📘 Si tu código no funciona en este punto consulta el repositorio de GitHub del [tutorial terminado de NFT Marketplace.](https://github.com/OMGWINNING/NFT-Marketplace-Tutorial) Si tiras de esto directamente, ¡el marketplace debería funcionarte!

### Conecta tu marketplace
En primer lugar, conecta tu mercado haciendo clic en el botón "Conectar Cartera" de tu barra de navegación.

Si estás en una red diferente a Goerli, MetaMask primero te pedirá que cambies de red.

Después te pedirá que te conectes a tu cuenta específica.

![](https://files.readme.io/1b4c436-meta.png)
![](https://files.readme.io/86ade0c-learn.png)

**Subir un NFT**

Después de un inicio de sesión con éxito, el mercado probablemente se parece a el de abajo.

Puede que falten NFTs ya que acabas de desplegar el contrato.

![](https://files.readme.io/311f09a-golden.png)
Página de inicio del mercado

Ahora, diríjete a la página "Listar mi NFT" en la barra de navegación y rellena los detalles para cargar el primer NFT. Debería parecerse a la siguiente antes de pulsar enviar:


![](https://files.readme.io/5b974ed-aniket_nft.png)
Formulario de carga del mercado de NFT


>📘 Asegúrate de que tienes algo de Goerli ETH de [Goerli Faucet](https://goerlifaucet.com/) en este punto. Si no tienes suficiente Goerli ETH, la transacción podría fallar debido a fondos insuficientes.

Ahora, si pulsas enviar y esperas un rato (hasta 5 minutos como máximo), deberías ver una alerta que dice "¡Se ha Cargado con éxito tu NFT!".

Si haces clic en Aceptar, se te redirigirá a la página de inicio del mercado.

Ahora, si te diriges al mercado y a tu perfil, deberías ver tu NFT.

### Comprar un NFT

Para probar la funcionalidad de comprar un NFT, primero cambia el monedero en tu Metamask a otro monedero yendo a "Mis Cuentas" en tu extensión de monedero MetaMask.

Aparecerá la siguiente pantalla.

![](https://files.readme.io/31fbb3b-localhost.png)

Si aún no tienes otra cuenta, crea una y [cárgala con Goerli ETH.](https://goerlifaucet.com/)

A continuación, dirigete a la página de un NFT individual y da clic en el botón "Comprar este NFT".

Después de un tiempo de espera, deberías ver una alerta que dice "¡NFT comprado con éxito!".

Ahora, si te diriges a la sección de tu perfil, ¡El NFT debería aparecer!


![](https://files.readme.io/004cab1-wallet-ad.png)
Página de perfil del monedero NFT


¡Voila!

Si todo eso te ha funcionado, ahora has construido con éxito una v1 funcional de un mercado NFT.

¡Legendario!

## Paso 11: [Opcional] Ampliación de la funcionalidad

¿Sabes lo que sería genial? Lo mejor sería que algunos fueran más alla de las funcionalidades que hemos implementado en este tutorial.

Un par de extensiones potenciales podrían ser

+ **Utilizar los puntos finales [getNFTs](https://docs.alchemy.com/reference/nft-api) y [getNFTsForCollection](https://docs.alchemy.com/reference/getnftsforcollection)** de Alchemy para obtener NFTs para el mercado y la página de perfil.
+ **Añadir una funcionalidad que permita a los usuarios listar NFTs** preexistentes en el mercado.
+ **Añadir regalías** para que el creador original de la NFT obtenga el 10% de los ingresos cada vez que se venda la NFT.

Si acabas implementando lo anterior o cualquier otra funcionalidad, ¡etiqueta a [@AlchemyPlatform](https://twitter.com/AlchemyPlatform) y compártelo con nosotros en Twitter! Puede que incluso la compartamos con nuestra comunidad de 40.000 desarrolladores (y creciendo).

Envía tu proyecto aquí para canjear un token de Prueba de Conocimiento (PoK): https://university.alchemy.com/discord

## Conclusión

Con este tutorial, ¡has construido con éxito tu propio mercado NFT desde cero!

¡Enhorabuena por completar la Semana 7 de Road to Web3!

No dudes en añadir más funciones, como utilizar las API de Alchemy y listar NFT antiguas.

Si te ha gustado este tutorial para crear un mercado de NFT, envíanos un tweet [@AlchemyPlatform](https://twitter.com/AlchemyPlatform). (O si tienes alguna pregunta o comentario, dale un toque al autor [@ankg404](https://twitter.com/ankg404)).

No olvides unirte a nuestro [servidor Discord](https://www.alchemy.com/discord) para conocer a otros desarrolladores, constructores y emprendedores de blockchain. También comparte lo que has construido con nosotros :tada::tada:

Siempre buscamos mejorar este viaje de aprendizaje, [¡comparte tus comentarios con nosotros!](https://alchemyapi.typeform.com/roadtofeedback)
