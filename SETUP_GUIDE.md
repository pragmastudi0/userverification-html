# 🚀 Setup Completo: Email Verification con Supabase y Vercel

## Tabla de Contenidos
1. [Opción 1: HTML Puro (Más Simple)](#opción-1-html-puro)
2. [Opción 2: Next.js (Más Robusto)](#opción-2-nextjs)
3. [Configuración de Supabase](#configuración-de-supabase)
4. [Deploy en Vercel](#deploy-en-vercel)

---

## Opción 1: HTML Puro ✨

### Archivos necesarios:
```
proyecto/
├── verification.html
├── vercel.json
└── README.md
```

### Paso 1: Descargar archivos

Copiar los siguientes archivos en tu carpeta:
- `verification.html`
- `vercel.json`

### Paso 2: Configurar credenciales

Editar `verification.html` y reemplazar:

```javascript
const SUPABASE_URL = 'https://TU-PROYECTO.supabase.co';
const SUPABASE_ANON_KEY = 'TU-ANON-KEY';
```

**Dónde encontrar estas credenciales:**

1. Ir a [supabase.com](https://supabase.com)
2. Seleccionar tu proyecto
3. Settings → API → Copiar:
   - Project URL → SUPABASE_URL
   - anon/public key → SUPABASE_ANON_KEY

### Paso 3: Personalizar URLs de redirección

En `verification.html`, buscar:

```javascript
function redirectToApp() {
    // CAMBIAR ESTA URL a tu dashboard
    window.location.href = 'https://tu-app.vercel.app/dashboard';
}

function requestNewLink() {
    // URL donde el usuario puede pedir re-envío
    window.location.href = 'https://tu-app.vercel.app/resend-verification';
}

function goHome() {
    // URL de inicio
    window.location.href = 'https://pragmastudio.com.ar';
}
```

### Paso 4: Deploy en Vercel

#### Opción A: Desde CLI

```bash
# 1. Instalar Vercel CLI
npm install -g vercel

# 2. Desde la carpeta del proyecto
vercel

# 3. Seguir las instrucciones:
# - Set up and deploy "~/proyecto"? → YES
# - Which scope? → Tu cuenta
# - Link to existing project? → NO
# - Project name? → email-verification
# - Directory? → ./
```

#### Opción B: Desde GitHub

```bash
# 1. Crear repositorio
git init
git add .
git commit -m "Initial email verification setup"

# 2. Crear repo en GitHub y push
git branch -M main
git remote add origin https://github.com/tu-usuario/verification.git
git push -u origin main

# 3. En vercel.com:
# - Importar proyecto desde GitHub
# - Seleccionar repo
# - Deploy (automático)
```

#### Opción C: Drag & Drop

1. Ir a [vercel.com/new](https://vercel.com/new)
2. Hacer drag & drop de la carpeta
3. Vercel lo deploy automáticamente

### Paso 5: Obtener URL de Vercel

Después del deploy, Vercel proporciona una URL como:
```
https://email-verification-abc123.vercel.app
```

Esta es tu **VERIFICATION_URL** que usarás en Supabase.

---

## Opción 2: Next.js 🎯

Para mejor control, logging y manejo de errores.

### Paso 1: Crear proyecto Next.js

```bash
# Crear proyecto
npx create-next-app@latest verification-app
cd verification-app

# Instalar dependencias
npm install @supabase/auth-helpers-nextjs @supabase/supabase-js
```

### Paso 2: Variables de entorno

Crear `.env.local`:

```env
NEXT_PUBLIC_SUPABASE_URL=https://tu-proyecto.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=tu-anon-key
SUPABASE_SERVICE_KEY=tu-service-key
```

### Paso 3: Crear página de verificación

Crear `pages/verify.tsx`:

```typescript
import { useEffect, useState } from 'react';
import { useRouter } from 'next/router';
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs';

export default function VerifyPage() {
  const router = useRouter();
  const supabase = createClientComponentClient();
  const [state, setState] = useState('loading');

  useEffect(() => {
    if (!router.isReady) return;
    verifyEmail();
  }, [router.isReady]);

  async function verifyEmail() {
    try {
      const { token, type } = router.query;
      
      if (!token || !type) {
        throw new Error('Token inválido');
      }

      // Usar API serverless para mayor seguridad
      const res = await fetch(`/api/verify?token=${token}&type=${type}`);
      const data = await res.json();

      if (!res.ok) {
        throw new Error(data.error || 'Verification failed');
      }

      setState('success');
      
      // Redirect después de 3 segundos
      setTimeout(() => {
        router.push('/dashboard');
      }, 3000);

    } catch (error) {
      console.error('Error:', error);
      setState('error');
    }
  }

  // ... JSX con estados (ver archivo next-version.tsx)
}
```

### Paso 4: Crear API serverless

Crear `pages/api/verify.ts`:

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_KEY!
);

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { token, type } = req.query;

    if (!token || !type) {
      return res.status(400).json({ error: 'Missing parameters' });
    }

    // Verificar token con Supabase
    const { data, error } = await supabase
      .from('verification_tokens')
      .select('*')
      .eq('token', token)
      .eq('type', type)
      .single();

    if (error || !data) {
      return res.status(400).json({ error: 'Invalid token' });
    }

    // Verificar que no haya expirado
    const expiresAt = new Date(data.expires_at);
    if (expiresAt < new Date()) {
      return res.status(400).json({ error: 'Token expired' });
    }

    // Actualizar usuario como verificado
    await supabase.auth.admin.updateUserById(data.user_id, {
      email_confirm: true,
    });

    // Eliminar token usado
    await supabase
      .from('verification_tokens')
      .delete()
      .eq('id', data.id);

    return res.status(200).json({ success: true });

  } catch (error) {
    console.error('Error:', error);
    return res.status(500).json({ error: 'Internal server error' });
  }
}
```

### Paso 5: Deploy

```bash
# Verificar que funciona localmente
npm run dev

# Deploy en Vercel
vercel
```

---

## Configuración de Supabase 🔐

### Paso 1: Template de Email

En tu proyecto Supabase:

1. **Authentication → Email Templates**
2. Editar **Confirm signup**
3. Reemplazar el contenido:

```html
<h2>Confirma tu email</h2>
<p>Hola {{ .Email }},</p>
<p>Gracias por registrarte. Confirma tu email haciendo clic:</p>

<a href="https://EMAIL-VERIFICATION-ABC.vercel.app?token={{ .ConfirmationURL }}&type=signup" 
   style="display: inline-block; padding: 12px 24px; background: #0ea5e9; color: white; text-decoration: none; border-radius: 6px; font-weight: bold;">
  ✓ Confirmar email
</a>

<p style="margin-top: 20px; font-size: 12px; color: #999;">
  Este enlace expira en 24 horas.
</p>
```

### Paso 2: Habilitar Email Confirm

En **Authentication → Providers → Email**:
- ✅ Confirm email
- ✅ Double confirm change

### Paso 3: CORS (si usas API)

En **Settings → API → CORS Allowed Origins**, agregar:
```
https://tu-proyecto.vercel.app
https://localhost:3000
```

---

## Deploy en Vercel 🚀

### Método 1: GitHub (Recomendado)

```bash
# En tu repositorio local
git init
git add .
git commit -m "Email verification setup"
git remote add origin https://github.com/user/repo.git
git push -u origin main
```

1. Ir a [vercel.com](https://vercel.com)
2. Click en "New Project"
3. Seleccionar repository de GitHub
4. Click "Deploy"

### Método 2: Vercel CLI

```bash
npm install -g vercel
vercel
```

### Configurar variables de entorno en Vercel

1. En Vercel Dashboard → Settings → Environment Variables
2. Agregar:
   - `SUPABASE_URL`
   - `SUPABASE_ANON_KEY`
   - `SUPABASE_SERVICE_KEY` (si usas Next.js)

### Domain personalizado

1. En Vercel Dashboard → Settings → Domains
2. Agregar dominio personalizado
3. Configurar DNS según instrucciones

---

## Testing 🧪

### 1. Local Testing

```bash
# HTML Puro
# Abrir verification.html en el navegador
# O con Live Server

# Next.js
npm run dev
# Visitar http://localhost:3000/verify?token=test&type=signup
```

### 2. Enviar email real

```javascript
// En tu app, al registrar usuario:
const { data, error } = await supabase.auth.signUp({
  email: 'usuario@example.com',
  password: 'password123',
});

// Supabase enviará automáticamente el email con el link
```

### 3. Verificar en logs

En Supabase → Authentication → Logs verás:
```
Confirmation email sent to usuario@example.com
```

---

## Troubleshooting 🔧

| Problema | Solución |
|----------|----------|
| "Token inválido" | Verificar que el token se pasa correctamente desde el email |
| Email no se envía | Habilitar "Confirm email" en Auth settings |
| CORS error | Agregar URL de Vercel en Settings → API → CORS |
| Redirect no funciona | Verificar que las URLs en `redirectToApp()` son correctas |
| "undefined" en console | Asegurar que SUPABASE_URL y SUPABASE_ANON_KEY están correctos |

---

## Checklist Final ✅

- [ ] Credenciales de Supabase configuradas
- [ ] URLs de redirección personalizadas
- [ ] Template de email actualizado
- [ ] Deploy en Vercel realizado
- [ ] Domain personalizado (opcional)
- [ ] Variables de entorno en Vercel
- [ ] Email de confirmación probado
- [ ] Verificación completada exitosamente

---

## URLs Útiles 🔗

- Supabase Docs: https://supabase.com/docs
- Vercel Docs: https://vercel.com/docs
- Next.js Docs: https://nextjs.org/docs
- Supabase Auth Helpers: https://supabase.com/docs/guides/auth/auth-helpers

---

**¿Dudas?**
- Revisar logs en Supabase
- Abrir Developer Tools en navegador (F12)
- Verificar Network tab para ver requests a Supabase

¡Éxito con tu página de verificación! 🎉
