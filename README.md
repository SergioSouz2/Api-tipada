# Api Tipada


## Índice
- [Get Started with API](#get-started-with-api)
- [Configurando e tipando validacao zod](#configurando-e-tipando-validacao-zod)


## Get Started with API
### Lib

- [ ] fastify  
- [ ] fastify-type-provider-zod   
- [ ] @fastify/cors 
- [ ] zod
- [ ] @fastify/swagger 
- [ ] @fastify/swagger-ui

```shell 
npm i fastify fastify-type-provider-zod @fastify/cors zod
```

lib para gerenciar a documentacao da api 

```
npm i @fastify/swagger @fastify/swagger-ui
```

### Lib para rodar o projeto em desenvolvimento

- [ ] typescript
- [ ] @types/node
- [ ] tsx


```shell 
npm i typescript @types/node tsx -D 

```

No arquivo package.json crie um script para rodar o server 

```json
    "scripts": {
        "dev": "tsx watch src/server.ts"
    },
```

### Configurando Typescript e criando o arquivo tsconfig 

```
npx tsc --init
```

recomendamos abrir  o https://github.com/tsconfig/bases?tab=readme-ov-file e pega o tsconfig.json da versao referente do nodeJS
e alterar o arquivo.


## Configurando e tipando validacao zod 

para o fastify conseguir intender que vamos usar o zod para validacao de dados das nossas rotas precisamos dizer pra ele, para isso vamos criar um FastifyTypeInstance 

criar um arquivo types.ts 

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
>
```
E no server e preciso  adicionar tambem o transform json 

```ts
app.register(fastifySwagger,{
    openapi:{
        info:{
            title: 'Typed API',
            version: '1.0.0'
        }
    },
    transform:jsonSchemaTransform
})
```

Exemplo:
```ts
import { randomUUID } from "node:crypto";
import { FastifyTypeInstance } from "../types/types";
import z from "zod";


interface User {
    id : string
    name : string
    email : string
}

const users:User[] = [] // arrey de usuario simulando um banco 

export async function  routes(app:FastifyTypeInstance){


    // ROUTES GET USERS
    app.get('/users', {
        schema:{
            tags:['users'],
            description: 'List user',
            response:{
                200: z.array(z.object({
                    id:z.string(),
                    name: z.string(),
                    email: z.string()
                }))
            }
        }
    }, () => {
        return users
    })


    // ROUTES POST USERS
    app.post('/users',{
        schema:{
            tags:['users'],
            description: 'Create a new user',
            body:z.object({
                name: z.string(),
                email: z.string().email()
            }),
            response: {
                201: z.null().describe('User created')
            }
        }
    }, async (request,reply)=> {

        const { name, email } =  request.body

        users.push({
            id: randomUUID(),
            name,
            email
        })

        return reply.status(201).send()
    })

}
```