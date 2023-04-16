# Cómo construir una dapp de intercambio de tokens con la API 0x

Aprende a crear una aplicación de intercambio de tokens (un simple Matcha.xyz) utilizando el endpoint de la API de intercambio 0x. Esta aplicación agrega liquidez en todo el ecosistema DEX y muestra el mejor precio al usuario.

¿Alguna vez has entrado en tu dapp de intercambio de tokens favorita para intercambiar ETH por DAI y te has preguntado cómo ha encontrado el mejor precio para ti?

![](https://files.readme.io/cceeaeb-Matcha.png)

Comerciando WETH<>DAI usando Matcha.xyz, un agregador DEX.

Es más que probable que utilice un [agregador de liquidez](https://docs.0x.org/introduction/introduction-to-0x#makers-and-takers) que busca todos los precios posibles **fuera de la cadena** (por ejemplo, creadores de mercado, libros de órdenes) y **dentro de la cadena** (por ejemplo, DEX, AMM) y dirige el mejor precio para el usuario.

En este tutorial, aprenderemos a utilizar el [punto final de intercambio de la API 0x](https://docs.0x.org/0x-api-swap/introduction), que permite a los usuarios **obtener cotizaciones disponibles a través de la oferta de liquidez** y utiliza el [enrutamiento inteligente de órdenes](https://blog.0xproject.com/0x-apis-smart-order-routing-7af0195515e5) para dividir una transacción a través de redes de intercambio descentralizadas para que se complete con **el menor deslizamiento posible**, minimizando al mismo tiempo los costes de transacción.

Este es el mismo endpoint que está detrás de los swaps en las principales carteras e intercambios como MetaMask, Coinbase wallet, Zapper, y [muchos más.](https://blog.0x.org/0x-ecosystem-update-18/)

Ten en cuenta que no necesitaremos escribir ningún contrato inteligente para encontrar y liquidar el intercambio. En su lugar, la API 0x permite a los desarrolladores web3 acceder fácilmente a los contratos inteligentes del Protocolo 0x que se encargan de toda la lógica utilizada para liquidar una operación, permitiendo a los desarrolladores web centrarse en construir la mejor experiencia de intercambio.

Al final de este tutorial, aprenderás a hacer lo siguiente:

+ Entender por qué es importante la **Agregación de Liquidez**
+ Consultar y mostrar una **lista de tokens ERC20**
+ Utilizar **0x API/swap Endpoint**
+ Establecer una **asignación de tokens**
+ Construir un **Simple Token Swap DApp** que se conecta a MetaMask usando **web3.js**

Versión video tutorial proximamente:

### Requisitos previos

Para prepararse para el resto de este tutorial, necesitas tener:

+ npm (npx) versión 8.5.5
+ node versión 16.13.1
+ [MetaMask](https://metamask.io/download/)

Lo siguiente no es necesario, pero extremadamente útil:

+ Cierta familiaridad con la [línea de comandos](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Command_line)
+ Cierta familiaridad con [JavaScript](https://www.codecademy.com/learn/introduction-to-javascript)
+ [Visual Studio Code](https://code.visualstudio.com/), mi IDE de codificación recomendado

Aquí está el [repositorio de github](https://github.com/0xProject/swap-demo-tutorial) si alguna vez necesitas ayuda.

¡Ahora vamos a empezar a construir nuestra dapp swap!

## Parte 1. Código de inicio

Lo primero que tienes que hacer es clonar el código de inicio del proyecto. Estaremos construyendo sobre este código de inicio para añadir la funcionalidad de intercambio de tokens.

```
git clone git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-1
cd swap-demo-tutorial-part-1
```

Abre el proyecto en tu IDE favorito e inicia un servidor de desarrollo local. Si utilizas [Visual Studio Code](https://code.visualstudio.com/), una forma de hacerlo es instalar [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) desde las extensiones del panel izquierdo. A continuación, haz clic con el botón derecho en el archivo ``index.html`` desde la ventana del explorador y haz clic en Abrir con ``Live Server`` para abrir el esqueleto del Agregador DEX.

![](https://lh3.googleusercontent.com/dq7ceae35BrwYU1HjNFasJjbfRm9tb_vsrqYWSDyGv8B6S4J8VuAxGRVaTKCq82qrsArPuphnnsoUISyirZrfQWaqDn8B6A2-p_sGi0m8hAeiNEOIoxJnpLXr5A1fjowDzObe7xnEQI7nr4j1VLGjg)

![](https://lh4.googleusercontent.com/bzTi7_w6lc7P_75rd9ER2vvp6YGLogL5vIs1IsTTW3nyJMJjL_mYkM1JzCNwSdscvUKh20QtmqMuy5v6pmav7738GKc09YJhGyggHdEo8wmd7rTiUZLOl9ibX1V_dPLfxCvAs-I3Mkd2hQz-ToeiVQ)

![](https://lh6.googleusercontent.com/krrMZkHHPyKP0_k2Yl8svN0V_a0n-mCzaakO6SeaRqPsprbQoTWd-mLInI6uOP1BOe1V5h_rRsD-14_ioNXt4cLOeZWqHpMdwHUfGemtzGNlmwVmjXrISnFpFVYzLvXcB_2lSQ64o7sJZE42oBus3Q)

Un par de características clave para tener en cuenta (el código que se encuentra en ``index.html``):

+ **Botón de inicio de sesión con MetaMask** - Cuando esta aplicación esté completa, al hacer clic en este botón el usuario podrá conectarse a su cartera MetaMask y activar el botón "Swap".
+ **Caja de intercambio**
  + **SELECCIONAR UN TOKEN** - Actualmente estas secciones sólo cambian de color cuando el cursor pasa sobre ellas; al final, los usuarios podrán hacer clic y mostrar una lista de tokens disponibles para intercambiar.
  + **cantidad** - Este es un formulario de entrada que permite al usuario introducir un número.
+ **Gas estimado** - El punto final de intercambio devolverá la fee de gas estimado para que se realice el intercambio. Lo mostraremos aquí.
+ **Botón Swap** - Como se mencionó anteriormente, el botón "Swap" está actualmente desactivado, pero lo habilitaremos cuando el usuario haya iniciado sesión en MetaMask.

Echa un vistazo alrededor de estos elementos en ``index.js`` tomando nota de sus ``ids``, ``clase``, así como su correspondiente estilo en ``style.css.``

## Parte 2. Conectarse a MetaMask

Ahora vamos a permitir al usuario conectarse a [MetaMask.](https://metamask.io/)

Primero descarga [MetaMask](https://metamask.io/download/) en tu navegador. MetaMask es un monedero criptográfico que te permite interactuar con aplicaciones blockchain. Nos guardará los fondos que utilizaremos para el comercio. En esta app, sólo queremos que los usuarios tengan la posibilidad de realizar un intercambio si tienen una cartera MetaMask instalada.

A continuación, ``index.js``, crea una función ``connect()`` y conéctala al elemento ``login_button``. Por favor, lee los comentarios para entender cómo comprobamos que MetaMask está conectado:\

```
async  function  connect() {
/** MetaMask injects a global API into websites visited by its users at `window.ethereum`. This API allows websites to request users' Ethereum accounts, read data from blockchains the user is connected to, and suggest that the user sign messages and transactions. The presence of the provider object indicates an Ethereum user. Read more: https://ethereum.stackexchange.com/a/68294/85979**/

// Check if MetaMask is installed, if it is, try connecting to an account
    if (typeof  window.ethereum !== "undefined") {
        try {
            console.log("connecting");
            // Requests that the user provides an Ethereum address to be identified by. The request causes a MetaMask popup to appear. Read more: https://docs.metamask.io/guide/rpc-api.html#eth-requestaccounts
            await  ethereum.request({ method:  "eth_requestAccounts" });
        } catch (error) {
        console.log(error);
        }
        // If connected, change button to "Connected"
        document.getElementById("login_button").innerHTML = "Connected";
        // If connected, enable "Swap" button
        document.getElementById("swap_button").disabled = false;
        } 
        // Ask user to install MetaMask if it's not detected 
        else {
        document.getElementById("login_button").innerHTML =
            "Please install MetaMask";
        }
    }
// Call the connect function when the login_button is clicked
document.getElementById("login_button").onclick = connect;
```

Ahora cuando abras ``index.html`` con el LiveServer, y hagas click en el botón "Sign-In with MetaMask", ese botón debería actualizarse automáticamente a "Connected" como se muestra a continuación:

![](https://lh6.googleusercontent.com/vS-zh0HwldlZgy_K6rToK54ALV-gMHkrwiOR2MafQBJRqTbYiU6A4_uSsy8fb4M_y0rVp-G6WYN_Q_06yPUgMcap0txzzymd2hqOPe5Adyb_ZWFuAzQrFgjtQB3kA_mT1tA2RGt202lO9gcs5meK7A)

Si MetaMask no lo está, el texto cambiará a "Por favor, instale MetaMask":

![](https://lh4.googleusercontent.com/G0igGEN7X1Ru8eVygU_5PnDXGifNez5mexehTOpRMesFkKxyI36Zkd34J_UBBSGsUXRmw0wuI6YQC--w-7Q3VdJPgcB3k5zpTWDtyZoKrTibHfOCdnelHnLZvULVU_79lBXLpuMa2OHJiOMgH9hJwg)

Genial. Ahora nuestra aplicación puede detectar cuando un usuario tiene un monedero conectado.

**Código Final para la Parte 2**

```
git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-2
```

## parte 3. crear un modal para la lista de tokens

Veamos ahora cómo crear una lista de tokens cuando un usuario hace clic en "SELECCIONAR UN TOKEN". 
podemos usar [el componente Modal de Bootstrap](https://getbootstrap.com/docs/4.0/components/modal/) para ayudarnos con esto.

copia y pega el código del Modal de ejemplo desde [aquí](https://getbootstrap.com/docs/4.0/components/modal/), y pégalo debajo del último ``</div>.``
puedes quitar los botones de guardar y cerrar. 
también, asegúrate de añadir el id="token_modal" para que podamos hacer referencia a este modal más tarde. 
a primera línea del modal debería ser como:

```
<div  class="modal"  id="token_modal"  tabindex="-1"  role="dialog">
```
El ``index.html`` debería tener ahora este aspecto:

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <title>Javascript Test</title>
    <script src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js" integrity="sha256-/xUj+3OJU5yExlq6GSYGSHk7tPXikynS7ogEvDej/m4=" crossorigin="anonymous"></script>
    <script src="https://unpkg.com/moralis/dist/moralis.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
    <link rel="stylesheet" href="./style.css">
  </head>
<body>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <a class="navbar-brand" href="#">My DEX Aggregator</a>
        <ul class="navbar-nav ml-auto">
          <li class="nav-item">
            <button id="login_button" class="btn btn-outline-primary my-2 my-sm-0" type="submit">Sign in with MetaMask</button>
          </li>
      </nav>
      <div class ="container">
        <div class="row">
            <div class="col col-md-6 offset-md-3" id="window">
                <h4>Swap</h4>
                <div id="form">
                    <div class="swapbox">
                        <div class="swapbox_select token_select" id="from_token_select">
                            SELECT A TOKEN
                        </div>
                        <div class="swapbox_select">
                            <input class="number form-control" placeholder="amount" id="from_amount">
                        </div>
                    </div>
                    <div class="swapbox">
                        <div class="swapbox_select token_select" id="to_token_select">
                            SELECT A TOKEN
                        </div>
                         <div class="swapbox_select">
                            <input class="number form-control" placeholder="amount" id="to_amount">
                        </div>
                    </div>  
                    <div class="gas_estimate_label">Estimated Gas: <span id="gas_estimate"></span></div>
                    <button disabled class="btn btn-large btn-primary btn-block" id="swap_button">Swap</button>                
                </div>
            </div>
        </div>
    </div>
    <!-- Add the new modal body here. Note we added id="token_modal" and updated the modal-title -->
    <div class="modal" id="token_modal" tabindex="-1" role="dialog">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Select a Token</h5>
            <button id="modal_close" type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <p>Modal body text goes here.</p>
          </div>
        </div>
      </div>
    </div>
      <script src="./index.js" type="text/javascript"></script>
    </body>
</html>
```

Ahora, necesitamos una forma de abrir nuestro modal cuando hagamos click en **SELECT A TOKEN**, el elemento html con el ``id='from_token_select' .``

En ``index.js,`` vamos a crear una función llamada ``openModal()`` que abra el modal cuando se haga clic en ese elemento. Añade la siguiente línea:

```
document.getElementById("from_token_select").onclick = openModal;
```
A continuación, crea la función ``openModal():``

```
function  openModal(){
document.getElementById("token_modal").style.display = "block";
}
```
Ahora, al hacer clic en "SELECCIONAR UN TOKEN", aparece este modal:

![](https://lh5.googleusercontent.com/iIRAVop-07sRkhGzHKVNCIfF0UfYFFmIogi5VtZgidkzKJA0F-jSvACY2WVGieg6Hbs95M5UA04I5DG2LF08lp42kyQ-gNOLRip1Ih91Gox9_VKvI-TVaLJwGdZ2DDpTgljFfa5YSaJz0X2bfqG_Gw)

Ahora no hay forma de cerrar el modal, así que vamos a añadirlo. El modal ya tiene un elemento ``modal_close`` en ``index.html``, esta es la X en la esquina superior derecha. Lo conectaremos a una función que cierre el modal.

En ``index.js``, crea una función llamada ``closeModal()`` que cierre el modal cuando se haga clic en ese elemento. Añade la siguiente línea:

```
document.getElementById("modal_close").onclick = closeModal;
```
A continuación, crea la función ``closeModal():``

```
function  closeModal(){
document.getElementById("token_modal").style.display = "none";
}
```
**Código final de la Parte 3**

```
git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-3
```

## Parte 4. Obtener y mostrar la lista de tokens desde la API de CoinGecko

Actualmente tenemos un modal, pero nada dentro. Vamos a recuperar una lista de tokens ERC20 para rellenar el modal, de modo que el usuario pueda seleccionar un token para intercambiar.

Uniswap tiene el proyecto [Token Lists](https://tokenlists.org/), un estándar para crear listas de tokens ERC20 para filtrar los tokens legítimos de alta calidad de las estafas, falsificaciones y duplicados. Lea más sobre la importancia de las listas de tokens [aquí](https://uniswap.org/blog/token-lists).

La lista CoinGecko es una de las más robustas, por lo que utilizaremos esta consulta ``https://tokens.coingecko.com/uniswap/all.json.`` Te recomiendo que la ejecutes directamente en tu navegador para ver qué te devuelve. Deberías ver un objeto JSON que contiene el nombre, logoURI, símbolo y decimales de cada token. Utilizaremos todos estos parámetros cuando construyamos esta aplicación.

![](https://lh3.googleusercontent.com/Tt46WuURUcJtagzPOVhTg3MOSjja4KZGM2v9sVPl0h7xrAx2ZDSEMIYdvHr7OoyNAx-TLQTd6te_S3i0EA3A62gbI-wEydepPN9tVP2CEKhIvZGHSsgtUlLDeRrxwLGcR_5PbhrS32QZxvbpA2ZN5Q)

Ahora, queremos cargar la lista de tokens antes de que el modal sea llamado; de otra manera, el usuario tiene que esperar a que se cargue. Podemos hacerlo cargándola en la función ``init()`` que se inicializa cuando la página se carga por primera vez.

En ``index.js``, crea una función ``init()``. Dentro, añade el siguiente fetch a la API de CoinGecko. Asegúrate también de llamarlo desde ``index.js``, por ejemplo, yo lo he añadido encima de nuestras llamadas onclick en la parte inferior del archivo ``index.js``:

```
async function init(){
    console.log("initializing");
    let response = await fetch('https://tokens.coingecko.com/uniswap/all.json');
    let tokenListJSON = await response.json();
    console.log("listing available tokens: ", tokenListJSON);
}
...

// Add init() call
init();

document.getElementById("login_button").onclick = connect;
document.getElementById("from_token_select").onclick = openModal;
document.getElementById("to_token_select").onclick = openModal;
document.getElementById("modal_close").onclick = closeModal;
```
Ahora, cuando abras tu aplicación y la inspecciones, deberías ver el objeto json de la lista de tokens impreso. Si es la primera vez que utilizas la herramienta Inspect DevTool de Chrome, consulta (este artículo)[https://www.hostinger.com/tutorials/website/how-to-inspect-and-change-style-using-google-chrome] para obtener más información.

![](https://lh5.googleusercontent.com/FODzE9pUIhzo9o3pzg0i1VnnfrFm17VOti3BDSa4D2MJX5_Agi46fsiTg1LjLdFYgSDT4vjriklsHtSsk1UEWeA824LDzmYYDUwNqUR_M0FOCzgjFrbDgeIVQDCybgwYMFwmjlNuGmtthiiVev-kUA)

Ten en cuenta que el objeto JSON de la lista de tokens que se devuelve contiene una clave tokens, que contiene un array de 4954 objetos JSON. Extraeremos la información de aquí para rellenar nuestra lista de tokens.

![](https://lh5.googleusercontent.com/iAmyie2d-4MSDM__R8UbRl-draQhO7JUsLQGbCdmwn05Isxz3S9mdZZA-cTfCGY6Ykk9I7l9i_vNpp_e1XgcmeLiLpJ-qA3VAe4Os5_LUdvNul5hb_Euqvj0r5BJBTdrhbi2fFF5WOynzDSmJPExtg)

Ahora, en lugar de crear nuestra lista de tokens directamente dentro de ``init()``, crearemos una nueva función ``listAvailableTokens()`` para crear una lista de sólo lo que necesitamos - dirección del token, símbolo, imagen, decimales - y llamarla desde ``init()``.

Añade lo siguiente a ``listAvailableTokens()`` dentro de ``index.js``. Asegúrate de leer los comentarios:

```
async function listAvailableTokens(){
  console.log("initializing");
  let response = await fetch('https://tokens.coingecko.com/uniswap/all.json');
  let tokenListJSON = await response.json();
  console.log("listing available tokens: ", tokenListJSON);
  tokens = tokenListJSON.tokens
  console.log("tokens:", tokens);

  // Create a token list for the modal
  let parent = document.getElementById("token_list");
  // Loop through all the tokens inside the token list JSON object
  for (const i in tokens){
    // Create a row for each token in the list
    let div = document.createElement("div");
    div.className = "token_row";
    // For each row, display the token image and symbol
    let html = `
    <img class="token_list_img" src="${tokens[i].logoURI}">
      <span class="token_list_text">${tokens[i].symbol}</span>
      `;
    div.innerHTML = html;
    parent.appendChild(div);
  }
}

```

Además, sustituyamos el texto ficticio del cuerpo del modal por el elemento ``"token_list"`` que acabamos de crear:

```
 <div class="modal" id="token_modal" tabindex="-1" role="dialog">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Select a Token</h5>
            <button id="modal_close" type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
              < !-- Replace the modal text with token_list -->
            <div id="token_list"></div>
          </div>
        </div>
      </div>
    </div>

```

Por último, añade este estilo para el modal a style.css para que la lista de tokens no fluya indefinidamente y haya una barra de desplazamiento:

```
.modal-body{
height: 500px;
overflow: scroll;
}
```
Ahora tú aplicación debe tener este aspecto cuando un usuario hace clic en "SELECCIONAR UN TOKEN" y aparece el modal:

![](https://lh3.googleusercontent.com/naQCbQ61sSC5rjDgf6ueMLaK5N0adZYrj5h9jY9xBFRq63Vpxz_K30q3cfNRv3YB9o2UCq9vRSpq-ZKXILFPmUcrMXBiIcADXOx-3zKOUkp52jf13XEfKtf01HcFVWK-gHa4oQq5hNRv--PMui-JCQ)



>**📘 Nota**
>
>No encontrarás ETH en esta lista porque en realidad no es un token ERC20. Necesita ser envuelto, WETH. [Más información sobre WETH](https://weth.io/). Algunas aplicaciones tienen una lista curada en lugar de mostrar todas las opciones.

**Código final de la Parte 4

```
git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-4
```

## Parte 5. Mostrar la imagen y el símbolo de la ficha seleccionada en la caja de intercambio

Ahora que tenemos una lista de tokens para que el usuario seleccione, queremos mostrar la imagen y el símbolo del token seleccionado en la caja de intercambio.

Primero, crea una función ``selectToken()``. Llámala desde la parte inferior de ``listAvailableTokens()`` utilizando una función anónima cuando se haga clic en una de las filas del div que indica un token:

```
async function listAvailableTokens(){
  // ... the rest of the function
    div.innerHTML = html;
    // selectToken() will be called when a token is clicked
    div.onclick = () => {
      selectToken(tokens[i]);
    };
    parent.appendChild(div);
  }
}

```

Antes de crear la función ``selectToken()``, necesitamos una forma de rastrear en qué lado del selector de token estamos. ¿Estamos en el lado de origen o en el lado de destino?

Para rastrear esto, crea un objeto global ``currentTrade`` y una variable ``currentSelectSide`` en la parte superior del código en ``index.js``:

```
// top of index.js
let  currentTrade = {};
let  currentSelectSide;
```

Ahora que tenemos este objeto global, debemos rastrear si el usuario está en el lado from o en el lado to dependiendo de qué selector de token se haga clic. Configura esta función anónima que pasa ``"from"`` cuando se selecciona ``"from_token_select"`` en la parte inferior de ``index.js``:

```
document.getElementById("from_token_select").onclick = () => {
openModal("from");
};
```

Y actualiza ``openModal()`` con el siguiente código:

```
// index.js
function  openModal(side){
    // Store whether the user has selected a token on the from or to side
    currentSelectSide = side;
    document.getElementById("token_modal").style.display = "block";
}
```

Ahora, crea una función ``selectToken()`` y añade el siguiente código:

```
function  selectToken(token) {
    // When a token is selected, automatically close the modal
    closeModal();
    // Track which side of the trade we are on - from/to
    currentTrade[currentSelectSide] = token;
    // Log the selected token
    console.log("currentTrade:" , currentTrade);
}
```

Ejecuta el programa y verifica que ``currentTrade`` se registra correctamente.

![](https://lh4.googleusercontent.com/0ia1FbhR5OXPoLR7P_zMkV3NdcdGJuKOTkmxK--ioH3nTpQrnfC-lqgysQqLXIi8UNkWlQsX6tr-HYWn8ji_SpIyBDVNW7K_RUAHgov5spdhnvLDyzN-xlEDNhUjN6YFPiPnEjqvmaLING0dTIilRw)


** Ahora para mostrar la imagen y símbolos de token.

Primero, añade "``from_token_img``" , "``from_token_text``" , "``to_token_img``" , y "``to_token_text``" en el ``index.html``: Crea una ``renderInterface()`` debajo de ``selectToken()`` y llamala desde ``selectToken()``.

```
< !-- Replace the SELECT A TOKEN text -->
< !-- From token -->
<div  class="swapbox_select token_select"  id="from_token_select">
    <img  class="token_img"  id="from_token_img">
    <span  id="from_token_text"></span>
</div>

< !-- To token -->
<div  class="swapbox_select token_select"  id="to_token_select">
    <img  class="token_img"  id="to_token_img">
    <span  id="to_token_text"></span>
</div>
```

Dentro de ``renderInterface()``, estableceremos las imágenes from/to token y el texto del símbolo llamando a los elementos asociados en ``index.html``. Recuerda que tanto ``logoURI`` como ``symbol`` son devueltos por CoinGeckoAPI:

```
function selectToken(token) {
  closeModal();
  currentTrade[currentSelectSide] = token;
  console.log("currentTrade:" , currentTrade);
  renderInterface();
}

// Function to display the image and token symbols 
function renderInterface(){
  if (currentTrade.from) {
    console.log(currentTrade.from)
    // Set the from token image
    document.getElementById("from_token_img").src = currentTrade.from.logoURI;
     // Set the from token symbol text
    document.getElementById("from_token_text").innerHTML = currentTrade.from.symbol;
  }
  if (currentTrade.to) {
      // Set the to token image
    document.getElementById("to_token_img").src = currentTrade.to.logoURI;
      // Set the to token symbol text
    document.getElementById("to_token_text").innerHTML = currentTrade.to.symbol;
  }
}
```

Por último, añade ``renderInterface()`` al final de la función ``selectToken()``:

```
function  selectToken(token) {
    closeModal();
    currentTrade[currentSelectSide] = token;
    console.log("currentTrade:" , currentTrade);
    // Display token image and symbol in swapbox
}
```

Ahora los usuarios pueden seleccionar entre tokens, así como introducir la cantidad que desean intercambiar en nuestra dapp.

![](https://lh6.googleusercontent.com/BdHwcftnwsqpQ4RnDWFg2VXHWrpv79zyqOVuxeM_gzIlWShO0_aFZ-IqxPFdgSEEiVMELK5VmANc9OZtrFIr6MXKEucrfXB2MEi2yG3W0H0rJNzEVXyYrthWjjMuSRuWYSF20Kl_X4fDmFmAvzaZ-g)

**Código final de la Parte 5**

```
git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-5
```

## Parte 6. Obtener precio

Ahora que los usuarios pueden seleccionar sus tokens y establecer la cantidad de tokens a intercambiar, ¡ayudémosles a encontrar el mejor precio!

Una funcionalidad que es común en las dapps de comercio DeFi y que es realmente intuitiva es la generación automática de un precio cuando se introduce la cantidad y el cursor del usuario deja el foco en la casilla de cantidad. Compruébalo aquí en Matcha.xyz.

El precio DAI se genera automáticamente una vez introducido el precio WETH: Para ello, podemos usar un evento Javascript llamado ``onblur``. Úsalo aquí cuando el usuario deje el foco del elemento "``from_amount``":

![](https://lh4.googleusercontent.com/rx2e_pc_dBBPCQyRV2F6rsN6O9Af_St6r-LxMv47ulpMRYsf8uuw5110weNY0nSmlY8nJQzo387slSM5KSbkUssN_1ZkwAqlLFj0xJ9ka_PTdaOx3cgjC1Fp1od8RDwL_irxVW_G4aa05uRPvr41Ng)

```
// at the bottom of index.js add this
document.getElementById("from_amount").onblur = getPrice;
```
### Endpoints /price vs /quote

Ahora necesitamos crear una función llamada ``getPrice()``. En su interior, llamaremos al punto final [GET /swap/v1/price.](https://docs.0x.org/0x-api-swap/api-references/get-swap-v1-price)

``/price`` es casi idéntico a ``/quote``, pero con algunas diferencias clave. ``/price`` no devuelve una transacción que pueda enviarse a la cadena; simplemente nos proporciona la misma información. Piense en él como la versión de "``read-only``" de ``/quote``".

Esto también es importante porque ``/quote`` **devuelve una orden en la que un Creador de Mercado ha comprometido sus activos.** Así que si hacemos demasiado ping a ``/quote`` cuando en realidad sólo estamos preguntando por un precio y no estamos listos para enviar una orden, **¡el sistema se puede atascar!**

Este es un ejemplo de una petición GET HTTP ``/price``. La llamada es casi idéntica a ``/quote``: ``https://api.0x.org/swap/v1/price?sellToken=ETH&buyToken=DAI&sellAmount=1000000000000000000``


>📘 Este endpoint de la API utiliza Ethereum (mainnet), como se indica en ``https://api.0x.org``
>[Aquí hay una lista de todos los puntos finales de la API para las redes soportadas.](https://docs.0x.org/0x-api-swap/api-references#introduction)

Para usar esto en nuestro ``index.js``, necesitamos importar el [módulo qs](https://www.npmjs.com/package/qs). Añade la declaración ``require`` en la parte superior del archivo:

```
const  qs = require('qs');
```

### Browserify para usar módulos Node en el navegador

Ahora, debido a que hemos instalado un módulo usando ``require``, nuestro navegador nos dará un golpe si intentamos cargar nuestro ``index.html`` ahora. Para arreglar esto, instala Browserify ejecutando esto en la terminal:

```
npm install -g browserify`
```

Instala el [módulo qs](https://www.npmjs.com/package/qs) con [npm](https://npmjs.org/):

```
npm i qs
```

Ahora agrupa recursivamente todos los módulos requeridos empezando por ``main.js`` en un único archivo llamado ``bundle.js`` con el comando browserify (Nota: Puede que necesites indicar explícitamente las rutas para ``./index.js`` y ``./bundle.js`` según sea específico para tu configuración).

```
browserify index.js --standalone bundle -o bundle.js
```

>❗️ Nota
> En adelante, asegúrate de volver a ejecutar el comando Browerify para generar un ``bundle.js`` actualizado cada vez que necesites utilizar el ``index.html``

Y asegúrate de actualizar el src del script en ``index.html`` de ``src=./index.js`` a ``src=./bundles.js``:

```
< !-- Make sure your script now sources from the correct file --> 
<script  src="./bundle.js"  type="text/javascript"></script>
```

## Crear la función getPrice()

Dentro de ``index.js``, crea ``getPrice()``. Desglosaré las diferentes secciones de esta función y luego mostraré un fragmento de código completo de la función al final.

Añadimos una declaración ``if`` porque sólo queremos ejecutar la consulta ``/price`` si se han seleccionado los tokens "from" y "to" y se ha rellenado la cantidad del token "from". También obtenemos el importe introducido por el usuario y lo multiplicamos por 10 a la potencia del número de decimales del código de origen. Por ejemplo, si el usuario ha introducido que quiere intercambiar 1 WETH, WETH tiene 18 decimales. La unidad más pequeña de WETH es wei. Por tanto, la cantidad que quiere intercambiar es (1 x 10 a la potencia de 18) wei. Puedes volver a comprobar los decimales mirando en EtherScan en Resumen del perfil [aquí.](https://etherscan.io/token/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2)

```
async  function  getPrice(){
    console.log("Getting Price");
    // Only fetch price if from token, to token, and from token amount have been filled in 
    if (!currentTrade.from || !currentTrade.to || !document.getElementById("from_amount").value) return;
    // The amount is calculated from the smallest base unit of the token. We get this by multiplying the (from amount) x (10 to the power of the number of decimal places)
    let  amount = Number(document.getElementById("from_amount").value * 10 ** currentTrade.from.decimals);
    ...
}
```

A continuación, dentro de ``getPrice()`` estableceremos nuestros parámetros y los introduciremos en nuestro presupuesto:

```
async  function  getPrice(){
...
const params = {
    sellToken: currentTrade.from.address,
    buyToken: currentTrade.to.address,
    sellAmount: amount,
  }
  // Fetch the swap price.
  const response = await fetch(
    `https://api.0x.org/swap/v1/price?${qs.stringify(params)}`
    );
...
}

```
Por último, una vez devuelta la respuesta, analizaremos el JSON. El objeto JSON contiene los pares clave/valor para ``buyAmount`` y ``estimatedGas`` que podemos rellenar directamente en los elementos html "``to_amount``" y "``gas_estimate``" en la interfaz de usuario:

```
async  function  getPrice(){
...
    // Await and parse the JSON response 
    swapPriceJSON = await  response.json();
    console.log("Price: ", swapPriceJSON);
    // Use the returned values to populate the buy Amount and the estimated gas in the UI
    document.getElementById("to_amount").value = swapPriceJSON.buyAmount / (10 ** currentTrade.to.decimals);
    document.getElementById("gas_estimate").innerHTML = swapPriceJSON.estimatedGas;
...
}
```

Ahora, para ejecutarlo, asegúrate de volver a ejecutar el comando Browerify para generar un ``bundle.js ``actualizado.

Ahora tu proyecto debería rellenar automáticamente los campos to-amount y Estimated Gas.

![](https://lh5.googleusercontent.com/y1eutHxYhqjEpLKDuNkbg7ii0aCUTDxsL25wS4NpQSOIRGP6-O1mnYdB0CHSn_qy9v4b27MJyMnzy2bEFXcKUG_NiVPwKzQ3I9zqgv2eaMN6PxVUDFHWhvtsaBGal8S13xQ0PzehOBZSHK0boqA1kw)

**Código final de la Parte 6**

```
git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-6
```


## Parte 7. Obtener cotización

Ahora, ¡a cotizar!

Aquí está el plan de juego para lo que queda:

+ ✅ Habilitar el botón "Swap" sólo cuando MetaMask está conectado
+ ⚪ Utilizar la dirección de la cuenta MetaMask de los usuarios para obtener una cotización
+ ⚪ Establecer una asignación de tokens
+ ⚪ Realizar el intercambio

Hemos completado el primer paso de la parte anterior (2. Conectarse a MetaMask). Vamos a crear una función para obtener la cotización.

**getQuote() - Utilizar la dirección de la cuenta MetaMask del usuario para obtener una cotización**

Esta función será muy similar a ``getPrice()``, la única diferencia es que vamos a pasar en un ``takerAddress``, que es la dirección que llenará la cotización. En nuestro caso, esta es nuestra cuenta MetaMask. Puedes leer más sobre el parámetro takerAddress [aquí](https://docs.0x.org/0x-api-swap/api-references/get-swap-v1-quote#request).

Voy a copiar y pegar el código ``getPrice()`` y haré las siguientes modificaciones en ``getQuote()`` para pasar y establecer el ``takerAddress``:


```
// index.js

// Function to get a quote using /swap/v1/quote. We will pass in the user's MetaMask account to use as the takerAddress
async function getQuote(account){
  console.log("Getting Quote");

  if (!currentTrade.from || !currentTrade.to || !document.getElementById("from_amount").value) return;
  let amount = Number(document.getElementById("from_amount").value * 10 ** currentTrade.from.decimals);

  const params = {
    sellToken: currentTrade.from.address,
    buyToken: currentTrade.to.address,
    sellAmount: amount,
    // Set takerAddress to account 
    takerAddress: account,
  }

  // Fetch the swap quote.
  const response = await fetch(
    `https://api.0x.org/swap/v1/quote?${qs.stringify(params)}`
    );
  
  swapQuoteJSON = await response.json();
  console.log("Quote: ", swapQuoteJSON);
  
  document.getElementById("to_amount").value = swapQuoteJSON.buyAmount / (10 ** currentTrade.to.decimals);
  document.getElementById("gas_estimate").innerHTML = swapQuoteJSON.estimatedGas;

  return swapQuoteJSON;
}
```

¡Paso 2 hecho!

+ ✅ Habilitar el botón "Swap" sólo cuando MetaMask está conectado
+ ✅ Utilice la dirección de la cuenta MetaMask de los usuarios para obtener una cotización
+ ⚪ Establecer una asignación de token
+ ⚪ Realizar el intercambio

Ahora vamos a echar un vistazo a la configuración de un token allowance

**Código final de la parte 7**

```
git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-7
```

## Parte 8. Establecer una Asignación de tokens

Se requiere una asignación de tokens si desea que un tercero mueva fondos en su nombre. En pocas palabras, les estás permitiendo mover tus tokens.

En nuestro caso, nos gustaría que el contrato inteligente 0x Exchange Proxy negociara nuestros tokens ERC20 por nosotros, por lo que necesitaremos aprobar una asignación (una cierta cantidad) para que este contrato mueva una cierta cantidad de nuestros tokens ERC20 en nuestro nombre.

Lo que se necesita para hacer esto:

+ (i) Conectar con el método ``approve()`` del token ERC20 usando un objeto web3
+ (ii) Establecer la cantidad de aprobación a ``maxApproval``
+ (iii) Usar ``approve()`` para dar a nuestro ``allowanceTarget`` una aprobación por un importe máximo

**(i) Conectar con el método ``approve()`` del token ERC20 utilizando un objeto web3**

Todos los tokens ERC20 deben implementar la función [approve(address spender, uint256 amount)](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#IERC20-approve-address-uint256-). Esta función establece ``amount`` como la asignación de ``spender`` sobre los tokens de la persona que llama (es decir, cuántos de estos tokens ERC20 puede mover el tercero en nombre de la persona que llama).

Devuelve un valor booleano que indica si la operación ha tenido éxito.

Lea más sobre cómo establecer la cantidad de tokens permitidos [aquí](https://docs.0x.org/0x-api-swap/advanced-topics/how-to-set-your-token-allowances).

Como se ve en la función [approve(address spender, uint256 amount)](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#IERC20-approve-address-uint256-) anterior, para llamar a la asignación de tokens, necesitamos un par de cosas:

+ **spender address**- Esta será la dirección 0x Exchange Proxy Smart Contract. Podemos obtener esto de la respuesta JSON como el parámetro ``allowanceTarget`` devuelto en nuestra consulta ``/quote``
+ **amount**- ¿Cuántos permisos queremos dar al contrato? En este caso, voy a demostrar con la cantidad máxima (``maxApproval``) posible, sin embargo, recomiendo establecer sólo lo que necesita si es posible. Echa un vistazo a este [ejemplo aquí](https://docs.0x.org/0x-api-swap/guides/swap-tokens-with-0x-api#examples) para saber cómo implementar eso (pista: reto semanal)
+ **caller** - La cuenta MetaMask del usuario será la dirección del caller (aka ``takerAddress``)

**Build trySwap()**

Vamos a empezar a construir ``trySwap()`` y voy a explicar cada pieza en el camino. Publicaré el código completo al final

En primer lugar, vamos a obtener el ``takerAddress`` y pasarlo a ``getQuote(address)`` para que podamos obtener ``swapQuoteJSON``  de nuevo para utilizarlo desde la solicitud ``/quote``:

```
// index.js
async  function  trySwap(){
  // The address, if any, of the most recently used account that the caller is permitted to access
  let accounts = await ethereum.request({ method: "eth_accounts" });
  let takerAddress = accounts[0];
  // Log the the most recently used address in our MetaMask wallet
  console.log("takerAddress: ", takerAddress);
    // Pass this as the account param into getQuote() we built out earlier. This will return a JSON object trade order. 
const  swapQuoteJSON = await  getQuote(takerAddress);
}
```
Ahora vamos a llamar al [método approve()](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#IERC20-approve-address-uint256-) del token ERC20. Como necesitaremos interactuar con los métodos del contrato ERC20, necesitamos crear un objeto web3.

Para poder interactuar con los métodos de un contrato ERC20, necesitamos crear un objeto web3, específicamente [web3.eth.Contract](https://web3js.readthedocs.io/en/v1.2.11/web3-eth-contract.html#web3-eth-contract):

``const ERC20TokenContract = new web3.eth.Contract(erc20abi, fromTokenAddress)``;

Este objeto necesita un ``erc20abi`` que es la representación json de un contrato erc20. El ``erc20abi`` es nuestro plano para interactuar con cualquier token que siga el estándar ERC20. Se representa en formato JSON. Una búsqueda rápida debería sacar un ``erc20abi.json`` ya que es un estándar. Yo estoy usando [este.](https://gist.github.com/veox/8800debbf56e24718f9f483e1e40c35c)

El objeto también necesita la dirección específica del token con el que estamos interesados en interactuar, en este caso, es el ``fromTokenAddrss`` porque queremos que el tercero (es decir, el 0x Smart Contract) actúe sobre los tokens con los que queremos comerciar.

Añade lo siguiente en ``trySwap()``:


```
// index.js
async  function  trySwap(){
...
// Setup the erc20abi in json format so we can interact with the approve method below
const erc20abi= [{ "inputs": [ { "internalType": "string", "name": "name", "type": "string" }, { "internalType": "string", "name": "symbol", "type": "string" }, { "internalType": "uint256", "name": "max_supply", "type": "uint256" } ], "stateMutability": "nonpayable", "type": "constructor" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "owner", "type": "address" }, { "indexed": true, "internalType": "address", "name": "spender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Approval", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "from", "type": "address" }, { "indexed": true, "internalType": "address", "name": "to", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "Transfer", "type": "event" }, { "inputs": [ { "internalType": "address", "name": "owner", "type": "address" }, { "internalType": "address", "name": "spender", "type": "address" } ], "name": "allowance", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "approve", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "account", "type": "address" } ], "name": "balanceOf", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "burn", "outputs": [], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "account", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "burnFrom", "outputs": [], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [], "name": "decimals", "outputs": [ { "internalType": "uint8", "name": "", "type": "uint8" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "subtractedValue", "type": "uint256" } ], "name": "decreaseAllowance", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "spender", "type": "address" }, { "internalType": "uint256", "name": "addedValue", "type": "uint256" } ], "name": "increaseAllowance", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [], "name": "name", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "symbol", "outputs": [ { "internalType": "string", "name": "", "type": "string" } ], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "totalSupply", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "recipient", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "transfer", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "internalType": "address", "name": "sender", "type": "address" }, { "internalType": "address", "name": "recipient", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "transferFrom", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }]
    // Set up approval amount for the token we want to trade from
    const fromTokenAddress = currentTrade.from.address;
    
    // In order for us to interact with a ERC20 contract's method's, need to create a web3 object. This web3.eth.Contract object needs a erc20abi which we can get from any erc20 abi as well as the specific token address we are interested in interacting with, in this case, it's the fromTokenAddrss
// Read More: https://web3js.readthedocs.io/en/v1.2.11/web3-eth-contract.html#web3-eth-contract
    const  web3 = new  Web3(Web3.givenProvider);
    const ERC20TokenContract = new web3.eth.Contract(erc20abi, fromTokenAddress);
    console.log("setup ERC20TokenContract: ", ERC20TokenContract);
...
}

```

**(ii) Establecer el importe de aprobación en maxApproval**

En este ejemplo, mostraré cómo establecer la cantidad máxima de aprobación. Te recomiendo que le eches un vistazo a [este ejemplo](https://docs.0x.org/0x-api-swap/guides/swap-tokens-with-0x-api#sell-100-dai-for-eth) para ver cómo establecer sólo lo necesario.

Usaremos la librería [BigNumber](https://github.com/MikeMcl/bignumber.js/) para crear un número realmente grande. Luego lo estableceremos en una variable ``maxApproval``. Introduce esto al final de ``trySwap()`` en ``index.js``

```
    // The max approval is set here. Using Bignumber to handle large numbers and account for overflow (https://github.com/MikeMcl/bignumber.js/)
    const maxApproval = new BigNumber(2).pow(256).minus(1);
    console.log("approval amount: ", maxApproval);
```

**(iii) Usar approve() para conceder a nuestro allowanceTarget una asignación por un importe máximo**

Ahora que podemos interactuar con el método ``approve()`` y la cantidad de aprobación, vamos a utilizarlo para conceder al ``allowanceTarget`` (es decir, la dirección del contrato inteligente 0x Exchange Proxy), una asignación para gastar nuestros tokens ERC20. Podemos obtener la dirección ``allowanceTarget`` directamente del objeto JSON devuelto por nuestra respuesta ``/quote``:

```
// index.js
async  function  trySwap(){
...
 // Grant the allowance target (the 0x Exchange Proxy) an  allowance to spend our tokens. Note that this is a txn that incurs fees. 
  const tx = await ERC20TokenContract.methods.approve(
    swapQuoteJSON.allowanceTarget,
    maxApproval,
  )
  .send({ from: takerAddress })
  .then(tx => {
    console.log("tx: ", tx)
  });
...
}
```

**¡Pruébalo ahora!**

+ Selecciona un token de origen (asegúrate de que tu billetera tiene suficientes tokens de ese tipo; de lo contrario obtendrá un error)
+ Selecciona un token
+ Introduce una cantidad de origen (asegúrate de que tu billetera tiene al menos esa cantidad, de lo contrario la cotización no se llevará a cabo)
+ Conecta tu billetera MetaMask, el botón "Swap" debería estar habilitado
+ Si haces clic en "Swap" debería aparecer una ventana emergente de MetaMask preguntando si apruebas el allowanceTarget, 0x Exchange Proxy contract address: ``0xdef1c0ded9bec7f1a1670819833240f027b25eff`` !

>**📘Nota**
>Esta transacción requiere el pago de tasas. Lee antes de firmar.

![](https://lh4.googleusercontent.com/TaDjZxZW-07aaGDEoYV4mvCOO4hWKV0K_2ofJhS_f6BcXVt1Bnkgf8NLyrQtpXr2Pd3ftRXH_ENtZNmtiDCpNMIX_x8TPyVx6AZC41rSZjwQ9qWprTmUtpPJhZ5a-VKyfLjJ5BFklXsSupzfisL0TA)

¡Casi listo!

+ ✅ Habilitar el botón "Intercambiar" sólo cuando MetaMask está conectado.
+ ✅ Utilizar la dirección de la cuenta MetaMask de los usuarios para obtener una cotización.
+ ✅ Establecer una asignación de token.
+ ⚪ Realizar el canje

**Código final de la parte 8**

```
git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-8
```
## Parte 9. Realizar el intercambio

Ahora la última parte, realizar el intercambio. Esta parte debería ser muy sencilla ya que lo que se devuelve desde /quote es un objeto JSON que está listo para ser firmado y enviado como una transacción válida en la cadena de bloques.

Añade esto al final de trySwap():

```
// index.js
async  function  trySwap(){
...
    // Perform the swap
    const  receipt = await  web3.eth.sendTransaction(swapQuoteJSON);
    console.log("receipt: ", receipt);
}
```

La razón por la que podemos pasar directamente la [respuesta /quote](https://docs.0x.org/0x-api-swap/api-references/get-swap-v1-quote#response) es porque contiene todos los parámetros necesarios para [web3.eth.setTransaction()](https://web3js.readthedocs.io/en/v1.2.11/web3-eth.html#id84) - from, to, value, gas, data, etc.

Ahora, si seleccionas un par de tokens (con un token from que poseas), introduces una cantidad from, y conectas tu cartera MetaMask, y pulsas "Swap", ¡deberías obtener esta ventana emergente preguntando si te gustaría hacer el intercambio! Y con eso, ¡tu aplicación está completa!


![](https://lh6.googleusercontent.com/Du31d4M_-0ApVIl-Bm3GtE4XqItaNeqQPoToifQPc3z19xNO8bZ5ELvy2u3bnXRLh0HHXzEj8qfFxnLemqZprMf36s-ovhF-EHw5ozN8YmqR8K7_xTVsGOEehTIoOg02puoIn2YoNIKi42e49Yqhyw)


+ ✅ Habilitar el botón "Swap" sólo cuando MetaMask está conectado
+ ✅ Utiliza la dirección de la cuenta MetaMask de los usuarios para obtener una cotización
+ ✅ Establecer una asignación de token
+ 🥳 Realizar el intercambio

Si sigues todo el proceso, podrás aprobar la asignación de tokens, realizar el intercambio y recibir los tokens intercambiados en tu cartera.

También puedes [comprobar tu transacción en Etherscan.](https://etherscan.io/tx/0x55501496927394ec5072ce287668e2ad4b6db933391e96c561575ea2256a8aaa)

![](https://lh3.googleusercontent.com/MrgiE433LUNATG0Egc6DUh4CwugGK-gO5I4AHo_LNprVlT7jSwYjx-405-G1WVYt0poiPzQFI7_IXPy2d07wWghPh9IcVF78bd0n5_qVF1FmdnYfD9zUaWB33UkhHigxHfDdagVDqPHb6rUZDi70gg)

**Código definitivo de la Parte 9**

```
git clone https://github.com/0xProject/swap-demo-tutorial/tree/main/swap-demo-tutorial-part-9
```

### Más información
+ [Introducción a 0x](https://docs.0x.org/introduction/introduction-to-0x)
+ [Más información sobre la API 0x /swap](https://docs.0x.org/0x-api-swap/introduction)
+ [Referencias de la API 0x](https://docs.0x.org/0x-api-swap/api-references#introduction)
+ [Guías y ejemplos de código para construir con 0x](https://docs.0x.org/introduction/guides)

### Desafíos

¡Ahora a llevar tu dapp al siguiente nivel! Aquí tienes algunos retos que puedes intentar por tu cuenta para poner a prueba tus conocimientos. [(Algunos consejos ofrecidos en el vídeo de YouTube si necesitas ayuda)](https://www.youtube.com/watch?v=tVvZ1ivp4X0&t=1s)

+ Mostrar el desglose porcentual de donde se obtuvo un intercambio utilizando el parámetro [sources response param](https://docs.0x.org/0x-api-swap/api-references/get-swap-v1-quote#response-1)(por ejemplo: el mejor precio proviene de 50% Uniswap, 50% Kyber)
+ Actualmente [la cantidad de tokens permitidos es la máxima](https://docs.alchemy.com/alchemy/road-to-web3/weekly-learning-challenges/9.-how-to-build-a-token-swap-dapp-with-0x-api#ii-set-the-approval-amount-to-maxapproval). Cambia esto para que sea más seguro y el usuario sólo apruebe la cantidad necesaria.
+ Calcular el precio cuando un usuario introduce un nuevo token "a" (ahora mismo sólo se calcula automáticamente cuando un usuario introduce un nuevo token "desde").
+ Mostrar el gas estimado en $
+ Filtrar la larga lista de tokens
+ Permite a los usuarios cambiar de cadena y recibir un presupuesto adecuado (¡recuerda que la lista de tokens también cambiará!)

Para recibir tu Prueba de Conocimiento NFT de esta semana, comparte tu experiencia trabajando en los desafíos en:  [the Alchemy University Discord](https://university.alchemy.com/discord)

Cuando termines tu desafío, tuitea sobre él etiquetando a [@AlchemyLearn](https://twitter.com/AlchemyLearn), [@0xProject](https://twitter.com/0xProject) y al autor [@hey_its_jlin](https://twitter.com/hey_its_jlin) en Twitter.
