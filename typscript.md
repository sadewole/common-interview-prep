# TypeScript Interview Guide (Beginner → Senior)

---

# Beginner

## 1. What is TypeScript and why use it instead of JavaScript?

### Answer

TypeScript is an open-source programming language developed by Microsoft that is a **superset of JavaScript**. Every valid JavaScript program is valid TypeScript, but TypeScript adds **static typing** and extra language features.

TypeScript code is compiled (transpiled) into plain JavaScript before running.

```ts
function greet(name: string) {
    return `Hello ${name}`;
}
```

Without TypeScript:

```js
greet(10); // No compile error
```

With TypeScript:

```
Argument of type 'number' is not assignable to parameter of type 'string'.
```

### Advantages

* Catch errors before runtime
* Better IntelliSense
* Easier refactoring
* Self-documenting code
* Better maintainability
* Excellent IDE support

### Interview Follow-up

**Why not just use JavaScript?**

JavaScript detects errors at runtime.

TypeScript detects many errors during development.

For large teams and enterprise applications, TypeScript greatly reduces bugs.

---

# 2. Difference between interface and type?

They are both used to describe shapes.

### Interface

```ts
interface User {
    id: number;
    name: string;
}
```

### Type

```ts
type User = {
    id: number;
    name: string;
};
```

### Major Differences

### Interface supports declaration merging

```ts
interface User {
    name: string;
}

interface User {
    age: number;
}

const user = {
    name: "Sam",
    age: 30
};
```

Works.

Type cannot.

---

### Type can represent more things

```ts
type Status = "success" | "failed";

type ID = string | number;

type Point = [number, number];
```

Interfaces cannot represent unions or tuples.

---

### Which should you use?

General rule:

* Interface → Objects/classes/public APIs
* Type → Unions, primitives, utility types

Many companies use interfaces for models and types everywhere else.

---

# 3. Difference between any, unknown and never?

## any

Turns off TypeScript.

```ts
let value: any;

value.foo.bar.test();
```

No compiler errors.

Dangerous.

---

## unknown

Safer version of any.

```ts
let value: unknown;

value.toUpperCase(); // Error
```

Need type checking first.

```ts
if (typeof value === "string") {
    value.toUpperCase();
}
```

---

## never

Represents values that never happen.

Example:

```ts
function throwError(): never {
    throw new Error();
}
```

Also used for exhaustive checking.

```ts
switch(status){

case "loading":
break;

case "success":
break;

default:
const x: never = status;
}
```

---

Interview answer:

* any = disable checking
* unknown = must check first
* never = impossible value

---

# 4. What are Generics?

Generics let you write reusable code.

Without generics:

```ts
function identity(value: string){
    return value;
}
```

Only works for strings.

Generic:

```ts
function identity<T>(value:T):T{
    return value;
}
```

Now works for everything.

```ts
identity(10)

identity("hello")

identity(true)
```

Another example:

```ts
interface ApiResponse<T>{

data:T;

success:boolean;

}
```

```ts
ApiResponse<User>

ApiResponse<Product>

ApiResponse<Order>
```

One interface.

Many uses.

---

# 5. Explain Enums

Enums represent fixed constants.

```ts
enum Role{

ADMIN,

USER,

EDITOR

}
```

Usage

```ts
const role = Role.ADMIN;
```

String enums

```ts
enum Status{

SUCCESS="SUCCESS",

FAILED="FAILED"

}
```

Interview tip:

Most modern TypeScript projects prefer string literal unions.

```ts
type Status =

"SUCCESS"

|

"FAILED";
```

Less generated JavaScript.

---

# 6. What are Union and Intersection Types?

## Union

Means OR.

```ts
type ID =

string

|

number;
```

Can be either.

---

## Intersection

Means BOTH.

```ts
type Person={

name:string;

}

type Employee={

salary:number;

}

type Staff=

Person & Employee;
```

Result

```ts
{

name,

salary

}
```

---

Interview memory:

Union = OR

Intersection = AND

---

# 7. Difference between Optional and Nullable?

Optional

May not exist.

```ts
interface User{

name:string;

age?:number;

}
```

Valid

```ts
{

name:"Sam"

}
```

---

Nullable

Property exists.

Value can be null.

```ts
interface User{

age:number|null;

}
```

Valid

```ts
{

age:null

}
```

---

Difference

Optional

```
undefined
```

Nullable

```
null
```

---

# Intermediate

# 8. How does TypeScript support Dependency Injection?

TypeScript itself doesn't implement Dependency Injection.

Frameworks like NestJS use:

* decorators
* metadata
* interfaces/classes

Example

```ts
@Injectable()

class UserService{

}
```

Constructor Injection

```ts
constructor(

private userService:UserService

){}
```

Benefits

* loose coupling
* easier testing
* mock dependencies
* maintainability

---

# 9. Explain Mapped Types

Mapped types transform existing types.

Example

```ts
type User={

name:string;

age:number;

}
```

Mapped

```ts
type ReadonlyUser={

readonly [K in keyof User]:

User[K];

}
```

Every property becomes readonly.

---

# 10. Utility Types

## Partial

Everything optional

```ts
Partial<User>
```

Equivalent

```ts
{

name?;

age?;

}
```

Useful for update APIs.

---

## Required

Everything required.

---

## Pick

Choose fields.

```ts
Pick<User,

"name"|"age">
```

---

## Omit

Remove fields.

```ts
Omit<User,

"password">
```

---

## Record

Creates object types.

```ts
Record<string,

User>
```

Example

```ts
{

admin:User,

manager:User

}
```

---

# 11. Conditional Types

Like if statements.

```ts
type IsString<T>

=

T extends string

?

true

:

false;
```

Example

```ts
IsString<string>

→ true

IsString<number>

→ false
```

Used heavily in libraries.

---

# 12. What is Declaration Merging?

Only interfaces.

```ts
interface User{

name:string;

}
```

Later

```ts
interface User{

age:number;

}
```

Merged

```ts
{

name,

age

}
```

Useful for extending third-party libraries.

---

# 13. Explain Type Narrowing

TypeScript narrows types after checks.

```ts
function print(

value:string|number

){

if(typeof value==="string"){

value.toUpperCase();

}

}
```

Compiler now knows it's a string.

Methods

* typeof
* instanceof
* in operator
* custom guards

---

# 14. What are Discriminated Unions?

Powerful way to model state.

```ts
type Loading={

status:"loading";

}

type Success={

status:"success";

data:string;

}

type Error={

status:"error";

message:string;

}
```

Union

```ts
type Response=

Loading|

Success|

Error;
```

Switch

```ts
switch(response.status){

case "success":

response.data

}
```

TypeScript automatically knows the type.

Excellent for API states and reducers.

---

# Senior

# 15. How would you design a reusable SDK using TypeScript?

A good SDK should have:

* strict typing
* generic API responses
* modular architecture
* typed errors
* configuration object
* request interceptors
* authentication support

Example

```ts
client.users.getById()

client.orders.create()

client.auth.login()
```

Avoid exposing raw fetch() everywhere.

Provide typed abstractions.

---

# 16. How do you enforce strict typing across frontend and backend?

This is common in full-stack TypeScript projects.

Approaches include:

* Share types in a common package within a monorepo.
* Generate client types from an OpenAPI specification.
* Use end-to-end type-safe solutions like tRPC where appropriate.
* Validate runtime data with libraries such as Zod, then infer TypeScript types from the schemas.

Example:

```ts
// shared package
export interface User {
  id: string;
  name: string;
}
```

Both frontend and backend import the same type instead of duplicating it.

---

# 17. How do you type large REST responses?

Don't create one giant interface.

Break responses into reusable pieces.

```ts
interface User {}

interface Address {}

interface Order {}

interface Payment {}
```

Compose them:

```ts
interface UserResponse {
  user: User;
  orders: Order[];
  address: Address;
}
```

For APIs with optional expansions:

```ts
interface ApiResponse<T> {
  data: T;
  meta: Meta;
}
```

Use generics and nested interfaces instead of repeating structures.

---

# 18. How do you organize types in a monorepo?

A common structure is:

```text
packages/
  shared/
    src/
      types/
      schemas/
      constants/
      utils/
apps/
  web/
  api/
  mobile/
```

Keep domain models (e.g., `User`, `Order`) in the shared package, while app-specific types stay within each application. Avoid circular dependencies and version shared packages carefully if they're published independently.

---

# 19. How would you migrate a million-line JavaScript codebase to TypeScript?

A gradual migration is the safest approach:

1. Enable TypeScript with `allowJs` so JavaScript and TypeScript can coexist.
2. Turn on `checkJs` to surface type errors in existing JavaScript.
3. Start with new features and core/shared modules.
4. Add type definitions incrementally, avoiding `any` where possible.
5. Enable stricter compiler options over time (`strict`, `noImplicitAny`, etc.).
6. Use ESLint and CI to prevent regressions.
7. Share common types and provide training/documentation for the team.

The goal isn't to convert everything overnight. Large organizations migrate incrementally while continuing to ship features.

---

## Senior Interview Tip

When answering senior TypeScript questions, don't stop at explaining the language feature. Explain **how you've used it in production**. For example:

* Shared types between React and a Node/NestJS backend.
* Generic API response wrappers for consistent REST responses.
* `Partial` for update DTOs and `Pick`/`Omit` for shaping API payloads.
* Zod or similar libraries for runtime validation alongside static typing.
* A shared types package in a monorepo to keep web, mobile, and backend in sync.

Connecting TypeScript concepts to real systems you've built is often what distinguishes a senior engineer from someone who has only studied the language.
