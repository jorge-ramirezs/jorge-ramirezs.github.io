# Como Usar Correo Electrónico En Vez Nombre De Usuario Para Firmarse en Django


&lt;!--more--&gt;

&lt;!-- Content --&gt;

# YouTube Video:

{{&lt; youtube 111111111 &gt;}}

# Instrucciones:

## 1. Instala Django

Obviamente lo primero que vamos a hacer es instalar Django, al día de hoy que estoy escribiendo este artículo la versión es 4.1.2.

Vamos entonces a crear un directorio, después creamos un ambiente virtual utilizando **pipenv**, instalamos django y creamos el proyecto.

```bash
$ mkdir my_project &amp;&amp; cd my_project
$ pipenv install django~=4.1.2
$ pipenv shell
(my_project)$ django-admin startproject auth .
(my_project)$ python manage.py runserver
```

**ATENCIÓN:**

&gt; Es muy importante que no ejecutes los comandos &#34;makemigrations&#34; o &#34;migrate&#34; antes de completar todos los pasos descritos en este artículo.

Si todo sale como lo planeamos, lo siguiente es dar clic en la liga que aparece en la terminal o también puedes abrir un navegador e ir a [http://127.0.0.1:8000/](http://127.0.0.1:8000/) en donde debe aparecer la página de bienvenida de Django.

![Pagina de Bienvenida de Django](/images/posts/django-email-login/django-bienvenida.png)

Para detener el servidor presiona `Ctrl &#43; c` en la terminal.

## 2. Crea La App Users

Django viene por defecto con un modelo de usuario que es el que utiliza para la autenticación.
Nosotros vamos a modificar este modelo creando uno propio que nos permita utilizar la cuenta de **Correo Electronico** en vez de un **Nombre de Usuario**.

Ahora es momento de crear una app en la cual vamos a modificar el modelo de usuario que viene por defecto con Django.

```bash
(my_project)$ python manage.py startapp users
```

Una vez creada la app, no te olvides de agregarla en INSTALLED*APPS en el archivo \_settings.py*

```settings.py
INSTALLED_APPS = [
    ...
    # other apps
    &#39;users&#39;,
]
```

## 3. Modifica El Archivo users/models.py

A continuación agrega el siguiente codigo al archivo _users/model.py_. Esto es necesario para hacer &#34;email&#34; el campo por defecto en vez de &#34;username&#34;.

```users/models.py
from django.db import models
from django.contrib.auth.models import AbstractBaseUser, PermissionsMixin, BaseUserManager


class CustomUserManager(BaseUserManager):
    def _create_user(self, email, password, first_name, last_name, is_email_verified, **extra_fields):
        if not email:
            raise ValueError(&#34;Email must be provided&#34;)
        if not password:
            raise ValueError(&#34;Password must be provided&#34;)

        user = self.model(
            email=self.normalize_email(email),
            first_name=first_name,
            last_name=last_name,
            is_email_verified=is_email_verified,
            **extra_fields
        )

        user.set_password(password)
        user.save(using=self.db)
        return user

    def create_user(self, email, password, first_name, last_name, is_email_verified, **extra_fields):
        extra_fields.setdefault(&#34;is_staff&#34;, False)
        extra_fields.setdefault(&#34;is_superuser&#34;, False)
        return self._create_user(email, password, first_name, last_name, is_email_verified, **extra_fields)

    def create_superuser(self, email, password, first_name, last_name, is_email_verified, **extra_fields):
        extra_fields.setdefault(&#34;is_staff&#34;, True)
        extra_fields.setdefault(&#34;is_superuser&#34;, True)

        if extra_fields.get(&#34;is_staff&#34;) is not True:
            raise ValueError(&#34;Superuser must have is_staff=True.&#34;)
        if extra_fields.get(&#34;is_superuser&#34;) is not True:
            raise ValueError(&#34;Superuser must have is_superuser=True.&#34;)

        return self._create_user(email, password, first_name, last_name, is_email_verified, **extra_fields)


class User(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(db_index=True, unique=True, max_length=254)
    first_name = models.CharField(max_length=240)
    last_name = models.CharField(max_length=255)
    is_email_verified = models.BooleanField(default=False)

    is_staff = models.BooleanField(default=True)  # Required
    is_active = models.BooleanField(default=True)
    is_superuser = models.BooleanField(default=False)

    objects = CustomUserManager()

    USERNAME_FIELD = &#39;email&#39;
    REQUIRED_FIELDS = [&#39;first_name&#39;, &#39;last_name&#39;, &#39;is_email_verified&#39;]

    class Meta:
        verbose_name = &#39;User&#39;
        verbose_name_plural = &#39;Users&#39;

```

## 4. Modifica Settings.py

Ahora es momento de decirle a Django que utilice nuestro modelo como modelo para la autenticación de los usuarios. Agrega el siguiente codigo al archivo _settings.py_

```settings.py
AUTH_USER_MODEL = &#39;users.User&#39;
```

## 5. Agrega el nuevo modelo a la consola de administración

Como modificamos el modelo del usuario que viene por defecto en Django, ahora lo que debemos hacer es agregar el nuevo modelo a la consola de administración, para eso debemos agregar lo siguiente al archivo _&#34;users/admin.py&#34;_

```users/admin.py
from django.contrib import admin

from .models import User


class CustomUserAdmin(admin.ModelAdmin):
    list_display = (
        &#39;email&#39;,
        # &#39;first_name&#39;,
        # &#39;last_name&#39;,
        &#39;is_email_verified&#39;,
        &#39;is_staff&#39;,
        &#39;is_active&#39;,
    )
    list_filter = (&#39;email&#39;, &#39;is_staff&#39;, &#39;is_email_verified&#39;,)
    fieldsets = (
        (None, {&#39;fields&#39;: (
            &#39;first_name&#39;,
            &#39;last_name&#39;,
            &#39;password&#39;,
            &#39;groups&#39;
        )}),
        (&#39;Permissions&#39;, {&#39;fields&#39;: (
            &#39;user_permissions&#39;,
            &#39;is_email_verified&#39;,
            &#39;is_staff&#39;,
            &#39;is_active&#39;,
            &#39;is_superuser&#39;
        )}),
    )
    search_fields = (&#39;email&#39;,)
    ordering = (&#39;email&#39;,)


admin.site.register(User, CustomUserAdmin)
```

## 6. Crea Las Tablas

A continuación actualiza las tablas en la base de datos corriendo los comandos _makemigrations_ y _migrate_ de la siguiente manera:

```bash
(my_project)$ python manage.py makemigrations
(my_project)$ python manage.py migrate
```

## 7. Crea El Superusuario

Luego vamos a crear el superusuario con el siguiente comando:

```bash
(my_project)$ python manage.py createsuperuser
Email: jorge@linuxfacil.mx
First name: Jorge
Last name: Ramirez
Is email verified: True
Password:
Password (again):
Superuser created successfully.
```

## 8. Prueba La Configuración

Por último vamos a correr el servidor y entrar a la consola de administración.
Ejecuta el servidor usando

```bash
(my_project)$ python manage.py runserver
```

Y en tu navegador abre la siguiente dirección: [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)

Ingresa el correo electrónico y el password del superusuario

Y listo.


---

> Autor: Jorge R  
> URL: http://localhost:1313/posts/como-usar-correo-electronico-para-firmarse-en-django/  

