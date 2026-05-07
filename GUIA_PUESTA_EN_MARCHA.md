# 🚀 ABC Territorial Intelligence — Guía de Puesta en Marcha
### Para propietarios de la plataforma sin conocimientos de programación

---

## ¿Qué tienes en este paquete?

| Archivo/Carpeta | Qué hace |
|---|---|
| `supabase/01_schema.sql` | Crea todas las tablas de la base de datos |
| `supabase/02_postgis_motor.sql` | Instala el motor GIS inteligente |
| `app/` | El código de la plataforma web |
| `lib/` | Conexiones a Stripe, PayPal y Supabase |
| `.env.example` | Lista de claves que debes pegar |
| `next.config.ts` | Seguridad y rendimiento del servidor |

**Tiempo estimado para publicar: 2-3 horas siguiendo estos pasos.**

---

## PASO 1 — Crear tu base de datos en Supabase

Supabase es donde se guarda toda la información (usuarios, terrenos, pagos).

1. Ve a **https://supabase.com** y crea una cuenta gratuita
2. Haz clic en **"New project"**
3. Ponle un nombre: `abc-territorial` · Elige una contraseña segura · Región: `South America (São Paulo)` → es la más cercana a RD
4. Espera ~2 minutos a que el proyecto se cree
5. En el menú izquierdo haz clic en **"SQL Editor"**
6. Haz clic en **"New query"**
7. Abre el archivo `supabase/01_schema.sql` con el Bloc de notas, **selecciona todo** (Ctrl+A), cópialo (Ctrl+C) y pégalo en el editor de Supabase
8. Haz clic en **"Run"** (botón verde) → Debe decir "Success"
9. Repite exactamente igual con `supabase/02_postgis_motor.sql`
10. Ve a **Settings → API** y copia estos 3 valores (los necesitarás en el Paso 4):
    - **Project URL** → empieza con `https://`
    - **anon public** → empieza con `eyJ`
    - **service_role** → empieza con `eyJ` (¡este es secreto, nunca lo compartas!)

---

## PASO 2 — Configurar Stripe para cobros con tarjeta

1. Ve a **https://stripe.com** y crea una cuenta
2. Activa tu cuenta con tus datos bancarios (necesario para recibir pagos reales)
3. En el menú ve a **Productos** → **Crear producto**
4. Crea estos 5 productos exactamente así:

| Nombre del producto | Precio | Tipo |
|---|---|---|
| Propietario Profesional | US$199 | Pago único |
| Propietario Premium | US$950 | Pago único |
| Validado ABC | US$2,500 | Pago único |
| Inversionista Profesional | US$149 | Recurrente mensual |
| Deal Access | US$750 | Recurrente mensual |

5. Para cada producto, copia el **Price ID** (empieza con `price_`) → los necesitarás en el Paso 4
6. Ve a **Desarrolladores → Claves de API** y copia:
   - **Secret key** (empieza con `sk_live_`)
   - **Publishable key** (empieza con `pk_live_`)

> ⚠️ Para probar primero usa las claves `sk_test_` y `pk_test_` que Stripe da gratis

---

## PASO 3 — Configurar PayPal como respaldo

1. Ve a **https://developer.paypal.com**
2. Inicia sesión con tu cuenta PayPal de negocio
3. Ve a **"My Apps & Credentials"**
4. Haz clic en **"Create App"** → ponle nombre `abc-territorial`
5. Copia:
   - **Client ID**
   - **Secret** (haz clic en "Show")
6. Para producción cambia el modo a `live` en el archivo `.env`

---

## PASO 4 — Subir el código a GitHub

GitHub es donde se guarda el código para que Vercel lo publique.

1. Ve a **https://github.com** y crea una cuenta gratuita
2. Haz clic en **"New repository"** (botón verde)
3. Nombre: `abc-territorial` · Márcalo como **Private** · Haz clic en **"Create"**
4. Descarga **GitHub Desktop** en **https://desktop.github.com**
5. Instálalo y conéctalo a tu cuenta de GitHub
6. En GitHub Desktop: **File → Add local repository** → selecciona la carpeta de este paquete
7. Escribe un mensaje como "Primera versión" y haz clic en **"Commit to main"**
8. Haz clic en **"Publish repository"** → selecciona `abc-territorial` → **"Publish"**

---

## PASO 5 — Publicar en Vercel (tu dominio web)

Vercel es donde vive tu plataforma en internet, gratis para empezar.

1. Ve a **https://vercel.com** y crea una cuenta con tu GitHub
2. Haz clic en **"Add New → Project"**
3. Selecciona el repositorio `abc-territorial` → haz clic en **"Import"**
4. Antes de hacer Deploy, haz clic en **"Environment Variables"**
5. Agrega cada variable de la siguiente tabla (copia el nombre exacto y pega el valor):

```
NEXT_PUBLIC_SUPABASE_URL          = [tu Project URL de Supabase]
NEXT_PUBLIC_SUPABASE_ANON_KEY     = [tu anon public key de Supabase]
SUPABASE_SERVICE_ROLE_KEY         = [tu service_role key de Supabase]

STRIPE_SECRET_KEY                 = sk_live_... [o sk_test_... para pruebas]
STRIPE_WEBHOOK_SECRET             = whsec_... [lo obtienes en el Paso 6]
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY= pk_live_...

STRIPE_PRICE_OWNER_PRO            = price_... [Propietario Profesional]
STRIPE_PRICE_OWNER_PREMIUM        = price_... [Propietario Premium]
STRIPE_PRICE_OWNER_VALIDATED      = price_... [Validado ABC]
STRIPE_PRICE_INV_PRO              = price_... [Inversionista Profesional]
STRIPE_PRICE_INV_DEAL             = price_... [Deal Access]

PAYPAL_ENV                        = live [o sandbox para pruebas]
PAYPAL_CLIENT_ID                  = [tu Client ID de PayPal]
PAYPAL_CLIENT_SECRET              = [tu Secret de PayPal]

NEXT_PUBLIC_SITE_URL              = https://tudominio.com [o el que Vercel te asigne]
```

6. Haz clic en **"Deploy"** → espera ~3 minutos
7. Vercel te dará una URL como `https://abc-territorial-xxxx.vercel.app` → esta es tu plataforma

---

## PASO 6 — Configurar el Webhook de Stripe

El webhook le avisa a tu plataforma cuando alguien paga.

1. En Stripe ve a **Desarrolladores → Webhooks → Add endpoint**
2. En **Endpoint URL** escribe: `https://TU-DOMINIO.vercel.app/api/stripe/webhook`
3. En **Events** selecciona:
   - `checkout.session.completed`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_failed`
4. Haz clic en **"Add endpoint"**
5. Copia el **Signing secret** (empieza con `whsec_`)
6. Ve a Vercel → **Settings → Environment Variables**
7. Agrega: `STRIPE_WEBHOOK_SECRET` = `whsec_...`
8. Haz clic en **"Redeploy"** en Vercel → la última versión

---

## PASO 7 — Probar que todo funciona

1. Abre tu URL de Vercel en el navegador
2. Debes ver la landing page con los planes
3. Haz clic en **"Pagar con tarjeta"** en cualquier plan
4. En Stripe, usa la tarjeta de prueba: `4242 4242 4242 4242` · Fecha: cualquier futura · CVC: `123`
5. Después del pago debes llegar a `/payment/success`
6. En Supabase → **Table Editor → payments** debe aparecer un registro nuevo
7. Prueba PayPal con tu cuenta sandbox de developer.paypal.com

---

## PASO 8 — Cargar las capas GIS reales

El motor de análisis necesita capas geográficas reales para funcionar.

1. En Supabase → **SQL Editor** → ejecuta este ejemplo para agregar una capa:

```sql
-- Agrega una capa de zonas protegidas
INSERT INTO public.gis_layers (name, layer_type, source_name, is_active, is_critical)
VALUES ('Áreas Protegidas RD', 'area_protegida', 'MIMARENA', true, true);
```

2. Para cargar geometrías reales necesitarás un archivo GeoJSON de la zona.
   ABC Projects puede asistirte en este paso con los datos del territorio dominicano.

3. Las capas recomendadas para cargar son:
   - Áreas protegidas (bloquean publicación)
   - Zonas inundables (alerta alta)
   - Riesgos ambientales (alerta alta)
   - Vías principales (mejora score de accesibilidad)
   - Zonas comerciales (mejora score de entorno)
   - Polos turísticos (mejora score de mercado)

---

## PASO 9 — Configurar el motor de análisis (sin código)

El motor usa parámetros editables. El administrador puede cambiarlos desde la API.

Para ajustar los pesos del score, envía una petición PUT a:
```
https://tudominio.vercel.app/api/admin/motor
```

Con estos valores (la suma de los pesos debe ser exactamente 100):
```json
{
  "peso_accesibilidad": 25,
  "peso_entorno": 25,
  "peso_mercado": 25,
  "peso_normativa": 15,
  "peso_riesgo": 10,
  "precio_m2_dn": 800,
  "precio_m2_altagracia": 600
}
```

> Puedes usar la app gratuita **Postman** (https://postman.com) para enviar estos cambios sin código.

---

## PASO 10 — Conectar tu dominio propio (opcional pero recomendado)

1. Compra un dominio en **Namecheap** o **GoDaddy**: ej. `abcterritorial.com`
2. En Vercel → **Settings → Domains** → **Add Domain**
3. Escribe tu dominio y sigue las instrucciones para apuntar el DNS
4. Actualiza `NEXT_PUBLIC_SITE_URL` en Vercel con tu dominio nuevo
5. Actualiza la URL del webhook en Stripe con el nuevo dominio
6. Haz Redeploy

---

## Modelo de negocio operativo

### Cómo ganas dinero con la plataforma

| Fuente | Cómo funciona | Cuándo cobras |
|---|---|---|
| **Suscripciones Propietarios** | Stripe cobra automáticamente US$199, $950 o $2,500 | Al momento del pago |
| **Suscripciones Inversionistas** | Stripe cobra $149 o $750 cada mes | Mensualmente, automático |
| **Fee de estructuración (3%)** | Lo registras manualmente en `commissions` cuando estructuras un proyecto | Al firmar el encargo |
| **Comisión conexión (2%)** | Se calcula automáticamente cuando apruebas una solicitud de inversión | Al aprobar la conexión |
| **Comisión por cierre (5%)** | Lo registras manualmente cuando se cierra la operación | Al cerrar la negociación |

### Seguimiento de comisiones
Todas las comisiones quedan registradas en la tabla `commissions` de Supabase.
Puedes verlas en: Supabase → Table Editor → commissions

---

## Advertencia legal obligatoria

Asegúrate de incluir este texto en tu plataforma:

> *"El análisis normativo ABC es orientativo y no sustituye permisos oficiales,
> certificaciones municipales, MIVED, Medio Ambiente u otras autoridades
> competentes de la República Dominicana."*

---

## Contacto técnico de emergencia

Si algo no funciona, los errores más comunes y sus soluciones:

| Error | Causa probable | Solución |
|---|---|---|
| "Plan no configurado" | Falta un Price ID de Stripe | Revisa las variables de entorno en Vercel |
| "Supabase admin env vars missing" | Faltan claves de Supabase | Revisa `NEXT_PUBLIC_SUPABASE_URL` y `SUPABASE_SERVICE_ROLE_KEY` |
| Webhook no llega | URL incorrecta en Stripe | Verifica que la URL del webhook sea exactamente `/api/stripe/webhook` |
| PayPal no redirige | Credenciales de sandbox en producción | Cambia `PAYPAL_ENV=live` |
| Error en análisis GIS | No hay capas GIS cargadas | Ejecuta el SQL del Paso 8 |

---

*ABC Territorial Intelligence Platform v3.0 — Producción*
*Arquitectura: Next.js 15 + Supabase + PostGIS + Stripe + PayPal + Vercel*
