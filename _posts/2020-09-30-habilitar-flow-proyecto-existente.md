---
layout: post
thumbnail-img: /assets/img/flow.png
title: Habilitar Flow en un proyecto existente
tags: [ios, android, react native, flow, typescript, babel]
---

Nuestro proyecto incluye ya un archivo llamado `.flowconfig` en el cual se especifica la versión de flow para la que fue generado. Para habilitar flow es necesario instalar justo esa versión en nuestro proyecto. En mi caso la versión es la `0.122.0`.

Con esta información, instalamos flow y el preset para babel, indicando la versión de flow que obtuvimos en el paso anterior:

```sh
yarn add --dev flow-bin@0.122.0 @babel/preset-flow
```

Nuestro siguiente paso es configurar nuestro archivo `babel.config.js` para indicar el preset de flow:

{% highlight javascript linenos %}
module.exports = {
    presets: [
        'module:metro-react-native-babel-preset',
        '@babel/preset-flow',
    ],
};
{% endhighlight %}

En la sección de scripts de `package.json` podemos agregar un script para iniciar y detener el servicio de flow:

{% highlight javascript linenos %}
{
  "name": "jTracker",
  "version": "1.1.0",
  "private": true,
  "scripts": {
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "start": "react-native start",
    "test": "jest",
    "lint": "eslint .",
    "postversion": "react-native-version",
    "flow": "flow"
  },
  ...
}
{% endhighlight %}

De esta forma ejecutaremos el siguiente comando para iniciar el servicio:

```sh
yarn flow
```

Para revisar el estado del servicio:

```sh
yarn flow status
```

Y para detener el servicio:

```sh
yarn flow stop
```

Por último, es necesario indicar que archivos serán evaluados. Esto se hace colocando la siguiente bandera (o comentario) en las primeras líneas de cada uno de los archivos que deseamos que flow evalúe:

```js
//@flow
```

Esta forma de trabajo es ideal ya que como iremos viendo, el trabajo con flow es hasta cierto punto doloroso, pero lo podemos ir implementando poco a poco en nuestro proyecto.

## Cambios a realizar en un componente

El siguiente es un componente sencillo que nos permite generar el típico Hola Mundo y mostrar algunas de las tareas básicas que tendremos que realizar en nuestro proyecto una vez que implementemos flow:

{% highlight jsx linenos %}
import React, { Component } from 'react';
import { Text, View } from 'react-native';
import PropTypes from 'prop-types';

export default class dummy extends Component {
    static propTypes = {
        name: PropTypes.string,
    }

    static defaultProps ={
        name: 'world',
    };

    render() {
        return (
            <View>
                <Text> Hello {this.props.name}! </Text>
            </View>
        );
    }
}
{% endhighlight %}

El primer cambio para realizar es agregar `//@flow` en la línea #1, así obtendremos nuestro primer error:

{: .box-error}
Cannot use  `Component` [1] with fewer than 1 type argument.

Este problema se encuentra en la definición de nuestro componente:

```jsx
export default class Dummy extends Component {
    ...
}
```

ya que espera uno o dos argumentos de tipo: **Props** y **State**. **Props** siempre será requerido y **State** puede ser omitido (por ejemplo, cuando un componente no maneja estado). Modificamos nuestra definición de la siguiente manera:

```jsx
export default class Dummy extends Component<{}, {}> {
    ...
}
```

Esto solucionará el error anterior pero ahora nos arrojará el siguiente:

{: .box-error}
Cannot get `this.props.name` because property `name` is missing in  object type [1].

En este caso, tenemos que agregar `name` al objeto de tipo Props:

```jsx
export default class Dummy extends Component<{name?: string}, {}> {
    ...
}
```

Aquí estamos indicando que el tipo de dato de `name` es `string.` El símbolo `?` indica que el prop es opcional, es decir puedo o no pasarlo, pero si lo paso, su valor será `string` o `undefined`. Con este cambio podemos deshacernos completamente de los famosos **PropTypes**. Hasta ahora nuestro componente va así:

{% highlight jsx linenos %}
//@flow
import React, { Component } from 'react';
import { Text, View } from 'react-native';

export default class Dummy extends Component<{name?: string}, {}> {
    static defaultProps ={
        name: 'world',
    };

    render() {
        return (
            <View>
                <Text> Hello {this.props.name}! </Text>
            </View>
        );
    }
}
{% endhighlight %}

Dentro de React podemos encontrar diferentes tipos que nos pueden ayudar a definir nuestros componentes. Tal es el caso del método `render` cuyo valor de retorno será `React.Node`. Para anotar el tipo de retorno de este método tendremos que importar primero React como un *namespace* del cual podremos hacer referencia a `React.Node`. De esta forma los cambios quedan así:

{% highlight jsx linenos %}
//@flow
import * as React from 'react';
import { Text, View } from 'react-native';

export default class Dummy extends React.Component<{name?: string}, {}> {
    static defaultProps ={
        name: 'world',
    };

    render(): React.Node {
        return (
            <View>
                <Text> Hello {this.props.name}! </Text>
            </View>
        );
    }
}
{% endhighlight %}

Otros [tipos definidos de React pueden ser aquí consultados](https://flow.org/en/docs/react/types/)
