---
layout: post
title: Galería (Parte III)
---

Siguiendo con la serie de **galeria**, en la <a href="/gallery-I/">primera entrega</a> vimos cómo crear una galería propia en Android, en la <a href="/gallery-II/">segunda entrega</a> cómo añadir multi selección y cómo grabar videos. En la última entrega de la serie, vamos a añadir la previsualización de la cámara trasera en la cabecera.

<h2>android.hardware.camera y android.hardware.camera2</h2>
Existen actualmente dos API para manejar la cámara de un dispositivo: [android.hardware.camera](https://developer.android.com/reference/android/hardware/Camera.html) y [android.hardware.camera2](https://developer.android.com/reference/android/hardware/camera2/package-summary.html). La primera fue **deprecada** en la versión 21 y sustituída por la segunda versión que añade nuevas funcionalidades como la posibilidad de controlar individualmente parámetros específicos de la cámara. Además el manejo de la API es completamente diferente.

El principal problema que nos enfrentamos al querer utilizar estas clases es que **camera2** no es retrocompatible con versiones inferiores a _Lollipop_ y cuya única solución consiste en hacer dos versiones de nuestra cámara. Aunque android.hardware.camera este deprecado, sigue siendo completamente funcional a día de hoy (comprobado con la beta de Android 7.0 Nougat) y es el camino que voy a seguir en este post.

<h2>TextureView o SurfaceView</h2>
La previsualización de la cámara es un proceso costoso en terminos de recursos. El hilo principal no puede estar atendiendo todas las llamadas de la API sin dejar de responder a otros eventos. Para ello existen las clases [*TextureView*](https://developer.android.com/reference/android/view/TextureView.html) y [*SurfaceView*](https://developer.android.com/reference/android/view/SurfaceView.html), cuya principal funcionalidad es que permiten ser pintadas y renderizadas desde un hilo separado. Ambas pueden ser utilizadas como vista para la previsualización de la cámara, SurfaceView no puede ser transformada ni escalada mientras que TextureView consume más recursos.

Para este ejemplo, he utilizado TextureView aunque con una simple transformación podría usar SurfaceView en su lugar.

<h2>TextureCameraPreview</h2>
La clase **TextureCameraPreview** que he creado para esta entrada, extiende *TextureView* e implementa la interfaz **SurfaceTextureListener** para obtener los callbacks que indican que la vista está preparada para recibir contenido. 

```java
public TextureCameraPreview(Context context, AttributeSet attrs) {
    super(context, attrs, 0);
    setSurfaceTextureListener(this);
}
```

La interfaz SurfaceTextureListener contiene 4 métodos a implementar aunque sólo necesitaremos dos *onSurfaceTextureAvailable* y *onSurfaceTextureDestroyed*.

```java
@Override
public void onSurfaceTextureAvailable(SurfaceTexture arg0, int arg1, int arg2) {
    this.surfaceTexture = arg0;
    // surface esta lista
}

@Override
public boolean onSurfaceTextureDestroyed(SurfaceTexture arg0) {
    // surface destruida
    return true;
}

@Override
public void onSurfaceTextureSizeChanged(SurfaceTexture arg0, int arg1, int arg2) {
    // surface cambia de tamaño
}

@Override
public void onSurfaceTextureUpdated(SurfaceTexture arg0) {
    // surface actualizada
}
```

Cuando la vista esté preparada podremos inicializar la cámara, setear parámetros y comenzar la previsualización. 

```java
mCamera = Camera.open();
mCamera.setPreviewTexture(surfaceTexture);
mCamera.startPreview();
setCameraParameters();
```

Al destruirse la vista deberemos liberar los recursos para que otras aplicaciones puedan usarlas (obtendremos una bónita excepción si no lo hacemos)

```java
mCamera.stopPreview();
mCamera.setPreviewCallback(null);
mCamera.release();
```

Con estas simples líneas obtendremos la previsualización de la cámara en nuestra vista. Uno de los problemas que tiene la API android.hardware.camera es que es síncrona. El hilo que haga la llamada a *Camera.open()* recibirá siempre los callbacks. Esto puede suponer un problema si tenemos que hacer algún tratamiento de imágen o simplemente si, cómo en este caso, queremos incluir la previsualización en un *RecyclerView*. Cuando ejecutas el código anterior, hay un pequeño lag al iniciar y al cerrar la cámara. Este efecto en un RecyclerView se ve reflejado cada vez que el ViewHolder entra y sale de la pantalla causando un molesto parón. 

Para solucionar este inconveniente, podemos usar la nueva API android.hardware.camera2 o bien tratar las llamadas al singleton *Camera* en un hilo separado.

<div align="center">
    <figure>
        <img src="/images/gallery/camera.png" alt="Preview camera" width="576">
          <figcaption>Previsualización de la camara</figcaption>
    </figure>
</div>

<h2>Multithreading</h2>
Como he comentado antes, el hilo que ejecute *Camera.open()* recibirá los callbacks. Debemos crear un nuevo hilo a través de un **HandlerThread** y obtener un **Handler** para poder comunicarnos con el. Una vez tengamos configurado el hilo separado, utilizaremos el hilo principal para setear parámetros de la cámara.

```java
private void initCameraThread() {
    cameraThread = new HandlerThread(CAMERA_THREAD);
    cameraThread.start();
    cameraHandler = new Handler(cameraThread.getLooper());
    mainHandler = new Handler(Looper.getMainLooper());
}
```
Una vez tenemos nuestros handlers preparados para recibir mensajes, los utilizaremos para inicializar la cámara. No debemos olvidar solicitar los <a href="/Permisos-Android-6/">permisos necesarios para Android 6.0</a> (Manifest.permission.CAMERA y Manifest.permission.WRITE_EXTERNAL_STORAGE). En este caso **PermissionsManager** es una pequeña clase que nos ayudará a solicitar estos permisos.

```java
@Override
public void onSurfaceTextureAvailable(SurfaceTexture arg0, int arg1, int arg2) {
    this.surfaceTexture = arg0;
    if (cameraThread == null) {
        initCameraThread();
    }
    checkPermission();
}

private void checkPermission() {
    try {
        PermissionsManager.requestMultiplePermissions((ViewGroup) getParent(), this::initCamera,
                Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE);
    } catch (Exception e) {
        //empty
    }
}

private void initCamera() {
    cameraHandler.post(() -> {
        getCameraInstance();
        try {
            if (mCamera != null) {
                mCamera.setPreviewTexture(surfaceTexture);
                mCamera.startPreview();
                mainHandler.removeCallbacksAndMessages(null);
                mainHandler.postDelayed(this::setCameraParameters, 500);
            }
        } catch (Throwable t) {
            //empty
        }
    });
}

@Override
public boolean onSurfaceTextureDestroyed(SurfaceTexture arg0) {
        surfaceTexture = null;
        mainHandler.removeCallbacksAndMessages(null);
        cameraHandler.post(() -> {
            try {
                if (mCamera != null) {
                    mCamera.stopPreview();
                    mCamera.setPreviewCallback(null);
                    mCamera.release();
                    mCamera = null;
                }
            } catch (Throwable t) {
                //empty
            }
        });
        return true;
}

```

Ahora si tenemos nuestra camara controlada por un hilo separado de la UI.

<h2>Extra: FileProvider</h2>
Con la llegada de Android 7.0 alias **Nougat** surgen algunos problemas con el código utilizado en esta serie. En concreto, una de las nuevas limitaciones (en aras de mejorar la seguridad todo sea dicho) es no poder pasar uris con el esquema *file://* a través de una intención. En la nueva versión de Android esto provoca una excepción de tipo *FileUriExposedException*. Para sortear esta limitación, desde hace varias versiones de la librería de soporte, se añadió la clase **FileProvider** que nos facilita el intercambio de ficheros entre apps.

Para configurar nuestro **FileProvider** nada más fácil que seguir la [documentación oficial](https://developer.android.com/reference/android/support/v4/content/FileProvider.html). Una vez configurado deberemos cambiar nuestro código

```java
mediaUri = Uri.fromFile(FileUtils.getOutputMediaFile(FileUtils.MEDIA_TYPE_IMAGE));
```

por este otro:

```java
File file = FileUtils.getOutputMediaFile(FileUtils.MEDIA_TYPE_IMAGE);
mediaPath = file != null ? file.getAbsolutePath() : null;
mediaUri = FileProvider.getUriForFile(activity, BuildConfig.APPLICATION_ID + ".provider", file);
```

En mi caso, almaceno mediaPath para poder posteriormente utilizarlo ya que mediaUri pasa a ser tal que así:

```markdown
content://(package_name).provider/external_files/Pictures/IMG_1473335502000.jpeg
```

<h2>Conclusiones</h2>
He actualizado el repositorio de <a href="https://github.com/guiguegon/GalleryModule">GalleryModule</a> con el código nuevo y aquí os dejo la dependencia para añadirla en vuestro fichero <em>build.gradle:</em>

```markdown
compile 'es.guiguegon:gallerymodule:1.2.3'
```