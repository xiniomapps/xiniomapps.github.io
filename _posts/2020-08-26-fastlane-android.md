---
layout: post
title: Configuración Fastlane (Android)
thumbnail-img: /assets/img/2020-08-26/fastlane.png
cover-img: /assets/img/2020-08-26/cover.png
tags: [jTracker, fastlane, android]
---

El proyecto de hoy tratará de echar a andar [fastlane](https://fastlane.tools/)
en nuestro proyecto jTracker. Fastlane es una herramienta que nos permite automatizar
ciertos aspectos relacionados con la liberación de nuestra app en las tiendas
como por ejemplo:

- Automatización de generación de Screenshots: permite generar en múltiples resoluciones
a la vez, por lo que también puede ser considerada una herramienta de pruebas para la
visualización correcta de nuestra app en diferentes pantallas.
- Generación y distribución de versiones de prueba interna, alfa, beta y productivas
- Firmado de código

{: .box-note}
**Ticket GitHub:** [Ticket #1](https://github.com/xiniomapps/jTracker/issues/1)

{: .box-note}
**Rama GitHub:** [Rama 1-configurar-fastlane-android](https://github.com/xiniomapps/jTracker/tree/1-configurar-fastlane-android)

### Prerequisito: Credenciales para acceder a Google Play Console

Se requiere generar y descargar una llave tipo JSON para poder aceder a Google
Play Console desde fastlane. El proceso se describe en la [documentación oficial
de fastlane](https://docs.fastlane.tools/getting-started/android/setup/#collect-your-google-credentials).


Una vez descargada la llave, se deberá colocar en un nuevo directorio llamado `keys`
en la carpeta raíz del proyecto. Este directorio **deberá ser excluido del control
de versiones** (git):

{% highlight text linenos %}
# .gitignore

# ignorar directorio keys:
keys/
{% endhighlight %}

Podemos comprobar que nuestra llave funciona correctamente y que nos permite el acceso a Google Play 
de la siguiente manera:

{% highlight text %}
bundle exec fastlane run validate_play_store_json_key json_key:../keys/jtracker.json
{% endhighlight %}

Este es el resultado cuando funciona correctamente:

![Conexión de prueba a Google Play Store](/assets/img/2020-08-26/002.png){: .mx-auto.d-block :}

### Instalar y configurar fastlane

Generar un archivo llamado ``Gemfile`` en la carpeta android de nuestro proyecto. Dentro de este archivo agregar las siguientes lineas:

{% highlight text linenos %}
source "https://rubygems.org"
gem "fastlane"
{% endhighlight %}

![Ubicación del archivo Gemfile](/assets/img/2020-08-26/001.png){: .mx-auto.d-block :}

a continuación hacer la instalación de fastlane, lo que generará un archivo llamado `Gemfile.lock`:

```sh
bundle update
```

Por último es necesario generar los archivos de configuración de fastlane:


```sh
export LC_ALL="en_US.UTF-8"

bundle exec fastlane init
```

{: .box-note}
**Nota:** La asignación de la variable LC_ALL se hace para evitar algunos
problemas como los [documentados aquí](https://github.com/fastlane/fastlane/issues/227).

El comando ``fastlane init`` nos solicitará los siguientes datos:

1. Package Name: **com.jtracker**
2. JSON Secret File: **ruta al archivo json generado anteriormente**
3. Deseamos usar fastlane para subir metadatos, screenshots y builds a Google Play?: **SI**

Si todo se ha configurado correctamente,
tendremos dos archivos y una carpeta en ``android/fastlane``:

![Archivos generados por fastlane](/assets/img/2020-08-26/003.png){: .mx-auto.d-block :}

- **Appfile:** Archivo de configuración, con los datos que nos fueron preguntados anteriormente.
- **Fastfile:** Definición de lanes.
- **metadata:** screenshots, iconos, descripciones, etc. En caso de que existieran
 en Google Play y hayamos configurado correctamente, estos se descargarán.


La configuración de `Appfile` for defecto se muestra a continuación:
{% highlight text linenos %}
# Appfile:

json_key_file("../keys/jtracker.json")
package_name("com.jtracker")
{% endhighlight %}

Para `Fastfile` hicimos cambios para definir dos procesos de liberación (lanes):
uno para hacer el envío del bundle AAB al canal de pruebas internas y otro
proceso para hacer el envío al canal beta:

{% highlight text linenos %}
# Fastfile:

platform :android do
    desc "Enviar a canal Internal Testing en Google Play Store"
    lane :internal do
        gradle(task: 'bundle', build_type: 'Release')
        upload_to_play_store(track: 'internal', skip_upload_apk: true)
    end

    desc "Enviar a canal Beta en Google Play Store"
    lane :beta do
        gradle(task: 'bundle', build_type: 'Release')
        upload_to_play_store(track: 'beta', skip_upload_apk: true)
    end
end
{% endhighlight %}

Para generar el bundle y subir a pruebas internas hacemos lo siguiente:

```
bundle exec fastlane internal
```

Cabe mencionar que en cada ejecución tendremos que hacer ajustes en el archivo
`build.gradle` para cambiar el número de versión (`versionCode`) a un numero único.
En una entrada futura de este blog automatizaremos esta parte.

{% highlight text linenos %}
# Extracto de android/app/build.gradle: 
    defaultConfig {
        applicationId "com.jtracker"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 6
        versionName "v1.0-beta2"
    }
{% endhighlight %}

Por último, revisamos en Google Play Console si todo ha funcionado correctamente:

![Bundle en Google Play Console](/assets/img/2020-08-26/004.png){: .mx-auto.d-block :}