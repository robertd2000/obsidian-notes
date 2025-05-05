```ts

import React, { memo, useCallback, useState, useMemo } from "react";

  

const URL = "https://jsonplaceholder.typicode.com/users";

  

type Company = {

bs: string;

catchPhrase: string;

name: string;

};

  

type User = {

id: number;

email: string;

name: string;

phone: string;

username: string;

website: string;

company: Company;

address: any;

};

  

interface IButtonProps {

onClick: any;

isDisabled: boolean;

children: React.ReactNode;

}

  

function Button({ onClick, isDisabled, children }: IButtonProps): JSX.Element {

console.log("Btn");

return (

<button type="button" disabled={isDisabled} onClick={onClick}>

{children}

</button>

);

}

  

const ButtonMemo = memo(Button);

  

interface IUserInfoProps {

user: User;

}

  

function UserInfo({ user }: IUserInfoProps): JSX.Element {

return (

<table>

<thead>

<tr>

<th>Username</th>

<th>Phone number</th>

</tr>

</thead>

<tbody>

<tr>

<td>{user.name}</td>

<td>{user.phone}</td>

</tr>

</tbody>

</table>

);

}

  

function App(): JSX.Element {

const [item, setItem] = useState<Record<number, User>>({});

const [currentId, setCurrentId] = useState<number>(0);

  

const [counter, setCounter] = useState(0);

  

const receiveRandomUser = async () => {

const id = Math.floor(Math.random() * (10 - 1)) + 1;

setCurrentId(id);

  

if (!item.hasOwnProperty("id")) {

const response = await fetch(`${URL}/${id}`);

const _user = (await response.json()) as User;

  

setItem((prev) => ({

...prev,

[id]: _user,

}));

}

};

  

const handleButtonClick = useCallback(

(event: React.MouseEvent<HTMLButtonElement, MouseEvent>) => {

event.stopPropagation();

  

receiveRandomUser();

setCounter((count) => {

return count + 1;

});

},

[]

);

  

const isDisabled = counter >= 10;

  

const memoized = useMemo(() => {

return <span>get random user</span>;

}, []);

  

return (

<div>

<header>Get a random user. Api request used {counter} from 10</header>

  

<ButtonMemo onClick={handleButtonClick} isDisabled={isDisabled}>

{memoized}

</ButtonMemo>

  

{item[currentId] && <UserInfo user={item[currentId]} />}

</div>

);

}

  

export default App;
import React, { memo, useCallback, useState, useMemo } from "react";

  

const URL = "https://jsonplaceholder.typicode.com/users";

  

type Company = {

bs: string;

catchPhrase: string;

name: string;

};

  

type User = {

id: number;

email: string;

name: string;

phone: string;

username: string;

website: string;

company: Company;

address: any;

};

  

interface IButtonProps {

onClick: any;

isDisabled: boolean;

children: React.ReactNode;

}

  

function Button({ onClick, isDisabled, children }: IButtonProps): JSX.Element {

console.log("Btn");

return (

<button type="button" disabled={isDisabled} onClick={onClick}>

{children}

</button>

);

}

  

const ButtonMemo = memo(Button);

  

interface IUserInfoProps {

user: User;

}

  

function UserInfo({ user }: IUserInfoProps): JSX.Element {

return (

<table>

<thead>

<tr>

<th>Username</th>

<th>Phone number</th>

</tr>

</thead>

<tbody>

<tr>

<td>{user.name}</td>

<td>{user.phone}</td>

</tr>

</tbody>

</table>

);

}

  

function App(): JSX.Element {

const [item, setItem] = useState<Record<number, User>>({});

const [currentId, setCurrentId] = useState<number>(0);

  

const [counter, setCounter] = useState(0);

  

const receiveRandomUser = async () => {

const id = Math.floor(Math.random() * (10 - 1)) + 1;

setCurrentId(id);

  

if (!item.hasOwnProperty("id")) {

const response = await fetch(`${URL}/${id}`);

const _user = (await response.json()) as User;

  

setItem((prev) => ({

...prev,

[id]: _user,

}));

}

};

  

const handleButtonClick = useCallback(

(event: React.MouseEvent<HTMLButtonElement, MouseEvent>) => {

event.stopPropagation();

  

receiveRandomUser();

setCounter((count) => {

return count + 1;

});

},

[]

);

  

const isDisabled = counter >= 10;

  

const memoized = useMemo(() => {

return <span>get random user</span>;

}, []);

  

return (

<div>

<header>Get a random user. Api request used {counter} from 10</header>

  

<ButtonMemo onClick={handleButtonClick} isDisabled={isDisabled}>

{memoized}

</ButtonMemo>

  

{item[currentId] && <UserInfo user={item[currentId]} />}

</div>

);

}

  

export default App;
```
