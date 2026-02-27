# 📚 Documentación Completa - Configuración Keycloak ConvCourse

**Fecha:** 27 de Febrero de 2026  
**Proyecto:** ConvCourse - Sistema de Validación de Convalidaciones  
**Servidor:** convalid.sakenaura.com

---

## 📋 Índice

1. [Resumen del Sistema](#resumen-del-sistema)
2. [Arquitectura y Componentes](#arquitectura-y-componentes)
3. [Keycloak - Configuración Completa](#keycloak-configuración-completa)
4. [Tema Personalizado](#tema-personalizado)
5. [CI/CD Automático](#cicd-automático)
6. [Troubleshooting](#troubleshooting)
7. [Próximos Pasos](#próximos-pasos)

---

## 🎯 Resumen del Sistema

### Componentes Desplegados

| Componente | URL | Puerto | Estado |
|------------|-----|--------|--------|
| Website (Frontend) | https://convalid.sakenaura.com | 3000 | ✅ Activo |
| Keycloak (Auth) | https://keycloak.sakenaura.com | 8080 | ✅ Activo |
| Backend API | - | 8000 | ⏸️ Pendiente |

### Credenciales Principales

**Keycloak Admin Console:**
- URL: https://keycloak.sakenaura.com/admin
- Usuario: `admin`
- Contraseña: `admin`
- ⚠️ **IMPORTANTE:** Cambiar en producción

**Realm Principal:**
- Nombre: `convalid`
- Cliente: `public-client`
- Tema de Login: `convcourse`

---

## 🏗️ Arquitectura y Componentes

### Estructura de Directorios en Servidor

```
/home/convalid/
├── Website/                    # Frontend Next.js
│   └── docker-compose.yml
├── BackEnd/                    # API Python/FastAPI
│   ├── docker-compose.dev.yml
│   └── docker-compose.prod.yml
├── Keycloak-LogIn-Theme/      # Tema personalizado Keycloak
│   ├── docker-compose.yml
│   ├── .env
│   └── themes/
│       └── convcourse/
│           └── login/
│               ├── login.ftl
│               ├── register.ftl
│               ├── template.ftl
│               ├── theme.properties
│               └── resources/
│                   └── css/
│                       ├── login.css
│                       └── login-v2.css
└── Deployment/                # Scripts CI/CD
    ├── CI-CD.sh
    ├── direction_file.txt
    └── Deploy.log
```

### Red Docker

Todos los servicios comparten la red: **`convcourse-network`**

### Nginx Reverse Proxy

```nginx
# Configuración SSL y dominios
keycloak.sakenaura.com → localhost:8080 (Keycloak)
convalid.sakenaura.com → localhost:3000 (Website)
```

---

## 🔐 Keycloak - Configuración Completa

### Instalación y Despliegue

#### Docker Compose

**Ubicación:** `/home/convalid/Keycloak-LogIn-Theme/docker-compose.yml`

```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:26.5.4
    container_name: keycloak
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      KC_BOOTSTRAP_ADMIN_USERNAME: ${KC_ADMIN_USERNAME:-admin}
      KC_BOOTSTRAP_ADMIN_PASSWORD: ${KC_ADMIN_PASSWORD:-admin}
      KC_HOSTNAME: ${KC_HOSTNAME:-keycloak.sakenaura.com}
      KC_PROXY_HEADERS: xforwarded
      KC_HTTP_ENABLED: "true"
      KC_HOSTNAME_STRICT: "false"
      KC_HOSTNAME_STRICT_HTTPS: "false"
    volumes:
      - keycloak_data:/opt/keycloak/data
      - ./themes:/opt/keycloak/themes
    command: start
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health/ready"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s

volumes:
  keycloak_data:
    name: keycloak_data
```

#### Variables de Entorno

**Archivo:** `.env`

```env
KC_ADMIN_USERNAME=admin
KC_ADMIN_PASSWORD=admin
KC_HOSTNAME=keycloak.sakenaura.com
```

⚠️ **Este archivo NO debe commitearse con credenciales reales**

#### Comandos de Gestión

```bash
# Iniciar Keycloak
cd /home/convalid/Keycloak-LogIn-Theme
docker-compose up -d

# Ver logs
docker logs -f keycloak

# Reiniciar (después de cambios en tema)
docker restart keycloak

# Detener
docker-compose down

# Ver estado
docker ps | grep keycloak
```

### Configuración del Realm "convalid"

#### 1. Crear/Configurar Realm

**Admin Console** → **Dropdown superior izquierda** → **Create Realm**

- **Realm name:** `convalid`
- **Enabled:** ✅ ON
- **Display name:** `Convalid`
- **HTML Display name:** `<strong>Convalid</strong>`

**Save**

#### 2. Configurar Login Settings

**Realm Settings** → **Login** tab:

```
✅ User registration           # Permite registro de usuarios
✅ Edit username               # Usuarios pueden editar su username
✅ Forgot password             # Opción de recuperar contraseña
✅ Remember me                 # Recordar sesión
❌ Verify email                # Deshabilitado (por ahora, sin SMTP)
❌ Login with email            # Solo username (opcional)
✅ Duplicate emails            # Permitir emails duplicados (NO recomendado en prod)
```

**Save**

#### 3. Configurar Temas

**Realm Settings** → **Themes** tab:

```
Login theme: convcourse
Account theme: keycloak.v3 (default)
Admin console theme: keycloak.v3 (default)
Email theme: keycloak (default)
```

**Save**

#### 4. Configurar Cliente "public-client"

**Clients** → **Create client** (o editar si existe)

**General Settings:**
```
Client type: OpenID Connect
Client ID: public-client
Name: Public Client
Description: Cliente público para la aplicación web
```

**Capability config:**
```
Client authentication: OFF (public client)
Authorization: OFF
Authentication flow:
  ✅ Standard flow
  ✅ Direct access grants
  ❌ Implicit flow
  ❌ Service accounts roles
  ❌ OAuth 2.0 Device Authorization Grant
```

**Login settings:**
```
Root URL: https://convalid.sakenaura.com
Home URL: https://convalid.sakenaura.com
Valid redirect URIs:
  - https://convalid.sakenaura.com/*
  - https://keycloak.sakenaura.com/*
  - http://localhost:3000/*
  
Valid post logout redirect URIs:
  - https://convalid.sakenaura.com/*
  - https://keycloak.sakenaura.com/*
  - http://localhost:3000/*

Web origins:
  - *
  
Admin URL: (dejar vacío)
```

**Save**

#### 5. Crear Usuario de Prueba

**Users** → **Create new user**

```
Username: testuser
Email: test@convalid.com
Email verified: ✅ ON
First name: Test
Last name: User
```

**Create**

**Establecer contraseña:**
- Pestaña **Credentials** → **Set password**
- Password: `Test123!`
- Password confirmation: `Test123!`
- Temporary: ❌ OFF

**Save password**

### URLs de Acceso

#### Admin Console
```
https://keycloak.sakenaura.com/admin
```

#### Página de Login
```
https://keycloak.sakenaura.com/realms/convalid/account
```

#### Página de Registro
```
https://keycloak.sakenaura.com/realms/convalid/protocol/openid-connect/auth?client_id=account-console&redirect_uri=https://keycloak.sakenaura.com/realms/convalid/account&response_type=code&scope=openid
```
Luego click en "Register"

#### Testing con Cliente
```
https://keycloak.sakenaura.com/realms/convalid/protocol/openid-connect/auth?client_id=public-client&redirect_uri=https://convalid.sakenaura.com&response_type=code&scope=openid
```

---

## 🎨 Tema Personalizado

### Estructura del Tema

```
themes/convcourse/login/
├── login.ftl              # Template de página de login
├── register.ftl           # Template de página de registro
├── template.ftl           # Template base (header, footer, estructura)
├── theme.properties       # Configuración del tema
└── resources/
    ├── css/
    │   ├── login.css      # Estilos personalizados principal
    │   └── login-v2.css   # Estilos adicionales
    └── img/
        └── favicon.ico    # Favicon personalizado
```

### Archivo theme.properties

```properties
parent=keycloak
import=common/keycloak

styles=css/login.css
```

### Colores del Tema

```css
:root {
    /* Primary colors (cyan/blue) */
    --primary-600: #0ea5e9;
    --primary-500: #38bdf8;
    --primary-400: #7dd3fc;
    --primary-300: #a5f3fc;
    --primary-200: #cffafe;
    
    /* Surface colors (dark grays) */
    --surface-900: #0f172a;  /* Fondo principal */
    --surface-800: #1e293b;  /* Cards */
    --surface-700: #334155;  /* Bordes */
    --surface-600: #475569;
    --surface-500: #64748b;
    --surface-400: #94a3b8;  /* Texto secundario */
    --surface-300: #cbd5e1;  /* Texto principal */
    
    /* Accent color (green) */
    --accent-500: #10b981;
    
    /* Semantic colors */
    --error: #ef4444;
    --success: #10b981;
    --warning: #f59e0b;
    --info: #0ea5e9;
}
```

### Características del Tema

✅ **Fondo oscuro** (#0f172a) con diseño moderno  
✅ **Logo de Convalid** con icono de escudo (shield) SVG  
✅ **Título con gradiente** de blanco a cyan  
✅ **Botones con glow effect** cyan brillante  
✅ **Inputs con borde oscuro** y focus cyan  
✅ **Cards con sombras profundas** y bordes sutiles  
✅ **Responsive design** adaptado a móviles  
✅ **Ancho optimizado:** 550px (se ve completo el email)  

### Actualizar el Tema

```bash
# En local
cd /home/javier/Documentos/convcourse/Keycloak-LogIn-Theme
git add -A
git commit -m "Descripción de cambios"
git push

# En servidor
ssh convalid
cd /home/convalid/Keycloak-LogIn-Theme
git pull
docker restart keycloak

# Ver logs
docker logs --tail 50 -f keycloak
```

⏱️ **Esperar ~30 segundos** para que Keycloak reinicie completamente

---

## 🔄 CI/CD Automático

### Script CI-CD.sh

**Ubicación:** `/home/convalid/Deployment/CI-CD.sh`

**Función:** Verifica cambios en repositorios cada 5 minutos y despliega automáticamente.

#### Características

✅ Rutas absolutas (no depende del directorio de ejecución)  
✅ Detección automática de rama actual  
✅ Manejo robusto de errores  
✅ Logs detallados con timestamps  
✅ Timeout en operaciones git (30s)  
✅ Verificación de existencia de archivos  
✅ Creación automática de red Docker  
✅ Soporte para múltiples repositorios  

#### Archivo direction_file.txt

**Ubicación:** `/home/convalid/Deployment/direction_file.txt`

```
# Directorios de repositorios a monitorear
# Rutas relativas al directorio de este script

../Website
../BackEnd
../Keycloak-LogIn-Theme
```

#### Configuración Crontab

```bash
# Editar crontab
crontab -e

# Añadir esta línea (ejecuta cada 5 minutos)
*/5 * * * * cd /home/convalid/Deployment && /home/convalid/Deployment/CI-CD.sh >> /home/convalid/Deployment/cron.log 2>&1
```

#### Ver Logs

```bash
# Logs del script
tail -f /home/convalid/Deployment/Deploy.log

# Logs del cron
tail -f /home/convalid/Deployment/cron.log

# Ver crontab actual
crontab -l
```

#### Ejecutar Manualmente

```bash
cd /home/convalid/Deployment
./CI-CD.sh
```

#### Flujo de Trabajo

1. **Cada 5 minutos** el cron ejecuta el script
2. Para cada repositorio en `direction_file.txt`:
   - Detecta la rama actual
   - Hace `git fetch origin`
   - Compara commit local vs remoto
   - Si hay cambios:
     - Hace `git pull`
     - Ejecuta `make build-prod`
     - Ejecuta `make start-prod`
     - Registra en logs
3. Si no hay cambios, solo registra estado
4. Continúa con el siguiente repositorio

---

## 🐛 Troubleshooting

### Keycloak no Arranca

**Problema:** El contenedor de Keycloak no inicia

```bash
# Ver logs completos
docker logs keycloak

# Verificar healthcheck
docker inspect keycloak | grep -A 10 Health

# Común: Puerto ocupado
sudo netstat -tlnp | grep 8080

# Reiniciar
docker restart keycloak
```

### Tema no se Aplica

**Problema:** Los cambios del tema no se ven

**Soluciones:**

1. **Limpiar caché del navegador:** `Ctrl + Shift + R`

2. **Verificar que el tema está montado:**
```bash
docker exec keycloak ls -la /opt/keycloak/themes/convcourse
```

3. **Verificar configuración en Keycloak:**
   - Admin Console → Realm convalid → Realm Settings → Themes
   - Login theme debe ser: `convcourse`

4. **Reiniciar Keycloak:**
```bash
docker restart keycloak
```

5. **Verificar logs:**
```bash
docker logs --tail 100 keycloak | grep -i error
```

### Error "Invalid parameter: redirect_uri"

**Problema:** Al intentar login/registro aparece error de redirect_uri

**Solución:**

1. **Verificar Valid Redirect URIs en el cliente:**
   - Clients → `public-client` → Settings
   - Valid redirect URIs debe incluir:
     - `https://convalid.sakenaura.com/*`
     - `https://keycloak.sakenaura.com/*`

2. **Verificar Web origins:**
   - Debe ser: `*` o las URLs específicas

### CI/CD no Ejecuta

**Problema:** Los cambios no se despliegan automáticamente

**Diagnóstico:**

```bash
# Verificar crontab
crontab -l

# Ver logs del cron
tail -50 /home/convalid/Deployment/cron.log

# Ver logs del script
tail -50 /home/convalid/Deployment/Deploy.log

# Ejecutar manualmente para ver errores
cd /home/convalid/Deployment
./CI-CD.sh
```

**Soluciones comunes:**

1. **Permisos de ejecución:**
```bash
chmod +x /home/convalid/Deployment/CI-CD.sh
```

2. **Credenciales Git:**
```bash
gh auth status
# Si no está autenticado:
gh auth login
```

3. **Usuario no en grupo docker:**
```bash
groups
# Si no aparece 'docker':
sudo usermod -aG docker convalid
# Cerrar sesión y volver a entrar
```

### Timeout del Login (3rd party iframe)

**Problema:** "Timeout when waiting for 3rd party check iframe message"

**Causa:** Problema de comunicación entre aplicación y Keycloak por proxy/CORS

**Solución:** Ya está configurado con:
```yaml
KC_PROXY_HEADERS: xforwarded
KC_HOSTNAME_STRICT: "false"
KC_HOSTNAME_STRICT_HTTPS: "false"
```

Si persiste:
1. Verificar configuración de nginx
2. Verificar que las cabeceras `X-Forwarded-*` se estén enviando
3. Verificar Web origins en el cliente

### Datos se Pierden al Reiniciar

**Problema:** Al reiniciar el servidor se pierden usuarios/configuraciones

**Causa:** No hay volumen persistente

**Verificación:**
```bash
docker volume ls | grep keycloak
# Debe aparecer: keycloak_data
```

**Solución:** Ya está configurado en docker-compose.yml:
```yaml
volumes:
  - keycloak_data:/opt/keycloak/data
```

Si no existe:
```bash
docker volume create keycloak_data
docker-compose down
docker-compose up -d
```

---

## 🚀 Próximos Pasos

### Configuración Pendiente

#### 1. Verificación de Email

**Cuando esté listo para producción:**

1. **Configurar servidor SMTP:**
   - Realm Settings → Email tab
   - Host: `smtp.gmail.com` (o tu servidor)
   - Port: `587`
   - Enable StartTLS: ✅
   - Username: tu email
   - Password: App Password de Google

2. **Habilitar verificación:**
   - Realm Settings → Login tab
   - ✅ Verify email

3. **Crear templates personalizados:**
```
themes/convcourse/email/
├── html/
│   └── email-verification.ftl
└── text/
    └── email-verification.ftl
```

#### 2. Cambiar Credenciales de Admin

**IMPORTANTE en producción:**

```bash
# Editar .env
KC_ADMIN_USERNAME=admin_produccion
KC_ADMIN_PASSWORD=ContraseñaSegura123!@#

# Reiniciar
docker-compose down
docker-compose up -d
```

#### 3. Configurar Backend

**Pendiente:** Lanzar el backend Python en puerto 8000

```bash
cd /home/convalid/BackEnd
# Configurar .env con credenciales de base de datos
docker-compose -f docker-compose.prod.yml up -d
```

#### 4. Integración con Frontend

**En la aplicación Next.js:**

Configurar variables de entorno:
```env
KEYCLOAK_ID=public-client
KEYCLOAK_SECRET= # vacío para public client
KEYCLOAK_ISSUER=https://keycloak.sakenaura.com/realms/convalid
```

#### 5. Mejorar Seguridad

**Checklist de seguridad:**

- [ ] Cambiar contraseña de admin
- [ ] Habilitar verificación de email
- [ ] Configurar rate limiting en nginx
- [ ] Configurar CORS correctamente (no usar `*` en producción)
- [ ] Habilitar 2FA para admin
- [ ] Configurar políticas de contraseña robustas
- [ ] Revisar logs periódicamente
- [ ] Backups automáticos de volumen keycloak_data

#### 6. Monitorización

**Configurar alertas:**

```bash
# Monitorear logs de errores
tail -f /home/convalid/Deployment/Deploy.log | grep ERROR

# Monitorear estado de contenedores
watch docker ps
```

#### 7. Backups

**Script de backup para volumen Keycloak:**

```bash
#!/bin/bash
# backup-keycloak.sh
DATE=$(date +%Y%m%d_%H%M%S)
docker run --rm \
  -v keycloak_data:/data \
  -v /home/convalid/backups:/backup \
  alpine tar czf /backup/keycloak_backup_$DATE.tar.gz /data

# Mantener solo últimos 7 backups
cd /home/convalid/backups
ls -t keycloak_backup_*.tar.gz | tail -n +8 | xargs rm -f
```

Añadir a crontab (diario a las 3 AM):
```cron
0 3 * * * /home/convalid/scripts/backup-keycloak.sh
```

---

## 📞 Soporte y Recursos

### Documentación Oficial

- **Keycloak:** https://www.keycloak.org/documentation
- **Keycloak Themes:** https://www.keycloak.org/docs/latest/server_development/#_themes
- **Docker Keycloak:** https://www.keycloak.org/server/containers

### Repositorios del Proyecto

- **Website:** https://github.com/convcourse/Website
- **Backend:** https://github.com/convcourse/BackEnd
- **Keycloak Theme:** https://github.com/convcourse/Keycloak-LogIn-Theme
- **Deployment:** https://github.com/convcourse/Deployment

### Estructura de Commits Recomendada

```
feat: Nueva funcionalidad
fix: Corrección de bug
docs: Cambios en documentación
style: Cambios de formato/estilo
refactor: Refactorización de código
test: Añadir tests
chore: Tareas de mantenimiento
```

---

## 📝 Notas Finales

### Estado Actual del Sistema

✅ **Funcionando:**
- Keycloak con persistencia de datos
- Tema personalizado aplicado
- CI/CD automático configurado
- Login y registro funcionales

⏸️ **Pendiente:**
- Verificación de email (sin SMTP)
- Backend API deployment
- Integración completa con frontend

⚠️ **Para Producción:**
- Cambiar credenciales por defecto
- Configurar SMTP
- Mejorar seguridad (CORS, rate limiting)
- Implementar backups automáticos
- Configurar monitorización

### Autor

**Fecha de configuración:** 27 de Febrero de 2026  
**Configurado por:** GitHub Copilot CLI + Javier  
**Entorno:** Servidor casero - convalid.sakenaura.com

---

**Última actualización:** 27/02/2026 12:42 UTC
