## Red Social
Primero iniciamos creando el proyecto con el comando ```rails new social-red --database=postgresql```.

Una vez creado el proyecto pasaremos a crear y migrar la base de datos que usara nuestra aplicación ```rails db:create db:migrate```.

Para verificar que todo funciono ejecutamos el servicio ```rails s```

### Publication
Ahora pasaremos a agregar las publicaciones que se podran hacer en nuestra red social

**Crear y migrar modelo**

*   ```rails g model publication description like:integer view:integer```: Con este comando generaremos el modelo base que vamos a usar para las publicaciones, posteriormente se modificara conforme avancemos. El nombre del modelo debe de estar en singular, esto es parte de las convenciones de rails.

*   ```rails db:migrate```: Pasaremos a migrar el modelo a nuestra base de datos.

*   ```rails db```: Para acceder a la base de datos existente en el motor de postgresql ```\d``` para listar las tablas en la base de datos.

**Rutas**

Pasaremos a agregar las rutas y a crear una vista preliminar para verificar que este funcionando bien.

*   En el archivo **routes.rb** es donde se pondrar las rutas de nuestra aplicación.

**routes.rb**
```rb
Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
  resources :publications
end
```

*   ```rails routes```: Nos servira para poder ver las rutas que existen en nuestra aplicación.

*   ```rails g controller Publications```: El controlador generado debe de estar en plural con respecto al nombre del modelo al cual se va a hacer la referencia.

*   Nos dirigimos a nuestro controlador recien creado ```publications_controller.rb``` por convencion el nombre lleva agregado a el ***_controller***. En el controlador definiremos el acceso de la ruta, por que aunque ya definimos las rutas aun no hemos especificado el acceso, el acceso a la ruta consta de dos partes, el **controlador** y la **vista**.

**publications_controller.rb**
```rb
class PublicationsController < ApplicationController
    def index
    end
end
```

*   Ya definimos el acceso en el controlador, ahora falta definir la vista, en el directorio **app/views** veremos que exista una folder llamado **publications** cuando generamos nuestro controlador tambien fue generado este folder, en el pondremos las vistas de nuestro controlador **publications_controller**, las vistas en rails tienen la extension **.html.erb**.

**index.html.erb**
```html
Hola mundo :D
```
*   Ahora para poder verificar que todo funciona bien ejecutaremos el servidor ```rails s``` y accederemos a la ruta ```http://localhost:3000/publications```.

## CRUD - Mantenimiento

Ahora se desarrollara el CRUD o mantenimiento para nuestro modelo, iniciaremos por **crear** información.

### Create
* Primero definiremos la ruta para **new**

**publications_controller.rb**
```rb
class PublicationsController < ApplicationController
    def new
    end
end
```
* Ya tenemos el acceso a la ruta **publications/new**, pero nos falta la vista para dicha ruta, en esta estara nuestro formulario asi que lo agregamos, para el formulario se usara **form_helper**.

**new.html.erb**
```html
<h3>Publicar</h3>

<%= form_for :publication, url: publications_path do |f|  %>
    <%= f.label :description %> <br>
    <%= f.text_field :description %>
    <br>
    <%= f.label :like %> <br>
    <%= f.number_field :like %>
    <br>
    <%= f.label :view %> <br>
    <%= f.number_field :view %>
    <br>
    <%= f.submit %>
<% end %>
```
**Nota:** Podras encontrar mas información sobre el uso de form_helper [aqui](https://guides.rubyonrails.org/form_helpers.html).

* Ya tenemos el formulario de la información que vamos a enviar, ahora definiremos la accion **create** para guardar la información.

**publications_controller.rb**
```rb
class PublicationsController < ApplicationController
    def new
    end
    def create
        @publication = Publication.new 
        @publication.save
    end
end
```

* Ahora si corremos el servicio y probamos el funcionamiento no tendremos ningun error, pero si checamos la información en nuestra base de datos no mostrara nada, la información que se guardo esta vacia, esto se debe a que no le hemos dado permiso para guardar la información recibida, para eso definiremos una variable como parametro de la información recibida que podra guardar y la agregamos cuando se crea la nueva publicación.

**publications_controller.rb**
```rb
class PublicationsController < ApplicationController
    def new
    end
    def create
        @publication = Publication.new publication_params #Se almacenan los parametros enviados
        @publication.save
    end
    private #Para definir los parametros requeridos/permitidos
    def publication_params
        params.require(:publication).permit :description, :like, :view
    end
end
```

* ¡Listo!, probamos que la información se guarde.

* Tambien se puede definir valores determinados para la información que se guardara. 

**publications_controller.rb**
```rb
class PublicationsController < ApplicationController
    def new
    end
    def create
        @publication = Publication.new publication_params #Se almacenan los parametros enviados
        @publication.like = 0
        @publication.view = 0
        @publication.save #Se guardan los parametros enviados
        redirect_to @publication
    end
    private #Para definir los parametros requeridos/permitidos
    def publication_params
        params.require(:publication).permit :description, :like, :view
    end
end
```

**new.html.erb**
```html
<h3>Publicar</h3>

<%= form_for :publication, url: publications_path do |f|  %>
    <%= f.label :description %> <br>
    <%= f.text_field :description %>
    <br>
    <%= f.submit %>
<% end %>
```
### READ
Ahora pasaremos a leer la información
* Definimos la ruta **publications/id_dato** esto se define en el controlador como **show**

**publications_controller.rb**
```rb
    def show
        @publication = Publication.find params[:id] #Hace una busqueda del dato, por su id para mostrarlo
    end
```

* Agregamos la vista para dicha ruta
**show.html.erb**
```html
<%= @publication.description %>
<br>
<%= @publication.like %>
<%= @publication.view %>
```

* Ahora podemos acceder a dicha información ejemplo seria **localhost:3000/publications/1**, pero vamos a direccionar la creacion del dato a la vista para dicho dato.

**publications_controller.rb**
```rb
    def create
        @publication = Publication.new publication_params #Se almacenan los parametros enviados
        @publication.like = 0
        @publication.view = 0
        @publication.save #Se guardan los parametros enviados
        redirect_to @publication
    end
```

* Ahora crearemos una vista para ver todos los elementos, definimos entonces dicha vista, la pondremos en index que seria la ruta **/publications**.

**publications_controller.rb
```rb
    def index
        @publications = Publication.all #Hace un pedido de todos los datos.
    end
```

* La vista para mostrar toda esa información

**index.html.erb**
```html
<% @publications.each do |publication| %> <!-- un ciclo para leer toda la información -->
    <%= publication.description %>
    <br>
    <%= publication.like %>
    <%= publication.view %>
    <br>
<% end %>
```

* !Listo¡, ahora probamos el funcionamiento.

### UPDATE
Ahora desarrollaremos el metodo de Update para las publicaciones:

* Agregamos un link que nos enviara a la edicion de la información.

**index.html.erb**
```html
<%= link_to 'Edit', edit_publication_path(publication) %><br>
```

* Definimos edit en el controlador y el metodo update para que pueda actualizarse
**publications_controller.rb**
```rb
    def edit
        @publication = Publication.find params[:id]
    end
    def update
        @publication = Publication.find params[:id]
        @publication.update publication_params
        redirect_to @publication
    end
```

* Ahora creamos el archivo que sera la vista con el formulario para editar.

**edit.html.erb**
```html
<h3>Editar</h3>
<%= form_for @publication do |f|  %>
    <%= f.label :description %> <br>
    <%= f.text_field :description %>
    <br>
    <%= f.submit %>
<% end %>
```

### Destroy
Ahora pasaremos a crear el metodo para destruir/eliminar una publicacion

* Para destruir una publicación primero agregamos el link que llamara al metodo, agregamos una confirmación para llamar al metodo.

**index.html.erb**
```html
<% @publications.each do |publication| %> <!-- un ciclo para leer toda la información -->
    <%= publication.description %>
    <br>
    <%= publication.like %>
    <%= publication.view %>
    <br>
    <%= link_to 'Edit', edit_publication_path(publication) %>
    <%= link_to 'Destroy', publication, method: :delete, data: { confirm: 'sure?'} %> <br>
<% end %>
```
* Agregamos el metodo **destroy** al controlador.

**publications_controller.rb**
```rb
    def destroy
        @publication.destroy
        redirect_to publications_path
    end
```
## Parciales
Una buena practica para el desarrollo en Ruby on Rails son la creacion de parciales, puedes verlos como modulos para la vista.
Esto nos servira para evitar el codigo repetido en las vistas. Ejemplo para este proyecto son la vista de la información de las publicaciones y el formulario para crear y actualizar la información.

*   Bien ahora pasaremos a crear un parcial, primero para la vista de la información, el nombre de los parciales inician con un "_", en la carpeta de vistas del controlador creamos el parcial.

**_publication.html.erb**
```html
<%= publication.description %>
<br>
<%= publication.like %>
<%= publication.view %>
<br>
<%= link_to 'Edit', edit_publication_path(publication) %>
<%= link_to 'Destroy', publication, method: :delete, data: { confirm: 'sure?'} %> <br>
```
*   Llamaremos el parcial desde index.html, esta forma es solo aplicable cuando el nombre del parcial es el singular del nombre de la vista.

**index.html.erb**
```html
<%= render @publications %>
```

*   Ahora llamaremos al parcial en la vista show, esta sera de la forma general en como se llama un parcial, entre '' ira el nombre del parcial, seguido del nombre de la variable dentro del parcial y el valor que le pasaremos a dicha variable.

**show.html.erb**
```html
<%= render 'publication', publication: @publication %>
```

Con eso deberia estar listo el parcial de nuestras vistas, ahora pasaremos a agregar el parcial para el formulario.

*   Agregamos el parcial **form** que contendra el formulario, copiamos el contenido de la vista **edit.html.erb** en este parcial.

**_form.html.erb**
```html
<%= form_for @publication do |f|  %>
    <%= f.label :description %> <br>
    <%= f.text_field :description %>
    <br>
    <%= f.submit %>
<% end %>
```

*   Y ahora pasamos a llamar el parcial en la vista edit.

**edit.html.erb**
```html
<h3>Editar</h3>

<%= render 'form' %>
```

*   En el caso de la vista en new no podemos simplemente llamarlo ya que hay algo que nos falta y es la variable **@publication**, asi que no iremos al **publications_controlador.rb** y en la definicion **new** agregaremos esta variable faltante inicializandola como un contenido vacio con la estructura de nuestra tabla.

**publications_controlador.rb**
```rb
    def new
        @publication = Publication.new
    end
```

*   Ahora pasamos a llamar el parcial desde la vista new.

**new.html.erb**
```html
<h3>Publicar</h3>

<%= render 'form' %>
```

*   Y ahora guardamos y corremos el servicio para verificar que todo vaya bien.