# Deroutex: Modern Deno Router

A Deno port of [Routex](https://routex.js.org).

This is not ready for production!

What is missing:

- Documentation (currently Routex's docs can be used, main changes are to the `listen` APIs)
- First-party packages for body parsing, cookies, WebSockets, etc
- Finding a home to release it
- Benchmarking and extensive testing

Feel free to test and give feedback 😃

### Features

- Easy to use, good performance
- Modern API, native Promise support, fully typed (TypeScript)
- Very few dependencies, small API surface, easy to fully understand and extend
- 100% code coverage, well tested

## [Example](./example.ts)

```ts
import { Deroutex, TextBody, JsonBody, ErrorWithBody } from "./mod.ts";

// Initial the Deroutex application
const app = new Deroutex();

// Basic GET router
app.get("/", () => {
  // Response bodies are returned
  return new TextBody("Hello Deroutex!");
});

// Path params
app.get("/greet/:name", (ctx) => {
  // Alternative to returning a body
  ctx.body = new JsonBody({ hello: ctx.params.name });
});

// This POST will trigger the default error handler (can be overriden)
app.post("/error", () => {
  throw new Error("Flow control with errors!");
});

// Create a child router
const privateRouter = app.child("/private");

// Add a middleware to our router. `app` is a router too!
privateRouter.middleware(async (ctx) => {
  // Pre-handler middleware
  if (!ctx.req.headers.has("Authorization")) {
    throw new ErrorWithBody(401, new JsonBody({ error: "UNAUTHORIZED" }));
  }

  return () => {
    // Post-handler middleware: This example wraps the handle's body ({"body": "You are authorized!"})
    ctx.body = new JsonBody({
      body: ctx.body?.toString(),
    });
  };
});

// Catch-all path
privateRouter.any("/", () => {
  return new TextBody("You are authorized!");
}, { exact: false });

// Start the server!
await app.listenAndServe(":5000");
```

## Support

Since Deno is evolving quickly, only the latest version is officially supported.

Please file feature requests and bugs at the [issue tracker](https://github.com/routexjs/deroutex/issues).
