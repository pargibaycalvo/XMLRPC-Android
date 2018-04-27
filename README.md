# XMLRPC-Android
Connection server PHP Synchronous and Asynchronous

# -Synchronous connection (Conexión síncrona)-

En esta explicación veremos como realizar una conexión a un server php de forma síncrona que contiene xmlrpc desde nuestra app de AndroidStudio. Yo seguí este tutorial https://github.com/gturri/aXMLRPC pero lo explico más detalladamente, para los que os gusta ir paso por paso.

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

# -Asynchronous connection (Conexión Asíncrona)-

Como vimos anteriormente, la conexión síncrona realiza una sola petición al server y espera a que este le responda, sin embargo la asíncrona le está mandando peticiones al server continuas así la conexión es más rápida al server.

Para realizar la conexión asíncrona mantenemos las declaraciones que hicimos anteriormente en el onCreate (URL y cliente), lo único que se va a diferenciar es en la forma de conexión ya que este usa hilos. Para empezar mantendremos el bloque try/catch y dentro de este añadimos:

         XMLRPCCallback listener = new XMLRPCCallback() { }

Una vez abramos los paréntesis se despliegan 3 métodos:

          try {
                XMLRPCCallback listener = new XMLRPCCallback() {
                    @Override
                    public void onResponse(long id, Object result) {
                        // Handling the servers response
                    }
                    @Override
                    public void onError(long id, XMLRPCException error) {
                        // Other errors
                    }
                    @Override
                    public void onServerError(long id, XMLRPCServerException error) {
                        // Errors coming from the server
                    }
                };
        } catch (Exception ex) {
            // External errors
        }

El primer método llamado onResponse es la respuesta que te manda el Server cuando se conecta con éxito.

El segundo método llamado onError devuelve una serie de errores, varios pueden ser del cliente, etc.

El tercer método llamado onServerError devuelve los errores producidos por el servidor.

Esto lo que haría sería mediante la conexión que le vamos a realizar de forma asíncrona nos permite tener varias conexiones a la vez aparte nos responderá si hay errores o no. 

Este sería el método en sí pero le falta la conexión, va situada después del "};" será justamente el cierre del listener en este caso:

                };
                client.callAsync(listener,"metodo", parametros);
                } catch (Exception ex) {
                // External errors
            }

Con esto ya tendremos casi realizada la llamada asíncrona, como se ve la función es casi igual solo que aparte de llamar al método del .php y sus parámetros también se llama al listener para que ejecute ese hilo. 

Tendría que quedar algo así:

       try {
                XMLRPCCallback listener = new XMLRPCCallback() {
                    @Override
                    public void onResponse(long id, Object result) {
                        // Handling the servers response
                    }

                    @Override
                    public void onError(long id, XMLRPCException error) {
                        // Other errors
                    }
                    @Override
                    public void onServerError(long id, XMLRPCServerException error) {
                        // Errors coming from the server
                    }
                };
                client.callAsync(listener,"metodo", parametros);

        } catch (Exception ex) {
            // External errors
        }

Si realizamos la llamada como el caso de la síncrona con su método y sus parámetros no deberái funcionar, el problema está en, que los parámetros tienen que ir en un Array de Object como expliqué en la síncrona:

        client.callAsync(listener,"login", new Object[]{usuario, contrasenha});
        
Y nos quedaría lo más importante, si os fijais cada método del hilo listener tiene un "long id" este id es generado automáticamente por el programa para diferenciar quien entra en la app. Pero hay que decirle que cliente entra y con que id, pues lo realizamos del siguiente modo, declaramos una variable en privado tipo long y la llamamos "iCon" y la inicializamos en 0, lo de private como bien sabréis es para que se pueda usar en toda la clase:

        private long iConexion = 0;
        
Ahora en donde realizamos la conexión del cliente le decimos que iCon es igual a esa conexión:
        
        iCon = client.callAsync(listener,"login", new Object[]{usuario, contrasenha});
     
Resumiendo, cuando te conectes esa id quedará almacenada en iCon y la usaremos para los métodos anteriores.

Para la utilización de los 3 métodos, como bien se explicaron 2 son para errores y otro es para la conexión, bien, en los métodos de errores inicializaremos en 0 el iCon, así cuando salte el catch este volverá a 0 y no guardará la id anterior, evitaremos posibles problemas si queremos conectarnos otra vez:

         Log.e("ID","Se han producido varios errores"+error.getMessage());
         iCon=0;
         
Los Toast no me funcionaron en mi proyecto con lo cual utilicé Log.e para ver que clase de errores da al ahcer Debug.

En el onResponse si queremos ver si funciona, saltando a otra MainActivity podemos ponerlo tal que así:

          @Override
          public void onResponse(long id, Object result) {
              // Handling the servers response
              if(iCon==id) {
                //función par abrir la otra MainActivity
                iConexion=0;
              }else{
                Log.e("ID","Se han producido varios errores");
              }
           }

Como veis hago la comparación del iCon con el id de la función si son iguales que abra la otra ventana. Podeis añadirle métodos o lo que os pida vuestra app.

# Comentarios

Esto lo estoy aun realizando en un proyecto puede variar a medida que voy avanzando, cualquier añadido por vuestra parte por muy menor que sea, estamos aquí para aprender.

Un saludo.

