## Django and modules

Django : Framework

djangorestframework : Provides restful templates over the django framework

django-filter : provides filtering options for django

django-cors-headers : module that helps solve the CORS problems that occurs when creating a restful app

drf-spectacular: a module that helps documentize the backend api
(supports OpenAPI 3.0 and provides a swagger view)

python-dotenv: .env helps load variables based on the local .env file


## Django Admin and Manage.py

Both django-admin and manage.py uses DJANGO_SETTINGS_MODULE to setup the settings.py

Start by calling `python manage.py startproject project_name` to create a new django project


Call `python manage.py startapp app_name` to create a new app

After creating the project / app django automatically creates these structures

views: views for the app

models: models and databases

admin: django-admin page template

apps: the name of the app

tests: for running tests

*urls.py: to map the url for the project, usually create one for each app

*serializers.py : use this to create serializers when using drf


### Using dotenv

```
from dotenv import load_dotenv

load_dotenv(override=True)

SECRET_KEY = os.getenv('SECRET_KEY')

elif SERVER == 'aws':
    # Amazon aws ec2 server db
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': os.getenv("NAME"),
            'USER': os.getenv("USER"),
            'PASSWORD': os.getenv("PASSWORD"),
            'HOST': os.getenv("HOST"),
            'PORT': os.getenv("PORT"),
        },
            
    }
```

load_dotenv function loads (overrides) the windows variable with the local .env file

.env
```
NAME = p-db
USER = root
PASSWORD = 1234
```
.env file is made with key = value syntax.


### Settings.py

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework', # DjangoRestFramework
    'django_filters', # Django Filtering
    'auths', # Auths app
    'hospital', # Hospital app
    'drf_spectacular', # Swagger (Openapi3.0)
    'corsheaders', # CORS Headers
]
```

You need to update the settings.py whenever you add an app, or a third party module.

```
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware', # cors middleware
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

Updating middlewares that loads inbetween communications : cors, csrf, logging etc..


Various settings

```
REST_FRAMEWORK = {
    # Pagination Options
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 40,
    
    # Filter options
    'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend'], 

    # Authentication options
    'DEFAULT_AUTHENTICATION_CLASSES' : ['rest_framework.authentication.SessionAuthentication'],

    # Permission options
    'DEFAULT_PERMISSION_CLASSES' : ['rest_framework.permissions.IsAuthenticated'],

    # DRF Spectacular (Swagger openapi)
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

# CORS Settings
CORS_ORIGIN_WHITELIST = ['http://127.0.0.1:3000',
                         'http://localhost:3000']

CSRF_TRUSTED_ORIGINS = ['http://localhost',
                        'http://127.0.0.1']

CORS_ALLOW_CREDENTIALS = True 

SPECTACULAR_SETTINGS = {
    'TITLE': 'Backend',
    'DESCRIPTION': "backend api",
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
    
    # available SwaggerUI configuration parameters
    # https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/
    "SWAGGER_UI_SETTINGS": {
        "deepLinking": True,
        "persistAuthorization": True,
        "displayOperationId": True,
    },
    # available SwaggerUI versions: https://github.com/swagger-api/swagger-ui/releases
    "SWAGGER_UI_DIST": "//unpkg.com/swagger-ui-dist@3.35.1", # default
}
```


## urls.py

```
from django.contrib import admin
from django.urls import path, include

from drf_spectacular.views import SpectacularAPIView, SpectacularRedocView, SpectacularSwaggerView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('auths/', include('auths.urls')),
    path('', include('hospital.urls')),

    # == Swagger views ==
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    # Optional UI:
    path('api/schema/swagger-ui/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    path('api/schema/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
]
```
Match the url using the path function within the urlpatterns list

`path('auths/', include('auths.urls'))` means to follow the urls.py inside the auths app if the path matches auths

`path('api/schema/', SpectacularAPIView.as_view(), name='schema')` specifying the path as such directly maps the url

class based view has to use an as_view() function at the end


## WSGI

wsgi is for web server gateway interface, wsgi.py is an interface that connects the web server such as uwsgi, and gunicorn

django provides a way to publish the web as such

`manage.py runserver 0:8000`​

*however it is not recommended for production






