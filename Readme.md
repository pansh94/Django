# Django
## Models
1. Each model is a Python class that subclasses django.db.models.Model.
2. Each attribute of the model represents a database field.
3. With all of this, Django gives you an automatically-generated database-access API; 

### Using Model
Define your app in settings.py to let django know that you created a model in **INSTALLED_AAPS**. When you add new apps to INSTALLED_APPS, be sure to run manage.py migrate, optionally making migrations for them first with manage.py makemigrations.

### Fields
Each field is specified as a class attribute, and each attribute maps to a database column. Be careful not to choose field names that conflict with the models API like **clean, save, or delete**.
Each field in your model should be an instance of the appropriate Field class. Django uses the field class types to determine a few things:

    1. The column type, which tells the database what kind of data to store (e.g. INTEGER, VARCHAR, TEXT).
    2. The default HTML widget to use when rendering a form field.
    3. The minimal validation requirements, used in Django’s admin and in automatically-generated forms.
    
You can write your custom fields too. 

#### Field options
Each field takes a certain set of field-specific arguments(**max_length**) and There’s also a set of common arguments available to all field types. 

    1. null : if True( Allow empty value as NULL), by default False.
    2. blank : if True, field allow to be blank, by default False.
    3. choices
```
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60, default="temp")
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
-------------------------------------------------------------------
p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```
4. default : Default value of the field. This can be a value or a callable object. If callable it will be called every time a new object is created.
5. primary_key : if True then field will be primary key for the model. If you don’t specify primary_key=True for any fields in your model, Django will automatically add an IntegerField to hold the primary key, so you don’t need to set primary_key=True on any of your fields unless you want to override the default primary-key behavior. The primary key field is read-only. If you change the value of the primary key on an existing object and then save it, a new object will be created alongside the old one. 
6. unique : if True then field will be unique.

##### Automatic primary key fields
By default django give automatic, auto-incremented primary key to each model. 
```
id = models.AutoField(primary_key=True)
```
If you’d like to specify a custom primary key, just specify primary_key=True on one of your fields. If Django sees you’ve explicitly set Field.primary_key, it won’t add the automatic id column. Model require exactly one field to have primary key.

##### Verbode Name Field
Each field type, except for ForeignKey, ManyToManyField and OneToOneField, takes an optional first positional argument – a verbose name. If the verbose name isn’t given, Django will automatically create it using the field’s attribute name, converting underscores to spaces.
```
first_name = models.CharField("person's first name", max_length=30)
first_name = models.CharField(max_length=30) #verbose name is : first name
```
ForeignKey, ManyToManyField and OneToOneField require the first argument to be a model class, so use the verbose_name keyword argument:
```
poll = models.ForeignKey(
    Poll,
    on_delete=models.CASCADE,
    verbose_name="the related poll",
)
sites = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(
    Place,
    on_delete=models.CASCADE,
    verbose_name="related place",
```
### Relationships
Django offers ways to define the three most common types of database relationships: many-to-one, many-to-many and one-to-one.

#### Many-to-one relationships
To define a many-to-one relationship, use **django.db.models.ForeignKey**. ForeignKey requires a positional argument: the class to which the model is related. It’s suggested, but not required, that the name of a ForeignKey field (manufacturer in the example above) be the name of the model, lowercase. As with ForeignKey, you can also create recursive relationships (an object with a many-to-many relationship to itself) and relationships to models not yet defined.
```
from django.db import models

class Manufacturer(models.Model):
    # ...
    pass

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
```

#### Many-to-Many
To define a many-to-many relationship, use **ManyToManyField**. You use it just like any other Field type: by including it as a class attribute of your model. It doesn’t matter which model has the ManyToManyField, but you should only put it in one of the models – not both.
```
from django.db import models

class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```
For example, consider the case of an application tracking the musical groups which musicians belong to. For these situations, Django allows you to specify the model that will be used to govern the many-to-many relationship. You can then put extra fields on the intermediate model. The intermediate model is associated with the ManyToManyField using the **through** argument to point to the model that will act as an intermediary. For our musician example, the code would look something like this:
```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
================================================================================
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")
>>> beatles = Group.objects.create(name="The Beatles")
>>> m1 = Membership(person=ringo, group=beatles,
...     date_joined=date(1962, 8, 16),
...     invite_reason="Needed a new drummer.")
>>> m1.save()
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>]>
>>> ringo.group_set.all()
<QuerySet [<Group: The Beatles>]>
>>> m2 = Membership.objects.create(person=paul, group=beatles,
...     date_joined=date(1960, 8, 1),
...     invite_reason="Wanted to form a band.")
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>]>
```
When you set up the intermediary model, you explicitly specify foreign keys to the models that are involved in the many-to-many relationship. This explicit declaration defines how the two models are related. You can also use **add(), create(), or set()** to create relationships, as long as your specify through_defaults for any required fields:
```
>>> beatles.members.add(john, through_defaults={'date_joined': date(1960, 8, 1)})
>>> beatles.members.create(name="George Harrison", through_defaults={'date_joined': date(1960, 8, 1)})
>>> beatles.members.set([john, paul, ringo, george], through_defaults={'date_joined': date(1960, 8, 1)})
```
If the custom through table defined by the intermediate model does not enforce uniqueness on the (model1, model2) pair, allowing multiple values, the **remove()** call will remove all intermediate model instances:
```
Membership.objects.create(person=ringo, group=beatles,
...     date_joined=date(1968, 9, 4),
...     invite_reason="You've been gone for a month and we miss you.")
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>, <Person: Ringo Starr>]>
>>> # This deletes both of the intermediate model instances for Ringo Starr
>>> beatles.members.remove(ringo)
>>> beatles.members.all()
<QuerySet [<Person: Paul McCartney>]>
```
The **clear()** method can be used to remove all many-to-many relationships for an instance:
```
>>> # Beatles have broken up
>>> beatles.members.clear()
>>> # Note that this deletes the intermediate model instances
>>> Membership.objects.all()
<QuerySet []>
===============================================================
# Find all the groups with a member whose name starts with 'Paul'
>>> Group.objects.filter(members__name__startswith='Paul')
<QuerySet [<Group: The Beatles>]>
=================================================================
# Find all the members of the Beatles that joined after 1 Jan 1961. As you are using an intermediate model you can query on its attribute.
>>> Person.objects.filter(
...     group__name='The Beatles',
...     membership__date_joined__gt=date(1961,1,1))
<QuerySet [<Person: Ringo Starr]>
```
If you need to access a membership’s information you may do so by directly querying the Membership model:
```
>>> ringos_membership = Membership.objects.get(group=beatles, person=ringo)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
```
Another way to access the same information is by querying the many-to-many reverse relationship from a Person object:
```
>>> ringos_membership = ringo.membership_set.get(group=beatles)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
```

#### One-to-One
To define a one-to-one relationship, use **OneToOneField**.
```
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

    def __str__(self):
        return "%s the place" % self.name

class Restaurant(models.Model):
    place = models.OneToOneField(
        Place,
        on_delete=models.CASCADE,
        primary_key=True,
    )
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)

    def __str__(self):
        return "%s the restaurant" % self.place.name

class Waiter(models.Model):
    restaurant = models.ForeignKey(Restaurant, on_delete=models.CASCADE)
    name = models.CharField(max_length=50)

    def __str__(self):
        return "%s the waiter at %s" % (self.name, self.restaurant)
======================================================================
>>> p1 = Place(name='Demon Dogs', address='944 W. Fullerton')
>>> p1.save()
>>> p2 = Place(name='Ace Hardware', address='1013 N. Ashland')
>>> p2.save()
>>> r = Restaurant(place=p1, serves_hot_dogs=True, serves_pizza=False)
>>> r.save()
>>> r.place
<Place: Demon Dogs the place>
>>> p1.restaurant
<Restaurant: Demon Dogs the restaurant>
```
#### Models across files
Relate a model to one from another app.
```
from django.db import models
from geography.models import ZipCode

class Restaurant(models.Model):
    # ...
    zip_code = models.ForeignKey(
        ZipCode,
        on_delete=models.SET_NULL,
        blank=True,
        null=True,
    )
```
#### Field name restrictions
1. A field name cannot be a Python reserved word, because that would result in a Python syntax error.
```
class Example(models.Model):
    pass = models.IntegerField() # 'pass' is a reserved word!
```
2. A field name cannot contain more than one underscore in a row, due to the way Django’s query lookup syntax works.
```
foo__bar = models.IntegerField() # 'foo__bar' has two underscores!
```
3. A field name cannot end with an underscore, for similar reasons.
SQL reserved words, such as join, where or select, are allowed as model field names, because Django escapes all database table names and column names in every underlying SQL query. It uses the quoting syntax of your particular database engine.

#### Custom field types
you can create your own field class.(writing custom model fields)[https://docs.djangoproject.com/en/2.2/howto/custom-model-fields/]

#### Meta options
Give your model metadata by using an inner class Meta,
```
from django.db import models

class Ox(models.Model):
    horn_length = models.IntegerField()

    class Meta:
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
```
Ordering options (ordering), database table name (db_table), or human-readable singular and plural names (verbose_name and verbose_name_plural).(More meta class options)[https://docs.djangoproject.com/en/2.2/ref/models/options/]

#### Model methods
Define custom methods on a model to add custom “row-level” functionality to your objects. Whereas Manager methods are intended to do “table-wide” things, model methods should act on a particular model instance.
```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birth_date = models.DateField()

    def baby_boomer_status(self):
        "Returns the person's baby-boomer status."
        import datetime
        if self.birth_date < datetime.date(1945, 8, 1):
            return "Pre-boomer"
        elif self.birth_date < datetime.date(1965, 1, 1):
            return "Baby boomer"
        else:
            return "Post-boomer"

    @property
    def full_name(self):
        "Returns the person's full name."
        return '%s %s' % (self.first_name, self.last_name)
```
There are couples of method that are always given to a model. But here are some that you always want to define.
1. __str__() : Object representation in string.
2. get_absolute_url(): This tells Django how to calculate the URL for an object. 

#### Overriding predefined model methods
In particular you’ll often want to change the way **save()** and **delete()** work. You’re free to override these methods (and any other model method) to alter behavior.
```
# If you want to do something before saving the object in table.
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        do_something()
        super().save(*args, **kwargs)  # Call the "real" save() method.
        do_something_else()
# If you wanna restrict some object from saving itself.
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        if self.name == "Yoko Ono's blog":
            return # Yoko shall never have her own blog!
        else:
            super().save(*args, **kwargs)  # Call the "real" save() method.
```
Unfortunately, there isn’t a workaround when creating or updating objects in bulk, since none of save(), pre_save, and post_save are called.

#### Performing custom SQL
(custom sql using raw)[https://docs.djangoproject.com/en/2.2/topics/db/sql/]

### Model Inheritance
Model inheritance in Django works almost identically to the way normal class inheritance works in Python, but base class should inherit **django.db.models.Model** class. The only decision you have to make is whether you want the parent models to be models in their own right (with their own database tables), or if the parents are just holders of common information that will only be visible through the child models.
There are three styles of inheritance that are possible in Django.

1. Often, you will just want to use the parent class to hold information that you don’t want to have to type out for each child model. This class isn’t going to ever be used in isolation, so (Abstract base classes)[https://docs.djangoproject.com/en/2.2/topics/db/models/#abstract-base-classes] are what you’re after.
2. If you’re subclassing an existing model (perhaps something from another application entirely) and want each model to have its own database table, Multi-table inheritance is the way to go.
3. Finally, if you only want to modify the Python-level behavior of a model, without changing the models fields in any way, you can use Proxy models.

#### Abstract base classes
Abstract base classes are useful when you want to put some common information into a number of other models. You write your base class and put **abstract=True** in the **Meta class**. This model will then not be used to create any database table. Instead, when it is used as a base class for other models, its fields will be added to those of the child class.
```
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```
The Student model will have three fields: name, age and home_group. The CommonInfo model cannot be used as a normal Django model, since it is an abstract base class. It does not generate a database table or have a manager, and cannot be instantiated or saved directly. Fields inherited from abstract base classes can be overridden with another field or value, or be removed with None.

##### Meta Inheritance
When an abstract base class is created, Django makes any Meta inner class you declared in the base class available as an attribute. If a child class does not declare its own Meta class, it will inherit the parent’s Meta. If the child wants to extend the parent’s Meta class, it can subclass it.
```
from django.db import models

class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ['name']

class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):
        db_table = 'student_info'
```
This means that children of abstract base classes don’t automatically become abstract classes themselves. Of course, you can make an abstract base class that inherits from another abstract base class. You just need to remember to explicitly set **abstract=True** each time.
Some attributes won’t make sense to include in the Meta class of an abstract base class. For example, including db_table would mean that all the child classes (the ones that don’t specify their own Meta) would use the same database table, which is almost certainly not what you want.

##### Be careful with related_name and related_query_name
If you are using related_name or related_query_name on a ForeignKey or ManyToManyField, you must always specify a unique reverse name and query name for the field. This would normally cause a problem in abstract base classes, since the fields on this class are included into each of the child classes, with exactly the same values for the attributes (including related_name and related_query_name) each time.
To work around this problem, when you are using related_name or related_query_name in an abstract base class (only), part of the value should contain '%(app_label)s' and '%(class)s'.
1. '%(class)s' is replaced by the lowercased name of the child class that the field is used in.
2. '%(app_label)s' is replaced by the lowercased name of the app the child class is contained within.
```
from django.db import models

class Base(models.Model):
    m2m = models.ManyToManyField(
        OtherModel,
        related_name="%(app_label)s_%(class)s_related",
        related_query_name="%(app_label)s_%(class)ss",
    )

    class Meta:
        abstract = True

class ChildA(Base):
    pass

class ChildB(Base):
    pass
```
The reverse name of the common.ChildA.m2m field will be common_childa_related and the reverse query name will be common_childas. The reverse name of the common.ChildB.m2m field will be common_childb_related and the reverse query name will be common_childbs
f you don’t specify a related_name attribute for a field in an abstract base class, the default reverse name will be the name of the child class followed by '_set', just as it normally would be if you’d declared the field directly on the child class. For example, in the above code, if the related_name attribute was omitted, the reverse name for the m2m field would be childa_set in the ChildA case and childb_set for the ChildB field.

##### Multi-table inheritance
The second type of model inheritance supported by Django is when each model in the hierarchy is a model all by itself. The inheritance relationship introduces links between the child model and each of its parents (via an automatically-created OneToOneField). 
```
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)
```
All of the fields of Place will also be available in Restaurant, although the data will reside in a different database table. So these are both possible:
```
>>> Place.objects.filter(name="Bob's Cafe")
>>> Restaurant.objects.filter(name="Bob's Cafe")
```
If you have a Place that is also a Restaurant, you can get from the Place object to the Restaurant object by using the lowercase version of the model name:
```
>>> p = Place.objects.get(id=12)
# If p is a Restaurant object, this will give the child class:
>>> p.restaurant
<Restaurant: ...>
```
However, if p in the above example was not a Restaurant (it had been created directly as a Place object or was the parent of some other class), referring to p.restaurant would raise a Restaurant.DoesNotExist exception.

##### Meta and multi-table inheritance
So a child model does not have access to its parent’s Meta class. However, there are a few limited cases where the child inherits behavior from the parent: if the child does not specify an ordering attribute or a get_latest_by attribute, it will inherit these from its parent.
If the parent has an ordering and you don’t want the child to have any natural ordering, you can explicitly disable it:
```
class ChildModel(ParentModel):
    # ...
    class Meta:
        # Remove parent's ordering effect
        ordering = []
```
##### Inheritance and reverse relations
Because multi-table inheritance uses an implicit OneToOneField to link the child and the parent, it’s possible to move from the parent down to the child, as in the above example. However, this uses up the name that is the default related_name value for ForeignKey and ManyToManyField relations. If you are putting those types of relations on a subclass of the parent model, you must specify the related_name attribute on each such field. If you forget, Django will raise a validation error.
```
class Supplier(Place):
    customers = models.ManyToManyField(Place)
```
This results in the error:
Adding related_name to the customers field as follows would resolve the error: models.ManyToManyField(Place, related_name='provider').

#### Proxy models
Sometimes, however, you only want to change the Python behavior of a model – perhaps to change the default manager, or add a new method. This is what proxy model inheritance is for: creating a proxy for the original model. You can create, delete and update instances of the proxy model and all the data will be saved as if you were using the original (non-proxied) model. The difference is that you can change things like the default model ordering or the default manager in the proxy, without having to alter the original. Proxy models are declared like normal models. You tell Django that it’s a proxy model by setting the proxy attribute of the Meta class to True. For example, suppose you want to add a method to the Person model. You can do it like this:
```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)

class MyPerson(Person):
    class Meta:
        proxy = True

    def do_something(self):
        # ...
        pass
```
The MyPerson class operates on the same database table as its parent Person class. In particular, any new instances of Person will also be accessible through MyPerson, and vice-versa:
```
>>> p = Person.objects.create(first_name="foobar")
>>> MyPerson.objects.get(first_name="foobar")
<MyPerson: foobar>
```
You could also use a proxy model to define a different default ordering on a model. You might not always want to order the Person model, but regularly order by the last_name attribute when you use the proxy. This is easy:
```
class OrderedPerson(Person):
    class Meta:
        ordering = ["last_name"]
        proxy = True
```
Now normal Person queries will be unordered and OrderedPerson queries will be ordered by last_name.
Proxy models inherit Meta attributes in the same way as regular models.
A proxy model must inherit from exactly one non-abstract model class. You can’t inherit from multiple non-abstract models as the proxy model doesn’t provide any connection between the rows in the different database tables. A proxy model can inherit from any number of abstract model classes, providing they do not define any model fields. A proxy model may also inherit from any number of proxy models that share a common non-abstract parent class.'
There is no way to have Django return, say, a MyPerson object whenever you query for Person objects.

##### Proxy Model Manager
If you don’t specify any model managers on a proxy model, it inherits the managers from its model parents. If you define a manager on the proxy model, it will become the default, although any managers defined on the parent classes will still be available.
```
from django.db import models

class NewManager(models.Manager):
    # ...
    pass

class MyPerson(Person):
    objects = NewManager()

    class Meta:
        proxy = True
```
The general rules are:
1. If you are mirroring an existing model or database table and don’t want all the original database table columns, use Meta.managed=False. That option is normally useful for modeling database views and tables not under the control of Django.
2. If you are wanting to change the Python-only behavior of a model, but keep all the same fields as in the original, use Meta.proxy=True. This sets things up so that the proxy model is an exact copy of the storage structure of the original model when data is saved.

#### Multiple Inheritance
this means that if multiple parents contain a Meta class, only the first one is going to be used, and all others will be ignored. Note that inheriting from multiple models that have a common id primary key field will raise an error. To properly use multiple inheritance, you can use an explicit AutoField in the base models:
```
class Article(models.Model):
    article_id = models.AutoField(primary_key=True)
    ...

class Book(models.Model):
    book_id = models.AutoField(primary_key=True)
    ...

class BookReview(Book, Article):
    pass
```

#### Field name “hiding” is not permitted
In normal Python class inheritance, it is permissible for a child class to override any attribute from the parent class. In Django, this isn’t usually permitted for model fields. If a non-abstract model base class has a field called author, you can’t create another model field or define an attribute called author in any class that inherits from that base class.

This restriction doesn’t apply to model fields inherited from an abstract model. Such fields may be overridden with another field or value, or be removed by setting field_name = None.

Note :
1. Model managers are inherited from abstract base classes. Overriding an inherited field which is referenced by an inherited Manager may cause subtle bugs.
2. Django will raise a FieldError if you override any model field in any ancestor model.
3. This restriction only applies to attributes which are Field instances. Normal Python attributes can be overridden if you wish. It also only applies to the name of the attribute as Python sees it: if you are manually specifying the database column name, you can have the same column name appearing in both a child and an ancestor model for multi-table inheritance (they are columns in two different database tables).
## Links
1. [Different fields for django.](https://docs.djangoproject.com/en/2.2/ref/models/fields/#model-field-types)
