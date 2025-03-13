# Typed API

## Index
- [Get Started with API](#get-started-with-api)
- [Configuring TypeScript and Creating the](#configuring-typescript-and-creating-the-tsconfig-file)
- [Configuring and Typing](#configuring-and-typing-validation-with-zod)
- [Usage Example](#usage-example)
## Get Started with API

### Libraries
- fastify
- fastify-type-provider-zod
- @fastify/cors
- zod
- @fastify/swagger
- @fastify/swagger-ui

Install dependencies with:

```sh
npm i fastify fastify-type-provider-zod @fastify/cors zod
```

Libraries for managing API documentation:

```sh
npm i @fastify/swagger @fastify/swagger-ui
```

Libraries for running the project in development:
- typescript
- @types/node
- tsx

```sh
npm i typescript @types/node tsx -D
```

In the `package.json` file, create a script to run the server:

```json
"scripts": {
    "dev": "tsx watch src/server.ts"
}
```

## Configuring TypeScript and Creating the `tsconfig` File

```sh
npx tsc --init
```

We recommend opening the [tsconfig/bases](https://github.com/tsconfig/bases?tab=readme-ov-file) repository, getting the `tsconfig.json` for the corresponding Node.js version, and modifying the file as needed.

---

## Configuring and Typing Validation with Zod

For Fastify to understand that we are using Zod for data validation in our routes, we need to configure it properly. To do this, we will create a `FastifyTypeInstance`.

Create a `types.ts` file:

```ts
import { 
    FastifyBaseLogger, 
    FastifyInstance, 
    RawReplyDefaultExpression, 
    RawRequestDefaultExpression, 
    RawServerDefault 
} from "fastify";

import { ZodTypeProvider } from "fastify-type-provider-zod";

export type FastifyTypeInstance = FastifyInstance<
    RawServerDefault,
    RawRequestDefaultExpression,
    RawReplyDefaultExpression,
    FastifyBaseLogger,
    ZodTypeProvider
>;
```

In the server, it is also necessary to add the `transform json`:

```ts
app.register(fastifySwagger, {
    openapi: {
        info: {
            title: 'Typed API',
            version: '1.0.0'
        }
    },
    transform: jsonSchemaTransform
});
```

---

## Usage Example

```ts
import { randomUUID } from "node:crypto";
import { FastifyTypeInstance } from "../types/types";
import z from "zod";

interface User {
    id: string;
    name: string;
    email: string;
}

const users: User[] = []; // Array of users simulating a database

export async function routes(app: FastifyTypeInstance) {
    // GET USERS ROUTE
    app.get('/users', {
        schema: {
            tags: ['users'],
            description: 'List users',
            response: {
                200: z.array(z.object({
                    id: z.string(),
                    name: z.string(),
                    email: z.string()
                }))
            }
        }
    }, () => {
        return users;
    });

    // POST USERS ROUTE
    app.post('/users', {
        schema: {
            tags: ['users'],
            description: 'Create a new user',
            body: z.object({
                name: z.string(),
                email: z.string().email()
            }),
            response: {
                201: z.null().describe('User created')
            }
        }
    }, async (request, reply) => {
        const { name, email } = request.body;

        users.push({
            id: randomUUID(),
            name,
            email
        });

        return reply.status(201).send();
    });
}
```

