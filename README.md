
# Introducción a Django

Primeros pasos con Django utilizando el entorno de desarrollo GoormIDE, implementando los tutoriales básicos de la documentación de Django.

Parte 1: Peticiones y respuestas 

## Entorno de desarrollo: GoormIDE
https://ide.goorm.io/

- Crea una cuenta (si no aún no dispones de ella)

- Crea un contenedor con Django

Completa el formulario con los valores deseados, por ejemplo:
    
    - Name: intro
    - Visibility: Private
    - Template: Default Template
    - Deployment: Heroku (crea o selecciona la aplicación en Heroku, si deseas poder desplegarlo en producción en este plataforma)
    - Stack: Django
    - Additional module/package: Install MongoDB

- Ejecuta el contenedor

Si has utilizado el Stack Django, Ya tendrás instalado Python, Django y, además, tendrás creado un proyecto sobre el que podrás comenzar a trabajar.

En el terminal, puedes comprobar la versión de Python:

```
$ python --version
Python 3.7.4
```

La versión de Django:

```
$ python -m django --version
2.2.4
```
Además, recuerda que ya dispones de un proyecto Django creado, por lo tanto, NO es necesario ejecutar el comando para crear el proyecto:

```
\\ NO NECESARIO CON EL STACK Django de GoormIDE
$ django-admin startproject intro

```

Con la siguiente estructura:

```
`-- intro
    |-- README.md
    |-- db.sqlite3
    |-- goorm.manifest
    |-- intro
    |   |-- __init__.py
    |   |-- __pycache__
    |   |-- settings.py
    |   |-- urls.py
    |   `-- wsgi.py
    `-- manage.py
```


- Inicia el proyecto

Aunque aún no tengamos nada implementado, podemos ver si todo va bien, iniciando el proyecto y accediendo desde el navegador.
Primero, busca la URL y el puerto que GoormIDE ha asignado para este entorno:

PROJECT > Properties > Runnig URL and Port

En mi caso, https://intro-django.run-eu-central1.goorm.io (si lo deseas puedes generar una nueva)

Una vez la hayas identificado, autoriza la URL del servidor en el proyecto Django, añadiendo el HOST en la sección ALLOWED_HOSTS del fichero settings.py dentro del directorio del proyecto (intro).

```
ALLOWED_HOSTS = ["intro-django.run-eu-central1.goorm.io"]
```

Inicia ahora el proyecto, desde el terminal, con el comando:

```
$ python manage.py runserver 0.0.0.0:80
```

Si todo ha ido bien, deberías ver en el terminal:
```
Django version 2.2.4, using settings 'intro.settings'
Starting development server at http://0.0.0.0:80/
Quit the server with CONTROL-C.
```
NOTA: ignora, de momento, el mensaje sobre las migraciones no aplicadas

Y podrías acceder desde el navegador al proyecto mediante la URL:
```
https://intro-django.run-eu-central1.goorm.io
```

Ahora, es el momento de comenzar a desarrollar la funcionalidad del proyecto. En el siguiente enlace podrás consultar la documentación de la versión 2.2 de Django:

```
https://docs.djangoproject.com/es/2.2/
```

# Crear una aplicación

Para implementar la funcionalidad en un proyecto Django tenemos que crear aplicaciones. Un proyecto puede contener varias aplicaciones y una aplicación puede estar en varios proyectos. Django dispone de una utilidad que genera automáticamente la estructura de una aplicación.

Siguiendo el tutorial "Escribiendo su primera aplicación en Django, parte 1" (https://docs.djangoproject.com/es/2.2/intro/tutorial01/) de la documentación de Django, vamos a crear la aplicación polls:

```
$ python manage.py startapp polls
```
```
`-- polls
    |-- __init__.py
    |-- admin.py
    |-- apps.py
    |-- migrations
    |   `-- __init__.py
    |-- models.py
    |-- tests.py
    `-- views.py
```

# Crear una vista (la más simple posible)

En el fichero polls/views inserta el siguiente código:

```python
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
Para crear las rutas para esta aplicación, crear el fichero polls/urls.py con el siguiente contenido:

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
Para cada ruta en nuestra aplicación añadiremos en urlpatterns una llamada a la función path(). La función path() puede recibir 4 parámetros:
- route: Patrón URL
- view: Vista que se iniciará
- kwargs: diccionario con información para la vista
- name: Nombre para identificar la ruta

El siguiente paso sería crear las rutas principales del proyecto. Por defecto, Django incluye una ruta a la aplicación "admin".
Para incluir nuestra ruta a la aplicación "polls" modificar el fichero urls.py del proyecto (intro/urls.py) para que incluya el siguiente contenido:

```
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
En este caso, hacemos uso de la función include() que permite hacer referencia a fichero con otras rutas; en este caso, el fichero urls.y de la aplicación (módulo) polls.

Ahora puede acceder a la vista index de la aplicación polls con https://intro-django.run-eu-central1.goorm.io/polls.

Si desea poder acceder a esta aplicación desde la URL base del proyecto puede modificar la llamada a path() con:

```
path('', include('polls.urls')),
```


# GoormIDE


```
┌───────────────────────────────────────────────┐
                                       _       
     __ _  ___   ___  _ __ _ __ ___   (_) ___  
    / _` |/ _ \ / _ \| '__| '_ ` _ \  | |/ _ \ 
   | (_| | (_) | (_) | |  | | | | | |_| | (_) |
    \__, |\___/ \___/|_|  |_| |_| |_(_)_|\___/ 
    |___/                                      
			     🌩 𝘼𝙣𝙮𝙤𝙣𝙚 𝙘𝙖𝙣 𝙙𝙚𝙫𝙚𝙡𝙤𝙥!
└───────────────────────────────────────────────┘
```

Welcome to goormIDE!

goormIDE is a powerful cloud IDE service to maximize productivity for developers and teams.  
**DEVELOP WITH EXCELLENCE**  

`Happy coding! The goormIDE team`


## 🔧 Tip & Guide

* Command feature
	* You can simply run your script using the shortcut icons on the top right.
	* Check out `PROJECT > Common/Build/Run/Test/Find Command` in the top menu.
	
* Get URL and Port
	* Click `PROJECT > URL/PORT` in top menu bar.
	* You can get default URL/Port and add URL/Port in the top menu.

* Useful shortcut
	
| Shortcuts name     | Command (Mac) | Command (Window) |
| ------------------ | :-----------: | :--------------: |
| Copy in Terminal   | ⌘ + C         | Ctrl + Shift + C |
| Paste in Terminal  | ⌘ + V         | Ctrl + Shift + V |
| Search File        | ⌥ + ⇧ + F     | Alt + Shift + F  |
| Terminal Toggle    | ⌥ + ⇧ + B     | Alt + Shift + B  |
| New Terminal       | ⌥ + ⇧ + T     | Alt + Shift + T  |
| Code Formatting    | ⌥ + ⇧ + P     | Alt + Shift + P  |
| Show All Shortcuts | ⌘ + H         | Ctrl + H         |

## 💬 Support & Documentation

Visit [https://ide.goorm.io](https://ide.goorm.io) to support and learn more about using goormIDE.  
To watch some usage guides, visit [https://help.goorm.io/en/goormide](https://help.goorm.io/en/goormide)
