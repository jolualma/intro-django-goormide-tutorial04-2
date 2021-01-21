
# Introducción a Django

Continuamos implementando los tutoriales básicos de la documentación de Django (https://docs.djangoproject.com/es/2.2/).

Parte 4: Formulario (https://docs.djangoproject.com/es/2.2/intro/tutorial04/)

En este ejercicio partimos del tutorial03 implementado en el ejercicio anterior. Puedes clonar el código desde:
https://github.com/jolualma/intro-django-goormide-tutorial03


## Crear un formulario sencillo

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


Insertemos en la vista details.html (polls/templates/polls/details.html) un formulario, quedando con el siguiente contenido:

```html
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>

<a href="{% url 'polls:index' %}">Volver</a>
```

Este formulario permitirá elegir la opción deseada para la pregunta de la cuál se muestra el detalle.

Ahora, vamos a incluir una vista para mostrar los votos recibidos para cada una de las opciones de una pregunta. En el fichero views.py (polls/views) modificamos el método vote con el siguiente contenido

´´´python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.http import Http404

from .models import Question, Choice

...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
...
```

El objeto request.POST es diccionario que incluye los datos enviados desde un formulario; en concreto, request.POST['choice'] devolverá el valor de la opción marcada en el formulario anterior.
Si el parámetro consultado (en este caso, choice) no aparece se lenzará una excepción KeyError. Si se produce la excepción se volverá a renderizar la vista details.html pasándole en el contexto la pregunta y un mensaje de error. Si no se lanza la excepción, se optendrá el objeto Choice concreto, se incrementará el número de votos y se redirigirá la salida a vista respuesta. La función reverse() recibe el nombre de las vista a la que deseamos pasar el control.

Modifiquemos la vista results:

```python
...
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
...
```

y creemos un template para ella:

```html
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```



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
