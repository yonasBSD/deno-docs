---
title: "How to create a RESTful API with Prisma and Oak"
description: "Guide to building a RESTful API using Prisma and Oak with Deno. Learn how to set up database schemas, generate clients, implement CRUD operations, and deploy your API with proper type safety."
url: /examples/prisma_tutorial/
oldUrl:
  - /runtime/manual/examples/how_to_with_npm/prisma/
  - /runtime/tutorials/how_to_with_npm/prisma/
---

[Prisma](https://prisma.io) has been one of our top requested modules to work
with in Deno. The demand is understandable, given that Prisma's developer
experience is top notch and plays well with so many persistent data storage
technologies.

We're excited to show you how to use Prisma with Deno.

In this How To guide, we'll setup a simple RESTful API in Deno using Oak and
Prisma.

Let's get started.

[View source](https://github.com/denoland/examples/tree/main/with-prisma) or
[check out the video guide](https://youtu.be/P8VzA_XSF8w).

## Setup the application

Let's create the folder `rest-api-with-prisma-oak` and navigate there:

```shell
mkdir rest-api-with-prisma-oak
cd rest-api-with-prisma-oak
```

Then, let's run `prisma init` with Deno:

```shell
npx prisma@latest init --generator-provider prisma-client --output ./generated
```

Let's understand the key parameters:

- `--generator-provider prisma-client`: Define o provedor "prisma-client" ao
  invés do padrão "prisma-client-js". O provedor "prisma-client" é otimizado
  para Deno e gera código TypeScript compatível com o runtime do Deno.

- `--output`: Define o diretório onde o Prisma salvará os arquivos do cliente
  gerado, incluindo definições de tipos e utilitários de acesso ao banco de
  dados.

This will generate
[`prisma/schema.prisma`](https://www.prisma.io/docs/orm/prisma-schema). Let's
update it with the following:

> [!TIP]
> Don't forget to add `runtime = "deno"` to the generator block in your
> schema.prisma file. This is required for Prisma to work correctly with Deno.

```ts
generator client {
  provider = "prisma-client"
  output   = "./generated"
  runtime = "deno"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Dinosaur {
  id          Int     @id @default(autoincrement())
  name        String  @unique
  description String
}
```

Prisma also generates a `.env` file with a `DATABASE_URL` environment variable.
Let's assign `DATABASE_URL` to a PostgreSQL connection string. In this example,
we'll use a free
[PostgreSQL database from Supabase](https://supabase.com/database).

Next, let's create the database schema:

```shell
deno run -A npm:prisma@latest db push
```

After that's complete, we'll need to generate a Prisma Client:

```shell
deno run -A npm:prisma@latest generate
```

## Setup Accelerate in the Prisma Data Platform

> Note: This is an optional step. Prisma Accelerate is not required for the
> basic functionality.

To get started with the Prisma Data Platform:

1. Sign up for a free [Prisma Data Platform account](https://console.prisma.io).
2. Create a project.
3. Navigate to the project you created.
4. Enable Accelerate by providing your database's connection string.
5. Generate an Accelerate connection string and copy it to your clipboard.

Assign the Accelerate connection string, that begins with `prisma://`, to
`DATABASE_URL` in your `.env` file replacing your existing connection string.

Next, let's create a seed script to seed the database.

## Seed your Database

Create `./prisma/seed.ts`:

```shell
touch prisma/seed.ts
```

And in `./prisma/seed.ts`:

```ts
import { Prisma, PrismaClient } from "./generated/client.ts";

const prisma = new PrismaClient({
  datasourceUrl: Deno.env.get("DATABASE_URL"),
});

const dinosaurData: Prisma.DinosaurCreateInput[] = [
  {
    name: "Aardonyx",
    description: "An early stage in the evolution of sauropods.",
  },
  {
    name: "Abelisaurus",
    description: "Abel's lizard has been reconstructed from a single skull.",
  },
  {
    name: "Acanthopholis",
    description: "No, it's not a city in Greece.",
  },
];

/**
 * Seed the database.
 */

for (const u of dinosaurData) {
  const dinosaur = await prisma.dinosaur.create({
    data: u,
  });
  console.log(`Created dinosaur with id: ${dinosaur.id}`);
}
console.log(`Seeding finished.`);

await prisma.$disconnect();
```

We can now run `seed.ts` with:

```shell
deno run -A --env prisma/seed.ts
```

> [!TIP]
>
> The `--env` flag is used to tell Deno to load environment variables from the
> `.env` file.

After doing so, you should be able to see your data on Prisma Studio by running
the following command:

```bash
deno run -A npm:prisma studio
```

You should see something similar to the following screenshot:

![New dinosaurs are in Prisma dashboard](./images/how-to/prisma/1-dinosaurs-in-prisma.png)

## Create your API routes

We'll use [`oak`](https://jsr.io/@oak/oak) to create the API routes. Let's keep
them simple for now.

Let's create a `main.ts` file:

```shell
touch main.ts
```

Then, in your `main.ts` file:

```ts
import { PrismaClient } from "./prisma/generated/client.ts";
import { Application, Router } from "jsr:@oak/oak";

/**
 * Initialize.
 */

const prisma = new PrismaClient({
  datasources: {
    db: {
      url: Deno.env.get("DATABASE_URL")!,
    },
  },
});
const app = new Application();
const router = new Router();

/**
 * Setup routes.
 */

router
  .get("/", (context) => {
    context.response.body = "Welcome to the Dinosaur API!";
  })
  .get("/dinosaur", async (context) => {
    // Get all dinosaurs.
    const dinosaurs = await prisma.dinosaur.findMany();
    context.response.body = dinosaurs;
  })
  .get("/dinosaur/:id", async (context) => {
    // Get one dinosaur by id.
    const { id } = context.params;
    const dinosaur = await prisma.dinosaur.findUnique({
      where: {
        id: Number(id),
      },
    });
    context.response.body = dinosaur;
  })
  .post("/dinosaur", async (context) => {
    // Create a new dinosaur.
    const { name, description } = await context.request.body.json();
    const result = await prisma.dinosaur.create({
      data: {
        name,
        description,
      },
    });
    context.response.body = result;
  })
  .delete("/dinosaur/:id", async (context) => {
    // Delete a dinosaur by id.
    const { id } = context.params;
    const dinosaur = await prisma.dinosaur.delete({
      where: {
        id: Number(id),
      },
    });
    context.response.body = dinosaur;
  });

/**
 * Setup middleware.
 */

app.use(router.routes());
app.use(router.allowedMethods());

/**
 * Start server.
 */

await app.listen({ port: 8000 });
```

Now, let's run it:

```shell
deno run -A --env main.ts
```

Let's visit `localhost:8000/dinosaurs`:

![List of all dinosaurs from REST API](./images/how-to/prisma/2-dinosaurs-from-api.png)

Next, let's `POST` a new user with this `curl` command:

```shell
curl -X POST http://localhost:8000/dinosaur -H "Content-Type: application/json" -d '{"name": "Deno", "description":"The fastest, most secure, easiest to use Dinosaur ever to walk the Earth."}'
```

You should now see a new row on Prisma Studio:

![New dinosaur Deno in Prisma](./images/how-to/prisma/3-new-dinosaur-in-prisma.png)

Nice!

## What's next?

Building your next app will be more productive and fun with Deno and Prisma,
since both technologies deliver an intuitive developer experience with data
modeling, type-safety, and robust IDE support.

If you're interested in connecting Prisma to Deno Deploy,
[check out this awesome guide](https://www.prisma.io/docs/guides/deployment/deployment-guides/deploying-to-deno-deploy).
