---
title: Principios SOLID en React
description: Cómo podemos aplicar los principios SOLID en React.
date: 2024-06-29 12:00:00 -0300
categories: [Blog, React]
tags: [react, frontend]
media_subpath: /assets/img/posts/2024-06-29-principios-solid-en-react
image:
  path: banner.png
  alt: React SOLID
---

Los principios SOLID fueron compilados por Bob Martin en un paper
en el año 2000, aunque el acrónimo fue inventando más adelante por
Michael Feathers. El "tio Bob" es el autor de los famosos libros
[Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
y [Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164).
En este último, en el capítulo 3 empieza a considerar los principios
SOLID y después le dedica un capítulo a cada principio.

Los principios SOLID nos ayudan a la hora de organizar funciones y
estructuras de datos en clases y determinar cómo estas se interconectan.
En React hoy en día se utilizan funciones en vez de clases para los
componentes y hooks, sin embargo los principios SOLID no son exclusivos
a la Programación Orientada a Objetos (OOP). El tío Bob explica en el
libro mencionado anteriormente que cuando habla de una clase él se
refiere a un grupo de funciones y datos acoplados, también se refiere
a esto como un **módulo**. La idea es que estos módulos (sean clases o no)
puedan **soportar los cambios** y también sean **fáciles de entender**.

Como se trata de principios, hay diferentes maneras de aplicarlos. Para
que nuestro proyecto o forma de programar se beneficie de ellos, tenemos que
entender bien la idea de cada principio. En este artículo consideramos
una breve explicación de cada principio y algunas aplicaciones
a proyectos de Frontend que utilizan React. También puedes ver este video donde
considero este tema:

{% include embed/youtube.html id='hZ8WTnivcr4' %}

## Single Responsibility Principle (SRP)

El principio de responsabilidad única (SRP) pareciera dar a entender que
una función debería hacer una sola cosa. Si bien esto es una buen práctica,
el tío Bob explica que SRP no se refiere a eso. La definición que el da es:
un módulo debería tener una sola razón para cambiar, o siendo más precisos,
un módulo debería ser responsable a un solo actor. En este contexto un actor
es una persona o un grupo de personas que piden cambios en el software.

Si bien Bob da un ejemplo usando OOP (más aplicable a backend), considerar 
este ejemplo puede ayudarnos a entender la idea.
Supongamos que tenemos una clase `Employee` (Empleado) que tiene tres métodos
o funciones:

```typescript
export class Employee {
  public calculatePay() {
  }

  public reportHours() {
  }

  public save() {
  }
}
```
{: file="employee.ts"}

Pensemos en los actores y los métodos:

- `calculatePay()` lo utiliza el departamento de contabilidad para
calcular el salario de los empleados
- `reportHours()` es utilizado
por el departamento de recursos humanos para saber cuántas horas trabajó cada
empleado
- `save()` es de interés al departamento técnico para guardar la información en la base de datos.

En esta clase tenemos un problema de *coupling* (acoplamiento) entre diferentes actores. Si el departamento de
contabilidad nos pide un cambio y retocamos esta clase, esto pudiera traer
cambios inesperados al código que utiliza el departamento de recursos humanos.
El resultado pudiera ser bugs que causen pérdidas de millones de dólares a la
empresa.

Aplicando SRP podemos separar esta clase pensando en los tres diferentes
actores mencionados, así organizamos el código en tres diferentes clases:

```typescript
export class PayCalculator {
  public calculatePay() {
  }
}

export class HourReporter {
  public reportHours() {
  }
}

export class EmployeeRepository {
  public save() {
  }
}
```
{: file="example-srp.ts"}

Con esto evitamos que el cambio que nos pide un actor afecte el código que
utilice otro actor.

### SRP en React

¿Cómo podemos llevar esta idea al Frontend en React? Pensemos en un componente
que muestra una lista de personas. Este componente se está encargando de 
determinar cómo se ve esta lista: el tamaño de la tipografía, los colores, si
es una tabla o una lista de tarjetas, etc. Pero también se encarga de traer
los datos de estas personas desde un API:

```tsx
import { useEffect, useState } from "react";
import { Person } from "../../types";
import { CircularProgress, Paper, Table, TableBody, TableCell, TableContainer, TableHead, TableRow } from "@mui/material";
import { ActionButton } from "./action-button";

export function PersonsList() {
  const [ persons, setPersons ] = useState<Person[]>([]);
  const [ isLoading, setIsLoading ] = useState<boolean>(true);

  // Load persons from API
  useEffect(() => {
    async function loadPersons() {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      const data = await response.json();
      setPersons(data);
      setIsLoading(false);
    }

    loadPersons();
  }, []);

  return <>
    {
      isLoading ?
        <CircularProgress /> :
        (<TableContainer component={Paper}>
          <Table sx={{ minWidth: 650 }} aria-label="simple table">
            <TableHead>
              <TableRow>
                <TableCell>Name</TableCell>
                <TableCell>Username</TableCell>
                <TableCell>E-mail</TableCell>
                <TableCell>Company</TableCell>
                <TableCell>Address</TableCell>
                <TableCell>Phone</TableCell>
                <TableCell>Website</TableCell>
                <TableCell>Edit</TableCell>
                <TableCell>Delete</TableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {persons.map((person) => (
                <TableRow
                  key={person.id}
                >
                  <TableCell component="th" scope="row">
                    {person.name}
                  </TableCell>
                  <TableCell>
                    {person.username}
                  </TableCell>
                  <TableCell>
                    {person.email}
                  </TableCell>
                  <TableCell>
                    {person.company.name}
                  </TableCell>
                  <TableCell>
                    {person.address.street}, {person.address.city}
                  </TableCell>
                  <TableCell>
                    {person.phone}
                  </TableCell>
                  <TableCell>
                    {person.website}
                  </TableCell>
                  <TableCell>
                    <ActionButton text='Edit' />
                  </TableCell>
                  <TableCell>
                    <ActionButton isDelete={true} />
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>)
    }
  </>
}
```
{: file="persons-list.tsx" .code-limit}

Entonces nos ponemos a pensar... ¿A qué actores responde este código?
¿Quiénes pudieran solicitar cambios de manera independiente? Identificamos
a dos actores:

- Diseñadores de UI/UX: determinan cómo se ve la aplicación y cómo el
usuario interactúa con ella. Seguramente en el futuro nos pedirán cambios.
- Proveedor de API: si bien pudiera ser uno mismo como programador full-stack,
en un proyecto grande pudiera ser un equipo de backend separado, o si el API
está tercerizado pudiera ser una compañía externa.

Teniendo en cuenta los actores, ahora decidimos separar el código, por un lado
tenemos un Hook que se encarga de traer los datos del API:

```typescript
export function usePersons() {
  const [ persons, setPersons ] = useState<Person[]>([]);
  const [ isLoading, setIsLoading ] = useState<boolean>(true);

  // Load persons from API
  useEffect(() => {
    async function loadPersons() {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      const data = await response.json();
      setPersons(data);
      setIsLoading(false);
    }

    loadPersons();
  }, []);

  return {
    persons,
    isLoading,
  }
}
```
{: file="use-persons.ts"}

Y por otro lado un componente que se encarga de determinar cómo se ve la
información, consumiendo el hook mencionado:

```tsx
export function PersonsList() {
  const { persons, isLoading } = usePersons();

  return <>
    {
      isLoading ?
        <CircularProgress /> :
        (<TableContainer component={Paper}>
          <Table sx={{ minWidth: 650 }} aria-label="simple table">
            <TableHead>
              <TableRow>
                <TableCell>Name</TableCell>
                <TableCell>Username</TableCell>
                <TableCell>E-mail</TableCell>
                <TableCell>Company</TableCell>
                <TableCell>Address</TableCell>
                <TableCell>Phone</TableCell>
                <TableCell>Website</TableCell>
                <TableCell>Edit</TableCell>
                <TableCell>Delete</TableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {persons.map((person) => (
                <TableRow
                  key={person.id}
                >
                  <TableCell component="th" scope="row">
                    {person.name}
                  </TableCell>
                  <TableCell>
                    {person.username}
                  </TableCell>
                  <TableCell>
                    {person.email}
                  </TableCell>
                  <TableCell>
                    {person.company.name}
                  </TableCell>
                  <TableCell>
                    {person.address.street}, {person.address.city}
                  </TableCell>
                  <TableCell>
                    {person.phone}
                  </TableCell>
                  <TableCell>
                    {person.website}
                  </TableCell>
                  <TableCell>
                    <ActionButton text='Edit' />
                  </TableCell>
                  <TableCell>
                    <ActionButton isDelete={true} />
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>)
    }
  </>
}
```
{: file="persons-list.tsx" .code-limit}

Podemos llevar la idea un paso más adelante. Seguimos pensando en los
diferentes departamentos de la compañía que utiliza el software y de ahí 
surgen diferentes roles:

- Departamento de contabilidad: se encarga de calcular el salario
dependiendo de las horas trabajadas y las horas extras ellos necesitan ver la
información de esta lista.
- Recursos humanos necesita también ver la cantidad de horas exactas que cada
persona trabajó en esta lista para determinar si necesitan contratar más
personal en ciertas áreas.

Como se trata de una lista de personas con datos similares en principio
decidimos utilizar el mismo componente para ambos. Pero si algo cambia en la 
manera en que contabilidad calcula las horas extras 
(como dice el ejemplo del libro de tío Bob),
esto pudiera afectar a recursos humanos de manera inesperada. Aplicando SRP
podemos separar el código de acuerdo con los actores, en este caso con un 
componente para cada uno:

```tsx
export function AccountingPersonsList() {
  const { persons, isLoading } = usePersons();

  // Renderizado de acuerdo al departamento de contabilidad.
  return <></>;
}

export function HumanResourcesPersonsList() {
  const { persons, isLoading } = usePersons();

  // Renderizado de acuerdo al departamento de recursos humanos.
  return <></>;
}
```
{: file="example-srp.tsx" }

## Open-Closed Principle (OCP)

El principio de abierto-cerrado (OCP) dice que un módulo debería estar
abierto a la extensión pero cerrado a la modificación. Dicho de otra
manera, el comportamiento de un módulo debería ser extensible, si tener
que modificarlo. Este es un principio muy profundo y hay diferentes
maneras de aplicarlo.

### OCP en React

Supongamos que nuestra aplicación requiere que el usuario confirme antes de
eliminar un registro. Para eso tenemos un componente de modal reutilizable
`ConfirmationModal`. Este componente utiliza [Material UI](https://mui.com/material-ui/react-dialog/)
para dibujar el modal, y recibe props que le van a permitir al código que 
consume este componente determinar cuál es el título, el texto de la 
confirmación así como también el texto de los botones:

```tsx
import Button from '@mui/material/Button';
import Dialog from '@mui/material/Dialog';

type ConfirmationModalProps = {
  isOpen: boolean;
  title: string;
  text: string;
  acceptButtonText: string;
  cancelButtonText: string;
  onConfirm: () => void;
  onCancel: () => void;
}

export function ConfirmationModal({
  isOpen,
  title,
  text,
  acceptButtonText,
  cancelButtonText,
  onConfirm,
  onCancel,
}: ConfirmationModalProps) {
  const handleConfirm = () => {
    onConfirm();
  };

  const handleClose = () => {
    onCancel();
  }

  return (
    <Dialog
      open={isOpen}
      onClose={handleClose}
    >
      <DialogTitle>
        {title}
      </DialogTitle>
      <DialogContent>
        <DialogContentText>
          {text}
        </DialogContentText>
      </DialogContent>
      <DialogActions>
        <Button onClick={handleConfirm}>{cancelButtonText}</Button>
        <Button onClick={handleClose} autoFocus>
          {acceptButtonText}
        </Button>
      </DialogActions>
    </Dialog>
  );
}
```
{: file="confirmation-modal.tsx" .code-limit }

![Dialogo de confirmación](ocp-dialog-before.png){: w="603" h="338" }

El consumidor del componente le está pasando mediante el prop `text` el
mensaje de confirmación incluyendo el nombre de una persona:

```tsx
export function Example() {
  const [personToBeDeleted, setPersonToBeDeleted] = useState<Person | undefined>();
  const { person } = useGetPerson();
  const isConfirmDeleteModalOpen = personToBeDeleted !== undefined;
  const confirmDeleteModalTitle = `Confirm Person Deletion`;
  const confirmDeleteDialogText =  personToBeDeleted ? `Are you sure you want to delete ${personToBeDeleted.name}?` : '';

  const onPersonDeleteClicked = (person: Person) => {
    setPersonToBeDeleted(person);
  };

  const onPersonDeleteConfirmed = () => {
    // Process person deletion
    setPersonToBeDeleted(undefined);
  };

  const onPersonDeleteCancelled = () => {
    setPersonToBeDeleted(undefined);
  }

  return <>
    { person ? <PersonCard person={person} onDeleteClicked={onPersonDeleteClicked} />  : <></>}
    <ConfirmationModal
      isOpen={isConfirmDeleteModalOpen}
      title={confirmDeleteModalTitle}
      text={confirmDeleteDialogText}
      acceptButtonText='Delete'
      cancelButtonText='Cancel'
      onConfirm={onPersonDeleteConfirmed}
      onCancel={onPersonDeleteCancelled}
      />
  </>
}
```
{: file="example.tsx" .code-limit }

Un día nos piden que el nombre de la persona se muestre con otro estilo,
quizá en **bold** (negrita) o con otro color, y que también el título de este 
modal de confirmación se vea diferente. Pero para esto no queremos hacer grandes
cambios en nuestro componente original `ConfirmationModal` ya que hay otros
consumidores que lo utilizan y que no tienen esas necesidades específicas.

Aplicando el principio de OCP podemos ajustar el componente para que reciba
el prop **children** en vez de recibir el texto, y para el
título recibimos un nodo de react, en vez de recibir `String` en ambos casos:

```tsx
import { PropsWithChildren, ReactNode } from 'react';

type ConfirmationModalProps = {
  isOpen: boolean;
  title: ReactNode;
  acceptButtonText: string;
  cancelButtonText: string;
  onConfirm: () => void;
  onCancel: () => void;
}

export function ConfirmationModal({
  isOpen,
  title,
  acceptButtonText,
  cancelButtonText,
  children,
  onConfirm,
  onCancel,
}: PropsWithChildren<ConfirmationModalProps>) {
  const handleConfirm = () => {
    onConfirm();
  };

  const handleClose = () => {
    onCancel();
  }

  return (
    <Dialog
      open={isOpen}
      onClose={handleClose}
    >
      <DialogTitle>
        {title}
      </DialogTitle>
      {children}
      <DialogActions>
        <Button onClick={handleConfirm}>{cancelButtonText}</Button>
        <Button onClick={handleClose} autoFocus>
          {acceptButtonText}
        </Button>
      </DialogActions>
    </Dialog>
  );
}
```
{: file="confirmation-modal.tsx" .code-limit }

El consumidor ahora puede personalizar cómo se va a mostrar tanto el título
como el texto, aplicando la etiqueta `<b>` (bold) de HTML:

```tsx
export function Example() {
  ...

  return <>
    { person ? <PersonCard person={person} onDeleteClicked={onPersonDeleteClicked} />  : <></>}
    <ConfirmationModal
      isOpen={isConfirmDeleteModalOpen}
      title={<div>Confirm Person <b>Deletion</b></div>}
      acceptButtonText='Delete'
      cancelButtonText='Cancel'
      onConfirm={onPersonDeleteConfirmed}
      onCancel={onPersonDeleteCancelled}
      >
        <ConfirmationModalContent>
          Are you sure you want to delete <b>{personToBeDeleted?.name}</b>?
        </ConfirmationModalContent>
      </ConfirmationModal>
  </>
}
```
{: file="example.tsx" }

Como se ve en el ejemplo, mediante composición estamos utilizando un componente
`ConfirmationModalContent` que también recibe **children** y utiliza
componentes de Material UI para mostrar el contenido:

```tsx
export function ConfirmationModalContent({
  children,
}: PropsWithChildren) {
  return (
    <DialogContent>
      <DialogContentText>
        {children}
      </DialogContentText>
    </DialogContent>
  )
}
```
{: file="confirmation-modal-content.tsx" }

El punto es que el `ConfirmationModal` ya no necesita modificarse para extender
la forma en que renderiza su contenido. Ahora entonces cuando mostramos el
modal de confirmación, el nombre de la persona está en **bold** (negrita):

![Dialogo de confirmación con bold](ocp-dialog-after.png){: w="609" h="369" }

## Liskov-Substitution Principle (LSP)

El principio de sustitución de Liskov lleva el nombre de su autora,
Barbara Liskov.
Este principio está fuertemente basado en la Programación Orientada a Objetos
(OOP), dice que los objetos de subtipos deberían ser sustituibles por objetos
de supertipos. Estamos hablanado de la herencia en la OOP donde hay clases
padre (super tipo) y clases hija (subtipos) que heredan métodos y propiedades
de su padre. Entonces, el principio podría explicarse como que si una clase B
extiende una clase A, entonces deberíamos poder utilizar B en cualquier lugar
donde usamos A, sin cambiar la funcionalidad importante de la aplicación.

Primero vamos a ver un ejemplo de backend para entender la idea. La clase padre
`Database` es lo suficientemente genérica para que la utilice mi aplicación,
que sencillamente necesita conectarse a una base de datos. Tenemos dos clases
hijas: las implementaciones MySQL y SQLite deben ser compatibles con el método
`connect()`:

```typescript
export class Database {
  public connect() {
  }
}

export class MySQLDatabase extends Database {
  public connect() {
    // Specific MySQL Code.
  }
}

export class SQLiteDatabase extends Database {
  public connect() {
    // Specific SQLite Code.
  }
}
```
{: file="database.ts" }

Cada una tendrá su manera diferente de conectarse. Sin embargo, en el código de
la aplicación vemos que cuando esta inicia se conecta a la base de datos, sin
importar cuál implementación está utilizando, porque estamos siguiendo el
principio de sustitución de Liskov:

```typescript
class Application {
  private database: Database;

  constructor(database: Database) {
    this.database = database;
  }

  public start() {
    this.database.connect();
  }
}

const database = new MySQLDatabase();
const application = new Application(database);
application.start();
```
{: file="application.ts" }

### LSP en React

Vamos a intentar aplicar esta idea en React donde generalmente no se utiliza
Programación Orientada a Objetos. Aunque no tenemos herencia, si utilizamos
**composición**.

En este ejemplo tenemos tres componentes para botones:

```tsx
import styled from '@emotion/styled';
import { Box, Button, ButtonProps } from '@mui/material';
import { FC } from 'react';

export function ButtonsExample() {

  const onButtonClicked = (buttonType: string) => {
    console.log(`Button ${buttonType} clicked!`);
  }

  return <>
    <Box display='flex'>
      <Button variant="contained" onClick={() => onButtonClicked('normal')}>Normal</Button>
      <SquareButton variant="contained" onClick={() => onButtonClicked('squared')}>Squared</SquareButton>
    </Box>
    <ContainedButton variant="contained" onClick={() => onButtonClicked('contained')}>Contained</ContainedButton>
  </>;
}

const SquareButton = styled(Button)({
  borderRadius: 0,
  marginLeft: '1rem',
});

const ContainedButton: FC<ButtonProps> = (props) => {
  return <Box marginY={2}>
    <Button fullWidth={true} {...props}>{props.children}</Button>
  </Box>;
}
```
{: file="lsp-example.tsx" }

- `Button` viene directamente de la libería Material UI.
- `SquareButton` es compatible con el primero, podemos pasarle los mismos
props. En este caso utilizamos styled de emotion para darle estilo al
componente con CSS-in-JS.
- `ContainedButton` utiliza composición para personalizar cómo se mostrará el
botón, en este caso dentro de un Box, ocupando todo el ancho del mismo. Cuando
hacemos este tipo de composición tenemos que pasarle todos los props al componente "padre" (en este caso Button) utilizando el *spread operator*
`{...props}`.

Los tres diferentes componentes para botones reciben las mismas propiedades
(`variant` y `onClick`). Al ser compatibles, podemos intercambiarlos sin romper
la funcionalidad de la aplicación.

## Interface Segregation Principle (ISP)

El principio de segregación de la interfaz dice que un módulo de software no
debería depender de interfaces que no utiliza.

Para entender la idea veamos un ejemplo de backend con OOP. Notemos las
siguientes tres clases de servicio: uno para generar reportes de usuario,
otro para crear usuarios y el último para eliminar usuarios:

```typescript
export class UserReportService {
  constructor(private userRepository: IUserRepository) {
  }

  public print() {
    const users = this.userRepository.getAll();
    console.log(`Printing users`, users);
  }
}

export class UserCreationService {
  constructor(private userRepository: IUserRepository) {
  }

  public create(user: User) {
    return this.userRepository.create(user);
  }
}

export class UserDeletionService {
  constructor(private userRepository: IUserRepository) {
  }

  public delete(user: User) {
    this.userRepository.delete(user);
  }
}
```
{: file="user-services.ts" .code-limit }

Estamos utilizando el patrón de *repository* (repositorio). Este es el código
de `UserRepository`, que se encargará del manejo de la base de datos. Permitirá
obtener la lista de usuarios, también crear o eliminarlos:

```typescript
export interface IUserRepository {
  getAll(): User[];
  create(user: User): User;
  delete(user: User): void;
}

export class UserRepository implements IUserRepository {
  public getAll(): User[] {
    return [];
  }

  public create(user: User) {
    console.log(`Creating user ${user.name}`);
    return user;
  }

  public delete(user: User): void {
    console.log(`Deleting user ${user.name}`);
  }
}
```
{: file="user-repository.ts" }

El repositorio está implementando la interfaz `IUserRepository`, esto es
importante ya que estamos hablando del principio de segregación de la 
**interfaz**.

Si miramos de nuevo a `UserReportService` que genera reportes, podemos notar
que solamente utiliza el método `getAll` del repositorio, claro nunca va a
crear o eliminar usuarios como lo hacen `UserCreationService` y 
`UserDeletionService`.

Ahora aplicando el principio de segregacion de la interfaz separamos
`IUserRepository` en dos: una con los métodos de lectura y otra con los métodos
de escritura. El repositorio sigue siendo el mismo, solamente que implementa
dos interfaces separadas:

```typescript
export interface IUserReadRepository {
  getAll(): User[];
}

export interface IUserWriteRepository {
  create(user: User): void;
  delete(user: User): void;
}

export class UserRepository implements IUserReadRepository, IUserWriteRepository {
  public getAll(): User[] {
    return [];
  }

  public create(user: User) {
    console.log(`Creating user ${user.name}`);
  }

  public delete(user: User): void {
    console.log(`Deleting user ${user.name}`);
  }
}
```
{: file="user-repository.ts" }

Esto permite a los diferentes tipos de clientes consumir solo la interfaz que 
ellos necesitan. Entonces en vez de tener una interfaz de propósito general,
ahora hemos segregado esa interfaz en algunas más específicas. No es necesario
crear una interfaz por cada cliente (eso podría resultar en muchas interfaces)
sino que la hemos separado por **tipo** de cliente, uno de lectura y otro de
escritura. Esto abre la posibilidad en el futuro de separar el repository,
uno de escritura y otro de lectura que se conectan a diferentes instancias de
la base de datos. Esto se hace más fácil gracias a que primero hemos segregado
la interfaz:

```typescript
export interface IUserReadRepository {
  getAll(): User[];
}

export interface IUserWriteRepository {
  create(user: User): void;
  delete(user: User): void;
}

export class UserReadOnlyRepository implements IUserReadRepository {
  public getAll(): User[] {
    return [];
  }
}

export class UserWriteRepository implements IUserWriteRepository {
  public create(user: User) {
    console.log(`Creating user ${user.name}`);
  }

  public delete(user: User): void {
    console.log(`Deleting user ${user.name}`);
  }
}
```
{: file="user-repository.ts" .code-limit}

### ISP en React

Ahora llevando esta idea al Frontend con React, supongamos que tenemos un
componente de reporte de usuario que está trayendo todos los usuarios desde un
hook `useUsers()` para imprimir los usuarios en pantalla:

```tsx
export function UserReport() {
  const { users, isLoadingUsers } = useUsers();

  return isLoadingUsers ?
    <>Loading Users</> :
    users.map((user) => (<div>{user.name}</div>))
}
```
{: file="user-report.tsx"}

También tenemos un formulario de creación de usuarios que permite crear un
usuario, y en este caso estamos utilizando el mismo hook `useUsers()`:

```tsx
export function UserCreationForm() {
  const { createUser } = useUsers();

  const [name, setName] = useState<string>();

  function handleNameChange(event: React.ChangeEvent<HTMLInputElement>) {
    setName(event.target.value);
  }

  function handleSubmit(event: React.FormEvent) {
    event.preventDefault();
    if (!name) {
      alert('User name is not valid');
      return;
    }
  
    createUser(name);
  }

  return <form onSubmit={handleSubmit}>
    <label>
      Name:
      <input type="text" value={name} onChange={handleNameChange} />
    </label>
    <input type="submit" value="Submit" />
  </form>
}
```
{: file="user-creation-form.tsx"}

El hook `useUsers()` está encapsulando toda la funcionalidad que tiene que ver 
con los usuarios. Está trayendo los usuarios desde el API y también tiene
funciones de creación y eliminación de usuarios:

```ts
export function useUsers() {
  const [ users, setUsers ] = useState<User[]>([]);
  const [ isLoadingUsers, setIsLoadingUsers ] = useState<boolean>(true);

  // Load users from API
  useEffect(() => {
    async function loadUsers() {
      const response = await fetch('/api/user');
      const data = await response.json();
      setUsers(data);
      setIsLoadingUsers(false);
    }

    loadUsers();
  }, []);

  const createUser = async (name: string) => {
    const user: Partial<User> = {
      name,
    };
    await fetch('/api/user', {
      method: 'POST',
      body: JSON.stringify(user),
    });
  }

  const deleteUser = async (userId: string) => {
    await fetch(`/api/user/${userId}`, {
      method: 'DELETE',
    });
  }

  return {
    users,
    createUser,
    deleteUser,
    isLoadingUsers,
  }
}
```
{: file="user-users.ts" .code-limit }

Ahora como estamos utilizando el mismo hook, tenemos el efecto no deseado de
que en el formulario que se utiliza para crear un usuario, se está llamando
al API para traer la lista de usuarios.

Aplicando el principio de segregación de la interfaz, podemos separar el hook
que tenía una interfaz más amplia o general, en dos diferentes hooks con
intefaces más específicas: uno para obtener los usuarios del API y otro para
administrar usuarios (crear o eliminarlos):

```typescript
export function useGetUsers() {
  const [ users, setUsers ] = useState<User[]>([]);
  const [ isLoadingUsers, setIsLoadingUsers ] = useState<boolean>(true);

  // Load users from API
  useEffect(() => {
    async function loadUsers() {
      const response = await fetch('/api/user');
      const data = await response.json();
      setUsers(data);
      setIsLoadingUsers(false);
    }

    loadUsers();
  }, []);

  return {
    users,
    isLoadingUsers,
  }
}

export function useManageUsers() {
  const createUser = async (name: string) => {
    const user: Partial<User> = {
      name,
    };
    await fetch('/api/user', {
      method: 'POST',
      body: JSON.stringify(user),
    });
  }

  const deleteUser = async (userId: string) => {
    await fetch(`/api/user/${userId}`, {
      method: 'DELETE',
    });
  }

  return {
    createUser,
    deleteUser,
  }
}
```
{: file="isp-hooks.ts" .code-limit }

Entonces el reporte ahora va a utilizar solamente el hook `useGetUsers()` para
traer la lista, mientras que el formulario utilizará `useManageUsers()` para
crear un usuario.

Apliquemos esta misma idea pero ahora a las props (propiedades) de un
componente considerándolas como su interfaz. Tenemos una lista de usuarios
donde ahora se nos pide mostrar una imagen de perfil para cada uno de ellos.

Entonecs creamos un componente donde inicialmente decidimos pasarle un objeto
del tipo `User` que tiene varias propiedades, entre ellas `profileThumbnail` que
tiene la URL de la imagen:

```typescript
type ThumbnailProps = {
  user: User;
}

export function Thumbnail({
  user,
}: ThumbnailProps) {
  return <img src={user.profileThumbnail} />
}
```
{: file="thumbnail.tsx" }

Como vemos, le estoy pasando más información de la que el componente necesita
ya que `User` tiene otras propiedades:

```typescript
export type User = {
  id: number;
  name: string;
  username: string;
  email: string;
  company: Company;
  address: Address;
  phone: string;
  website: string;
  profileThumbnail: string;
}
```
{: file="user.ts" }

Esto trae un problema cuando trabajamos con una lista de compañías donde
también necesitamos mostrar una imagen, ya no podemos utilizar el componente 
`Thumbnail` porque estamos trabajando con otro tipo de datos, `Company`:

```typescript
export type Company = {
  name: string;
  catchPhrase: string;
  bs: string;
  logoThumbnail: string;
}

export function CompanyList() {
  const { companies, isLoadingCompanies } = useGetCompanies();

  return isLoadingCompanies ?
    <>Loading Companies</> :
    companies.map((company) => (
    <div>
      <div>Company Name: {company.name}</div>
      {/*
        No podemos usar Thumbnail, porque company no es compatible con user
        <Thumbnail user={company} />
      */}
    </div>
    ))
}
```
{: file="company-list.tsx" }

Aplicando el principio de segregación de la interfaz, el componente `Thumbnail`
que antes recibía un usuario ahora solamente recibe `imageUrl`, ya que por ahora
lo único que necesita el componente es una URL para cargar la imagen:

```tsx
type ThumbnailProps = {
  imageUrl: string;
}

export function Thumbnail({
  imageUrl,
}: ThumbnailProps) {
  return <img src={imageUrl} />
}
```

Como hemos simplificado la interfaz -los props que recibe el componente-
entonces podemos reutilizarlo tanto en la lista de usuarios como en la lista de
compañías.

Podríamos resumir esta aplicación del principio de ISP como: un componente
solamente debería depender de las props que verdaderamente necesita.

## Dependency-Inversion Principle (DIP)

El principio de inversión de dependencias dice que tenemos que
**depender de una abstracción y no de una implementación**.

Primero veamos cómo se aplica en OOP. En el siguiente diagrama tenemos una clase `PhotoService` que tiene una dependencia: `PhotoRepository`, que se
encarga de las llamadas a la base de datos.

![Alto acoplamiento con dependencia directa](dip-01.png){: w="339" h="394" }

Como se está dependiendo directamente de una implementación, existe un alto
[acoplamiento (*coupling*)](https://es.wikipedia.org/wiki/Acoplamiento_(inform%C3%A1tica)){:target="_blank"}
entre ambas clases. Esto pudiera traernos problemas en le futuro si tenemos
que cambiar `PhotoRepository` (por ejemplo si ahora tiene que comunicarse con
una base de datos diferente).

Aplicando el principio de inversión de dependencias, creamos una absracción:
en este caso una interfaz `IPhotoRepository` que define un contrato.

![Bajo acoplamiento mediante una interfaz](dip-02.png){: w="339" h="606" }

Ahora `PhotoService` depende de esa interfaz y no de la implementación. Por su
parte, `PhotoRepository` implementa ese contrato. Ahora el sistema es más
flexible porque cuando necesite cambiar `PhotoRepository`, mientras resepete
la interfaz (que sirve de contrato) no tendría que haber problemas de
compatibilidad.

El código se vería algo así en un proyecto de NestJS que cuenta con un sistema
de [*dependency injection* (inyección de dependencias)](https://docs.nestjs.com/providers#dependency-injection){:target="_blank"}
:

```typescript
export interface IPhotoRepository {
  findAll(): Promise<Photo[]>;
}
```
{: file="iphoto.repository.ts" }

```typescript
import { Inject } from '@nestjs/common':

export class PhotoService {
  constructor(
    @Inject('photoRepository')
    private readonly photoRepository: IPhotoRepository,
  ) {
  }

  public async findAll(): Promise<Photo[]> {
    return await this.photoRepository.findAll();
  }
}
```
{: file="photo.service.ts" }

`PhotoService` depende de la interfaz (una abstracción) y no de una
implementación. Con el decorador `@Inject` al cual se le pasa el token
`photoRepository` le estamos diciendo a NestJS se encargue de inyectar la
dependencia. ¿Cómo sabe NestJS qué implementación utilizar? Al configurar el
módulo se especifican los *providers* (proveedores) donde se asocia el
*dependency injection token* `photoRepository` con una clase que implementa
la interfaz:

```typescript
@Module({
  providers: [
    {
      provide: 'photoRepository',
      useClass: PhotoRepository,
    }
  ],
})
export class PhotoModule {}
```
{: file="photo.module.ts" }

Otro beneficio de aplicar este principio es que los unit tests son más fáciles,
ya que se puede inyectar fácilmente una clase *mock* en el lugar de la
dependencia.

### DIP en React

Aunque no se utiliza a menudo en el Frontend, vamos a ver cómo se podría aplicar
la idea. Volviendo a nuestro componente de reporte de usuarios, ahora está
trayendo la lista de usuarios de un hook pero algo cambió:

```tsx
import { UserService } from "../service";

export function UsersReport() {
  const { users, isLoadingUsers } = UserService.useGetUsers();

  return isLoadingUsers ?
    <>Loading Users</> :
    users.map((user) => (<div>{user.name}</div>))
}
```
{: file="users-report.tsx" }

¿Notaste algo diferente? Estamos utilizando un servicio, `UserService`. Esto no
es muy común en el mundo del Frontend, lo que estamos haciendo es agrupar
funciones relacionadas en un objeto como una manera de organizar el código:

```typescript
export const UserService = {
  useGetUsers,
  useManageUsers,
}

function useGetUsers() {
  ...

  return {
    users,
    isLoadingUsers,
  }
}

function useManageUsers() {
  ...

  return {
    createUser,
    deleteUser,
  }
}
```
{: file="user.service.ts" }

Como vemos los hooks siguen siendo funciones individuales, pero se exportan
por medio de un objeto. De esa manera cuando se consume el hook, se lo llama
mediante el objeto `UserService.useGetUsers()`, comunicando el contexto al cual
pertenece el hook. No es que necesitemos agrupar el código de esta manera para
aplicar DIP, pero nos ayuda en la comparación con el código que mostramos que
está basado en OOP (donde se usan clases para agrupar métodos relacionados).

Ahora veamos el código de `useGetUsers()`. Trata de identificar algo nuevo:

```typescript
function useGetUsers() {
  const { getAll }: IUserRepository = useUserRepository();
  const [ users, setUsers ] = useState<User[]>([]);
  const [ isLoadingUsers, setIsLoadingUsers ] = useState<boolean>(true);

  // Load users from API
  useEffect(() => {
    async function loadUsers() {
      const data = await getAll();
      setUsers(data);
      setIsLoadingUsers(false);
    }

    loadUsers();
  }, [getAll]);

  return {
    users,
    isLoadingUsers,
  }
}
```
{: file="user.service.ts" }

Estamos llamando a `useUserRepository()` que está devolviendo la implementación
de una interfaz `IUserRepository` que define un contrato de funciones:

```typescript
export interface IUserRepository {
  getAll(): Promise<User[]>;
  create(user: Partial<User>): Promise<void>;
  update(user: Partial<User>): Promise<void>;
  remove(userId: string): Promise<void>;
}
```
{: file="user.repository.ts" }

De esta manera estamos dependiendo de una interfaz y no de la implementación.
¿Pero cómo podemos proveer la implementación en React? Una manera de hacerlo
es utilizando Context. Originalmente se pensó para compartir estado dentro de
un árbol de componentes y [así evitar la perforación de props](https://es.react.dev/learn/passing-data-deeply-with-context){:target="_blank"}.
Pero aquí vemos como usar Context para inyección de dependencias. Primeramente
necesitamos un context que hace referencia a la interfaz:

```tsx
import { createContext } from "react";

export const UserRepositoryContext = createContext<IUserRepository | null>(null);
```
{: file="user-repository.provider.tsx" }

También necesitamos un Provider, donde asociamos la interfaz con la
implementación (en este caso `UserFetchRepository`):

```tsx
type UserRepositoryProviderProps = {
  children: React.ReactNode;
};

export function UserRepositoryProvider({
  children,
}: UserRepositoryProviderProps) {

  const contextValue: IUserRepository = new UserFetchRepository();

  return (
    <UserRepositoryContext.Provider value={contextValue}>
      {children}
    </UserRepositoryContext.Provider>
  );
}
```
{: file="user-repository.provider.tsx" }

Finalmente agregamos un hook que hace disponible el context que contiene la
dependencia `IUserRepository`:

```tsx
export function useUserRepository() {
  const context = useContext(UserRepositoryContext);
  if (!context) {
    throw new Error(`useDependencies must be used within UserRepositoryProvider`);
  }
  return context;
}
```
{: file="user-repository.provider.tsx" }

Para que esto funcione tenemos que asegurarnos de utilizar el provider en el
renderizado de la app, antes de renderizar el componente de reporte (que utiliza
el servicio que a su vez hace uso de la dependencia):

```tsx
export function App() {
  return <UserRepositoryProvider>
    <UsersReport />
    <UserCreationForm />
  </UserRepositoryProvider>
}
```
{: file="user-repository.provider.tsx" }

El punto importante es que estamos dependiendo de la interfaz `IUserRepository`
(una abstracción) y no de una implementación específica:

```typescript
function useGetUsers() {
  const { getAll }: IUserRepository = useUserRepository();
```

Volviendo al Provider, que es donde estamos proveyendo la implementación,
podemos cambiarla por otra, en este caso utilizando `UserAxiosRepository` en
vez de `UserFetchRepository`:

```typescript
export function UserRepositoryProvider({
  children,
}: UserRepositoryProviderProps) {

  const contextValue: IUserRepository = new UserAxiosRepository();
```

Para referencia, esta es la clase de implementación `UserAxiosRepository()`:

```typescript
import axios from "axios";
import { User } from "../../../types";
import { IUserRepository } from "../interface";

export class UserAxiosRepository implements IUserRepository {
  public async getAll(): Promise<User[]> {
    return axios.get('/api/user');
  }
  
  public async create(user: Partial<User>): Promise<void> {
    return axios.post('/api/user', user);
  }
  
  public async update(user: Partial<User>): Promise<void> {
    return axios.put('/api/user', user);
  }
  
  public async remove(userId: string): Promise<void> {
    await axios.delete(`/api/user/${userId}`);
  }
  
}
```
{: file="user-axios.repository.ts" }

Muchas librerías javascript utilizan clases. Esta es una manera de inyectar una
instancia global que puede accederse desde cualquier hook.
Ahora si no queremos usar una clase también podemos utilizar un objeto con
funciones como habíamos visto anteriormente, en este caso para
`UserFetchRepository`:

```tsx
import { User } from "../../../types";
import { IUserRepository } from "../interface";

export const UserFetchRepository: IUserRepository = {
  getAll,
  create,
  update,
  remove,
}

async function getAll(): Promise<User[]> {
  const response = await fetch('/api/user');
  return await response.json();
}

async function create(user: Partial<User>): Promise<void> {
  await fetch('/api/user', {
    method: 'POST',
    body: JSON.stringify(user),
  });
}

async function update(user: Partial<User>): Promise<void> {
  await fetch('/api/user', {
    method: 'PUT',
    body: JSON.stringify(user),
  });
}

async function remove(userId: string): Promise<void> {
  await fetch(`/api/user/${userId}`, {
    method: 'DELETE',
  });
}
```
{: file="user-fetch.repository.ts" }

La diferencia es que al inyectar la dependencia no tenemos que crear una
instancia de una clase sino directamente hacer referencia al objeto:

```typescript
export function UserRepositoryProvider({
  children,
}: UserRepositoryProviderProps) {

  const contextValue: IUserRepository = UserFetchRepository;
```
{: file="user-repository.provider.tsx" }

Sin importar cuál implementación estamos utilizando, los hooks de `UserService`
no cambian. Esto resulta beneficioso cuando utilizamos una librería que pudiera
cambiar en el futuro (ya sea una nueva versión de la misma u otra librería
diferente). Como tenemos bajo acoplamiento, solamente necesitamos cambiar una
parte del código y el resto (protegido mediante la interfaz) no cambia.

Ahora veamos una manera más sencilla de aplicar el principio de inversión de
dependencias. Tenemos un formulario de creación o actualización de usuario.
Cuando se hace *submit* del formulario, se llama a `handleSubmit()` que decide
si se trata de crear o actualizar dependiendo de si ya existe el usuario que
se recibe como prop opcional en `UserForm`: 

```tsx
type UserFormProps = {
  user?: User;
}

export function UserForm({
  user,
}: UserFormProps) {
  const { createUser, updateUser } = useManageUsers();

  const [name, setName] = useState<string>(user ? user.name : '');

  function handleNameChange(event: React.ChangeEvent<HTMLInputElement>) {
    setName(event.target.value);
  }

  function handleSubmit(event: React.FormEvent) {
    event.preventDefault();
    if (!name) {
      alert('User name is not valid');
      return;
    }

    if (user) {
      const updatedUser = {
        ...user,
        name,
      };
      updateUser(updatedUser);
    } else {
      createUser(name);
    }
  
  }

  return <form onSubmit={handleSubmit}>
    <label>
      Name:
      <input type="text" value={name} onChange={handleNameChange} />
    </label>
    <button type="submit">Create</button>
  </form>
}
```
{: file="user-form.tsx" }

Comparemos con esta otra implementación. El formulario ahora recibe como prop
una función `onSubmit`:

```tsx
type UserFormProps = {
  user?: User;
  onSubmit: (user: Partial<User>) => Promise<void>;
}

export function UserForm({
  user,
  onSubmit,
}: UserFormProps) {
  const [name, setName] = useState<string>(user ? user.name : '');

  function handleNameChange(event: React.ChangeEvent<HTMLInputElement>) {
    setName(event.target.value);
  }

  function handleSubmit(event: React.FormEvent) {
    event.preventDefault();
    if (!name) {
      alert('User name is not valid');
      return;
    }

    const updatedUser: Partial<User> = {
      ...user,
      name: name
    };

    onSubmit(updatedUser);
  }

  return <form onSubmit={handleSubmit}>
    <label>
      Name:
      <input type="text" value={name} onChange={handleNameChange} />
    </label>
    <button type="submit">Create</button>
  </form>
}
```
{: file="user-form.tsx" }

De esta manera se **invierte el control** al padre,
quien define qué hacer una vez que el formulario está listo para enviar la
información al backend. `UserCreate` creará al usuario:

```tsx
export function UserCreate() {
  const { createUser } = useManageUsers();

  async function handleSubmit(user: Partial<User>) {
    if (!user.name) {
      return;
    }

    await createUser(user.name);
  }

  return <UserForm onSubmit={handleSubmit} />
}
```
{: file="user-create.tsx" }

Mientras que `UserUpdate` actualizará al usuario:

```tsx
export function UserUpdate() {
  const { updateUser } = useManageUsers();

  async function handleSubmit(user: Partial<User>) {
    if (!user.name) {
      return;
    }

    await updateUser(user);
  }

  return <UserForm onSubmit={handleSubmit} />
}
```
{: file="user-update.tsx" }

## Conclusión

Los principios SOLID no solucionan todos los problemas de diseño del código.
No se debería forzar su aplicación, sino que es mejor utilizarlos cuando hay
una razón. Si después de aplicarlos el código es más entendible y se facilita
su mantenimiento (soporta los cambios), entonces lograron su objetivo. Pero si
el código se vuelve demasiado complicado sin una razón, si no aportan ningún
beneficio, entonces se está forzando los principios SOLID y no valen la pena.

En la programación siempre hay varias maneras de hacer las cosas bien. Estos
principios están basados primeramente en la Programación Orientada a Objetos,
pero las ideas se pueden aplicar a la programación funcional y a React, incluso
en el Frontend.
