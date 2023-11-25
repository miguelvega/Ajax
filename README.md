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
