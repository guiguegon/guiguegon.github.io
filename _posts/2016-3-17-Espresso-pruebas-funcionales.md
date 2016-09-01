---
layout: post
title: Espresso, haciendo pruebas funcionales en Android
---

Todos sabemos la importancia del testing en software. Detectar errores o un mal funcionamiento de una o más partes de una aplicación, permite incrementar sustancialmente la experiencia final del usuario. En aplicaciones móviles también es importante y todas deberían tener una batería de pruebas. Google nos proporciona las herramientas necesarias para testear una aplicación y en este post vamos a ver cómo desarrollar pruebas funcionales en Android con Espresso.
<h3>Pruebas funcionales</h3>
Las pruebas funcionales son aquellas en las que se ejecutan y revisan las funcionalidades de una aplicación. Son ensayos exhaustivos que buscan comprobar que el software cumple con los requisitos de las especificaciones. Por norma general, estas pruebas determinan que a partir de una entrada (acción) debe darse cierta salida (resultado). Además, se realizan sobre el software real y completo, incluyendo todo hardware que sea necesario. En aplicaciones móviles, es necesario utilizar un dispositivo físico o un emulador que disponga de todas las capacidades de uno real.

En el desarrollo guiado por pruebas (test drive development o TDD), las pruebas funcionales se escriben a posteriori, tras la toma de requisitos. El objetivo es que las pruebas fuercen el buen funcionamiento de la aplicación y se debe desarrollar en torno a ellas (de ahí el nombre de esta práctica). Existen multitud de frameworks para llevar a cabo este cometido.

En el caso de Android, Google liberó en 2013 Espresso, un framework para la automatización de pruebas funcionales. Aunque ya existían otras soluciones similares como <em>Roboelectric</em> o <em>Robotium</em>, Espresso creció rápidamente gracias al apoyo de Google como first party. El uso de una sintaxis fluida que facilita la lectura del código y la reutilización de librerías ya usadas como <em>Hamcrest</em> o <em>Assertj </em>animaron a los desarrolladores a usarla.
<h3>Espresso: Android UI Tests</h3>
La API de Espresso está enfocada a utilizar el patrón ViewMatcher/ViewAction/ViewAssertion, que viene a ser una sintaxis en la que, utilizando el propio lenguaje, queda definida la acción a realizar sobre una vista/objeto y su correspondiente comprobación. La base de este patrón son las tres clases citadas:

<div align="center">
    <figure>
        <img src="/images/espresso/espresso_chart.png" alt="Funcionamiento Espresso" width="600">
        <figcaption>ViewMatcher/ViewAction/ViewAssertion</figcaption>
    </figure>
</div>
 
<ul>
    <li>ViewMatcher: encuentra una vista que cumpla una o más condiciones (withId, withText, isChecked, ...).</li>
    <li>ViewAction: acciones a realizar sobre una vista (click, typeText, swipeLeft, ...).</li>
    <li>ViewAssertion: comprueba que el resultado es el que debe de ser.</li>
</ul>
A la hora de escribir las sentencias de una prueba para facilitar la comprensión y la legibilidad, se hacen imports estáticos para eliminar el nombre de las clases. Android studio nos ayuda a realizar estos imports de manera sencilla. El resultado es una frase que el lector puede entender con facilidad:

<div align="center">
    <figure>
        <img src="/images/espresso/static_import.png" alt="import estatico" width="600">
        <figcaption>Import estático</figcaption>
    </figure>
</div>
     
```markdown
 onView(withId(R.id.email)).perform(typeText("email@dominio.com"))
```
La anterior sentencia podría leerse como: "en la vista con id <em>email</em> haz la acción escribir el texto <em>email@dominio.com</em>"

Las posibilidades de Espresso son bastante grandes y a veces puede parecer difícil encontrar la manera exacta de probar cierta funcionalidad. La documentación y los ejemplos que proporciona Google son de gran ayuda en este sentido, y quiero destacar la "chuleta" disponible en la página de Espresso, que nos será de gran ayuda. En ella podemos ver todas las sentencias posibles<em> </em>con todas las posibilidades que nos ofrece la librería.

<div align="center">
    <figure>
        <img src="/images/espresso/espresso_full_chart.png" alt="chuleta espresso" width="600">
        <figcaption>Chuleta espresso</figcaption>
    </figure>
</div>

<h3>Código de ejemplo</h3>
En mi cuenta de <a href="https://github.com/guiguegon" target="_blank">GitHub</a> he subido una <a href="https://github.com/guiguegon/EspressoTestingExample" target="_blank">aplicación de ejemplo</a> para ilustrar este post. He utilizado el asistente de Android Studio para crear dos activities: <em>login</em> y <em>main.</em> El funcionamiento es simple: si introduces las credenciales correctas, accedes a la pantalla principal; y, en caso contrario, aparece un mensaje de error. Con dos sencillas pruebas tenemos nuestras pruebas funcionales:

```java 
@Test
 public void testLoginWrong() {
    onView(withId(R.id.email)).perform(typeText("email@dominio.com"));
    onView(withId(R.id.password)).perform(replaceText("contraseña"));
    onView(withId(R.id.email_sign_in_button)).perform(click());
    onView(withId(R.id.password)).check(matches(hasErrorText(activityRule.getActivity().getString(R.string.error_incorrect_password))));
}
```

Este primer test comprueba que tras introducir credenciales inválidas se muestra un mensaje de error

```java
@Test
public void testLoginSuccess() {
    Intents.init();
    onView(withId(R.id.email)).perform(typeText("foo@example.com"));
    onView(withId(R.id.password)).perform(replaceText("hello"));
    onView(withId(R.id.email_sign_in_button)).perform(click());
    intended(hasComponent(MainActivity.class.getName()));
    Intents.release();
}
```

El segundo test comprueba que se lanza MainActivity si se introducen credenciales válidas. Aquí se ve el uso de una extensión de Espresso llamada Intents que básicamente chequea que se lanza el intent correspondiente tras una acción.

<h3>Conclusiones</h3>
Las pruebas funcionales son una pieza fundamental para asegurar la calidad de una aplicación. El hecho de que sean costosas en tiempo no debería decantar la balanza para no hacer testing. Si pintáramos una gráfica de tiempo versus esfuerzo en dos proyectos, uno con pruebas y otro sin, conforme avancen los proyecto el número de bugs y errores disminuirán considerablemente, haciendo que el esfuerzo final sea mucho menor y compense el esfuerzo inicial.

Felices pruebas!