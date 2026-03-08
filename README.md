# swagger-gen-skill

A Claude Code skill that automatically generates Swagger/OpenAPI 3.0 documentation from your project source code.

## What it does

This skill analyzes your codebase — routes, types, middleware, validation schemas — and produces a complete `openapi.yaml` file. No decorators or annotations required.

### Supported Frameworks

| Language | Frameworks |
|----------|-----------|
| TypeScript/JavaScript | Express, Fastify, Hono, Elysia, Koa, Next.js (App Router) |
| Python | FastAPI, Flask, Django |
| Go | Gin, Echo, Fiber, Chi |
| Java | Spring Boot, JAX-RS |
| Rust | Actix-web, Axum |

### What it extracts

- **Routes** — HTTP method + path from router definitions
- **Request params** — path, query, body, headers (from types, Zod, Pydantic, struct tags)
- **Response schemas** — success + error responses with accurate types
- **Authentication** — Bearer, API key, Cookie from middleware analysis
- **Type mappings** — TypeScript interfaces, Zod schemas, Pydantic models, Go structs → OpenAPI schemas

## Installation

### Claude Code (recommended)

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "skills": {
    "swagger-gen": {
      "source": "github:showkkd133/swagger-gen-skill"
    }
  }
}
```

Or copy `skills/swagger-gen/SKILL.md` to `~/.claude/skills/swagger-gen/SKILL.md`.

### Manual

1. Clone this repo
2. Copy `skills/swagger-gen/SKILL.md` to your Claude Code skills directory:
   ```bash
   cp skills/swagger-gen/SKILL.md ~/.claude/skills/swagger-gen/SKILL.md
   ```

## Usage

In Claude Code, just say:

- "generate swagger" / "生成 swagger"
- "generate api docs" / "生成 api 文档"
- "write openapi spec for this project"

The skill will:
1. Detect your project framework
2. Scan all API routes
3. Analyze request/response types
4. Generate `openapi.yaml` (or `docs/openapi.yaml`)
5. Validate with swagger-cli or redocly

## Example Output

```yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
paths:
  /api/users:
    get:
      summary: List all users
      operationId: listUsers
      tags: [Users]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
          format: email
      required: [id, name, email]
```

## Schema Generation Rules

| Source Type | OpenAPI Type |
|------------|-------------|
| `string` | `type: string` |
| `number` / `float` | `type: number` |
| `int` / `integer` | `type: integer` |
| `boolean` | `type: boolean` |
| `Date` / `datetime` | `type: string, format: date-time` |
| `Array<T>` | `type: array, items: {$ref}` |
| `enum` | `type: string, enum: [...]` |
| `File` / `UploadFile` | `type: string, format: binary` |

### Required field detection

- TypeScript: properties without `?` marker
- Zod: fields without `.optional()`
- Pydantic: fields without default values
- Go: struct fields without `omitempty` tag

## License

MIT
