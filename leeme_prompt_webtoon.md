Perfecto — te doy **la primera parte completa y lista para ejecutar**: desde crear la carpeta del proyecto en tu equipo hasta tener un CRUD funcional (solo **Usuario**) con templates, rutas, registro en admin y servidor en el puerto **8001**. Todo en **Python + Django** usando **VS Code**. Voy paso a paso, numerado y con los archivos de código que necesitas pegar.

---

# 1 — Crear carpeta del proyecto

En tu sistema de archivos crea la carpeta:

```
UIII_Webtoon_0456
```

---

# 2 — Abrir VS Code sobre la carpeta

Abre VS Code y:

* Menú **File > Open Folder...** -> selecciona `UIII_Webtoon_0456`
  O desde terminal (si `code` está en tu PATH):

```bash
code UIII_Webtoon_0456
```

---

# 3 — Abrir terminal en VS Code

Dentro de VS Code: usa el atajo:

```
Ctrl + `
```

(o menú **Terminal > New Terminal**). La terminal se abrirá en la raíz `UIII_Webtoon_0456`.

---

# 4 — Crear carpeta entorno virtual `.venv` desde terminal de VS Code

Desde la terminal (en la raíz `UIII_Webtoon_0456`):

```bash
python -m venv .venv
```

---

# 5 — Activar el entorno virtual

* En Windows (PowerShell):

```powershell
.venv\Scripts\Activate.ps1
```

* En Windows (cmd):

```cmd
.venv\Scripts\activate
```

* En macOS / Linux:

```bash
source .venv/bin/activate
```

Nota: si usas PowerShell y hay restricción de ejecución, ejecuta como admin `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` (solo si sabes lo que haces).

---

# 6 — Activar intérprete de Python en VS Code

1. Presiona `Ctrl+Shift+P`.
2. Escribe `Python: Select Interpreter`.
3. Selecciona el intérprete ubicado en `UIII_Webtoon_0456/.venv/...` (aparece como `.venv`).

También podrás hacer clic en la esquina inferior derecha donde aparece el intérprete y elegir `.venv`.

---

# 7 — Instalar Django

Con el entorno activo:

```bash
pip install django
```

(Verás que instala la versión disponible en tu entorno).

---

# 8 — Crear proyecto `backend_Webtoon` **sin duplicar carpeta**

Importante: si ejecutas `django-admin startproject backend_Webtoon` desde la carpeta raíz, Django creará otra carpeta `backend_Webtoon` dentro de la actual. Para que no duplique, haz **esto** (ejecutarlo desde `UIII_Webtoon_0456`):

```bash
# Crear el proyecto en la carpeta actual con el "dot"
django-admin startproject backend_Webtoon .
```

El `.` al final hace que los archivos del proyecto se creen **en la carpeta actual** en lugar de crear una subcarpeta extra.

Estructura mínima resultante:

```
UIII_Webtoon_0456/
  manage.py
  backend_Webtoon/
    __init__.py
    settings.py
    urls.py
    wsgi.py
    asgi.py
```

---

# 9 — Ejecutar servidor en el puerto 8001

Desde la raíz (donde está `manage.py`), primero aplicar migraciones (ver sección 12). Luego:

```bash
python manage.py runserver 8001
```

---

# 10 — Copiar y pegar el link en el navegador

Abre:

```
http://127.0.0.1:8001/
```

ó

```
http://localhost:8001/
```

(puedes copiar la URL que te muestre la terminal).

---

# 11 — Crear aplicación `app_Webtoon`

En la raíz del proyecto:

```bash
python manage.py startapp app_Webtoon
```

Esto crea la carpeta `app_Webtoon` con `views.py`, `models.py`, etc.

---

# 12 — Archivo `models.py` (solo **Usuario** por ahora)

Como pediste, **por lo pronto solo trabajamos con el MODELO: USUARIO**. Deja pendiente Webtoon y Membresía.

Reemplaza el `app_Webtoon/models.py` por este contenido:

```python
# app_Webtoon/models.py
from django.db import models

# ==========================================
# MODELO: USUARIO
# ==========================================
class Usuario(models.Model):
    nombre_usuario = models.CharField(max_length=50)
    email = models.EmailField(unique=True)
    contraseña = models.CharField(max_length=128)
    fecha_registro = models.DateTimeField(auto_now_add=True)
    avatar = models.CharField(max_length=255, blank=True, null=True)
    biografia = models.TextField(blank=True, null=True)
    tipo_usuario = models.CharField(
        max_length=20,
        choices=[
            ('Lector', 'Lector'),
            ('Autor', 'Autor'),
            ('Admin', 'Admin')
        ],
        default='Lector'
    )

    def __str__(self):
        return self.nombre_usuario

# ==========================================
# NOTA: MODELO: WEBTOON y MODELO: MEMBRESIA
# Los dejamos pendientes para más tarde (como solicitaste).
# ==========================================
```

---

# 12.5 — Procedimiento para realizar migraciones

En terminal (con venv activo):

1. Crear migraciones de la app:

```bash
python manage.py makemigrations app_Webtoon
```

2. Aplicar migraciones:

```bash
python manage.py migrate
```

---

# 13 — Trabajamos primero con el MODELO: USUARIO

(Ya definido arriba)

---

# 14 — `views.py` en `app_Webtoon` (funciones solicitadas)

Crea/edita `app_Webtoon/views.py` con este código. No usamos `forms.py`; manejamos `request.POST` directo. Para guardar contraseña la **hasheamos** con `make_password` (mejora básica de seguridad aunque pediste sin validación).

```python
# app_Webtoon/views.py
from django.shortcuts import render, redirect, get_object_or_404
from .models import Usuario
from django.contrib import messages
from django.contrib.auth.hashers import make_password

def inicio_webtoon(request):
    usuarios = Usuario.objects.all().order_by('-fecha_registro')
    return render(request, 'inicio.html', {'usuarios': usuarios})

def agregar_usuario(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre_usuario', '')
        email = request.POST.get('email', '')
        contraseña = request.POST.get('contraseña', '')
        avatar = request.POST.get('avatar', '')
        biografia = request.POST.get('biografia', '')
        tipo = request.POST.get('tipo_usuario', 'Lector')

        # Guardar (hasheando la contraseña)
        usuario = Usuario(
            nombre_usuario=nombre,
            email=email,
            contraseña=make_password(contraseña),
            avatar=avatar,
            biografia=biografia,
            tipo_usuario=tipo
        )
        usuario.save()
        # messages.success(request, 'Usuario agregado')
        return redirect('app_Webtoon:ver_usuario')
    return render(request, 'usuario/agregar_usuario.html')

def ver_usuario(request):
    usuarios = Usuario.objects.all().order_by('id')
    return render(request, 'usuario/ver_usuario.html', {'usuarios': usuarios})

def actualizar_usuario(request, usuario_id):
    usuario = get_object_or_404(Usuario, id=usuario_id)
    return render(request, 'usuario/actualizar_usuario.html', {'usuario': usuario})

def realizar_actualizacion_usuario(request, usuario_id):
    usuario = get_object_or_404(Usuario, id=usuario_id)
    if request.method == 'POST':
        usuario.nombre_usuario = request.POST.get('nombre_usuario', usuario.nombre_usuario)
        usuario.email = request.POST.get('email', usuario.email)
        nueva_contraseña = request.POST.get('contraseña', '')
        if nueva_contraseña:
            usuario.contraseña = make_password(nueva_contraseña)
        usuario.avatar = request.POST.get('avatar', usuario.avatar)
        usuario.biografia = request.POST.get('biografia', usuario.biografia)
        usuario.tipo_usuario = request.POST.get('tipo_usuario', usuario.tipo_usuario)
        usuario.save()
        return redirect('app_Webtoon:ver_usuario')
    return redirect('app_Webtoon:actualizar_usuario', usuario_id=usuario.id)

def borrar_usuario(request, usuario_id):
    usuario = get_object_or_404(Usuario, id=usuario_id)
    if request.method == 'POST':
        usuario.delete()
        return redirect('app_Webtoon:ver_usuario')
    return render(request, 'usuario/borrar_usuario.html', {'usuario': usuario})
```

---

# 15 — Crear carpeta `templates` dentro de `app_Webtoon`

Ruta:

```
app_Webtoon/templates/
```

---

# 16 — Archivos HTML en `templates`: `base.html`, `header.html`, `navbar.html`, `footer.html`, `inicio.html`

Crea estos archivos dentro de `app_Webtoon/templates/`. Las vistas lo referenciarán.

## `base.html`

```html
<!-- app_Webtoon/templates/base.html -->
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Sistema de Administración Webtoon{% endblock %}</title>

    <!-- Bootstrap CSS CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Bootstrap Icons -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css" rel="stylesheet">

    <style>
      :root{
        --bg-soft:#f6f8fb;
        --card:#ffffff;
        --accent:#7aa7d9;
        --muted:#6b7280;
      }
      body { background: var(--bg-soft); color: #222; }
      .navbar-brand { font-weight:700; color: #2b4162; }
      footer{ background: #f1f5f9; padding: 12px 0; color: #333; position: fixed; bottom: 0; width:100%;}
      .container-main{ padding-bottom: 80px; } /* espacio para footer fijo */
    </style>
  </head>
  <body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}

    <main class="container container-main mt-4">
      {% block content %}{% endblock %}
    </main>

    {% include 'footer.html' %}

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
  </body>
</html>
```

## `header.html`

```html
<!-- app_Webtoon/templates/header.html -->
<header class="py-3" style="background: linear-gradient(90deg,#f7fbff,#eef7ff);">
  <div class="container d-flex align-items-center">
    <h1 class="me-3 mb-0" style="font-size:1.25rem;">COMECT - Sistema de Administración Webtoon</h1>
    <small class="text-muted">Panel de gestión</small>
  </div>
</header>
```

## `navbar.html`

```html
<!-- app_Webtoon/templates/navbar.html -->
<nav class="navbar navbar-expand-lg navbar-light" style="background:transparent;">
  <div class="container">
    <a class="navbar-brand" href="{% url 'app_Webtoon:inicio' %}">
      <i class="bi bi-book"></i> Sistema de Administración Webtoon
    </a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navmenu">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navmenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'app_Webtoon:inicio' %}"><i class="bi bi-house-door"></i> Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"><i class="bi bi-people"></i> Usuario</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'app_Webtoon:agregar_usuario' %}">Agregar Usuario</a></li>
            <li><a class="dropdown-item" href="{% url 'app_Webtoon:ver_usuario' %}">Ver Usuario</a></li>
            <li><a class="dropdown-item" href="#">Actualizar Usuario</a></li>
            <li><a class="dropdown-item" href="#">Borrar Usuario</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"><i class="bi bi-journal-bookmark"></i> Webtoons</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar webtoons</a></li>
            <li><a class="dropdown-item" href="#">Ver webtoons</a></li>
            <li><a class="dropdown-item" href="#">Actualizar webtoons</a></li>
            <li><a class="dropdown-item" href="#">Borrar webtoons</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"><i class="bi bi-award"></i> Membresías</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar membresía</a></li>
            <li><a class="dropdown-item" href="#">Ver membresías</a></li>
            <li><a class="dropdown-item" href="#">Actualizar membresía</a></li>
            <li><a class="dropdown-item" href="#">Borrar membresía</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

## `footer.html`

```html
<!-- app_Webtoon/templates/footer.html -->
<footer class="text-center">
  <div class="container">
    <small>© {{ now.year }} | Todos los derechos reservados | Creado por Tec. Ingrid Alvarado, Cbtis 128</small>
  </div>
</footer>
```

> En tus vistas o context processors puedes enviar `now` con datetime; para simplicidad puedes usar `{% now "Y" %}` en plantilla si no quieres modificar vistas. Ejemplo: `© {% now "Y" %}`.

## `inicio.html`

```html
<!-- app_Webtoon/templates/inicio.html -->
{% extends 'base.html' %}
{% block title %}Inicio - Webtoon{% endblock %}

{% block content %}
<div class="row">
  <div class="col-md-8">
    <div class="card mb-3">
      <div class="card-body">
        <h4>Sistema Webtoon</h4>
        <p>Bienvenido al sistema de administración. Aquí puedes gestionar usuarios y (más adelante) webtoons y membresías.</p>
      </div>
    </div>

    <div class="card">
      <div class="card-body">
        <h5>Usuarios recientes</h5>
        {% if usuarios %}
          <ul class="list-group">
            {% for u in usuarios|slice:":5" %}
              <li class="list-group-item d-flex justify-content-between align-items-center">
                {{ u.nombre_usuario }} <span class="text-muted small">{{ u.tipo_usuario }}</span>
              </li>
            {% endfor %}
          </ul>
        {% else %}
          <p class="text-muted">No hay usuarios aún.</p>
        {% endif %}
      </div>
    </div>
  </div>

  <div class="col-md-4">
    <div class="card">
      <img src="https://images.pexels.com/photos/459653/pexels-photo-459653.jpeg" class="card-img-top" alt="webtoon">
      <div class="card-body">
        <h6 class="card-title">¿Qué es un Webtoon?</h6>
        <p class="card-text">Un webtoon es un cómic digital optimizado para lectura en pantalla, especialmente dispositivos móviles.</p>
      </div>
    </div>
  </div>
</div>
{% endblock %}
```

---

# 21 — Subcarpeta `usuario` dentro de templates

Crea:

```
app_Webtoon/templates/usuario/
```

---

# 22 — Templates para usuario: `agregar_usuario.html`, `ver_usuario.html`, `actualizar_usuario.html`, `borrar_usuario.html`

## `agregar_usuario.html`

```html
<!-- app_Webtoon/templates/usuario/agregar_usuario.html -->
{% extends 'base.html' %}
{% block content %}
<h3>Agregar Usuario</h3>
<form method="post">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input class="form-control" name="nombre_usuario">
  </div>
  <div class="mb-3">
    <label class="form-label">Email</label>
    <input class="form-control" name="email" type="email">
  </div>
  <div class="mb-3">
    <label class="form-label">Contraseña</label>
    <input class="form-control" name="contraseña" type="password">
  </div>
  <div class="mb-3">
    <label class="form-label">Avatar (URL)</label>
    <input class="form-control" name="avatar">
  </div>
  <div class="mb-3">
    <label class="form-label">Biografía</label>
    <textarea class="form-control" name="biografia"></textarea>
  </div>
  <div class="mb-3">
    <label class="form-label">Tipo de usuario</label>
    <select class="form-select" name="tipo_usuario">
      <option value="Lector">Lector</option>
      <option value="Autor">Autor</option>
      <option value="Admin">Admin</option>
    </select>
  </div>
  <button class="btn btn-primary" type="submit">Guardar</button>
</form>
{% endblock %}
```

## `ver_usuario.html`

```html
<!-- app_Webtoon/templates/usuario/ver_usuario.html -->
{% extends 'base.html' %}
{% block content %}
<h3>Usuarios</h3>
<table class="table table-striped">
  <thead>
    <tr>
      <th>ID</th>
      <th>Nombre</th>
      <th>Email</th>
      <th>Tipo</th>
      <th>Acciones</th>
    </tr>
  </thead>
  <tbody>
    {% for u in usuarios %}
    <tr>
      <td>{{ u.id }}</td>
      <td>{{ u.nombre_usuario }}</td>
      <td>{{ u.email }}</td>
      <td>{{ u.tipo_usuario }}</td>
      <td>
        <a class="btn btn-sm btn-info" href="{% url 'app_Webtoon:actualizar_usuario' u.id %}">Editar</a>
        <a class="btn btn-sm btn-danger" href="{% url 'app_Webtoon:borrar_usuario' u.id %}">Borrar</a>
      </td>
    </tr>
    {% empty %}
    <tr><td colspan="5">No hay usuarios</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}
```

## `actualizar_usuario.html`

```html
<!-- app_Webtoon/templates/usuario/actualizar_usuario.html -->
{% extends 'base.html' %}
{% block content %}
<h3>Actualizar Usuario</h3>
<form method="post" action="{% url 'app_Webtoon:realizar_actualizacion_usuario' usuario.id %}">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input class="form-control" name="nombre_usuario" value="{{ usuario.nombre_usuario }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Email</label>
    <input class="form-control" name="email" type="email" value="{{ usuario.email }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Contraseña (si la dejas en blanco no cambia)</label>
    <input class="form-control" name="contraseña" type="password">
  </div>
  <div class="mb-3">
    <label class="form-label">Avatar (URL)</label>
    <input class="form-control" name="avatar" value="{{ usuario.avatar }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Biografía</label>
    <textarea class="form-control" name="biografia">{{ usuario.biografia }}</textarea>
  </div>
  <div class="mb-3">
    <label class="form-label">Tipo de usuario</label>
    <select class="form-select" name="tipo_usuario">
      <option value="Lector" {% if usuario.tipo_usuario == 'Lector' %}selected{% endif %}>Lector</option>
      <option value="Autor" {% if usuario.tipo_usuario == 'Autor' %}selected{% endif %}>Autor</option>
      <option value="Admin" {% if usuario.tipo_usuario == 'Admin' %}selected{% endif %}>Admin</option>
    </select>
  </div>
  <button class="btn btn-success" type="submit">Actualizar</button>
</form>
{% endblock %}
```

## `borrar_usuario.html`

```html
<!-- app_Webtoon/templates/usuario/borrar_usuario.html -->
{% extends 'base.html' %}
{% block content %}
<h3>Confirmar borrado</h3>
<p>¿Está seguro que desea eliminar al usuario <strong>{{ usuario.nombre_usuario }}</strong> (ID: {{ usuario.id }})?</p>
<form method="post">
  {% csrf_token %}
  <a class="btn btn-secondary" href="{% url 'app_Webtoon:ver_usuario' %}">Cancelar</a>
  <button class="btn btn-danger" type="submit">Sí, borrar</button>
</form>
{% endblock %}
```

---

# 23 — No utilizar `forms.py`

Todo el ejemplo usa `request.POST` directo tal como pediste. (Sin validaciones adicionales.)

---

# 24 — `urls.py` en `app_Webtoon`

Crea `app_Webtoon/urls.py` con:

```python
# app_Webtoon/urls.py
from django.urls import path
from . import views

app_name = 'app_Webtoon'

urlpatterns = [
    path('', views.inicio_webtoon, name='inicio'),
    path('usuario/agregar/', views.agregar_usuario, name='agregar_usuario'),
    path('usuario/ver/', views.ver_usuario, name='ver_usuario'),
    path('usuario/actualizar/<int:usuario_id>/', views.actualizar_usuario, name='actualizar_usuario'),
    path('usuario/realizar_actualizacion/<int:usuario_id>/', views.realizar_actualizacion_usuario, name='realizar_actualizacion_usuario'),
    path('usuario/borrar/<int:usuario_id>/', views.borrar_usuario, name='borrar_usuario'),
]
```

---

# 25 — Agregar `app_Webtoon` en `settings.py` de `backend_Webtoon`

En `backend_Webtoon/settings.py`:

1. En `INSTALLED_APPS` añade `'app_Webtoon',`

```python
INSTALLED_APPS = [
    # ... apps por defecto
    'app_Webtoon',
]
```

2. Añade la ruta de templates (opcional si Django detecta templates dentro de cada app, pero para seguridad):

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

TEMPLATES = [
    {
        ...
        'DIRS': [os.path.join(BASE_DIR, 'app_Webtoon', 'templates')],
        ...
    },
]
```

3. (Opcional) Configurar `STATIC_URL` si vas a usar archivos estáticos:

```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
```

---

# 26 — Configurar `urls.py` de `backend_Webtoon` para enlazar `app_Webtoon`

En `backend_Webtoon/urls.py` reemplaza por:

```python
# backend_Webtoon/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Webtoon.urls', namespace='app_Webtoon')),
]
```

---

# 27 — Registrar modelos en `admin.py` y volver a realizar migraciones

En `app_Webtoon/admin.py`:

```python
from django.contrib import admin
from .models import Usuario

@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    list_display = ('id', 'nombre_usuario', 'email', 'tipo_usuario', 'fecha_registro')
    search_fields = ('nombre_usuario', 'email')
```

Luego crear migraciones (si no las hiciste ya):

```bash
python manage.py makemigrations
python manage.py migrate
```

---

# 27 (nota) — Por ahora solo categoría Usuario

Como pediste, dejaremos pendientes Webtoon y Membresía (no los usamos activamente todavía).

---

# 28 — Diseño: colores suaves, sencillos y modernos

En los templates se usó Bootstrap con paleta suave (`--bg-soft`, `--accent`). Puedes adaptar fácilmente los colores en `base.html` `<style>`.

---

# 29 — Al inicio crear la estructura completa de carpetas y archivos

Estructura mínima recomendada:

```
UIII_Webtoon_0456/
├─ .venv/
├─ backend_Webtoon/
│  ├─ settings.py
│  └─ urls.py
├─ app_Webtoon/
│  ├─ migrations/
│  ├─ templates/
│  │  ├─ inicio.html
│  │  ├─ base.html
│  │  ├─ header.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  └─ usuario/
│  │     ├─ agregar_usuario.html
│  │     ├─ ver_usuario.html
│  │     ├─ actualizar_usuario.html
│  │     └─ borrar_usuario.html
│  ├─ models.py
│  ├─ views.py
│  ├─ urls.py
│  └─ admin.py
├─ manage.py
```

---

# 30 — Proyecto totalmente funcional (Usuario CRUD)

Con los archivos indicados y los pasos dados, tendrás el CRUD de usuarios funcionando.

---

# 31 — Finalmente: ejecutar servidor en el puerto 8001

Asegúrate de haber aplicado migraciones:

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 8001
```

Luego abre:

```
http://127.0.0.1:8001/
```

---

### Recomendaciones finales y notas rápidas

* Si quieres acceder al admin: crea superusuario:

  ```bash
  python manage.py createsuperuser
  ```

  y visita `http://127.0.0.1:8001/admin/`.
* Los templates usan imágenes externas (red). Si necesitas assets locales, crea `static/` y configura `STATICFILES_DIRS`.
* No se hizo validación (tal como pediste). Si más adelante quieres agregar validación o `forms.py`, lo integramos.
* Guardé la contraseña hashed con `make_password` — útil para seguridad si luego se implementa login.

---

¿Quieres que:

* Te entregue todos esos archivos listos para descargar como un .zip?
* O continúo ahora mismo con los modelos `Webtoon` y `Membresia` y las relaciones cuando tú me digas?

