# XMLRPC-Android
Connection server PHP

En esta explicación veremos como realizar una conexión a un server php de forma síncrona que contiene xmlrpc desde nuestra app de AndroidStudio.

1º -> Añadiremos la librería (aXMLRPC) en el bulid.gradle(Module:app) y sincronizamos nuestra app:

    implementation 'fr.turri:aXMLRPC:1.9.0'

2º  -> En el AndroidManifest.xml añadimos los permisos de Internet para que la app pueda acceder:

    <uses-permission android:name="android.permission.INTERNET" />
    
Una vez tenemos todo esto nuestra app estará lista para utilizar la librería aXMLRPC y su conexión a internet.

Para empezar a realizar nuestra conexión, tenemos que saber donde tenemos ubicado nuestro script .php en nuestro equipo, o de un compañero, o bien en un servidor. Si tenemos el script alojado en carpetas recordad que tenéis que poner toda la ruta.

-> http://url/archivos/server/pruebaserver.php

Antes de empezar a picar código probaremos que desde nuestro equipo y mediante un buscador (Chrome, Mozilla, etc) que tenemos acceso sin problemas a ese script .php y nos muestra el contenido correcto y sin problemas.

En el onCreate.

1º -> Añadimos la URL

        // URL server php
        try {
            url = new URL("http://url/pruebaserver.php");
        } catch (java.net.MalformedURLException ex) {
            Toast.makeText(MainActivity.this, "Error de URL: " + ex.getMessage(), Toast.LENGTH_LONG);
        }
Con este bloque try/catch obtenemos la URL si es correcta, si no obtendremos un error.

2º -> Crearemos un cliente donde le añadiremos la URL declarada anteriormente:

        // Client that collects the URL of the php server
        client = new XMLRPCClient(url);

Con esto ya tenemos el cliente creado con la URL del server donde se conectará.

Ahora explicaré como funciona el método para la conexión. 
Para realizar la conexión la librería de aXMLRPC que añadimos anteriormente cuenta con una función llamada (.call) que realiza la llamada desde el cliente al método script .php de forma síncrona, si os fijáis cuenta también con una función similar la (.callAsync) que es para realizar la llamada de forma asíncrona que lo explicaré más adelante.

Para realizar la conexión de forma síncrona:

        client.call("login");
        
Aquí se ve como el cliente realiza una llamada al método del script.php, en este caso es "login" que aprobará nuestra conexión, todo depende de como tengáis vuestro script configurado.
Si vuestro método del .php cuenta con parámetros de login (usuario o contraseña o ambos) tenemos que añadírselos a la función .call. 
No hace falta que se llamen de igual manera que en el .php con tal de saber sin son String, Int, etc:

        client.call("login", usuario, contrasenha);

Ahora tenemos el método añadido con sus parámetros, que desde nuestra app le daremos los valores deseados para que nos deje realizar la conexión, se puede realizar un Array de String u Object dentro de la función:

        client.call("examples.login", new String[]{usuario, contrasenha});
        client.call("examples.login", new Object[]{usuario, contrasenha});
        
 Esto todo nos va a decir que necesita un bloque try/catch y nos tiene que quedar algo así:

        try {
            client.call("login", usuario, contrasenha);
            Toast.makeText(MainActivity.this, "Conectado ", Toast.LENGTH_LONG).show();

        } catch (XMLRPCServerException ex) {
            // Errors coming from the server
            String errorS = "Errores Servidor: " + ex.getMessage();
            Toast.makeText(MainActivity.this, errorS, Toast.LENGTH_LONG).show();

        } catch (XMLRPCException ex) {
            // Errors coming from the client
            String errorC = "Errores Cliente: " + ex.getMessage();
            Toast.makeText(MainActivity.this, errorC, Toast.LENGTH_LONG).show();
        }
        
Estos catch son esenciales ya que uno nos va a informar de posibles errores en el servidor y el otro del cliente en este caso seríamos nosotros. Con esto deberíamos poder conectarnos al script .php de forma síncrona. 

OJO! Esto una vez te conectes puede tardar un poco en realizar la conexión, ya que la conexión de forma síncrona el cliente hace una sola llamada al server y espera su respuesta.
