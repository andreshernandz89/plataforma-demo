# Instalación de una instancia de la plataforma

Guía para montar la plataforma en un colegio nuevo. Tiempo estimado: 45–60 minutos.
Cada colegio tiene su **propia** instancia: su proyecto de Firebase, sus archivos y su enlace.
Nunca se mezclan datos de dos colegios.

## 1. Crear el proyecto de Firebase del colegio

1. En [console.firebase.google.com](https://console.firebase.google.com) → **Crear un proyecto** (sin Google Analytics, plan gratuito Spark, **sin** activar facturación).
2. **Agregar aplicación web** (`</>`): copiar el objeto `firebaseConfig` que muestra la consola.
3. **Build → Realtime Database → Crear base de datos** (modo bloqueado). En la pestaña **Reglas**, pegar el JSON de la sección 5 de esta guía y **Publicar**.
4. **Build → Authentication → Comenzar** → pestaña *Sign-in method* → activar **Correo/contraseña**.
5. En **Authentication → Users → Agregar usuario**: correo `admin@plataforma.local` y la contraseña temporal del dueño del colegio.

## 2. Reemplazar los tokens de la plantilla

Buscar y reemplazar en **todos** los archivos `.html` de la plantilla:

| Token | Valor |
|---|---|
| `__FB_APIKEY__` | `apiKey` del firebaseConfig |
| `__FB_AUTHDOMAIN__` | `authDomain` |
| `__FB_DBURL__` | `databaseURL` completa (https://…firebaseio.com) |
| `__FB_HOST__` | Solo el host de la base (…-default-rtdb.firebaseio.com) |
| `__FB_PROJECTID__` | `projectId` |
| `__FB_BUCKET__` | `storageBucket` |
| `__FB_SENDERID__` | `messagingSenderId` |
| `__FB_APPID__` | `appId` |
| `__NOMBRE_COLEGIO__` | Nombre oficial del colegio (ej.: COLEGIO SAN MARTÍN) |
| `__NIT_COLEGIO__` | NIT que sale en el certificado de convivencia |
| `__TELEFONO_COLEGIO__` | Teléfono/WhatsApp público del colegio |
| `__CORREO_COLEGIO__` | Correo institucional |
| `__DIRECCION_COLEGIO__` | Dirección del colegio |
| `__LICENCIA_TEXTO__` | Texto de la licencia de funcionamiento |
| `__URL_VERIFICAR__` | URL pública de la página de verificación (ej.: https://colegio.com/verificar) |
| `__URL_VERIFICAR_TEXTO__` | La misma URL sin `https://` |
| `__URL_MATRICULA__` | URL pública del formulario de matrícula |
| `__ALIAS_FORMSUBMIT__` | Alias de FormSubmit del correo del colegio (aviso de solicitudes de certificados; opcional) |
| `__PUENTE_URL__` | URL del puente de Apps Script si el colegio usará Alegra/Wallet (opcional; va en `PlataformaEDU/index.html`, constante `PUENTE_URL`) |

## 3. Marca del colegio

Reemplazar en la carpeta de la instancia:

- `escudo.png` y `favicon-48.png` — escudo real del colegio.
- `PlataformaEDU/membrete.png` — hoja membreteada oficial (vertical, fondo blanco).
- `PlataformaEDU/firma-secretaria.png` — firma de quien firma las constancias (fondo transparente).
- `documentos/` — `manual-convivencia.pdf`, la planeación en PDF y `qr-breb.jpg` (QR de pago del colegio), cuando aplique.

La carpeta trae versiones de ejemplo generadas por script; se pueden usar mientras el colegio entrega las suyas.

## 4. Publicar

Subir toda la carpeta de la instancia a un repositorio de GitHub y activar **GitHub Pages**
(o cualquier hosting estático). La estructura debe quedar:

```
/PlataformaEDU/   (la plataforma, ingreso con usuario y contraseña)
/padres/          (consulta pública para padres)
/matricula2027/   (formulario público de matrícula)
/verificar/       (verificación pública de certificados)
/documentos/      (PDF y QR públicos)
```

## 5. Reglas de la base de datos (pegar tal cual en el paso 1.3)

```json
{
  "rules": {
    "eduDB": { ".read": "auth != null", ".write": "auth != null" },
    "owners": { ".read": "auth != null", ".write": "root.child('owners').child(auth.uid).exists()" },
    "uidPorPerfil": { ".read": "auth != null", ".write": "auth != null" },
    "usuarioPorUid": { "$uid": { ".read": "auth != null", ".write": "auth.uid === $uid || root.child('owners').child(auth.uid).exists()" } },
    "loginMap": { "$usuario": { ".read": true, ".write": "root.child('owners').child(auth.uid).exists() || root.child('usuarioPorUid').child(auth.uid).val() === $usuario" } },
    "clavesVisibles": { ".read": "root.child('owners').child(auth.uid).exists()", "$pid": { ".read": "root.child('owners').child(auth.uid).exists() || root.child('uidPorPerfil').child($pid).val() === auth.uid", ".write": "root.child('owners').child(auth.uid).exists() || root.child('uidPorPerfil').child($pid).val() === auth.uid" } },
    "pendientes": { "$uid": { ".read": "auth.uid === $uid || root.child('owners').child(auth.uid).exists()", ".write": "auth.uid === $uid || root.child('owners').child(auth.uid).exists()" } },
    "matriculas2027": { ".read": "auth != null", "$id": { ".write": "auth != null || (!data.exists() && newData.exists())", ".validate": "newData.hasChildren(['ts', 'estudiante', 'acudiente'])" } },
    "solicitudesCert": { ".read": "auth != null", "$id": { ".write": "auth != null || (!data.exists() && newData.exists())", ".validate": "newData.hasChildren(['ts', 'tipo', 'estudiante', 'clave'])" } },
    "consulta2027": { ".write": "auth != null", "$clave": { ".read": true, ".write": "auth != null" } },
    "archivos": { ".read": "auth != null", ".write": "auth != null" }
  }
}
```

## 6. Primer ingreso y puesta en marcha

1. Entrar a `/PlataformaEDU` con usuario **admin** y la contraseña temporal del paso 1.5.
2. Completar **Mi perfil** (la plataforma lo exige en el primer ingreso) y poner el **correo real**:
   desde ese momento ese correo es el de ingreso y recuperación.
3. En **Instituto**: nombre, ciudad, año, licencia, sede y la **llave Bre-B/Nequi** para certificados pagos.
4. En **Control de matrículas y pensiones**: fijar las **tarifas** de cada grado.
5. Crear los perfiles del personal (Perfiles) y cargar los estudiantes (Cursos y estudiantes, o desde el buzón de Matrículas 2027).
6. Capacitación de 1 hora con el personal y entrega del enlace de la consulta a los padres.

## Notas

- El nodo de matrículas se llama `matriculas2027` por dentro; es solo un nombre interno.
- Las integraciones con Alegra (facturación) y Wallet (transacciones) son **opcionales** y se configuran por colegio desde Instituto → Integraciones y APIs (solo el dueño).
- Antes de entregar el acceso al colegio, debe estar firmado el contrato de suscripción con la cláusula de tratamiento de datos personales de menores (Ley 1581 de 2012).
