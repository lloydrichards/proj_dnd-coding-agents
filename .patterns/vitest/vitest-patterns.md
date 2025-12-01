# Vitest v4.0+ Patterns Reference

Comprehensive patterns and examples for Vitest v4.0+ testing.

## Modern Mocking Patterns

### Module Mocking with vi.hoisted (v4.0+)

```typescript
// ✅ Best: Use vi.hoisted for module mocks
const mockGetUser = vi.hoisted(() => vi.fn());

vi.mock("./api", () => ({
  getUser: mockGetUser(),
}));

// Partial module mocking
vi.mock("./module", async (importOriginal) => {
  const actual = await importOriginal();
  return {
    ...actual,
    mocked: vi.fn(),
  };
});
```

### HTTP Mocking with MSW (Recommended)

```typescript
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";

const handlers = [
  http.get("/api/users/:id", ({ params }) => {
    return HttpResponse.json({ id: params.id, name: "John Doe" });
  }),
  http.post("/api/users", async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 1, ...body }, { status: 201 });
  }),
];

const server = setupServer(...handlers);

beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### Timer Mocking

```typescript
vi.useFakeTimers();
vi.advanceTimersByTime(1000);
vi.useRealTimers(); // Cleanup
```

### Spies

```typescript
const spy = vi.spyOn(object, "method");
spy.mockImplementation(() => "custom");
```

## Concurrent Testing

```typescript
// ✅ Use context expect in concurrent tests
test.concurrent("test 1", ({ expect }) => {
  expect(value).toMatchSnapshot();
});

// ❌ Avoid: Global expect in concurrent tests
test.concurrent("test 2", () => {
  expect(value).toMatchSnapshot(); // May fail
});

// Mark entire suite as concurrent
describe.concurrent("parallel suite", () => {
  test("auto concurrent 1", async () => {});
  test("auto concurrent 2", async () => {});
});
```

## Async Testing

```typescript
// Use async/await
it("should fetch data", async () => {
  const data = await fetchData();
  expect(data).toEqual(expected);
});

// Polling for async conditions (v4.0+)
await expect.poll(() => getStatus(), { timeout: 1000 }).toBe("ready");

// Promise assertions
await expect(promise()).resolves.toBe(value);
await expect(failingPromise()).rejects.toThrow("Error");
```

## Parameterized Testing

### Object Format

```typescript
describe.each([
  { a: 1, b: 1, expected: 2 },
  { a: 2, b: 2, expected: 4 },
])("add($a, $b)", ({ a, b, expected }) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected);
  });
});
```

### Array Format

```typescript
test.each([
  [1, 1, 2],
  [2, 2, 4],
])("add(%i + %i = %i)", (a, b, expected) => {
  expect(a + b).toBe(expected);
});
```

## Test Fixtures

```typescript
import { test as baseTest } from "vitest";

const test = baseTest.extend({
  db: async ({}, use) => {
    const db = await setupDatabase();
    await use(db);
    await db.close();
  },
  user: async ({ db }, use) => {
    const user = await db.createUser({ name: "Test" });
    await use(user);
    await db.deleteUser(user.id);
  },
});

test("with fixtures", async ({ db, user }) => {
  const data = await db.query();
  expect(data).toBeDefined();
});
```

## Component Testing

```typescript
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

it("should handle click", async () => {
  render(<Button onClick={handleClick} />);
  await userEvent.click(screen.getByRole("button"));
  expect(handleClick).toHaveBeenCalled();
});

// Use userEvent.setup() for stateful interactions
const user = userEvent.setup();
await user.type(input, "text");
await user.click(button);
```

## Configuration Best Practices

```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true,
    environment: "node", // or 'jsdom', 'happy-dom'
    setupFiles: ["./setup.ts"],
    coverage: {
      provider: "v8",
      include: ["src/**/*.ts"], // Required in v4.0+
      reporter: ["text", "json", "html"],
      lines: 80,
      functions: 80,
      branches: 75,
    },
    clearMocks: true,
    restoreMocks: true,
  },
});
```

## Full Test Examples

### Unit Test for Utility Function

```typescript
// src/utils/formatCurrency.test.ts
import { describe, it, expect } from "vitest";
import { formatCurrency } from "./formatCurrency";

describe("formatCurrency", () => {
  it("should format positive numbers with USD", () => {
    expect(formatCurrency(1234.56)).toBe("$1,234.56");
  });

  it("should format zero", () => {
    expect(formatCurrency(0)).toBe("$0.00");
  });

  it("should format negative numbers", () => {
    expect(formatCurrency(-1234.56)).toBe("-$1,234.56");
  });

  it("should handle very large numbers", () => {
    expect(formatCurrency(1234567890.12)).toBe("$1,234,567,890.12");
  });

  it("should round to two decimal places", () => {
    expect(formatCurrency(10.999)).toBe("$11.00");
  });
});
```

### Integration Test with Modern Mocking

```typescript
// src/services/UserService.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { UserService } from "./UserService";

const mockGet = vi.hoisted(() => vi.fn());

vi.mock("../api/client", () => ({
  get: mockGet(),
}));

describe("UserService", () => {
  let service: UserService;

  beforeEach(() => {
    vi.clearAllMocks();
    service = new UserService();
  });

  it("should fetch user by id", async () => {
    const mockUser = { id: 1, name: "John" };
    mockGet().mockResolvedValue(mockUser);

    const result = await service.getUser(1);

    expect(mockGet()).toHaveBeenCalledWith("/users/1");
    expect(result).toEqual(mockUser);
  });

  it("should handle fetch errors", async () => {
    mockGet().mockRejectedValue(new Error("Network error"));
    await expect(service.getUser(1)).rejects.toThrow("Network error");
  });

  it("should cache user data", async () => {
    const mockUser = { id: 1, name: "John" };
    mockGet().mockResolvedValue(mockUser);

    await service.getUser(1);
    await service.getUser(1);

    expect(mockGet()).toHaveBeenCalledTimes(1);
  });
});
```

### Component Interaction Test

```typescript
// src/components/LoginForm.test.ts
import { describe, it, expect, vi } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { LoginForm } from "./LoginForm";

describe("LoginForm", () => {
  it("should submit form with credentials", async () => {
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await userEvent.type(screen.getByLabelText(/email/i), "user@example.com");
    await userEvent.type(screen.getByLabelText(/password/i), "password123");
    await userEvent.click(screen.getByRole("button", { name: /login/i }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: "user@example.com",
      password: "password123",
    });
  });

  it("should show validation errors", async () => {
    render(<LoginForm onSubmit={vi.fn()} />);
    await userEvent.click(screen.getByRole("button", { name: /login/i }));

    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    expect(screen.getByText(/password is required/i)).toBeInTheDocument();
  });

  it("should disable submit during loading", async () => {
    render(<LoginForm onSubmit={vi.fn()} isLoading={true} />);
    expect(screen.getByRole("button", { name: /login/i })).toBeDisabled();
  });
});
```

### HTTP API Test with MSW

```typescript
// src/api/users.test.ts
import { describe, it, expect, beforeAll, afterEach, afterAll } from "vitest";
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";
import { fetchUser, createUser } from "./users";

const handlers = [
  http.get("/api/users/:id", ({ params }) => {
    return HttpResponse.json({ id: params.id, name: "John Doe" });
  }),
  http.post("/api/users", async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 1, ...body }, { status: 201 });
  }),
];

const server = setupServer(...handlers);

beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe("User API", () => {
  it("fetches user by id", async () => {
    const user = await fetchUser("123");
    expect(user).toEqual({ id: "123", name: "John Doe" });
  });

  it("creates new user", async () => {
    const newUser = await createUser({ name: "Jane", email: "jane@example.com" });
    expect(newUser).toMatchObject({ id: 1, name: "Jane" });
  });

  it("handles server errors", async () => {
    server.use(
      http.get("/api/users/:id", () => {
        return HttpResponse.json({ error: "Not found" }, { status: 404 });
      })
    );
    await expect(fetchUser("999")).rejects.toThrow("User not found");
  });
});
```

### Database Fixtures Test

```typescript
// src/services/database.test.ts
import { test as baseTest, expect } from "vitest";
import { Database } from "./database";

const test = baseTest.extend({
  db: async ({}, use) => {
    const db = new Database(":memory:");
    await db.initialize();
    await use(db);
    await db.close();
  },
  testUser: async ({ db }, use) => {
    const user = await db.createUser({ name: "Test User", email: "test@example.com" });
    await use(user);
    await db.deleteUser(user.id);
  },
});

test("queries user data", async ({ db, testUser }) => {
  const found = await db.findUser(testUser.id);
  expect(found).toEqual(testUser);
});

test("updates user", async ({ db, testUser }) => {
  await db.updateUser(testUser.id, { name: "Updated Name" });
  const updated = await db.findUser(testUser.id);
  expect(updated.name).toBe("Updated Name");
});
```
