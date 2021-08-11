# Deploy en Vercel y Heroku

## Setup Backend

Importante poner Nodemon como dependencia de produccion haciendo

```js
npm i nodemon
```

### <b>archivo index.js en api</b>

cambiar la conexion del puerto a la siguiente forma:

```js
conn.sync({ force: true }).then(() => {
  server.listen(process.env.PORT, () => {
    console.log("%s listening at 3000"); // eslint-disable-line no-console
  });
});
```

### <b>archivo app.js en api</b>

instalar cors y requerirlo en el back:

```js
const cors = require("cors");

server.use(cors());
```

despues en el 'Access-Control-Allow-Origin' setear el 2do parametro con un '\*'
para permitir conexiones externas

### <b>en db.js: </b>

reemplazar la conexion de Sequelize:

```js
let sequelize =
  process.env.NODE_ENV === "production"
    ? new Sequelize({
        database: DB_NAME,
        dialect: "postgres",
        host: DB_HOST,
        port: 5432,
        username: DB_USER,
        password: DB_PASSWORD,
        pool: {
          max: 3,
          min: 1,
          idle: 10000,
        },
        dialectOptions: {
          ssl: {
            require: true,
            // Ref.: https://github.com/brianc/node-postgres/issues/2009
            rejectUnauthorized: false,
          },
          keepAlive: true,
        },
        ssl: true,
      })
    : new Sequelize(
        `postgres://${DB_USER}:${DB_PASSWORD}@${DB_HOST}/development`,
        { logging: false, native: false }
      );
```

## Setup Front

```js
npm i dotenv
```

en index.js agregar:

```js
import axios from "axios";
import dotenv from "dotenv";
dotenv.config();

axios.defaults.baseURL = process.env.REACT_APP_API || "http://localhost:3001";
```

Con eso que agregamos arriba ya seteamos directamente la url base para Axios, por lo que ahora tendremos que modificar directamente en
nuestras actions de Redux cambiando por ej:

```js
axios.get("http://localhost:3001/ejemplo");
```

A

```js
axios.get("/ejemplo");
```

Justamente porque la URL base ya esta seteada en el index.

================================================================

# Setup Heroku

- 1- Clickeamos en 'Create App' y ponemos el nombre en 'App Name'.
- 2- Conectamos en Deployment Method al repositorio donde tenemos el proyecto a Deployar (Click en GitHub y buscamos en el Searchbar).
- 3- Ir a la pestaña Resources y poner en 'add-ons' y buscar 'Heroku Postgres', seleccionamos la opcion 'Hobby Dev' en 'Plan name' y
  le damos a 'Submit Order Form'.
- 4- Le damos click a Heroku Postgres para que se abra en otra pestaña y ahi vamos a la pestaña "Settings" y "View Credentials"
- 5- Seguimos ahora en la pestaña 'Settings' de nuestra app de Heroku y vamos a Config Vars(Aca es donde vamos a setear las variables de entorno usando la
  pestaña 'Settings' de Heroku Postgres)
- 6- En Config Vars vamos a setear las variables de entorno de la siguiente manera:

| Key          | Value                                                |
| ------------ | ---------------------------------------------------- |
| DATABASE_URL | (valor que obtenemos de Heroku Postgres en URI)      |
| DB_USER      | (valor que obtenemos de Heroku Postgres en User)     |
| DB_PASSWORD  | (valor que obtenemos de Heroku Postgres en Password) |
| DB_HOST      | (valor que obtenemos de Heroku Postgres en Host)     |
| DB_NAME      | (valor que obtenemos de Heroku Postgres en Database) |
| PROJECT_PATH | /api                                                 |

Una vez seteado todo esto vamos un poco mas abajo donde tenemos la opcion "Buildpacks"
Vamos a presionar en 'Add Buildpack' y pondremos lo siguiente:

- En enter buildpack URL pegamos esto https://github.com/timanovsky/subdir-heroku-buildpack
- Despues seleccionamos Node.js (Es muy importante seguir el orden porque si no lo hacen tira error de Build)

Luego volvemos a la pestaña 'Deploy', habilitamos los Automatic Deploys y elegimos la Branch Main(Esto es para que se actualice automaticamente cuando haya algun cambio en la rama)
Clickeamos 'Enable Automatic Deploys' y para finalizar presionamos en 'Deploy Branch'

Listo! Ya tenemos el deploy del back hecho.

## Setup Vercel

- Vamos a vercel.com/new y preferentemente nos logueamos con nuestro GitHub para tener los repositorios listos a importar.
- Seleccionamos 'Import' en el Repo en cuestion en la pestaña 'import Git Repository'
- En la opcion de Create a Team le damos a Skip
- Wn configure Project seteamos el Project name
- Ponemos como Framework Preset 'Create React App'
- Wn Root directort ponemos 'edit' y seleccionamos la carpeta 'client'
- Luego en Environment Variables pondremos la unica variable de entorno que vamos a necesitar como REACT_APP_API
- Como valor le vamos a asignar nuestra pagina principal de Heroku por ejemplo: mi-aplicacion.herokuapp.com (es importante
  que NO tenga la barra al final cuando lo copiamos porque daria error ej: mi-aplicacion.herokuapp.com/ ) presionamos ADD.
- Terminamos presionando DEPLOY y ya con eso tendriamos nuestra App completa en Deploy!
