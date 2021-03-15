# Django tuto

Criamos o projeto, e logo vamos iniciar uma app de Polls

```bash
python manage.py startapp polls
```

Vai criar uma pasta com varios arquivos, na view colocamos isso:

```py
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

Agora devemos definir a rota na pasta polls, com isso, criamos o arquivo py de rotas e colocamos isso:

```py
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

> Isso indica que no arquivo view no metodo index temos um caminho vazio, e o name é um apelido da rota que usamos como substituto com lincagens.

Agora vamos no arquivo de urls da pasta principal do projeto e colocamos isso:

```py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

> Note que neste ele instancia a rota feita na pasta polls e arquivo urls, e no caminho colocamos polls. 

A função [`include()`](https://docs.djangoproject.com/pt-br/3.1/ref/urls/#django.urls.include "django.urls.include") permite referenciar outras URLconfs. Qualquer lugar que o Django encontrar [`include()`](https://docs.djangoproject.com/pt-br/3.1/ref/urls/#django.urls.include "django.urls.include"), irá recortar todas as partes da URL encontrada até aquele ponto e enviar a string restante para URLconf incluído para processamento posterior.

A idéia por trás do [`include()`](https://docs.djangoproject.com/pt-br/3.1/ref/urls/#django.urls.include "django.urls.include") é facilitar plugar URLs. Uma vez que polls está em sua própria URLconf (`polls/urls.py`), ele pode ser colocado depois de “/polls/”, ou depois de “/fun_polls/”, u depois de “/content/polls/”, ou qualquer outro início de caminho, e a aplicação ainda irá funcionar

## config db

Fazemos um migrate:

```bash
python manage.py migrate
```

E partimos para criar nosso modelo. Iremos para `polls/models.py` e escrevemos isso:

```py
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Aqui criamos duas tabelas de nosso db

Para a possibilidade de fazer a migration add o app no settings:

```py
INSTALLED_APPS = [
    'polls.apps.PollsConfig', <---------
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

Tal PollsConfig é a classe em app.py da pasta polls

Com isso fazemos a migracao:

```bash
python manage.py makemigrations polls
```

Ao executar `makemigrations`, você está dizendo ao Django que você fez algumas mudanças em seus modelos (neste caso, você fez novas modificações) e que você gostaria que as alterações sejam armazenadas como uma *migração*.

As migrações são como o Django armazena as alterações nos seus modelos (e, portanto, seu esquema de banco de dados) - eles são apenas arquivos no disco. Você pode ler a migração para seu novo modelo, se quiser; é o arquivo `polls/migrations/0001_initial.py`. Não se preocupe, você não precisa lê-los cada vez que o Django cria um, mas eles são projetados para serem editados no caso de você precisar ajustar manualmente como Django muda as coisas.

Logo colocamos isso:

```bash
python manage.py sqlmigrate polls 0001
```

Isso vai gerar o arquivo sql com seus escritos. E com isso:

```bash
python manage.py migrate
```

Vamos colocar tais informacoes no db.

O comando [`migrate`](https://docs.djangoproject.com/pt-br/3.1/ref/django-admin/#django-admin-migrate) pega todas as migrações que ainda não foram aplicadas (Django rastreia quais foram aplicadas usando uma tabela especial em seu banco de dados chamada `django_migrations`) e aplica elas no seu banco de dados - essencialmente, sincronizando as alterações feitas aos seus modelos com o esquema no banco de dados.

Migrações são muito poderosas e permitem que você altere seus modelos ao longo do tempo, à medida que desenvolve o seu projeto, sem a necessidade de remover o seu banco de dados ou tabelas e criar de novo - ele é especializado em atualizar seu banco de dados ao vivo, sem perda de dados. Nós vamos cobri-los com mais profundidade em uma parte posterior do tutorial, mas para agora, lembre-se deste guia de três passos para fazer mudanças nos seus modelos:

- Mude seus modelos (em `models.py`).
- Rode [`python manage.py makemigrations`](https://docs.djangoproject.com/pt-br/3.1/ref/django-admin/#django-admin-makemigrations) para criar migrações para suas modificações
- Rode [`python manage.py migrate`](https://docs.djangoproject.com/pt-br/3.1/ref/django-admin/#django-admin-migrate) para aplicar suas modificações no banco de dados.

## Uso da API para db

Entramos no terminal

```bash
python manage.py shell
```

```bash
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

E

```bash
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

## Criar superusuario

Vamos criar um ambiente onde os desenvolvedores podem manipular os dados do db.

```bash
python manage.py createsuperuser
```

E fazemos um cadastro:

```bash
Username: admin
```

e

```bash
Email address: admin@example.com
```

e

```bash
Password: **********
Password (again): *********
Superuser created successfully.
```

> Fazer isso no bash pode nao funcionar MINGW64

Em tal rota /admin/ podemos colocar as informacoes de nosso cadastro. Entrando, nos deparamos com a tela de controle que podemos personalizar no arquivo de admin.py

```py
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

## Rotas e vistas

Criamos novos codigos em views de polls

```py
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

E em rotas colocamos :

```py
from django.urls import path

from . import views

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

Isto indica como funciona o metodo das rotas, usando id para poder ir em uma pagina especifica, para mostrar as informacoes de esta tabela.

Em nosso view vamos ter acesso ao db usandos metodos que vimos na api:

```py
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged
```

Logo vamos criar uma pasta em polls de templates: 

polls/templates/polls/index.html

```jinja2
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

E fazemos mudanças no metodo index:

```py
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

Esse código carrega o template chamado `polls/index.html` e passa um contexto para ele. O contexto é um dicionário mapeando nomes de variáveis ​​para objetos Python.

É um estilo muito comum carregar um template, preenchê-lo com um contexto e retornar um objeto [`HttpResponse`](https://docs.djangoproject.com/pt-br/3.1/ref/request-response/#django.http.HttpResponse "django.http.HttpResponse") com o resultado do template renderizado. O Django fornece este atalho. Aqui esta toda a view `index()` reescrita:

```py
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

Assim fica mais limpo o codigo

Agora vamos colocar um tela de erro 404

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

E criamos uma pagina html para detail:

```jinja2
{{ question }}
```

Podemos usar outro atalho:

```py
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

E na pagina html:

```py
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

Mudamos em index.html a parte do link, pois nao é inteligente e dinamico de mudar:

```jinja2
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

> note que usamos o apelido detail como substituto do caminho em url.

Em urls add isso:

```py
app_name = 'polls'
```

Para indicar qual aplicacao sera usada para a referencia.

E mudamos de novo no index.html:

```jinja2
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

## Formulario

No detail.html escrevemos:

```py
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
```

Criamos o controle de voto:

```py
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
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

Logo colocamos mais isso:

```py
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

E criamos o html de results

```jinja2
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

Lembrando em admin.py add Choice para criar a tabela de escolhas. 

## Views genericas

Mudamos as url:

```py
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

E vamos as views

```py
from django.views import generic

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
```

Add novas e mantemos apenas o vote

Nós estamos usando duas views genéricas aqui: [`ListView`](../ref/class-based-views/generic-display.html#django.views.generic.list.ListView "django.views.generic.list.ListView") e [`DetailView`](../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView"). Respectivamente, essas duas `views` abstraem o conceito de exibir uma lista de objetos e exibir uma página de detalhe para um tipo particular de objeto.

- Cada “view” genérica precisa saber qual é o modelo que ela vai agir. Isto é fornecido usando o atributo `model`.
- A view genérica [`DetailView`](../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") espera o valor de chave primaria capturado da URL chamada `"pk"`, então mudamos `question_id` para `pk` para as views genérica.

Por padrão, a “view” genérica [`DetailView`](../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") utiliza um template chamado ``<app name>/<model name>_detail.html`. Em nosso caso, ela vai utilizar o template `"polls/question_detail.html"``. O atributo `template_name` é usado para indicar ao Django para usar um nome de template em vez de auto gerar uma nome de template. Nós também especificamos `template_name` para a view de listagem de `results` – isto garante que a view de detalhe tem uma aparência quando renderizada, mesmo que sejam tanto um class:~django.views.generic.detail.DetailView por trás das cenas.
