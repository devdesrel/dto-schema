## DTO Schema

[![Build](https://github.com/umidbekkarimov/dto-schema/workflows/Build/badge.svg?branch=master)](https://github.com/umidbekkarimov/dto-schema/actions)
[![codecov](https://codecov.io/gh/umidbekkarimov/dto-schema/branch/master/graph/badge.svg)](https://codecov.io/gh/umidbekkarimov/dto-schema)
[![npm version](https://img.shields.io/npm/v/dto-schema.svg)](https://npmjs.com/dto-schema)
[![npm minzipped size](https://img.shields.io/bundlephobia/minzip/dto-schema.svg)](https://bundlephobia.com/result?p=dto-schema)
[![npm type definitions](https://img.shields.io/npm/types/dto-schema.svg)](https://npmjs.com/dto-schema)
[![npm downloads](https://img.shields.io/npm/dm/dto-schema.svg)](https://npmjs.com/dto-schema)
[![npm license](https://img.shields.io/npm/l/dto-schema.svg)](https://npmjs.com/dto-schema)

### Installation

```bash
# NPM
npm i dto-schema
# Yarn
yarn add dto-schema
```

#### Babel

Setup [@babel/plugin-proposal-decorators](https://babeljs.io/docs/en/babel-plugin-proposal-decorators) with `legacy` mode.

```json
{
  "plugins": [["@babel/plugin-proposal-decorators", { "legacy": true }]]
}
```

#### TypeScript

Enable `experimentalDecorators` property in `tsconfig.json`

```json
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

#### Browser

Current build targets modern browsers that support ESModules (see the [browserlist](https://github.com/babel/babel/blob/master/packages/babel-compat-data/data/native-modules.json)), so you will need to polyfill required **ES2015** features like `Map` and `WeakMap` and transpile unsupported syntax like **classes** or **arrow functions**.

### API

- Traversers
  - [`parseDTO`](#parsedto)
  - [`serializeDTO`](#serializedto)
- Decorators
  - [`@BooleanProp`](#booleanprop)
  - [`@NumberProp`](#numberprop)
  - [`@StringProp`](#stringprop)
  - [`@DateProp`](#dateprop)
  - [`@ObjectProp`](#objectprop)
  - [`@Prop`](#prop)
  - [`@ArrayProp`](#arrayprop)

#### `parseDTO`

```typescript
function parseDTO<T extends object>(Cls: DTOConstructor, raw: T | object): T;
```

Transforms provided `raw` value with the rules defined in the `Cls` constructor.

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, StringProp } from 'dto-schema';

class UserDTO {
  @StringProp() guid: string;
  @StringProp() name: string;
  password: string; // this one is not annotated, so it would not be transformed.
}

expect(parseDTO(UserDTO, {})).toEqual({ guid: '', name: '' });

expect(
  parseDTO(UserDTO, { guid: 1, name: 'Leeroy Jenkins', password: 'yolo' }),
).toEqual({ guid: '1', name: 'Leeroy Jenkins' });

const user = new UserDTO();
user.name = 'John';
expect(parseDTO(UserDTO, user)).toEqual({ guid: '', name: 'John' });
```

</details>

#### `serializeDTO`

```typescript
function serializeDTO<T extends object>(
  Cls: DTOConstructor,
  raw: T | object,
): object;
```

Unlike `parseDTO`, `serializedDTO` uses `DTOSchemaOptions#serialize` method which prepares a valid JSON object.
Check [`DateProp`](#dateprop) and [`Prop`](#prop) examples.

#### `@BooleanProp`

```typescript
interface BooleanPropOptions {
  defaultValue?: null | boolean;
}

function BooleanProp(options?: BooleanPropOptions): PropertyDecorator;
```

Annotates property as a `boolean`.
Accepts:

- `defaultValue` - default value to use when input value is `null` or `undefined`

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, BooleanProp } from 'dto-schema';

class UserDTO {
  @BooleanProp() isVerified: boolean;
  @BooleanProp({ defaultValue: true }) isActive: boolean;
  @BooleanProp({ defaultValue: null }) isSubscribed: boolean;
}

expect(parseDTO(UserDTO, {})).toEqual({
  isVerified: false,
  isActive: true,
  isSubscribed: null,
});

expect(
  parseDTO(UserDTO, {
    isVerified: true,
    isActive: false,
    isSubscribed: false,
  }),
).toEqual({
  isVerified: true,
  isActive: false,
  isSubscribed: false,
});
```

</details>

#### `@NumberProp`

```typescript
interface NumberPropOptions {
  defaultValue?: null | number;
  clampMin?: number;
  clampMax?: number;
  round?: boolean | 'ceil' | 'floor' | 'trunc';
}

function NumberProp(options?: NumberPropOptions): PropertyDecorator;
```

Annotates property as a `number`.
Accepts:

- `clampMin` - lower bound of the number to clamp
- `clampMax` - upper bound of the number to clamp
- `round` - round method to use to adjust a value
- `defaultValue` - default value to use when input value is `null` or `undefined`

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, NumberProp } from 'dto-schema';

class ProductFilter {
  @NumberProp()
  id: number;

  @NumberProp({ defaultValue: 1, round: true, clampMin: 1 })
  page: number;

  @NumberProp({ defaultValue: null, round: true, clampMin: 1, clampMax: 50 })
  pageSize: number;
}

expect(parseDTO(ProductFilter, {})).toEqual({
  id: NaN,
  page: 1,
  pageSize: null,
});

expect(
  parseDTO(ProductFilter, {
    id: 42,
    page: 0,
    pageSize: 100,
  }),
).toEqual({
  id: 42,
  page: 1,
  pageSize: 50,
});

expect(
  parseDTO(ProductFilter, {
    id: 42,
    page: 3.33333,
    pageSize: 3.3333,
  }),
).toEqual({
  id: 42,
  page: 3,
  pageSize: 3,
});
```

</details>

#### `@StringProp`

```typescript
interface StringPropOptions {
  defaultValue?: null | string;
  trim?: boolean | 'start' | 'end';
}

function StringProp(options?: StringPropOptions): PropertyDecorator;
```

Annotates property as a `string`.
Accepts:

- `defaultValue` - default value to use when input value is `null` or `undefined`
- `trim` - trim method to adjust input value

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, StringProp } from 'dto-schema';

class UserDTO {
  @StringProp() guid: string;
  @StringProp({ trim: true }) name: string;
  @StringProp({ trim: true, defaultValue: null }) email: string;
}

expect(parseDTO(UserDTO, {})).toEqual({
  guid: '',
  name: '',
  email: null,
});

expect(
  parseDTO(UserDTO, {
    guid: '123',
    name: ' Leeroy\n ',
    email: ' leeroy@jenkins.dev \n',
  }),
).toEqual({
  guid: '123',
  name: 'Leeroy',
  email: 'leeroy@jenkins.dev',
});
```

</details>

#### `@DateProp`

```typescript
interface DatePropOptions {
  defaultValue?: () => null | number | string | Date;
}

function DateProp(options?: DatePropOptions): PropertyDecorator;
```

Annotates property as a `Date`.
Converted to the `ISO` string when serialized.
Accepts:

- `defaultValue` - default value to use when input value is `null` or `undefined`

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, serializeDTO, DateProp } from 'dto-schema';

class PostDTO {
  @DateProp() updatedAt: string;
  @DateProp({ defaultValue: null }) deletedAt: string;
  @DateProp({ defaultValue: () => Date.UTC(2019, 4, 24) })
  createdAt: string;
}

expect(parseDTO(PostDTO, {})).toEqual({
  updatedAt: new Date(NaN), // Invalid Date,
  deletedAt: null,
  createdAt: new Date(Date.UTC(2019, 4, 24)),
});

expect(serializeDTO(PostDTO, {})).toEqual({
  updatedAt: null, // Invalid Date,
  deletedAt: null,
  createdAt: '2019-05-24T00:00:00.000Z',
});

expect(
  parseDTO(PostDTO, {
    createdAt: Date.UTC(2019, 4, 24),
    updatedAt: '2019-05-24T00:00:00.000Z',
    deletedAt: new Date(Date.UTC(2019, 4, 24)),
  }),
).toEqual({
  updatedAt: new Date(Date.UTC(2019, 4, 24)),
  deletedAt: new Date(Date.UTC(2019, 4, 24)),
  createdAt: new Date(Date.UTC(2019, 4, 24)),
});

expect(
  serializeDTO(PostDTO, {
    createdAt: Date.UTC(2019, 4, 24),
    updatedAt: '2019-05-24T00:00:00.000Z',
    deletedAt: new Date(Date.UTC(2019, 4, 24)),
  }),
).toEqual({
  updatedAt: '2019-05-24T00:00:00.000Z',
  deletedAt: '2019-05-24T00:00:00.000Z',
  createdAt: '2019-05-24T00:00:00.000Z',
});
```

</details>

#### `@UnknownProp`

```typescript
interface UnknownPropOptions {
  defaultValue?: () => null | number | string | Date;
}

function UnknownProp(options?: UnknownPropOptions): PropertyDecorator;
```

Annotates property as a `Date`.
Converted to the `ISO` string when serialized.
Accepts:

- `defaultValue` - default value to use when input value is `null` or `undefined`

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, serializeDTO, UnknownProp } from 'dto-schema';

class UserDTO {
  @UnknownProp() role: unknown;
}

expect(parseDTO(UserDTO, {}), { role: null });
expect(serializeDTO(UserDTO, {}), { role: null });
expect(parseDTO(UserDTO, { role: 1 }), { role: 1 });
expect(serializeDTO(UserDTO, { role: 1 }), { role: 1 });
expect(parseDTO(UserDTO, { role: 'a' }), { role: 'a' });
expect(serializeDTO(UserDTO, { role: 'a' }), { role: 'a' });
expect(parseDTO(UserDTO, { role: null }), { role: null });
expect(serializeDTO(UserDTO, { role: null }), { role: null });
```

</details>

#### `@ObjectProp`

```typescript
interface ObjectPropOptions {
  nullable?: boolean;
}

function ObjectProp(options?: ObjectPropOptions): PropertyDecorator;
```

Annotates property as a `Object` with another DTO type.

Accepts:

- `nullable` - converts value to `null` when it's `null` or `undefined`, otherwise converts into DTO with an empty object

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, ObjectProp, StringProp } from 'dto-schema';

class UserDTO {
  @StringProp() guid: string;
}

class PostDTO {
  @ObjectProp(() => UserDTO) createdBy: UserDTO;
  @ObjectProp(() => UserDTO, { nullable: true }) updatedBy: null | UserDTO;
}

expect(parseDTO(PostDTO, {})).toEqual({
  createdBy: { guid: '' },
  updatedBy: null,
});

expect(
  parseDTO(PostDTO, {
    createdBy: { guid: 123 },
    updatedBy: { guid: 456 },
  }),
).toEqual({
  createdBy: { guid: '123' },
  updatedBy: { guid: '456' },
});
```

</details>

#### `@Prop`

```typescript
interface DTOSchemaOptions<TValue = unknown> {
  type: string;
  testType: (value: unknown) => boolean;
  normalize: (value: unknown) => null | TValue;
  serialize?: (value: null | TValue) => unknown;
}

function Prop<T>(options?: DTOSchemaOptions<T>): PropertyDecorator;
```

Annotates property with with another DTO type.

Accepts:

- `type` - name of the schema type, used for error messages
- `testType` - used to validate object type of the schema
- `normalize` - used to convert input value to required
- `serialize` - used to convert value to JSON value

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, serializeDTO, Prop } from 'dto-schema';

class UserFilter {
  @Prop<Set<string>>({
    type: 'set',
    testType(value) {
      return value instanceof Set;
    },
    normalize(value) {
      return value instanceof Set
        ? value
        : Array.isArray(value)
        ? new Set(value)
        : new Set();
    },
    serialize(value) {
      return value == null ? [] : Array.from(value);
    },
  })
  roles: Set<string>;
}

expect(parseDTO(UserFilter, {})).toEqual({
  roles: new Set(),
});

expect(serializeDTO(UserFilter, {})).toEqual({
  roles: [],
});

expect(
  parseDTO(UserFilter, {
    roles: ['admin', 'editor', 'admin'],
  }),
).toEqual({
  roles: new Set(['admin', 'editor']),
});

expect(serializeDTO(UserFilter, {})).toEqual({
  roles: ['admin', 'editor'],
});
```

</details>

#### `@ArrayProp`

```typescript
interface ArrayPropOptions<T> {
  defaultValue?: () => null | T[];
}

function ArrayProp<T>(options?: ArrayPropOptions<T>): PropertyDecorator;
```

Annotates property as an `Array`, used in combination with another decorator.

Accepts:

- `defaultValue` - default value to use when input value is `null` or `undefined`

<details>
<summary>Usage</summary>

```typescript
import { parseDTO, ArrayProp, StringProp } from 'dto-schema';

class PostDTO {
  @ArrayProp({ defaultValue: () => null }) meta: unknown[];
  @ArrayProp({ defaultValue: () => ['News'] }) @StringProp() tags: string[];
}

expect(parseDTO(PostDTO, {})).toEqual({
  meta: null,
  tags: ['News'],
});

expect(
  parseDTO(PostDTO, {
    meta: ['this', null, 'can', undefined, 'be', NaN, 'anything', true],
    tags: ['this', null, 'will', undefined, 'be', NaN, 'transformed', true],
  }),
).toEqual({
  meta: ['this', null, 'can', undefined, 'be', NaN, 'anything', true],
  tags: ['this', '', 'will', '', 'be', 'NaN', 'transformed', 'true'],
});
```

</details>
