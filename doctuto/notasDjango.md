# Notas Django

Baixamos a versao atual de python e pip, depois disso instalamos django com:

```bash
import django
```

Criamos uma pasta para o projeto e pelo terminal dentro da pasta colocamos:

```bash
django-admin startproject nomeprojeto
```

Na pasta teremos o arquivo `manage.py` e outra pasta com 4 arquivos.

O manage.py nos dara comandos para ajudar no projeto, como por exemplo o artisan do laravel.

## Criar db

No terminal escrevemos a primeira migracao 

```bash
python manage.py migrate
```

Sera criado um arquivo db.sqlite3, e logo ativamos os servidores com: 

```bash
python manage.py runserver
```

Ao fazer isso nos dara a direcao para executar o site. Pode ser tanto /http://127.0.0.1:8000/  ou localhost:8000

## Arquivos estaticos

A pasta de arquivos estaticos deve ficar no mesmo nivel do arquivo manage.py, para poder funcionar.

## Area admin

Pra entrar na area de admin, temos um rota `admin/` que nos leva para a tela de login,
mas teremos que criar um **superusuario** para executar isso.

No terminal colocamos:

```
python manage.py createsuperuser
```

Logo vamos criar nosso cadastro colocando **nome**, **email**, e **senha**.
Depois disso, ja podemos entrar na area de admin.

---

Na tela inicial ja vemos uma tabela de grupos e usuarios, semelhante ao phpmyadmin.
No codigo podemos fazer modificacoes para adicionar funcionalidades em nossa area de 
admin.

Tais modificacoes fazemos em `admin.py`

```py
from django.contrib import admin

from dbconex.models import Pessoa, Cores # <---

# Register your models here.

admin.site.register(Pessoa) # <----
```

Como vemos no codigo acima, importamos a base de dados e colocamos o codigo
abaixo. Ao fazer isso, na tela inicial, vamos poder editar a tabela de Pessoa. O mesmo podemos fazer para Cores.

## Uso da API django

Usando o `python shell` podemos colocar, editar e eliminar infos da DB projeto
termnal

usando::> `python manage.py shell`

```bash
>>> from gestionPedidos.models import Artigos //importar a tabela da db

>>> art=Artigos(nome='mesa', secao='decoracao', preco=90) 
```

Criamos uma var de tal tabela e colocamos os valores de cada coluna criada

```bash
>>> art.save() //para salvar as alteracoes

>>> art2=Artigos(nome='cadeira', secao='moveis', preco=50)      
>>> art2.save()
```

Criamos mais um

```bash
>>> art2=Artigos.create(nome='cadeira', secao='moveis', preco=50) 
```

Desta forma ja salvamos sem fazer isso: art2.save()

Para atualizar basta pegar uma variavel por exemplo // art.preco=91
Logo colocar // `art.save()` // assim atualizamos a coluna de preco

Para deletar fazemos isso:

```bash
art3=Artigos.objects.get(id=1) //para deletar uma linha
```

Usamos uma nova variavel e logo // `art3.delete()`

## Template engine

Django usa Jinja como motor de template, e ele usa tags especias no arquivo html.
Usamos a tag para resgatar uma variavel do controlador:

```jinja2
{{variavel}}
```

Tag para extender herança de escritos html:

```jinja2
{% block body %} xxxxxxx {% endblock %}
```

Tag de extender:

```jinja2
{% extends "ssss.html" %}
```

> Obs: as aspas duplas nao podem tocar no % -> {% extends "ssss.html"%} (X)

Podemos colocar codigos python com essas tags especiais:

```jinja2
{% for x in dd %} sdd {% endfor %}
```

Assim como outras possibilidades.

### include

Para incluir um codigo html em outro arquivo html usamos:

```jinja2
{% include "sdsd".html %}
```

Colocamos a tag onde desejamos incluir tal codigo.

### Templates caminho

Em `settings.py` podemos definir o caminho padrao de onde estao armazenadas os templates

```py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',

        'DIRS': [], <----- clocamos aqui o caminho do template

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

Supondo que tenho uma pasta templates e uma pasta que guarda o arquivo de layout que vai sofrer herança com os demais arquivos html.
No caminho padrao devemos colocar o caminho de todas as pastas que voce estiver usando de
seu template:

```py
'DIRS': ["./dbconex/templates", "./dbconex/templates/fundo"],
```

Note que colocamos os caminhos das pastas existenetes neste projeto. Nao precisa indicar
os arquivos, somente o caminho das pastas.

## Static files

Para definir o caminho de arquivos estaicos de css, js e outros temos ir em `settings.py` e definir o caminho.

```py
STATICFILES_DIRS = [
BASE_DIR / "static",
]
```

Adicionamos este codigo para definir o caminho onde esta a pasta static, e a pasta tem que 
ter esse nome.

> A pasta static tem que ser criada no mesmo nivel do arquivo `manage.py`, para poder 
> funcionar.

## Instalacoes uteis

Podemos instalar isso:

```
pip install pylint-django
```

Para poder importar valores do banco de dados, e logo com control+shift+p entrando em `Configure Language Specific Settings` e colocamos este codigo, lembrando de colocar 
continuidade nas virgulas:

```
"python.linting.pylintEnabled": true,
"python.linting.pylintArgs": [
"--disable=C0111", // missing docstring
"--load-plugins=pylint_django,pylint_celery",
```

Com isso o problema desaparece
