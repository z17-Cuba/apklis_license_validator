# Validador de pagos de licencias en Apklis

## Clase ApklisLicensePaymentStatus

La clase `ApklisLicensePaymentStatus` está diseñada para leer el estado de las operaciones de pago de licencias en la plataforma Apklis. Esta clase sirve para obtener toda la información relevante sobre la verificación de licencias.

## Atributos
- ✅ **Estado del pago**: Si la licencia está pagada o no (después de la operación).
- 👤 **Información del usuario**: El nombre de usuario de la persona para la que se realiza la acción de compra/comprobación.
- 🔑 **Datos de la licencia**: La licencia de pago activa (si se solicita verificación).
- ❌ **Manejo de errores**: Cualquier error y códigos de estado asociados de la API.

## Ejemplos de uso

### 1. **Pago satisfactorio**
Cuando el pago de una licencia se completa con éxito:
```dart
final status = ApklisLicensePaymentStatus(
  paid: true,
  username: "Juanito Alcachofa",
  license: "abc123-def456-ghi789",
);
```

### 2. **Pago fallido**
Cuando un pago falla con información de error:
```dart
final status = ApklisLicensePaymentStatus(
  paid: false,
  username: "Juanita Alcachofa",
  error: "Ya se tiene una licencia activa",
  statusCode: 402,  
);
```

### 3. **Verificación de licencia activa**
Al comprobar el estado de una licencia existente (y hay una licencia activa):
```dart
final status = ApklisLicensePaymentStatus(
  paid: true,
  username: "Juanito Alcachofa",
  license: "abc123-def456-ghi789",
);
```

## Métodos y Status code asociados

`verifyCurrentLicense`: 
- **402**: si debes pagar por una licencia
- **403**: si las credenciales no se han reconocido (puede haber expirado el access_token del usuario) - se recomienda al usuario que abra la aplicación de Apklis y se autentique (si no está autenticado), o que realice alguna acción para que solicite el refresh_token y el access_token en consecuencia
- **404**: si el grupo de licencias no se ha publicado

`verifyCurrentLicense`: 
- **400**: problemas con el pago (suele ocurrir si da timeout la llamada al API del pago), con reintentar se resuelve
- **403**: si las credenciales no se han reconocido (puede haber expirado el access_token del usuario) - se recomienda al usuario que abra la aplicación de Apklis y se autentique (si no está autenticado), o que realice alguna acción para que solicite el refresh_token y el access_token en consecuencia


## Estructura y clases nativas de Kotlin

- **📁 api_helpers**: Carpeta con las clases requeridas para hacer las peticiones a la API de Apklis (**`ApiService.kt`** 🌐), para un wrapper de las respuestas de la API, ya sean de éxito o error y manejar de forma más eficiente cada estado de la verificación (**`ApiResult.kt`** ✅❌), y el interceptor para leer y probar de forma más cómoda el intercambio entre la API y el plugin (**`LoggingInterceptor.kt`** 📋).

- **📁 models**: Carpeta con las clases de dato (o modelos) requeridas para hacer/o leer las peticiones a la API de Apklis 📄.

- **📁 signature_helper**: Carpeta con la clase que se encarga de validar que la petición a la API de Apklis y su respuesta se realizan de forma segura y sin intermediarios (**`SignatureVerificationService.kt`** 🔐).

- **🔌 `ApklisDataGetter.kt`**: Clase se llama mediante la app de ejemplo en Flutter para obtener los datos del Provider expuesto en la app de Apklis necesarios para la validación.

- **🔌 `ApklisLicenseValidatorPlugin.kt`**: Clase padre que se llama mediante la app de ejemplo en Flutter que reconoce los métodos llamativos y devuelve los valores/errores.

- **⚙️ `PurchaseAndVerify.kt`**: Clase que contiene los métodos a llamar desde la clase padre **`ApklisLicenseValidatorPlugin`** y que contiene la lógica de la verificación y pago de licencias 💳.

- **📱 `QRDialogManager.kt`**: Clase que se encarga de manejar, dibujar y mostrar el código QR del pago de Transfermóvil 📲.

- **🔌 `WebSocketClient.kt` + `WebSocketService.kt`**: Clase (y servicio) que se encarga de conectarse a un servidor WebSocket para la retroalimentación inmediata del pago y el estado de la licencia en el dispositivo (incluso en 2do plano) ⚡. De forma automática se encarga de la conexión al canal de la licencia asociada al dispositivo y al usuario, de la reconexión cada cierto tiempo para evitar desconexiones y de cerrar la conexión cuando ha terminado para ahorrar recursos 🔄.


### Notas

Bajo la ruta **android/src/main/assets/license_private_key.pub** se debe colocar la llave de cifrado generada para cada desarrollador con la que se realiza la comprobación de cifrado para validar que la petición viene de un origen de confianza y se emite la validación en consecuencia

Actualmente la aplicación móvil de Apklis soporta estas versiones de Android, desde [API 24](https://developer.android.com/tools/releases/platforms#7.0) hasta [API 35](https://developer.android.com/tools/releases/platforms#15). La intención inicial era seguir en la misma "idea" de soportar desde Android 7.0, pero para lograr el soporte en la validación de la firma cifrada en el plugin de la licencia se tuvo que migrar el plugin desde minSdk = 24 a [minSdk = 26](https://developer.android.com/tools/releases/platforms#8.0). 


