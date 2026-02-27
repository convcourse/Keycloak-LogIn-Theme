# Keycloak ConvCourse Theme

Tema personalizado de Keycloak para el proyecto ConvCourse, diseñado para coincidir con el estilo visual de [convalid.sakenaura.com](https://convalid.sakenaura.com).

## 🎨 Colores del tema

- **Primary**: Cyan/Blue (#0ea5e9, #38bdf8)
- **Surface**: Dark grays (#0f172a, #1e293b, #334155)
- **Accent**: Green (#10b981)

## 🚀 Instalación

1. Clonar el repositorio
2. Editar `.env` y configurar las credenciales si es necesario
3. Lanzar con Docker Compose:

```bash
docker-compose up -d
```

## 📁 Estructura

```
Keycloak/
├── docker-compose.yml       # Configuración de Docker
├── .env                      # Variables de entorno
├── .gitignore               # Archivos ignorados
├── README.md                # Documentación
└── themes/
    └── convcourse/          # Tema personalizado
        └── login/
            ├── theme.properties
            └── resources/
                ├── css/
                │   └── login.css
                └── img/
```

## ⚙️ Configuración

### Activar el tema en Keycloak

1. Acceder a la consola de administración: https://keycloak.sakenaura.com
2. Ir a **Realm Settings** > **Themes**
3. Seleccionar **convcourse** en **Login theme**
4. Guardar cambios

### Variables de entorno

Editar `.env`:

```env
KC_ADMIN_USERNAME=admin
KC_ADMIN_PASSWORD=tu_password_seguro
KC_HOSTNAME=keycloak.sakenaura.com
```

## 🔧 Desarrollo

Para modificar el tema:

1. Editar archivos en `themes/convcourse/login/resources/css/`
2. Los cambios se aplican automáticamente (el volumen está montado)
3. Refrescar la página de login para ver los cambios

## 🔒 Seguridad

- No commitear el archivo `.env` con credenciales reales
- Cambiar las credenciales por defecto en producción
- Usar contraseñas seguras para el admin

## 📝 Notas

- El tema está basado en el tema `keycloak` por defecto
- Compatible con Keycloak 26.5.4
- Diseñado para funcionar detrás de un reverse proxy (nginx) con SSL
