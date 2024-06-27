---
title: Lo nuevo de React 19
description: Descubre las novedades de la próxima versión de React
date: 2024-06-27 12:29:00 -0300
categories: [Blog, React]
tags: [react, frontend]
media_subpath: /assets/img/posts/2024-06-27-lo-nuevo-de-react-19
image:
  path: banner.png
  alt: Lo nuevo de React 19
---

Después de más de 2 años, se acerca una nueva versión mayor de React, una de las librerías más populares para construir interfaces de usuario en la web.

La [versión beta de React 19](https://react.dev/blog/2024/04/25/react-19) ya está disponible para probar en npm. Aunque todavía no está lista para que actualicemos nuestros proyectos, es un buen momento para aprender cuáles son las novedades.

## useTransition

Este hook ya existía, pero la novedad es que **ahora permite llamar funciones asíncronas**, como un _request_ (llamada) a un API. El propósito de `useTransition` es actualizar el estado sin bloquear el UI para el usuario. Esto se hace al marcar una actualización de estado como una **transición**.

Supongamos que tenemos una función `updateName()` para que el usuario actualice su nombre. Internamente esta función hace un request al API y devuelve un `Promise` (promesa). Luego de que el usuario escribe su nombre en un `input`, llamamos a esa función. Si queríamos renderizar el UI para que la operación se muestre como pendiente, por ejemplo deshabilitando el botón de guardar para no enviar más de un request a la vez, teníamos que hacer seguimiento con una variable de estado, posiblemente con `useState()`:

```tsx
function UpdateNameControl({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  // Seguimiento de la operación asíncrona
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    // Marcar la operación como pendiente
    setIsPending(true);
    const error = await updateName(name);
    // Marcar la operación como completada
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    }
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```
{: file="update-name-control.tsx" }

Ahora veamos cómo nos ayuda `useTransition()`. Este hook devuelve un array con el estado de la operación (`isPending`) y una función (`startTransition`) para manejar la operación asíncrona:

```tsx
function UpdateNameControl({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
    })
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```
{: file="update-name-control.tsx" }

En el callback `handleSubmit()` ahora llamamos a `startTransition()`, y como parámetro le pasamos la función asíncrona. Cuando el usuario presione el botón y el callback llame a `startTransition`, el valor de `isPending` pasará a ser `true`, lo cual renderizará el botón deshabilitado. Cuando `updateName()` resuelva su promesa, manejamos el error si es necesario, y al terminar la transición, automáticamente `isPending` pasará a `false`, habilitando nuevamente el botón.

Como vemos, `useTransition()` nos ayuda a mantener una UI reactiva e interactiva mientras se procesan las operaciones asíncronas como una llamada al API.

## Actions

Las funciones que utilizan transiciones asíncronas ahora se llaman *actions* (acciones), y ayudan con el envío de datos al API:

- Proveen el estado de si está pendiente, como vimos en el ejemplo con `isPending`.
- Soportan el nuevo hook `useOptimistic` (que se explica más adelante en este artículo) para actualizaciones optimistas donde se prefiere responder al usuario instantáneamente mientras se está enviando el request.
- Proveen manejo de errores: así se puede mostrar un [Error Boundary](https://es.react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) (barrera de error) cuando falla un request.
- Al elemento de formulario `<form>` se le puede pasar una función en el prop `action` o `formAction` para hacer el _submit_ del mismo:

```tsx
<form action={actionFunction}>
```

## useActionState

Este hook tiene el propósito de actualizar el estado en base al resultado de un _form action_ (acción de formulario). Recibe una función como action, y devuelve una action lista para ser llamada.

Volviendo al ejemplo de actualización del nombre de un usuario, se puede simplificar utilizando `useActionState` de la siguiente manera:

```tsx
// Using <form> Actions and useActionState
function UpdateNameControl({ name, setName }) {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get("name"));
      if (error) {
        return error;
      }
      redirect("/path");
      return null;
    },
    null,
  );

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>Update</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```
{: file="update-name-control.tsx" }

## useFormStatus

Otro hook nuevo, en este caso tiene la utilidad de acceder al estado del formulario en un componente hijo (como si el formulario fuera un `Context`). Esto permite escribir componentes de diseño sin hacer prop-drilling:

```tsx
function SaveButton() {
  const { pending } = useFormStatus();
  return <button type="submit" disabled={pending} />
}
```

## useOptimistic

Este hook está basado en el patrón de actualizaciones optimistas, esto se refiere a cuando enviamos un request al API y queremos renderizar el UI de tal manera que damos por hecho que el request va a ser exitoso, por eso el término optimista.

En este ejemplo, antes de llamar al API estamos llamando `setOptimisticName()`, lo cual hará que optimisticName se renderice con ese valor inmediatamente mientras el request está en progreso. Cuando el request se complete o si falla, `optimisticName` volverá al `currentName`:

```tsx
function UserForm({currentName, onUpdateName}) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async formData => {
    const newName = formData.get("name");
    setOptimisticName(newName);
    const updatedName = await updateName(newName);
    onUpdateName(updatedName);
  };

  return (
    <form action={submitAction}>
      <p>Your name is: {optimisticName}</p>
      <p>
        <label>Change Name:</label>
        <input
          type="text"
          name="name"
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```
{: file="user-form.tsx" }

## use

Con este nombre, uno pudiera pensar que es un hook, pero en realidad no es un hook, sino un nuevo API de React que se puede llamar en el renderizado de un componente o hook. Recibe como parámetro una promesa o un Context.

Cuando se lo llama con una promesa, React va a suspender el renderizado hasta que se resuelva la promesa. Combinado con [Suspense](https://es.react.dev/reference/react/Suspense), podemos mostrar un componente o elemento alternativo mientras la promesa no esté resuelta:

```tsx
import { use } from 'react';

function Comments({commentsPromise}) {
  // `use` suspenderá hasta que la promesa resuelva
  const comments = use(commentsPromise);
  return comments.map(comment => <p key={comment.id}>{comment}</p>);
}

function Page({commentsPromise}) {
  // Cuando `use` suspenda en el componente Comments,
  // Suspense mostrará el fallback o componente/elemento alternativo
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  )
}
```
{: file="page-example.tsx" }

Ahora bien, use() no solamente acepta una promesa sino tambien un Context. Esto permite acceder a un Context de manera condicional:

```tsx
import { use } from 'react';
import ThemeContext from './ThemeContext'

function Heading({children}) {
  if (children == null) {
    return null;
  }
  
  // Esto no funcionaría con useContext
  // por el return temprano.
  const theme = use(ThemeContext);
  return (
    <h1 style={{color: theme.color}}>
      {children}
    </h1>
  );
}
```
{: file="heading.tsx" }

Esto muestra una característica interesante de `use()`: se lo puede llamar condicionalmente en el renderizado de un componente o hook, algo que no podemos hacer con un hook.

## ref como un prop 

Ahora se puede acceder `ref` como un prop directamente, sin tener que utilizar `forwardRef`. Esto se utiliza para exponer un nodo DOM al componente padre. Veamos primero cómo teníamos que hacer antes, por ejemplo para exponer el nodo de un input a un formulario padre:

```typescript
const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});

function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Ingresa tu nombre" ref={ref} />
      <button type="button" onClick={handleClick}>
        Editar
      </button>
    </form>
  );
}
```
{: file="form-example.tsx" }

Ahora sencillamente podemos acceder al `ref` como un prop, sin utilizar `forwardRef`:

```tsx
const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});
```
{: file="my-input.tsx" }

Esto se puede hacer solamente con functional components (componentes funcionales) y no con class componentes (componentes construidos con clases) porque estos últimos hacen referencia a la instancia del componente.

## Context como provider 

Ahora se puede renderizar `<Context>` como un provider en vez de con `<Context.Provider>`:

```tsx
const ThemeContext = createContext('');

function App({children}) {
  return (
    <ThemeContext value="dark">
      {children}
    </ThemeContext>
  );  
}
```
{: file="apps.tsx" }

## Nuevo Compilador

El nuevo compilador se encargará de la [memoizar](https://es.wikipedia.org/wiki/Memoizaci%C3%B3n) (almacenar en caché) valores o funciones de forma automática. Para memoizar teníamos que utilizar `useMemo`, `useCallback`, y `React.memo`.
Pero para que la aplicación tenga un buen rendimiento, había que entender bien cuándo memoizar y cuándo no hacerlo,
para que no tener actualizaciones que no son eficientes. Con el nuevo compilador ya no tenemos que preocuparnos por esto,
no será necesario agregar código para memoizar (no más `useMemo`, `useCallback`, y `React.memo`).

El compilador se vale de su conocimiento de JavaScript y las reglas de react para memoizar valores o grupos de valores en los componentes y hooks. El resultado de utilizar este compilador se notará en el código donde no se haya implementado la memoización de forma correcta: al realizarse ahora de forma automática, mejorará el rendimiento de la aplicación.

## Conclusión

React 19 está cargado de mejoras prometedoras. Una vez que sea estable, será muy bueno probar el nuevo compilador y los nuevos hooks relacionados con las acciones que simplificarán el código y darán herramientas para mejorar la experiencia del usuario.