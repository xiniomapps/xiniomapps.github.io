---
layout: post
thumbnail-img: /assets/img/yarn.png
title: Incremento automático de versión y número de compilación
tags: [ios, android, yarn, npm, react native]
---

En un proyecto de React Native para Android y iOS tenemos por lo menos 3 lugares en donde se establece la versión de nuestra App:

1. Package.json
2. build.gradle (Android)
3. Info.plist (iOS)

Estos 3 lugares deben ser modificados cada vez que modificamos la versión o cuando generamos una nueva compilación. Esto último es muy importante al subir a las tiendas ya que se requiere que el número de compilación se haya modificado en relación con la versión subida anteriormente.

Si bien son cambios sencillos, también suele suceder que nos encontremos frecuentemente repitiendo este proceso, por lo que, para simplificar y evitar errores, es recomendable automatizar nuestro día a día utilizando `yarn version` (o si usamos npm: `npm version`).

Básicamente, nos permite modificar la propiedad `version` del archivo `package.json` de nuestro proyecto de forma automática:

{% highlight javascript linenos %}
{
    "name": "jTracker",
    "version": "1.0.0",
    "scripts": {
        "android": "react-native run-android",
        "ios": "react-native run-ios",
        "start": "react-native start",
        "test": "jest",
        "lint": "eslint ."
    },
    ...
}
{% endhighlight %}

Hasta aquí muy simple. Pero también hace adicionalmente dos cosas en caso de estar dentro de un repositorio git:

1. Hará el commit con el cambio realizado.
2. Generará un tag de versión

Esto es fácil de comprobar:

- Ejecutar `yarn version` en modo interactivo: nos solicitará el número de versión en el formato `MAJOR.MINOR.PATCH` (por ejemplo 1.0.0):

![Ejemplo ejecución yarn version](/assets/img/2020-09-14/yarn-version.png){: .mx-auto.d-block :}

{: .box-note}
**Nota:** Tanto yarn como npm esperan que la versión se encuentre en el formato conocido como semver (Versionado Semántico). Para conocer más sobre esta especificación consultar [semver.org](https://semver.org)

- Mostrar el último commit con el comando `git log -p -1`:

![Ejemplo ejecución yarn version commit](/assets/img/2020-09-14/yarn-version-commit.png){: .mx-auto.d-block :}

- Mostrar el tag con el comando `git tag`:

![Ejemplo ejecución yarn version tag](/assets/img/2020-09-14/yarn-version-tag.png){: .mx-auto.d-block :}

Con esto, hemos modificado solo uno de los 3 lugares que habíamos mencionado requieren ser modificados cada que creemos una nueva versión. En caso de las modificaciones de los archivos específicos de Android y iOS, estos los modificaremos usando una herramienta llamada react-native-version y la encadenaremos con lo aprendido hasta ahora.

Una característica que hace muy útil al comando `yarn version` es que en el mismo archivo `package.json` dentro de la propiedad scripts, se definen tres propiedades llamadas preversion, version y postversion que se ejecutan en la siguiente secuencia:

1. `preversion` se ejecuta antes de cualquier cosa y establece una variable de ambiente llamada `npm_package_version`. En el script `preversion` esta variable se refiere a la versión antes del cambio.
2. hacer el cambio de versión en `package.json`
3. ejecutar el script `version` donde `npm_package_version` ahora se refiere a la nueva versión.
4. Commit y etiquetado en git.
5. ejecutar script `postversion`

Es así como podemos ejecutar automáticamente `react-native-version` cada vez que ejecutamos `yarn version` haciendo uso del script `postversion`:

Instalación de `react-native-version`:

```sh
yarn add react-native-version --dev
```

Agregar en `postversion`:

{% highlight javascript linenos %}
{
  "name": "jTracker",
  "version": "1.0.1",
  "private": true,
  "scripts": {
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "start": "react-native start",
    "test": "jest",
    "lint": "eslint .",
    "postversion": "react-native-version"
  },
  ...
}
{% endhighlight %}

Ejecutar `yarn version`

![Ejemplo de ejecución react-native-version](/assets/img/2020-09-14/react-native-version.png){: .mx-auto.d-block :}
