# Class Based View
Import the generic to design a class based view for your app. We have many generic view like generic.DetailView, generic.ListView and generic.DeleteView etc. By default django use field call slug or pk to get data, if you dont define get_queryset for detail view, it alredy has underline error checking so no try and except block. Class based view handles all the rendering and all the packaging for ListView or others.
```
from django.views import generic

class IndexView(generic.ListView):
    template_name = template_path #define if you dnt wanna use django convention
    model = model_name #tells view that what table are you using.
    
    def get_queryset(): # to return custom set otherwise django will go for all rows.
      return Question.objects.order_by('')[:5]
```

## url conf for class based view
as_view is a static function, package the class based view as callable so that it can be used by django.
```
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

## Mixins
We use this for following:
1. modular functionality : you dont have to write same code again and again.
2. Helps to follow dry principal which means dont repeat yourself.

## Generic Views
Advantages:
1. Basic CBV.
2. Defines the fewest methods.
3. Extremely easy to extend using the HTTP Verbs.
Get, POST, PUT, DELETE
