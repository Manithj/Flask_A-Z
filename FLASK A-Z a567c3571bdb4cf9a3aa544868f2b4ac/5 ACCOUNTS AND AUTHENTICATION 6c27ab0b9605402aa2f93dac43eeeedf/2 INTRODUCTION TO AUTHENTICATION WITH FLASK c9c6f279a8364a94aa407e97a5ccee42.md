# 2. INTRODUCTION TO AUTHENTICATION WITH FLASK

### **Intro to Authentication**

Authentication is the process of verifying that an individual has permission to perform an action. Without authentication, there would be no way of knowing or enforcing access control on our browser for our applications.

Our strategy of authenticating users depends on discerning whether a password is valid or not in order to allow the user to perform further actions in the application.

In the next lesson we’ll get to know some of the tools that we can use to authenticate in Python Flask applications by building a web application that allows authenticated users to view an awesome secret recipe.

### **Meet Flask-Login**

When building a web application we might first start with the base of our application serving an endpoint saying “Hello World”.

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello Authentication World!'
```

The application we will be building will show how to use tools in Flask to authenticate users. The primary tool we can use to achieve our purposes of authenticating in Flask is [Flask-Login](https://flask-login.readthedocs.io/en/latest/).

Flask-Login is a third-party package that allows us to use pieces of code that enable us to perform authentication actions in our application.

We can manage user logins with the `LoginManager` object from within Flask-Login, as shown below:

```python
from flask_login import LoginManager

login_manager = LoginManager()
```

- `LoginManager` is imported from the `flask_login` package
- a new `LoginManager` object named `login_manager` is created

Once a `LoginManager` object is defined, we need to initialize the manager with our application. This can be done with the `init_app()` [method](https://www.codecademy.com/resources/docs/general/method) of a `LoginManager`:

```python
login_manager.init_app(app)
```

- our instance of `LoginManager`, `login_manager`, calls its `init_app()` method with `app`, an initialized Flask app, as an [argument](https://www.codecademy.com/resources/docs/general/argument)

app.py

```python
from flask import Flask
# import LoginManager here:
from flask_login import LoginManager

app = Flask(__name__)

# create login_manager here:
login_manager = LoginManager()
# initialize login_manager here:
login_manager.init_app(app)

@app.route('/')
def hello_world():
    return 'Hello, Authentication World!'
```

![Untitled](2%20INTRODUCTION%20TO%20AUTHENTICATION%20WITH%20FLASK%20c9c6f279a8364a94aa407e97a5ccee42/Untitled.png)

### **Protecting Pages**

Protecting pages is the primary objective of authentication. We can leverage some very useful functions from Flask-Login to ensure our different pages or routes are protected.

One of the key pieces of code that we previously added is the `LoginManager` object that we initialized with our instance of the Flask application. `LoginManagers` have a [method](https://www.codecademy.com/resources/docs/general/method) `user_loader` that needs to be defined in order to load and verify a user from our [database](https://www.codecademy.com/resources/docs/general/database).

```python
@login_manager.user_loader
def load_user(id):
    return User.query.get(int(id))

```

- this method retrieves our `User` with an id value `id` from our database
- without this function, we won’t be able to verify users on our protected routes!

Next we need to import the `login_required` function from `flask_login` at the top of our file:

```python
from flask_login import login_required
```

We can now add the `@login_required` function as a decorator to different routes to make logging in necessary.

```python
@app.route('/home')
@login_required
def home():
    return render_template('logged_in.html')
```

The `@login_required` decorator will force the user to login before being able to view the page

app.py

```python
import flask
from flask_sqlalchemy import SQLAlchemy
# import login_required here:
from flask_login import LoginManager, UserMixin, login_required
from flask import request, render_template, flash, redirect,url_for
from datetime import datetime
from werkzeug.security import generate_password_hash, check_password_hash

app = flask.Flask(__name__)
login_manager = LoginManager()
login_manager.init_app(app)
db = SQLAlchemy(app)

class User(UserMixin,db.Model):
  id = db.Column(db.Integer, primary_key=True)
  username = db.Column(db.String(64), index=True, unique=True)
  email = db.Column(db.String(120), index=True, unique=True)
  password_hash = db.Column(db.String(128))
  posts = db.relationship('Post', backref='author', lazy='dynamic')

  def __repr__(self):
    return '<User {}>'.format(self.username)

  def set_password(self, password):
    self.password_hash = generate_password_hash(password)

  def check_password(self, password):
    return check_password_hash(self.password_hash, password) 

@login_manager.user_loader
def load_user(id):
  return User.query.get(int(id))
    
@app.route('/')
def index():
	return render_template('index.html')

@app.route('/home')
# add login_required decorator here:
@login_required
def home():
	return render_template('logged_in.html')
```

![Untitled](2%20INTRODUCTION%20TO%20AUTHENTICATION%20WITH%20FLASK%20c9c6f279a8364a94aa407e97a5ccee42/Untitled%201.png)

### **Error Handling**

We’ve all experienced a time when we thought we were logged into a site and tried to access a protected page. Some sites handle this better than others, by letting the user know that the requested page is only for authenticated users.

When our user tries to access protected pages without logging in or encounters an error upon login, its best we communicate this somehow to the user.

We can catch authorization issues by adding a new route or endpoint with the `@login_manager.unauthorized_handler` decorator:

```python
@login_manager.unauthorized_handler
def unauthorized():
  # do stuff
  return "Sorry you must be logged in to view this page"
```

- the `@login_manager.unauthorized_handler` decorator ensures that any time there is an authorization issue, the `unauthorized()` route is called
- the message in the `return` statement is HTML that is served to non-authenticated users. We can replace this with a template that users who fail to login see.

app.py

```python
import flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager, UserMixin, login_required
from flask import request, render_template, flash, redirect,url_for
from datetime import datetime
from werkzeug.security import generate_password_hash, check_password_hash

app = flask.Flask(__name__)
app.secret_key = 'secretkeyhardcoded'
login_manager = LoginManager()
login_manager.init_app(app)
db = SQLAlchemy(app)

class User(UserMixin,db.Model):
  id = db.Column(db.Integer, primary_key=True)
  username = db.Column(db.String(64), index=True, unique=True)
  email = db.Column(db.String(120), index=True, unique=True)
  password_hash = db.Column(db.String(128))
  posts = db.relationship('Post', backref='author', lazy='dynamic')

  def __repr__(self):
    return '<User {}>'.format(self.username)

  def set_password(self, password):
    self.password_hash = generate_password_hash(password)

  def check_password(self, password):
    return check_password_hash(self.password_hash, password) 

@login_manager.user_loader
def load_user(id):
  return User.query.get(int(id))
    
@app.route('/')
def index():
	return render_template('index.html')

@app.route('/home')
@login_required
def home():
	return render_template('logged_in.html')

# add a decorator here to handle unauthorized users:
@login_manager.unauthorized_handler
def unauthorized():
  # do stuff
  return "Sorry you must be logged in to view this page"
```

### **Logging in a User**

Best practices for user authentication using Flask is to make it hard for someone to use a stolen credential.

To achieve this in Flask use the Flask’s Werkzeug library which has `generate_password_hash` [method](https://www.codecademy.com/resources/docs/general/method) to generate a hash, and `check_password_hash` method to compare login input with the value returned from the `check_password_hash` method.

Our login code will check whether the value passed in is the same as the hardcoded user we are using to emulate a [database](https://www.codecademy.com/resources/docs/general/database).

We create a `User` class to represent a user. This object takes advantage of `UserMixin` (Mixins are prepackaged code of common code needs). In this case we use `UserMixin` because it allows us to take advantage of common user account functions without having to write it all ourselves from scratch.

The code below is the logic we use to log a user in if their password is correct.

```python
@app.route('/', methods=['GET', 'POST'])
def index():
  if flask.request.method == 'GET':
    return '''
    <p>Your credentials:
    username: TheCodeLearner
    password: !aehashf0qr324*&#W)*E!
    </p>
               <form action='/' method='POST'>
                <input type='text' name='email' id='email' placeholder='email'/>
                <input type='password' name='password' id='password' placeholder='password'/>
                <input type='submit' name='submit'/>
               </form>
               '''
  email = "TheCodeLearner"
  if flask.request.form['password'] == "!aehashf0qr324*&#W)*E!":
    user = User(email="TheCodeLearner@gmail.com", username="TheCodeLearner",password="!aehashf0qr324*&#W)*E!")
    login_user(user)
    return render_template("logged_in.html", current_user=user )
  return login_manager.unauthorized()
```

Take a look at the second conditional:

```python
if flask.request.form['password'] == "!aehashf0qr324*&#W)*E!":
```

Here, we’re checking that the form was submitted with a password that has the value `"!aehashf0qr324*&#W)*E!"`. If the password matches `"!aehashf0qr324*&#W)*E!"` exactly, then we can create a new `User` instance with the properties specified above and save the object to `user`. We then use the `login_user(user)` to load the newly created `User` instance. Once logged in, we can load the proper page using `render_template("logged_in.html", current_user=user)`. If the password isn’t correct, we return `login_manager.unauthorized()`.

app.py

```python
import flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager, UserMixin, login_required, login_user
from flask import request, render_template, flash, redirect,url_for
from datetime import datetime
from werkzeug.security import generate_password_hash, check_password_hash

app = flask.Flask(__name__)
app.secret_key = 'secretkeyhardcoded'
login_manager = LoginManager()
login_manager.init_app(app)
db = SQLAlchemy(app)

class User(UserMixin,db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)

@login_manager.user_loader
def load_user(id):
    return User.query.get(int(id))
    
@app.route('/', methods=['GET', 'POST'])
def index():
  if flask.request.method == 'GET':
    return '''
    <p>Your credentials:
    username: TheCodeLearner
    password: !aehashf0qr324*&#W)*E!
    </p>
               <form action='/' method='POST'>
                <input type='text' name='email' id='email' placeholder='email'/>
                <input type='password' name='password' id='password' placeholder='password'/>
                <input type='submit' name='submit'/>
               </form>
               '''
  email = "TheCodeLearner"
  if flask.request.form['password'] == "!aehashf0qr324*&#W)*E!":
    user = User(email="TheCodeLearner@gmail.com", username="TheCodeLearner",password="!aehashf0qr324*&#W)*E!")
    login_user(user)
    return render_template("logged_in.html", current_user=user )
  return login_manager.unauthorized()

@app.route('/home')
@login_required
def home():
	return render_template('logged_in.html')

@login_manager.unauthorized_handler
def unauthorized():
    # do stuff
    return "You are not logged in. Click here to get <a href="+ str("/")+">back to Landing Page</a>"
```

![Untitled](2%20INTRODUCTION%20TO%20AUTHENTICATION%20WITH%20FLASK%20c9c6f279a8364a94aa407e97a5ccee42/Untitled%202.png)

### **Show Logged in user**

In the previous exercise, we were able to write the login code. Now in this section, we will show the information related to the logged-in user.

Let’s zoom into this code: Notice how we pass in user into the current_user object. We will be using that current_user object in our HTML.

```python
  user = User(email="TheCodeLearner@gmail.com", username="TheCodeLearner",password="!aehashf0qr324*&#W)*E!")
    login_user(user)
    return render_template("logged_in.html", current_user=user )
  return 'Bad login'
```

Now when a user logs in successfully they are sent to a page showing our logged-in info. Most likely in our application, we will be serving dynamic pages of HTML. We can use Jinja templates to render the data from the backend. To display the user, pass it in from the endpoint and access that variable in our HTML.

```python
<h1>Welcome to Our Home Page</h1>

<p>Welcome back {{current_user.username}}</p>

<a class="blue pull-left" href="{{ url_for('index') }}">back</a>
```

This will enable our users to see their data when they log in!

logged_in.html

```html
<center><h1>The Ultimate Pancake</h1>
<a class="blue pull-left" href="{{ url_for('index') }}">Home</a></center>
<!-- Add your code below -->
<p>👋 {{current_user.username}}, thanks for checking this page out!</p>

<p>3 avacados - peeled, pitted, and mashed
  1 lime, juiced
  1 teaspoon salt 
  1/2  cup diced onion
  3 tablespoons chopped fresh cilantro
  2 roma (plum) tomatoes, diced
  1 teaspoon minced garlic
  1 pinch ground cayenne pepper
</p>

<li>In a large bowl, sift together the flour, baking powder, salt and sugar. Make a well in the center and pour in the milk, egg and melted butter; mix until smooth.</li>
<li>Heat a lightly oiled griddle or frying pan over medium high heat. Pour or scoop the batter onto the griddle, using approximately 1/4 cup for each pancake. Brown on both sides and serve hot.</li>
```

app.py

```python
import flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager, UserMixin, login_required, login_user
from flask import request, render_template, flash, redirect,url_for
from datetime import datetime
from werkzeug.security import generate_password_hash, check_password_hash

app = flask.Flask(__name__)
app.secret_key = 'secretkeyhardcoded'
login_manager = LoginManager()
login_manager.init_app(app)
db = SQLAlchemy(app)

class User(UserMixin,db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)

@login_manager.user_loader
def load_user(id):
    return User.query.get(int(id))
    
@app.route('/', methods=['GET', 'POST'])
def index():
  if flask.request.method == 'GET':
    return '''
    <p>Your credentials:
    username: TheCodeLearner
    password: !aehashf0qr324*&#W)*E!
    </p>
               <form action='/' method='POST'>
                <input type='text' name='email' id='email' placeholder='email'/>
                <input type='password' name='password' id='password' placeholder='password'/>
                <input type='submit' name='submit'/>
               </form>
               '''
  email = "TheCodeLearner"
  if flask.request.form['password'] == "!aehashf0qr324*&#W)*E!":
    user = User(email="TheCodeLearner@gmail.com", username="TheCodeLearner",password="!aehashf0qr324*&#W)*E!")
    login_user(user)
    return render_template("logged_in.html", current_user=user )
  return login_manager.unauthorized()

@app.route('/home')
@login_required
def home():
	return render_template('logged_in.html')

@login_manager.unauthorized_handler
def unauthorized():
    # do stuff
    return "You are not logged in. Click here to get <a href="+ str("/")+">back to Landing Page</a>"
```

### **Logout**

You’ve successfully authenticated, Nice. Now only logged in users can see our zesty recipe! But now that you’re done you don’t want to remain logged in. Let’s enable the logout feature to protect the recipe.

Flask-login provides us with a `logout_user` [method](https://www.codecademy.com/resources/docs/general/method) to facilitate this.

```python
@app.route("/logout")
@login_required
def logout():
    logout_user()
    return redirect(url_for('index'))
```

Awesome! We have our `logout()` function set up to call `flask-login`‘s `logout_user()` function to log out the user. Then we redirect the user to go back to the [index](https://www.codecademy.com/resources/docs/general/database/index) page. Let’s implement a logout link in our HTML to trigger the logout code to run.

in **logged_in.html** update the code with the logout link:

```html
<!DOCTYPE>
<head>
</head>
<body>
<a href="{{ url_for('logout') }}">Logout</a>
</body>
```

When a user clicks the logout link, they’ll call the `logout()` function we just created.

app.py

```python
import flask
from flask_sqlalchemy import SQLAlchemy
# Import logout_user below:
from flask_login import LoginManager, UserMixin, login_required, login_user, logout_user
from flask import request, render_template, flash, redirect,url_for
from datetime import datetime
from werkzeug.security import generate_password_hash, check_password_hash

app = flask.Flask(__name__)
app.secret_key = 'secretkeyhardcoded'
login_manager = LoginManager()
login_manager.init_app(app)
db = SQLAlchemy(app)

class User(UserMixin,db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)

@login_manager.user_loader
def load_user(id):
    return User.query.get(int(id))
    
@app.route('/', methods=['GET', 'POST'])
def index():
  if flask.request.method == 'GET':
    return '''
    <p>Your credentials:
    username: TheCodeLearner
    password: !aehashf0qr324*&#W)*E!
    </p>
               <form action='/' method='POST'>
                <input type='text' name='email' id='email' placeholder='email'/>
                <input type='password' name='password' id='password' placeholder='password'/>
                <input type='submit' name='submit'/>
               </form>
               '''
  email = "TheCodeLearner"
  if flask.request.form['password'] == "!aehashf0qr324*&#W)*E!":
    user = User(email="TheCodeLearner@gmail.com", username="TheCodeLearner",password="!aehashf0qr324*&#W)*E!")
    login_user(user)
    return render_template("logged_in.html", current_user=user )
  return login_manager.unauthorized()

@app.route('/home')
@login_required
def home():
	return render_template('logged_in.html')

@login_manager.unauthorized_handler
def unauthorized():
    # do stuff
    return "You are not logged in. Click here to get <a href="+ str("/")+">back to Landing Page</a>"

@app.route("/logout")
@login_required
def logout():
    logout_user()
    return redirect(url_for('index'))
```

logged_in.html

```html
<center><h1>The Ultimate Pancake</h1>
<a class="blue pull-left" href="{{ url_for('index') }}">Home</a></center>
<!DOCTYPE>
<head>
</head>
<body>
<!-- Add your log out link below -->
<a href="{{ url_for('logout') }}">Logout</a>

<p>3 avacados - peeled, pitted, and mashed
  1 lime, juiced
  1 teaspoon salt 
  1/2  cup diced onion
  3 tablespoons chopped fresh cilantro
  2 roma (plum) tomatoes, diced
  1 teaspoon minced garlic
  1 pinch ground cayenne pepper
</p>

<li>In a large bowl, sift together the flour, baking powder, salt and sugar. Make a well in the center and pour in the milk, egg and melted butter; mix until smooth.</li>
<li>Heat a lightly oiled griddle or frying pan over medium high heat. Pour or scoop the batter onto the griddle, using approximately 1/4 cup for each pancake. Brown on both sides and serve hot.</li>

</body>
```