Django without the REST architecure uses the templates as like flask (jinja2 template)
Django uses the MTV architecture (Model Template View), contrast to Springs' MVC (Model View Controller)

Templates provide the frontend view (form) to the user


```
#Flask
href = "{{url_for('static', filename='index.css'}}" # Flask에서의 static 파일 참조

url_for('views.home')
```

```
#Django
{% load static %} # HTML 파일 위에 static을 불러와준다

<link rel="stylesheet" href ="{% static "blog/index.css" %} " # Django에서의 static 파일 참조

{% url 'blog-home' %} 

```

Flask uses the double braces to use a function within the template -> url_for() to refer to the static file

Django uses the braces percentage to load the static route, and uses the route to refer to the file

Django uses the name="" param set from the urls.py file to create a link


### Django Template Form
```
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm

class UserRegisterForm(UserCreationForm):
    email = forms.EmailField(required=False)
    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']
```

UserCreationForm provides a default standard user register form easily

### Django Template View
```
from .forms import UserRegisterForm # 직접만든 Form Class

def register(request):
    if request.method == 'POST': # Form에게서 Post request를 통해 자료를 넘겨받는다
        form = UserRegisterForm(request.POST)

        if form.is_valid():
            username = form.cleaned_data.get('username')
            messages.success(request, f'Successfully created Account {username}!')
            form.save() # Commits the new User object based on form
            return redirect('blog-home')            
        
    else:
        form = UserRegisterForm()
        
    return render(request, 'users/register.html', {'form' : form} )
```
