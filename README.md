In this workshop, we will create and deploy a python web application that will allow you to track companies stock information. If, at any time, you get lost or get behind, follow the instructions below as we will be going through this during the workshop. You can also come back to it after the workshop to go through it.

## Pre-requisites

There are some prerequisites that need to be completed prior to starting this workshop. Have your parent ensure that each of these is met and if not, they should download or create these accounts for you. There is information below in case you ever decide to come back and complete this workshop.

### Install python
Since this is a python workshop, you will need python installed on your system. Your parent can get that here: https://www.python.org/downloads/

### Create a github account
Your parent should have created a github account for you. If not, please do so with their supervision. Here is the link: https://github.com/. You cannot complete the workshop without a github account.

### Install git
In order to start, we will need some sort of source control for our code. Source control is the practice of tracking and managing changes to your code.
It provides us a history of development and helps resolve conflicts when merging contributions from multiple people. Every good development team has some way of managing
source control. The reason source control is so important is that it allows developers to maintain a single source of truth. It helps to facilitate collaboration
and helps accelerate releases. Many developrs can work on the same codebase at the same time without stepping on each other's toes.

One very popular source control management system is git, which is what we will be using for this workshop. So, make sure git is installed on your system.

#### MacOS
Open up a Terminal window by typing "Terminal" in your search bar on your Mac. Then, install Homebrew (if not already installed) by running the following command:
```
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)" 
```
Next, we are ready to install git by running this command: 
```
$ brew install git
```

### Create a Finnhub account:
To create our stock dashboard, we will need to get information via a REST API provided by Finnhub. Finnhub gives free access to some of their APIs and those will be sufficient
for us today. I will go into more detail about REST APIs below. But first, make sure your parent has created a Finnhub account for you. If not, have them create one for you
here: https://finnhub.io/register. 

Once the account is created, we only need the API token so can call the APIs. Once logged in, head to https://finnhub.io/dashboard and you will see a box with
the API key. Let's save that key somewhere that is readily accessible. 

### Create a Heroku account:
Heroku is where we will deploy our web application so that we can share it with our friends! You parent should have create an account for you already. If not,
please have them do so right now here: https://signup.heroku.com/. Only the free account is necessary for this workshop.

### Install Heroku CLI
You will need the Heroku CLI to be able to deploy the app. You can find instructions on downloading that here: https://devcenter.heroku.com/articles/heroku-cli#download-and-install. 
### PostgreSQL
PostgreSQL is an open source relational database that we will be using to store our data. Make sure that is downloaded on your computer as well. If not,
have your download it here: https://www.postgresql.org/download/

## Let's start coding!

### Project Setup

#### Let's create a directory to work in
```
$ mkdir stock-dashboard && cd stock-dashboard
```

#### Initialize a new git repository
```
$ git init
```
At this step, you may need to use the github credentials to log in if this is the first time you're doing this.

#### In the directory, create a virtual environment that we can use for the application
```
$ python3.7 -m venv env
$ source env/bin/activate
```

#### Let's set up some basics
```
$ touch app.py .gitignore requirements.txt
```

#### Copy and paste the following into .gitignore
```
env/
__pycache__/
```
#### Copy and paste the following into requirements.txt
```
alembic==1.4.3
certifi==2020.6.20
chardet==3.0.4
click==7.1.2
Flask==1.1.1
Flask-Migrate==2.5.2
Flask-Script==2.0.6
Flask-SQLAlchemy==2.4.1
gunicorn==20.0.4
idna==2.10
itsdangerous==1.1.0
Jinja2==2.11.2
Mako==1.1.3
MarkupSafe==1.1.1
psycopg2-binary==2.8.6
python-dateutil==2.8.1
python-editor==1.0.4
requests==2.24.0
six==1.15.0
SQLAlchemy==1.3.20
urllib3==1.25.10
Werkzeug==1.0.1
```

#### We need to install a lot of packages to get our app running. 
```
$ python -m pip install -r requirements.txt
```

#### Open up app.py in your favorite IDE and add the following code
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def dashboard():
    return "hello"

if __name__ == "__main__":
    app.run()
```

#### Run the app now
```
$ python app.py
```

#### Navigate to https://127.0.0.1:5000 to view your app

### Heroku Setup
You should already have heroku and Heroku CLI installed on your computer. Login with the credentials after running the below command.
```
$ heroku login
```

#### Create a Procfile in the current directory(stock-dashboard, folder where we initialized the git repo)
```
$ touch Procfile
```

#### Add the following line
```
web: gunicorn app:app
```

#### We also need to specify a Python version so that Heroku uses the correct runtime to run the app with. So create a file called runtime.txt:
```
$ touch runtime.txt
$ echo "python-3.8.1" > runtime.txt
```

#### Now, we're ready to create the Heroku app and add a remote to our local repository
```
$ heroku create
$ heroku git:remote -a <APP-NAME>
```

#### Let's push the code we have now
```
$ git add .
$ git commit -m "first commit"
$ git push heroku master
```
You should get a link that you can now navigate to and be able to see your app!

#### We'll need to setup a config file to define some environment variables.
Create a file called config.py and add the following code:
```
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    DEBUG = False
    TESTING = False
    CSRF_ENABLED = True

class ProductionConfig(Config):
    DEBUG = False
```

#### Let's create an environment variable that tells us which config to use
```
$ export APP_SETTINGS=config.ProductionConfig
$ heroku config:set APP_SETTINGS=config.ProductionConfig
```

#### We need to make sure the app uses the correct config. So update app.py so that it looks like this:
```
import os
from flask import Flask
app = Flask(__name__)

app.config.from_object(os.environ['APP_SETTINGS'])

@app.route("/")
def dashboard():
    return "hello"

if __name__ == "__main__":
    app.run()
```
Commit and push these changes now

### PostgreSQL setup

#### Create the database by using the PostgreSQL app that was listed as a prerequisite
```
# create database stocks;
# \q
```

#### Update config.py to add the database URL:
```
import os

class Config(object):
    ...
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']
```

#### config.py should look like this:
```
import os
basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    DEBUG = False
    TESTING = False
    CSRF_ENABLED = True
    SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class ProductionConfig(Config):
    DEBUG = False
```

#### Set the local environment variable so that the app can find the correct databse
```
$ export DATABASE_URL="postgresql:///stocks"
```

#### In app.py, we can connect to the databse
```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config.from_object(os.environ['APP_SETTINGS'])
db = SQLAlchemy(app)

from models import Stock

@app.route("/")
def dashboard():
    return "hello"

if __name__ == "__main__":
    app.run()
```

#### Next, let's create our data model. Create a file called models.py and copy in the following code:
```
from app import db
from sqlalchemy.dialects.postgresql import JSON

class Stock(db.Model):
    __tablename__ = "stocks"

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String())

    def __init__(self, name):
        self.name = name
    
    def __repr__(self):
        return "<id {}>".format(self.id)
```

#### Create a new file called manage.py
```
import os
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand

from app import app, db
app.config.from_object(os.environ['APP_SETTINGS'])

migrate = Migrate(app, db)
manager = Manager(app)

manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()
```

#### Now, run the following:
```
$ python manage.py db init
$ python manage.py db migrate
$ python manage.py db upgrade
```
Our database is now ready!

#### We will need to do the same on Heroku and push our changes:
```
$ heroku addons:create heroku-postgresql:hobby-dev
```

### Add features to the app

Create a folder called templates and a file inside it called index.html. Paste the following code into index.html
```
<!DOCTYPE html>
<html>

<head>
    <link rel="stylesheet" type="text/css" href="{{url_for('static', filename='styles/index.css')}}">
</head>

<body>
    <div class="row">
        <h1>Dashboard</h1>
            <h2>Add a company</h2>
            <form action="{{ url_for('dashboard') }}" method="post">
                <input type="text" id="company" name="company"><input type="submit" value="Submit">
            </form>
            {% if select_companies %}
            <p>Found multiple matching, select from the list below the company whose stock you want to add to your
                watchlist:</p>
            <form action="{{ url_for('dashboard') }}" method="post">
                {% for company in select_companies %}
                <input id={{company.description}} type="radio" name="ticker" value="{{company.symbol}}">
                <label for={{company.description}}>{{company.description}}</label><br>
                {% endfor %}
                <input type="submit" value="Submit">
            </form>
            {% endif %}
            <h2>Watchlist:</h2>
            {% if data %}
            {% for stock in data %}
            <div class="card">
                <h4><a href={{stock.weburl}}>{{stock.name}}:</a></h4>
                <p>Share price: ${{stock.stock_price}}</p>
                {% if stock.news %}
                <ul>
                    {% for article in stock.news %}
                    <li><a href={{article.link}}>{{article.headline}}</a></li>
                    {% endfor %}
                </ul>
                {% endif %}
            </div>
            {% endfor %}
            {% endif %}

    </div>
</body>

</html>
```

Let's also create a stylesheet for the template. Create a folder called static, then another in static called styles. Now, create a file called index.css and add the following:
```
body {
    color: rgb(39, 247, 212);
    border:2px rgb(39, 247, 212);

}
.row {
    padding-left: 20px;
}
.row:after {
    content: "";
    display: table;
    clear: both;
}
a {
    color: rgb(39, 247, 212);
}

input[type="submit" i] {
    color: rgb(39, 247, 212);
    background-color: white;
    border: rgb(39, 247, 212);
}
.column {
    float: left;
    width: 50%;
  }
  
  /* Clear floats after the columns */
  .row:after {
    content: "";
    display: table;
    clear: both;
}
h1 {
    text-align: center;
}
.card {
    /* Add shadows to create the "card" effect */
    box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2);
    transition: 0.3s;
    padding: 10px;
  }
  
  /* On mouse-over, add a deeper shadow */
  .card:hover {
    box-shadow: 0 8px 16px 0 rgba(0,0,0,0.2);
  }
  
  /* Add some padding inside the card container */
  /* .container {
    padding-left: 10px;
    padding: 2px 16px;
  } */
```
This will create our front end for the app.

Open up app.py and let's see how we can add a stock to our dashboard. Open up app.py, it should look like this:
```
import os
from flask import Flask
from flask import render_template
from flask import request
from flask_sqlalchemy import SQLAlchemy
import json
import os 
import requests

app = Flask(__name__)
app.config.from_object(os.environ['APP_SETTINGS'])
db = SQLAlchemy(app)

from models import Stock

API_TOKEN = os.environ['API_TOKEN']

@app.route("/", methods=["GET","POST"])
def dashboard():
    select_companies = []
    if request.method == "POST":
        if request.form.get("ticker", None):
            # the user gave us the exact ticker, let's just add it IFF it doesn't exist already
            ticker = request.form["ticker"]
            stocks = Stock.query.filter(Stock.name == ticker).all()
            if (len(stocks)) == 0:
                stock = Stock(name=ticker)
                db.session.add(stock)
                db.session.commit()
        elif request.form.get("company", None):
            response = requests.get(f"https://finnhub.io/api/v1/stock/symbol?exchange=US&token={API_TOKEN}")
            resp = json.loads(response.text)
            #we may get multiple matches, let's have user select which one they wanted to add
            for item in resp:
                if request.form["company"].upper() in item["description"]:
                    select_companies.append(item)
            if len(select_companies) == 0:
                #normally, we would raise an error like a 404 here, but for now let's just pass
                pass
            elif len(select_companies) == 1:
                # we got just one match, let's commit to db
                stocks = Stock.query.filter(Stock.name == select_companies[0]["symbol"]).all()
                if len(stocks) == 0:
                    stock = Stock(select_companies[0]["symbol"])
                    db.session.add(stock)
                    db.session.commit()
    # always need to do the get so the page stays updated
    stocks = Stock.query.all()
    data = []
    for stock in stocks:
        _data = {
            "name" : stock.name,
            "news": []
        }
        resp = requests.get(f"https://finnhub.io/api/v1/quote?symbol={stock.name}&token={API_TOKEN}").json()
        _data["stock_price"] = resp["c"]
        resp = requests.get(f"https://finnhub.io/api/v1/stock/profile2?symbol={stock.name}&token={API_TOKEN}").json()
        _data["weburl"] = resp["weburl"]
        resp = requests.get(f"https://finnhub.io/api/v1/company-news?symbol={stock.name}&from=2020-11-05&to=2020-11-10&token={API_TOKEN}").json()
        for article in resp:
            _news = {
                "headline": article["headline"],
                "link": article["url"]
            }
        _data["news"].append(_news)
        data.append(_data)
    if len(select_companies) > 1:
        return render_template("index.html", data=data, select_companies=select_companies)
    return render_template("index.html", data=data)

if __name__ == '__main__':
    app.run()
```

There's an environment variable we need to set here. When your parent created a Finnhub account, they were provided an API token. Navigate to https://finnhub.io/dashboard and get that API token. Then, do the following:

```
$ export API_TOKEN=<API-TOKEN>
$ heroku config:set API_TOKEN=<API-TOKEN>
```

Commit and push the changes, then run:
```
$ heroku run python manage.py db upgrade
```

Your app is ready to use!
