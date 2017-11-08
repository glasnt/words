## Getting to know the Django ORM

You might have heard of [Django](https://www.djangoproject.com/) before, the Python web framework for "perfectionists with deadlines". It's that one with the [cute pony](http://www.djangopony.com/). 

One of the most powerful features of Django is it's Object Relational Model, or the **ORM**. It's the way you can interact with your database, like you would with SQL. In fact, all the ORM is is just a way to pythonically create SQL to query and manipulate your database, and get results in a pythonic fashion. Well, I say __just__ but it's actually some really clever engineering that takes advantage of some of the more complex parts of Python to make developers' lives easier. 

Before we start looking into how the ORM works, we need a database to manipulate. If you're familiar with relational databases, we need to define a bunch of tables and their relationships: the way they relate to each other. Let's use something familiar: say we want to model a blog that has blog posts, authors, and comments. An author has a name and an email. An author can have many blog posts. A blog post can have many authors, and has a title, content, and a published date. A blog post can have many comments, and those comments have an author, content, and a posted date. 

In Django-ville, this concept of posts authors and comments could be called our Blog app. In this context, an **app** is a self-contained set of models, views and controllers that describes the behaviour and functionality of our blog. Packaged in the right way, many Django projects could use our Blog app. In our project, the Blog could just be one app. We might also have a Forum app, for example. But we'll stick with the scope of our Blog app. 

Some time passes, and you have a website! You have users and content, and it's your task to go into the application and take a look around. There are two ways to do this. You can login to the **django admin**, a web-based backend that has all the apps listed, and ways to manipulate them. We'll get back to that. We're interested in the ORM. 

Assuming you have access to the application, we can get to the ORM by running `python manage.py shell` from the main directory of our Django project

```shell
/srv/web/django/ $ python manage.py shell

Python 3.5.1 (default, Jul 31 2017, 11:25:41)
[GCC 4.2.1 Compatible Apple LLVM 8.1.0 (clang-802.0.42)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>>
```

This will bring us into an interactive console. The `shell` command did a lot of setup for us, including importing our settings and starting Django. But it's not all there yet. To get access to our Blog model, we need to import it. 

```python
>>> from blog.models import *
```

This imports all the blog models, so we can play with our blog posts, authors, and comments. 

For starters, let's get a list of all the Authors
```python
>>> Author.objects.all()
```

What we'll get from this command is a list of all the Authors in a representative string. We also won't fill our entire console, because if there are a lot of results, django will automatically truncate the printed results. 

```python
>>> Author.objects.all()
<QuerySet [<Author: VM (Vicky) Brasseur>, <Author: Rikki Endsley>, <Author: Jen Wike Huger>, '...(remaining elements truncated)...']
```

We can select a single author using `get` instead of `all`. But we need a bit more information to `get` a single record. In our database we have a __primary key__ field on each table to make sure all entries are unique. However, author names are not unique. Many people may [share the same name](https://2016.katieconf.xyz/), so it's not a good unique constraint. A way to get around this is to have a __Universal Unique Identifer (UUID)__ as the primary key. But since these aren't nicely useable by humans, we can still manipulate our author objects by using name. 

```python
>>> Author.objects.get(name="VM (Vicky) Brasseur")
<Author: VM (Vicky) Brasseur>
```

This time, we have a single object that we can interact with, instead of a QuerySet list. With this object, we can interact with it pythonically, using any of the table columns as attributes to look at the object.

```python
>>> vmb = Author.objects.get(name="VM (Vicky) Brasseur")
>>> vmb.name
u'VM (Vicky) Brasseur'
```

And this is where the cool stuff happens. Normally in relational databases, if we want to show information for other tables we'd need to write `LEFT JOIN`s or other table coupling functions, making sure that our foreign keys match up between tables. Django takes care of that for us. 

So in our model, authors write many posts. So, with our Author object, we can just check what posts they've made. 

```python
>>> vmb.posts
QuerySet[<Post: "7 tips for nailing your job interview">, <Post: "5 tips for getting the biggest bang for your cover letter buck">, <Post: "Quit making these 10 common resume mistakes">, '...(remaining elements truncated)...']
```

We can manipulate QuerySets using normal pythonic list manipulations

```python
>>> for post in vmb.posts:
...   print(post.title)
7 tips for nailing your job interview
5 tips for getting the biggest bang for your cover letter buck
Quit making these 10 common resume mistakes
```

If we want to do more complex querying, we can use filters instead of just getting everything. Here is where it gets tricky. In SQL, you have options like `like`, `contains`, and other filtering objects. The ORM uses an interesting way to accept these arguments. 

If I call a function `do_thing()` in my Python script, I'd expect there somewhere there would be a matching `def do_thing`. This is an explicit functionl definition. However in the ORM, you can call a function that __isn't explicitly defined__. Before we were using `name` to match on a name. But, if we wanted to do a substring search, we can use `name__contains`. 

```python
>>> Author.objects.filter(name__contains="Vic")
QuerySet[<Author: VM (Vicky) Brasseur>, <Author: Victor Hugo">]
```

Now, a small note about the double underscore (`__`). These are __very__ python. You may have seen `__main__` or `__repr__` in your travels in Pythonland. These are sometimes referred to as **dunder methods**, a shortening of "**d**ouble **under**score". There are only a few non-alphanumeric characters that can be used in object names in Python; underscore is one of them. So, in the ORM these are used as an explicit separator of different parts of the filter key name. Under the hood the string is split by these underscores, and the tokens are processed separately. `name__contains` gets changed into `attribute: name, filter: contains`. In other programming languages you may use arrows instead, such as `name->contains` in PHP. Don't let dunders scare you, they're just pythonic helpers! (And if you squit, you could say they look like little snakes, little pythons that want to help you with your code.)

The ORM is extremely powerful and very pythonic. But what about that Django Admin you might have heard about?

TODO: insert image here
 
One of the brilliant user accessibility features of Django is it's Admin. If you define your models, you get a nice web-based editing portal, for free. 

And what powers this? **The ORM**

That's right! Given the time you took to define your models in the first place (a topic we skipped this time), Django can turn that into a web-based portal using the same raw functions we used earlier behind the scenes. By default, the admin is basic, but it's just a matter of adding more definitions in your model to change how the admin looks. With a bit of work, you can make an interface that feels like a full content management system that allows your users to edit their own content with ease. 


If you'd like to know more, the [djangogirls tutorial](https://djangogirls.org) section on [the ORM](https://tutorial.djangogirls.org/en/django_orm/) has a detailed walk through. There's also copious amounts of documentation on the [djangoproject website](https://docs.djangoproject.com/en/1.11/topics/db/)
