---
layout: post
title: SineView
---

Aprovechando el post de <a href="http://katade.com/librerias-android-jcenter/">cómo subir tu propia librería al repositorio jCenter</a>, he decidido rescatar de uno de los últimos proyectos de <a href="https://aluxion.com/">Aluxion</a> un componente gráfico que creé desde cero. Se trata de una vista que genera una función senoidal que puede utilizarse para dar feedback al usuario de que una operación se está llevando a cabo: <a href="https://github.com/guiguegon/SineView">SineView</a>.
<h2>Vistas en Android</h2>
Los componentes gráficos en Android se dividen en dos grupos: <a href="https://developer.android.com/reference/android/view/View.html">View</a> y <a href="https://developer.android.com/reference/android/view/ViewGroup.html">ViewGroup</a>. Sus propios nombres nos generan una idea de cuál es la función de cada uno. Mientras que <em>View</em> es el componente gráfico básico, <em>ViewGroup </em>es un conjunto o contenedor invisible de <em>views</em>.

<div align="center">
    <figure>
        <img src="/images/sineview/viewgroup_view.png" alt="viewGroup View" width="300">
          <figcaption>ViewGroup. View</figcaption>
    </figure>
</div>

En la documentación de Android Developers podemos comprobar que cualquier <em>widget </em>o elemento que usamos en nuestros layouts extienden la clase <em>View. TextView, EditText, ImageView </em>o <em>Button </em>son ejemplos de widgets. Aunque cada uno tenga una función y unas propiedades diferentes, todos tienen cosas en común; y la más importante es que heredan de <em>View.</em>

El ciclo de vida de un <em>View </em>empieza con su creación (por código o por xml). Posteriormente, se determina el espacio y la localización de la misma; y, finalmente, se pinta. Esto se traduce en varios métodos:
<ul>
  <li>Constructors <--> onFinishInflate()</li>
  <li>onMeasure(int width, int height) <--> onSizeChanged(int w, int h, int oldw, int oldh)</li>
  <li>onDraw(Canvas canvas)</li>
  <li>onSaveInstanceState() <--> onRestoreInstanceState</li>
</ul>
<h2>SineView</h2>
SineView fue creado con el pretexto de ser una barra de progreso infinita. La idea era dar un feedback al usuario con un elemento de la UI mientras la app hacía operaciones en segundo plano que podían conllevar varios segundos. El hecho de recrear una forma de ola me hizo pensar en la fórmula matemática de la función trigonométrica seno.

La función seno puede ser representada en una gráfica con respecto al tiempo, utilizando la fórmula de una onda senoidal:

```markdown
y = Amplitud * sen ( Frecuencia * x + Fase)
```
<div align="center">
    <figure>
        <img src="/images/sineview/sine.jpg" alt="sine" width="150">
          <figcaption>Onda senoidal</figcaption>
    </figure>
</div>

En ella, un par de puntos (x,y) y los siguientes parámetros definen la onda:
<ul>
  <li>Amplitud: distancia entre el eje X y el punto más alto de la onda</li>
  <li>Frecuencia: frecuencia de una onda</li>
  <li>Fase: desplazamiento con respecto al eje Y</li>
</ul>
Una vez claro el concepto, podemos traducir este comportamiento a una vista en Android:

```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    generatePath(canvas);
}

protected void generatePath(Canvas canvas) {
    mPath = new Path();
    float x = 0;
    float y = generateSinePoint(x);
    mPath.moveTo(x, y);
    for (; x < mRectF.width(); x += STEP_X) {
        y = generateSinePoint(x);
        mPath.lineTo(x + mRectF.left, y + mRectF.top + scaleY);
    }
    mPath.lineTo(measuredWidth, measuredHeight + Y_OFFSET);
    mPath.lineTo(0, measuredHeight + Y_OFFSET);
    mPath.close();
    canvas.drawPath(mPath, mPaint);
}

protected float generateSinePoint(float x) {
    return (float) (amplitude * Math.sin(x * frequency - phase));
}
```
<ol>
  <li><em>onDraw, </em>donde se recibe el objeto <em>Canvas, </em>que es el encargado de "dibujar" elementos básicos (un rectángulo, una línea, texto, un bitmap) para construir la vista completa.</li>
  <li><em>generatePath </em>recorre toda la anchura de la vista utilizando la variable estática STEP_X para definir la distancia entre puntos.</li>
  <li>El método <em>generateSinePoint</em> es la traducción literal de la formula de la onda.</li>
</ol>
El resultado final es el siguiente:

<div align="center">
    <figure>
        <img src="/images/sineview/sineview.png" alt="SineView" width="323">
          <figcaption>SineView</figcaption>
    </figure>
</div>

<h2>Generando Feedback al usuario: animación</h2>
Ya tenemos construida nuestra vista estática que pinta un seno en el layout. Pero el objetivo era utilizarlo como una barra de progreso, por lo que debemos animar la onda.

El parámetro fase nos indica la situación instantánea de una onda, desplazando cierta distancia hacia un lado o hacia otro. Si conseguimos crear pequeños pasos incrementales de la fase de nuestra onda, parecerá que se está moviendo en el tiempo.

Android, desde la API 11, añadió la clase ValueAnimator, que permite modificar propiedades de un <em>View</em><em> </em>para hacer un efecto animado (aparecer, rotar, escalar, etc.). Una de las funciones que tiene esta clase es la de obtener los valores de una animación para utilizarlos de manera manual. De tal manera, podemos obtener un <em>animator</em> que nos sirva para animar una onda entre 0 - 2PI y completar así un paso de nuestra animación.

```java
protected void setValueAnimator() {
    progressValueAnimator = ValueAnimator.ofFloat(0, (float) (2 * Math.PI));
    progressValueAnimator.setDuration(paramSineAnimTime);
    progressValueAnimator.setRepeatCount(ValueAnimator.INFINITE);
    progressValueAnimator.setRepeatMode(ValueAnimator.RESTART);
    progressValueAnimator.setInterpolator(new LinearInterpolator());
}
```

Nuestro ValueAnimator nos proporcionará valores lineales (de ahí el interpolador lineal) entre 0 y 2PI con modo de repetición infinito, para que nunca pare (hasta que lo hagamos manualmente) y en modo repetición.

En cada iteración de la animación deberemos pintar la onda de nuevo con los valores actualizados. ValueAnimator nos proporciona la interfaz <em>AnimatorUpdateListener, </em>donde podremos modificar los parámetros de la onda y repintar.

```java
@Override
public void onAnimationUpdate(ValueAnimator animation) {
    step = (float) animation.getAnimatedValue();
    calculatePhase();
    invalidate();
}
```

El método <em>calculatePhase()</em> modificará la fase de la onda en función de la variable <em>step</em> y que, posteriormente, invalidaremos con el método <em>invalidate()</em> para que la vista vuelva al proceso de medida y pintado.

<div align="center">
    <figure>
        <img src="/images/sineview/sineview.gif" alt="SineView" width="323">
          <figcaption>SineView animado</figcaption>
    </figure>
</div>

<h2>Conclusiones</h2>
En este post hemos visto cómo crear una vista propia utilizando una función trigonométrica. <a href="https://github.com/guiguegon/SineView">Aquí podéis encontrar</a> el repositorio con el código y, si queréis utilizarla en vuestros proyectos, sólo tendréis que añadir la siguiente dependencia en vuestro fichero <em>build.gradle:</em>

```markdown
compile 'es.guiguegon:sineview:1.0.0'
```