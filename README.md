# Email Verification Page - PRAGMA Studio

## Descripción

Página de verificación de email en HTML puro, con estilo moderno similar a PRAGMA Studio, diseñada para integrarse con Supabase y ejecutarse en Vercel.

## Features

✅ Diseño responsivo y moderno (similar a pragmastudio.com.ar)
✅ Estados dinámicos: Cargando → Éxito/Error
✅ Integración con Supabase
✅ Animaciones suaves
✅ Optimizado para mobile
✅ Ready para Vercel deployment

## Instalación

### 1. Clonar o crear proyecto

```bash
# Crear carpeta del proyecto
mkdir email-verification
cd email-verification

# Copiar los archivos
# - verification.html
# - vercel.json
# - README.md (este archivo)
```

### 2. Configurar Supabase

#### En tu proyecto de Supabase:

1. **Ir a Authentication → Email Templates**
   - Editar el template "Confirm signup"
   - Agregar el enlace que apunte a tu URL de Vercel:

```html
<a href="https://tu-proyecto.vercel.app?token={{ .ConfirmationURL }}&type=signup">
  Verificar email
</a>
```

O si usas variables personalizadas:

```html
<a href="https://tu-proyecto.vercel.app?token={{ .TokenHash }}&type=email_change">
  Confirmar cambio de email
</a>
```

2. **Obtener credenciales**
   - URL del proyecto: Settings → API → Project URL
   - Anon Key: Settings → API → Project API keys → anon/public

3. **Reemplazar en verification.html**

```javascript
const SUPABASE_URL = 'https://tu-proyecto.supabase.co';
const SUPABASE_ANON_KEY = 'tu-anon-key';
```

### 3. Variables de entorno en Vercel (Opcional)

Puedes usar variables de entorno en Vercel:

```
VITE_SUPABASE_URL=https://tu-proyecto.supabase.co
VITE_SUPABASE_ANON_KEY=tu-anon-key
```

Luego actualizar el código:

```javascript
const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL || 'https://tu-proyecto.supabase.co';
const SUPABASE_ANON_KEY = import.meta.env.VITE_SUPABASE_ANON_KEY || 'tu-anon-key';
```

### 4. Desplegar en Vercel

#### Opción A: Desde GitHub

1. Crear repositorio en GitHub
2. Push de los archivos:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/tu-usuario/tu-repo.git
git push -u origin main
```

3. En Vercel:
   - Conectar GitHub
   - Importar proyecto
   - No necesita configuración especial (vercel.json está incluido)
   - Deploy automático

#### Opción B: CLI de Vercel

```bash
# Instalar Vercel CLI
npm i -g vercel

# Desplegar
vercel

# Seleccionar proyecto o crear uno nuevo
# Seguir las instrucciones
```

#### Opción C: Drag & Drop

1. Ir a vercel.com
2. Drag & drop de los archivos en la interfaz
3. Vercel crea el proyecto automáticamente

### 5. Configurar redirect en tu app

En tu aplicación (cuando el usuario se registra), enviar email con:

```
https://tu-proyecto.vercel.app?token=TOKEN&type=signup
```

O si usas Auth0/similar:

```
https://tu-proyecto.vercel.app?token=${session.user.id}&type=email_confirm
```

## Personalización

### Cambiar colores

Editar las CSS variables en `<style>`:

```css
:root {
    --primary: #0ea5e9;        /* Color principal (azul) */
    --primary-dark: #0284c7;   /* Azul oscuro */
    --success-color: #10b981;  /* Verde */
    --error-color: #ef4444;    /* Rojo */
}
```

### Cambiar textos

Buscar las secciones:

```html
<!-- Estado: Éxito -->
<h1 class="success-title">¡Email verificado!</h1>
<p class="success-message">Tu cuenta ha sido verificada correctamente...</p>
```

### Cambiar URLs de redirección

Actualizar las funciones:

```javascript
function redirectToApp() {
    window.location.href = 'https://tu-app.vercel.app/dashboard';
}

function requestNewLink() {
    window.location.href = 'https://tu-app.vercel.app/resend-verification';
}

function goHome() {
    window.location.href = 'https://pragmastudio.com.ar';
}
```

## Integración con Supabase (Backend)

### Crear función SQL para verificación

```sql
-- En Supabase SQL Editor
CREATE OR REPLACE FUNCTION verify_email(
    token_input TEXT,
    type_input TEXT
)
RETURNS JSON
LANGUAGE plpgsql
AS $$
DECLARE
    result JSON;
BEGIN
    -- Verificar si el token existe y es válido
    -- Tu lógica de verificación aquí
    
    -- Si es válido:
    UPDATE auth.users 
    SET email_confirmed_at = NOW()
    WHERE id = token_input;
    
    result := json_build_object('success', true);
    RETURN result;
    
EXCEPTION WHEN OTHERS THEN
    result := json_build_object('success', false, 'message', SQLERRM);
    RETURN result;
END;
$$;
```

### RLS Policy (Row Level Security)

Si necesitas proteger el endpoint:

```sql
CREATE POLICY "Allow verification requests"
ON public.verifications
FOR INSERT
WITH CHECK (true);
```

## Troubleshooting

### "Parámetros inválidos en la URL"
- Verificar que el token se está pasando correctamente desde el email
- Revisar que `?token=` y `&type=` estén presentes

### "Error en la verificación"
- Verificar credenciales de Supabase
- Comprobar que SUPABASE_URL y SUPABASE_ANON_KEY son correctos
- Revisar CORS en Supabase (Settings → API)

### Email no se envía
- Verificar que el email está confirmado en Supabase Auth
- Revisar template de email en Authentication → Email Templates
- Revisar logs de Supabase (Authentication → Logs)

## Security Best Practices

1. **Nunca exponer secretos**
   - La API key debe ser "anon" (pública)
   - No incluir service role key en frontend

2. **Validar tokens**
   - Implementar expiración de tokens
   - Usar tokens únicos y de un solo uso

3. **Rate limiting**
   - Limitar intentos de verificación
   - Implementar cooldown entre re-envíos

4. **HTTPS obligatorio**
   - Vercel automáticamente usa HTTPS
   - Configurar Strict-Transport-Security en vercel.json

## Ejemplo de Email Template (Supabase)

```html
<h2>Confirma tu email</h2>
<p>Hola {{ .Email }},</p>
<p>Gracias por registrarte en PRAGMA Studio. Por favor confirma tu email haciendo clic en el botón:</p>

<a href="https://tu-proyecto.vercel.app?token={{ .ConfirmationURL }}&type=signup" 
   style="display: inline-block; padding: 12px 24px; background: #0ea5e9; color: white; text-decoration: none; border-radius: 6px;">
  Confirmar email
</a>

<p style="margin-top: 20px; color: #999; font-size: 12px;">
  Este enlace expira en 24 horas. Si no pediste este registro, ignora este email.
</p>
```

## Support

Para problemas o preguntas:
- Documentación Supabase: https://supabase.com/docs
- Documentación Vercel: https://vercel.com/docs
- Issues: GitHub (si usas este repo)

---

**Hecho con ❤️ por PRAGMA Studio**
