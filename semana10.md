# Cómo crear un Twitter descentralizado con el protocolo Lens

Últimamente ha habido muchas discusiones sobre cuánto control tienen los gigantes de las redes sociales como Facebook y Twitter. Independientemente de cuál sea tu opinión al respecto, me gustaría compartir contigo un experimento que es súper emocionante.

Este año, la gente del equipo de Aave ha lanzado un proyecto llamado **Lens Protocol**. Es una de las tecnologías más emocionantes que han entrado en el ecosistema web3 últimamente porque utiliza la tecnología blockchain para devolver el poder y el control de los datos a los usuarios que los generan.

Así que pensé, ¿por qué no la exploramos todos juntos? Sería una lección final apropiada para combinar todo lo que has aprendido hasta ahora 🙂 .

En esta lección, aprenderás:

+ Cómo configurar una app Next.js con un cliente Apollo GraphQL.
+ Cómo usar la API del protocolo Lens para obtener perfiles, publicaciones y otros datos almacenados en la blockchain de Polygon
+ Una introducción a la API MintKudos -- ¡para que puedas integrar tus tokens PoK en tu dapp!
+ Una introducción al Protocolo Lit -- en caso de que quieras encriptar ciertos posts para que sólo se muestren a varios miembros de la comunidad
+ Cómo desplegar tu aplicación de redes sociales descentralizada en un sitio web usando Repl.it
+ Múltiples opciones para ampliar este proyecto

¡GOOOO!

![](https://files.readme.io/bef84b9-rtw3-yt-banner.png)

## Paso 0. Recorrido por el ecosistema Lens Protocol

![](https://files.readme.io/8aa056c-lens_protocol.png)
https://lens.xyz/

Empecemos por echar un vistazo a lo que ofrece Lens Protocol.

Si visitas el sitio [web de Lens](https://lens.xyz/), verás inmediatamente dos enlaces.

+ **Developer Garden**: te lleva a la documentación y las guías.
+ **Únete a Discord**: te lleva al servidor de chat de la comunidad.
  
Visitaremos el Jardín del Desarrollador en un segundo, pero antes de eso, echemos un vistazo a las opciones de la barra de menú en la parte superior derecha.

![](https://files.readme.io/82d2ec7-top_right.png)

Puedes ver las aplicaciones que ya han sido creadas por la comunidad.

Aquí, voy a llamar tú atención sobre dos cosas:

1. **Claim Handle** - A partir de ahora (7/13/2022), Lens Protocol está todavía en beta abierta, por lo que tendrás que estar en la lista de permitidos para poder reclamar un handle para la dirección de tu monedero. Por suerte, si has seguido el programa de la comunidad Road To Web3 y has obtenido NFT de prueba de conocimiento en el pasado, es probable que ya estés incluido en la lista de autorizados. Si no es así, entra en el [Discord de Alchemy](http://alchemy.com/discord) y pregunta sobre el desafio #week-10, Una vez que tengas tu handle, (visita la comunidad Road To Web3 en Lenster)(https://lenster.xyz/communities/0x25c4-0x0c) 🔥.

2. **Aplicaciones** - Este enlace te lleva a una lista de aplicaciones web creadas por la comunidad, incluyendo algunas como Lensfrens, Lenster, Phaver, Alps Finance, Refract, y más.

Quiero dedicar un momento a ilustrar el poder del Protocolo Lens.

Si visitas esta página de perfil en lensfrens - https://www.lensfrens.xyz/thatguyintech.lens - verás mi cuenta, que debería tener este aspecto:

![](https://files.readme.io/818e04f-lensfrens.png)
lensfrens

Ahora, si visitas esta aplicación web completamente diferente Lenster - https://lenster.xyz/u/thatguyintech.lens - me verás de nuevo, con exactamente los mismos datos de perfil, excepto una experiencia completamente diferente, incluso incluyendo posts/comentarios/reacciones de otros perfiles.

![](https://files.readme.io/8cefd23-lenz.png)
Lenster0

Nada super alucinante hasta ahora, pero quédate conmigo aquí.

Echa un vistazo a las otras aplicaciones:

+ **Phaver** - Aplicación móvil social compatible con Lens que TAMBIÉN te permite hacer "stake" para curar las publicaciones de otras personas, permitiendo así que la gente gane dinero por su contenido.

![](https://files.readme.io/0449f13-Phaver.png)
Phaver

+**Refract**- Es como Hacker News, excepto que todos los enlaces y mensajes que se comparten aquí son impulsados por el Protocolo Lens.

![](https://files.readme.io/30229d2-Refract.png)
Refract


Todas estas aplicaciones han sido creadas por personas y equipos diferentes, con experiencias de usuario y objetivos de producto distintos, pero todos los datos subyacentes son los mismos, como si todas compartieran la misma base de datos y las mismas API.

**¿Cómo es posible?**

Resulta que una base de datos compartida y una API pública son exactamente las ideas fundamentales del Protocolo Lens. Por eso esta tecnología tiene tanto potencial.

Cada dato es un NFT.

Cada post, cada comentario, cada reacción, cada FOLLOW. Cada uno de estos datos se almacena como un token no fungible creado y controlado por ti, el creador.

Esto significa que el contenido digital y las relaciones que creamos como usuarios nos pertenecen y pueden llevarse a cualquier aplicación construida sobre el protocolo.

Ahora, ¡manos a la obra!

## Paso 1. Configura una aplicación Next.js e instala Apollo
  
Abre una línea de comandos. Utiliza ``create-next-app`` para iniciar un proyecto que se llamará ``road-to-lens``

```
npx create-next-app road-to-lens
```
  
![](https://files.readme.io/62d675d-rtl.png)

El repositorio generado se llamará road-to-lens, y deberías tener la siguiente estructura de directorios:

```
thatguyintech@albert road-to-lens % tree -L 1
.
├── README.md
├── next.config.js
├── node_modules
├── package.json
├── pages
├── public
├── styles
└── yarn.lock
  
```
  
Vamos a instalar también nuestro cliente graphql mientras estamos aquí. Utilizaremos [Apollo](https://www.apollographql.com/docs/react/data/queries/) para consultar los datos de Lens Protocol.

```
npm install @apollo/client graphql
```
  
Después de esto, podemos realizar una comprobación iniciando un servidor local y cargando la página web:

```
thatguyintech@albert road-to-lens % npm run dev

> road-to-lens@0.1.0 dev
> next dev

ready - started server on 0.0.0.0:3000, url: http://localhost:3000
info  - SWC minify release candidate enabled. https://nextjs.link/swcmin
event - compiled client and server successfully in 4.2s (169 modules)

```
  
Al cargar http://localhost:3000 debería aparecer una página de plantilla básica con el siguiente aspecto:

  
![](https://files.readme.io/0db0905-next.js.png)

## Segundo paso. Prueba Apollo GraphQL en la página index.js con perfiles recomendados de Lens

Vamos a familiarizarnos con Apollo y GraphQL cargando perfiles recomendados de Lens en la página de inicio.

En primer lugar, configuremos el proveedor Apollo para que envuelva toda nuestra aplicación, de forma que tengamos acceso a métodos como ``useQuery`` y ``useMutation`` más adelante.

Crea un archivo en el directorio de nivel superior llamado ``apollo-client.js``

```
thatguyintech@albert road-to-lens % touch apollo-client.js
```
  
Inicializaremos un cliente aquí con la url base apuntando a [la API Mainnet de Lens Matic:](https://docs.lens.xyz/docs/api-links)

```
// ./apollo-client.js

import { ApolloClient, InMemoryCache } from "@apollo/client";

const client = new ApolloClient({
    uri: "https://api.lens.dev",
    cache: new InMemoryCache(),
});

export default client;
```

Con este cliente GraphQL inicializado, podemos importarlo en nuestro archivo ``/pages/_app.js`` y usarlo para envolver nuestro Componente global de la app:

```

// pages/_app.js

import '../styles/globals.css'
import { ApolloProvider } from "@apollo/client";
import client from "../apollo-client";

function MyApp({ Component, pageProps }) {
  return (
    <ApolloProvider client={client}>
      <Component {...pageProps} />
    </ApolloProvider>
  );
}

export default MyApp
  
```
  
Aquí puedes ver que hemos añadido el ``<ApolloProvider client={client}>`` como envoltorio. Esto le da superpoderes a toda nuestra app -- en todas partes podremos usar métodos de utilidad como ``useQuery`` y ``useMutation`` para obtener datos de la API de Lens y también para enviar actualizaciones.

Una última actualización antes de que podamos comprobar de nuevo en localhost. Vamos a actualizar ``/pages/index.js`` para hacer una consulta para obtener perfiles recomendados de Lens:

```
import { useQuery, gql } from "@apollo/client";

const recommendProfiles = gql`
  query RecommendedProfiles {
    recommendedProfiles {
          id
        name
        bio
        attributes {
          displayType
          traitType
          key
          value
        }
          followNftAddress
        metadata
        isDefault
        picture {
          ... on NftImage {
            contractAddress
            tokenId
            uri
            verified
          }
          ... on MediaSet {
            original {
              url
              mimeType
            }
          }
          __typename
        }
        handle
        coverPicture {
          ... on NftImage {
            contractAddress
            tokenId
            uri
            verified
          }
          ... on MediaSet {
            original {
              url
              mimeType
            }
          }
          __typename
        }
        ownedBy
        dispatcher {
          address
          canUseRelay
        }
        stats {
          totalFollowers
          totalFollowing
          totalPosts
          totalComments
          totalMirrors
          totalPublications
          totalCollects
        }
        followModule {
          ... on FeeFollowModuleSettings {
            type
            amount {
              asset {
                symbol
                name
                decimals
                address
              }
              value
            }
            recipient
          }
          ... on ProfileFollowModuleSettings {
          type
          }
          ... on RevertFollowModuleSettings {
          type
          }
        }
    }
  }
`;

export default function Home() {
  const {loading, error, data} = useQuery(recommendProfiles);

  if (loading) return 'Loading..';
  if (error) return `Error! ${error.message}`;

  return (
    <div>
      Hello
      {data.recommendedProfiles.map((profile, index) => {
        console.log(`Profile ${index}:`, profile);
        return (
          <div>
            <h1>{profile.name}</h1>
            <p>{profile.bio}</p>
            <div>{profile.attributes.map((attr, idx) => {
              if (attr.key === "website") {
                return <div><a href={`${attr.value}`}>{attr.value}</a><br/></div>
              } else if (attr.key === "twitter") {
                return <div><a href={`https://twitter.com/${attr.value}`}>@{attr.value}</a><br/></div>;
              }
              return(<div>{attr.value}</div>);
            })}</div>
          </div>
        );
      })}
    </div>
  )
}
  
```

Estamos haciendo un par de cosas clave con este cambio:

1. Definir una consulta GraphQL llamada ``RecommendedProfiles.``
2. Obtener una lista de perfiles llamando a ``useQuery`` con la consulta ``RecommendedProfiles`` -> que se devuelve en la variable ``data``.
3. Muestra alguna información del perfil como ``data.profile.name``, ``data.profile.bio``, y ``data.profile.attributes.``
  
Vuelve a http://localhost:3000/ (asegúrate de que tu servidor local está funcionando), y entonces voilá, ¡deberías ver esta lista de perfiles realmente simple!

![](https://files.readme.io/ce8bb50-localhost.png)

Feo, pero guay, ¿verdad?

Si quieres hacer una pausa aquí y explorar un poco más, puedes añadir una declaración ``console.log`` en tu archivo ``/pages/index.js`` para ver más datos devueltos por la consulta GraphQL.

Por ejemplo, añadiendo ``console.log(data)`` , podrás abrir la consola de desarrollo en tu navegador y ver los datos del perfil.


![](https://files.readme.io/c1692a0-jouni.png)
  
Echa un vistazo a la respuesta ``RecommendedProfiles`` en la consola, o en los documentos para desarrolladores

Veremos estos datos más adelante y limpiaremos los diseños de las páginas 🙂 .

**y un recordatorio:** ¡todos esos datos se almacenan en la blockchain de Polygon como NFTs!

Lo que el equipo de Lens está haciendo con su API es simplemente indexar todos los datos de la cadena para que sea más fácil para los desarrolladores obtenerlos y construirlos usando las NFT.

Una rápida comprobación en este punto...

Tú estructura de directorios debe ser algo como esto:
  
```
thatguyintech@albert road-to-lens % tree -L 2
.
├── README.md
├── apollo-client.js      <- we created this
├── next.config.js
├── node_modules
├── package-lock.json
├── package.json
├── pages
│   ├── api
│   ├── _app.js           <- we modified this
│   └── index.js          <- we modified this
├── public
│   ├── favicon.ico
│   └── vercel.svg
├── styles
│   ├── Home.module.css
│   └── globals.css
└── yarn.lock
```
                             
## Segundo paso. Mejoremos el aspecto de la lista de perfiles
                             
En esta parte, vamos a refactorizar nuestro código para que sea más fácil de navegar, y vamos a dar estilo a nuestros componentes para que la interfaz de usuario se vea mucho más limpia.

Comenzando con la refactorización:

Aquí está nuestro nuevo ``/pages/index.js``

```
import { useQuery } from "@apollo/client";
import recommendedProfilesQuery from '../queries/recommendedProfilesQuery.js';
import Profile from '../components/Profile.js';

export default function Home() {
  const {loading, error, data} = useQuery(recommendedProfilesQuery);

  if (loading) return 'Loading..';
  if (error) return `Error! ${error.message}`;

  return (
    <div>
      {data.recommendedProfilesQuery.map((profile, index) => {
        console.log(`Profile ${index}:`, profile);
        return <Profile key={profile.id} profile={profile} displayFullProfile={false} />;
      })}
    </div>
  )
}
  
```
  
Vamos a mover el ``recommendProfilesQuery`` en un documento separado Graphql en una nueva carpeta que debes crear llamada ``queries``:

```
mkdir queries
touch queries/recommendedProfilesQuery.js
```
  
y luego copia el documento ``RecommendedProfiles`` a este archivo:

```
// queries/recommendedProfilesQuery.js

import {gql} from '@apollo/client';

export default gql`
  query RecommendedProfiles {
    recommendedProfiles {
          id
        name
        bio
        attributes {
          displayType
          traitType
          key
          value
        }
          followNftAddress
        metadata
        isDefault
        picture {
          ... on NftImage {
            contractAddress
            tokenId
            uri
            verified
          }
          ... on MediaSet {
            original {
              url
              mimeType
            }
          }
          __typename
        }
        handle
        coverPicture {
          ... on NftImage {
            contractAddress
            tokenId
            uri
            verified
          }
          ... on MediaSet {
            original {
              url
              mimeType
            }
          }
          __typename
        }
        ownedBy
        dispatcher {
          address
          canUseRelay
        }
        stats {
          totalFollowers
          totalFollowing
          totalPosts
          totalComments
          totalMirrors
          totalPublications
          totalCollects
        }
        followModule {
          ... on FeeFollowModuleSettings {
            type
            amount {
              asset {
                symbol
                name
                decimals
                address
              }
              value
            }
            recipient
          }
          ... on ProfileFollowModuleSettings {
          type
          }
          ... on RevertFollowModuleSettings {
          type
          }
        }
    }
  }
`;
```
  
Luego vamos a crear el componente ``Profile`` que introdujimos en la página ``index.js`` más arriba ( ``import Profile from '../components/Profile.js';)``

Debemos organizarlo en un nuevo directorio llamado ``components``.

```
mkdir components
touch components/Profile.js
```
  
Y luego simplemente copiamos esta estructura:

  ```
// components/Profile.js

import Link from "next/link";
export default function Profile(props) {
  const profile = props.profile;

  // When displayFullProfile is true, we show more info.
  const displayFullProfile = props.displayFullProfile;

  return (
    <div className="p-8">
      <Link href={`/profile/${profile.id}`}>
        <div className="max-w-md mx-auto bg-white rounded-xl shadow-md overflow-hidden md:max-w-2xl">
          <div className="md:flex">
            <div className="md:shrink-0">
              {profile.picture ? (
                <img
                  src={
                    profile.picture.original
                      ? profile.picture.original.url
                      : profile.picture.uri
                  }
                  className="h-48 w-full object-cover md:h-full md:w-48"
                />
              ) : (
                <div
                  style={{
                    backgrondColor: "gray",
                  }}
                  className="h-48 w-full object-cover md:h-full md:w-48"
                />
              )}
            </div>
            <div className="p-8">
              <div className="uppercase tracking-wide text-sm text-indigo-500 font-semibold">
                {profile.handle}
                {displayFullProfile &&
                  profile.name &&
                  " (" + profile.name + ")"}
              </div>
              <div className="block mt-1 text-sm leading-tight font-medium text-black hover:underline">
                {profile.bio}
              </div>
              <div className="mt-2 text-sm text-slate-900">{profile.ownedBy}</div>
              <p className="mt-2 text-xs text-slate-500">
                following: {profile.stats.totalFollowing} followers:{" "}
                {profile.stats.totalFollowers}
              </p>
            </div>
          </div>
        </div>
      </Link>
    </div>
  );
}
```  
  
Este componente Perfil acepta props como entrada, y espera que el objeto props tenga un campo de perfil que incluya toda la información que queramos mostrar cuando rendericemos el componente.

Ya que estamos aquí, ¡añadimos también la foto de perfil y el recuento de seguidores/seguidos!

Ahora, una vez más, vamos a comprobar nuestro trabajo. Revisar nuestro trabajo a menudo es un buen hábito porque nos permite detectar errores rápidamente

Asegúrate de que tu servidor está funcionando:
  
```
thatguyintech@albert road-to-lens % npm run dev

> road-to-lens@0.1.0 dev
> next dev

ready - started server on 0.0.0.0:3000, url: http://localhost:3000

```
Deberías ver algo como esto

![](https://files.readme.io/e5da8e9-stani.lens.png)

Todavía no se ve muy bien, pero ¡tenemos fotos!

Antes de terminar este paso, aprovechemos el CSS que incluimos en el componente Perfil.

Si nunca has usado el CSS de Tailwind, estas etiquetas:

``uppercase tracking-wide text-sm text-indigo-500 font-semibold``

provienen de los fundamentos de utilidad del diseño de Tailwind [(ver documentación).](https://tailwindcss.com/docs/responsive-design)

Así que realmente todo lo que tenemos que hacer aquí es instalar Tailwind, según sus [instrucciones de instalación:](https://tailwindcss.com/docs/guides/nextjs)

```
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
  
```
  
Copiamos estas configuraciones para que Tailwind sepa en qué rutas cargar los estilos de nuestro proyecto. Queremos que todo en la carpeta ``pages`` y la carpeta ``components``esté cubierto, así que usamos estas rutas:

```

// tailwind.config.js

module.exports = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx}",
    "./components/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
  
```

Ahora terminamos añadiendo las directivas Tailwind a nuestro archivo CSS:

```
/* ./styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
  
```
  
¡Y luego reinicia el servidor y mira cómo se ve ahora!


![](https://files.readme.io/5ca2381-garethLens.png)
  
mucho mejor aspecto después de estilizar con Tailwind CSS

  
Tiene un aspecto increíble. Incluso tiene sombras y todo 🙂 .

¡Ahora .. algunos de ustedes pueden haber notado que he incluido algunos <Links> en el componente Perfil, y luego, cuando se hace clic en uno de los perfiles, se vincula a una nueva página!

Por ejemplo, al hacer clic en la tarjeta de ``DAVIDEV.LENS`` nos lleva a http://localhost:3000/profile/0x16 y muestra un error 404.


![](https://files.readme.io/16cdd1a-404.png)
404 ¡oh no!


No tengas miedo, en el siguiente paso vamos a construir esta página y hacer una segunda consulta GraphQL para obtener Publicaciones (aka "posts" como en otras plataformas de redes sociales).

## Paso 3. Crear páginas de perfil individuales
  
Una de las cosas más interesantes de construir rápidamente con el framework Next.js es el enrutamiento. Para nuestras páginas de perfil individuales, todo lo que tenemos que hacer para crear la página es crear una carpeta bajo la carpeta pages que tenga el mismo nombre que nuestro componente ``Profile``. Y luego añadir un archivo ``[id].js`` que se puede utilizar para cualquier id dinámico pasado a la ruta.

Esto es lo que quiero decir:

```
road-to-lens % mkdir pages/profile
road-to-lens % touch pages/profile/\[id\].js
```

Llenaremos el archivo ``pages/profile/[id].js`` en un segundo. Pero con esta estructura de archivos en su lugar, nuestra aplicación ``Next.js`` será capaz de responder a solicitudes de URLs como:

+ http://localhost:3000/profile/0x9752
+ http://localhost:3000/profile/0x25c4

Así que vamos a terminar esto mediante la creación de una nueva solicitud GraphQL a la API de Lens para obtener información de perfil, y luego vamos a terminar mediante la construcción del componente en el archivo ``[id].js``.

Para obtener información de perfiles individuales, vamos a crear un documento GraphQL como lo hicimos anteriormente para obtener toda la lista de perfiles recomendados:

```
road-to-lens % touch queries/fetchProfileQuery.js
```
  
```
// queries/fetchProfileQuery.js

import { gql } from '@apollo/client';

export default gql`
query($request: SingleProfileQueryRequest!) {
    profile(request: $request) {
        id
        name
        bio
        attributes {
          displayType
          traitType
          key
          value
        }
        followNftAddress
        metadata
        isDefault
        picture {
          ... on NftImage {
            contractAddress
            tokenId
            uri
            verified
          }
          ... on MediaSet {
            original {
              url
              mimeType
            }
          }
          __typename
        }
        handle
        coverPicture {
          ... on NftImage {
            contractAddress
            tokenId
            uri
            verified
          }
          ... on MediaSet {
            original {
              url
              mimeType
            }
          }
          __typename
        }
        ownedBy
        dispatcher {
          address
          canUseRelay
        }
        stats {
          totalFollowers
          totalFollowing
          totalPosts
          totalComments
          totalMirrors
          totalPublications
          totalCollects
        }
        followModule {
          ... on FeeFollowModuleSettings {
            type
            amount {
              asset {
                symbol
                name
                decimals
                address
              }
              value
            }
            recipient
          }
          ... on ProfileFollowModuleSettings {
            type
          }
          ... on RevertFollowModuleSettings {
            type
          }
        }
    }
  }
`;
```

Ahora vamos a utilizar esta consulta en la página del perfil individual.
  
```
// pages/profile/[id].js

import { useQuery } from "@apollo/client";
import { useRouter } from "next/router";
import fetchProfileQuery from "../../queries/fetchProfileQuery.js";

import Profile from "../../components/Profile.js";

export default function ProfilePage() {
  const router = useRouter();
  const { id } = router.query;

  console.log("fetching profile for", id);
  const { loading, error, data } = useQuery(fetchProfileQuery, {
    variables: { request: { profileId: id } },
  });

  if (loading) return "Loading..";
  if (error) return `Error! ${error.message}`;

  console.log("on profile page data: ", data);

  return <Profile profile={data.profile} displayFullProfile={true}/>
}
```

Si echas un vistazo al archivo que hemos añadido, verás una llamada ``useQuery()`` familiar que toma el documento ``fetchProfileQuery`` GraphQL importado de ``"../../queries/fetchProfileQuery.js"``.

La novedad es que ahora también introducimos el hook ``useRouter()``. Este hook nos permite tomar el valor de la variable ``[id]`` de los parámetros de la consulta en la url en tiempo de ejecución, y utilizarlo en nuestro código.

Esto significa que si tenemos estas URLs:

+ http://localhost:3000/profile/0x9752
+ http://localhost:3000/profile/0x25c4
  
La llamada a ``useRouter()`` obtendría los siguientes ids para ``const { id } = router.query;`` :

+ 0x9752
+ 0x25c4

Y luego usamos esos ``id``s para pasarlos a la ``fetchProfileQuery`` de modo que podamos obtener sólo el perfil de la persona por la que estamos navegando en ese momento. Puedes encontrar la documentación completa de la [petición Get Profile aquí.](https://docs.lens.xyz/docs/get-profile)

Vale, pulsa Guardar. Actualiza la página. ¿Qué ves?

![](https://files.readme.io/e1a8ed7-vitto.png)
id: 0x9752
  
![](https://files.readme.io/f7aa1da-albert.png)
id: 0x25c4

¡Vitto y Albert!

Muy bien, si todo va bien hasta ahora - asegúrete de celebrar este hito (o pedir ayuda si estás atascado).

Ya has recorrido más de la mitad del camino.

## Paso 4. Cargar las publicaciones de los usuarios en la página de perfil
  
Vamos a hacer las páginas de perfil un poco más interesantes mediante la obtención de algunos de los mensajes que el usuario ha hecho.

En Lens Protocol, las publicaciones, los comentarios y las réplicas (también conocidas como "retweets") se clasifican en el modelo de datos ``Publications``.

Puedes encontrar la documentación de los [endpoints de Publicación aquí.](https://docs.lens.xyz/docs/publication-1) Nos centraremos en la operación [GET Publication.](https://docs.lens.xyz/docs/get-publication)

Una vez más, vamos a seguir el mismo orden de:

1. Configurar el documento de consulta GraphQL.
2. Actualizar la página de perfil para utilizar la nueva consulta GraphQL.
3. Crear una nueva definición de componente para "posts".
4. Pasar los datos devueltos por Lens API a los nuevos componentes.
5. Probar y depurar la página.
  
### 1. Configurar el documento de consulta GraphQL
  
Esta vez no necesitaremos un nuevo archivo de documento. Podemos modificar la consulta que iniciamos en ``fetchProfileQuery.js`` para pedir también ``publications`` en la misma solicitud. ¡Ese es el poder de GraphQL! A diferencia de las llamadas a la API REST, puedes solicitar tanto o tan poco como quieras en una sola solicitud. Hagamos esta actualización a ``queries/fetchProfileQuery.js``:

```
import { gql } from "@apollo/client";

export default gql`
  query (
    $request: SingleProfileQueryRequest!
    $publicationsRequest: PublicationsQueryRequest!
  ) {
    publications( request: $publicationsRequest) {
      items {
        __typename
        ... on Post {
          ...PostFields
        }
        ... on Comment {
          ...CommentFields
        }
        ... on Mirror {
          ...MirrorFields
        }
      }
      pageInfo {
        prev
        next
        totalCount
      }
    }
    profile(request: $request) {
      id
      name
      bio
      attributes {
        displayType
        traitType
        key
        value
      }
      followNftAddress
      metadata
      isDefault
      picture {
        ... on NftImage {
          contractAddress
          tokenId
          uri
          verified
        }
        ... on MediaSet {
          original {
            url
            mimeType
          }
        }
        __typename
      }
      handle
      coverPicture {
        ... on NftImage {
          contractAddress
          tokenId
          uri
          verified
        }
        ... on MediaSet {
          original {
            url
            mimeType
          }
        }
        __typename
      }
      ownedBy
      dispatcher {
        address
        canUseRelay
      }
      stats {
        totalFollowers
        totalFollowing
        totalPosts
        totalComments
        totalMirrors
        totalPublications
        totalCollects
      }
      followModule {
        ... on FeeFollowModuleSettings {
          type
          amount {
            asset {
              symbol
              name
              decimals
              address
            }
            value
          }
          recipient
        }
        ... on ProfileFollowModuleSettings {
          type
        }
        ... on RevertFollowModuleSettings {
          type
        }
      }
    }
  }

  fragment MediaFields on Media {
    url
    mimeType
  }

  fragment ProfileFields on Profile {
    id
    name
    bio
    attributes {
      displayType
      traitType
      key
      value
    }
    isFollowedByMe
    isFollowing(who: null)
    followNftAddress
    metadata
    isDefault
    handle
    picture {
      ... on NftImage {
        contractAddress
        tokenId
        uri
        verified
      }
      ... on MediaSet {
        original {
          ...MediaFields
        }
      }
    }
    coverPicture {
      ... on NftImage {
        contractAddress
        tokenId
        uri
        verified
      }
      ... on MediaSet {
        original {
          ...MediaFields
        }
      }
    }
    ownedBy
    dispatcher {
      address
    }
    stats {
      totalFollowers
      totalFollowing
      totalPosts
      totalComments
      totalMirrors
      totalPublications
      totalCollects
    }
    followModule {
      ... on FeeFollowModuleSettings {
        type
        amount {
          asset {
            name
            symbol
            decimals
            address
          }
          value
        }
        recipient
      }
      ... on ProfileFollowModuleSettings {
        type
      }
      ... on RevertFollowModuleSettings {
        type
      }
    }
  }

  fragment PublicationStatsFields on PublicationStats {
    totalAmountOfMirrors
    totalAmountOfCollects
    totalAmountOfComments
  }

  fragment MetadataOutputFields on MetadataOutput {
    name
    description
    content
    media {
      original {
        ...MediaFields
      }
    }
    attributes {
      displayType
      traitType
      value
    }
  }

  fragment Erc20Fields on Erc20 {
    name
    symbol
    decimals
    address
  }

  fragment CollectModuleFields on CollectModule {
    __typename
    ... on FreeCollectModuleSettings {
      type
      followerOnly
      contractAddress
    }
    ... on FeeCollectModuleSettings {
      type
      amount {
        asset {
          ...Erc20Fields
        }
        value
      }
      recipient
      referralFee
    }
    ... on LimitedFeeCollectModuleSettings {
      type
      collectLimit
      amount {
        asset {
          ...Erc20Fields
        }
        value
      }
      recipient
      referralFee
    }
    ... on LimitedTimedFeeCollectModuleSettings {
      type
      collectLimit
      amount {
        asset {
          ...Erc20Fields
        }
        value
      }
      recipient
      referralFee
      endTimestamp
    }
    ... on RevertCollectModuleSettings {
      type
    }
    ... on TimedFeeCollectModuleSettings {
      type
      amount {
        asset {
          ...Erc20Fields
        }
        value
      }
      recipient
      referralFee
      endTimestamp
    }
  }

  fragment PostFields on Post {
    id
    profile {
      ...ProfileFields
    }
    stats {
      ...PublicationStatsFields
    }
    metadata {
      ...MetadataOutputFields
    }
    createdAt
    collectModule {
      ...CollectModuleFields
    }
    referenceModule {
      ... on FollowOnlyReferenceModuleSettings {
        type
      }
    }
    appId
    hidden
    mirrors(by: null)
    hasCollectedByMe
  }

  fragment MirrorBaseFields on Mirror {
    id
    profile {
      ...ProfileFields
    }
    stats {
      ...PublicationStatsFields
    }
    metadata {
      ...MetadataOutputFields
    }
    createdAt
    collectModule {
      ...CollectModuleFields
    }
    referenceModule {
      ... on FollowOnlyReferenceModuleSettings {
        type
      }
    }
    appId
    hidden
    hasCollectedByMe
  }

  fragment MirrorFields on Mirror {
    ...MirrorBaseFields
    mirrorOf {
      ... on Post {
        ...PostFields
      }
      ... on Comment {
        ...CommentFields
      }
    }
  }

  fragment CommentBaseFields on Comment {
    id
    profile {
      ...ProfileFields
    }
    stats {
      ...PublicationStatsFields
    }
    metadata {
      ...MetadataOutputFields
    }
    createdAt
    collectModule {
      ...CollectModuleFields
    }
    referenceModule {
      ... on FollowOnlyReferenceModuleSettings {
        type
      }
    }
    appId
    hidden
    mirrors(by: null)
    hasCollectedByMe
  }

  fragment CommentFields on Comment {
    ...CommentBaseFields
    mainPost {
      ... on Post {
        ...PostFields
      }
      ... on Mirror {
        ...MirrorBaseFields
        mirrorOf {
          ... on Post {
            ...PostFields
          }
          ... on Comment {
            ...CommentMirrorOfFields
          }
        }
      }
    }
  }

  fragment CommentMirrorOfFields on Comment {
    ...CommentBaseFields
    mainPost {
      ... on Post {
        ...PostFields
      }
      ... on Mirror {
        ...MirrorBaseFields
      }
    }
  }
`;
  
```
  
Ten en cuenta que hemos añadido un nuevo argumento de entrada de consulta: ``$publicationsRequest``: ``PublicationsQueryRequest!``. Esto es algo que tendremos que asegurarnos de pasar como argumento de entrada cuando lancemos la consulta.

### 3. Actualizar la página de perfil para utilizar la nueva consulta GraphQL.
  
En la página de perfil, la única cosa que realmente necesitamos hacer para este paso es añadir la solicitud de publicación para los mensajes que queremos. La solicitud en sí debe tener este aspecto:

```
publicationsRequest: {
  profileId: id,
  publicationTypes: ["POST"], // We really only want POSTs
},
```
Así que vamos a añadirlo a las opciones del argumento de entrada useQuery:
  
```
// pages/profile/[id].js

import { useQuery, useMutation } from "@apollo/client";
import { useRouter } from "next/router";
import fetchProfileQuery from "../../queries/fetchProfileQuery.js";
import Profile from "../../components/Profile.js";

export default function ProfilePage() {
  const router = useRouter();
  const { id } = router.query;

  console.log("fetching profile for", id);
  const { loading, error, data } = useQuery(fetchProfileQuery, {
    variables: {
      request: { profileId: id },
      publicationsRequest: {
        profileId: id,
        publicationTypes: ["POST"], // We really only want POSTs
      },
    },
  });

  if (loading) return "Loading..";
  if (error) return `Error! ${error.message}`;

  console.log("on profile page data: ", data);
  return (
    <div className="flex flex-col p-8 items-center">
      <Profile profile={data.profile} displayFullProfile={true} />
    </div>
  );
}
```
  
### 3. Crear una nueva definición de componente para "posts".
  
Necesitamos un nuevo componente React para mostrar los datos recuperados sobre los posts del usuario. Vamos a crear uno en ``components/Post.js``:

```
// components/Post.js
export default function Post(props) {
  const post = props.post;

  return (
    <div className="p-8">
      <div className="max-w-md mx-auto bg-white rounded-xl shadow-md overflow-hidden md:max-w-2xl">
        <div className="md:flex">
          <div className="p-8">
            <p className="mt-2 text-xs text-slate-500 whitespace-pre-line">
              {post.metadata.content}
            </p>
          </div>
        </div>
      </div>
    </div>
  );
}
```

Si comparas este componente simple con el componente ``Profile``, verás que ambos siguen un patrón muy simple, que es que aceptan un objeto ``props`` que tiene los datos que necesita el componente para renderizar ciertas partes del mismo.

En este caso, realmente sólo necesitamos una pieza central de información de los posts:

+ ``post.metadata.content``
  
Dado que este campo de metadatos era el único que necesitábamos, puedes volver al documento GraphQL más tarde y eliminar los campos que no estamos utilizando en la funcionalidad final.

Este componente también utiliza Tailwind CSS para dar estilo.

### 4. Pasar los datos devueltos por Lens API a los nuevos componentes
  
Volvemos a la página de perfil ``(pages/profile/[id].js)``:

Ahora podemos importar el componente ``<Post/>`` a la página de perfil y utilizarlo para renderizar todas las publicaciones que obtenemos de la API.
  
```
import { useQuery, useMutation } from "@apollo/client";
import { useRouter } from "next/router";
import fetchProfileQuery from "../../queries/fetchProfileQuery.js";
import Profile from "../../components/Profile.js";
import Post from "../../components/Post.js";

export default function ProfilePage() {
  const router = useRouter();
  const { id } = router.query;

  console.log("fetching profile for", id);
  const { loading, error, data } = useQuery(fetchProfileQuery, {
    variables: {
      request: { profileId: id },
      publicationsRequest: {
        profileId: id,
        publicationTypes: ["POST"],
      },
    },
  });

  if (loading) return "Loading..";
  if (error) return `Error! ${error.message}`;

  return (
    <div className="flex flex-col p-8 items-center">
      <Profile profile={data.profile} displayFullProfile={true} />
      {data.publications.items.map((post, idx) => {
        return <Post key={idx} post={post}/>;
      })}
    </div>
  );
}

```
  
Observa cerca de la parte inferior del fragmento que en realidad eran estas líneas de código:
  
```
      {data.publications.items.map((post, idx) => {
        return <Post key={idx} post={post}}/>;
      })}
  
```
que están haciendo el trabajo pesado en este paso. Pero con esto añadido, ¡deberíamos poder probar nuestra función de posts!

### 5. Probar y depurar
  
Si todo va bien, ahora deberías ser capaz de ver los mensajes poblados bajo la tarjeta de perfil :tada:

![](https://files.readme.io/1be65ab-nada.lens.png)

Enhorabuena, estás en el buen camino para convertirte en un desarrollador de redes sociales descentralizadas :)

### Otras API y recursos
  
Con suerte, a estas alturas, ya habrás completado algunos de los otros desafíos de Road To Web3 y habrás disfrutado del proceso de llegar a este punto. La lección de esta semana trataba sobre cómo utilizar APIs construidas sobre NFTs y un protocolo descentralizado.

Te recomiendo encarecidamente que te tomes tu tiempo para explorar la documentación de Lens, probar diferentes puntos finales y leer el código de ejemplo alojado en este repositorio:

[GitHub - aave/lens-api-examples: ejemplos de código de cómo hablar con la API lens](https://github.com/aave/lens-api-examples)

Aquí hay algunas otras apis web3 en la vanguardia:

+ [Las APIs MintKudos PoK Tokens](https://contributionlabs.notion.site/Public-APIs-054cc0a8110a45a9be326fda382bb6bc#6e2810e0d8984496ad7a8674adf12dfd) - creadas por [Kei](https://twitter.com/keikumata) @ Contribution Labs
+ [Lit Protocol SDK y tutorial](https://litprotocol.notion.site/Decentralized-Unlockable-Content-21a2a1dadb4a4845a5a77ab2589069a0) - creado por [Deb](https://twitter.com/deb_i_deb) @ Lit Protocol
  
Con estas herramientas y el acceso a los datos en tus manos, estoy realmente emocionado de ver lo que todos podran crear 🙂.

¿Una Red Social de Yoga Web3?

¿Un lugar de encuentro exclusivo para los poseedores de tokens Road to Web3 PoK?

Un aula en línea de la nueva era donde los profesores reciben el pago directamente de los estudiantes por el contenido y la educación pieza por pieza a través de su dapp?

Las posibilidades son infinitas :fire:

### Desafíos
  
El contenido de esta semana es complejo, así que las opciones de desafío que te doy son intencionadamente vagas para que puedas explorar como quieras. Elige al menos una de las opciones que te propongo a continuación y comprueba hasta dónde puedes llegar.

1. Utilizando la API Lens, consulta una nueva información que no se haya tratado en el tutorial y muéstrala en tu sitio web.
2. Utilizando la API de Lens, averigua cómo realizar autenticaciones y mutaciones para acciones como reaccionar, seguir o crear nuevas publicaciones.
3. Despliega tu aplicación en la World Wide Web, utilizando un servicio como Repl.it o Vercel.
4. Utiliza el SDK de Lit Protocol para token-gatear el acceso a tus publicaciones del protocolo Lens, de modo que sólo los poseedores del token MintKudos puedan verlas.

No olvides enviar tu trabajo aquí para conseguir la Prueba de Conocimiento NFT de esta semana.

Discord de la Universidad de Alquimia: [https://university.alchemy.com/discord)

### ¡Enhorabuena!

**¡Has aprendido con éxito a navegar, consultar y construir usando Lens Protocol! 🔥**
