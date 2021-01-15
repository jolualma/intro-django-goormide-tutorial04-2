
# Introducción a Django

Continuamos implementando los tutoriales básicos de la documentación de Django (https://docs.djangoproject.com/es/2.2/).

Parte 3: Vistas y plantillas (https://docs.djangoproject.com/es/2.2/intro/tutorial03/)

En este ejercicio partimos del tutorial02 implementado en el ejercicio anterior. Puedes clonar el código desde:
https://github.com/jolualma/intro-django-goormide-tutorial02.git


## Crear vistas simples

En el fichero polls/views.py vamos a crear 3 nuevas vistas muy simples, pero que en este caso contienen parámetros:

```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)

```
Registremos las rutas para estas nuevas vistas en el fichero polls/urls.py:

```
urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

```
Estas nuevas rutas tienen parte dinámicas, el objeto petición junto con las partes dinámicas son pasadas como parámetos al método indicado en la vista. Así, por ejemplo, en una petición polls/10 que casaría con la segunda llamada a la función path() resultaria que:
```
path('<int:question_id>/', views.detail, name='detail')
```
invocaría el método detail de las vista, de la forma:
```
detail(request=<HttpRequest object>, question_id=10)
```
Esto nos ha permitido comprobar como definir las URLs y asociarlas con las vistas determinadas, pero realmente no hemos creado ninguna funcionalidad para nuestra aplicación.


## Crear vistas más complejas usando templates

En Django toda vista recibe un objeto HttpRequest y debe generar como respuesta un objeto HttpResponse o una excepción.
De forma habitual de funcionamiento sería cargar datos en un contexto y devolver el objeto HttpResponse hacia una plantilla que mostrará los resultados. Para facilitar este funcionamiento Django proporciona la función render(). La función render() recibe como parámetro el objeto petición, una plantilla como su segundo argumento y un diccionario opcional donde podemos pasar el contexto, y devuelve un objeto HttpResponse generado a partir de la plantilla donde puede utilizarse el contexto.

Para implementar nuestras plantillas, creamos un directorio "templates" dentro de nuestra aplicación, ya que tal como está configurado Django por defecto en el fichero settings.py (item TEMPLATES), buscará plantillas con el template de Django (DjangoTemplates) en ese directorio. Además, para organizar las plantillas de cada aplicación, es recomendable crear dentro de ese directorio template un directorio con el nombre de la aplicación, donde finalmente ubicaremos las plantillas.

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```


Mejoremos nuestra vista index para mostrar las cinco últimas preguntas (Question) mediante una plantilla; para ello, modifiquemos el fichero polls/view con el siguiente código:

```python
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

Para incluir la plantilla para la vista index, cree dentro el fichero templates/polls/index.html que incluya el siguiente contenido:


```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

Vamos a mejorar ahora la vista detail:

```python
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```
O bien, usando la función get_object_or_404():

```python
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```
Creemos ahora la plantilla templates/polls/detail.html:

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

Mejoremos las URLs en los enlaces de nuestras vistas mediamnte {% url %}. Primero, incluyamos un espacio de nombres para la aplicación pools incluyendo en el fichero polls/urls.py el item app_name.

```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Ahoa, en la plantilla index.html cambiemos la línea:

```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

por

```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

y, finalmente, incluyamos en la plantilla detail.html un enlace para volver a la vista index:

```
<a href="{% url 'polls:index' %}">Volver</a>
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
