# 3. FLASK FORMS

### **Introduction**

An important role of websites is gathering information from the user. Whether a user is signing into their Codecademy account, ordering clothes online or leaving feedback for a company, web forms have provided a simple way to enter and submit data over the internet.

The use of forms in a site can be an involved process. The designer must gather the right information, display the fields in a pleasing manner and ensure the data is collected correctly. Over the years this has become easier thanks to frameworks like Flask, which help streamline the process of displaying fields and gathering data.

This lesson assumes a foundational knowledge of web forms and the steps involved in sending the data to the web [server](https://www.codecademy.com/resources/docs/general/server). In the following exercises we are going to look at how Flask can help gather data from regular web forms as well as create forms in an entirely new way.

**Instructions**

To help us learn about forms we will be using a cookbook app that lists recipes on a main page and links to individual recipe pages.

- The main Flask app is contained in **app.py** and has three routes: `index`, `recipe` and `about`. The `index` route has method `POST` added to handle form submission.
- The file **helper.py** contains the mock data for the app and has two functions, `add_ingredients()` and `add_instructions()` to help populate the data.
- The main web page is rendered from the template **index.html**. There is a title, list of recipes and a new recipe form. The form has fields for the recipe name, description, ingredients and instructions.
- The other template is **recipe.html** which renders each individual recipe using the mock data.

Review the site structure and head to the next exercise when you’re ready.

```html
{% extends "base.html" %}
{% block content %}
  <h1>Cooking By Myself</h1>
  <p>Welcome to my cookbook. These are recipes I like.</p>
  {% for id, name in template_recipes.items() %}
    <p><a href="/recipe/{{ id }}">{{ name }}</a></p>
  {% endfor %}

  <form action="/" method="POST">
    <h3>Add Recipe</h3>
    <p>
      <label for="recipe">Name:</label>
      <input type="text" name="recipe"/>
    </p>
    <p>
      <label for="description">Description:</label>
      <textarea name="description"></textarea>
    </p>
    <p>
      <label for="ingredients">Ingredients:</label>
      <textarea name="ingredients"></textarea>
    </p>
    <p>
      <label for="instructions">Instructions:</label>
      <textarea name="instructions"></textarea>
    </p>
    <p><input type="submit" name="submit_recipe"/></p>
  </form>
{% endblock %}
```

![Untitled](3%20FLASK%20FORMS%20b0d46793807a4ddb935daa6814cc872a/Untitled.png)

### **Flask Request Object**

Every time a client communicates with a [server](https://www.codecademy.com/resources/docs/general/server) it does so through a *request*. By default our Flask routes only support GET requests. These are requests for data such as what to display in a browser window. When submitting a form through a website, the form data is sent as a POST request. This type of request wants to add data to the app. Routes can handle POST requests if it is specified in the `methods` [argument](https://www.codecademy.com/resources/docs/general/argument) of the `route()` decorator.

```python
@app.route("/", methods=["GET", "POST"])
```

The code above shows a route that now supports both GET and POST requests. By default `methods` is set to [“GET”]. When adding “POST” to a route’s `methods`, be sure to include “GET” if you plan for the route to handle those requests as well.

Flask provides access to the data in the request through the `request` object. Importing the `request` object allows us to access everything about the requests to our app including form data and the request [method](https://www.codecademy.com/resources/docs/general/method) such as `GET` or `POST`.

```python
from flask import request
```

When data is sent via a form submission it can be accessed using the `form` attribute of the `request` object. The `form` attribute is a [dictionary](https://www.codecademy.com/resources/docs/general/data-structures/dictionary) with the form’s field names as the keys and the associated data as the values. For example, if a text input had the name `"my_text"`, then the data access would look like this.

```python
text_in_field = request.form["my_text"]
```

app.py

```python
from flask import Flask, render_template, request
from helper import recipes, descriptions, ingredients, instructions, add_ingredients, add_instructions

app = Flask(__name__)

@app.route('/', methods=["GET", "POST"])
def index():
  new_id = len(recipes) + 1
  if len(request.form) > 0:
    #### Add the recipe name to recipes[new_id] 
    recipes[new_id] = request.form['recipe']
    #### Add the recipe description to descriptions[new_id]
    descriptions[new_id] = request.form['description']
    #### Add the values to new_ingredients and new_instructions
    new_ingredients = request.form['ingredients']
    new_instructions = request.form['instructions']
    add_ingredients(new_id, new_ingredients)
    add_instructions(new_id, new_instructions)
  return render_template("index.html", template_recipes=recipes)

@app.route("/recipe/<int:id>")
def recipe(id):
  return render_template("recipe.html", template_recipe=recipes[id], template_description=descriptions[id], template_ingredients=ingredients[id], template_instructions=instructions[id])

@app.route('/about')
def about():
  return render_template("about.html")
```

index.html

```html
{% extends "base.html" %}
{% block content %}
    <h1>Cooking By Myself</h1>
    <p>Welcome to my cookbook. These are recipes I like.</p>
    {% for id, name in template_recipes.items() %}
      <p><a href="/recipe/{{ id }}">{{ name }}</a></p>
    {% endfor %}

    <form action="/" method="POST">
      <h3>Add Recipe</h3>
      <p>
        <label for='recipe'>Name:</label>
        <input type='text' name='recipe'/>
      </p>
      <p>
        <label for='description'>Description:</label>
        <textarea name='description'></textarea>
      </p>
      <p>
        <label for='ingredients'>Ingredients:</label>
        <textarea name='ingredients'></textarea>
      </p>
      <p>
        <label for='instructions'>Instructions:</label>
        <textarea name='instructions'></textarea>
      </p>
      <input type='submit' name='submit_recipe'/>
    </form>
{% endblock %}
```

### **Route Selection**

As sites get larger and their file structure becomes more complex the paths of Flask routes may change. In this case paths that are hard coded into the navigation elements such as hyperlinks and forms may break.

Flask addresses the challenge of expanding file structures with `url_for()`. The function `url_for()` takes a route’s function name as an [argument](https://www.codecademy.com/resources/docs/general/argument) and returns the associated [URL](https://www.codecademy.com/resources/docs/general/url) path. Given the following Flask route declaration:

```python
@app.route('/')
def index:
```

These two hyperlinks below are identical.

```python
<a href="/">Index Link</a>

<a href="{{ url_for('index') }}">Index Link</a>
```

Breaking down the second line of above code, we can see a few things:

- `url_for()` is inside an expression delimiter
- the argument for `url_for()` is inside single quotes
- the entire statement is inside double quotes

Because of the last 2 points it is important to use one type of quotes for the whole statement and the other type of quotes for the `url_for()` argument.

To pass variables from the template to the app, keyword arguments can be added to `url_for()`. They will be sent as arguments attached to the URL. It can be accessed the same way as if it was added onto the path manually.

```html
<a href="{{ url_for('my_page', id=1) }}">One</a>
```

This line creates a link that sends the value `1` to the route with the function name `my_page`. The route can access the variable through `my_id`.

```python
@app.route("/my_path/<int:my_id>"), methods=["GET", "POST"])
def my_page(my_id):
    # Access flask_name in this function
    new_variable = my_id
    ...
```

base.html

```html
<!DOCTYPE html>
<html>
  <body>
    <div>
      <!-- Replace URL string with url_for -->
      <a href="{{ url_for('index') }}">Recipes</a>
      | 
      <!-- Replace URL string with url_for -->
      <a href="{{ url_for('about') }}">About</a>
    </div>
    {% block content %}
    {% endblock %}
  </body>
</html>
```

index.html

```html
{% extends "base.html" %}
{% block content %}
  <h1>Cooking By Myself</h1>
  <p>Welcome to my cookbook. These are recipes I like.</p>
  {% for id, name in template_recipes.items() %}
    <!-- Use url_for to pass the id variable -->
    <p><a href="{{ url_for('recipe', id=id) }}">{{ name }}</a></p>
  {% endfor %}

  <form action="/" method="POST">
    <h3>Add Recipe</h3>
    <p>
      <label for='recipe'>Name:</label>
      <input type='text' name='recipe'/>
    </p>
    <p>
      <label for='description'>Description:</label>
      <textarea name='description'></textarea>
    </p>
    <p>
      <label for='ingredients'>Ingredients:</label>
      <textarea name='ingredients'></textarea>
    </p>
    <p>
      <label for='instructions'>Instructions:</label>
      <textarea name='instructions'></textarea>
    </p>
    <input type='submit' name='submit_recipe'/>
  </form>
{% endblock %}
```

### **FlaskForm Class**

Flask provides an alternative to web forms by creating a form class in the application, implementing the fields in the template and handling the data back in the application.

A Flask form class inherits from the class `FlaskForm` and includes attributes for every field:

```python
class MyForm(FlaskForm):
    my_textfield = StringField("TextLabel")
    my_submit = SubmitField("SubmitName")
```

This simple class will enable the creation of a form with a text field and a submit button.

The class inherits from the class `FlaskForm` which allows it to implement the form as template variables and then collect the data once submitted. `FlaskForm` is a part of [FlaskWTF](https://flask-wtf.readthedocs.io/en/0.15.x/).

Access to the fields of this form class is done through the attributes, `my_textfield` and `my_submit`. The `StringField` and `SubmitField` classes are the same as `<input type=text...` and `<input type=submit...` respectively and are part of the [WTForms library](https://wtforms.readthedocs.io/en/2.3.x/).

Below is a simple Flask app with the form class.

```python
from flask import Flask, render_template
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField

app = Flask(__name__)
app.config["SECRET_KEY"] = "my_secret"

class MyForm(FlaskForm):
    my_textfield = StringField("TextLabel")
    my_submit = SubmitField("SubmitName")

@app.route("/")
def my_route():
    flask_form = MyForm()
    return render_template("my_template", template_form=flask_form)
```

First note the new import statements. `FlaskForm` is imported from the `flask_wtf` module and both form fields import from `wtforms`.

The next new line is:

```python
app.config["SECRET_KEY"] = "my_secret"
```

This line is a way to protect against CSRF or Cross-Site Request Forgery. Without going into too much detail, CSRF is an attack that used to gain control of a web application.

Next is the `MyForm` class definition. It inherits from `FlaskForm` and has attributes for the text and submit fields. For each field the label is passed as the only [argument](https://www.codecademy.com/resources/docs/general/argument).

Lastly, in order to use this form in our template, we must create an instance of it and pass it to the template using `render_template()`. We will look at applying the form in the template in the next exercise.

app.py

```python
from flask import Flask, render_template, request
from helper import recipes, descriptions, ingredients, instructions, add_ingredients, add_instructions, comments
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField

app = Flask(__name__)
app.config['SECRET_KEY'] = 'mysecret'

#### Create form class here
class CommentForm(FlaskForm):
  comment = StringField("Comment")
  submit = SubmitField("Add Comment")

@app.route('/', methods=["GET", "POST"])
def index():
  return render_template("index.html", template_recipes=recipes)

@app.route("/recipe/<int:id>", methods=["GET", "POST"])
def recipe(id):
  #### Instantiate form class here
  comment_form = CommentForm()
  return render_template("recipe.html", template_recipe=recipes[id], template_description=descriptions[id], template_ingredients=ingredients[id], template_instructions=instructions[id], template_comments=comments[id], template_form=comment_form)

@app.route('/about')
def about():
  return render_template("about.html")
```

### **Template Form Variables**

Creating a form in the template is done by accessing attributes of the form passed to the template.

Let’s use the following form as we work toward implementing it in a template:

```python
class MyForm(FlaskForm):
    my_textfield = StringField("TextLabel")
    my_submit = SubmitField("SubmitName")
```

In our application route we must instantiate the form and assign that instance to a template variable.

```python
my_form = MyForm()

return render_template(template_form=my_form)
```

Moving to the template, creating a form we simply use the form class attributes as expressions.

```html
<form action="/" method="post">
    {{ template_form.hidden_tag() }}
    {{ template_form.my_textfield.label }}
    {{ template_form.my_textfield() }}
    {{ template_form.my_submit() }}
</form>
```

Inside the standard `<form>` are all the FlaskForm objects accessed through `template_form`.

The first line `{{ template_form.hidden_tag() }}` is the other end of the CSRF protection. While not visible in the form, this field handles the necessary tasks to protect from CSRF.

The next two lines are for the text box. The first accesses the `label` of the field, which we specified as an argument when we created the field. The second `my_textfield` line is the input field itself.

The last line of the form is the submit button. Just like the HTML version, this will initiate sending the form data back to the server.

The HTML created from this form implementation is as follows:

```html
<form action="/" method="post">
    <input id="csrf_token" name="csrf_token" type="hidden" value="ImI1YzIxZjUwZWMxNDg0ZDFiODAyZTY5M2U5MGU3MTg2OThkMTJkZjQi.XupI5Q.9HOwqyn3g2pveEHtJMijTu955NU">
    <label for="my_textfield">TextLabel</label>
    <input id="my_textfield" name="my_textfield" type="text" value="">
    <input id="my_submit" name="my_submit" type="submit" value="SubmitName">
</form>
```

app.py

```python
from flask import Flask, render_template, request
from helper import recipes, descriptions, ingredients, instructions, add_ingredients, add_instructions, comments
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField

app = Flask(__name__)
app.config['SECRET_KEY'] = 'mysecret'

class CommentForm(FlaskForm):
  comment =  StringField('Comment')
  submit = SubmitField('Add Comment')

@app.route('/', methods=["GET", "POST"])
def index():
  return render_template("index.html", template_recipes=recipes)

@app.route("/recipe/<int:id>", methods=["GET", "POST"])
def recipe(id):
  comment_form = CommentForm()
  return render_template("recipe.html", template_recipe=recipes[id], template_description=descriptions[id], template_ingredients=ingredients[id], template_instructions=instructions[id], template_comments=comments[id], template_form=comment_form)

@app.route('/about')
def about():
  return render_template("about.html")
```

recipe.html

```html
{% extends "base.html" %}
{% block content %}
  <h1>{{ template_recipe | title }}</h1>
  {% if template_description %}
    <p>{{ template_description }}</p>
  {%else%}
    <p>A {{ template_recipe }} recipe.</p>
  {% endif %}
  <h3>Ingredients - {{ template_ingredients | length}}</h3>
  <ul>
  {% for ingredient in template_ingredients %}
      <li>{{ ingredient }}</li>
  {% endfor %}
  </ul>
  <h3>Instructions</h3>
  <ul>
  {% for key, instruction in template_instructions|dictsort %}
      <li>{{ instruction }}</li>
  {% endfor %}
  </ul>
  <h3>Comments</h3>
  <ul>
  {% for comment in template_comments %}
    <li>{{ comment }}</li>
  {% endfor %}
  </ul>
  <form method="post">
  {{ template_form.hidden_tag() }}
  <!-- Insert StringField elements here -->  
  {{ template_form.comment.label }}
  {{ template_form.comment() }}
  <!-- Insert SubmitField element here -->
  {{ template_form.submit() }}
  </form>
{% endblock %}
```

![Untitled](3%20FLASK%20FORMS%20b0d46793807a4ddb935daa6814cc872a/Untitled%201.png)

### **Handling FlaskForm Data**

Once a form is submitted, the data is sent back to the [server](https://www.codecademy.com/resources/docs/general/server) through a `POST` request. Previously we covered accessing this data through the `request` object provided by Flask.

Using our FlaskForm class, data is now accessible through the form instance in the Flask app. The data can be directly accessed by using the `data` attribute associated with each field in the class.

```python
form_data = flask_form.my_textfield.data
```

Keeping all the information and functionality attached to the form object has streamlined the form creation and data gathering process.

Remember that when a route handles a form it is necessary to add the `POST` [method](https://www.codecademy.com/resources/docs/general/method) to the route decorator.

```python
methods=["GET", "POST"]
```

app.py

```python
from flask import Flask, render_template, request
from helper import recipes, descriptions, ingredients, instructions, add_ingredients, add_instructions, comments
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField

app = Flask(__name__)
app.config['SECRET_KEY'] = 'mysecret'

class CommentForm(FlaskForm):
  comment =  StringField('Comment')
  submit = SubmitField('Add Comment')

@app.route('/', methods=["GET", "POST"])
def index():
  return render_template("index.html", template_recipes=recipes)

@app.route("/recipe/<int:id>", methods=["GET", "POST"])
def recipe(id):
  comment_form = CommentForm()
  #### Process data here
  new_comment = comment_form.comment.data
  comments[id].append(new_comment)
  return render_template("recipe.html", template_recipe=recipes[id], template_description=descriptions[id], template_ingredients=ingredients[id], template_instructions=instructions[id], template_comments=comments[id], template_form=comment_form)

@app.route('/about')
def about():
  return render_template("about.html")
```

recipe.html

```html
{% extends "base.html" %}
{% block content %}
  <h1>{{ template_recipe | title }}</h1>
  {% if template_description %}
    <p>{{ template_description }}</p>
  {%else%}
    <p>A {{ template_recipe }} recipe.</p>
  {% endif %}
  
  <h3>Ingredients - {{ template_ingredients | length}}</h3>
  <ul>{% for ingredient in template_ingredients %}
    <li>{{ ingredient }}</li>
  {% endfor %}</ul>

  <h3>Instructions</h3>
  <ul>{% for key, instruction in template_instructions|dictsort %}
    <li>{{ instruction }}</li>
  {% endfor %}</ul>

  <h3>Comments</h3>
  <ul>{% for comment in template_comments %}
    <li>{{ comment }}</li>
  {% endfor %}</ul>

  <form method="post">
    {{ template_form.hidden_tag() }}
    <p>{{ template_form.comment.label }}
    {{ template_form.comment() }}</p>
    <p>{{ template_form.submit() }}</p>
  </form>
    
{% endblock %}
```

### **Validation**

In order to submit a form, it is common that certain required text fields must have data, date fields need to have a specific format, or a checkbox agreeing to certain terms needs to be checked.

*Validation* is when form fields must contain data or a certain format of data in order to move forward with submission. We enable validation in our form class using the `validators` [parameter](https://www.codecademy.com/resources/docs/general/parameter) in the form field definitions.

Validators come from the `wtform.validators` module. Importing the `DataRequired()` validator is accomplished like this:

```python
from wtforms.validators import DataRequired
```

The `DataRequired()` validator simply requires a field to have something in it before the form is submitted. Notifying the user that data is required is handled automatically.

```python
my_textfield = StringField("TextLabel", validators=[DataRequired()])
```

The `validators` [argument](https://www.codecademy.com/resources/docs/general/argument) takes a list of validator instances.

The `FlaskForm` class also provides a [method](https://www.codecademy.com/resources/docs/general/method) called `validate_on_submit()`, which we can used in our route to check for a valid form submission.

```python
if my_form.validate_on_submit():
    # get form data
```

As we saw in the second exercise pertaining to the `request` object, in order to avoid gathering data on first access to the route we had to put the data gathering code inside an `if` statement. The `validate_on_submit()` function does this exact task.

The `validate_on_submit()` function returns `True` when there is a `POST` request and all the associated form validators are satisfied. At this point, the data can be gathered and processed. When the function returns `False` the route function can move on to other tasks such as rendering the template.

app.py

```python
from flask import Flask, render_template, request
from helper import recipes, descriptions, ingredients, instructions, add_ingredients, add_instructions, comments
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField
#### Note the new import statement
from wtforms.validators import DataRequired

app = Flask(__name__)
app.config['SECRET_KEY'] = 'mysecret'

class CommentForm(FlaskForm):
  #### Add a validator argument in the StringField
  comment =  StringField('Comment', validators=[DataRequired()])
  submit = SubmitField('Add Comment')

@app.route('/', methods=["GET", "POST"])
def index():
  return render_template("index.html", template_recipes=recipes)

@app.route("/recipe/<int:id>", methods=["GET", "POST"])
def recipe(id):
  comment_form = CommentForm(csrf_enabled=False)
  #### Replace 'True' with form validation
  if comment_form.validate_on_submit():
    new_comment = comment_form.comment.data
    comments[id].append(new_comment)
  return render_template("recipe.html", template_recipe=recipes[id], template_description=descriptions[id], template_ingredients=ingredients[id], template_instructions=instructions[id], template_comments=comments[id], template_form=comment_form)

@app.route('/about')
def about():
  return render_template("about.html")
```

### **More Form Fields**

We’ve now covered the operation cycle of forms using FlaskWTF. Now let’s look at some additional form fields included in WTForms.

### **TextAreaField**

The TextAreaField is a text field that supports multi-line input. The data returned from a `TextAreaField` instance is a [string](https://www.codecademy.com/resources/docs/general/data-types/string) that may include more whitespace characters such as newlines or tabs.

```python
#### Form class declaration
my_text_area = TextAreaField("Text Area Label")
```

### **BooleanField**

A checkbox element is created using the BooleanField object. The data returned from a `BooleanField` instance is either `True` or `False`.

```python
#### Form class declaration
my_checkbox = BooleanField("Checkbox Label")
```

### **RadioField**

A radio button group is created using the `RadioField` object. Since there are multiple buttons in this group, the instance declaration takes an [argument](https://www.codecademy.com/resources/docs/general/argument) for the group label and a keyword argument `choices` which takes a list of tuples.

Each [tuple](https://www.codecademy.com/resources/docs/general/data-structures/tuple) represents a button in the group and contains the button identifier string and the button label string.

```python
#### Form class declaration
my_radio_group = RadioField("Radio Group Label", choices=[("id1", "One"), ("id2","Two"), ("id3","Three")])
```

Since the RadioField() instance generally contains multiple buttons it is necessary to iterate through it to access the components of the subfields.

app.py

```python
from flask import Flask, render_template, request
from helper import recipes, types, descriptions, ingredients, instructions, add_ingredients, add_instructions, comments
from forms import RecipeForm, CommentForm

app = Flask(__name__)
app.config["SECRET_KEY"] = "mysecret"

@app.route("/", methods=["GET", "POST"])
def index():
  recipe_form = RecipeForm(csrf_enabled=False)
  if recipe_form.validate_on_submit():
    new_id = len(recipes)+1
    recipes[new_id] = recipe_form.recipe.data
    #### Add type data here
    types[new_id] = recipe_form.recipe_type.data
    descriptions[new_id] = recipe_form.description.data
    new_ingredients = recipe_form.ingredients.data
    new_instructions = recipe_form.instructions.data
    add_ingredients(new_id, new_ingredients)
    add_instructions(new_id, new_instructions)
    comments[new_id] = []
  return render_template("index.html", template_recipes=recipes, template_form=recipe_form)

@app.route("/recipe/<int:id>", methods=["GET", "POST"])
def recipe(id):
  comment_form = CommentForm(csrf_enabled=False)
  if comment_form.validate_on_submit():
    new_comment = comment_form.comment.data
    comments[id].append(new_comment)
  return render_template("recipe.html", template_recipe=recipes[id], template_type=types[id], template_description=descriptions[id], template_ingredients=ingredients[id], template_instructions=instructions[id], template_comments=comments[id], template_form=comment_form)

@app.route("/about")
def about():
  return render_template("about.html")
```

forms.py

```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, TextAreaField, RadioField
from wtforms.validators import DataRequired

class RecipeForm(FlaskForm):
  recipe_categories = [("Breakfast","Breakfast"), ("Lunch","Lunch"), ("Dinner","Dinner")]
  recipe = StringField("Recipe", validators=[DataRequired()])  
  #### Add `recipe_type` and assign it a new radio field instance
  recipe_type = RadioField("Type", choices=recipe_categories)
  description = StringField("Description")
  ingredients = TextAreaField("Ingredients")
  instructions = TextAreaField("Instructions")
  submit = SubmitField("Add Recipe")

class CommentForm(FlaskForm):
  comment = StringField("Comment", validators=[DataRequired()])
  submit = SubmitField("Add Comment")
```

index.html

```html
{% extends "base.html" %}
{% block content %}
  <h1>Cooking By Myself</h1>
  <p>Welcome to my cookbook. These are recipes I like.</p>
  {% for id, name in template_recipes.items() %}
    <p><a href="{{ url_for('recipe', id=id) }}">{{ name }}</a></p>
  {% endfor %}
  <form action="/" method="POST">
    {{ template_form.hidden_tag() }}
    <p>{{ template_form.recipe.label }}
    {{ template_form.recipe() }}</p>
    <table><tr>
      {% for btn in template_form.recipe_type %}
      <!-- Put the button variable then the button label 
      in the following td tags-->
      <td>{{ btn() }}</td>
      <td>{{ btn.label }}</td>
    {% endfor %}
    </tr></table>
    <p>{{ template_form.description.label }}
    {{ template_form.description() }}</p>
    <p>{{ template_form.ingredients.label }}
    {{ template_form.ingredients() }}</p>
    <p>{{ template_form.instructions.label }}
    {{ template_form.instructions() }}</p>
    <p>{{ template_form.submit() }}</p>
  </form>
{% endblock %}
```

### **Redirecting**

Besides rendering templates from our routes, it can be important to move from one route to another. This is the role of the function `redirect()`.

Consider the case where we create our form in one route, but after the form submission we want the user to end up in another route. While we can set the `action` attribute in the HTML `<form>` tag go to any path, `redirect()` is the best option to move from one route to another.

```python
redirect("url_string")
```

Using this function inside another route will simply send us to the URL we specify. In the case of a form submission, we can use `redirect()` after we have processed and saved our data inside our `validate_on_submit()` check.

Why don’t we just render a different template after processing our form data? There are many reasons for this, one being that each route comes with its own business logic prior to rendering its template. Rendering a template outside the initial route would mean you need to repeat some or all of this code.

Once again, to avoid possible URL string pitfalls, we can utilize `url_for()` within `redirect()`. This allows us to navigate routes by specifying the route function name instead of the URL path.

```python
redirect(url_for("new_route", _external=True, _scheme='https'))
```

- we must add two new keyword arguments to our call of `url_for()`
- the keyword arguments `_external=True` and `_scheme='https'` ensure that the URL we redirect to is a secure HTTPS address and not an insecure HTTP address

Similarly, regular keyword arguments can be added if necessary.

```python
redirect(url_for("new_route", new_var=this_var, _external=True, _scheme='https'))
```

app.py

```python
from flask import Flask, render_template, request, redirect, url_for
from helper import recipes, types, descriptions, ingredients, instructions, add_ingredients, add_instructions, comments
from forms import RecipeForm, CommentForm

app = Flask(__name__)
app.config['SECRET_KEY'] = 'mysecret'

@app.route('/', methods=["GET", "POST"])
def index():
  recipe_form = RecipeForm(csrf_enabled=False)
  if recipe_form.validate_on_submit():
    new_id = len(recipes)+1
    recipes[new_id] = recipe_form.recipe.data
    types[new_id] = recipe_form.recipe_type.data
    descriptions[new_id] = recipe_form.description.data
    new_igredients = recipe_form.ingredients.data
    new_instructions = recipe_form.instructions.data
    add_ingredients(new_id, new_igredients)
    add_instructions(new_id, new_instructions)
    comments[new_id] = []
    #### Redirect to recipe route here
    return redirect(url_for("recipe", id=new_id, _external=True, _scheme='https'))
  return render_template("index.html", template_recipes=recipes, template_form=recipe_form)

@app.route("/recipe/<int:id>", methods=["GET", "POST"])
def recipe(id):
  comment_form = CommentForm(csrf_enabled=False)
  if comment_form.validate_on_submit():
    new_comment = comment_form.comment.data
    comments[id].append(new_comment)
  return render_template("recipe.html", template_recipe=recipes[id], template_type=types[id], template_description=descriptions[id], template_ingredients=ingredients[id], template_instructions=instructions[id], template_comments=comments[id], template_form=comment_form)

@app.route('/about')
def about():
  return render_template("about.html")
```

![Untitled](3%20FLASK%20FORMS%20b0d46793807a4ddb935daa6814cc872a/Untitled%202.png)