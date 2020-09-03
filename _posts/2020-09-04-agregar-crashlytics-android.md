---
layout: post
title: Agregar Crashlytics (Android) con React Native Firebase
tags: [errors, android, firebase, react native]
---

La configuración de Crashlytics es un proceso bastante sencillo y [bien documentado](https://firebase.google.com/docs/crashlytics/get-started) en el sitio de Firebase por lo que no nos detendremos mucho en esto. Los únicos cambios requeridos son en ambos archivos ``build.gradle`` (top-level y app-level). ([Ver cambios en github](https://github.com/xiniomapps/jTracker/pull/12/files))

Una vez hecha la configuración, en el sitio de Firebase, sección Crashlytics, se nos pide forzar un crash para poder validar que la configuración es correcta:

![Crashlytics en espera](/assets/img/2020-09-04/crashlytics-screenshot.png){: .mx-auto.d-block :}

Para poder forzar el crash usaremos una librería llamada **React Native Firebase**.

[React Native Firebase](https://rnfirebase.io) es una colección de paquetes que nos permiten trabajar con los servicios de Firebase en React Native de forma sencilla, ya sea para apps en iOS o Android. En este blog estaremos revisando algunos de estos servicios y su equivalente React Native Firebase por lo que es un buen momento para implementar la librería en el proyecto.

Los únicos pasos para instalar son los siguientes:

```text
yarn add @react-native-firebase/app
yarn add @react-native-firebase/crashlytics
```

Con lo que ya podremos hacer uso de esta API para generar un [crash](https://rnfirebase.io/reference/crashlytics#crash), entre otras cosas. Para lograr esto vamos a agregar un botón en jTracker, específicamente en el tab Acerca de:

Agregamos el módulo de Crashlytics de React Native Firebase:

```text
import crashlytics from '@react-native-firebase/crashlytics';
```

Creamos un botón que al oprimirse ocasionará que haya un crash nativo en la app:

```html
<Button
    onPress={() => crashlytics().crash()}
    title='Crash me'
/>
```

Este es el resultado:

<iframe width="560" height="315" src="https://www.youtube.com/embed/0CD125GSOG0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>{: .mx-auto.d-block :}

Si todo se ha configurado bien, tendremos los datos de nuestro crash directamente en nuestra cuenta de Firebase. Ejemplo:

![Firebase Crashlytics Stack Trace](/assets/img/2020-09-04/stack-trace.png){: .mx-auto.d-block :}

![Firebase Crashlytics Data](/assets/img/2020-09-04/data.png){: .mx-auto.d-block :}
