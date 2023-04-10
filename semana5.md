# Conecta APIs a tus Contratos Inteligentes usando Chainlink

Aprende a construir un NFT dinámico que cambie en función de los datos del mercado utilizando oráculos Chainlink que extraigan datos de fuentes externas en un contrato inteligente.

## ¿Qué es un NFT dinámico?

Un [NFT dinámico](https://blog.chain.link/what-is-a-dynamic-nft/) es un token no fungible que puede cambiar en función de determinadas circunstancias.

Por ejemplo, actualmente hay ocho diferentes [NFTs de LaMelo Ball](https://lameloball.io/#/), y cada NFT registra un conjunto diferente de estadísticas del jugador LaMelo, desde rebotes y asistencias hasta puntos anotados, y cambia en función de estas (¿10 asistencias? color diferente - 1 punto anotado, balón diferente).

Los propietarios de NFT dinámicos pueden recibir acceso especial a sorteos y otras ventajas específicas de los NFT en función del rendimiento del jugador LaMelo.

Y se pone aún más interesante;

Uno de estos ocho NFT, el Gold Evolve NFT, viene con una promesa única:

Si LaMelo Ball ganaba el premio al Novato del Año de la temporada 2021 de la NBA, el NFT evolucionaría para reflejar una nueva imagen. LaMelo ganó el premio y la NFT evolucionó.

![NFT dinámica dorada de Lamelo Ball.](https://lh5.googleusercontent.com/9h2EM5WTg1JmXnsRfjUpadHzHGJPu8el1cD-JW28WAoij0fksdIiyfzAeWXAaPokyJIlHR_jQAtm-h2bLTevqOPK5DtsysjEF2mLto5ewKrrVQPuDAIlksmVt_sOA1fdE-0NvBl-mfxIsVspyg)

Empecemos el tutorial.

## Acerca de este Tutorial de NFT Dinámico

En este tutorial, vas a construir un NFT Dinámico utilizando [la red descentralizada de oráculos y criptográficamente segura de Chainlink](https://chain.link/) para obtener y rastrear datos de precios de los activos.

A continuación, utilizarás [la red Chainlink Keepers](https://docs.chain.link/docs/chainlink-keepers/introduction/) para automatizar el contrato inteligente para actualizar los NFT de acuerdo con los datos de precios de los activos que se están rastreando.

Si el precio del mercado sube, el contrato inteligente elegirá aleatoriamente el URI del NFT para que apunte a una de estas tres imágenes alcistas y el NFT se actualizará dinámicamente:

![](https://files.readme.io/504caec-party_bull.png)
![](https://files.readme.io/66c7dc6-simple_bull.png)
![](https://files.readme.io/6058ded-gamer_bull.png)


Si los datos del feed de precios se mueven a la baja, el NFT se actualizará dinámicamente a una de estas imágenes bajistas, ¡que también se seleccionan aleatoriamente!


![](https://files.readme.io/420bd83-beanie_bear.png)
![](https://files.readme.io/0f1a2d4-coolio_bear.png)
![](https://files.readme.io/54dad70-simple_bear.png)


Por último, utilizaremos [la función aleatoria verificable de Chainlink](https://docs.chain.link/docs/chainlink-vrf/) para añadir aleatoriedad criptográficamente garantizada a nuestro contrato inteligente para seleccionar una imagen NFT aleatorea de una lista de opciones.

>📘 Proximamente video en español.

¡A hackear!

## Herramientas necesarias y pre-requisitos

Este tutorial asume que tienes alguna experiencia previa programando y que has seguido el contenido precedente en el programa [Camino hacia Web3](https://docs.alchemy.com/docs/welcome-to-the-road-to-web3) que [Alchemy](https://alchemy.com/?a=6b90d91edc) ha construido para ti.

### 1. IDE
En este tutorial, vamos a utilizar el [IDE de Remix](https://remix.ethereum.org/) y la red de blockchain "London VM" incorporada, pero lo mismo se puede hacer utilizando Hardhat o cualquier otro framework de desarrollo de solidity de contratos inteligentes y tu editor de código favorito.

### 2. Repo de Github
Aquí, hay un [repositorio de Github](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy) para el tutorial de NFT Dinámico que hemos hecho para ti.

El repo refleja la estructura que vamos a seguir.

**La rama principal**

La rama ``main`` contiene la línea de base ERC721 Token utilizando el [Asistente OpenZeppelin.](https://docs.openzeppelin.com/contracts/4.x/wizard)

**La rama price-feeds**

La rama ``price-feeds`` añade la implementación de Chainlink Keepers y se conecta a los datos de precios de Chainlink Asset que utilizaremos para rastrear el precio de un activo específico.

**La rama de aleatoriedad**

La rama de ``randomness``  contiene la lógica para añadir aleatoriedad para que nuestro NFT dinámico se elija al azar de los URI de metadatos NFT que tenemos en nuestro contrato inteligente.

¡Esta parte es para que la hagas como una tarea especial para mejorar tus habilidades!

### 3. Acompañamiento IPFS
Instala [la extensión de navegador IPFS Companion](https://chrome.google.com/webstore/detail/ipfs-companion/nibjojkomfdiaoajekhjakgkdhaomnch?hl=en) (para cualquier navegador basado en Chromium).

Esto contendrá el URI de tu token y la información de metadatos.

### 4. Fuentes y tokens de red de pruebas
Asegúrate de que tu monedero MetaMask está [conectado a Rinkeby.](https://www.alchemy.com/overviews/rinkeby-testnet#testnet-3)

Una vez que tu billetera esté conectada a Rinkeby, obten [Rinkeby ETH en Alchemy.](https://rinkebyfaucet.com/)

También necesitarás [conseguir tokens LINK de testnet.](https://faucets.chain.link/)

Para tu reto, añadirás aleatoriedad, pero te desplegarás en la testnet Goerli de Ethereum.

Si necesitas tokens de testnet Goerli, [consigue Goerli ETH en Alchemy.](https://goerlifaucet.com/)

## 1. Configurar el token ERC721
Empezaremos con OpenZeppelin y Remix para crear un contrato inteligente NFT.

Como esto ya se ha cubierto, tienes dos opciones para obtener tu contrato inteligente NFT:

### i. Crear el contrato desde cero

Sigue las instrucciones de [Welcome to the Road to Web3](https://docs.alchemy.com/docs/welcome-to-the-road-to-web3) y recuerda nombrar el NFT con "Bull&Bear".

Luego dale un nombre de Símbolo "BBTK" y lee atentamente la parte de abajo para actualizar la lógica de ``mintToken()``

![](https://lh4.googleusercontent.com/RkhSkia6FLcWpRLOmwFe3Vcuq647JBBfzc39XlXYOgRIiiI6o5BWhD-mMGaEd4t37qgjsKcC9ViFlV_gaLnLJIAQpyyjXwypKvsNzsezV55hNRf_LDL9cLISqn0uFcLOXWySVyHshhsXdG-k8A)

### ii. Copiar el código del contrato inteligente del repositorio Github de referencia

Esta es probablemente la forma más rápida: [copia el código del contrato inteligente](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/blob/main/contracts/Bull%26Bear.sol) de la rama principal del repositorio de referencia y guárdalo en Remix como ``Bull&Bear.sol.``

Observa cómo hemos añadido las referencias a las matrices de URIs IPFS y también hemos actualizado el método ``safeMint()`` para establecer un URI de token inicial como punto de partida.


## 2. Actualizar los enlaces en los URI de IPFS

Ahora debes asegurarte de actualizar los enlaces en los URIs IPFS para ``bullUrisIpfs`` y ``bearUrisIpfs`` para que apunten a los archivos alojados en tu nodo IPFS del navegador.

Para configurar los datos NFT en su nodo IPFS de navegador:

* Copia los metadatos de token JSON y los archivos de imagen de esta [carpeta en el repositorio.](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/tree/main/ipfs)
* Haz clic en el icono de extensión del navegador del IPFS Companion para abrir tu nodo IPFS local
* Importa todos estos archivos a la sección FILES de tu nodo.
* Para obtener el enlace URI, haz clic en los tres puntos y copia la cadena de "Share Link".

![](https://files.readme.io/bce1c8d-files.png)

> 📘 Necesitarás tanto los JSON como los PNG en tu nodo IPFS, pero sólo los archivos JSON para tu contrato inteligente porque los archivos JSON apuntan a los PNG.

## 3. Completa una comprobación de compilación

* Elige el compilador correcto en Remix basándote en el pragma del contrato inteligente ( 0.8.0 en adelante)
* Asegúrate de que está compilando el archivo correcto- Bull&Bear.sol
* Despliega tu contrato en el entorno JavaScript VM London en el navegador
* Copia la dirección de la billetera proporcionada por Remix y pégala en el campo ``safeMint`` para mintear un token

![](https://files.readme.io/b0a48c8-deployed_contact.png)

* Desplázate hacia abajo y has clic en tokenURI con el argumento "0".

Como tu primer token tiene un ID de token de cero, devolverá el ``tokenURI`` que apunta al archivo JSON "gamer bull".

![](https://files.readme.io/a95aae5-toenuri.png)

Genial - ¡tu contrato inteligente NFT está funcionando!

## 4. Haz compatible tu contrato con Keepers

Ahora, podemos hacer que nuestro contrato NFT no sólo sea dinámico, ¡sino automáticamente dinámico!

Este código está referenciado en la [rama price-feeds](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/tree/price-feeds) del repo.

En primer lugar, añadimos la capa de automatización con Chainlink Keepers, lo que significa que tenemos que ampliar nuestro contrato inteligente NFT para hacerlo "Compatible con Keepers".

### Aquí están los pasos clave:

1. Importar "@chainlink/contracts/src/v0.8/KeeperCompatible.sol"
2. Haz que tu contrato herede de ``KeeperCompatibleInterface``
3. Ajusta tu constructor para tomar un periodo de intervalo que se establece como una variable de estado del contrato y esto establece los intervalos en los que se producirá la automatización.
4. Implementa las funciones ``checkUpkeep`` y ``performUpkeep`` en el contrato inteligente NFT para complementar la interfaz.
5. [Registrar el contrato "upkeep"](https://docs.chain.link/docs/chainlink-keepers/register-upkeep/) en la red Chainlink Keeper Network.

La red de guardianes Chainlink comprobará nuestra función ``checkUpkeep()`` cada vez que se añada un nuevo bloque a la cadena de bloques y simulará la ejecución de nuestra función fuera de la cadena.

**Esa función devuelve un booleano:**

* Si es false, significa que aún no se ha realizado ningúna automatización.
* Si devuelve true, significa que el ``intervalo`` que establecimos ha pasado, y se debe realizar una acción de seguimiento.

La Red de Keepers llama a nuestra función ``performUpkeep()`` automáticamente, y ejecuta la lógica en la cadena.

No es necesaria ninguna acción por parte del desarrollador.

¡Es como magia!

Nuestro ``checkUpkeep`` será sencillo porque sólo queremos comprobar si el ``intervalo`` ha expirado y devolver ese booleano, pero nuestro ``performUpkeep`` necesita comprobar un feed de precios.

Para ello, necesitamos que nuestro Contrato Inteligente interactúe con los oráculos de precios de Chainlinks.

Usaremos el [contrato proxy de alimentación BTC/USD en Rinkeby](https://rinkeby.etherscan.io/address/0xECe365B379E1dD183B20fc5f022230C044d51404), pero puedes [elegir otro](https://docs.chain.link/docs/ethereum-addresses/) de la red Rinkeby.

## 5. Interactuar con los oráculos de precios de Chainlink

Para interactuar con el oráculo Price Feed elegido, necesitamos utilizar ``AggregatorV3Interface.``

>📘 Asegúrate de entender [cómo funcionan los feeds de datos](https://docs.chain.link/docs/using-chainlink-reference-contracts/) y [cómo utilizarlos.](https://docs.chain.link/docs/get-the-latest-price/)

En [nuestro código de referencia en la rama price-feeds](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/blob/price-feeds/contracts/Bull&Bear.sol), el constructor acepta la dirección del agregador del oráculo como un parámetro en el constructor. Aceptar un parámetro en tiempo de despliegue es super útil ya que lo hace configurable cuando desarrollamos localmente.

Para interactuar con un oráculo en vivo en Rinkeby, nuestro contrato necesita ser desplegado en Rinkeby. Esto es necesario para las pruebas de integración, pero durante el desarrollo nos ralentiza un poco.

¿Cómo podemos acelerar nuestro bucle de desarrollo local editar-compilar-depurar?

### Simulando Contratos Inteligentes Live Net

En lugar de redistribuir constantemente a una red de prueba como Rinkeby, pagando ETH de prueba, etc, podemos utilizar [mocks.](https://en.wikipedia.org/wiki/Mock_object)(mientras iteramos en nuestro contrato inteligente)

Por ejemplo, podemos simular el contrato agregador de precios utilizando este [contrato simulado de precios.](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/blob/price-feeds/contracts/MockPriceFeed.sol)

La ventaja es que podemos desplegar el mock en nuestro entorno Remix, en el navegador London VM y ajustar los valores que devuelve para probar diferentes escenarios, sin tener que desplegar constantemente nuevos contratos a las redes en vivo, a continuación, aprobar las transacciones a través de MetaMask y pagar ETH de prueba cada vez.

**Esto es lo que hay que hacer:**

* Copia ese archivo a tu Remix
* Guárdalo como ``MockPriceFeed``
* Despliégalo

Es simplemente importar [el mock que Chainlink ha escrito](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.6/tests/MockV3Aggregator.sol) para el proxy de feed de precios.

>**📘 Nota**
> debes cambiar el compilador a 0.6.x para compilar este mock.

Al desplegar un mock necesitas pasar los decimales con los que el feed de precios calculará los precios.

Puedes encontrar estos decimales en [la lista de direcciones de contratos de precios](https://docs.chain.link/docs/ethereum-addresses/), después de hacer clic en "Mostrar más detalles".


![](https://files.readme.io/22e9b17-dec.png)

El feed BTC/USD toma 8 decimales.

También se debe introducir el valor inicial del feed.

Como elegí al azar el precio del activo BTC/USD, le pasé un valor antiguo que obtuve cuando estaba probando: ``3034715771688``

> 📘Cuando lo despliegues localmente, asegúrate de anotar la dirección del contrato que te da Remix.
> Esto es lo que pasas al constructor de tu Contrato Inteligente NFT para que sepa que debe usar el simulacro como fuente de precios.

![](https://files.readme.io/16bab29-mockV3.png)

También deberias jugar con el simulacro de alimentación de precios desplegado localmente.

Llama a ``latestRoundData`` para ver el último precio de la fuente de precios simulada, y otros datos que se ajusten a la [API de fuente de precios Chainlink.](https://docs.chain.link/docs/price-feeds-api-reference/#latestrounddata)

Puedes actualizar el precio llamando a ``updateAnswer`` y pasando un valor superior o inferior (para simular la subida y bajada de los precios).

Puedes hacer que el precio baje pasando ``2534715771688`` o que suba pasando ``4534715771688``.

Muy útil para probar en el navegador tu contrato inteligente NFT!.

Volviendo al contrato inteligente NFT, asegúrate de actualizarlo para reflejar el [código de referencia.](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/blob/price-feeds/contracts/Bull%26Bear.sol)

**Esto es lo que te sugiero que hagas:**

1. Primero lee este breve documento sobre [cómo hacer compatible nuestro contrato inteligente NFT Keepers](https://docs.chain.link/docs/chainlink-keepers/compatible-contracts/)
2. Leer la forma sencilla de [utilizar los feeds de datos](https://docs.chain.link/docs/get-the-latest-price/)
3. Desplegar [el feed de datos falso](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/blob/price-feeds/contracts/MockPriceFeed.sol)
4. Leer el [código fuente](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.6/tests/MockV3Aggregator.sol) para entender cómo se escriben los contratos inteligentes Chainlink Price Feed

Una vez que hayas leído estos recursos, inténtelo tú mismo.

Si quieres ir directamente a nuestra implementación, está en la rama price-feeds.

Ten en cuenta que hemos establecido ``price feed`` como una variable de estado pública para que podamos cambiarla, utilizando el método ``setPriceFeed()``, y también hemos añadido la lógica NFT dinámica clave para ``performUpkeep().``

Cada vez que la red Chainlink Keepers llame a eso, ejecutará esa lógica en la cadena y si el Chainlink Price Feed informa un precio diferente al que rastreamos por última vez, las URI se actualizan.

>📘 Esta demo no optimiza los costes de gas de actualizar todos los Token URIs en el contrato inteligente. Nos centramos en cómo los NFT pueden hacerse dinámicos.
>
>Los costos de actualizar todos los NFT que están en circulación podrían ser extremadamente altos en la red Ethereum, así que considéralo cuidadosamente, y explora soluciones de capa 2 u otras arquitecturas para optimizar las tarifas de gas.

## Resumiendo el flujo de trabajo
Cuando hayas hecho todo esto, este es el aspecto que tendrá tu flujo de trabajo de pruebas:

**1) Despliega el Mock Price Fee en Remix**

Puedes usar los argumentos del constructor ``8,3034715771688`` para empezar y copiar su dirección.

>📘 Recuerda configurar el compilador Remix en el rango 0.6.x para esto.

**2) Vuelve a desplegar el contrato de token inteligente ``Bull&Bear``**

>📘Recuerda actualizar la versión del compilador.

Para los argumentos del constructor, puedes pasar 10 segundos para el intervalo y la dirección del Mock Price Feed como segundo argumento.

**3) Mintea una o dos fichas**

Mintea uno o dos tokens y comprueba sus ``tokenURIs`` haciendo clic en ``tokenURI`` después de pasar 0, 1, o cualquiera que sea el ID del token minteado que tengas.

Todos los token URI deberían ser por defecto el ``gamer_bull.json.``

**4. Comprueba el constructor del contrato NFT**

Comprueba que el constructor del contrato NFT llama a ``getLatestPrice()`` y que a su vez actualiza la variable de estado ``currentPrice.``

Para ello, haga clic en el botón ``currentPrice`` - el resultado debe coincidir con el precio que ha establecido en su Mock Price Feed.

**5) Pasar un array vacío**

Haz clic en ``checkUpkeep`` e introduce una matriz vacía (``[]``) como argumento. Debería devolver un booleano de true porque pasamos 10 segundos como duración del ``interval`` y habrán pasado 10 segundos desde que se desplegó ``Bull&Bear``.

El [repo de referencia incluye una función setter](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/blob/price-feeds/contracts/Bull%26Bear.sol#L156) para que puedas actualizar el campo intervalo por conveniencia.

>📘Ten en cuenta que cuando despliegues en Rinkeby querrás establecer el intervalo al más largo - cada feed actualiza sus precios agregados a [intervalos configurados](https://ethereum.stackexchange.com/questions/112408/how-often-are-chainlink-price-feeds-updated) o si el precio se desvía por un umbral establecido.
>
>Si configuras tus comprobaciones Keepers con demasiada frecuencia, será un desperdicio de tus tokens LINK de prueba.
>
>Por eso para el mocking le pasamos un intervalo muy corto de 10 segundos, porque no gastamos LINK de prueba, y también porque podemos ejecutar rápidamente ``performUpkeep().``

**6) Asegúrate de que el Mock Price Feed está actualizado**

Asegúrate que el Mock Price Feed se actualiza para devolver un precio diferente del que se tiene almacenado actualmente en el campo ``currentPrice`` del NFT Smart Contract.

Si actualizas el contrato simulado con un número inferior, por ejemplo, es de esperar que el contrato inteligente NFT cambie los NFT para mostrar un token URI "bajista".

**7) Simula la llamada a tu contrato**

Haz clic en ``performUpkeep`` después de pasarle un array vacío. Así es como simula que el contrato es llamado por la Red de Chainlink Keepers de la Cadena en Rinkeby.

No olvides que tienes que desplegarte en Rinkeby y [registrar tu mantenimiento](https://docs.chain.link/docs/chainlink-keepers/register-upkeep/) y conectarte a los precios de Rinkeby como parte de tu tarea.

Como ahora mismo estamos en la red Remix dentro del navegador, tenemos que simular el flujo de automatización llamandonos  ``performUpkeep`` a nosotros mismos.

**8) Comprobar el último precio y actualizar todos los token URIs**

``performUpkeep`` debes comprobar el último precio y actualizar todos los token URIs.

>📘Esto es instantáneo en el navegador Remix. En Rinkeby esto puede tomar algún tiempo.

No necesitas firmar ninguna transacción en MetaMask cuando lo haces localmente, pero cuando te conectas a Rinkeby tendrás MetaMask pidiéndote que firmes transacciones para cada paso.

**9) Actualiza el ``currentPrice`` y comprueba el ``tokenURI``**

Si haces clic en ``currentPrice`` deberías ver el precio basado en el Mock Price Feed actualizado.

A continuación, da clic de nuevo en ``tokenURI``, y deberías ver que el tokenURI ha cambiado.

Si el precio cayera por debajo del nivel anterior, se cambiaría a oso.

Si el último token URI era bajista y el precio aumentó, debería cambiar a token URI alcista.

## Tarea de la semana 5

Esta tarea utiliza una nueva herramienta: la función aleatoria verificable Chainlink.

Esta herramienta proporciona aleatoriedad criptográficamente demostrable y se utiliza ampliamente en juegos y otras aplicaciones en las que la aleatoriedad demostrable y resistente a la manipulación es esencial para obtener resultados justos.

En este momento, hemos codificado qué token URI aparece: el primer URI (índice 0) de la matriz. Tenemos que hacer que sea un número de índice aleatorio para que aparezca una imagen NFT aleatoria como URI del token.

## Estos son los pasos:

**1) Revisa un ejemplo de Chainlink VRF**

Mira el super breve ejemplo de uso de Chainlink VRF - sólo tienes que implementar dos funciones para obtener aleatoriedad criptográficamente demostrable dentro del contrato inteligente NFT.

**2) Actualiza tu contrato inteligente NFT para utilizar dos funciones VRF**

Actualiza tu contrato inteligente NFT para utilizar ``requestRandomWords`` y ``fulfillRandomWords``

**3) Utiliza el mock VRF en la rama de aleatoriedad**

Utiliza [el mock VRF](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/blob/randomness/contracts/MockVRFCoordinator.sol) proporcionado en la rama randomness del repositorio de referencia, y asegúrate de leer cuidadosamente las instrucciones comentadas en el mock VRF para saber exactamente cómo usarlo.

### Despliega tu NFT Dinámico en Rinkeby
Por último, una vez que hayas jugado con el contrato inteligente NFT y hayas conseguido que cambie el tokenURI dinámicamente unas cuantas veces en Remix, conecta Metamask y Remix a Rinkeby y despliega el NFT.

>📘Cuando despliegues el NFT en Rinkeby, puedes seguir utilizando los mocks, pero necesitas desplegarlos también y en el orden correcto.

Completa lo siguiente en el orden correcto:

**1) Conecte su Metamask a Rinkeby**

**2) Adquiere LINK de prueba y ETH de prueba del Chainlink Faucet **.**

Si tienes previsto desplegar el agregador de precios de prueba y actualizarlo posteriormente al agregador de precios de Chainlink Rinkeby, despliegua el mock ahora. Del mismo modo, si tienes intención de realizar pruebas en Rinkeby utilizando el Coordinador VRF simulado, debes desplegarlo en Rinkeby.

**3) Despliegue del contrato inteligente NFT en Rinkeby**

Asegúrate de pasar los parámetros correctos al constructor.

Si estás utilizando los mocks, asegúrate de que se despliegan primero para que puedas pasar sus direcciones Rinkeby al constructor del contrato NFT.

Si utiliza una fuente de precios en tiempo real de Chainlink, su dirección debe ser la misma que la del repositorio de referencia o la dirección de la fuente de precios de Rinkeby que [puedes elegir aquí.](https://docs.chain.link/docs/ethereum-addresses/)

Dado que puedes conectar el "entorno" Remix al contrato NFT desplegado en Rinkeby, y llamar al ``performUpkeep`` del contrato NFT desde Remix, puedes mantener el intervalo corto para la primera ejecución de prueba.

>📘Recuerda aumentar el intervalo llamando a ``setInterval``, de lo contrario la red de Keepers ejecutará su ``performUpkeep`` mucho más a menudo de lo que el Price Feed mostrará nuevos datos.

También puedes cambiar la dirección de tu Price Feed llamando a setPriceFeed y pasando la dirección a la que quieres que apunte.

> 📘Si ``performUpkeep`` comprueba que no hay cambios en el precio, las URI de los tokens no se actualizarán.

**4) Mintea tu primer token, y comprueba su URI a través de Remix.**

Debería ser el ``gamer_bull.json``. Compruébalo en OpenSea si quieres.

**5) Juega con los valores simulados**

Si estás usando los dos mocks, juega con los valores y mira los cambios en los NFTs llamando a ``tokenURI.``

**6) Cambia a los contratos Chainlink en vivo en Rinkeby**

Cuando estés listo para cambiar a los contratos Chainlink en vivo en Rinkeby, actualiza la dirección del feed de precios y el ``vrfCoordinator`` en el contrato NFT llamando a sus funciones setter.

**7) Registra tu contrato inteligente NFT**

A continuación, registra el contrato inteligente NFT que se despliega en Rinkeby como un nuevo "mantenimiento" en el [Chainlink Keepers Registry](https://docs.chain.link/docs/chainlink-keepers/register-upkeep/)

**8) [Crea y financia una suscripción VRF**.**](https://docs.chain.link/docs/get-a-random-number/#create-and-fund-a-subscription)

Si estás utilizando el VRF de Chainlink en vivo en Rinkeby asegúrate de llamar a setVrfCoordinator() para no seguir utilizando tu VRF Mock en Rinkeby.

Si no lo has implementado, eso es parte de tu aprendizaje, y puedes comprobar el [repo de referencia.](https://github.com/zeuslawyer/chainlink-dynamic-nft-alchemy/tree/randomness/contracts)

**9) Comprueba OpenSea en una o dos horas**

Dependiendo de la frecuencia con la que cambien los precios (y si quieres inmediatamente, entonces sigue usando los mocks en Rinkeby).

>📘OpenSea almacena en caché los metadatos y puede que no se muestren durante un tiempo, aunque puedes llamar a ``tokenURI`` y ver los metadatos actualizados.
>
>Puedes intentar [forzar una actualización en OpenSea](https://docs.opensea.io/docs/3-viewing-your-items-on-opensea) con el parámetro ``force_update`` pero puede que no actualice las imágenes a tiempo. El nombre de la NFT debería actualizarse como mínimo.

### Conclusión
¡Enhorabuena! Haz codificado una NFT dinámica que refleja los datos de precios del mundo real, y con aleatoriedad criptográficamente probada y a prueba de manipulaciones que selecciona imágenes de NFT dinámicas!

Envía tu proyecto aquí para canjear tu Proof of Knowledge (PoK) token: https://university.alchemy.com/discord

Si quieres explorar otros sorprendentes casos de uso de esta potente tecnología, echa un vistazo a [16 formas de crear NFT dinámicas utilizando oráculos encadenados.](https://blog.chain.link/create-dynamic-nfts-using-chainlink-oracles/)
