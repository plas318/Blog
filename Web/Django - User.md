User class is already provided by Django

To modify the User Class, I need to Inherit either the AbstractBaseUser or AbstractUser class

AbstractBaseUser ==> Need to alter/override all the default class into a custom class
AbstractUser ==> Override the default user class with some additional behaviors

```
class User(AbstractBaseUser, PermissionsMixin):
    '''
    User Model
    '''

    email = models.EmailField(unique=True)
    name = models.CharField(max_length=30)
    dept = models.ForeignKey(
        Department, 
        on_delete=models.SET_NULL,
        related_name='employees',
        null=True
    )

    is_active = models.BooleanField(default=True) # account active state
    is_admin = models.BooleanField(default=False) # Custom Permission
    is_staff = models.BooleanField(default=False) # Django admin permission
    is_superuser = models.BooleanField(default=False) # Django admin & crud for all models
    objects = UserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = [
        'name',
    ]

    class Meta:
        verbose_name = 'Employee'
        verbose_name_plural = 'Employees'
    

    def __str__(self):
       return f'{self.email} {self.name}'

```

PermissionsMixin Class is required to inherit to use the django's permissions classes.

django's default user class provides basic fields as
id, email, name, create_date, is_active, is_staff, is_superuser and such.. details are available from django docs

objects=UserManager() is required since django needs to call this function to define the User
therefore you need to create a UserManager function also to customize the user class.

The Meta class allows you to define additional info about the class
verbose_name and such determines how your model get's displayed on django-admin

__str__ function determines how your model displays on the cli and such


```
from django.db import models
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin


class UserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        '''
        Creates a user with credentials
        '''

        if not email:
            raise ValueError('Email is required')

        email =self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using = self._db)

        return user

    def create_superuser(self, email, password=None, **extra_fields):
        '''
        Creates a superuser(admin) with credentials
        '''
        
        extra_fields.setdefault('is_admin', True)
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        return self.create_user(email, password, **extra_fields)
```
UserManager class has two functions create_user, create_superuser to create accounts
this function could be used within the django-admin cli to create a superaccount and such.


superuser basically is a user account with some boolean fields set to true (is_admin, is_superuser..)

set_password is a function to hash and save the password instead of passing the literal string value


The default User class can be accessed by 

django.contrib.auth.models
author = models.ForeignKey(User, on_delete=models.CASCADE) # User


After creating the model, to actually apply the model you need to migrate
to migrate the model

```
python manage.py makemigrations
python manage.py migrate
```


To see the actual sql code that gets run
`python sqlmigrate blog 0001`

```
CREATE TABLE "blog_post" (
"id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, 
"title" varchar(20) NOT NULL, 
"content" text NOT NULL, 
"date_posted" datetime NOT NULL, 
"author_id" integer NOT NULL REFERENCES 
"auth_user" ("id") DEFERRABLE INITIALLY DEFERRED);
CREATE INDEX "blog_post_author_id_dd7a8485" ON "blog_post" ("author_id");
```







