# Ajax

En un comienzo al iniciar el servidor web proporcionado por la aplicación Rails con el comando `rails server` se obtiene el mensaje de error `ActiveRecord::PendingMigrationError` que indica que hay una migración pendiente que debe ejecutarse, resolvemos este problema ejecutando el comando `bin/rails db:migrate RAILS_ENV=development
` en la terminal, luego analizando los archivos de nuestro proyecto vemos que tenemos unos errores de sintaxis en el archivo `movie.rb`, con lo cual comemtamos las lineas de codigo que estan ocasionando dichos problemas.

Luego, analizando el archivo application_controller.rb nos damos con la siguiente linea de codigo :

```ruby
  @current_user ||= Moviegoer.where(:id => session[:user_id])

```
Con lo cual el controlador ApplicationController está intentando acceder a la tabla 'moviegoers', sin embargo al ver el contenido del archivo schema.rb, podemos apreciar que tenemos  creado las tablas 'movies' y 'reviews', pero no se menciona la tabla 'moviegoers'. Ademas al dirigirnos al archivo seeds.rb vemos que se está creando instancias del modelo Movie y almacenándolas en la tabla movies de la base de datos con los datos proporcionados en el array more_movies. 

![1](https://github.com/miguelvega/Ajax/assets/124398378/df2450d6-b79b-45bd-ba7a-d28b46f41e7b)

Por lo cual, para que la aplicacion funcione correctamente tenemos que modificar la linea de codigo anterior, quedando el archivo application_controller.rb de la siguiente manera:


```ruby
class ApplicationController < ActionController::Base
    before_action :set_current_user  # change before_filter
    protected # prevents method from being invoked by a route
    def set_current_user
        # we exploit the fact that the below query may return nil
        @current_user ||= Movie.where(:id => session[:user_id])
        redirect_to login_path and return unless @current_user
    end
end

```
Ejecutamos el comando `rails server` nuevamente, nos dirigirnos a nuestro navegador y colocamos http://127.0.0.1:3000. Con lo cual, ahora si podemos ver la tabla que muestra información sobre películas.

![2](https://github.com/miguelvega/Ajax/assets/124398378/c5c695a0-6271-4154-b648-6d5bf6d200ee)


Sin embargo, cuando queremos editar una pelicula nos indica que estamos intentando acceder a la acción edit del controlador MoviesController, pero Rails no puede encontrar la plantilla correspondiente para renderizar en el formato text/html. Por lo tanto, agragamos la vista denominandola edit.html.erb, pero al actualizar la informacion de la pelicula en cuestion nos muestra un error.


## Parte 1

Incorporamos la siguiente línea de código con el propósito de exhibir la acción del controlador responsable de renderizar una vista parcial denominada 'movie'. Además, se transmite la variable @movie a dicha vista, la cual contiene la información de la película. Esto se ejecutará en caso de que la solicitud sea de tipo AJAX, procesando así una vista parcial sencilla en lugar de la vista completa.

```ruby
render(:partial => 'movie', :object => @movie) if request.xhr?
```
Con esta modificacion de nuestra acción 'show' del controlador estará diseñada de manera que puede gestionar tanto solicitudes normales como solicitudes AJAX.

```ruby
def show
    id = params[:id] # retrieve movie ID from URI route
    @movie = Movie.find(id) # look up movie by unique ID
    render(:partial => 'movie', :object => @movie) if request.xhr?
    # will render app/views/movies/show.<extension> by default
end

```
Segun la convención en Rails para vistas parciales debemos comenzar el nombre del archivo con un guion bajo (_) seguido del nombre de la vista, es decir nuestro archivo de vista parcial se denominara `_movie.html.erb` y estara ubicado en el directorio app/views/movies de nuestro proyecto y contendra la siguientes lineas de codigo :

```ruby
 <p> <%= movie.description %> </p>
 <%= link_to 'Edit Movie', edit_movie_path(movie), :class => 'btn btn-primary' %>
 <%= link_to 'Close', '', :id => 'closeLink', :class => 'btn btn-secondary' %>
```
Entonces, si la solicitud es una solicitud AJAX, este código en la vista parcial 'movie' se renderizará y se enviará al cliente. La vista parcial mostrará la descripción de la película. 
Estos elementos formarán la respuesta que se envía al cliente cuando se realiza una solicitud AJAX para la acción show del controlador
## Parte 2

### ¿Cómo debería construir y lanzar la petición XHR el código JavaScript?

Primero debemos crear un archivo javascript  movie_popup.js

```ruby

var MoviePopup = {
  setup: function() {
    // add hidden 'div' to end of page to display popup:
    let popupDiv = $('<div id="movieInfo"></div>');
    popupDiv.hide().appendTo($('body'));
    $(document).on('click', '#movies a', MoviePopup.getMovieInfo);
  }
  ,getMovieInfo: function() {
    $.ajax({type: 'GET',
            url: $(this).attr('href'),
            timeout: 5000,
            success: MoviePopup.showMovieInfo,
            error: function(xhrObj, textStatus, exception) { alert('Error!'); }
            // 'success' and 'error' functions will be passed 3 args
           });
    return(false);
  }
  ,showMovieInfo: function(data, requestStatus, xhrObject) {
    // center a floater 1/2 as wide and 1/4 as tall as screen
    let oneFourth = Math.ceil($(window).width() / 4);
    $('#movieInfo').
      css({'left': oneFourth,  'width': 2*oneFourth, 'top': 250}).
      html(data).
      show();
    // make the Close link in the hidden element work
    $('#closeLink').click(MoviePopup.hideMovieInfo);
    return(false);  // prevent default link action
  }
  ,hideMovieInfo: function() {
    $('#movieInfo').hide();
    return(false);
  }
};
$(MoviePopup.setup);

```
Este código en JavaScript está diseñado para mostrar información detallada sobre una película en una ventana emergente (popup) cuando el usuario hace clic en un enlace asociado a esa película.

Primero se crea un objeto llamado MoviePopup que contiene cuatro funciones (setup, getMovieInfo, showMovieInfo y hideMovieInfo).

La funcion setup se encarga de la configuración inicial. 
Creamos un nuevo elemento de tipo div en jQuery y lo asigna a la variable popupDiv. Este nuevo div tiene el ID "movieInfo" donde la notación $('<div></div>') en jQuery se utiliza para crear un nuevo elemento HTML en memoria, este div creado se utilizará para contener y mostrar la información detallada de la película en una ventana emergente. El ID "movieInfo" se usa para referenciar este elemento más adelante en el código, por ejemplo, para establecer su contenido y estilo.
Luego, `popupDiv.hide()` oculta el elemento div. Esto significa que, inicialmente, la ventana emergente está configurada para no ser visible cuando se crea y `.appendTo($('body'))` adjunta el elemento div al final del cuerpo del documento ($('body')). Esto significa que el div oculto se agrega al final del cuerpo HTML en el DOM. Con esto agregamos el elemento al cuerpo y se prepara para su posterior visualización y manipulación. 
La última línea dentro de la función setup tenemos  $(document), con esto seleccionamos el objeto document, que representa todo el documento HTML y `.on('click', '#movies a', MoviePopup.getMovieInfo)` establece un manejador de eventos en el documento (document). Los parámetros de esta función `.on` son los siguientes:
'click': Especifica que estamos manejando el evento de clic.
'#movies a': Selecciona todos los elementos de anclaje (a) que son descendientes de elementos con el ID movies.
MoviePopup.getMovieInfo: Es la función que se ejecutará cuando se haga clic en uno de esos enlaces.

La función getMovieInfo se encarga de recuperar información detallada sobre una película. Para lograrlo, realiza una solicitud AJAX ($.ajax) de tipo GET a la URL especificada en el atributo href del enlace que fue clicado ($(this).attr('href')). Se establece un límite de tiempo de 5 segundos (timeout: 5000). En caso de que la solicitud sea exitosa, la función MoviePopup.showMovieInfo se invoca con los datos recibidos. En caso de un error en la solicitud, se muestra una alerta. La función retorna false con el fin de prevenir la acción predeterminada del enlace.

La función showMovieInfo despliega información detallada de la película en una ventana emergente. Para lograrlo, centra la ventana en la pantalla, ajustando sus propiedades de posición y tamaño. Luego, llena la ventana con el contenido HTML obtenido como respuesta de la solicitud AJAX (data), y la presenta visualmente mediante $('#movieInfo').show(). Además, se vincula un evento de clic al enlace de cierre (#closeLink), el cual invoca la función MoviePopup.hideMovieInfo. Finalmente, la función retorna false para evitar la ejecución de la acción predeterminada del enlace.



