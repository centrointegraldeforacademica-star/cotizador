# Arboleda Fest: cotizador con base de datos y panel de reservas

Qué incluye este paquete:

| Archivo | Para qué sirve |
|---|---|
| `index.html` | La página que ven tus clientes (el cotizador). |
| `admin.html` | Tu panel privado: cotizaciones, calendario de reservas e inventario. |
| `config.js` | Aquí pegas los datos de tu proyecto de Supabase. |
| `supabase/setup.sql` | Crea la base de datos con sus reglas de seguridad. |
| `logo/` | Tu logo en vector y PNG. |

Si dejas `config.js` vacío, el cotizador sigue funcionando como antes (solo envía por WhatsApp, sin guardar nada).

---

## Parte 1. Crear la base de datos en Supabase (unos 15 minutos)

Los nombres de los menús de Supabase pueden cambiar un poco con el tiempo; si algo no aparece donde dice, búscalo por su nombre.

1. Entra a **supabase.com**, crea una cuenta y toca **New project**. Ponle de nombre `arboleda-fest`, elige una región cercana (por ejemplo East US) y anota la contraseña de la base de datos en un lugar seguro.
2. En el menú izquierdo abre **SQL Editor**, toca **New query**, pega TODO el contenido de `supabase/setup.sql` y toca **Run**. Debe terminar sin errores. Es seguro ejecutarlo dos veces.
3. Crea tu usuario: **Authentication, Users, Add user, Create new user**. Escribe tu correo y una contraseña larga y marca la opción de confirmar el usuario automáticamente.
4. Busca en Authentication la opción **Allow new users to sign up** y desactívala. Así nadie más puede crear cuentas. (Aunque alguien lo lograra, las reglas de seguridad no le darían acceso a nada.)
5. Hazte administradora. En **SQL Editor** ejecuta esto, cambiando el correo por el tuyo:

   ```sql
   insert into public.admins (user_id)
   select id from auth.users where email = 'tu-correo@ejemplo.com';
   ```
6. Copia dos datos del proyecto: la **Project URL** (algo como `https://abcdxyz.supabase.co`) y la **clave pública** (en la sección de API Keys, la "Publishable key", que empieza con `sb_publishable_`; si no existe, toca "Create new API keys"). Abre `config.js` y pégalas ahí.

> La clave `sb_publishable_` es pública por diseño: puede estar en GitHub. **Nunca** pegues en el proyecto una clave `secret` ni `service_role`.

## Parte 2. Publicar la página en GitHub Pages (gratis)

1. En GitHub crea un repositorio nuevo (por ejemplo `cotizador`).
2. Sube TODOS los archivos de esta carpeta, incluidos `.nojekyll`, `supabase/` y `logo/`.
3. En el repositorio: **Settings, Pages, Deploy from a branch**, rama `main`, carpeta `/ (root)`, **Save**.
4. En unos minutos tendrás: `https://TU-USUARIO.github.io/cotizador/` (clientes) y `https://TU-USUARIO.github.io/cotizador/admin.html` (tu panel).

Con un repositorio público, GitHub Pages es gratis. Con uno privado, depende de tu plan de GitHub.

## Parte 3. Primera prueba

1. Abre el panel (`admin.html`), entra con tu correo y contraseña y ve a **Inventario**. Las unidades que trae son **de ejemplo**: pon las que realmente tienes y toca Guardar en cada pieza.
2. Abre el cotizador, arma una cotización de prueba, elige una fecha y toca el botón de WhatsApp. Debe abrir tu chat con el mensaje, y en el panel debe aparecer la cotización con el mismo código.
3. En el panel toca **Fecha libre, confirmar**. Esa cotización ya bloquea sus piezas: en el cotizador, otra persona que elija esa fecha (o un día antes o después) verá un aviso de que hay piezas reservadas.

---

## Cómo funciona

- Cuando el cliente toca el botón, la cotización se **guarda primero** con su código (por ejemplo `AF-261018-X7K`) y **después** se abre WhatsApp con ese mismo código. Si la conexión falla, abre WhatsApp igual y el mensaje dice "(Cotización no registrada en línea)".
- Una cotización cuenta como real cuando el código que llega a tu WhatsApp coincide con uno guardado en el panel.
- Las cotizaciones nuevas **no** bloquean piezas. Solo bloquean las que tú pasas a **Fecha confirmada**, **Abono recibido** o **Entregada**.
- Cada reserva ocupa sus piezas desde 1 día antes hasta 1 día después del evento (cámbialo en Inventario, "Entrega y devolución").
- Estados: Nueva, Fecha confirmada, Abono recibido, Entregada, Devuelta y Cancelada. Una cotización cancelada libera las piezas.

## Seguridad: lo que sí y lo que no

- Los visitantes solo pueden hacer dos cosas: enviar una cotización y preguntar si una fecha está disponible. No pueden leer cotizaciones, nombres ni usuarios de nadie.
- Hay límites contra abusos (máximo de cotizaciones por hora por conexión y por usuario de Instagram) y un campo trampa para robots.
- El total de la cotización lo calcula el navegador del cliente y la base de datos solo lo guarda. Es informativo: el precio real lo confirmas tú por WhatsApp.
- Esta versión no verifica pagos ni que la cuenta de Instagram escrita sea real.
- Guardas nombre, usuario de Instagram y datos de la fiesta de tus clientes. Conviene tener un aviso de privacidad y revisar con alguien que conozca la norma lo que aplique en Panamá (por ejemplo, la Ley 81 de 2019 de protección de datos personales).
- Usa una contraseña larga y exclusiva para el panel. El panel pide inicio de sesión y no aparece en buscadores, pero igual no compartas su dirección.

## Cambios frecuentes

- **Precios, combos y descuentos:** abre `index.html`, busca el bloque `CONFIGURACIÓN` y edita `ITEMS`, `COMBOS` y `CONFIG.tiers`.
- **Una pieza nueva:** agrégala en `ITEMS` de `index.html` Y como fila nueva en la tabla `inventario` (el `item_id` debe ser idéntico), por ejemplo:
  `insert into public.inventario (item_id, nombre, unidades) values ('arco-nuevo', 'Arco nuevo', 2);`
- **Número de WhatsApp:** en `config.js`.
- **Etiquetas para anuncios:** agrega al enlace `?utm_source=instagram&utm_medium=anuncio&utm_campaign=combos`. Cada cotización guarda de dónde vino.

## Si algo falla

- **El cotizador no muestra el aviso de disponibilidad ni guarda:** revisa que `config.js` tenga la URL y la clave bien pegadas, sin espacios.
- **El panel dice "no es administrador":** falta el paso 5 de la Parte 1 (o el correo no coincide).
- **"Permission denied" o "relation does not exist":** el `setup.sql` no se ejecutó completo; vuelve a correrlo.
- **Un cliente recibe "demasiadas solicitudes":** es el límite por hora; puedes subirlo en la tabla `config` (`max_cotizaciones_hora`).
