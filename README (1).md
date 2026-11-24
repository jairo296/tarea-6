# Biblioteca personal (Flask + KeyDB)

Aplicación web para gestionar una biblioteca personal. Está hecha con **Flask**, guarda los datos en **KeyDB** (compatible con Redis, usando `redis-py`) y usa plantillas Jinja2 con **Bootstrap 5**.

## Funcionalidades

- **Agregar** un libro (título, autor, género y estado de lectura).
- **Editar** un libro con el formulario precargado.
- **Eliminar** un libro con ventana de confirmación.
- **Listar** todos los libros en una tabla ordenada por título.
- **Buscar** por título, autor o género (búsqueda parcial, sin distinguir mayúsculas) y filtrar por estado.
- **Alertas visuales** para operaciones exitosas o con error.
- **Validaciones:** campos vacíos, longitudes máximas, estado válido y libros duplicados (mismo título y autor, sin distinguir mayúsculas ni espacios extra).

## Estructura del proyecto

```
biblioteca_flask/
├── app.py              # Aplicación Flask y rutas
├── libros.py           # Lógica de datos: validación y repositorio sobre KeyDB
├── config.py           # Configuración (lee el archivo .env)
├── .env                # Credenciales locales (no se sube a git)
├── .env.example        # Plantilla de .env
├── requirements.txt    # Dependencias
├── templates/          # Vistas HTML (Jinja2)
│   ├── base.html       #   plantilla base: navbar + alertas
│   ├── index.html      #   listado, búsqueda y modal de eliminación
│   ├── form.html       #   formulario de alta / edición
│   └── error.html      #   páginas de error
└── static/
    ├── css/style.css
    └── js/app.js       # completa el modal de confirmación
```

## Instalación y ejecución

### 1. Levantar KeyDB

Con Docker:

```bash
docker run -d --name keydb -p 6379:6379 eqalpha/keydb
```

(o usa una instalación local de KeyDB / Redis en el puerto 6379).

### 2. Instalar dependencias

```bash
python -m venv .venv
source .venv/bin/activate        # En Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Configurar `.env`

Edita `.env` si tu KeyDB no está en `localhost:6379` o si requiere contraseña:

```
KEYDB_HOST=localhost
KEYDB_PORT=6379
KEYDB_DB=0
KEYDB_PASSWORD=
SECRET_KEY=<clave-aleatoria>
FLASK_DEBUG=1
```

Para generar una `SECRET_KEY`: `python -c "import secrets; print(secrets.token_hex(32))"`

### 4. Ejecutar

```bash
python app.py
```

Abre <http://127.0.0.1:5000>.

## Rutas

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/` | Listado; admite `?q=texto&campo=titulo\|autor\|genero\|todos&estado=Leído` |
| GET, POST | `/libros/nuevo` | Formulario y alta de libro |
| GET, POST | `/libros/<id>/editar` | Formulario precargado y actualización |
| POST | `/libros/<id>/eliminar` | Eliminación |

## Modelo de datos en KeyDB

| Clave | Tipo | Contenido |
|-------|------|-----------|
| `libro:<uuid>` | hash | `titulo`, `autor`, `genero`, `estado`, `creado`, `actualizado` |
| `libros:ids` | set | UUID de todos los libros (índice para listar) |
| `libros:unicos` | hash | `titulo\|\|autor` normalizado → UUID (evita duplicados) |

## Notas

- Los formularios incluyen un token **CSRF** guardado en la sesión, por lo que `SECRET_KEY` debe mantenerse en `.env`.
- Jinja2 escapa automáticamente el HTML de los datos, evitando inyección de scripts.
- Si KeyDB no está disponible, la app muestra una página de error 503 con instrucciones en lugar de fallar.
- Bootstrap se carga desde un CDN, por lo que se necesita conexión a internet para ver los estilos.
