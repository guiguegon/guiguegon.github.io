---
layout: post
title: Galería (Parte I)
---

En cualquier aplicación puede que tengamos que mostrar al usuario una serie de imágenes de manera gráfica para que pueda seleccionarla como su avatar, imagen de fondo, etc. Si estas imágenes no son predefinidas, la primera aproximación suele ser llevada a cabo usando la aplicación <strong>Galería</strong> que todos los dispositivos tienen. Lanzando una intención con la acción <em>ACTION_PICKER, </em>mostramos al usuario una interfaz conocida que permite seleccionar cualquier imagen propia.

Pero ¿qué ocurre si necesitamos algo más personalizado? ¿Cómo obtener y pintar en una lista o cuadrícula todas las imágenes o videos que están almacenados en el dispositivo? El hecho de personalizar una galería en lugar de utilizar la app nativa del dispositivo, nos permite jugar con algunas opciones que iré desvelando en próximas entregas.
<h2>Galería</h2>
El primer paso será construir un grid para pintar todos los ficheros. Para aprovechar al máximo la pantalla, he decidido calcular, a partir de un ancho mínimo, el número de columnas. Así podremos usarla en diferentes modelos de dispositivo con un número dinámico de elementos en pantalla.

```java
int columns = getMaxColumns();
galleryAdapter = new GalleryAdapter(getContext(), columns);
staggeredGridLayoutManager = new StaggeredGridLayoutManager(columns, StaggeredGridLayoutManager.VERTICAL);
galleryRecyclerView.setLayoutManager(staggeredGridLayoutManager);
galleryRecyclerView.setAdapter(galleryAdapter);
```

Utilizo un StaggeredGridLayoutManager, ya que una de sus funciones es permitir a un elemento ocupar más de una columna. Gracias a esto, podemos tener una cabecera que ocupe todo el espacio y colocar diversas opciones (en nuestro caso será un botón para abrir la cámara).

<div align="center">
    <figure>
        <img src="/images/gallery/galeria.png" alt="Galería" width="720">
          <figcaption>Galería de animalitos</figcaption>
    </figure>
</div>

```java
@Override
public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
 View v;
 switch (viewType) {
    case VIEW_HOLDER_TYPE_HEADER:
        v = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.item_gallery_header, viewGroup, false);
        return new GalleryHeaderViewHolder(v);
    case VIEW_HOLDER_TYPE_ITEM:
    default:
        v = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.item_gallery, viewGroup, false);
        return new GalleryItemViewHolder(v);
    }
}

@Override
public void onBindViewHolder(RecyclerView.ViewHolder viewHolder, int position) {
 try {
     final ViewGroup.LayoutParams layoutParams = viewHolder.itemView.getLayoutParams();
     StaggeredGridLayoutManager.LayoutParams sglayoutParams =
     (StaggeredGridLayoutManager.LayoutParams) layoutParams;
     if (viewHolder instanceof GalleryHeaderViewHolder) {
         sglayoutParams.setFullSpan(true);
         fill((GalleryHeaderViewHolder) viewHolder);
     } else {
         sglayoutParams.setFullSpan(false);
         fill((GalleryItemViewHolder) viewHolder, galleryMedias.get(position - 1));
     }
     viewHolder.itemView.setLayoutParams(sglayoutParams);
  } catch (Exception e) {
    Log.e(TAG, "[onBindViewHolder] ", e);
  }
}
```

Utilizando el método <i>setFullSpan(true)</i> sobre el ViewHolder obtenemos este comportamiento.
<h2>Obtener las imágenes</h2>
Una vez tenemos configurado nuestra vista, debemos obtener del dispositivo las imágenes a mostrar. Para ello, debemos utilizar una de las herramientas que nos proporciona el SDK de Android: <strong>ContentProvider.</strong>

Un ContentProvider no es más que un proveedor de contenido entre aplicaciones a través de la interfaz <strong>ContentResolver</strong>. Un ejemplo de contenido que se comparte entre apps son los contactos, que están almacenados en una base de datos. Este contenido se considera exportable y puede ser solicitado por otra app. En nuestro caso, necesitamos obtener todas las imágenes y videos que estén en la galería del dispositivo.

```java
final String[] columns = {
   MediaStore.Images.Media.DATA, 
   MediaStore.Images.Media._ID, 
   MediaStore.Images.Media.MIME_TYPE,
   MediaStore.Images.Media.DATE_TAKEN };
final String orderBy = MediaStore.Images.Media.DATE_TAKEN;
Cursor imageCursor = context.getContentResolver()
   .query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, columns, null, null, orderBy + " DESC");
```

Este snippet nos proporciona todo lo que necesitamos:
<ul>
  <li>Las columnas data, id, mimeType y dateTaken.</li>
  <li>En orden descendente del campo dateTaken</li>
  <li>La uri MediaStore.Images.Media.EXTERNAL_CONTENT_URI en un ContentResolver.</li>
</ul>
Como veis, tiene una sintaxis parecida a una petición SQL (los parámetros null serían para hacer selection sobre los resultados). Una vez completada la consulta, obtenemos un <strong>Cursor</strong> que recorremos para mapear todos los ficheros y poder devolvérselos a nuestro adapter.

```java
imageCursor.moveToFirst();
int imageCount = imageCursor.getCount();
for (int i = 0; i < imageCount; i++) {
    ...
    imageCursor.moveToNext();
}
imageCursor.close();
```

Una vez obtenidas todas las imágenes, solo nos queda solicitar todos los videos; ya que no se pueden obtener en la misma petición. Se pueden obtener utilizando MediaStore.Video.Media.EXTERNAL_CONTENT_URI, como hemos visto con las imágenes.

Es importante resaltar que estas operaciones son de un gran costo en términos de rendimiento y no deben ser nunca ejecutadas en el hilo principal. Existen multitud de opciones, yo he tomado el <strong>camino reactivo </strong>y he utilizado <a href="http://katade.com/programacion-reactiva-en-android-rxjava/">RxJava</a>.

```java
public void getGalleryAsync() {
    final Observable<List<GalleryMedia>> observable =
           Observable.create((Observable.OnSubscribe<List<GalleryMedia>>) subscriber -> {
               subscriber.onNext(getGallery());
               subscriber.onCompleted();
           }).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());
    observable.subscribe(this::onGalleryMedia, this::onGalleryError);
}
```

<h2>Obtener nueva foto de la cámara</h2>
La cabecera que hemos dejado libre servirá al usuario para lanzar la app de la cámara y hacer una foto nueva. Para ello, simplemente necesitaremos lanzar una intención y que la reciba el sistema. Además, para tener mayor control sobre el fichero nuevo, proporcionaremos la uri donde se escribirá el nuevo contenido. Este paso es opcional, pues podemos permitir al sistema manejarlo directamente.

```java
public static File getOutputMediaFile(int type) {
    File mediaStorageDir =
            new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES), MEDIA_FOLDER);
    if (!mediaStorageDir.exists() && !mediaStorageDir.mkdirs()) {
        Log.e(TAG, "Failed to create directory " + MEDIA_FOLDER);
        return null;
    }
    String timeStamp = now();
    File mediaFile;
    switch (type) {
        case MEDIA_TYPE_IMAGE:
            mediaFile = new File(mediaStorageDir.getPath() + File.separator + "VID_" + timeStamp + ".jpeg");
            break;
        case MEDIA_TYPE_VIDEO:
            mediaFile = new File(mediaStorageDir.getPath() + File.separator + "VID_" + timeStamp + ".mp4");
            break;
        default:
            mediaFile = null;
            break;
    }
    return mediaFile;
}
```

```java
Intent cameraIntent = new Intent();
cameraIntent.setAction(MediaStore.ACTION_IMAGE_CAPTURE);
ContentValues values = new ContentValues(1);
mimeType = MIME_TYPE_IMAGE;
values.put(MediaStore.Images.Media.MIME_TYPE, mimeType);
mediaUri = Uri.fromFile(FileUtils.getOutputMediaFile(FileUtils.MEDIA_TYPE_IMAGE));
cameraIntent.putExtra(MediaStore.EXTRA_OUTPUT, mediaUri);
cameraIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
activity.startActivityForResult(cameraIntent, REQUEST_CODE_CAMERA);
```

Una vez completada la operación, recibiremos los datos directamente desde onActivityResult y deberemos refrescar nuestro grid para añadir la nueva imagen:

```java
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    GalleryMedia galleryMedia = cameraHelper.onGetPictureIntentResults(requestCode, resultCode, data);
    if (galleryMedia != null) {
        onGalleryMedia(galleryMedia);
    }
}
```

```java
public GalleryMedia onGetPictureIntentResults(final int requestCode, final int resultCode, final Intent data) {
    if (requestCode == REQUEST_CODE_CAMERA) {
        if (resultCode == Activity.RESULT_OK) {
            galleryAddPic(mediaUri);
            return new GalleryMedia().setMediaUri(mediaUri.getPath()).setMimeType(mimeType);
        }
    }
    return null;
}
```

<h2>GalleryModule</h2>
Llegados a este punto, quiero presentaros la librería que estoy realizando y que sirve de base para este post: <a href="https://github.com/guiguegon/GalleryModule">GalleryModule</a>. Es muy sencillo utilizarla, solamente debéis importarla en vuestro proyecto y utilizar la siguiente sentencia para lanzar la galería:

```java
public void openGallery(View view) {
    startActivityForResult(GalleryActivity.getCallingIntent(this), REQUEST_CODE_GALLERY);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
   super.onActivityResult(requestCode, resultCode, data);
   if (requestCode == REQUEST_CODE_GALLERY && resultCode == RESULT_OK) {
       GalleryMedia galleryMedia = data.getParcelableExtra(GalleryActivity.RESULT_GALLERY_MEDIA);
       Toast.makeText(this, "Gallery Media " + galleryMedia.getMediaUri(), Toast.LENGTH_LONG).show();
   }
}
```

Aquí os dejo la dependencia para añadirla en vuestro fichero <em>build.gradle:</em>

```markdown
compile 'es.guiguegon:gallerymodule:1.0.2'
```

Tengo en mente varias entregas sobre este mismo repositorio para añadir nuevas funcionalidades como multiselección, grabar videos, personalizar los colores o ver la cámara directamente en la galería. Estad atentos a los <a href="/gallery-II/" target="_blank">próximos post</a>.
