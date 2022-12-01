# Como crear el contrato inteligente de un NFT (ERC721) con Alchemy

Aprende a construir un contrato inteligente de un NFT
___

**Desarrollar un contrato inteligente con Solidity y desplegarlo en la blockchain** puede sonar desalentador al principio: solidity, la seguridad, la optimización del gas, el entorno del desarrollador y las tarifas del gas, son sólo algunas de las cosas que tendrás que pasar para desplegar tu código en la blockchain.

Sin embargo, por suerte, en los últimos meses se han lanzado muchas herramientas para que el trabajo de los desarrolladores de contratos inteligentes sea mucho más sencillo.
Herramientas como **el Asistente de OpenZeppelin** que ofrece a los desarrolladores funcionalidades de un clic y escritura para crear **contratos inteligentes seguros** y compilables en un abrir y cerrar de ojos, usadas con herramientas para desarrolladores de Web3 como **Alchemy**, hacen que la experiencia de escribir y desplegar código en la blockchain sea fácil, rápida y fiable como nunca antes.
En este tutorial, vas a aprender **cómo desarrollar y desplegar un contrato inteligente ERC721 (NFT)** utilizando **Alchemy, OpenZeppelin, Remix y Ethereum Goerli.**

Concretamente, aprenderás:
+ Cómo escribir y modificar el contrato inteligente usando OpenZeppelin y Remix
+ Obtener ETH Goerli gratis usando https://goerlifaucet.com/
+ Desplegarlo en la blockchain de prueba de Ethereum: Goerli, para ahorrar en gastos de gas
+ Alojar los metadatos de los tokens NFT en IPFS usando Filebase
+ Mintear un NFT y visualízalo en OpenSea
 
También puedes seguir el video tutorial:

**Próximamente video**

Empecemos creando el contrato inteligente.

## Desarrollar el contrato inteligente ERC721 con el asistente de contratos de OpenZeppelin.

Como hemos dicho antes, en este tutorial, vas a **utilizar el Asistente de OpenZeppelin para crear el contrato inteligente**, por dos razones principales:

+ Es seguro.
+ Ofrece contratos inteligentes estándar.

Cuando se trata de escribir contratos inteligentes, **la seguridad** es clave. Hay miles de ejemplos de robo de cientos de millones de dólares a causa de vulnerabilidades en contratos inteligentes debido a su mala seguridad.  

*No quieres que alguien te robe todas tus preciadas criptomonedas o NFTs una vez que las despliegues en la blockchain ¿verdad?*

**Uno de los propósitos de OpenZeppelin es este**, siendo uno de los mayores verificadores de estándares de contratos inteligentes (ERC20, ERC721, etc.), permitiendo a los desarrolladores utilizar código auditado a fondo para desarrollar contratos fiables.

Lo primero que hay que hacer para desarrollar nuestro contrato inteligente ERC721 NFT es ir a [la página del asistente de contratos inteligentes de Open Zeppelin.](https://docs.openzeppelin.com/contracts/4.x/wizard)

Una vez en la página, verás el siguiente editor:

![IMG1](https://files.readme.io/9e820fe-erc721.png)


Haz clic en el botón ERC721 en la esquina superior izquierda para seleccionar el tipo de estándar ERC a utilizar y el tipo de contrato que se desea escribir:

![IMG2](https://files.readme.io/828ee6a-Erc721_2.png)


Ahora que has seleccionado el estándar de contrato, en el menú de la izquierda deberías ver una serie de opciones:

Empecemos por elegir el nombre y el símbolo de nuestros Tokens. Haz clic en el cuadro de texto con "MyToken" y dale un nombre, haz lo mismo con el Símbolo, y deja el campo URI base en blanco (el nombre del token será utilizado por OpenSea y Rarible como nombre de la colección).

![IMG3](https://files.readme.io/7c6e435-Alh_set.png)


## Selecciona las características del token NFT (ERC721)

Ahora tendrás que seleccionar las características que quieres integrar en el contrato inteligente, justo después de la sección de configuración, encontrarás la sección de características donde podrás seleccionar los diferentes módulos a incluir en tu contrato inteligente.

![IMG4](https://files.readme.io/acaa48a-Miint.png)

En este caso, vas a seleccionar las siguientes integraciones:

+ **Mintable** - creará una función mint que sólo podrán llamar las cuentas con privilegios
+ **Autoincrement IDs** - asignará IDs con incremento automático a sus NFTs
+ **Enumerable** - le dará acceso a la enumeración de tokens en la cadena y a funciones como "totalSupply", no presentes por defecto en el estandar ERC721 
+ **Almacenamiento URI** - para asociar metadatos e imágenes a cada uno de sus NFTs

![IMG5](https://files.readme.io/d4a7d1d-Screenshot_2022-05-01_at_17.20.48.png)


Por el bien de este tutorial, y porque no quieres crear ningún tipo de Tokenomic alrededor de nuestros NFTs, deja los siguientes módulos sin marcar:

+ **Burnable** - para quemar tokens
+ **Pausable** - para pausar las transferencias de tokens, las ventas, etc.
+ **Votes** - da acceso a características similares a la gobernanza como delegados y votos

Si quieres saber más sobre estos módulos, [consulta la documentación oficial de OpenZeppelin sobre el estándar ERC721.](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721)

Ahora que has seleccionado las características que quieres, el Asistente de OpenZeppelin rellenará el código del Contrato Inteligente, que debería tener el siguiente aspecto:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract Alchemy is ERC721, ERC721Enumerable, ERC721URIStorage, Ownable {
    constructor() ERC721("Alchemy", "ALC") {}

    function safeMint(address to, uint256 tokenId, string memory uri)
        public
        onlyOwner
    {
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }

    // The following functions are overrides required by Solidity.

    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._beforeTokenTransfer(from, to, tokenId);
    }

    function _burn(uint256 tokenId) internal override(ERC721, ERC721URIStorage) {
        super._burn(tokenId);
    }

    function tokenURI(uint256 tokenId)
        public
        view
        override(ERC721, ERC721URIStorage)
        returns (string memory)
    {
        return super.tokenURI(tokenId);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}

```

Es el momento de copiar nuestro código y llevarlo a Remix IDE para modificarlo y desplegarlo en la Blockchain.

## Modifica y despliega tu contrato ERC721 con REMIX IDE

Ahora que tienes tu contrato inteligente ERC721, vamos a modificarlo y desplegarlo en la red de prueba Goerli. Para ello, utilizarás **Remix IDE**, un entorno de desarrollo integrado gratuito y basado en la web, diseñado específicamente para el desarrollo de contratos inteligentes con Solidity.

En primer lugar, como habrás notado, en la parte superior del editor de OpenZeppelin Wizard, está el botón "Open in Remix":

![IMG6](https://files.readme.io/108c79e-Remix.png)


Al hacer clic en él, se abrirá REMIX IDE en una nueva pestaña de su navegador.

## Uso de Remix para modificar el contrato inteligente NFT

Empezando por la parte superior del contrato, está el "SPDX-License-Identifier" que especifica el tipo de licencia bajo la que se publicará tu código - es una buena práctica en las aplicaciones web3 mantener el código abierto ya que asegurará la fiabilidad.


```
// SPDX-License-Identifier: MIT
```
Luego está el pragma - la versión del compilador que querrás usar para compilar el código del contrato inteligente. El pequeño símbolo "^", indica al compilador que todas las versiones entre 0.8.0 y 0.8.9 son adecuadas para compilar nuestro código.

```
pragma solidity ^0.8.4;
```

Luego importamos las siguientes bibliotecas e inicializamos el contrato inteligente.

```
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

```

A continuación, vamos a inicializar el contrato, heredando todos los estándares que estamos importando del repositorio de OpenZeppelin:

```
contract Alchemy is ERC721, ERC721Enumerable, ERC721URIStorage, Ownable {...}
```
Como puedes notar, la función safeMint tiene el modificador "only owner" - esto permitirá sólo al propietario del contrato inteligente (la dirección de la cartera que desplegó el contrato inteligente) mintear los NFTs. Lo más probable es que quieras que cualquiera pueda mintear NFTs, para ello tendrás que eliminar el modificador onlyOwner de la función Mint.

```
function safeMint(address to, string memory uri) public {
        uint256 tokenId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }

```

También se puede eliminar de la declaración del contrato "Ownable" y la importación de su biblioteca.


```
import "@openzeppelin/contracts/access/Ownable.sol";

```

Ahora que todo el mundo puede mintear nuestros NFTs, habrá que evitar que la gente haga mint de más NFTs que el número máximo de NFTs de nuestra colección. Para ello vamos a especificar el número máximo de NFTs minteables.

Digamos que queremos que los usuarios puedan mintear hasta un total de 10.000 NFT. Para ello, crearemos una nueva variable uint256, llamada MAX_SUPPLY, y asignémosle 10.000.

```
Counters.Counter private _tokenIdCounter;
    uint256 MAX_SUPPLY = 100000;

    constructor() ERC721("Alchemy", "ALCH") {}
```

A continuación, pasemos a la función safeMint y añadamos una declaración de requisito en la línea 18:

```
require(_tokenIdCounter.current() <= MAX_SUPPLY, "Lo siento llegamos al máximo");
```

Dediquemos un par de palabras a entender mejor qué es la declaración "require" en Solidity.

Puedes leer más sobre la declaración "require" de Solidity, [en la documentación oficial.](https://docs.soliditylang.org/en/v0.4.24/control-structures.html#error-handling-assert-require-revert-and-exceptions)

Ahora que hemos limitado el suministro máximo de nuestros NFTs, es el momento de compilar el contrato inteligente y desplegarlo en la red de prueba Goerli. Para ello, necesitarás crear una cuenta gratuita en [Alchemy.com](https://alchemy.com/?a=roadtoweb3weekone), añadirla como proveedor de nodos en Metamask, y conseguir algo de ETH de Goerli gratis.

## Crear una cuenta gratuita en Alchemy
En primer lugar, tenemos que navegar a [alchemy.com](https://alchemy.com/?a=roadtoweb3weekone) hacer clic en "Login" y crear una nueva cuenta:

![IMG7](https://files.readme.io/56d3600-web3_eco.png)


Seleccionamos Ethereum

![IMG8](https://files.readme.io/7719099-ether_eco.png)


Dale un nombre a tu aplicación y a tu equipo, elige la red Goerli y haz clic en crear App:

![IMG9](https://files.readme.io/45f0d4b-Screen_Shot_2022-09-22_at_10.00.53_AM.png)

Crear App

Una vez que hayas completado el proceso de inscripción, serás redirigido al dashboard. Haz clic en la aplicación con el nombre que hayas decidido, en este caso, "test", haz clic en el botón "VER CLAVE" en la esquina superior derecha, y copia la URL HTTP:

![IMG10](https://files.readme.io/d2c8933-Screen_Shot_2022-09-22_at_10.13.54_AM.png)

Detalles de tu aplicación

A continuación, tendrás que añadir Alchemy a Metamask como proveedor RPC de Goerli. Si no tienes Metamask instalado, asegúrate de seguir esta guía para añadirlo a tu navegador y crear una nueva billetera.

## Añadir Alchemy Goerli a la billetera Metamask
Una vez instalado Metamask, haz clic en el menú desplegable de la Red, y en "añadir red".

![IMG11](https://files.readme.io/af70d3f-Screen_Shot_2022-09-22_at_1.28.09_PM.png)

Serás redirigido a la siguiente página, donde tendrás que rellenar la información de la red Goerli y la URL RPC.

![IMG12](https://files.readme.io/6ca3599-Screen_Shot_2022-09-22_at_1.39.28_PM.png)

Rellena la Red Goerli y la Url RPC

Añade la siguiente información al formulario:

+ **Nombre de la red:** Alchemy Goerli
+ **Nueva URL RPC:** la URL HTTP de la aplicación Goerli Alchemy
+ **ID de la cadena:** 5
+ **Símbolo de la moneda:** GoerliETH
+ **Explorador de bloques:** https://goerli.etherscan.io

¡Genial, acabas de añadir Goerli a Metamask, usando Alchemy! 🎉

Ahora es el momento de desplegar nuestro Contrato Inteligente en Goerli, pero primero, necesitarás conseguir algo de ETH de prueba de Goerli.

## Conseguir ETH de prueba de Goerli gratis

Conseguir Goerli Test ETH es súper sencillo, sólo tienes que navegar por goerlifaucet.com, copiar la dirección de tu billetera en la barra de texto y hacer clic en "Send Me ETH":

![IMG13](https://files.readme.io/f3d2940-goe.png)

Después de 10-20 segundos verás que el ETH de Goerli aparece en la billetera de Metamask.

Podrás obtener hasta 0,1 ETH cada 24 sin iniciar sesión, o 0,5 con una cuenta de Alchemy.

Ahora que tienes el ETH de prueba, es el momento de compilar y desplegar nuestro contrato inteligente NFT en la blockchain.

## Compilar y desplegar el contrato inteligente NFT en la red de prueba Goerli

De vuelta en Remix, hagamos clic en el menú del compilador en el lado izquierdo de la página y hagamos clic en el botón azul "Compilar":

![IMG14](https://files.readme.io/c0c058f-solid_compiler.png)

A continuación, has clic en el menú "Deploy and Run Transactions", has clic en el menú desplegable Environment y selecciona "Injected Web3":

**Asegúrate de que la billetera de Metamask está en la red Alchemy Goerli**, selecciona el contrato inteligente NFT en el menú desplegable Contract y haz clic en Deploy.

![IMG15](https://files.readme.io/e249393-dep.png)

Aparecerá una ventana emergente de Metamask, haz clic en "firmar", y procede a pagar las tasas de gas.

Si todo ha funcionado como se esperaba, después de 10 segundos deberías ver el contrato en la lista de Contratos Desplegados:

![IMG16](https://files.readme.io/6630f89-deployed_contracts.png)

Ahora que el contrato inteligente está desplegado en la red de pruebas de Goerli, es el momento de mintear nuestros NFTs, pero primero, necesitarás crear y subir los metadatos en IPFS, vamos a entender lo que queremos decir con el término "metadatos".
## ¿Qué son los metadatos de los NFTs?

![IMG17](https://files.readme.io/8a5f1ab-octopus.png)
Para que OpenSea pueda obtener metadatos para los tokens ERC721 fuera de la cadena, **el contrato tendrá que devolver un URI que apunte a los metadatos alojados.** Para encontrar este URI, OpenSea, Rarible y otros mercados populares **utilizarán el método tokenURI contenido en el estándar ERC721Uristorage.**

La función tokenURI en el ERC721 debería devolver una URL HTTP o IPFS, como:

ipfs://bafkreig4rdq3nvyg2yra5x363gdo4xtbcfjlhshw63we7vtlldyyvwagbq. Cuando se consulta, esta URL debería devolver un bloque de datos JSON con los metadatos de su ficha.

Puedes leer más sobre los estándares de metadatos en [la documentación oficial de OpenSea.](https://docs.opensea.io/docs/metadata-standards)

## Cómo formatear sus metadatos NFT

Según la documentación de OpenSea, los metadatos NFT deben almacenarse en un archivo ´´´.json´´´ y estructurarse de la siguiente manera:

```
{ 
  "description": "TU DESCRIPCION",
  "external_url": "TU URL",
  "image": "LA URL DE LA IMAGEN",
  "name": "TITULO", 
  "attributes": [
    {
      "trait_type": "base", 
      "value": "estrella de mar"
    }, 
    {
      "trait_type": "Ojos", 
      "value": "Grandes"
    }, 
    {
      "trait_type": "Boca", 
      "value": "Sorprendida"
    }, 
    {
      "trait_type": "Nivel", 
      "value": 5
    }, 
    {
      "trait_type": "Estamina", 
      "value": 1.4
    }, 
    {
      "trait_type": "Personalidad", 
      "value": "triste"
    }, 
    {
      "display_type": "Numero de potencia", 
      "trait_type": "Poder de agua", 
      "value": 40
    }, 
    {
      "display_type": "Porcentaje de potencia", 
      "trait_type": "Incremento de Estamina", 
      "value": 10
    }, 
    {
      "display_type": "Numero", 
      "trait_type": "Generación", 
      "value": 2
    }]
  }

```


A continuación, una breve explicación de lo que almacena cada propiedad:

| propiedad | Explicación |
| --- | --- |
|image | Es la URL de la imagen del elemento. Puede ser casi cualquier tipo de imagen (incluyendo SVGs, que serán cacheados en PNGs por OpenSea), y pueden ser URLs o rutas IPFS. Se recomienda utilizar una imagen de 350 x 350.|
|image_data| Datos de imagen SVG sin procesar, si desea generar imágenes sobre la marcha (no se recomienda). Utilízalo sólo si no incluyes el parámetro de la imagen. |
|external_url:|	Esta es la URL que aparecerá debajo de la imagen del activo en OpenSea y permitirá a los usuarios salir de OpenSea y ver el elemento en su sitio. |
|description| Una descripción legible del elemento. Se admite Markdown.|
|name| Nombre del elemento|
|attributes| Estos son los atributos del elemento, que aparecerán en la página de OpenSea del elemento. (ver más abajo) |
|background_color| Color de fondo del elemento en OpenSea. Debe ser un hexadecimal de seis caracteres sin un preámbulo|
|animation_url| Una URL a un archivo adjunto multimedia para el elemento. Se admiten las extensiones de archivo GLTF, GLB, WEBM, MP4, M4V, OGV y OGG, así como las extensiones de audio MP3, WAV y OGA. Animation_url también es compatible con las páginas HTML, lo que nos permite construir experiencias ricas y NFTs interactivas utilizando el lienzo de JavaScript, WebGL, y más. Ahora se admiten scripts y rutas relativas dentro de la página HTML. Sin embargo, no se admite el acceso a las extensiones del navegador. youtube_url Una URL a un vídeo de YouTube|

Ahora que tenemos una breve comprensión de lo que contendrán los metadatos de tus tokens, vamos a aprender a crearlos y almacenarlos en IPFS.

## Creación y carga de los metadatos en IPFS
En primer lugar, navega a [filebase.com](https://filebase.com/) y crea una nueva cuenta.

Una vez iniciada la sesión, haz clic en el botón de almacenamiento en el menú lateral izquierdo, y crea un nuevo almacenamiento:

![IMG18](https://files.readme.io/6ecc402-metadata.png)

Navega en la carpeta, haz clic en el botón de subir, y sube la imagen que quieres usar para tu NFT, [yo usaré la siguiente.](https://ipfs.filebase.io/ipfs/bafybeihyvhgbcov2nmvbnveunoodokme5eb42uekrqowxdennt2qyeculm)

Una vez subida haz clic en ella y copia la URL de la pasarela IPFS:
![IMG19](https://files.readme.io/8767656-cid.png)

Usando cualquier editor de texto, **pega el siguiente código JSON**

```
{ 
  "description": "This NFT proves I've created and deployed my first ERC20 smart contract on Goerli with Alchemy Road to Web3",
  "external_url": "Alchemy.com/?a=roadtoweb3weekone",
  "image": "https://ipfs.filebase.io/ipfs/bafybeihyvhgbcov2nmvbnveunoodokme5eb42uekrqowxdennt2qyeculm",
  "name": "A cool NFT", 
  "attributes": [
    {
      "trait_type": "Base", 
      "value": "Starfish"
    }, 
    {
      "trait_type": "Eyes", 
      "value": "Big"
    }, 
    {
      "trait_type": "Mouth", 
      "value": "Surprised"
    }, 
    {
      "trait_type": "Level", 
      "value": 5
    }, 
    {
      "trait_type": "Stamina", 
      "value": 1.4
    }, 
    {
      "trait_type": "Personality", 
      "value": "Sad"
    }, 
    {
      "display_type": "boost_number", 
      "trait_type": "Aqua Power", 
      "value": 40
    }, 
    {
      "display_type": "boost_percentage", 
      "trait_type": "Stamina Increase", 
      "value": 10
    }, 
    {
      "display_type": "number", 
      "trait_type": "Generation", 
      "value": 2
    }]
  }

```


Y guarda el archivo como "metadata.json". Vuelve a Filebase y sube el archivo ``metadata.json`` en el mismo bucket donde subimos la Imagen.

![IMG20](https://files.readme.io/6ecc402-metadata.png)


Por último, haz clic en el CID y cópialo, lo necesitaremos en la siguiente parte para construir el URI del token al mintear el NFT:

![IMG21](https://files.readme.io/8767656-cid.png)


## Mintea tu NFT de Goerli

Vuelve a Remix y en el menú Deploy & Run Transactions, ve a "deployed contracts" - y haz clic en el contrato que acabamos de desplegar, se abrirá una lista de todos los métodos que contiene tu contacto inteligente:

![IMG22](https://files.readme.io/0a614b5-rin_nft.png)

**Los métodos naranjas** son métodos que realmente escriben en la blockchain mientras que **los métodos azules** son métodos que leen de la blockchain.

Haz clic en el icono desplegable del método safeMint y **pega tu dirección** y la siguiente cadena en el campo uri:

``` ipfs://\<your\_metadata\_cid> ```

Al hacer clic en "transact" se creará una ventana emergente de Metamask que te pedirá que pagues las tasas de gas.

Has clic en "firmar" y continua para mintear tu primer NFT.

Espera un par de segundos y, para asegurarte de que el minteo se ha realizado con éxito, copia y pega tu dirección en la entrada del método balanceOf, y ejecútalo - debería mostrar que tienes 1 NFT.

Haz lo mismo con el método tokenUri, insertando "0" como argumento de id - debería mostrar tu tokenURI.

Muy bien. ¡Acabas de mintear tu primer NFT! 🎉

Ahora es el momento de pasar a OpenSea para comprobar si los metadatos son de lectura correcta.

## Visualiza tu NFT en OpenSea

Navega a [testnets.opensea.io](https://testnets.opensea.io/) e inicia sesión con tu cartera Metamask. A continuación, haga clic en su imagen de perfil, debería ver allí su NFT recién minteado. Si la imagen aún no es visible, haz clic en ella, y haz clic en el botón "refrescar metadatos".

![IMG23](https://files.readme.io/3256828-testnet.png)


A veces OpenSea tiene dificultades para reconocer los metadatos de la red de prueba, y puede tardar hasta 6 horas en verlos. Después de algún [tiempo tu NFT debería ser visible como el siguiente:](https://testnets.opensea.io/assets/mumbai/0x5a411430964664412e69cff1134759f6bb57c5d7/1)

![IMG24](https://files.readme.io/b36cf81-erc_testnet.png)


**Felicidades, has creado, modificado y desplegado con éxito tu primer contrato inteligente. ¡Has minteado tu primer NFT, y has publicado tu imagen en IPFS!** 🔥

**¿El siguiente paso?** ¿Por qué no modificas tu contrato inteligente para permitir a los usuarios mintear sólo hasta un cierto número de NFTs? 5 por usuario debería ser suficiente, ¡o alguien podría empezar a mintear miles de NFTs!

Para hacerlo, mira el tipo de mapeo, aquí hay una guía increíble para guiarte.

Para ganar un NFT de Prueba de Conocimiento, completa el desafío anterior y luego comparte una reflexión en [el servidor Discord de la Universidad de Alchemy en el canal #proof-of-knowledge submission](https://university.alchemy.com/discord)

¿Quieres la versión en vídeo de este tutorial? Suscríbete [al canal de YouTube de Alchemy](https://www.youtube.com/channel/UCtvTdPZWUwW4whk9CLlCBug) y únete a nuestra comunidad de [Discord](https://discord.gg/3AyCvMJrAr) para encontrar miles de desarrolladores dispuestos a ayudarte.

Siempre buscamos mejorar este viaje de aprendizaje, ¡comparte cualquier comentario que tengas con nosotros! Puedes tuitear a la comunidad etiquetando a [@AlchemyLearn](https://twitter.com/alchemylearn), o puedes sugerir ediciones a este documento haciendo clic en "Sugerir ediciones" en la parte superior derecha.

¡Nos vemos en el próximo reto!
