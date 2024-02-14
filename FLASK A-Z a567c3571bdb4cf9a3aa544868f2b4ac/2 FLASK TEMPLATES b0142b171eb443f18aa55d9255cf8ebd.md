# 2. FLASK TEMPLATES

**FLASK TEMPLATES**

### **Introduction**

When you navigate through a website you may notice that many of the pages on the site have a similar look and feel. This aspect of a website can be achieved with the use of *templates*. In this lesson the term *template* refers to an HTML file that can represent multiple web pages with the same structure and functionality.

We will be using the Flask framework for our application in this lesson. Flask uses the [Jinja2 template engine](https://jinja.palletsprojects.com/en/2.11.x/) to render HTML files that include application variables and control structures. The Jinja2 template engine is a powerful tool that supports an organized and growth oriented application.

In this lesson we will look at:

- How to organize our site file structure
- Use our application data with our templates
- Leverage control structures within our templates
- Share common elements across many templates

The application we will be building in the following exercises is a cookbook site that consists of a main page and individual recipe pages. Currently our app consists of 2 routes that return HTML strings for the browser to display. Explore the application to begin your path to learning templates!

```python
from flask import Flask
from helper import recipes, descriptions, ingredients, instructions

app = Flask(__name__)

@app.route('/')
def index():
  return '''
    <!DOCTYPE html>
    <html>
      <body>
        <h1>Cooking By Myself</h1>
        <p>Welcome to my cookbook. These are recipes I like.</p>
        <a href="/recipe/1">Fried Egg</a>
      </body>
    </html>
    '''

#### Add the variable `id` to the route URL
#### and make it the sole function parameter
@app.route("/recipe/<int:id>")
def recipe(id):
  return '''
    <!DOCTYPE html>
    <html>
      <body>
        <a href="/">Back To Recipe List</a>
        <p>names[id] = ''' + recipes[id] + '''</p>
        <p>descriptions[id] = ''' + descriptions[id] + '''</p>
        <p>ingredients[id] = ''' + str(ingredients[id]) + '''</p>
        <p>instructions[id] = ''' + str(instructions[id]) + '''</p>
      </body>
    </html>
    '''
```

### **Rendering Templates**

Having routes return full web pages as strings is not a realistic way to build our site. Containing our HTML in files is the standard and more organized approach to structuring our web app.

To work with files, which we will call templates, we use the Flask function `render_template()`. Used in the return statement, this function takes a template file name as an argument and returns the content to send to the client. It uses the Jinja2 template engine to generate HTML using the template file as blueprint.

```python
return render_template("my_template.html")
```

To use `render_template()` in our routes we need to import it from the `flask`. A simple app with an index route would look like this.

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def index():
    return render_template("index.html")
```

Inside the application directory `render_template()` looks for templates inside a directory called **templates**. All template files should be kept inside this directory. To view the application file structure in this exercise click the folder icon in the top left corner of the code editor.

```python
from flask import Flask, render_template
from helper import recipes, descriptions, ingredients, instructions

app = Flask(__name__)

@app.route('/')
def index():
  
  #### Return a rendered index.html file
  return render_template("index.html")

@app.route("/recipe/<int:id>")
def recipe(id):
    
  #### Return a rendered index.html file
  return render_template("fried_egg.html")
```

### **Template Variables**

Instead of having an HTML file for each recipe, it would be a lot easier having one file for many recipes. Being able to pass data to template files is how we can begin to accomplish this goal.

After the filename argument in `render_template()` we can add keyword arguments to be used as variables within the template. These variables are assigned values or app data we would like to access within the template.

```python
flask_variable = "Text for my template"

render_template("my_template.html",
                 template_variable=flask_variable)
```

In this example we’re assigning the value of `flask_variable` to `template_variable` which can be used in **my_template.html**. To add more than one variable separate each assignment with a comma.

```python
render_template("my_template.html",
                template_var1="A string!",
                template_var2=100)
```

Our template now has access to the variables `template_var1` and `template_var2` which hold a string and an integer respectively.

App data can be passed as literal values or the values stored inside variables. We can pass strings, integers, lists, dictionaries or any other objects to our templates.

It is possible to give keyword arguments and the assignment variables the same name `var1=var1`. For these exercises all variables from our flask app will start with `flask` and all template variables will start with `template`.

To access the variables in our templates we need to use the expression delimiter: `{{ }}`.

```python
{{ template_variable }}
```

The delimiter can be used inline with text and alongside HTML elements.

```python
<h1>My Heading: {{ template_variable }}</h1>
```

Certain operations can be performed inside expression delimiters `{{ }}`.

With `template_variable = 20`.

```python
<p>Template number plus ten: {{ template_variable + 10 }}</p>

OUTPUT
Template number plus ten: 30
```

List and dictionary elements can be accessed individually inside the expression delimiters `{{ }}`.

With `template_list = ["A", "B", "C"]`

```python
<p>Element at index 1: {{ template_list[1] }}</p>

OUTPUT
Element at index 1: B
```

![FireShot Capture 002 - Flask Tutorial_ Templates - Python Tutorial - pythonbasics.org.png](2%20FLASK%20TEMPLATES%20b0142b171eb443f18aa55d9255cf8ebd/FireShot_Capture_002_-_Flask_Tutorial__Templates_-_Python_Tutorial_-_pythonbasics.org.png)

### **Variable Filters**

Now that we can use variables in our templates, let’s look at different ways we can perform actions on them.

*Filters* are used by the template engine to act on template variables. To use them simply follow the variable with the filter name inside the delimiter and separate them with the `|` character.

```python
{{ variable | filter_name }}
```

The character `|` separating the variable and the filter is called a *pipe* or *vertical bar*.

The filter `title` acts on a [string](https://www.codecademy.com/resources/docs/general/data-types/string) variable and capitalizes the first letter in every word. This is good for using as formatting on heading strings. Given the variable assignment `template_heading = "my very interesting website"`.

```python
{{ template_heading |  title }}

OUTPUT
My Very Interesting Website
```

Notice that the first letter of every word is now capitalized.

Filters can also take arguments. The `default` filter will output the text in its [argument](https://www.codecademy.com/resources/docs/general/argument) when a variable isn’t passed to the template. Consider if `no_template_variable` is missing from the `render_template()` arguments.

```python
{{ no_template_variable | default("I am not from a variable.") }}

OUTPUT
I am not from a variable.
```

The `default` filter does not work on empty strings `""` or `None` values. We will look at this scenario in the next exercise.

While filters perform more complex functions than simple operators, they are still small, focused actions. Here is a list of commonly applied filters and their descriptions. More information can be found in the [Jinja2 documentation](https://jinja.palletsprojects.com/en/2.11.x/templates/#builtin-filters)

- `title`: Capitalizes the first letter of each word in a string, known as titlecase
- `capitalize`: Capitalizes the first character of a string, such as in a sentence
- `lower`/`upper`: Makes **all** the characters in a string lowercase/uppercase
- `int`/`float`: Changes any number variable to an integer/float
- `default`: Defines a default string if the variable is not defined
- `length`: Calculates the length of a string, list or [dictionary](https://www.codecademy.com/resources/docs/general/data-structures/dictionary) variable
- `dictsort`: Sorts a dictionary by its keys

app.py

```python
from flask import Flask, render_template
from helper import recipes, descriptions, ingredients, instructions

app = Flask(__name__)

@app.route('/')
def index():
  return render_template("index.html")

@app.route("/recipe/<int:id>")
def recipe(id):
  return render_template("recipe.html", template_recipe=recipes[id], template_ingredients=ingredients[id], template_instructions=instructions[id])
```

recipe.html

```html
<!DOCTYPE html>
<html>
  <body>
    <a href="/">Back To Recipe List</a>
    <!-- Make template_recipe title case -->
    <h1>{{ template_recipe | title }}</h1>
    <!-- Ensure a default description -->  
    <p>{{ template_description | default("A " + template_recipe + " recipe.") }}</p>
    <!-- Output number of ingredients --> 
    <h3>Ingredients {{ template_ingredients | length }}</h3>
    <ul>
      <li>{{ template_ingredients[0] }}</li>
      <li>{{ template_ingredients[1] }}</li>
      <li>{{ template_ingredients[2] }}</li>
    </ul>
    <!-- Ensure sorted instruction dictionary -->
    <h3>Instructions</h3>
    <ol>
      <li>{{ template_instructions | dictsort }}</li>
    </ol>
  </body>
</html>
```

![Untitled](2%20FLASK%20TEMPLATES%20b0142b171eb443f18aa55d9255cf8ebd/Untitled.png)

### **If Statements**

Including *conditionals* such as if and if/else statements in our templates allows us to control how data is handled.

Let’s say we have a string variable passed to our template. When the variable contains an empty string will you want to output it or will you want to output another string? Remember the `default` filter doesn’t work in this situation so an if statement is needed.

Using if statements in a template happens inside a statement delimiter block: `{% %}`.

```python
{% if condition %}
  <p>This text will output if condition is True</p>
{% endif %}
```

Notice the `{% endif %}` delimiter is necessary to close the `if` statement.

The *condition* can include a variable that is tested using standard comparison operators, `<`, `>`, `<=`, `>=`, `==`, `!=`.

```python
{% if template_variable == "Hello" %}
  <p>{{template_variable}}, World!</p>
{% endif %}
```

While inside statement delimiters `{% %}` we can access variables without using the usual expression delimiter `{{ }}`.

Variables can also be tested on their own. A variable defined as `None` or `False` or equates to `0` or contains an empty sequence such as `""` or `[]` will test as `False`.

The `{% else %}` and `{% elif %}` delimiters can be included to create multi-branch if statements.

Given the assignment `template_number = 20`.

```python
{% if template_number < 20 %}
  <p>{{ template_number }} is less than 20.</p>
{% elif template_number > 20 %}
   <p>{{ template_number }} is greater than 20.</p>
{% else %}
   <p>{{ template_number }} is equal to 20.</p>
{% endif %}

OUTPUT
20 is equal to 20.
```

As expected the `{% else %}` branch is the one that is followed.

app.py

```python
from flask import Flask, render_template
from helper import recipes, descriptions, ingredients, instructions

app = Flask(__name__)

@app.route('/')
def index():
  return render_template("index.html")

@app.route("/recipe/<int:id>")
def recipe(id):
  return render_template("recipe.html", template_recipe=recipes[id], template_description=descriptions[id], template_ingredients=ingredients[id], template_instructions=instructions[id])
```

index.html

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Cooking By Myself</h1>
    <p>Welcome to my cookbook. These are recipes I like.</p>
    <p><a href="/recipe/1">Fried Egg</a></p>
    <p><a href="/recipe/2">Buttered Toast</a></p>
  </body>
</html>
```

recipe.html

```html
<!DOCTYPE html>
<html>
  <body>
    <a href="/">Back To Recipe List</a>
    <h1>{{ template_recipe | title }}</h1>
    <!-- Insert description if statement here -->
    
    <p>{{ template_description }}</p>
    <!-- Include else here -->

    <p>A {{ template_recipe }} recipe.</p>
    <!-- Be sure to close with an endif block -->
    
    <h3>Ingredients - {{ template_ingredients | length}}</h3>
    <ul>
      <li>{{ template_ingredients[0] }}</li>
      <li>{{ template_ingredients[1] }}</li>
      <!-- Insert ingredient if statement -->
    
      <li>{{ template_ingredients[2] }}</li>
      <!-- Be sure to close with an endif block -->
      
    </ul>
    <h3>Instructions</h3>
    <ol>
      <li>{{ template_instructions | dictsort }}</li>
    </ol>
  </body>
</html>
```

### **For Loops**

Repetitive tasks are standard in most computer applications and template rendering is no different. Creating lists, tables or a group of images are all repetitive tasks that can be solved using *for loops*.

Using the same statement delimiter block as if statements `{% %}`, for loops step through a range of numbers, lists and dictionaries.

The following code will create an ordered list where each line will output the [index](https://www.codecademy.com/resources/docs/general/database/index) of the sequence:

```python
<ol>
{% for x in range(3) %}
  <li>{{ x }}</li>
{% endfor%}
</ol>

OUTPUT
1. 0
2. 1
3. 2
```

The syntax is similar to a Python for loop where we define a loop variable `x` to step through a series of numbers using `range(3)`. The local loop variable can be used inside our loop with the expression delimiter `{{x}}`.

Similar to the if statements we need to close the loop with an `{%endfor%}` block.

The following are a few more applications of a for loop.

### **Iterate through a list variable:**

```python
{% for element in template_list %}
```

### **Iterate through a string:**

```python
{% for char_in_string in “Hello!” %}
```

### **Iterate through the keys of a [dictionary](https://www.codecademy.com/resources/docs/general/data-structures/dictionary) variable:**

```python
{% for key in template_dict %}
```

### **Iterate through keys AND values of a dictionary with `items()`:**

```python
{% for key, value in template_dict.items() %}
```

Using the filter `dictsort` in a loop outputs the key/value pairs just like `items()`

index.html

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Cooking By Myself</h1>
    <p>Welcome to my cookbook. These are recipes I like.</p>
    <!-- Implement a for loop using `template_recipes`-->
    {% for id, name in template_recipes.items() %}
    <p><a href="/recipe/{{ id }}">{{ name }}</a></p>
    {% endfor %}
  </body>
</html>
```

recipe.html

```html
<!DOCTYPE html>
<html>
  <body>
    <a href="/">Back To Recipe List</a>
    <h1>{{ template_recipe | title }}</h1>
    
    {% if template_description %}
      <p>{{ template_description }}</p>
    {%else%}
      <p>A {{ template_recipe }} recipe.</p>
    {% endif %}
    
    <h3>Ingredients - {{ template_ingredients | length}}</h3>
    <ul>
      <!-- Implement a for loop to iterate through 
      `template_ingredients`-->
      {% for ingredient in template_ingredients %}
      <li>{{ ingredient }}</li>
      {% endfor %}
    </ul>

    <h3>Instructions</h3>
    <ul>
    {% for key, instruction in template_instructions|dictsort %}
      <!-- Add the correct dictionary element to list 
      the instructions -->
      <p>{{ key }}: {{ instruction }}</p>
    {% endfor %}
    </ul>
  </body>
</html>
```

![Untitled](2%20FLASK%20TEMPLATES%20b0142b171eb443f18aa55d9255cf8ebd/Untitled%201.png)

### **Inheritance**

If you go to any website you may notice certain elements exist across different web pages.

The navigation bar is a good example of a common page element. This is the banner at the top of most sites that has links to different pages. No matter what page you’re on the navigation bar is there.

Imagine having separate files for each web page and wanting to make a change to the navigation bar. Would you have to change the content of every template of the site? No, that would take too long.

To solve this problem template files are used to share content across multiple templates. The simplest case is a file that includes the top portion of the templates through the `<body>` tag and then the closing `</body>` and `</html>` tags. Jinja2 statement delimiters are then used to identify the area of the template where specific content will be substituted in.

```python
<html>
  <head>
    <title>MY WEBSITE</title>
  </head>
  <body>
  {% block content %}{% endblock %}
  </body>
</html>
```

For this exercise we will name the above template **base.html**.

To inherit this content in another template we will use the `extends` statement. The code to be substituted should then be surrounded by `{%block content%}` and `{%endblock%}`. All together this looks like the following template:

```python
{% extends "base.html"  %}

{% block content %}
    <p>This is my paragraph for this page.</p>
{% endblock %}
```

This template is named **index.html**.

When a route returns `render_template("index.html")` the rendered page will have this content.

```python
<html>
  <head>
    <title>MY WEBSITE</title>
  </head>
  <body>
    <p>This is my paragraph for this page.</p>
  </body>
</html>
```

base.html

```html
<!DOCTYPE html>
<html>
  <body>
  {%block content%}{%endblock%}
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
      <p><a href="/recipe/{{ id }}">{{ name | title }}</a></p>
    {% endfor %}
{% endblock %}
```

recipe.html

```html
{% extends "base.html" %}
{% block content %}
    <a href="/">Back To Recipe List</a>
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
{% endblock %}
```