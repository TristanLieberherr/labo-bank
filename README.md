# Labo Banque <!-- omit in toc -->

- **Durée**: 6 périodes + travail à la maison
- **Format**: travail en binôme

L'objectif de ce laboratoire est de créer une interface web pour interagir avec une banque numérique.

L'interface web se fera avec la bibliothèque [Flask](http://flask.pocoo.org/).
Il existe d'autres *framework* web pour Python comme [Django](https://www.djangoproject.com/), [Pyramid](https://trypyramid.com/), [CherryPy](http://cherrypy.org/) ou [Twisted](https://twistedmatrix.com/trac/), mais nous choisissons ici le plus simple.

## Concepts à découvrir

Les concepts à découvrir lors de ce labo sont les suivants :

- [Flask](https://flask.palletsprojects.com/en/1.1.x/installation/)
- JSON
- Base du langage HTML
- Templates avec Jinja2 (intégrées à Flask)
- Routes
- Méthodes HTTP GET et POST

## Cahier des charges

Un nouveau service internet se propose de gérer le portefeuille numérique de clients au travers de transactions simples. Chaque client peut émettre des transactions et consulter l'historique de ses transactions. Cette société n'ayant pas les moyens de payer des employés confie aux clients la responsabilité de gérer l'intégrité des données.

Un client accède à la plateforme via un navigateur web et peut se loguer avec sa connexion et son mot de passe.

Une fois connecté, l'utilisateur accède à une interface pour insérer des transactions bancaires.

Les fonctionnalités sont :

- saisir et une nouvelle transaction (*destinataire*, *montant*) ;
- refuser une transaction s'il ne reste plus d'argent au client ;
- consulter son compte ;
- consulter l'historique des transactions ;

La démarche proposée est la suivante :

1. Faire le tutoriel Flask (2 périodes)
2. Réaliser le travail demandé (4 périodes + travail à la maison)

## Tutoriel Flask

Créez un répertoire `tutorial` pour réaliser le tutoriel Flask.

### Flask

Le module [Flask](https://flask.palletsprojects.com/en/1.1.x/installation/) permet de créer un serveur web en Python. Voici un exemple minimal de serveur web qui affiche un message sur la page d'accueil :

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return "Hello, World!"

if __name__ == '__main__':
    app.run()
```

Flask utilise les décorateurs de Python pour associer une fonction à une route. Dans l'exemple ci-dessus, la fonction `index` est associée à la route `/`. Lorsque l'utilisateur se connecte à la page d'accueil, la fonction `index` est appelée et retourne le message `Hello, World!`.

Une route est une URL qui permet d'accéder à une page web. Si votre serveur est accessible depuis l'URL `http://localhost:5000`, la route `/` correspond à l'URL `http://localhost:5000/`. Vous pouvez par exemple ajouter une route pour obtenir une information différente `http://localhost:5000/the-answer` :

```python
@app.route('/the-answer')
def the_answer():
    return "42"
```

Le contenu retourné par une route peut être du HTML. Par exemple, la route `/` peut retourner une page HTML qui possède un bouton qui permet de connaître la réponse à la grande question sur la vie, l'univers et le reste :

```python
@app.route('/')
def index():
    return """
    <html>
        <body>
            <h1>Bienvenue</h1>
            <p>Voulez-vous connaître la réponse à la grande
            question sur la vie, l'univers et le reste?</p>
            <p>Cliquez <a href="/the-answer">ici</a></p>
        </body>
    </html>
    """
```

### Templates

Il n'est pas très pratique d'écrire du HTML directement dans le code Python. Flask permet d'utiliser des templates HTML. Pour cela, il est usuel de créer un dossier `templates` dans lequel on place les fichiers HTML. Par exemple, le fichier `templates/index.html` contient le code HTML suivant :

```html
<html>
    <head>
        <title>Bienvenue</title>
    </head>
    <body>
        <h1>Bienvenue</h1>
        <p>Voulez-vous connaître la réponse à la grande question sur la vie, l'univers et le reste?</p>
        <p>Cliquez <a href="/the-answer">ici</a></p>
    </body>
</html>
```

De la même manière, on peut créer un fichier `templates/the-answer.html` :

```html
<html>
    <head>
        <title>The answer</title>
    </head>
    <body>
        <style>
            .bounce {
                font-size:100px;
                font-weight: bold;
                color: #6F28B6;
                position:fixed;
                left:50%;
                bottom:0;
                margin-top:-25px;
                margin-left:-25px;
                -webkit-animation:bounce 2s infinite;
            }

            @-webkit-keyframes bounce {
                0%       { bottom:5px; }
                25%, 75% { bottom:120px; }
                50%      { bottom:90px; }
                100%     {bottom:0;}
            }
        </style>
        <div class="bounce">42</div>
    </body>
</html>
```

Pour utiliser ces templates, il faut utiliser la fonction `render_template` de Flask :

```python
from flask import Flask, render_template
app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/the-answer')
def the_answer():
    return render_template('the-answer.html')
```

Enfin, les templates peuvent utiliser des variables. Par exemple, si la réponse à la grande question sur la vie, l'univers et le reste est stockée dans une variable `answer`, on peut l'afficher dans le template `the-answer.html`. Il faut modifier la ligne suivante:

```html
<div class="bounce">42</div>
```

Par:

```html
<div class="bounce">{{ answer }}</div>
```

Et modifier la fonction `the_answer` pour passer la variable `answer` au template :

```python
@app.route('/the-answer')
def the_answer():
    return render_template('the-answer.html', answer=42)
```

### Authentification

Il est parfois nécessaire de n'autoriser que certaines personnes à accéder à certaines pages. Flask permet de mettre en place une authentification basique. Voici un exemple de serveur web qui n'autorise que les utilisateurs `alice` et `bob` à accéder à la page d'accueil :

``` python
from flask import Flask
from flask_httpauth import HTTPBasicAuth
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)
auth = HTTPBasicAuth()

users = {
    "alice": generate_password_hash("tricot"),
    "bob": generate_password_hash("cars")
}

@auth.verify_password
def verify_password(username, password):
    if username in users and \
            check_password_hash(users.get(username), password):
        return username

@app.route('/')
@auth.login_required
def index():
    return "Hello, {}!".format(auth.current_user())

if __name__ == '__main__':
    app.run()
```

### Personnalisation de l'interface

Pour personnaliser l'interface, il est possible d'utiliser des fichiers CSS et JavaScript. Une possibilité est d'utiliser la bibliothèque [Bootstrap](https://getbootstrap.com/).

Bootstrap se compose de deux parties:

1. Un fichier CSS qui permet de personnaliser l'interface
2. Un fichier JavaScript qui permet d'ajouter des fonctionnalités à l'interface

Pour utiliser Bootstrap, on va profiter de la puissance de Flask et Jinja2. Il est possible de lier plusieurs templates les unes aux autres. Par exemple, on peut créer un template `base.html` qui contient le code HTML commun à toutes les pages. Ce template contient des *placeholders* qui seront remplacés par le contenu des autres templates. Par exemple, le fichier `templates/base.html` contient le code HTML suivant :

```html
<!doctype html>
<html lang="en">
  <head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">

    <title>Hello, world!</title>
  </head>
  <body>
    {% include "nav.html" %}
    <div class="container">
        {% block content %}
        {% endblock %}
    </div>

    <!-- Optional JavaScript -->
    <!-- jQuery first, then Popper.js, then Bootstrap JS -->
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
  </body>
</html>
```

Notez les liens vers les fichiers CSS et JavaScript de Bootstrap. On pourrait également copier ces fichiers localement, mais il est plus simple de les charger depuis un CDN. Un CDN est un serveur qui permet de charger des fichiers (CSS, JavaScript, images, etc.) depuis un serveur distant. La plupart des sites internet utilisent des CDN pour accélérer le chargement des pages.

On peut également rajouter une barre de navigation en créant un fichier `templates/nav.html` contenant le code HTML suivant :

```html
<nav class="navbar navbar-expand-lg navbar-light bg-light">
    <a class="navbar-brand" href="#">{{ brand }}</a>
    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNav"
        aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item active">
                <a class="nav-link" href="/">Home <span class="sr-only">(current)</span></a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="/the-answer">Answer</a>
            </li>
        </ul>
    </div>
</nav>
```

Enfin, votre template `index.html` peut maintenant être plus simple:

```html
{% extends "base.html" %}
{% block content %}
<h1>Test</h1>
{% endblock %}
```

On voit qu'elle étend le template `base.html` et qu'elle remplace le contenu du *placeholder* `content` par celui du template.

Concernant la barre de navigation, vous pouvez utiliser Jinja pour afficher dynamiquement le nom de l'utilisateur connecté et le titre du site. Par exemple, vous pouvez associer `{{ brand }}` au nom de votre interface et rajouter par exemple à droite `{{ current_user }}` le nom de l'utilisateur connecté.

Vous pouvez également générer dynamiquement la navigation qui serait liée à un dictionnaire Python :

```python
nav = {
    'Home': '/',
    'Answer': '/the-answer'
}
```

### Échange de données entre le client et le serveur

Pour échanger des données entre le client et le serveur, on va utiliser des formulaires. Un formulaire est un ensemble de champs qui permettent de saisir des données.

Ces données sont transmises au serveur depuis votre navigateur sous forme de requête HTTP. Le serveur peut alors récupérer ces données et les traiter. Il existe deux types principaux de requêtes HTTP : `GET` et `POST`. La première utilise l'URL pour transmettre les données, la seconde utilise le corps de la transaction HTTP.

Créons une nouvelle route nommée `/multiply` qui va permettre de multiplier deux nombres. Pour cela, on va créer un formulaire qui va envoyer les données au serveur. Le formulaire sera composé de deux champs de type `number` et d'un bouton de type `submit`. Créez une template `multiply.html` qui contient le code HTML suivant :

```html
{% extends "base.html" %}
{% block content %}
<form method="GET" action="/multiply">
    <div class="form-group">
        <label for="number1">Number 1</label>
        <input type="number" class="form-control" id="number1" name="a" placeholder="Enter number 1">
    </div>
    <div class="form-group">
        <label for="number2">Number 2</label>
        <input type="number" class="form-control" id="number2" name="b" placeholder="Enter number 2">
    </div>
    <button type="submit" class="btn btn-primary">Compute</button>
</form>

{% if result %}
    <div class="alert alert-success" role="alert">
        {{ result }}
    </div>
{% endif %}

{% endblock %}
```

Vous pouvez ensuite modifier votre route `/multiply` pour récupérer les données du formulaire :

```python
@app.route('/multiply', methods = ['POST', 'GET'])
def multiply():
    if request.method == 'POST':
        a = request.form['a']
        b = request.form['b']
    else:
        a = request.args.get('a')
        b = request.args.get('b')
    result = None
    if a is not None and b is not None:
        result = int(a) * int(b)
    return render_template('multiply.html', brand="Démo", result=result)
```

On peut donner un autre exemple, qui affiche un tableau de données. Pour cela, on va créer une route `/table` qui va afficher un tableau de données construit depuis le *backend*. Par exemple, une table de logarithmes :

```python
@app.route('/log')
def log():
    data = [(n, math.log(n), math.log10(n), math.log2(n)) for n in range(1, 100)]
    return render_template('log.html', brand="Démo", data=data)
```

La template sera:

```html
{% extends "base.html" %}
{% block content %}

<h1>Table de logarithmes</h1>

<table class="table">
    <thead>
        <tr>
            <th scope="col">Valeur d'entrée</th>
            <th scope="col">Logarithme naturel</th>
            <th scope="col">Logarithme base 10</th>
            <th scope="col">Logarithme base 2</th>
        </tr>
    </thead>
    <tbody>
        {% for item in data -%}
        <tr>
            {% for i in item %}
            <td>{{ i }}</td>
            {%- endfor %}
        </tr>
        {%- endfor %}
    </tbody>
</table>
{% endblock %}
```

## Travail à réaliser

### Gestion des données

Les données côté serveur sont stockées dans un fichier JSON faisant office de base de données.

Voici un exemple pour lire un fichier JSON fourni dans le répertoire `db` :

```python
>>> import json
>>> with open("db/users.json", "r") as f:
>>>     data = json.load(f)
>>> data[0]['name']
Bob
```

#### Utilisateurs

La base de données comportera une liste d'utilisateurs ayant les champs suivants :githubg

```json
[
    {
        "name": "Bob",
        "password": "pbkdf2:sha256:600000$dFQYwNkjS8rGfmg7$51d08c1237d008fe7e25863adff5af12a0d442ce91bc1bc17cad67b52bf25570",
        "last_connection": 1581884311.560531
    },
    {
        "name": "Alice",
        "password": "pbkdf2:sha256:600000$FCar26XKpvjyEcz8$9d94e77422bd45010fea967759cb31037207d86dab02fbd06e22d7913f7555bb",
        "last_connection": 1581884311.560531
    },
    {
        "name": "HEIG-VD",
        "password": "",
        "last_connection": null
    },
]
```

Pour simplifier le travail, le nom est également le login de la personne.

Le mot de passe est la version hachée du mot de passe en utilisant l'algorithme `pbkdf2:sha256`. Ceci permet de ne pas stocker directement en clair un mot de passe dans la base de données. Lorsque le client se logue, il envoie son mot de passe et ce dernier est haché immédiatement. Il suffit de comparer si le hash du mot de passe est le même que celui contenu dans la base de données pour authentifier un utilisateur. Notons que le mot de passe est salé pour renforcer la sécurité (c.f. [salage](https://fr.wikipedia.org/wiki/Salage_(cryptographie))).

Pour la gestion du mot de passe, vous pouvez utiliser la bibliothèque [Werkzeug](https://pypi.org/project/Werkzeug/) :

```python
>>> from werkzeug.security import generate_password_hash, check_password_hash
>>> password = "1234"
>>> hash = generate_password_hash(password)
'pbkdf2:sha256:600000$rhudl4khhKQvVM8c$a37f07ecb63b98ba36b89bea44649668def9a232850afd410ab4783156cb8758'
```

Dans notre exemple, le mot de passe de nos utilisateurs est le nom de nos utilisateurs (`bob` pour Bob et `alice` pour Alice).

Le champ `last_connection` renseigne sur l'heure de la dernière connexion de l'utilisateur. Les *timestamp* sont calculés de la façon suivante :

```python
>>> from datetime import datetime
>>> now = datetime.now()
>>> timestamp = datetime.timestamp(now)
1619040784.132735
```

Dans ce travail pratique, vous aurez besoin d'au moins 2 utilisateurs, nous avons choisi **Bob** et **Alice** ([Alice et Bob](https://fr.wikipedia.org/wiki/Alice_et_Bob)). Sentez-vous libre d'en ajouter d'autres.

#### Transactions

Les transactions sont stockées dans un fichier JSON faisant office de base de données. Voici un exemple de transaction contenu dans `db/transactions.json` :

```json
[
    {
        "sender": "HEIG-VD",
        "recipient": "Alice",
        "amount": 5000
    },
    {
        "sender": "HEIG-VD",
        "recipient": "Bob",
        "amount": 1000
    }
    {
        "sender": "Bob",
        "recipient": "Alice",
        "amount": 50
    },
    {
        "sender": "Alice",
        "recipient": "Bob",
        "amount": 10
    }
]
```

### Routes

Ce serveur offrira les routes suivantes qui seront toutes protégées par une authentification :

Pages HTML :

- `/` Page d'accueil.
- `/add` Permets d'ajouter une nouvelle transaction. Cette page HTML propose un formulaire pour insérer une transaction. Une liste déroulante ([select](https://www.w3schools.com/tags/tag_select.asp) permet de sélectionner le destinataire et un champ permet de saisir le montant. Une erreur est affichée en cas de solde insuffisant. La nouvelle transaction est insérée dans le fichier JSON.
- `/transactions` Affiche sous forme de table, toutes les transactions de l'utilisateur connecté ainsi que le solde de son compte.

### Classe `Bank`

Toute la logique des transactions est gérée par une classe `Bank` qui est instanciée dans le module principal. Cette classe permet:

- De générer une transaction
- Lister les utilisateurs
- Vérifier si un utilisateur est authentifié
- Interdire les transactions si un utilisateur n'a pas le solde nécessaire

```python
class Bank:
    def __init__(self, users, transactions):
        # Charge la liste des utilisateurs
        # Charge la liste des transactions
    def transaction(self, sender, recipient, amount):
        # Génère une transaction
        # Retourne une erreur si le solde est insuffisant
    def users(self):
        # Retourne la liste des utilisateurs (sans les mots de passes)
    def is_authenticated(self, name, password):
        # Vérifie si un utilisateur est authentifié
    def balance(self, name):
        # Retourne le solde d'un utilisateur
    def transactions(self, name):
        # Retourne la liste des transactions d'un utilisateur
```

### Script principal

Votre module dispose d'un exécutable (qui utilise `click`), voir [ici](https://flask.palletsprojects.com/en/1.1.x/cli/#custom-scripts). Cet exécutable permet de démarrer le serveur. Le script principal contient le serveur Flask et est contenu dans `__main__.py`.

Documentez-vous comment créer un serveur Flask avec Click. L'objectif est de pouvoir démarrer le serveur avec la commande suivante :

```bash
$ python -mserver run
```

Ce module permet également de consulter les mêmes données que les pages HTML mais sous forme de JSON. Par exemple, pour consulter les transactions :

```bash
$ python -mserver transactions
```

Ou alors la liste des utilisateurs:

```bash
$ python -mserver users
```

Ou de générer une transaction :

```bash
$ python -mserver transfert --sender=Alice --recipient=Bob --amount=100
```

## Checklist

À l'issue de ce travail pratique, votre module `server` doit :

- [ ] Avoir un fichier `__init__.py` qui contient la classe `Bank`
- [ ] Avoir un fichier `__main__.py` qui contient le serveur Flask et l'application Click
- [ ] Le serveur web
  - [ ] Doit avoir une page d'accueil
  - [ ] Doit avoir une barre de navigation
  - [ ] Doit utiliser Bootstrap
  - [ ] Doit avoir une page pour ajouter une transaction
  - [ ] Doit avoir une page pour consulter les transactions
- [ ] Le script principal doit pouvoir être appelé avec:
  - [ ] `python -mserver run` pour démarrer le serveur
  - [ ] `python -mserver users` pour consulter la liste des utilisateurs
  - [ ] `python -mserver transactions` pour consulter la liste des transactions
  - [ ] `python -mserver transfert --sender=Alice --recipient=Bob --amount=100` pour générer une transaction
