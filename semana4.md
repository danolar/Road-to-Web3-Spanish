# Como crear una galeria para NFTs


Bienvenido a la semana 4 de la serie del camino hacia Web3. En este tutorial, aprenderás a desarrollar una galería de NFTs que muestre los NFTs por la dirección de billetera y dirección de contrato inteligente.

Tener datos públicos almacenados en el blockchain no hace que sean fácilmente accesibles. De hecho consultar la blockchain es una de las mayores dificultades para los desarrolladores, ya que tienen que empezar desde el origen de la cadena para reunir los datos que necesitan para conectar los puntos y responder a sus preguntas.

Digamos que un desarrollador quiere responder a la siguiente pregunta "¿Qué NFTs posee una billetera?".

Parece una pregunta sencilla, pero la respuesta no es nada fácil.

En este caso, el desarrollador tendría que encontrar todos los NFT minteados en una cadena determinada, luego, seguir todas las transferencias para entender quién es el propietario actual de los NFT.

Esta tarea podría llevar semanas. Por suerte, existe una solución: con [una cuenta gratuita de Alchemy](https://alchemy.com/?a=RTW3-Week-4) y [la API de NFT de Alchemy](https://docs.alchemy.com/reference/nft-api), permite a los desarrolladores obtener información de los NFT en la cadena de bloques en milisegundos, no en semanas.

Dado que Alchemy ya ha consultado toda la cadena de bloques y ha indexado sus datos, la API de NFT te permite tener acceso completo a la información.

## Tutorial sobre cómo crear una galería NFT

En este tutorial vas a aprender a utilizar la API de NFT de Alchemy para crear una galería de NFT capaz de obtener NFTs basándose en tres cosas:

1. Dirección de la billetera
2. Dirección de la colección
3. Dirección de la billetera + Dirección de la colección

![IMG1](https://files.readme.io/5266633-Blavk.png)
Ejemplo de galería NFT


### Herramientas necesarias

Utilizaremos las llamadas a la API de NFT de Alchemy para obtener NFTs, sus metadatos, mostrar imágenes, descripciones e IDs, en nuestra galería de NFT, utilizando lo siguiente:

* Next JS
* Tailwind CSS
* API de NFT de Alchemy
* Cuenta de Alchemy

>### 📘 ¿Prefieres ver un vídeo?
>¡Próximamente video en español!

## 1. Crear la configuración del proyecto

Lo primero que tendrás que hacer es crear el paquete Next JS de desarrollo del proyecto e instalar TailwindCSS que se encargará de tus estilos.

Abre tu terminal y escribe el siguiente código:

```
npx create-next-app -e with-tailwindcss nameoftheproject
```

Dirigete a tu proyecto e inicia VSCode:
```
cd nameoftheproject 
```
Ahora que se han creado los paquetes de desarrollo del proyecto, prueba que todo funciona.

Ejecuta el siguiente código en el directorio del proyecto:

```
npm run dev
```

Al navegar a localhost:3000 debería generar la siguiente página:

![](https://files.readme.io/d48c519-nextjs.png)

Pagina cargada luego de navegar a localhost:3000

Como no lo necesitaremos, puedes eliminar todo el código dentro de las etiquetas <div> en pages/index.tsx.

Tú código dentro del archivo index.tsx debería tener ahora el siguiente aspecto

```
import type { NextPage } from 'next'
import Head from 'next/head'
import Image from 'next/image'

const Home: NextPage = () => {
  return (

  )
}

export default Home
```
 
Dado que este tutorial utilizará Javascript y no Typescript, antes de escribir su código, convierte los archivos index.tsx y _app.tsx de archivos ``.tsx`` a archivos ``.jsx.``

* Cambia las extensiones de ambos archivos a ``.jsx``
* Elimina las importaciones en la parte superior de ambos archivos
  
## 2. Crear una página de inicio
  
  
El primer paso para crear una galería de NFT con capacidad de búsqueda es crear las entradas de texto donde se añadirá la dirección del monedero y la dirección de la colección que se utiliza para buscar los NFT.

En la página ``index.jsx``, añade el siguiente código:
  
```
  const Home = () => {
 
  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
      </div>
    </div>
  )
}

export default Home

```

Como puedes observar ya estamos implementando las clases CSS de Tailwind dentro de la propiedad className de nuestras entradas. No vamos a entrar en los detalles de cómo funciona TailwindCSS, si quieres saber más sobre él, puedes consultar [la documentación oficial](https://tailwindcss.com/docs/installation).

### Crear dos variables para almacenar las direcciones de la cartera y la colección

A continuación, tendremos que crear dos variables para almacenar tanto la dirección del monedero como la de la colección que insertaremos en los campos de entrada, utilizando [el hook de react useState().](https://reactjs.org/docs/hooks-state.html)

Importa el hook "useState" de "react" y añade el siguiente código, justo encima de la declaración de retorno del componente "Home":

```
  import { useState } from 'react'

  const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  
  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
      </div>
    </div>
  )
}

export default Home
 
```

Para almacenar el valor de las entradas de texto dentro de las variables "wallet" y "collection", utilice el controlador de eventos "onChange".

El [controlador de eventos "onChange"](https://reactjs.org/docs/handling-events.html) se activará cada vez que cambie el valor del campo de entrada, tomando las entradas y almacenándolas en las respectivas variables usando las funciones setWallet y ``setCollectionAddress``

También querrás reflejar los cambios en tus variables "wallet" y "collection" asignando su valor mostrado en el input.

En las etiquetas de entrada de texto, añade el siguiente código:

```
import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  
  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input onChange={(e)=>{setWalletAddress(e.target.value)}} value={wallet} type={"text"} placeholder="Add your wallet address"></input>
        <input onChange={(e)=>{setCollectionAddress(e.target.value)}} value={collection} type={"text"} placeholder="Add the collection address"></input>
      </div>
    </div>
  )
}

export default Home

```

Tus entradas ahora se almacenarán en sus respectivas variables (las direcciones que escribiremos dentro).

### Comprueba que todo funciona con las "React Developer Tools" de tu navegador

Vamos a realizar el proceso de instalación en Chrome, pero todo esto se aplica también a Mozilla Firefox.

* Ve a la tienda de extensiones del navegador
* Busca [React developer tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
* Haz clic en "Añadir a Chrome".

Una vez descargada e instalada, podrás ver la pestaña de herramientas de desarrollo de React en la vista "inspeccionar" dentro de las herramientas de desarrollo de Chrome.

Vuelve a la terminal, dentro de la carpeta de tu aplicación, e inicia de nuevo tu aplicación escribiendo


```
npx run dev
```

* ve a la página de tu aplicación (localhost:3000)
* haz clic con el botón derecho en cualquier parte de la página
* haz clic en inspeccionar
* haga clic en el símbolo ">>".
* dirígete a "Components".

![](https://files.readme.io/674bdfd-components.png)

En la vista de "Componentes" podremos ver una lista de todos los componentes incluidos en nuestra página y, a la derecha, los diferentes estados guardados en los hooks y otra información sobre la aplicación:

![](https://files.readme.io/0c0dd48-fo.png)

Con las herramientas para desarrolladores de React instaladas, podemos probar los hooks ``useState()`` escribiendo en sus entradas y comprobando si el valor de sus estados se actualiza:

![](https://files.readme.io/8f86eb9-wallet_col.png)

Ahora que ya puedes almacenar la dirección de la billetera y de la colección, vamos a terminar la página de inicio.

## Completar la página de inicio

A continuación, tendrás que añadir un botón que activará las funciones para obtener las información de la API de NFT de Alchemy, y un conmutador para decidir si quieres buscar por la dirección del monedero, la colección o ambas.

* debajo de la entrada de texto de la colección añade una nueva entrada de tipo "checkbox"
* envuélvelo en una etiqueta para añadir algo de texto
* crea el botón

```
 import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  
  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"}>Let's go! </button>
      </div>
    </div>
  )
}

export default Home

```

Ahora que tienes un botón, añade un controlador ``onClick()`` que activará la función ``fetchNFTs()`` para obtener los NFTs en función de la billetera que los posea:


```
import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  
  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"} onClick={
          () => {
        
          }
        }>Let's go! </button>
      </div>
    </div>
  )
}

export default Home

```


Ahora, crea la función ``fetchNFTs()`` para añadirla dentro del controlador ``onClick`` y obten tus NFTs.

Sin embargo, necesitas crear una nueva aplicación para obtener una clave de API de Alchemy.

## 3. Crear una nueva aplicación Alchemy

Navega hasta [Alchemy](https://alchemy.com/?a=RTW3-Week-4) y haz clic en "Create App" para crear una nueva aplicación:


![](https://files.readme.io/0e7ff54-new_app.png)
Crear una nueva aplicación en el panel de control de Alchemy.


Estos son los detalles que hay que añadir:

* Nombre de la aplicación
* Descripción
* Cadena (Ethereum)
* Red (Mainnet)

Seleccionar la red principal de Ethereum permitirá obtener NFTs sólo de Ethereum.

Si quieres obtener NFTs en Polygon u otras cadenas, tendrás que crear una nueva aplicación con la cadena respectiva y cambiar la URL base para reflejar la cadena que quieres usar, por ejemplo, la URL de Polygon sería:

https://polygon-mumbai.g.alchemy.com/v2/YOUR-API-KEY

Ahora que nuestra aplicación Alchemy está en marcha, vamos a crear la función ``fetchNFTs`` para obtener todos los NFTs que posee una dirección.

## 4. Crear la función FetchNFTs

Para obtener los NFTs que posee una dirección de monedero, utilice el EndPoint [getNFTs](https://docs.alchemy.com/reference/getnfts) de la API de NFT de Alchemy.

En el componente de inicio, declara una nueva variable ``useState()``, como hicimos antes con las direcciones de billetera y colección, para almacenar los NFT que obtendremos utilizando la API de NFT de Alchemy:

```
import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  const [NFTs, setNFTs] = useState([])

  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"} onClick={
          () => {
          }
        }>Let's go! </button>
      </div>
    </div>
  )
}

export default Home
```
Ahora, añade la función ``fetchNFTs().`` siempre dentro del componente de la página de inicio

```
const fetchNFTs = async() => {
  let nfts; 
  console.log("fetching nfts");
  const api_key = "A8A1Oo_UTB9IN5oNHfAc2tAxdR4UVwfM"
  const baseURL = `https://eth-mainnet.g.alchemy.com/v2/${api_key}/getNFTs/`;
  var requestOptions = {
      method: 'GET'
    };
   
  if (!collection.length) {
  
    const fetchURL = `${baseURL}?owner=${wallet}`;

    nfts = await fetch(fetchURL, requestOptions).then(data => data.json())
  } else {
    console.log("fetching nfts for collection owned by address")
    const fetchURL = `${baseURL}?owner=${wallet}&contractAddresses%5B%5D=${collection}`;
    nfts= await fetch(fetchURL, requestOptions).then(data => data.json())
  }

  if (nfts) {
    console.log("nfts:", nfts)
    setNFTs(nfts.ownedNfts)
  }
}
```
Lo primero que hay que notar en la función ``fetchNFTs()``, es la palabra clave async, que permitirá obtener datos sin bloquear toda la aplicación.

>📘Lee más sobre el flujo de trabajo [async await de JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).

A continuación, declaramos una nueva variable para almacenar los NFTs que se obtienen, para luego crear la URL base que alimentará el [fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) para recuperar los nfts.

La URL base, según [la documentación de la API de NFT de Alchemy](https://docs.alchemy.com/reference/nft-api), está compuesta por:

* URL base de la aplicación
* Tu clave de API
* Dirección del propietario
* Dirección de la colección - opcional

Si se proporciona la dirección de la colección, la API de NFT filtrará los NFT recuperados por colección, si no se proporciona, la API recuperará todos los NFT propiedad de la dirección de la billetera proporcionada.

Por ello, se necesitará una manera de saber si la dirección de la colección ha sido proporcionada o no, por lo tanto: quieres filtrar por colección.

Para ello, puede añadir una declaración "if" que compruebe si la variable "collection" está vacía o no:

Si está vacía, entonces sólo proporcionas los parámetros de la dirección de la cartera:

```
const fetchURL = `${baseURL}?owner=${wallet}`;
```
Si no lo está, entonces se obtienen todos los NFTs propiedad de una billetera y se filtran por colección:

```
const fetchURL = `${baseURL}?owner=${wallet}&contractAddresses%5B%5D=${collection}`;
```
La cadena "5B%5D" justo después de los parámetros "contractAddresses" especifica que el parámetro "contractAddresses" es un array y no una simple cadena. Esto se debe a que en realidad se puede filtrar por múltiples "contractAddresses", no sólo por uno.

En ambos casos, filtrando por colección o no, quieres introducir tu "baseURL" en una función ``fetch()``, esperar a que llegue el resultado y convertirlo en JSON [usando la función .json().](https://developer.mozilla.org/en-US/docs/Web/API/Response/json)

Al imprimir los datos recuperados por nuestras funciones ``fetch()``, se registraría el siguiente objeto:

![](https://files.readme.io/f2a9d32-jsx.png)

Dentro del objeto recuperado tienes más información de la que necesitas, ya que sólo necesitarás el array que contiene los NFTs que pertenecen a la dirección de la billetera que hemos proporcionado.

Esa es la razón por la que en la función ``setNFTs()`` estás alimentando nfts.ownednfts y no sólo nfts, ya que sólo necesitarás el array ``ownednfts`` para almacenarlo y usarlo después para mostrar tus NFTs.

Ahora que tu función ``fetchNFTs()`` está lista, necesitarás implementar una nueva función para obtener los NFTs por colección sin un propietario de NFT.

## 5. Crear las funciones FetchNFTs by Collection

Para obtener los NFT por colección, puede utilizar el EndPoint [getNFTsForCollection](https://docs.alchemy.com/reference/getnftsforcollection) de Alchemy.

El EndPoint ``getNFTsForCollection`` requerirá dos parámetros:

* ``contractAddress`` - dirección del contrato de la colección de NFT **[cadena]**
* ``withMetadata`` - (opcional) **** si se establece en ``true``, devuelve los metadatos de NFT; en caso contrario, sólo devolverá los tokenIds. Por defecto a ``false`` **[boolean]**

El primer argumento es para especificar la dirección del contrato de la colección que se quiere recuperar.

El segundo argumento especifica a la API de NFT si también desea obtener los metadatos (por ejemplo, título, imagen, descripción, atributos) de las NFT contenidas en la colección o sólo sus ID.

Puedes leer más en la [API de NFT de Alchemy.](https://docs.alchemy.com/reference/getnftsforcollection)

Como hemos hecho antes, copiemos primero el código y entendamos lo que hace:

```
const fetchNFTsForCollection = async () => {
  if (collection.length) {
    var requestOptions = {
      method: 'GET'
    };
    const api_key = "A8A1Oo_UTB9IN5oNHfAc2tAxdR4UVwfM"
    const baseURL = `https://eth-mainnet.g.alchemy.com/v2/${api_key}/getNFTsForCollection/`;
    const fetchURL = `${baseURL}?contractAddress=${collection}&withMetadata=${"true"}`;
    const nfts = await fetch(fetchURL, requestOptions).then(data => data.json())
    if (nfts) {
      console.log("NFTs in collection:", nfts)
      setNFTs(nfts.nfts)
    }
  }
}
```

Es muy similar a la función ``fetchNFTs()`` que construimos antes, con dos diferencias importantes:

* El punto final utilizado
* El valor almacenado en la variable de estado NFTs

Esto es lo que ocurre:

1. Primero estás verificando que la dirección de la colección no está vacía
2. Luego has declarado ``requestOptions`` para decirle a fetch() que tu petición [HTTP será una solicitud "GET"](https://reqbin.com/Article/HttpGet)
3. Por último, construyes la ``baseURL`` pasando el valor de la dirección de la colección como parámetro ``contractAddress`` y el parámetro ``withMetadata`` a true.

Por último, conviertes los datos obtenidos a JSON, utilizando el flujo de trabajo async await + la función ``json().``

Si registramos en la consola el objeto JSON que contiene los NFTs obtenidos, veremos que contiene 2 propiedades:

![](https://files.readme.io/9c7e67e-arrayNft.png)
Ejemplo de registro en la consola


* Siguiente Token
* nfts

En este caso sólo necesitarás la propiedad "nfts" que contiene tu array (matriz) de NFTs pasando "nfts.nfts" en la función ``setNFTs().``

Tu código en este punto debería ser el siguiente:

```
import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  const [NFTs, setNFTs] = useState([])

  const fetchNFTs = async() => {
    let nfts; 
    console.log("fetching nfts");
    const api_key = "A8A1Oo_UTB9IN5oNHfAc2tAxdR4UVwfM"
    const baseURL = `https://eth-mainnet.g.alchemy.com/v2/${api_key}/getNFTs/`;
    var requestOptions = {
        method: 'GET'
      };
     
    if (!collection.length) {
    
      const fetchURL = `${baseURL}?owner=${wallet}`;
  
      nfts = await fetch(fetchURL, requestOptions).then(data => data.json())
    } else {
      console.log("fetching nfts for collection owned by address")
      const fetchURL = `${baseURL}?owner=${wallet}&contractAddresses%5B%5D=${collection}`;
      nfts= await fetch(fetchURL, requestOptions).then(data => data.json())
    }
  
    if (nfts) {
      console.log("nfts:", nfts)
      setNFTs(nfts.ownedNfts)
    }
  }
  
  const fetchNFTsForCollection = async () => {
    if (collection.length) {
      var requestOptions = {
        method: 'GET'
      };
      const api_key = "A8A1Oo_UTB9IN5oNHfAc2tAxdR4UVwfM"
      const baseURL = `https://eth-mainnet.g.alchemy.com/v2/${api_key}/getNFTsForCollection/`;
      const fetchURL = `${baseURL}?contractAddress=${collection}&withMetadata=${"true"}`;
      const nfts = await fetch(fetchURL, requestOptions).then(data => data.json())
      if (nfts) {
        console.log("NFTs in collection:", nfts)
        setNFTs(nfts.nfts)
      }
    }
  }
  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"} onClick={
          () => {
          }
        }>Let's go! </button>
      </div>
    </div>
  )
}

export default Home
```

Ahora que tus funciones para obtener los NFTs están funcionando, tendrás que adjuntarlas al trigger "onClick" del botón que creaste hace unas secciones.

## 6. Activar las funciones FetchNFTs y FetchNFTsByCollection

Lo primero que tendrás que hacer es añadir una nueva variable de estado llamada "fetchForCollection" que comprobará si quieres buscar por colección o por dirección de cartera:

```
import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  const [NFTs, setNFTs] = useState([])
  const [fetchForCollection, setFetchForCollection]=useState(false)


  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"} onClick={
          () => {
         
          }
        }>Let's go! </button>
      </div>
    </div>
  )
}

export default Home
```

Esta variable será manejada por el checkbox input que hemos creado, actualizando su valor en función de si el checkbox está marcado o desmarcado:

* **Marcada:** estamos buscando por colección - la variable de estado será verdadera
* **Desmarcada:** estamos buscando por la dirección de la billetera - la variable de estado será falsa

Para ello, tendrás que añadir otro controlador "onChange" a la entrada, pero esta vez utilizando el valor ["e.target.checked"](https://bobbyhadz.com/blog/react-check-if-checkbox-is-checked), como valor de la entrada de estado, en lugar de e.target.value:

```
import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  const [NFTs, setNFTs] = useState([])
  const [fetchForCollection, setFetchForCollection]=useState(false)


  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input onChange={(e)=>{setFetchForCollection(e.target.checked)}} type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"} onClick={
          () => {
          }
        }>Let's go! </button>
      </div>
    </div>
  )
}

export default Home
```
Podemos volver atrás y comprobar si la variable se actualiza correctamente utilizando las herramientas de desarrollo de React.

Ahora que sabes si estás buscando NFTs por cartera o por colección, asegúrate de que tu botón es capaz de disparar la función correcta basada en nuestra variable "fetchForCollection":

```
import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  const [NFTs, setNFTs] = useState([])
  const [fetchForCollection, setFetchForCollection]=useState(false)


  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input onChange={(e)=>{setFetchForCollection(e.target.checked)}} type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"} onClick={
           () => {
            if (fetchForCollection) {
              fetchNFTsForCollection()
            }else fetchNFTs()
          }
        }>Let's go! </button>
      </div>
    </div>
  )
}

export default Home
```
Aquí estamos esencialmente diciéndole a nuestro botón que:

* si "fetchForCollection" es verdadero
* entonces ejecuta la función fetchNFTsForCollection(), si no, simplemente fetchNFTs.

Para asegurarse de no añadir la dirección de la billetera en la entrada de la dirección del billetera cuando busques NFTs basados en la colección, puedes añadir la propiedad "disabled" a la entrada del monedero, y desactivarla siempre que fetchForCollection sea verdadero:

```
import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  const [NFTs, setNFTs] = useState([])
  const [fetchForCollection, setFetchForCollection]=useState(false)


  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input disabled={fetchForCollection} type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input onChange={(e)=>{setFetchForCollection(e.target.checked)}} type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"} onClick={
           () => {
            if (fetchForCollection) {
              fetchNFTsForCollection()
            }else fetchNFTs()
          }
        }>Let's go! </button>
      </div>
    </div>
  )
}

export default Home
```
Genial, el siguiente paso es visualizar los NFTs. Para ello necesitas crear el componente ``NFTCard.``

## 7. Crear el componente NFT Card

En la carpeta raíz del proyecto, crea una nueva carpeta y llámala "components".

Dentro de esta carpeta crea un nuevo archivo y llámalo "nftCard.jsx".

Tu tarjeta NFT tomará un NFT como prop (aprende más sobre props en la documentación de ReactJS), obtendrás sus metadatos para mostrarlo en una tarjeta, que tendrá el siguiente aspecto:


![](https://files.readme.io/20eb9a4-bored.png)

Para ello, en el archivo que acabas de crear, añade el siguiente código:

```
export const NFTCard = ({ nft }) => {

    return (
        <div className="w-1/4 flex flex-col ">
        <div className="rounded-md">
            <img className="object-cover h-128 w-full rounded-t-md" src={nft.media[0].gateway} ></img>
        </div>
        <div className="flex flex-col y-gap-2 px-2 py-3 bg-slate-100 rounded-b-md h-110 ">
            <div className="">
                <h2 className="text-xl text-gray-800">{nft.title}</h2>
                <p className="text-gray-600">Id: {nft.id.tokenId}</p>
                <p className="text-gray-600" >{nft.contract.address}</p>
            </div>

            <div className="flex-grow mt-2">
                <p className="text-gray-600">{nft.description}</p>
            </div>
        </div>

    </div>
    )
}
```

Como puedes observar, estamos mostrando 5 propiedades:

* Imagen
* Título
* TokenId
* Dirección del contrato
* Descripción

Para acceder a estas propiedades podemos mirar de nuevo el objeto NFT:


![](https://files.readme.io/05d3fb3-nftObject.png)

Así es como se obtiene la imagen NFT

* accede al primer índice del objeto multimedia
* accede a la propiedad gateway dentro de él para obtener la URL de la imagen
* asigna la URL de la imagen a la etiqueta img en el código anterior "media[0].gateway"

Para acceder al título de la NFT sólo tienes que acceder a la propiedad title dentro del propio objeto NFT.

Con la tarjeta NFT, dirigete al archivo ``home.jsx`` e impórtalo para crear la Galería NFT.


## 8. Crear la Galería NFT

En el archivo ``pages>index.js``, justo debajo de tu botón, importa el siguiente código:

```
 import { NFTCard } from "../components/nftCard"
 import { useState } from 'react'

const Home = () => {
  const [wallet, setWalletAddress] = useState("");
  const [collection, setCollectionAddress] = useState("");
  const [NFTs, setNFTs] = useState([])
  const [fetchForCollection, setFetchForCollection]=useState(false)


  return (
    <div className="flex flex-col items-center justify-center py-8 gap-y-3">
      <div className="flex flex-col w-full justify-center items-center gap-y-2">
        <input disabled={fetchForCollection} type={"text"} placeholder="Add your wallet address"></input>
        <input type={"text"} placeholder="Add the collection address"></input>
        <label className="text-gray-600 "><input onChange={(e)=>{setFetchForCollection(e.target.checked)}} type={"checkbox"} className="mr-2"></input>Fetch for collection</label>
        <button className={"disabled:bg-slate-500 text-white bg-blue-400 px-4 py-2 mt-3 rounded-sm w-1/5"} onClick={
           () => {
            if (fetchForCollection) {
              fetchNFTsForCollection()
            }else fetchNFTs()
          }
        }>Let's go! </button>
      </div>
      <div className='flex flex-wrap gap-y-12 mt-4 w-5/6 gap-x-2 justify-center'>
        {
          NFTs.length && NFTs.map(nft => {
            return (
              <NFTCard nft={nft}></NFTCard>
            )
          })
        }
      </div>
    </div>
  )
}

export default Home
```
Esto es lo que ocurre en este código:

1. La NFTCard fue importada en la parte superior del archivo.
2. Dentro del componente de la página de inicio, se creó un nuevo div, abrimos las llaves, y comprobamos si hay NFTs en nuestra variable de estado usando la renderización condicional.
3. Usamos la función map para iterar sobre el array de NFTs y devolver una NFTCard por cada NFT, pasando la propia NFT como prop de la NFTCard.

Cada vez que busquemos NFTs, y almacenemos un array en la variable de estado NFTs devolverá ahora una tarjeta NFT para cada NFT, mostrando su información.

Y ya está. Ya tienes una galería de NFTs que funciona :)

Ahora, vamos a añadir algunos retos divertidos para que los pruebes por ti mismo antes de enviar tu proyecto:

1. Añade un [icono](https://www.flaticon.com/free-icons/copy) junto a las direcciones de NFT para que la gente que vea tu página pueda copiar fácilmente la dirección del contrato.

2. Añade un sistema de paginación para ver más de 100 NFTs, utilizando el parámetro pageKey del [endpoint getNFTs.](https://docs.alchemy.com/reference/getnfts)

Comparte tu proyecto en el discord de la [Universidad de Alchemy para ganar una Prueba de Conocimiento (PoK) NFT](https://university.alchemy.com/discord)

Para crear galerías de NFT como esta u otras aplicaciones web3 interesantes, [regístrate hoy mismo para obtener una cuenta de desarrollador de Alchemy gratuita.](https://alchemy.com/?a=RTW3-Week-4)
