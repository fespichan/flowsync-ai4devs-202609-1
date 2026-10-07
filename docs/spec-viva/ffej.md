## Purpose

Cuentas y acceso permite que una persona cree su cuenta en FlowSync con email y contraseña, inicie y cierre sesión, y consulte su perfil, tanto a través de la API como desde la aplicación web. Es la puerta de entrada que identifica a cada persona antes de que pueda trabajar en el resto del producto.

## Requirements

### Requirement: Registro de cuenta en la API

El sistema SHALL permitir crear una cuenta nueva mediante `POST /api/v1/auth/signup` con `fullName`, `email`, `password` y `passwordConfirmation`, y SHALL responder con los datos públicos de la cuenta creada y un token de acceso ya válido, de modo que el registro deja la sesión iniciada.

#### Scenario: Registro correcto

- **WHEN** se envía `POST /api/v1/auth/signup` con un `email` válido que no está registrado, una `password` de entre 8 y 32 caracteres, una `passwordConfirmation` idéntica y un `fullName` de texto
- **THEN** la respuesta es `200` con un cuerpo `{ "data": { "user": { ... }, "token": "..." } }`, donde `user` contiene `id`, `fullName`, `email`, `createdAt`, `updatedAt` e `initials`, y `token` es una cadena opaca que empieza por `oat_`

#### Scenario: Registro sin nombre

- **WHEN** se envía `POST /api/v1/auth/signup` con datos válidos y `fullName` igual a `null` o a la cadena vacía
- **THEN** la cuenta se crea igualmente y la respuesta devuelve `user.fullName` como `null`

#### Scenario: La contraseña nunca se devuelve

- **WHEN** se crea una cuenta correctamente
- **THEN** ni la contraseña ni ninguna derivación de ella aparecen en la respuesta

### Requirement: Validación del registro en la API

El sistema SHALL rechazar con `422` cualquier registro cuyos datos no cumplan las reglas, sin crear la cuenta, y SHALL devolver un cuerpo `{ "errors": [ ... ] }` donde cada error indica `field`, `rule` y `message` (y `meta` cuando la regla tiene límites).

#### Scenario: Email ya registrado

- **WHEN** se envía `POST /api/v1/auth/signup` con un `email` que ya pertenece a otra cuenta
- **THEN** la respuesta es `422` con un error sobre `email` con regla `database.unique` y no se crea una segunda cuenta

#### Scenario: Email con formato inválido

- **WHEN** se envía `POST /api/v1/auth/signup` con un `email` que no tiene formato de dirección de correo
- **THEN** la respuesta es `422` con un error sobre `email` con regla `email`

#### Scenario: Email demasiado largo

- **WHEN** se envía `POST /api/v1/auth/signup` con un `email` de más de 254 caracteres
- **THEN** la respuesta es `422` con un error sobre `email` con regla `maxLength`

#### Scenario: Contraseña fuera de longitud

- **WHEN** se envía `POST /api/v1/auth/signup` con una `password` de menos de 8 caracteres o de más de 32
- **THEN** la respuesta es `422` con un error sobre `password` con regla `minLength` o `maxLength`, e indicando en `meta` el límite incumplido

#### Scenario: Confirmación distinta de la contraseña

- **WHEN** se envía `POST /api/v1/auth/signup` con una `passwordConfirmation` que no coincide con `password`
- **THEN** la respuesta es `422` con un error sobre `passwordConfirmation` con regla `sameAs`

#### Scenario: Faltan campos obligatorios

- **WHEN** se envía `POST /api/v1/auth/signup` sin `email`, sin `password`, sin `passwordConfirmation` o sin la clave `fullName`
- **THEN** la respuesta es `422` con un error de regla `required` sobre cada campo ausente

### Requirement: Inicio de sesión en la API

El sistema SHALL permitir iniciar sesión mediante `POST /api/v1/auth/login` con `email` y `password`, y SHALL emitir un token de acceso nuevo en cada inicio de sesión correcto.

#### Scenario: Credenciales correctas

- **WHEN** se envía `POST /api/v1/auth/login` con el `email` y la `password` de una cuenta existente
- **THEN** la respuesta es `200` con `{ "data": { "user": { ... }, "token": "oat_..." } }`, con los mismos campos de `user` que en el registro

#### Scenario: Varios inicios de sesión conviven

- **WHEN** la misma cuenta inicia sesión dos veces
- **THEN** se obtienen dos tokens distintos y ambos siguen siendo válidos a la vez

#### Scenario: Credenciales incorrectas

- **WHEN** se envía `POST /api/v1/auth/login` con un `email` que no existe o con una `password` que no corresponde a ese `email`
- **THEN** la respuesta es `400` con `{ "errors": [ { "message": "Invalid user credentials" } ] }`, idéntica en ambos casos, de modo que no revela si el email está registrado

#### Scenario: Datos de inicio de sesión mal formados

- **WHEN** se envía `POST /api/v1/auth/login` sin `email`, con un `email` de formato inválido o sin `password`
- **THEN** la respuesta es `422` con los errores de validación por campo y no se emite ningún token

### Requirement: Acceso autenticado con token

El sistema SHALL exigir un token de acceso válido, enviado como `Authorization: Bearer <token>`, en todas las operaciones bajo `/api/v1/account`, y SHALL rechazar con `401` cualquier petición sin token o con un token desconocido o revocado.

#### Scenario: Petición sin token

- **WHEN** se llama a `GET /api/v1/account/profile` o a `POST /api/v1/account/logout` sin cabecera `Authorization`
- **THEN** la respuesta es `401` con `{ "errors": [ { "message": "Unauthorized access" } ] }`

#### Scenario: Token inválido o revocado

- **WHEN** se llama a una operación de `/api/v1/account` con un token inventado o con uno que ya se usó para cerrar sesión
- **THEN** la respuesta es `401` con el mismo cuerpo de error

#### Scenario: El token no caduca por tiempo

- **WHEN** se usa un token emitido hace tiempo y que no se ha revocado con un cierre de sesión
- **THEN** el sistema lo sigue aceptando

### Requirement: Consulta del perfil en la API

El sistema SHALL devolver, mediante `GET /api/v1/account/profile`, los datos públicos de la cuenta a la que pertenece el token presentado.

#### Scenario: Perfil de la persona autenticada

- **WHEN** se llama a `GET /api/v1/account/profile` con un token válido
- **THEN** la respuesta es `200` con `{ "data": { "id", "fullName", "email", "createdAt", "updatedAt", "initials" } }` de la cuenta dueña del token, y sin la contraseña

#### Scenario: Iniciales a partir del nombre

- **WHEN** la cuenta tiene un `fullName` con al menos dos palabras separadas por espacio, por ejemplo "Ada Lovelace"
- **THEN** `initials` son la primera letra de las dos primeras palabras en mayúsculas ("AL")

#### Scenario: Iniciales con nombre de una sola palabra

- **WHEN** la cuenta tiene un `fullName` de una sola palabra, por ejemplo "Ada"
- **THEN** `initials` son las dos primeras letras de esa palabra en mayúsculas ("AD")

#### Scenario: Iniciales sin nombre

- **WHEN** la cuenta no tiene `fullName`, por ejemplo con email "ada@example.com"
- **THEN** `initials` son la primera letra de la parte anterior a la arroba y la primera de la parte posterior, en mayúsculas ("AE")

### Requirement: Cierre de sesión en la API

El sistema SHALL revocar, mediante `POST /api/v1/account/logout`, exactamente el token con el que se hace la petición, sin afectar a otros tokens de la misma cuenta.

#### Scenario: Cierre de sesión correcto

- **WHEN** se llama a `POST /api/v1/account/logout` con un token válido
- **THEN** la respuesta es `200` con `{ "message": "Logged out successfully" }` y, a partir de ese momento, ese token recibe `401` en cualquier operación de `/api/v1/account`

#### Scenario: Otras sesiones siguen abiertas

- **WHEN** una cuenta tiene dos tokens activos y cierra sesión con uno de ellos
- **THEN** el otro token sigue siendo aceptado

### Requirement: Formato común de las respuestas de la API

El sistema SHALL responder siempre en JSON en las operaciones de cuentas y acceso, envolviendo los datos correctos en una clave `data` (salvo el cierre de sesión, que devuelve `message`) y los fallos en una lista `errors`.

#### Scenario: Respuesta JSON aunque el cliente pida otra cosa

- **WHEN** se hace cualquier petición de cuentas y acceso con una cabecera `Accept` distinta de JSON, o sin ella
- **THEN** la respuesta, correcta o de error, llega en JSON

### Requirement: Pantalla de registro

La aplicación web SHALL ofrecer en `/register` un formulario de registro con los campos "Nombre completo (opcional)", "Email", "Contraseña" y "Repite la contraseña", y SHALL dejar a la persona con la sesión iniciada y en su perfil cuando el registro tiene éxito.

#### Scenario: Registro correcto desde la pantalla

- **WHEN** una persona sin sesión rellena el formulario con datos válidos y pulsa "Crear cuenta"
- **THEN** el botón pasa a "Creando cuenta…" y queda deshabilitado mientras se envía, y al terminar la persona ve su perfil con la sesión iniciada

#### Scenario: Indicación de longitud de la contraseña

- **WHEN** una persona ve el formulario de registro sin errores en la contraseña
- **THEN** bajo el campo "Contraseña" se muestra la indicación "Entre 8 y 32 caracteres."

#### Scenario: Contraseñas que no coinciden

- **WHEN** una persona escribe en "Repite la contraseña" un valor distinto del de "Contraseña" y pulsa "Crear cuenta"
- **THEN** bajo "Repite la contraseña" aparece "Las contraseñas no coinciden." y no se envía nada al servidor

#### Scenario: Email ya registrado

- **WHEN** una persona intenta registrarse con un email que ya tiene cuenta
- **THEN** bajo el campo "Email" aparece "Ese email ya está registrado. Inicia sesión en su lugar."

#### Scenario: Errores de validación por campo

- **WHEN** el servidor rechaza el registro por un email con formato inválido, un campo vacío o una contraseña fuera de longitud
- **THEN** cada mensaje aparece en castellano bajo el campo afectado (por ejemplo "Introduce una dirección de email válida.", "Falta rellenar el email." o un aviso del mínimo o máximo de caracteres) y el formulario conserva lo escrito

#### Scenario: Enlace a inicio de sesión

- **WHEN** una persona sin sesión está en la pantalla de registro
- **THEN** ve "¿Ya tienes cuenta? Inicia sesión" y al pulsar el enlace llega a la pantalla de inicio de sesión

### Requirement: Pantalla de inicio de sesión

La aplicación web SHALL ofrecer en `/login` un formulario con "Email" y "Contraseña", y SHALL llevar a la persona a su perfil cuando las credenciales son correctas.

#### Scenario: Inicio de sesión correcto desde la pantalla

- **WHEN** una persona sin sesión introduce un email y una contraseña correctos y pulsa "Entrar"
- **THEN** el botón pasa a "Entrando…" y queda deshabilitado mientras se envía, y al terminar la persona ve su perfil

#### Scenario: Credenciales incorrectas

- **WHEN** una persona introduce un email no registrado o una contraseña equivocada y pulsa "Entrar"
- **THEN** arriba del formulario aparece un aviso destacado con "El email o la contraseña no son correctos." y sigue en la pantalla de inicio de sesión

#### Scenario: Campos vacíos o email mal escrito

- **WHEN** una persona pulsa "Entrar" con el email vacío o con formato inválido, o con la contraseña vacía
- **THEN** el mensaje correspondiente en castellano aparece bajo el campo afectado

#### Scenario: Enlace a registro

- **WHEN** una persona sin sesión está en la pantalla de inicio de sesión
- **THEN** ve "¿Aún no tienes cuenta? Crea una" y al pulsar el enlace llega a la pantalla de registro

### Requirement: Errores de conexión y del servidor en pantalla

La aplicación web SHALL explicar en castellano, sin dejar a la persona sin respuesta, cualquier fallo que no sea de validación al registrarse o iniciar sesión.

#### Scenario: Servidor inaccesible

- **WHEN** una persona envía el formulario de registro o de inicio de sesión y el servidor no responde
- **THEN** aparece arriba del formulario "No se pudo conectar con el servidor. Comprueba que el backend está arrancado." y el botón vuelve a estar disponible

#### Scenario: Error inesperado del servidor

- **WHEN** una persona envía el formulario y el servidor falla de forma inesperada
- **THEN** aparece arriba del formulario "Algo ha ido mal en el servidor. Inténtalo de nuevo en un momento." y el botón vuelve a estar disponible

### Requirement: Persistencia de la sesión en el navegador

La aplicación web SHALL recordar la sesión iniciada entre recargas y visitas en el mismo navegador, y SHALL confirmar con el servidor que sigue siendo válida antes de mostrar contenido protegido.

#### Scenario: Recarga con sesión válida

- **WHEN** una persona con la sesión iniciada recarga la página o vuelve más tarde en el mismo navegador
- **THEN** ve primero un indicador de carga a pantalla completa y después su perfil, sin tener que volver a iniciar sesión

#### Scenario: Sesión rechazada por el servidor

- **WHEN** la aplicación intenta recuperar una sesión guardada y el servidor ya no la reconoce
- **THEN** la persona llega a la pantalla de inicio de sesión con el aviso "Tu sesión ha caducado. Vuelve a iniciar sesión." y la sesión guardada se descarta

#### Scenario: Servidor caído al recuperar la sesión

- **WHEN** la aplicación intenta recuperar una sesión guardada y el servidor no responde o falla
- **THEN** la persona llega a la pantalla de inicio de sesión con un aviso que explica el problema, y la sesión guardada se conserva para que al recargar con el servidor disponible vuelva a entrar sin credenciales

#### Scenario: El aviso desaparece al entrar

- **WHEN** una persona que ve un aviso de sesión perdida inicia sesión con éxito
- **THEN** el aviso deja de mostrarse

### Requirement: Control de acceso a las pantallas

La aplicación web SHALL mostrar el perfil solo a personas con sesión iniciada y SHALL reservar las pantallas de registro e inicio de sesión para personas sin sesión.

#### Scenario: Perfil sin sesión

- **WHEN** una persona sin sesión abre `/profile`
- **THEN** es llevada a la pantalla de inicio de sesión

#### Scenario: Registro o inicio de sesión con sesión iniciada

- **WHEN** una persona con la sesión iniciada abre `/login` o `/register`
- **THEN** es llevada a su perfil

#### Scenario: Dirección desconocida

- **WHEN** una persona abre cualquier dirección de la aplicación que no es `/login`, `/register` ni `/profile`
- **THEN** es llevada a `/profile`, y desde ahí a la pantalla de inicio de sesión si no tiene sesión

#### Scenario: Navegación hacia atrás tras redirección

- **WHEN** una persona ha sido redirigida por no tener o por tener sesión y pulsa "atrás" en el navegador
- **THEN** no vuelve a la dirección de la que fue redirigida

### Requirement: Pantalla de perfil

La aplicación web SHALL mostrar en `/profile` los datos de la cuenta con la sesión iniciada.

#### Scenario: Perfil con nombre

- **WHEN** una persona con nombre completo abre su perfil
- **THEN** ve un círculo con sus iniciales, su nombre, su email y la fila "Miembro desde" con la fecha de alta en formato largo en castellano (por ejemplo "6 de octubre de 2026")

#### Scenario: Perfil sin nombre

- **WHEN** una persona que se registró sin nombre abre su perfil
- **THEN** en lugar del nombre ve "Sin nombre", y el círculo muestra las iniciales derivadas de su email

### Requirement: Cierre de sesión desde la pantalla

La aplicación web SHALL permitir cerrar la sesión desde el perfil, y SHALL dejar a la persona fuera de su cuenta en ese navegador aunque el servidor no confirme el cierre.

#### Scenario: Cerrar sesión

- **WHEN** una persona pulsa "Cerrar sesión" en su perfil
- **THEN** es llevada a la pantalla de inicio de sesión, sin aviso de error, y al recargar o abrir `/profile` ya no tiene la sesión iniciada

#### Scenario: Cerrar sesión con el servidor caído

- **WHEN** una persona pulsa "Cerrar sesión" y el servidor no responde
- **THEN** la sesión se cierra igualmente en ese navegador y la persona llega a la pantalla de inicio de sesión



## Parte B

Requisitos escritos por el agente: 14
COMPROBADOS POR MI: 1
R1 registro API: parcial

FIN DEL RELOJ

el R1, corregido a completa
el R2, parcial.

### Incoherencias
Ninguna en lo que comprobé (R1 y R2).
### Bug o contrato
No supe decidir si que el login no limite la longitud de la contraseña podría ser un descuido.
No supe decidir si que el login no limite la longitud de la contraseña es una decisión sensata, o un descuido

