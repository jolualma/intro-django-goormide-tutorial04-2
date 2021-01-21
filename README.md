
# Introducción a Django

Continuamos implementando los tutoriales básicos de la documentación de Django (https://docs.djangoproject.com/es/2.2/).

Parte 4-2: Vistas Genéricas (https://docs.djangoproject.com/es/2.2/intro/tutorial04/#use-generic-views-less-code-is-better)


En este ejercicio partimos del tutorial04 implementado en el ejercicio anterior. Puedes clonar el código desde:
https://github.com/jolualma/intro-django-goormide-tutorial04-1


## Vistas Genéricas

Django proporciona un sistema, denominado «vistas genéricas», que permite abstraerse de los patrones comunes en el desarrollo web,d e tal forma que gran parte del código no es necesario implementarlo ya que Django lo hace por nosotros mediante convenciones.

Para hacer uso de este sistema:

Primero, modifiquemos las rutas de la aplicación (fichero polls/urls.py) con el siguiente contenido:

```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Modifiquemoa, ahora, las vistas (fichero polls/urls.py) con este contenido:


```python
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


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
```

Estamos utilizando dos vistas genéricas aquí: ListView y DetailView. Respectivamente, esas dos vistas abstraen los conceptos de «mostrar una lista de objetos» y «mostrar una página de detalles para un tipo específico de objeto.»

    Cada vista genérica tiene que saber cuál es el modelo sobre el que estará actuando. Esto se proporciona utilizando el atributo model.
    La vista genérica DetailView espera que el valor de la clave primaria capturado desde la URL sea denominado "pk", por lo que hemos cambiado``question_id`` a pk para las vistas genéricas.

Por defecto, la vista genérica DetailView utiliza una plantilla llamada <app name>/<model name>_detail.html. En nuestro caso, utilizaría la plantilla "polls/question_detail.html". El atributo template_name se utiliza para indicarle a Django que utilice un nombre de plantilla específico en vez del nombre de plantilla generado de forma automática. También especificamos el atributo template_name para la vista de lista results, esto garantiza que la vista de resultados y la vista detalle tengan un aspecto diferente cuando sean creadas, a pesar de que las dos son una vista genérica DetailView en segundo plano.

Del mismo modo, la vista genérica ListView utiliza una plantilla predeterminada llamada <app name>/<model name>_list.html; utilizamos el atributo template_name para indicarle a ListView que utilice nuestra plantilla "polls/index.html" existente.

En partes anteriores del tutorial, las plantillas se han dotado de un contexto que contiene las variables contextuales question y latest_question_list. Para DetailView se suministra la variable question de forma automática, ya que estamos utilizando un modelo (Question) de Django, Django puede determinar un nombre adecuado para la variable contextual. Sin embargo, para ListView, la variable contextual generada de forma automática es question_list. Para anular esto, proporcionamos el atributo``context_object_name`` especificando que queremos utilizar en cambio latest_question_list. Como un método alternativo, usted puede modificar sus plantillas para que coincidan con las nuevas variables contextuales predeterminadas, pero es mucho más sencillo solo indicarle a Django que utilice la variable que usted quiere.


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
