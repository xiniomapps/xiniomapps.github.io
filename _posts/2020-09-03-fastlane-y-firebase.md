---
layout: post
title: Configurar carril fastlane para Firebase (Android)
thumbnail-img: /assets/img/fastlane.png
tags: [jTracker, fastlane, android]
---



Se asume que tanto Firebase como fastlane están instalados y configurados
previamente, por lo que solo necesitamos instalar el plugin ``firebase_app_distribution``
([repo](https://github.com/fastlane/fastlane-plugin-firebase_app_distribution)). Ejecutamos el siguiente comando dentro de nuestro directorio android:

```text
bundle exec fastlane add_plugin firebase_app_distribution
```

Esto genera cambios en los archivos ``Gemfile``, ``Gemfile.lock`` para agregar la
configuración del path donde encontraremos los plugins, siendo el defecto el
archivo ``Pluginfile``:

````text
source "https://rubygems.org"

gem "fastlane"

plugins_path = File.join(File.dirname(__FILE__), 'fastlane', 'Pluginfile')
eval_gemfile(plugins_path) if File.exist?(plugins_path)
````

en base a esta configuración se genera un archivo nuevo llamado ``Pluginfile``
con el siguiente contenido:

```text
gem 'fastlane-plugin-firebase_app_distribution'
```

A continuación configuramos el carril en el archivo ``Fastfile``:

{% highlight text linenos %}
platform :android do
    desc "Enviar Beta a Firebase"
    lane :firebase do
        # Generar un APK de release
        gradle(task: 'assemble', build_type: 'Release')

        # Configuración Firebase
        firebase_app_distribution(
            # App Id:
            app: "x:xxxxxxxxxxxx:android:xxxxxxxxxxxxxxxxxxxxxx",
            
            # Lista de correos de los testers
            testers: "alguien@correo.com, otro@correo.com",
            
            # Notas de Liberación
            release_notes: "Una lista de cambios y su descripción para nuestros testers"  
        )
    end
end
{% endhighlight %}

Este pudiera ser el fin de esta entrada del blog pero, por el momento, sigo teniendo
la duda de si App Id es un dato sensible que no deba ser agregado al control
de versiones por temas de seguridad. Ante la duda lo consideraremos como un
secreto por lo que implementaremos otra forma de obtener
esa información de configuración sin necesidad de tenerla en el control de versiones.

Y lo mismo va para la lista de testers y las notas de liberación. Todos estos datos
son de configuración y cambiarán continuamente por lo que hace sentido almacenarlos
en el ambiente.

{: .box-note}
**Nota:** Existe una guía llamada [The Twelve-Factor App](https://12factor.net/es/) que
nos indica, entre otras cosas, que las configuraciones se deben de guardar siempre
en el entorno y nunca en el control de versiones.

Estos secretos y/o datos de configuración se almacenarán en un archivo de ambiente
el cual quedará siempre fuera del control de versiones. La ventaja de esta
alternativa es que podemos definir diferentes ambientes dependiendo de las necesidades.

Para implementar esta solución instalaremos ``dotenv`` en la raíz del proyecto android:

En el archivo ``Gemfile`` agregar:

```text
gem "dotenv"
````

y ejecutamos el siguiente comando para instalar todas las dependencias:

```sh
bundle update
```

A continuación creamos el archivo ``.env.default`` con las siguientes variables:

```sh
FIREBASEAPPDISTRO_APP="x:xxxxxxxxxxxx:android:xxxxxxxxxxxxxxxxxxxxxx"
FIREBASEAPPDISTRO_TESTERS="alguien@correo.com, otro@correo.com"
FIREBASEAPPDISTRO_RELEASE_NOTES="Pruebas pruebas y más"
```

Este archivo se leerá automáticamente por fastlane para obtener los datos de
configuración específicos de este plugin. Cabe mencionar que cada plugin maneja
sus propias variables de ambiente por lo que si estamos usando algún otro plugin,
tendremos que fijarnos en su documentación para ver que valores tendremos que
declarar en el archivo ``.env.default``:

Por último hay que modificar FastFile para eliminar todos estos datos, quedando
de la siguiente forma:

```text
desc "Enviar Beta a Firebase"
lane :firebase do
    gradle(task: 'assemble', build_type: 'Release')
    firebase_app_distribution()
end
```

Para probar, ejecutamos nuestro carril:

```text
bundle exec fastlane firebase
````

Y comprobamos que todo haya funcionado correctamente.

Un último paso será modificar nuestro archivo ``.gitignore`` para **NO** agregar este archivo al control de versiones:

```text
# Ignore dotenv
.env*
```
