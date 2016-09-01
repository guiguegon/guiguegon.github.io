---
layout: post
title: Permisos en Android 6
---

Desde el lanzamiento de la última versión mayor del sistema operativo, Android 6, Google instauró un nuevo sistema para acceder a las diversas funcionalidades que ofrece el sistema operativo. Los permisos declarados en el fichero AndroidManifest.xml ahora incluyen un matiz importante ya que el propio usuario debe aceptar o rechazar de manera individual permisos considerados “peligrosos”.

Los permisos se han dividido según el riesgo que conllevan en normales y peligrosos. Los permisos normales cubren áreas donde no existen riesgos para la privacidad del usuario o la seguridad del dispositivo, siendo los peligrosos aquellos que pueden dar acceso a información sensible tal como la localización del usuario, los contactos, etc.

El nuevo paradigma hace que los denominados permisos peligrosos deban ser solicitados por la aplicación en el momento en el que sean requeridos. El usuario debe decidir si quiere o no aceptarlos y la aplicación debe estar preparada para que una funcionalidad no este disponible.
<h3><a id="Nueva_pantalla_de_detalle_de_aplicacin_6"></a>Nueva pantalla de detalle de aplicación en Android 6</h3>
Otro de los cambios para los usuarios de Android es un mayor control sobre las aplicaciones instaladas en su teléfono desde la nueva pantalla de detalle de aplicación. Esta novedad hace que sea mucho más fácil comprobar cuánto espacio ocupa en el teléfono, qué uso de red y batería está haciendo y qué permisos tiene concedidos. Desde esta vista, el usuario puede conceder o denegar un permiso en concreto, incluso con la aplicación abierta (aunque en este caso se forzaría el cierre de la misma).

<div align="center">
     <figure>
        <img src="/images/permisos/info_app_1.png" alt="App Info 1" width="300">
        <img src="/images/permisos/info_app_2.png" alt="App Info 2" width="300">
        <img src="/images/permisos/info_app_3.png" alt="App Info 3" width="300">
        <figcaption>Información de la app</figcaption>
    </figure>
</div>

<h3><a id="Solicitar_un_permiso_peligroso_10"></a>Solicitar un permiso peligroso en Android 6</h3>
Si una aplicación está compilada con la versión 23 del SDK y es ejecutada en un móvil con Android 6.0, será necesario solicitar los permisos peligrosos que se utilicen. Los teléfonos con versiones anteriores están exentos de este paso.

El flujo para solicitar un permiso es el siguiente:
<ul>
 	<li>Comprobar si un permiso ha sido denegado alguna vez o ha sido denegado permanentemente</li>
 	<li>En caso de que haya sido denegado alguna vez, mostrar mensaje aclaratorio.</li>
 	<li>Solicitar permiso en caso necesario.</li>
 	<li>Continuar la ejecución con o sin la funcionalidad.</li>
</ul>

<div align="center">
    <figure>
        <img src="/images/permisos/diagrama_permisos.png" alt="Diagrama permisos" width="600">
        <figcaption>Flujo para solicitar permisos</figcaption>
    </figure>
</div>

Cuando un permiso es denegado la primera vez, la aplicación puede mostrar un diálogo donde se explique o aclare para qué necesita ese permiso. En la segunda solicitud el usuario puede denegar permanentemente el permiso solicitado. El SDK de Android proporciona herramientas para comprobar el estado de uno o varios permisos. Con este sistema, Google quiere evitar que los usuarios acepten todos los permisos que la aplicación quiera como se hacía hasta ahora y responsabilizar al usuario de aceptar permisos que puedan afectar a su privacidad.
<h3><a id="Aplicacin_de_ejemplo_22"></a>Aplicación de ejemplo de Android 6</h3>
He creado una <a href="https://github.com/guiguegon/AndroidPermissionsExample">pequeña aplicación</a> de ejemplo para ilustrar este post. Al clicar en uno de los botones, se comprueba el estado del permiso a solicitar. En caso necesario se muestra un diálogo que explica por qué se necesita el permiso y se solicita. Los resultados de la acción del usuario se reciben en el callback onRequestPermissionsResult donde se actualiza la UI:


```java
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    switch (requestCode) {
        case REQUEST_CODE_PERMISSION_WRITE_EXTERNAL_STORAGE:
            updatePermissionText(writeExternalStorageStatusTextView, Manifest.permission.WRITE_EXTERNAL_STORAGE, grantResults[0]);
            break;
        case REQUEST_CODE_PERMISSION_READ_CONTACTS:
            updatePermissionText(readContactsStatusTextView, Manifest.permission.READ_CONTACTS, grantResults[0]);
            break;
        case REQUEST_CODE_PERMISSION_ACCESS_FINE_LOCATION:
            updatePermissionText(locationTextView, Manifest.permission.ACCESS_FINE_LOCATION, grantResults[0]);
            break;
    }
}
```