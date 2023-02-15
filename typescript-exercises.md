# Typescript Exercises

## 1

```ts
export type user = unknown;

export const users: unknown[] = [
  {
    name: "max mustermann",
    age: 25,
    occupation: "chimney sweep",
  },
  {
    name: "kate müller",
    age: 23,
    occupation: "astronaut",
  },
];

export function logperson(user: unknown) {
  console.log(` - ${user.name}, ${user.age}`);
}

console.log("users:");
users.foreach(logperson);
```

1번 문제의 경우 `User`의 타입만 작성하면 되는 간단한 문제이다<br />
아래 `users`를 보면 `name`, `age`, `occupation`으로 구성된걸로 보아 타입도 이렇게 선언해 주면 될 것이다

```ts
export type User = {
  name: string;
  age: number;
  occupation: string;
};

export const users: User[] = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
];

export function logPerson(user: User) {
  console.log(` - ${user.name}, ${user.age}`);
}

console.log("Users:");
users.forEach(logPerson);
```

## 2

```ts
interface User {
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  name: string;
  age: number;
  role: string;
}

export type Person = unknown;

export const persons: User[] /* <- Person[] */ = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Jane Doe",
    age: 32,
    role: "Administrator",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
  {
    name: "Bruce Willis",
    age: 64,
    role: "World saver",
  },
];
```

2번 문제의 부분만 잘라서 가져왔다<br />
`persons` 변수 주변의 주석을 보면 `Person[]`로 바꿔달라고 하는 것 같다.<br />
하지만 `Person` 타입의 경우 `unknown`이라 정확한 타입을 명시해 줘야 하는데, `persons`의 값을 보면 `Admin` 타입과 `User` 타입이 번갈아 가며 사용되는 걸 볼 수 있다.<br />
따라서 Person의 타입은 `Admin | User` 가 된다

```ts
interface User {
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  name: string;
  age: number;
  role: string;
}

export type Person = User | Admin;

export const persons: Person[] /* <- Person[] */ = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Jane Doe",
    age: 32,
    role: "Administrator",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
  {
    name: "Bruce Willis",
    age: 64,
    role: "World saver",
  },
];

export function logPerson(user: Person) {
  console.log(` - ${user.name}, ${user.age}`);
}

persons.forEach(logPerson);
```

## 3

```ts
export function logPerson(person: Person) {
  let additionalInformation: string;
  if (person.role) {
    additionalInformation = person.role;
  } else {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}
```

`role`의 경우 `Admin`타입에만 존제하는 것이기 때문에 role을 조회하게 되면 에러가 발생하게 된다<br />
`person`은 role값이 있을 수 있기도 하고 없을 수도 있기 때문에 타입 가드를 통해 role값을 검증해 줘야 한다<br />
검증은 js의 [in 연산자](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/in)를 통해 쉽게 할 수 있다

```ts
export function logPerson(person: Person) {
  let additionalInformation: string;
  if ("role" in person) {
    additionalInformation = person.role;
  } else {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}
```

## 4

```ts
export function isAdmin(person: Person) {
  return person.type === "admin";
}

export function isUser(person: Person) {
  return person.type === "user";
}

export function logPerson(person: Person) {
  let additionalInformation: string = "";
  if (isAdmin(person)) {
    additionalInformation = person.role;
  }
  if (isUser(person)) {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}
```

`logPerson`함수에서 `isAdmin`과 `isUser`로 타입을 검증했지만 typescript는 이걸 믿지 못 하는 상황이다<br />
때문에 저 타입을 검증하는 함수에서 person값을 특정 타입으로 좁혀 줘야한다<br />
여기서 typescript의 [is 연산자](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)를 통해 타입을 좁힐 수 있다

```ts
export function isAdmin(person: Person): person is Admin {
  return person.type === "admin";
}

export function isUser(person: Person): person is User {
  return person.type === "user";
}
```

## 5

우선 `test.ts`를 보자

```ts
typeAssert<
  IsTypeAssignable<SecondArgument<typeof filterUsers>, { name: string }>
>();
typeAssert<
  IsTypeAssignable<SecondArgument<typeof filterUsers>, { age: number }>
>();
typeAssert<
  IsTypeAssignable<
    SecondArgument<typeof filterUsers>,
    { name: string; age: number }
  >
>();
typeAssert<
  IsTypeAssignable<SecondArgument<typeof filterUsers>, { occupation: string }>
>();
typeAssert<
  IsTypeAssignable<
    SecondArgument<typeof filterUsers>,
    { name: string; age: number; occupation: string }
  >
>();
```

`filterUsers`의 두 번째 매개변수로 User 타입의 몇 가지 property가 없이 들어오는 것을 볼 수 있다

```ts
export function filterUsers(persons: Person[], criteria: User): User[] {}
```

하지만 `filterUsers`함수는 두 번째 매개변수로 그냥 `User` 순정(?) 타입을 받고 있다<br />
이 경우 User의 몇가지 property가 undefined값으로 들어와야 하게 함으로 typescript의 유틸 타입인 [Partial](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype)을 통해 criteria 타입을 변경해 준다.

```ts
export function filterUsers(
  persons: Person[],
  criteria: Partial<User>
): User[] {}
```

진짜 솔루션은 Omit을 통해 `type` property를 제거 하고 Partial로 옵셔널 처리를 해준다

```ts
export function filterUsers(
  persons: Person[],
  criteria: Partial<Omit<User, "type">>
): User[] {}
```

## 6

이번 문제의 경우 정답을 보긴 했지만 타입스크립트에 함수 오버라이드라는 게 있다는 것을 알게 되어서 좋긴 했다

```ts
export function logPerson(person: Person) {
  console.log(
    ` - ${person.name}, ${person.age}, ${
      person.type === "admin" ? person.role : person.occupation
    }`
  );
}

export function filterPersons(
  persons: Person[],
  personType: string,
  criteria: unknown
): unknown[] {
  return persons
    .filter((person) => person.type === personType)
    .filter((person) => {
      let criteriaKeys = Object.keys(criteria) as (keyof Person)[];
      return criteriaKeys.every((fieldName) => {
        return person[fieldName] === criteria[fieldName];
      });
    });
}

export const usersOfAge23 = filterPersons(persons, "user", { age: 23 });
export const adminsOfAge23 = filterPersons(persons, "admin", { age: 23 });

console.log("Users of age 23:");
usersOfAge23.forEach(logPerson);

console.log();

console.log("Admins of age 23:");
adminsOfAge23.forEach(logPerson);
```

우선 test.ts에서 `criteria`값에 Admin과 User 타입을 넣어주고 있기 때문에 매개변수 타입을 Partial<Person>으로 변경해 준다<br />
이번엔 아래 `filterPersons`의 return 값을 사용하는 걸 보면 `Person`타입을 원하는 걸 볼 수 있다. 때문에 `filterPersons`의 반환 타입은 `Person[]`가 된다<br />
마지막으로 `test.ts`에서 `filterPerson`의 두 번째 값에 `user`가 들어오면 `User`타입의 값이 반환이 되고 `admin`값이 들어오면 `Admin` 타입의 값이 반환 되는 걸 원한다<br />
때문에 함수 오버라이드를 통해 test.ts의 요구사항에 맞게 코드를 작성해 주면 된다

```ts
export function filterPersons(
  persons: Person[],
  personType: "admin",
  criteria: Partial<Person>
): Admin[];
export function filterPersons(
  persons: Person[],
  personType: "user",
  criteria: Partial<Person>
): User[];
export function filterPersons(
  persons: Person[],
  personType: string,
  criteria: Partial<Person>
): Person[] {
  return persons
    .filter((person) => person.type === personType)
    .filter((person) => {
      let criteriaKeys = Object.keys(criteria) as (keyof Person)[];
      return criteriaKeys.every((fieldName) => {
        return person[fieldName] === criteria[fieldName];
      });
    });
}
```

## 7

```ts
export function swap(v1, v2) {
  return [v2, v1];
}
```

이 함수에 매개변수로 number, string, boolean 등등 다양한 값이 들어온다<br />
나는 여기서 `v1`과 `v2`에 `any` 타입을 주고 반환 타입에는 `[any, any]`를 줘서 해결했다 ㅎㅎ

```ts
export function swap(v1: any, v2: any): [any, any] {
  return [v2, v1];
}
```

하지만 진짜 솔루션은 Generics를 활용해서 해결한다<br />
여기서 처음 알게 된 사실은 Generics를 굳이 지정하지 않으면 사용자가 넣은 매개변수의 타입으로 대체가 된다는 것이다<br />
그래서 Generics를 통해 타입 2개를 받고 반환할 떄는 두 타입의 위치를 바꿔주면 쉽게 해결할 수 있다

```ts
export function swap<T1, T2>(v1: T1, v2: T2): [T2, T1] {
  return [v2, v1];
}
```

## 8

```ts
type PowerUser = unknown;

export type Person = User | Admin | PowerUser;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
  {
    type: "powerUser",
    name: "Nikki Stone",
    age: 45,
    role: "Moderator",
    occupation: "Cat groomer",
  },
];
```

이번 문제의 경우 `PowerUser` 타입을 잘 선언해 주면 해결되는 문제인데 얼마나 간지나게 타입을 선언하느냐가 관건이다<br />
하지만 나는 좀 멋이 없게 해결해 버렸다

```ts
type PowerUser = {
  type: "powerUser";
  name: string;
  age: number;
  role: string;
  occupation: string;
};
```

이것보다 간지나게 작성하는 방법은 `PowerUser`가 `User`와 `Admin`타입을 모두 가지고 있기 때문에 `PowerUser`를 `User`와 `Admin`으로 확장시켜주면 된다.
하지만 여기서 문제가 발생한다. `User`와 `Admin`은 `type`이라는 property에 할당되는 값이 다르기 때문에 `type` 부분만 제거해서 확장시켜줘야 한다

```ts
interface PowerUser extends Omit<User, "type">, Omit<Admin, "type"> {
  type: "powerUser";
}
```
