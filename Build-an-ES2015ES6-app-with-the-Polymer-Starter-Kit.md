# Construye una aplicación ES2015 con Polymer Starter Kit

## Requisitos previos

  Para la realización de este tutorial vamos a usar los siguientes componentes:
  1. [Download and install the Polymer Starter Kit](https://github.com/PolymerElements/polymer-starter-kit/archive/v1.3.0.zip)

  2. Necesitaras lanzar `npm install && bower install` para instalar las dependencias del proyecto.

## Añadiendo soporte para ES2015
  Para beneficiarnos de las maravillas del ES6 necesitaremos 'transpilar' nuestro JS de ES2015 a ES6, y para ello usaremos [Babel](https://babeljs.io/)

  ### Crear una tarea en gulp que 'transpile'
  Install the gulp Babel, Sourcemap and Crisper plugins:

  ```
  npm install --save-dev gulp-babel gulp-sourcemaps gulp-crisper babel-preset-es2015
  ```

  Añadimos la tarea en ´gulpfile.js´ detrás de la tarea de vulcanize:

  ```
  // Transpile all JS to ES5.
  gulp.task('js', function () {
   return gulp.src(['app/**/*.{js,html}', '!app/bower_components/**/*'])
     .pipe($.if('*.html', $.crisper({scriptInHead:false}))) // Extract JS from .html files
     .pipe($.sourcemaps.init())
     .pipe($.if('*.js', $.babel({
       presets: ['es2015']
     })))
     .pipe($.sourcemaps.write())
     .pipe(gulp.dest('.tmp/'))
     .pipe(gulp.dest('dist/'));
  });
  ```
  Esta tarea transpilará todos los archivos JS y el JS dentro del HTML, generando un sourcemap. Los archivos resultantes serán generados en el directorio .tmp

  > [Crisper](https://github.com/PolymerLabs/crisper) extrae todo el JS que esté dentro del HTML en un único y separado archivo. Lo necesitamos ya que Babel no da soporte a la transpilación del JS dentro de los archivos HTML.

  >[Sourcemap](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/) son archivos que son genrados por herramientas que transforman el código fuente (e.g. minificadores, transpiladores). Estos pueden ser usados para trasnformar el código a su formato original.

  ### Hacer que Vulcanice entienda nuestro código transpilado

  Actualizamos la tarea de vulcanize para que use el código transpilado in .tmp, en lugar del que esta en el directorio app

  ```
  // Vulcanize granular configuration
  gulp.task('vulcanize', function() {
    return gulp.src('.tmp/elements/elements.html') // look in .tmp dir!
      .pipe($.vulcanize({
        stripComments: true,
        inlineCss: true,
        inlineScripts: true
      }))
      .pipe(gulp.dest(dist('elements')))
      .pipe($.size({title: 'vulcanize'}));
  });
  ```


  Para que la tarea vulcanize busque correctamente todas nuestras dependencias, tendremos que modificar la tarea de copia también para que coloque los componentes potencialmente referenciados en el directorio .tmp. La tarea de copia ya contiene las subtareas app y bower. Deja los que están, y agrega una tercera subtarea llamada tmp.


  ```
  // Copy all files at the root level (app)
  gulp.task('copy', function() {

    var app = ...

    var bower = ...

    // Add components to .tmp dir so they can get concatenated
    // when we vulcanize
    var tmp = gulp.src(['app/bower_components/**/*'])
      .pipe(gulp.dest('.tmp/bower_components'));

    return merge(app, bower, tmp)
      .pipe($.size({
        title: 'copy'
      }));

  });
  ```

  ### Integrando la tarea transpiladora

  Necesitamos asegurarnos de que la tarea js de gulp es lanzada por las tareas comunes de construcción.

  En primer lugar aseguremonos de que la tarea js es lanzada inicialmente y en los cambios de archivos HTML y JS:

  ```
  gulp.task('serve', ['styles', 'js'], function () { // Added 'js' here!

    ...

    gulp.watch(['app/**/*.html', '!app/bower_components/**/*.html'], ['js', reload]); // Added 'js' here!
    gulp.watch(['app/styles/**/*.css'], ['styles', reload]);
    gulp.watch(['app/scripts/**/*.js'], ['js', reload]); // Added 'js' here!
    gulp.watch(['app/images/**/*'], reload);
  });
  ```

  Esto nos asegura que la tarea de js es lanzada siempre que modifiquemos un archivo HTML o JS mientras esté corriendo el servidor de gulp para que el código sea transpilado y tu app se refresque cuando estemos desarrollando.

  Entonces, en la tarea por defecto nos aseguramos que la tarea que trata los js corra en paralelo a la que trata las  imagenes, fuentes y html

  ```
  gulp.task('default', ['clean'], function (cb) {
    // Uncomment 'cache-config' if you are going to use service workers.
    runSequence(
      ['ensureFiles', 'copy', 'styles'],
      ['images', 'fonts', 'html', 'js'],
      'vulcanize', // 'cache-config',
      cb);
  });
  ```

  La tarea html se encarga de tener preparado el código HTML, CSS y JS para producción, minificandolo y concatenando archivos. Tenemos que asegurarnos de que la versión transpilada de los archivos JS se están usando en lugar de la original hecha en ES2015. Para hacer esto elimina "app" del "useref" en la tarea que trata el html:

  ```
  // Scan Your HTML For Assets & Optimize Them
  gulp.task('html', function () {
    var assets = $.useref.assets({searchPath: ['.tmp', 'dist']}); // Removed 'app' here!

    ...

  });
  ```

  ###Comprovemos que todo funciona correctamente

  Lanza gulp para  empezar una construcción completa de la aplicación

  Ahora asegurate que el directorio .tmp ha sido creado y contiene, almenos, estos archivos en elements/my-list/ and scripts/:

  ```
  polymer-starter-kit-1.3.0/
    .tmp/  <!-- The temporary build folder -->
      elements/  <!-- Folder containing custom Polymer elements -->
        my-list/  <!-- A sample custom Polymer element's folder -->
          my-list.html  <!-- Template of the custom 'my-list' element -->
          my-list.js  <!-- JS code separated from the HTML by Crisper -->
      ...
      scripts/  <!-- Folder containing non-Polymer-elements JS code -->
        app.js  <!-- A JS file -->
    ...
  ```

  Esto nos muestra que nuestra nueva tarea de creación de js ha estado funcionando al separar el código JavaScript del HTML de los elementos Polymer. Si echas un vistazo a my-list.js deberías ver un gran bloque de código comentado en la parte inferior que comienza con sourceMappingURL. Ése es el sourcemap, compilado en el JavaScript de nuestro elemento.

  ¡Ala, campeón! A disfrutar escribiendo ES2015 en tu aplicación.

###Creación de un elemento Polymer con ES2015

  Vamos a ver como crear un elemento Polymer usando la sintaxis ES2015.

  Crea el siguiente directorio: app/elements/awesome-meme/

  En este directorio crea un archivo llamado awesome-meme.html el cual contendrá el código de tu elemento.

  Necesitaremos una imagen de fondo. Guarda una imagen en app/images y nombrala awesome.png

  Para crear nuestro elemento Polymer “awesome-meme” necesitamos definir un <dom-module> que contendrá la etiquetas ```<style>```, ```<template>``` and ```<script>```. Añade esto a awesome-meme.html:

  ```
  <link rel="import" href="../../bower_components/polymer/polymer.html">


  <dom-module id="awesome-meme">
    <template>
      <style>
        /* TODO: add CSS */
      </style>

      <!-- TODO: add the template's HTML content -->
    </template>

    <script>
      /* TODO: create the Polymer element's definition in ES2015 */
    </script>
  </dom-module>
  ```

  El  Meme simplemente consistirá en dos configurables bloques de texto sobre uuna imagen de fondo. Añade esto dentro de la etiqueta ```<template>```:

  ```
  <div class="top">{{top}}</div>
  <div class="bottom">{{bottom}}</div>
  ```

  Los dos bloques de texto estarán posicionados de forma absoluta. Añade esto en la etiqueta ```<style>```:

  ```
  :host {
    display: block;
    background-image: url("/images/awesome.png");
    width: 300px;
    height: 300px;
    position: relative;
  }
  .top, .bottom{
    position: absolute;
    font-size: 30px;
    font-weight: bold;
    width: 100%;
    text-align: center;
  }
  .top {
    top: 20px;
  }
  .bottom {
    bottom: 20px;
  }
  ```

  Lo siguiente será definir el elemtno Polymer usando una definición de clase en ES2015. Será muy simple, tendrá las propiedades top y bottom y las pondrá en mayusculas. Añad esto a la etiqueta ```<script>```:

  ```
  class AwesomeMeme {
    beforeRegister() {
      this.is = 'awesome-meme';
      this.properties = {
        top: {
          type: String,
          value: ''
        },
        bottom: {
          type: String,
          value: ''
        }
      };
    }
    created() {}
    ready() {
      this.top = this.top.toUpperCase();
      this.bottom = this.bottom.toUpperCase();
    }
    attached() {}
    detached() {}
    attributeChanged() {}
  }

  Polymer(AwesomeMeme);
  ```

  Hemos listado todos los callback de Polymer simplemente por fines didácticos pero realmente los podemos eliminar.

  Usando clases para definir los elementos Polymer nos beneficiamos de una mejor rehusabilidad y más fácil mantenimiento además de poder configurar más facilmente las herencias.

  Opcionalmente puedes actualizar el código JavaScript de los siguientes elementos Polymer a ES2015:

  ```
  app/elements/my-greeting/my-greeting.html
  app/elements/my-list/my-list.html
  ```

  ###Let´s Rock!!

  Ahora podemos uasar y configurar el nuevo elemento Polymer en nuestra app. Primero, importamos el elemento en nuestra página principal. Añade la siguiente línea al archivo ```app/elements/elements.html```:

  ```
  ...

  <link rel="import" href="../styles/app-theme.html">
  <link rel="import" href="my-greeting/my-greeting.html">
  <link rel="import" href="my-list/my-list.html">
  <!-- Added the line below -->
  <link rel="import" href="awesome-meme/awesome-meme.html">
  ```

  Vamos a añadir el Meme a nuestra aplicación. Añade la siguiente línea al archivo app/index.html

  ```
  ...

  <paper-material elevation="1">
    <p class="paper-font-body2">This is another card.</p>
    <!-- Added the line below -->
    <awesome-meme top="ES2015" bottom="is awesome!"></awesome-meme>
  </paper-material>

  ...
  ```

  Ahora ya puedes lanzar gulp serve y comprobar el resultado.
