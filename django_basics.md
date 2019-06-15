# Django Basics
Django is based on MVC.

Django provides :
1. ORM(Objects Relastion mapping) : To map db queries.
2. URL routing : Find what logic to follow depends upon the url that hads been requested. 
3. HTML Templating : For view and integrating data in html.
4. Form Handling
5. Unit Testing.

Django is not :
1. Programming language
2. web server

## Getting Started
To start new project go to your desired folder and type :
```
django-admin.py startproject <project name>
```

## Structure
1. **manage.py** : use to run various command for project.
2. **__init__.py** : tells python that this folder contain python files.
3. **wsgi.py** : provide hook for webservers.
4. **settings.py** : configure django.
5. **url.py** : routes request based on url.
6. **apps.py** : controls settings specific to an app.
7. **models.py** : contain database settings or data layes, which django uses to construct db schema and queries.
8. **admin.py** : administrative interface to see and add data.
9. **views.py** : control the flow and accept the request and generate HTTPresponse.
10. **test.py** : use to write test cases for unit testing.
11. **migrations/** : holds file which django uses to migrate and make changes to the schema.   

## Relastionship b/w Model, view, urlpattern and template
When django receive a url request, it uses url pattern to determine which view to pass this request to. UrlPattern always
define in parent app in urls.py.
Then view provide logic and control flow of the project. They receive Http request as argument and return Http response or exception. View can access django model to perform query and get results.
Django Model has class which inherit Model class to define schema and underline structure.
View also leverage template to render view and decide how html response look like. Templates must be present in template folder. And they contain HTML and other tools to display view.

## Models
Models define in models.py of an app. Model classes inherit **django.db.models.Model** class and define field as class attribute. Each model is a table in db and each field is a column in table and each record is row in a table.
1. [Model intro](https://docs.djangoproject.com/en/2.2/topics/db/models/)
2. [Field Information](https://docs.djangoproject.com/en/2.2/ref/models/fields/)

### Migration
When models are added to a model file, the corresponding tables are not reflected in schema without migration command.
**When migration is needed?**
1. When new model is created initial migration is needed to reflect the changes in db.
2. When a field is added, removed or attributes of field are change.
command:
```
python manage.py makemigrations
python manage.py migrate
```
#### Makemigrations
1. Generate migrate files(0001_initial.py in migrations folder and started with no.1) 
2. Determine what changes are needed in db to match the models file.
```
python manage.py showmigrations 
this show that, what is migrated and what not
```

#### Migrate
1. Run all migrations that have not been run yet.
2. You also run migrate for a specific app and no.
```
migrate <appname> <no.>
```
## Admin
we use admin.py to define our admin interface. And use **@admin.register(model_name)** decorator to define associated model.
To see the look and feel you should create super user.
```
python manage.py createsuperuser
```
you can create class in admin.py and associate that with the desired decorator. You can modify your view as list and details here by using **list_display and other things**.

**Note :** Use the following command to run the shell.
```
python manage.py shell
```
