>Boa parte do projeto foi beaseado <a href="https://www.youtube.com/watch?v=wtl8ZyCbTbg&list=PLcM_74VFgRhpyCtsNXyBUf27ZRbyQnEEb">nesse vídeo</a> e <a href="https://pythonacademy.com.br/blog/construcao-de-apis-com-django-rest-framework">nesse artigo</a>. Se assim como eu, você entrou agora no mundo BackEnd, recomendo fortemente tais materiais. <br>Escrevi esse readme com a intenção de revisar o que aprendi e também ajudar aqueles com caminhos similares no mundo tech. Espero que você aprenda algo novo! 👍

# API para uma biblioteca

* [Introdução](#introdução)
* [Preparando o Ambiente](#preparando-o-ambiente)
* [Projeto x App](#projeto-x-app)
* [Criando os modelos e API](#criando-os-modelos-e-api)
* [Criação das rotas](#criação-das-rotas)
* [Getting Started](#getting-started)

# Introdução
A ideia do projeto é que possamos armazenar livros e seus atributos dentro de um banco de dados e gerenciar tudo isso sem precisar de uma interface gráfica. Assim, outra aplicação poderá se comunicar com a nossa de forma eficiente.<br> Esse é o conceito de API (Application Programming Interface)

# Preparando o ambiente
Aqui temos a receita de bolo pra deixar a sua máquina pronta para levantar um servidor com o django e receber aquele **200** bonito na cara

```bash
>python -m venv venv #criando ambiente virtual na sua versao do python
>./venv/Scripts/Activate.ps1 #Ativando o ambiente virtual
>pip install django djangorestframework #instalação local das nossas dependências
```
O lance do ambiente virtual é que todas suas dependências *(que no python costumam ser muitas)*  ficam apenas num diretório específico. <br>
Logo, com uma venv você pode criar projetos que usam versões diferentes da mesma biblioteca sem que haja conflito na hora do import.

# Projeto x App
No django cada **project** pode carregar múltiplos **apps**, como um projeto site de esportes que pode ter um app para os artigos, outro para rankings etc.<br>
Ainda no terminal usamos os comandos a seguir para criar o project **library** que vai carregar nosso app **books**. 

```bash
>django-admin startproject library . #ponto indica diretório atual
>django-admin startapp books
>python manage.py runserver #pra levantarmos o servidor local com a aplicação
```
Sua estrutura de pastas deve estar assim:

![imagem da estrutura](img/imagem-estrutura.jpg)

Para criar as tabelas no banco de dados (Por enquanto *Sqlite3*) executamos o comando
```bash
>python manage.py migrate
```
Isso evita que a notificação *unapplied migrations* apareça na próxima vez que você levantar o servidor 

![imagem unapplied](img/18unapplied.png)

# Criando os modelos e API
No arquivo **./library/settings.py** precisamos indicar ao nosso projeto library sobre a existência do app books e também o uso do rest framework. Portanto adicionamos as seguintes linhas sublinhadas

![imagem das linhas](img/library_settings.jpg)

Agora em **./library/books/models.py** iremos criar nosso modelo com os atributos que um livro deve ter.

```py
from django.db import models
from uuid import uuid4

class Books(models.Model):
    #criando os atributos do livro
    id_book = models.UUIDField(primary_key=True, default=uuid4, editable=False)
    title = models.CharField(max_length=255)
    author = models.CharField(max_length=255)
    release_year = models.IntegerField()
```
## Serializers e Viewsets
Dentro de **./library/books** iremos criar a pasta **/api** com os arquivos 
* serializers.py 
* viewsets.py 

### Serializers
```py
from rest_framework import serializers
from books import models

class BooksSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Books
        fields = '__all__' #todos os campos do model id_book, author..
```

### Viewsets
```py
from django.db import models
from uuid import uuid4

class Books(models.Model):
    #criando os atributos do livro
    id_book = models.UUIDField(primary_key=True, default=uuid4, editable=False)
    title = models.CharField(max_length=255)
    author = models.CharField(max_length=255)
    release_year = models.IntegerField()
```
# Criação das rotas
Agora com o viewset e o serializer a única coisa que falta é uma rota. Portanto vamos para **./library/urls.py** resolver esse problema

```py
from django.contrib import admin
from django.urls import path, include

from rest_framework import routers
from books.api import viewsets as booksviewsets
#criando nosso objeto de rota
route = routers.DefaultRouter()
route.register(r'books', booksviewsets.BooksViewSet, basename="Books")

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include(route.urls))
]
```
Como criamos um modelo novo lá em cima, precisamos avisar e em seguida migrar todos essas novas informações para o banco de dados

```bash
>python manage.py makemigrations 
>python manage.py migrate
>python manage.py runserver 
```
Agora você pode usar um programa como <a href="https://insomnia.rest/">Insomnia</a> para testar os métodos http no link do seu servidor local. 🥰

![insomnia](img/insomnia.png)

>O python facilita bastante coisas para a gente, como os serializers (que convertem objetos para strings na comunicação cliente-servidor) e os verbos http (GET, POST, PUT, DELETE) que de certa forma também vem por padrão. Não me aprofundei neles durante o readme porque também preciso entender melhor como essas coisas funcionam

## Deploy no heroku
<p>Irei dar o passo a passo presumindo que você acabou de clonar o repositório ou criar o seu próprio projeto de API
</p>
Antes de tudo precisamos:

* Iniciar um repositório git : git init
* Escrever um arquivo .gitignore a no seu diretório atual com   

```py
venv
*.pyc
.env
.DS_Store
```
Talvez você precise ignorar mais coisas. Usar um [template](https://github.com/github/gitignore/blob/master/Python.gitignore) de .gitignore para projetos python é uma boa nesses casos.

* Agora você precisa conectar o seu git no heroku. Para isso siga as linhas abaixo 

* Criar o arquivo Procfile ainda nessa pasta e preencher com o conteúdo abaixo 
```py
web: gunicorn library.wsgi #configuração do django para serviços web
```
library.wsgi é o nome do projeto + extensão wsgi. Você vai achar esse nome em **./library/settings.py**  
![nome_projeto](img/nome_projeto.png)

* Write on the terminal
```py
pip freeze > requirements.txt 
# Cria um arquivo txt com todas as bibliotecas que o servidor heroku vai precisar instalar
```
* Now you just have to setup your git to connect with heroku. Follow the lines below

```bash 
> heroku login #pressione qualquer tecla exceto Q e faça o login
> heroku create nome_do_projeto
> pip install django-heroku #lembre de estar com a venv ativada
> pip install guinicorn
> git add .
> git commit -m "Commited yay"
> heroku git:remote -a mywebapp
> git push heroku master #This part can take several minutes
> heroku ps:scale web=1 #your app is already deployed but this make sure will be only one runnig around
> heroku open # :D
```

# Getting Started
```bash
# Clone repository
git clone https://github.com/Mesheo/Biblioteca-API.git && cd Biblioteca-API

# Create Virtual Environment
python -m venv venv && ./venv/Scripts/Activate.ps1

# Install dependencies
pip install django djangorestframework

# Run Application
python manage.py runserver
```