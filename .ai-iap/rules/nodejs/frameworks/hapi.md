# Hapi

> **Scope**: Apply these rules for Hapi applications (JavaScript or TypeScript).

## Overview

Hapi is a configuration-centric Node.js framework with built-in input validation, auth, and caching.

## Best Practices

**MUST**:
- Use Joi for validation
- Use plugins for modularity
- Define routes as objects
- Use async handlers

**SHOULD**:
- Use @hapi plugins for features
- Use pre/post handlers for middleware
- Use server methods for shared logic
- Configure via options

**AVOID**:
- Missing validation schemas
- Logic in route definitions
- Synchronous operations

## Project Structure
```
src/
├── plugins/          # Auth, database
├── routes/           # Route definitions
├── handlers/         # Request handlers
├── schemas/          # Joi validation schemas
├── services/
├── types/            # (TS only)
└── server.js|ts
```

## Key Patterns

```javascript
// Server Setup
const server = Hapi.server({ port: 3000, routes: { cors: true } });
await server.register([authPlugin]);
server.route(userRoutes);

// Routes with Joi Validation
export const userRoutes = [{
  method: 'POST',
  path: '/api/users',
  options: {
    auth: 'jwt',
    handler: handlers.createUser,
    validate: { payload: Joi.object({ email: Joi.string().email().required() }) }
  }
}];

// Handler
export async function createUser(request, h) {
  const user = await userService.create(request.payload);
  if (!user) throw Boom.notFound('User not found');
  return h.response({ data: user }).code(201);
}

// Auth Plugin
export const authPlugin = {
  name: 'auth',
  register: async (server) => {
    await server.register(Jwt);
    server.auth.strategy('jwt', 'jwt', { keys: process.env.JWT_SECRET });
    server.auth.default('jwt');
  }
};
```

## Best Practices
- **Joi Validation**: Always validate params, query, and payload
- **Boom Errors**: Consistent, type-safe HTTP errors
- **Plugin System**: Encapsulate features as plugins
- **request.payload**: Access body (not request.body)

