# Making Queries
Database we are going to use.
```
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```
## Creating Objects
A model class represents a database table, and an instance of that class represents a particular record in the database table.
```
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save() #django does not touch the database untill you call save button.
```
To create and save an object in a single step, use the create() method.

## Saving changes to objects
This performs an UPDATE SQL statement behind the scenes. Django doesn’t hit the database until you explicitly call save().
```
>>> b5.name = 'New name'
>>> b5.save()
```

## Saving ForeignKey and ManyToManyField fields
simply assign an object of the right type to the field in question.
```
>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```
Updating a ManyToManyField works a little differently – use the add() method on the field to add a record to the relation. 
```
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```
To add multiple records to a ManyToManyField in one go, include multiple arguments in the call to add(), like this:
```
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

## Retrieving objects
A QuerySet represents a collection of objects from your database. It can have zero, one or many filters. Filters narrow down
the query results based on the given parameters. In SQL terms, a QuerySet equates to a SELECT statement, and a filter is a 
limiting clause such as WHERE or LIMIT.
You get a QuerySet by using your model’s Manager. Each model has at least one Manager, and it’s called objects by default. 
```
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```
Note:
1. Managers are accessible only via model classes, rather than from model instances, to enforce a separation between
“table-level” operations and “record-level” operations.
2. The Manager is the main source of QuerySets for a model. For example, Blog.objects.all() returns a QuerySet that
contains all Blog objects in the database.

## Retrieving all objects
use the all() method on a Manager:
```
>>> all_entries = Entry.objects.all()
```
## Retrieving specific objects with filters
The two most common ways to refine a QuerySet are:
1. **filter(**kwargs)** : Returns a new QuerySet containing objects that match the given lookup parameters.
2. **exclude(**kwargs)** : Returns a new QuerySet containing objects that do not match the given lookup parameters.
```
Entry.objects.filter(pub_date__year=2006)
```
## Chaining filters
The result of refining a QuerySet is itself a QuerySet, so it’s possible to chain refinements together. For example:
```
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime.date(2005, 1, 30)
... )
```

## Filtered QuerySets are unique
