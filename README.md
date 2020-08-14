# Dicas de Django

Várias dicas de Django - assuntos diversos.

1. [Django boilerplate e cookiecutter-django](#1---django-boilerplate-e-cookiecutter-django)
2. [Django extensions](#2---django-extensions)
3. [Django bulk_create e django-autoslug](#3---django-bulk_create-e-django-autoslug)
4. [Django Admin personalizado](#4---django-admin-personalizado)
5. [Django Admin Date Range filter](#5---django-admin-date-range-filter)
6. [Geradores de senhas randômicas - uuid, hashids, secrets](#6---geradores-de-senhas-rand%C3%B4micas---uuid-hashids-secrets)
7. [Rodando o ORM do Django no Jupyter Notebook](#7---rodando-o-orm-do-django-no-jupyter-notebook)
8. [Conhecendo o Django Debug Toolbar](#8---conhecendo-o-django-debug-toolbar)
9. [Escondendo suas senhas python-decouple](#9---escondendo-suas-senhas-python-decouple)
10. [Prototipagem de web design (Mockup)](#10---prototipagem-de-web-design-mockup)
11. [Bootstrap e Bulma + Colorlib](#11---bootstrap-e-bulma--colorlib)
12. [Imagens: pexels e unsplash](#12---imagens-pexels-e-unsplash)
13. [Cores](#13---cores)
14. [Herança de Templates e Arquivos estáticos](#14---herança-de-templates-e-arquivos-estáticos)
15. [Busca por data no frontend](#15---busca-por-data-no-frontend)
16. [Filtros com django-filter](#16---filtros com django-filter)

## This project was done with:

* Python 3.8.2
* Django 2.2.13

## How to run project?

* Clone this repository.
* Create virtualenv with Python 3.
* Active the virtualenv.
* Install dependences.
* Run the migrations.

```
git clone https://github.com/rg3915/dicas-de-django.git
cd dicas-de-django
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python contrib/env_gen.py
python manage.py migrate
```

## Este projeto foi feito com:

* Python 3.8.2
* Django 2.2.13

## Como rodar o projeto?

* Clone esse repositório.
* Crie um virtualenv com Python 3.
* Ative o virtualenv.
* Instale as dependências.
* Rode as migrações.

```
git clone https://github.com/rg3915/dicas-de-django.git
cd dicas-de-django
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python contrib/env_gen.py
python manage.py migrate
```

# 1 - Django boilerplate e cookiecutter-django

[boilerplatesimple.sh](https://gist.github.com/rg3915/b363f5c4a998f42901705b23ccf4b8e8)

[boilerplate2.sh](https://gist.github.com/rg3915/a264d0ade860d2f2b4bf)

[cookiecutter-django](https://github.com/pydanny/cookiecutter-django)

```
python -m venv .venv
source .venv/bin/activate

pip install "cookiecutter>=1.7.0"
cookiecutter https://github.com/pydanny/cookiecutter-django
pip install -r requirements/local.txt 
python manage.py migrate

createdb myproject -U postgres

python manage.py migrate
```

# 2 - Django extensions

https://django-extensions.readthedocs.io/en/latest/index.html

```
pip install django-extensions
```

```python
# settings.py
INSTALLED_APPS = (
    ...
    'django_extensions',
)
```

```
python manage.py show_urls
```

```
python manage.py shell_plus
```


# 3 - Django bulk_create e django-autoslug

### python-slugify

https://pypi.org/project/python-slugify/

```python
from slugify import slugify
text = 'Dicas de Django'
print(slugify(text))
url = f'example.com/{slugify(text)}'
```

#### bulk-create

https://docs.djangoproject.com/en/3.0/ref/models/querysets/#bulk-create


#### django-autoslug

https://pypi.org/project/django-autoslug/

```
pip install django-autoslug
```


#### models.py

```python
from django.db import models
from autoslug import AutoSlugField


class Article(models.Model):
    title = models.CharField('título', max_length=200)
    subtitle = models.CharField('sub-título', max_length=200)
    slug = AutoSlugField(populate_from='title')
    category = models.ForeignKey(
        'Category',
        related_name='categories',
        verbose_name='categoria',
        on_delete=models.SET_NULL,
        null=True,
        blank=True
    )
    published_date = models.DateTimeField(
        'criado em',
        auto_now_add=True,
        auto_now=False
    )

    class Meta:
        ordering = ('title',)
        verbose_name = 'artigo'
        verbose_name_plural = 'artigos'

    def __str__(self):
        return self.title


class Category(models.Model):
    title = models.CharField('título', max_length=50, unique=True)

    class Meta:
        ordering = ('title',)
        verbose_name = 'categoria'
        verbose_name_plural = 'categorias'

    def __str__(self):
        return self.title
```


#### shell_plus

```python
categories = [
    'dicas',
    'django',
    'python',
]

aux = []

for category in categories:
    obj = Category(title=category)
    aux.append(obj)

Category.objects.bulk_create(aux)


titles = [
    {
        'title': 'Django Boilerplate',
        'subtitle': 'Django Boilerplate',
        'category': 'dicas'
    },
    {
        'title': 'Django extensions',
        'subtitle': 'Django extensions',
        'category': 'dicas'
    },
    {
        'title': 'Django Admin',
        'subtitle': 'Django Admin',
        'category': 'admin'
    },
    {
        'title': 'Django Autoslug',
        'subtitle': 'Django Autoslug',
        'category': 'dicas'
    },
]

aux = []

for title in titles:
    category = Category.objects.filter(title=title['category']).first()
    article = dict(
        title=title['title'],
        subtitle=title['subtitle']
    )
    if category:
        obj = Article(category=category, **article)
    else:
        obj = Article(**article)
    aux.append(obj)

Article.objects.bulk_create(aux)
```



# 4 - Django Admin personalizado

https://docs.djangoproject.com/en/3.0/ref/contrib/admin/#modeladmin-options

#### admin.py

```python
from django.conf import settings
from django.contrib import admin
from .models import Article, Category
# from .forms import ArticleAdminForm


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'get_published_date')
    search_fields = ('title',)
    list_filter = (
        'category',
    )
    readonly_fields = ('slug',)
    date_hierarchy = 'published_date'
    # form = ArticleAdminForm

    def get_published_date(self, obj):
        if obj.published_date:
            return obj.published_date.strftime('%d/%m/%Y')

    get_published_date.short_description = 'Data de Publicação'


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    actions = None

    def has_add_permission(self, request, obj=None):
        return False

    if not settings.DEBUG:
        def has_delete_permission(self, request, obj=None):
            return False
```




# 5 - Django Admin Date Range filter

https://github.com/tzulberti/django-datefilterspec

```
pip install django-daterange-filter
```

```python
# settings.py
INSTALLED_APPS = (
    ...
    'daterange_filter'
)
```

```python
# admin.py
from daterange_filter.filter import DateRangeFilter
...

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    ...
    list_filter = (
        ('published_date', DateRangeFilter),
        'category',
    )
```

# 6 - Geradores de senhas randômicas - uuid, hashids, secrets

## 6.1 - uuid

https://docs.python.org/3/library/uuid.html

```python
import uuid

uuid.uuid4()

uuid.uuid4().hex
```

```python
# models.py
import uuid
from django.db import models


class UuidModel(models.Model):
    slug = models.UUIDField(unique=True, editable=False, default=uuid.uuid4)

    class Meta:
        abstract = True

class Category(UuidModel):
    title = models.CharField('título', max_length=50, unique=True)
    ...
```

## 6.2 - shortuuid

https://pypi.org/project/shortuuid/

```
pip install shortuuid
```

```python
>>> import shortuuid
>>> 
>>> shortuuid.uuid()
'823MMBZx7LNEnnPBtCAorG'
>>> 
>>> shortuuid.uuid(name='example.com')
'exu3DTbj2ncsn9tLdLWspw'
>>> 
>>> shortuuid.ShortUUID().random(length=22)
'4CHN7TshKtrVnW4KLgVMhY'
>>> 
>>> shortuuid.set_alphabet('regis')
>>> shortuuid.uuid()
'gerigigiesreissgisrsggrrseieereggierrgreriissreiiisiegrr'
>>> 
```

## 6.3 - hashids

https://gist.github.com/rg3915/4684721a603cf6d0dd3b9495744482fe

https://pypi.org/project/hashids/

```
pip install hashids
```

```python
from hashids import Hashids
hashids = Hashids()

>>> hashids.encode(42)
'9x'
>>> hashids.decode('9x')
(42,)
>>> hashids.encode(665190)
'k7qWJ'
>>> hashids.decode('k7qWJ')
(665190,)
>>> hashids.encode(1122, 4200, 32665)
'ELmhW0mFD7o'
>>> hashids.decode('ELmhW0mFD7o')
(1122, 4200, 32665)
>>> hashids = Hashids(alphabet='abcdefghijklmnopqrstuvwxyz1234567890', min_length=22)
>>> for i in range(10): hashids.encode(i)
... 
'9xkwnvoj3ejwgp6481y5mq'
'ml6kz731jdkoe524rxn0yq'
'kwp7yx456gl9g91lm23v8n'
'0qr6jxo9memje214w8zlvp'
'9poy2jq1xdn0e037nwv4zl'
'nz97pw01jgo5el24yrxv6m'
'q4pkmy631epjenrxv70w5l'
'n97kyw8q0dq9eo143z2x6v'
'x7n4zl0pkgr4d6o3vq92wy'
'6y27mjnzkev3d3549vq0xl'
>>> 
```

### django-hashid-field

https://pypi.org/project/django-hashid-field/

```
pip install django-hashid-field==3.1.3
```

```python
# models.py
from hashid_field import HashidAutoField


class Article(models.Model):
    id = HashidAutoField(primary_key=True)
    ...
```


```python
# python manage.py shell_plus
from hashid_field import Hashid

articles = Article.objects.all()
for article in articles:
    hashid = Hashid(article.id)
    print(article.id, hashid.id)
```

https://www.howtogeek.com/howto/30184/10-ways-to-generate-a-random-password-from-the-command-line/

### Gerando senhas no terminal Linux

```
date +%s | sha256sum | base64 | head -c 32 ; echo

openssl rand -base64 32

strings /dev/urandom | grep -o '[[:alnum:]]' | head -n 30 | tr -d '\n'; echo

date | md5sum
```


```
# apt install -y gpw

gpw

gpw 3 32

gpw 1 5
```

### Gerando senhas com Python

#### com random

```python
import random

chars = "abcdefghijklmnopqrstuvwxyz01234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ!@#$%^&*()?"
size = 8
secret_key = "".join(random.sample(chars,size))
print(secret_key)
```

#### com random e string

https://pynative.com/python-generate-random-string/

```python
# Generate a random string of specific letters only

import random
import string

def randString(length=5):
    # put your letters in the following string
    your_letters='abcdefghi'
    return ''.join((random.choice(your_letters) for i in range(length)))

print("Random String with specific letters ", randString())
print("Random String with specific letters ", randString(5))
```

#### com secrets

https://docs.python.org/3/library/secrets.html

*New in Python 3.6*

```python
import secrets

secrets.token_hex(16)

secrets.token_urlsafe(16)

url = 'https://mydomain.com/reset=' + secrets.token_urlsafe()
```

### Django

```
vim contrib/env_gen.py
```

```python
"""
Django SECRET_KEY generator.
"""
from django.utils.crypto import get_random_string


chars = 'abcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*(-_=+)'

CONFIG_STRING = """
DEBUG=True
SECRET_KEY=%s
ALLOWED_HOSTS=127.0.0.1, .localhost
""".strip() % get_random_string(50, chars)

print(CONFIG_STRING)

# Writing our configuration file to '.env'
with open('.env', 'w') as configfile:
    configfile.write(CONFIG_STRING)
```

```
python contrib/env_gen.py
```

# 7 - Rodando o ORM do Django no Jupyter Notebook

Instale

```
pip install ipython[notebook]
```

Rode

```
python manage.py shell_plus --notebook
```

**Obs**: No Django 3.x talvez você precise dessa configuração [async-safety](https://docs.djangoproject.com/en/3.0/topics/async/#async-safety).

`os.environ["DJANGO_ALLOW_ASYNC_UNSAFE"] = "true"`


# 8 - Conhecendo o Django Debug Toolbar

https://django-debug-toolbar.readthedocs.io/en/latest/

```
pip install django-debug-toolbar
```

#### Configurando o `settings.py`

```
INSTALLED_APPS = [
    # ...
    'django.contrib.staticfiles',
    # ...
    'debug_toolbar',
]

MIDDLEWARE = [
    # ...
    'debug_toolbar.middleware.DebugToolbarMiddleware',
    # Deve estar por último.
]

INTERNAL_IPS = [
    # ...
    '127.0.0.1',
    # ...
]

STATIC_URL = '/static/'
```

#### Configurando o `urls.py`

```
from django.conf import settings
from django.urls import include, path

if settings.DEBUG:
    import debug_toolbar
    urlpatterns = [
        path('__debug__/', include(debug_toolbar.urls)),
    ] + urlpatterns
```

# 9 - Escondendo suas senhas python-decouple

https://github.com/henriquebastos/python-decouple

[Video do Henrique Bastos na Live de Python #97](https://www.youtube.com/watch?v=zYJGpLw5Wv4)

```
pip install python-decouple
```

Crie um arquivo `.env` com o seguinte conteúdo (de exemplo)

```
DEBUG=True
SECRET_KEY=c9^3g^bn6wgo8tabf*dl$@vx@m-!9ux%*9)88qnun&hk++sa90
ALLOWED_HOSTS=127.0.0.1,.localhost
POSTGRES_DB=mydb
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypass
DB_HOST=localhost

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_STORAGE_BUCKET_NAME=

# console ou smtp
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend
EMAIL_HOST=smtp.sendgrid.net
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=
EMAIL_HOST_PASSWORD=
```

repare que não deve haver espaços e nem aspas.

E em `settings.py` faça

```
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default=[], cast=Csv())

EMAIL_BACKEND = config('EMAIL_BACKEND')
EMAIL_HOST = config('EMAIL_HOST')
EMAIL_PORT = config('EMAIL_PORT')
EMAIL_USE_TLS = config('EMAIL_USE_TLS')
EMAIL_HOST_USER = config('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD')

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': config('POSTGRES_DB'),
        'USER': config('POSTGRES_USER'),
        'PASSWORD': config('POSTGRES_PASSWORD'),
        'HOST': config('DB_HOST', 'localhost'),
        'PORT': '5432',
    }
}
```

# 10 - Prototipagem de web design (Mockup)

[excalidraw.com](https://excalidraw.com/)

[moqups.com](https://moqups.com/)

[balsamiq.com](https://balsamiq.com/)

[marvelapp.com](https://marvelapp.com/)

[mockflow.com](https://www.mockflow.com/)


# 11 - Bootstrap e Bulma + Colorlib

[getbootstrap.com](https://getbootstrap.com/)

[getbootstrap.com/docs/4.5/examples](https://getbootstrap.com/docs/4.5/examples/)

[bulma.io](https://bulma.io/)

[bulmatemplates.github.io/bulma-templates](https://bulmatemplates.github.io/bulma-templates/)

[bulmathemes.com](https://bulmathemes.com/)

[colorlib.com](https://colorlib.com/)


# 12 - Imagens: pexels e unsplash

[pexels.com](https://www.pexels.com/pt-br/)

[unsplash.com](https://unsplash.com/)


# 13 - Cores

[color.adobe.com/pt/create/color-wheel](https://color.adobe.com/pt/create/color-wheel)

[coolors.co](https://coolors.co/)

[materialuicolors.co](https://materialuicolors.co/)

[htmlcolorcodes.com](https://htmlcolorcodes.com/)

[clrs.cc](http://clrs.cc/)


# 14 - Herança de Templates e Arquivos estáticos

[Video Introdução a Arquitetura do Django - Pyjamas 2019](https://www.youtube.com/watch?v=XjXpwZhOKOs)

![diagrama](templates-diagram.png)

![templates-pyjamas](https://raw.githubusercontent.com/rg3915/pyjamas2019-django/master/img/final.png)

# 15 - Busca por data no frontend

Considere um template com os campos:

```html
<input class="form-control" name='start_date' type="date">
<input class="form-control" name='end_date' type="date">
```

Em `views.py` basta fazer:

```python
def article_list(request):
    template_name = 'core/article_list.html'
    object_list = Article.objects.all()

    start_date = request.GET.get('start_date')
    end_date = request.GET.get('end_date')

    if start_date and end_date:
        object_list = object_list.filter(
            published_date__range=[start_date, end_date]
        )

    context = {'object_list': object_list}
    return render(request, template_name, context)
```

# 16 - Filtros com [django-filter](https://django-filter.readthedocs.io/en/stable/)

Instale o [django-filter](https://django-filter.readthedocs.io/en/stable/)

```
pip install django-filter
```

Acrescente-o ao `INSTALLED_APPS`

```python
INSTALLED_APPS = [
    ...
    'django_filters',
]
```

Crie um arquivo `filters.py`

```python
import django_filters
from .models import Article


class ArticleFilter(django_filters.FilterSet):
    title = django_filters.CharFilter(lookup_expr='icontains')
    subtitle = django_filters.CharFilter(lookup_expr='icontains')

    class Meta:
        model = Article
        fields = ('title', 'subtitle')
```

Em `views.py`

```python
from .filters import ArticleFilter


def article_list(request):
    template_name = 'core/article_list.html'
    object_list = Article.objects.all()
    article_filter = ArticleFilter(request.GET, queryset=object_list)

    ...

    context = {
        'object_list': article_filter,
        'filter': article_filter
    }
    return render(request, template_name, context)
```

Em `article_list.html`

```html
  <div class="row">
    <div class="col-md-4">
      <form method="GET">
        {{ filter.form.as_p }}
        <input type="submit" />
      </form>
    </div>

    <div class="col-md-8">
      <table class="table">
        <thead>
          <tr>
            <th>Título</th>
            <th>Sub-título</th>
            <th>Data de publicação</th>
          </tr>
        </thead>
        <tbody>
          {% for obj in filter.qs %}
            <tr>
              <td>{{ obj.title }}</td>
              <td>{{ obj.subtitle }}</td>
              <td>{{ obj.published_date }}</td>
            </tr>
          {% endfor %}
        </tbody>
      </table>
    </div>
  </div>
```

