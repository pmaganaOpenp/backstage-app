# POC Backstage Openpay


### Tecnologías del proyecto

* Backstage latest version
* GitHub
* Docker
* PostgreSQL
* Node.js

### Prerequisitos

* Acceso a consola Linux, MacOS o Susbsitema de Linux en Windows
  * En MacOs ejecutar ```xcode-select --install```
* curl o wget instalado 
* Uso de nvm
  * https://github.com/nvm-sh/nvm#install--update-script
* yarn
  * https://classic.yarnpkg.com/en/docs/install#mac-stable
* docker
  * https://docs.docker.com/engine/install/
* git
  * https://github.com/git-guides/install-git

### Creación de la aplicación

Para la creación del aplicativo.

* Cree un directorio en donde se alojará el aplicativo
* Ingrese al directorio y posteriormente ejecute el siguiente comando:

```npx @backstage/create-app@latest```

  * Dentro de la ejecución del comando se solicitará un nombre para el aplicativo.


### Configuración del ambiente de ejecución

* Ingresar al directorio donde se creo la aplicación.
* Ejecutar el siguiente comando para agregar el plugin de conexión a base de datos PostgreSQL

```yarn --cwd packages/backend add pg```

Dentro de la carpeta raíz abrir el archivo app-config.yaml en el apartado de "backend.database" borrar o comentar la configuración de SQLite
y pegar lo siguiente:

```
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
```

* Configura tu base de datos

Configura tu base de datos en el apartado "Configuración base de datos PostgreSQL"


Configura tus variables de entorno de la siguiente manera:
```
export POSTGRES_HOST=localhost
export POSTGRES_USER=backstageuser
export POSTGRES_PASSWORD=backstagepass
export POSTGRES_PORT=5432
```

Para probar la configuración básica sin autenticación de un proveedor externo ejecuta:
```
yarn dev
```

Para detener la ejecución ejecuta el siguiente comando:
```
CTRL + C 
```

* Configura un metodo de autenticación

Configura tu metodo de autenticación en el apartado "Configuración de Autenticación GitHub"

* Una vez realizado las actualizaciones correspondientes ingresa las siguientes variables de entorno:

Los valores "CLIENTE ID" y "SECRET" corresponden a los valores asociados a tu cuenta de GitHub configurada.

```
export AUTH_GITHUB_CLIENT_ID="CLIENTE ID"
export AUTH_GITHUB_CLIENT_SECRET="SECRET"
```

Ahora ya puedes ejecutar nuevamente el aplicativo para validar que puedas acceder mediante la autenticación con GitHub

```
yarn dev
```

### Configuración base de datos PostgreSQL

Para crear una base de datos de pruebas. Si no cuentas con un usuario de PostgreSQL
Dentro del directorio de éste proyecto se cuenta con un archivo docker-compose-dev.yml para crear un contendor de base de datos PostgreSQL
y el cliente pgadmin.

* Descargar el archivo, posicionate en el directorio y ejecuta el siguiente comando:

```
docker compose -f docker-compose-dev.yml up
```

Para visualizar la base de datos en tu navegador coloca localhost ó localhost:80 
El usuario es: admin@admin.com
La contraseña es: admin


La configuración de base de datos es:

```
POSTGRES_HOST=db
POSTGRES_USER=backstageuser
POSTGRES_PASSWORD=backstagepass
POSTGRES_PORT=5432
```


### Configuración de Autenticación GitHub

Registrate en la plataforma de GitHub
En el apartado https://github.com/settings/developers

Creas un nuevo OAuth App

* Deberás proporcionar un nombre para la aplicación
* En Homepage URL de manera local es:
  * http://localhost:3000
* En Authorization callback URL se debe colocar:
  * http://localhost:7007/api/auth/github/handler/frame

Al registra el aplicativo se proporcionará un Client ID

Posteriormente se tiene que generar un nuevo Client Secret

En el archivo app-config.yaml en el apartado auth colocar:

```
auth:
  environment: development
  providers:
    github:
      development:
        clientId: ${AUTH_GITHUB_CLIENT_ID}
        clientSecret: ${AUTH_GITHUB_CLIENT_SECRET}
        signIn:
         resolvers:
          - resolver: usernameMatchingUserEntityName
```


En el directorio root del aplicativo ejecuta los siguientes comandos:

```
yarn --cwd packages/backend add @backstage/plugin-catalog-backend-module-github
```
```
yarn --cwd packages/backend add @backstage/plugin-catalog-backend-module-bitbucket
```

* Ingresa al archivo /packages/app/src/App.tsx para configurar la página de sesión

Se tienen que importar las siguientes líneas:

```
import { githubAuthApiRef, bitbucketAuthApiRef } from '@backstage/core-plugin-api';
```

* El siguiente import en algunos casos ya existe, si ese es el caso omite el importarlo. 

```
import { SignInPage } from '@backstage/core-components';
```

* En el apartado de components colocar lo siguiente: 

```
components: {
  SignInPage: props => (
    <SignInPage
    {...props}
    auto
    providers={[
      'guest',
      {
        id: 'github-auth-provider',
        title: 'GitHub',
        message: 'Sign in using GitHub',
        apiRef: githubAuthApiRef,
      },
      {
        id: 'bitbucket-auth-provider',
        title: 'Bitbucket',
        message: 'Sign in using Bitbucket',
        apiRef: bitbucketAuthApiRef,
      }
      ]}
    />
    ),
    },
```


* Ingresa al archivo /packages/backend/src/index.ts

En el apartado de auth plugin agregar la siguientes líneas

```
backend.add(import('@backstage/plugin-auth-backend-module-github-provider'));
backend.add(import('@backstage/plugin-auth-backend-module-bitbucket-provider'));
```

* Ingresa al archivo /examples/org.yaml

Ingresa los siguiente:

```
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: "Usuario GitHub"
spec:
  memberOf: [guests]
```

- **NOTA**: En metadata.name coloca tu usuario de GitHub.

### Configurar archivo app-config.yaml 

Elimina el apartado siguiente:

```proxy:```

Para que docker pueda iniciar sesión con Github es necesario indicar en el apartado de catalog el archivo org.yaml de la siguiente manera:

```
    - type: url
      target: https://bitbucket.org/openpay-dev/openpay-backstage-backend-poc/src/develop/org.yaml
      rules:
        - allow: [User, Group]
``` 

- **NOTA**: Dentro de target debe de ir la url de tu archivo org.yaml


Para que el archivo org.yaml pueda ser accesado desde los repositorios de Bitbucket, es necesario configurar una contraseña de acceso.

https://support.atlassian.com/bitbucket-cloud/docs/create-an-app-password/


Y se debe de ingresar en el apartado de "integrations" de la siguiente manera:

```
integrations:
  bitbucketCloud:
    - username: ${BITBUCKET_CLOUD_USERNAME}
      appPassword: ${BITBUCKET_CLOUD_PASSWORD}
```


- **NOTA**: El username es el de tu cuenta de bitbucket y lo puedes encontrar en (https://bitbucket.org/account/settings/) Account Settings - Bitbucket profile settings - Username.
            El appPassword es el password generado.


### Configurar archivo app-config.production.yaml

Elimina los apartados siguientes:


```auth:``` y ```catalog:```




### Configuración Docker

En la ruta /packages/backend/Dockerfile

Quitar la configuración de SQLite.

* En este aplicativo ya se tiene un archivo Dockerfile, puedes optar por copiar el archivo y pegarlo en tu proyecto.


Dentro del directorio raíz del aplicativo ejecutar lo siguiente:

```
yarn install --frozen-lockfile
```
```
yarn tsc
```
```
yarn build:backend --config ../../app-config.yaml
```

* Para crear la imagen de docker ejecutar lo siguiente:

```
docker image build . -f packages/backend/Dockerfile --tag backstage
```

* Para crear el contenedor a partir de la imagen creada, ejecuta el siguiente comando:

```
docker run --name backstage-app -p 7007:7007 -e POSTGRES_HOST="Host de la base de datos" -e POSTGRES_PORT="Puerto de la base de datos" -e POSTGRES_USER="Usuario de la base de datos" -e POSTGRES_PASSWORD="Password de la base de datos" -e AUTH_GITHUB_CLIENT_ID="cliente auth de Github" -e AUTH_GITHUB_CLIENT_SECRET="Secreto auth de GitHub" -e BITBUCKET_CLOUD_USERNAME="User name bitbucket" -e BITBUCKET_CLOUD_PASSWORD="password app bitbucket" --network "Red a la que pertenecera el contendor" backstage 
```

* Ingresa al aplicativo en http://localhost:7007

















