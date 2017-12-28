## Table of Contents

- [Queries and Mutations](#queries-and-mutations)
  - [Fields](#fields)
  - [Arguments](#arguments)
  - [Aliases](#aliases)
  - [Fragments](#fragments)
  - [Operation name](#operation-name)
  - [Variables](#variables)
  - [Directives](#directives)
  - [Mutations](#mutations)
  - [Inline Fragments](#inline-fragments)
- [Schemas and Types](#schemas-and-types)
  - [Object types and fields](#object-types-and-fields)
  - [Arguments](#arguments)
  - [The Query and Mutation types](#the-query-and-mutation-types)
  - [Scalar types](#scalar-types)
  - [Enumeration types](#enumeration-types)
  - [Lists and Non-Null](#lists-and-non-null)
  - [Interfaces](#interfaces)
  - [Union types](#union-types)
  - [Input types](#input-types)
- [Execution](#execution)
  - [Resolver](#resolver)
- [Apollo Best Practice](#apollo-best-practice)

## Queries and Mutations

### Fields

```
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

### Arguments

```
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

```json
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 5.6430448
    }
  }
}
```

### Aliases

> Aliases dùng để đổi tên type cần query để tránh bị trùng

```
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

```json
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

### Fragments

> Fragments dùng để tái sử dụng query

```
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

Fragment được gắn với 1 type cố định. Như trong ví dụ trên, nó được gắn với type Charactor (```on Character```). Vì vậy nó còn được dùng để query theo type. Rõ hơn ở phần [Inline Fragments](#inline-fragments)

### Operation name

Các query ở các phần trước là shortcut. Viết đầy đủ:

```
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}
```

> Operation type: query
>
> Operation name: HeroNameAndFriends

Có các operation type như: query, mutation, subscription

### Variables

```
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}

// pass variable
{
  "episode": "JEDI"
}
```

Variable có thể là: scalars, enums, hoặc input object types

Để require variable, thêm ! vào sau type: ```query HeroNameAndFriends($episode: Episode!)```

Nếu không require, variable có thể khai báo default value: ```query HeroNameAndFriends($episode: Episode = "JEDI")```

### Directives

> Directives are used to dynamically change the structure and shape of our queries using variables

```
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}

// pass variables
{
  "episode": "JEDI",
  "withFriends": false
}
```

Directive có thế dùng cho ```field``` và ```fragment```

GraphQL có 2 built-in directive:

- @include(if: Boolean)
- @skip(if: Boolean

Ngoài ra có có thể khai báo thêm custom directive

### Mutations

> Mutation giống query nhưng dùng để modify data

```
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}

// pass variable
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

Note:

- ```$review``` là  **input object type**
- query được thực hiện song song còn mutation được thực hiện lần lượt để tránh race condition

### Inline Fragments

> Inline Fragments dùng để query dựa trên underlying concreate type của interface hoặc union type

```
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}

// pass variable
{
  "ep": "JEDI"
}
```

Trong ví dụ trên, ```hero``` field trả về ```type Character``` (interface), underlying concreate type có thể là ```Human``` hoặc ```Droid``` phụ thuộc vào ```episode``` truyền vào. Vì vậy cần dùng Inline Fragment để query các field của riêng ```Human``` và ```Droid``` tuỳ thuộc vào underlying concreate type.

Cũng có thể dùng Named Fragment nhưng Inline Fragment ngắn gọn hơn. Chỉ cần dùng Named Fragment khi muốn tái sử dụng

## Schemas and Types

### Object types and fields

```
type Character {
  name: String!
  appearsIn: [Episode]!
}
```

### Arguments

```
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

Note: Argument của GraphQL truyền theo tên, k theo order như JavaScript

### The Query and Mutation types

Hai type đặc biệt:

```
schema {
  query: Query
  mutation: Mutation
}
```

Hai type này giống hệt object type thông thường ngoại trừ nó có thể định nghĩa entry point

Để query được như sau:

```
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

Nghĩa là GraphQL service phải định nghĩa Query type với 2 field ```hero``` và ```droid```

```
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```

### Scalar types

> Scalar types không có sub-fields. They are the leaves of the query

```
{
  hero {
    name
    appearsIn
  }
}
```

-> ```name``` và ```appearsIn``` là scalar types

Các Scalar types của GraphQL:

- Int: A signed 32‐bit integer.
- Float: A signed double-precision floating-point value.
- String: A UTF‐8 character sequence.
- Boolean: true or false.
- ID: The ID scalar type represents a unique identifier.

### Enumeration types

> Enumeration types là 1 dạng scalar types đặc biệt. Nó giới hạn trước 1 set các value có thể sử dụng

```
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

### Lists and Non-Null

> Lists và Non-null là type modifiers

```
type Character {
  name: String!
  appearsIn: [Episode]!
}
```

### Interfaces

> An Interface is an abstract type that includes a certain set of fields that a type must include to implement the interface

```
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

Interface thường được dùng để abstract return type của 1 field

### Union types

> Union types are very similar to interfaces, but they don't get to specify any common fields between the types

```
union SearchResult = Human | Droid | Starship
```

```
{
  search(text: "an") {
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
```

### Input types

> Input types dùng để truyền vào argument là complex object thay vì scalar values. Thường gặp khi sử dụng mutation

```
input ReviewInput {
  stars: Int!
  commentary: String
}
```

```
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}

// pass variable
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

## Execution

Mọi field đều có 1 function đứng đằng sau gọi là **resolver**. Resolver sẽ trả về next value của field mỗi khi execute

### Resolver

```
Query: {
  human(obj, args, context) {
    return context.db.loadHumanByID(args.id).then(
      userData => new Human(userData)
    )
  }
}
```

- obj: object được trả về ở bước trước (parent)
- args: tham số truyền vào khi query
- context: object dùng chung giữa các resolver, lưu các thông tin như db, user...

Note: resolver của ```human field``` trả về **Promise** do truy vấn db là async

Sau khi có ```Human object```, tiếp tục query field của Human

```
Human: {
  name(obj, args, context) {
    return obj.name
  }
}
```

## Apollo Best Practice

- Manually update store after mutation (or poll if implment pagination)
- Optimistic UI
- Reuse Fragment
- Prefetch
- Split query