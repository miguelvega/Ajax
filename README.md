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

Agregamos la siguiente linea de código para mostrar la acción del controlador que renderizara una vista parcial llamada 'movie'   y pasa la variable @movie a esa vista que contiene la información de la película, siempre y cuando sea una solicitud AJAX.

```ruby
render(:partial => 'movie', :object => @movie) if request.xhr?
```

De tal manera que nuestra acción show del controlador estará diseñada para manejar tanto solicitudes normales como solicitudes AJAX.
Esta modificacion de la acción show está respondiendo a una petición AJAX, esto procesará la sencilla vista parcial del siguiente codigo en lugar de la vista completa.

```ruby
 <p> <%= movie.description %> </p>
 <%= link_to 'Edit Movie', edit_movie_path(movie), :class => 'btn btn-primary' %>
 <%= link_to 'Close', '', :id => 'closeLink', :class => 'btn btn-secondary' %>
```
El codigo anterior estara contenido en el archivo `_movie.html.erb`, ya que segun la convención en Rails para vistas parciales debemos comenzar el nombre del archivo con un guion bajo (_) seguido del nombre de la vista y estara ubicado en el siguiente directorio app/views/movies.
`.
## Parte 2
