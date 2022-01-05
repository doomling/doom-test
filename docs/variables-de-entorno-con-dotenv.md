# Manejando variables de entorno con dotenv

En estos días se habló mucho de GitHub copilot y sus supuestos riesgos exponiendo API keys y API secrets. Esta es obviamente una preocupación legitima, ya que si una persona se hace con estos datos podría acceder a servicios que contienen datos sensibles o **hacer peticiones en nuestro nombre.**

Al margen de GitHub copilot, y el hecho de que no debería estar leyendo y mucho menos usando información de repositorios privados (si esto se ccmprueba, auch, error garrafal de su parte) la realidad es que GitHub está lleno de repositorios públicos donde la gente, distraída, **se olvida y expone información sensible**.

El gran problema de generar commits con información sensible es que si bien podemos borrarlo por la naturaleza de git y su control de versiones **toda la información queda guardada en el history**. Esto no es un problema mientras estamos trabajando de forma local ya que podríamos solucionarlo con un rebase, pero una vez que hacemos push hacer cambios en el history puede traer serios problemas en nuestro proyecto.

Por eso es mejor evitar el problema completamente y tomar cómo práctica **nunca incluir API keys o secrets en nuestros repositorios**. Para esto podemos utilizar **variables de ambiente o entorno.**

En node configurar variables de ambiente es muy simple, para eso vamos a usar dos herramientas, por un lado el objeto global de node **Process** y una pequeña y potente librería: **[dotenv](https://www.npmjs.com/package/dotenv).**

**Process**, como mencionamos antes, **es un objeto global que nos da acceso al proceso actual de node en ejecución** ¿Qué significa esto? Cuando iniciamos nuestra aplicación haciendo uso del comando `node parametros unarchivo.js` se genera un objeto con información a la que podemos acceder de forma inmediata desde cualquier parte de nuestra aplicación.

El objeto Process tiene muchas propiedades, una de ellas es env y puede acceder como `process.env`.

Esta propiedad es un objeto que **contiene información de entorno** del usuario que ejecuta el proceso

```
{
  USER: 'doomling',
  LOGNAME: 'doomling',
  HOME: '/Users/doomling',
  SHELL: '/bin/zsh',
  PWD: '/Users/doomling/repos/twitterator',
  ZSH: '/Users/doomling/.oh-my-zsh',
  TERM_PROGRAM: 'vscode',
  LANG: 'en_US.UTF-8',
  _: '/usr/local/bin/node'
}

```

El objeto process se puede modificar asignando nuevas propiedades, por ejemplo:

```jsx
process.env.secret = "notasecret";
console.log(process.env);
```

Da como resultado:

```jsx
{
  USER: 'doomling',
  LOGNAME: 'doomling',
  HOME: '/Users/doomling',
  SHELL: '/bin/zsh',
  PWD: '/Users/doomling/repos/twitterator',
  ZSH: '/Users/doomling/.oh-my-zsh',
  TERM_PROGRAM: 'vscode',
  LANG: 'en_US.UTF-8',
  _: '/usr/local/bin/node',
	secret: 'notasecret'
}

```

esto resulta muy útil para lo que vamos a hacer a continuación, que es **mover todas las keys secretas por fuera del código para consumirlas directamente desde el proceso de node.**

Obviamente modificar el proceso desde nuestro archivo .js no tiene sentido, porque seguimos exponiendo las claves que tanto queremos ocultar. Es aquí donde **dotenv** tiene su momento de brillar. Esta librería **nos permite consumir un archivo .config** donde podemos guardar todas aquellas variables que no queremos que otros accedan y consumirlas directamente desde el objeto process.env

Para configurarla primero instalamos la dependencia:

```jsx
npm install dotenv —save

```

y creaos un archivo en la raíz de nuestro proyecto con el nombre .env. **En este archivo vivirán todas las variables que queremos consumir:**

```jsx
API_KEY = somevalue;
```

Una vez que tenemos nuestro archivo con variables creado solo queda requerir la dependencia en el código y consumirla. **Dotenv genera todos los cambios necesarios en el objeto process a través de su método config()**, \*\*\*\*consumiendo el archivo .env

```jsx
const dotenv = require("dotenv");
dotenv.config();
```

Si realizamos otro `console.log(process.env)` ahora deberíamos ver

```jsx
{
  USER: 'doomling',
  LOGNAME: 'doomling',
  HOME: '/Users/doomling',
  SHELL: '/bin/zsh',
  PWD: '/Users/doomling/repos/twitterator',
  ZSH: '/Users/doomling/.oh-my-zsh',
  TERM_PROGRAM: 'vscode',
  LANG: 'en_US.UTF-8',
  _: '/usr/local/bin/node',
	API_KEY: 'somevalue'
}

```

Finalmente en nuestro código consumimos las variables directamente desde el objeto process

```jsx
const { API_KEY } = process.env;
console.log(API_KEY); // somevalue
```

Y listo, eso es todo, **no olvides de agregar el archivo .env a tu .gitignore para evitar que termine en el repositorio remoto por error.** Es una buena práctica incluir un archivo .env de muestra por ejemplo `.env.default` que contenga el formato esperado por el proyecto siempre utilizando _placeholders_ o keys falsas ¡Obviamente!

También es importante aclarar en el .readme las instrucciones para que cada persona que clona nuestro repositorio **pueda obtener las credenciales que necesita** para ejecutar la funcionalidad de forma local.

## ¿Cómo llevamos estas variables a producción?

Quizás te estés haciendo esta pregunta, porque si nuestro repositorio no tiene información de credenciales ¿Cómo va a funcionar con servicio de deploy o integración continua? En ese caso vamos a **tener que hacer un paso extra dependiendo de donde viva nuestra aplicación** para darle información sobre que variables debería utilizar a la hora de correr o _buildear_.

Si estás utilizando instancias o servicios de deploy como DigitalOcean, Heroku o Vercel podés buscar directamente su documentación donde vas a encontrar guias detalladas sobre como crear variables de ambiente en cada servicio

Espero que este blog ayude a mantener más credenciales seguras, por cualquier duda, consulta o comentario no dudes en contactarme.
