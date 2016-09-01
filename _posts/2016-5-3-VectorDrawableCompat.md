---
layout: post
title: VectorDrawableCompat, optimización de recursos en Android
---

Las aplicaciones móviles cada vez requieren un mayor esfuerzo en la parte de diseño. Animaciones, iconos, imágenes, recursos en general que deben ser cuidados al detalle para no desbaratar la experiencia de usuario diseñada. Hace poco hablábamos de la importancia de todas las partes que componen un proyecto móvil: diseño, mobile y backend. Hoy voy a comentar mi experiencia tratando con los recursos en un proyecto Android y vamos a ver como optimizar y reducir el impacto de los recursos en la app. Una manera, que hasta hace poco no era posible, es utilizar vectores en lugar de imágenes. La clase VectorDrawableCompat nos ayudará a utilizarlos en nuestros proyectos Android.
<h3>Recursos en Android</h3>
Los recursos en Android se dividen según tipos:

<ul>
    <li>Animaciones: son ficheros en xml donde se definen las diversas animaciones que pueden hacerse sobre un elemento en pantalla.</li>
    <li>Colores: los colores son definidos en RGB en ficheros xml.</li>
    <li>Layout: definen la estructura de una vista mediante elementos en ficheros xml.</li>
    <li>Strings: cadenas de caracteres para mostrar en la app, también en ficheros xml.</li>
    <li>Estilos: unión de varios elementos anteriores para aplicar conjuntamente a la app. XML.</li>
    <li>Drawables: imágenes, iconos, bitmaps tanto en formato png, jpeg, gif o en <strong>formato xml.</strong></li>
    <li>Raw: el resto de recursos posibles: sonidos, tipografía, etc.</li>
</ul>

<div align="center">
    <figure>
        <img src="/images/vectordrawablecompat/resources_structure.png" alt="Organización de recursos" width="250">
          <figcaption>Organización de recursos</figcaption>
    </figure>
</div>

Prácticamente todos los recursos pueden ser definidos en formato XML, el cual sólo se trata de cadenas caracteres. En cambio, las imágenes suelen ser en formatos tipo png o jpeg, que aunque son formatos que admiten compresión, ocupan bastante espacio dentro de la app (sobre todo las de resoluciones xx ó xxxhdpi). Existen multitud de herramientas que permiten optimizar, reducir y eliminar información superflua para reducir el tamaño de una imagen pero siempre habrá un mínimo. Además, siempre serán necesarias imágenes en diferentes resoluciones para acomodar correctamente el diseño en Android.

Otro problema añadido de las imágenes son sus dimensiones fijas. A veces, es necesario tener la misma imagen duplicada porque nuestro amigo diseñador (esas grandes personas, muy queridas por nosotros, los desarrolladores) ha decidido que esa imagen aparezca en varios puntos. Si sólo tenemos una imagen, tendrá que ser escalada para adaptarse a uno o más tamaños con la consecuente pérdida de calidad y tiempo de cpu.
<h3>Imágenes vectoriales</h3>
Los vectores o imágenes vectoriales son aquellas que están definidas mediante objetos geométricos independientes (<em>líneas</em>, <em>círculos</em>, <em>elipses</em>, etc...) que juntas generan una imagen. El equivalente sería reconstruir una imagen a partir de unas claves, como si de un mapa se tratase. No voy a entrar en las ventajas e inconvenientes de utilizar gráficos vectoriales, pues está fuera del alcance de este post; pero sí nos interesa una de ellas: la posibilidad de convertir una imagen en una serie de elementos descritos por separado. ¿No os suena? Exacto, podemos tener una imagen descrita en un xml e importarlo a nuestro proyecto Android.

<div align="center">
    <figure>
        <img src="/images/vectordrawablecompat/svg_logo.svg" alt="Organización de recursos" width="100">
          <figcaption>Logo SVG</figcaption>
    </figure>
</div>

Os dejo una muestra. Al abrir el logo del formato SVG con un editor de texto, encontraremos algo parecido a lo siguiente:

```json
<svg xmlns="http://www.w3.org/2000/svg" 
    xmlns:xlink="http://www.w3.org/1999/xlink" 
    width="100%" height="100%" viewBox="0 0 100 100">

  <title>SVG Logo</title>

    <rect
        id="background"
        fill="#FF9900"
        width="100"
        height="100"
        rx="4"
        ry="4"/>

    <rect
        id="top-left"
        fill="#FFB13B"
        width="50"
        height="50"
        rx="4"
        ry="4"/>

  ...
```

Existen muchos formatos para las imágenes vectoriales. El formato SVG es una recomendación del W3C desde 2001 y es ampliamente utilizado para exportar gráficos. Quiero recalcar que no todas las imágenes pueden ser convertidas a vectores, pero sí una amplia mayoría. Para una muestra, os dejo <a href="http://www.freepik.es/vectores-populares" target="_blank">el siguiente enlace</a>.
<h3>VectorDrawableCompat</h3>
Los vectores llevan rondando Android desde hace tiempo. Exactamente desde la versión 5 (Lollipop), cuando fue introducida en la API la clase VectorDrawable. Con un formato ligeramente diferente al SVG, podían incluirse en el proyecto gráficos vectoriales con la limitación de que sólo podían ser utilizados en la API 21.

Como cualquier desarrollador Android sabrá, es mejor utilizar la librería de soporte para acceder a una funcionalidad o elemento si está disponible, ya que siempre estará más actualizada que la versión incluida en el SDK. Mejoras, corrección de bugs o algunos cambios son incluidos con regularidad en las librerías de soporte. En la versión 23.2, Google introdujo la nueva clase VectorDrawableCompat, que nos permite utilizar vectores desde la API 7 en adelante.

<div align="center">
    <figure>
        <img src="/images/vectordrawablecompat/google_icons.png" alt="https://design.google.com/icons/" width="300">
          <figcaption>https://design.google.com/icons/</figcaption>
    </figure>
</div>

Para hacer pruebas, os dejo la propia <a href="https://design.google.com/icons/" target="_blank">web de iconos de material design de Google</a>, donde podréis encontrar multitud de iconos en formatos png, svg o tipográficos. Como comenté antes, el formato svg no es compatible directamente con Android y tenemos que hacer una conversión. Desde Android Studio 2.0 se pueden importar vectores directamente a un proyecto, pero en ocasiones no funciona correctamente. Yo prefiero utilizar la herramienta gratuita <a href="http://inloop.github.io/svg2android/" target="_blank">svg2android</a>.

<div align="center">
    <figure>
        <img src="/images/vectordrawablecompat/svg2android.png" alt="svg2android">
          <figcaption>Convirtiendo a vector un svg con svg2android</figcaption>
    </figure>
</div>

Una vez convertido a vector, debemos crear un fichero dentro de la carpeta <em>drawable </em>con el contenido que nos proporcione la herramienta. He cogido el icono <em>add</em> del apartado <em>Content</em>  y la diferencia de tamaño entre utilizar un vector o utilizar ficheros png es de 6KB. Si multiplicamos por todos los iconos que use nuestra aplicación, podemos disminuir el tamaño total en varios cientos de KB.
<h3>Configurando un proyecto para usar VectorDrawableCompat</h3>
Como siempre, os dejo un enlace al <a href="https://github.com/guiguegon/VectorDrawableCompat" target="_blank">repositorio</a> con código de ejemplo. Además de incluir la librería de soporte v7, es necesario una sentencia en el fichero gradle que depende de la versión que estés usando:

```java
// Gradle Plugin 2.0+  
 android {  
   defaultConfig {  
     vectorDrawables.useSupportLibrary = true  
    }  
 }  


 // Gradle Plugin 1.5  
 android {  
   defaultConfig {  
     generatedDensities = []  
  }  

  // This is handled for you by the 2.0+ Gradle Plugin  
  aaptOptions {  
    additionalParameters "--no-version-vectors"  
  }  
 }
```

Una vez añadido el vector al proyecto como se explica en el punto anterior, utilizamos el atributo <em>srcCompat</em> para utilizarlo como fuente en un ImageView:

```markdown
<ImageView
        android:id="@+id/vector"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/vector_text"
        app:srcCompat="@drawable/ic_add_vector"/>
```
Y el resultado lo podéis ver a continuación. Gracias las propiedades de los vectores, podremos ajustar al tamaño deseado sin perder calidad. Os animo a utilizar vectores en vuestros próximos proyectos.

<div align="center">
    <figure>
        <img src="/images/vectordrawablecompat/vectordrawablecompat_app.png" alt="vectordrawablecompat" width="578">
          <figcaption>App de ejemplo: VectorDrawableCompat</figcaption>
    </figure>
</div>