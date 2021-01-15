
# Introducción a Django

Continuamos implementando los tutoriales básicos de la documentación de Django (https://docs.djangoproject.com/es/2.2/).

Parte 2: Modelos y el sitio administrativo 

En este ejercicio partimos del tutorial01 implementado en el ejercicio anterior. Puedes clonar el código desde:
https://github.com/jolualma/intro-django-goormide-tutorial01.git

```
|-- README.md
|-- db.sqlite3
|-- goorm.manifest
|-- intro
|   |-- __init__.py
|   |-- __pycache__
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
|-- manage.py
`-- polls
    |-- __init__.py
    |-- __pycache__
    |-- admin.py
    |-- apps.py
    |-- migrations
    |   `-- __init__.py
    |-- models.py
    |-- tests.py
    |-- urls.py
    `-- views.py
```


## Configurar la Base de Datos

Por defecto los proyectos Django utilizan la base de datos SQLite, en el fichero db.sqlite3, como se puede comprobar en el item DATABASE del fichero settings.py.

```
...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

...
```
En este ejercicio utilizaremos SQLite, más adelante se incluirá un tutorial para el uso de otros SGBD.

Django dispone de un ORM (Object Relation Mapped) que genera y persiste la información de los modelos en la base de datos. Para ello, será necesario incluir en el item INSTALLED_APPS del fichero settings.py; esto es, los modelos generados en las aplicaciones indicadas en el item INSTALLED_APPS será persistidas. Por defecto:

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
La persistencia en la base de datos se realiza mediante el concepto de migraciones. El comando migrate será el encargado de realizar este proceso, aplicará todas las migraciones necesarias.

```
$ python manage.py migrate

```

Para ir analizando lo que ocurre en la BD, instalamos el cliente sqlite3

```
$ sudo apt update
$ sudo apt install sqlite3
```
Podemos iniciarlo con:
```
$ sqlite3 db.sqlite3
SQLite version 3.22.0 2018-01-22 18:45:57
Enter ".help" for usage hints.
sqlite>
```
Puede ver el esquema (la creación de tablas) en la base de datos con:
```
sqlite> .schema
```

## Crear modelos

Para la aplicación Polls, vamos a crear dos modelos: Question y Choice. Question contedrá un atributo de tipo texto para incluir la pregunta y un campo de tipo DATETIME para la fecha de publicación. Choice tendrá un campo de texto para guardar la elección, un campo de tipo Integer para el conteo de votos y un atributo para asociarlo a una Question.

Para ello, creamos dos clases de Python en el fichero polls/models.py:

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
Ahora, para que estos modelos se persistan en la BD debemos incluir una referencia a la clase de configuración de la aplicación Polls en el item INSTALLED_APPS del fichero settings.py, de la forma:

```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

Ahora, podemos generar las migraciones para nuestra aplicación con:

```
$ python manage.py makemigrations polls
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```

Antes de aplicar las migraciones, podemos ver cuál sería el efecto de aplicar una determinada migración. Como puede ver en la salida del comando anterior se ha generado el fichero polls/migrations/0001_initial.py, identificado por 0001. Para ver su efecto ejecutamos el comando sqlmigrate:

```
$ python manage.py sqlmigrate polls 0001
```

Para hacer persistentes esos cambios en la BD ejectamos:

```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```

## Shell para Django

La shell de python puede ser utilizada para interactuar con nuestros modelos de Django. Para ello, ejecutamos el comando:

```
$ python manage.py shell
Python 3.7.4 (default, Nov  4 2020, 10:17:35)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.19.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: _
```
Ahora vamos a interractuar con nuestros modelos:

```
Python 3.7.4 (default, Nov  4 2020, 10:17:35)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.19.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from polls.models import Question, Choice

In [2]: from django.utils import timezone

In [3]: q1 = Question(question_text="Qué es nuevo?", pub_date=timezone.now())

In [4]: q1.save()

In [5]: q1.id
Out[5]: 1

In [6]: q1.question_text
Out[6]: 'Qué es nuevo?'

In [7]: q1.pub_date
Out[7]: datetime.datetime(2021, 1, 14, 15, 58, 8, 796412, tzinfo=<UTC>)

In [8]: q1.question_text="Qué pasa?"

In [9]: q1.save()

In [10]: Question.objects.all()
Out[10]: <QuerySet [<Question: Question object (1)>]>
```

Mejoremos la identificación de los objetos implementando un método _str_() en cada modelo y un nuevo método para el modelo Question:

```
from django.db import models
import datetime
from django.utils import timezone


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text

```

Continuamos interractuando con nuestros modelos:

```
$ python manage.py shell
Python 3.7.4 (default, Nov  4 2020, 10:17:35)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.19.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from polls.models import Question, Choice

In [2]: Question.objects.all()
Out[2]: <QuerySet [<Question: Qué tal?>]>

In [3]: Question.objects.filter(id=1)
Out[3]: <QuerySet [<Question: Qué tal?>]>

In [4]: Question.objects.filter(question_text__startswith='Qué')
Out[4]: <QuerySet [<Question: Qué tal?>]>

In [5]: from django.utils import timezone

In [6]: current_year = timezone.now().year

In [7]: Question.objects.filter(pub_date__year=current_year)
Out[7]: <QuerySet [<Question: Qué tal?>]>

In [8]: Question.objects.get(pk=1)
Out[8]: <Question: Qué tal?>

In [9]: q1 = Question.objects.get(pk=1)

In [10]: q1
Out[10]: <Question: Qué tal?>

In [11]: q1.was_published_recently()
Out[11]: True

In [12]: q1.choice_set.all()
Out[12]: <QuerySet []>

In [13]: q1.choice_set.create(choice_text='No mucho', votes=0)
Out[13]: <Choice: No mucho>

In [14]: q1.choice_set.create(choice_text='Solo un poco', votes=0)
Out[14]: <Choice: Solo un poco>

In [15]: q1.choice_set.create(choice_text='Algo más', votes=0)
Out[15]: <Choice: Algo más>

In [16]: c1 = Choice.objects.get(pk=1)

In [17]: c1
Out[17]: <Choice: No mucho>

In [18]: c1.question
Out[18]: <Question: Qué tal?>

In [19]: q1.choice_set.all()
Out[19]: <QuerySet [<Choice: No mucho>, <Choice: Solo un poco>, <Choice: Algo más>]>

In [20]: q1.choice_set.count()
Out[20]: 3

In [21]: Choice.objects.filter(question__pub_date__year=current_year)
Out[21]: <QuerySet [<Choice: No mucho>, <Choice: Solo un poco>, <Choice: Algo más>]>

In [22]: c2 = q1.choice_set.filter(choice_text__startswith='Solo')

In [23]: c2
Out[23]: <QuerySet [<Choice: Solo un poco>]>

In [24]: c2.delete()
Out[24]: (1, {'polls.Choice': 1})

In [25]: q1.choice_set.count()
Out[25]: 2

In [26]: q1.choice_set.all()
Out[26]: <QuerySet [<Choice: No mucho>, <Choice: Algo más>]>

In [27]:
```


## Aplicación admin


Django dispone de una aplicación que permite la gestión de usuarios, además, de ofrecer una interfaz web para gestionar nuestras aplicaciones.

Lo primero que tenemos que hacer es crear el usuario administrador:

```
$ python manage.py createsuperuser
Username (leave blank to use 'root'): admin
Email address: jolualma@gmail.com
Password:
Password (again):
Superuser created successfully.
```

Ahora, podemos ejecutar el proyecto:

```
$ python manage.py runserver 0.0.0.0:80
```

y acceder a la aplicación admin desde el navegador e identificarnos con el usuario creado:

```
https://intro-django.run-eu-central1.goorm.io/admin/
```

Finalmente, vamos a registrar nuestra aplicación polls para que podamos gestionarla desde la aplicación admin. Para ello, modifica el fichero polls/admin.py para que incluya el siguiente contenido:

```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```
Ahora, puedes gestionar el modelo Question desde la aplicación admin. Puedes hacer lo mismo con el modelo Choice.




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
