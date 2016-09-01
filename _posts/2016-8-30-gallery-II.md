---
layout: post
title: Galería (Parte II)
---

En la <a href="/gallery-I/">primera entrega</a> de esta serie vimos cómo crear una galería propia en Android, cómo obtener las fotos y vídeos del dispositivo, y cómo obtener una nueva foto de la cámara. En esta segunda entrega de la galería he añadido dos funcionalidades que serán muy útiles para cualquier aplicación. De hecho, se pueden extrapolar fácilmente a otros contextos.

La primera de ellas es la selección múltiple: poder seleccionar en una vista varios elementos pintados en un adapter. La segunda es más sencilla: poder grabar vídeos utilizando la app nativa del dispositivo. Veamos cómo implementar ambas en los siguientes puntos.

<h2>Selección multiple</h2>
Para poder seleccionar múltiples elementos, lo primero que debemos hacer es crear un elemento visual que nos permita mostrar al usuario diferentes estados (seleccionado o no). Una de las opciones más sencillas es colocar un <em>View</em> que ocupe todo el espacio y sea invisible mientras el elemento no está seleccionado y visible cuando lo esté. Utilizando un background con un alpha o con un shape, podemos dar un toque diferente a los elementos.

Para mantener los estados de los elementos, necesitamos una lista. Una manera es utilizar un <a href="https://developer.android.com/reference/android/util/SparseBooleanArray.html">SparseBooleanArray</a>, donde podemos almacenar una dupla "posición" -> "valor" para controlar qué elementos hemos seleccionado. Por último, tendremos que notificar que el elemento ha cambiado para que sea repintado. También, para proporcionar una buena experiencia al usuario, mostraremos en la toolbar el número de elementos seleccionados (el tamaño de nuestra lista).

Cuando hayamos terminado la selección, con un bucle que recorra las posiciones contenidas en nuestro SparseBooleanArray, obtendremos todos los ficheros.

<div align="center">
    <figure>
        <img src="/images/gallery/multiple_selection.png" alt="Galería" width="576">
          <figcaption>Multiple selección</figcaption>
    </figure>
</div>

<h2>SelectableAdapter</h2>
Todo lo necesario lo he resumido en una clase: <strong>SelectableAdapter. </strong>Extendiendo esta clase con nuestro adapter, podremos reutilizar esta funcionalidad en cualquier punto de nuestra aplicación. Se basa en el mismo principio explicado anteriormente, una lista en forma de SparseBooleanArray que contiene las posiciones seleccionadas.

Con los métodos <em>isSelected(int position)</em> y <em>togglePosition(int position) </em>controlamos el estado de los elementos. Tenemos métodos auxiliares que seleccionan y deseleccionan todos (<em>selectAll</em> y <em>unSelectAll</em>) y un método que nos devuelve una lista con todas las posiciones marcadas (<em>getSelectedItemPosition</em>).

Hay que tener cuidado al añadir un nuevo elemento a nuestra galería, ya que la lista de posiciones seleccionadas se puede incrementar (o decrementar), y para ello hay otros dos métodos auxiliares (<em>notifySelectableAdapterItemInserted </em>y <em>notifySelectableAdapterItemRemoved</em>).

Si nuestra app tiene cambios de orientación, tendremos que recordar guardar las posiciones seleccionadas.
<h2>Grabar vídeos</h2>
Esta funcionalidad, en realidad, estaba ya implementada en la versión anterior, aunque no estaba del todo lista. La problemática surge porque no podemos utilizar el mismo intent para grabar un vídeo que para capturar una foto. Debemos mostrar un diálogo al usuario para que elija antes de lanzar la intención. En código, sustituyendo la acción en el intent, lanzará la cámara en el modo correcto. Además, <a href="https://developer.android.com/reference/android/provider/MediaStore.html#ACTION_VIDEO_CAPTURE">se pueden añadir</a> flags para grabar en un formato u en otro, limitar la calidad, etc.

<div align="center">
    <figure>
        <img src="/images/gallery/photo_video_dialog.png" alt="Galería" width="576">
          <figcaption>Grabar video o hacer foto</figcaption>
    </figure>
</div>

El código quedaría como sigue:

```java
Intent cameraIntent = new Intent();
cameraIntent.setAction(MediaStore.ACTION_VIDEO_CAPTURE);
ContentValues values = new ContentValues(1);
mimeType = MIME_TYPE_VIDEO;
values.put(MediaStore.Images.Media.MIME_TYPE, mimeType);
mediaUri = Uri.fromFile(FileUtils.getOutputMediaFile(FileUtils.MEDIA_TYPE_VIDEO));
cameraIntent.putExtra(MediaStore.EXTRA_OUTPUT, mediaUri);
cameraIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
activity.startActivityForResult(cameraIntent, REQUEST_CODE_CAMERA);
```

Con respecto al post anterior, hemos modificado la acción (MediaStore.ACTION_VIDEO_CAPTURE) y el mimeType ("video/mp4"). Una vez recibido el vídeo a través de onActivityResult, es necesario, en nuestro caso, obtener la duración del vídeo. Con ayuda de la clase <em>MediaMetadataRetriever</em> y la uri del vídeo, extraemos la información.

```java
private void setGalleryMediaDuration(GalleryMedia galleryMedia) {
    MediaMetadataRetriever retriever = new MediaMetadataRetriever();
    retriever.setDataSource(context, mediaUri);
    String time = retriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION);
    galleryMedia.setDuration(Long.parseLong(time));
}
```

<h2>Conclusiones</h2>
He actualizado el repositorio de <a href="https://github.com/guiguegon/GalleryModule">GalleryModule</a> con el código nuevo y aquí os dejo la dependencia para añadirla en vuestro fichero <em>build.gradle:</em>

```markdown
compile 'es.guiguegon:gallerymodule:1.1.1'
```

Para la próxima entrega, que será la última relacionada con la Galería, he dejado la traca final: añadir al botón de la cámara un elemento para poder ver en tiempo real la cámara trasera. ¡Estad atentos en las próximas semanas!