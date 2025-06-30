# Jest Testing Best Practices Guide

## Table of Contents
1. [Test Structure & Organization](#test-structure--organization)
2. [Writing Effective Tests](#writing-effective-tests)
3. [Mocking Best Practices](#mocking-best-practices)
4. [Async Testing](#async-testing)
5. [Setup and Teardown](#setup-and-teardown)
6. [Configuration Best Practices](#configuration-best-practices)
7. [Testing Patterns](#testing-patterns)
8. [Performance Optimization](#performance-optimization)
9. [Error Handling & Debugging](#error-handling--debugging)
10. [CI/CD Integration](#cicd-integration)
11. [Common Anti-Patterns to Avoid](#common-anti-patterns-to-avoid)
12. [React-Specific Testing](#react-specific-testing)

---

## Test Structure & Organization

### File Structure

```javascript
// Use descriptive test file names
user.service.test.js
payment.controller.test.js
auth.integration.test.js

// Or co-locate tests
src/
  components/
    Button/
      Button.js
      Button.test.js
      Button.stories.js
```

### Test Grouping

```javascript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', () => {
      // Test implementation
    });

    it('should throw error for invalid email', () => {
      // Test implementation
    });
  });

  describe('updateUser', () => {
    // Related tests grouped together
  });
});
```

---

## Writing Effective Tests

### Test Naming Conventions

```javascript
// ❌ Poor naming
it('user test', () => {});

// ✅ Descriptive naming
it('should return user profile when valid ID provided', () => {});
it('should throw ValidationError when email format is invalid', () => {});
it('should update user status to active after email verification', () => {});
```

### AAA Pattern (Arrange, Act, Assert)

```javascript
it('should calculate total price with tax', () => {
  // Arrange
  const items = [
    { price: 10, quantity: 2 },
    { price: 5, quantity: 1 }
  ];
  const taxRate = 0.1;

  // Act
  const result = calculateTotal(items, taxRate);

  // Assert
  expect(result).toBe(27.5);
});
```

---

## Mocking Best Practices

### Mock External Dependencies

```javascript
// Mock modules at the top level
jest.mock('../services/apiService');
jest.mock('axios');

describe('DataService', () => {
  beforeEach(() => {
    // Clear mocks between tests
    jest.clearAllMocks();
  });

  it('should fetch user data', async () => {
    const mockUser = { id: 1, name: 'John' };
    apiService.getUser.mockResolvedValue(mockUser);

    const result = await dataService.fetchUser(1);
    
    expect(apiService.getUser).toHaveBeenCalledWith(1);
    expect(result).toEqual(mockUser);
  });
});
```

### Partial Mocking

```javascript
// Mock only specific methods
jest.mock('../utils/logger', () => ({
  ...jest.requireActual('../utils/logger'),
  error: jest.fn(),
  debug: jest.fn()
}));
```

### Mock Functions vs Mock Modules

```javascript
// Use jest.fn() for simple function mocks
const mockCallback = jest.fn();

// Use jest.spyOn() to spy on existing methods
const consoleSpy = jest.spyOn(console, 'error').mockImplementation();

// Restore after tests
afterEach(() => {
  consoleSpy.mockRestore();
});
```

---

## Async Testing

### Async/Await Pattern

```javascript
it('should handle async operations', async () => {
  const result = await asyncFunction();
  expect(result).toBeDefined();
});

// Test error handling
it('should handle async errors', async () => {
  await expect(failingAsyncFunction()).rejects.toThrow('Error message');
});
```

### Testing Promises

```javascript
it('should resolve with correct data', () => {
  return expect(promiseFunction()).resolves.toBe('expected value');
});

it('should reject with error', () => {
  return expect(promiseFunction()).rejects.toThrow();
});
```

---

## Setup and Teardown

### Lifecycle Hooks

```javascript
describe('Database tests', () => {
  beforeAll(async () => {
    // Setup once before all tests
    await database.connect();
  });

  afterAll(async () => {
    // Cleanup once after all tests
    await database.disconnect();
  });

  beforeEach(() => {
    // Setup before each test
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup after each test
    jest.restoreAllMocks();
  });
});
```

### Test Data Management

```javascript
// Use factories for test data
const createMockUser = (overrides = {}) => ({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  ...overrides
});

// Use constants for repeated values
const VALID_EMAIL = 'test@example.com';
const INVALID_EMAIL = 'invalid-email';
```

---

## Configuration Best Practices

### Jest Configuration (jest.config.js)

```javascript
module.exports = {
  // Test environment
  testEnvironment: 'node', // or 'jsdom' for browser-like

  // Coverage settings
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },

  // File patterns
  testMatch: [
    '**/__tests__/**/*.[jt]s?(x)',
    '**/?(*.)+(spec|test).[jt]s?(x)'
  ],

  // Setup files
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],

  // Module paths
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },

  // Clear mocks between tests
  clearMocks: true,
  restoreMocks: true
};
```

### Setup Files

```javascript
// setupTests.js
import 'jest-extended'; // Additional matchers

// Global test utilities
global.testUtils = {
  createMockUser: (overrides) => ({ /* mock data */ }),
  wait: (ms) => new Promise(resolve => setTimeout(resolve, ms))
};

// Suppress console errors in tests
beforeEach(() => {
  jest.spyOn(console, 'error').mockImplementation(() => {});
});

afterEach(() => {
  console.error.mockRestore();
});
```

---

## Testing Patterns

### Parameterized Tests

```javascript
describe.each([
  ['add', 1, 2, 3],
  ['subtract', 5, 3, 2],
  ['multiply', 2, 4, 8]
])('Calculator %s operation', (operation, a, b, expected) => {
  it(`should return ${expected} when ${operation}(${a}, ${b})`, () => {
    const result = calculator[operation](a, b);
    expect(result).toBe(expected);
  });
});
```

### Test Utilities

```javascript
// testUtils.js
export const renderWithProviders = (ui, options = {}) => {
  // Custom render function with providers
  return render(ui, { wrapper: AllTheProviders, ...options });
};

export const createMockStore = (initialState) => {
  // Mock store creation
};

export const waitForLoadingToFinish = () => {
  return waitFor(() => {
    expect(screen.queryByTestId('loading')).not.toBeInTheDocument();
  });
};
```

---

## Performance Optimization

### Test Performance

```javascript
// Use --maxWorkers for CI/CD
// package.json
{
  "scripts": {
    "test": "jest",
    "test:ci": "jest --maxWorkers=2 --coverage"
  }
}

// Skip slow tests in development
const runSlowTests = process.env.RUN_SLOW_TESTS === 'true';

(runSlowTests ? describe : describe.skip)('Slow integration tests', () => {
  // Slow tests here
});
```

### Memory Management

```javascript
// Clear modules between test suites if needed
afterAll(() => {
  jest.resetModules();
});

// Use --logHeapUsage to monitor memory
// jest --logHeapUsage
```

---

## Error Handling & Debugging

### Better Error Messages

```javascript
// Custom matchers for better errors
expect.extend({
  toHaveValidUser(received) {
    const pass = received.id && received.email && received.name;
    return {
      message: () => `Expected ${received} to be a valid user object`,
      pass
    };
  }
});

// Detailed assertions
expect(result).toEqual(
  expect.objectContaining({
    id: expect.any(Number),
    email: expect.stringMatching(/\S+@\S+\.\S+/)
  })
);
```

### Debug Mode

```javascript
// Enable debug output
process.env.DEBUG = 'myapp:*';

// Use --verbose for detailed output
// jest --verbose

// Debug specific tests
fit('debug this test', () => {
  console.log('Debug info:', data);
  // test implementation
});
```

---

## CI/CD Integration

### Package.json Scripts

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:ci": "jest --ci --coverage --watchAll=false",
    "test:staged": "jest --findRelatedTests"
  }
}
```

### Coverage and Quality Gates

```javascript
// Fail builds on coverage thresholds
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    // Per-file thresholds
    './src/critical-module.js': {
      statements: 95
    }
  }
};
```

---

## Common Anti-Patterns to Avoid

### ❌ What NOT to do

```javascript
// Don't test implementation details
expect(component.state.isLoading).toBe(false);

// Don't write tests that are too coupled
expect(mockFunction).toHaveBeenCalledTimes(3);

// Don't test external libraries
it('should call axios.get', () => {
  // This tests axios, not your code
});

// Don't use random data without seeding
const randomId = Math.random();
```

### ✅ Better Approaches

```javascript
// Test behavior, not implementation
expect(screen.getByText('Submit')).toBeEnabled();

// Test outcomes, not call counts
expect(result.users).toHaveLength(expectedUsers.length);

// Test your integration with libraries
expect(apiCall).toHaveReturnedWith(expectedData);

// Use deterministic test data
const testId = 12345;
```

---

## React-Specific Testing

### Component Testing with React Testing Library

```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('should handle user interaction', async () => {
  const user = userEvent.setup();
  render(<LoginForm onSubmit={mockSubmit} />);
  
  await user.type(screen.getByLabelText(/email/i), 'test@example.com');
  await user.click(screen.getByRole('button', { name: /submit/i }));
  
  expect(mockSubmit).toHaveBeenCalledWith({
    email: 'test@example.com'
  });
});
```

### Testing Hooks

```javascript
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

it('should increment counter', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

### Testing with Context

```javascript
const renderWithTheme = (ui, { theme = 'light', ...options } = {}) => {
  const Wrapper = ({ children }) => (
    <ThemeProvider theme={theme}>
      {children}
    </ThemeProvider>
  );
  
  return render(ui, { wrapper: Wrapper, ...options });
};

it('should apply theme styles', () => {
  renderWithTheme(<Button>Click me</Button>, { theme: 'dark' });
  expect(screen.getByRole('button')).toHaveClass('dark-theme');
});
```

---

## Quick Reference Checklist

### Before Writing Tests
- [ ] Understand what behavior you're testing
- [ ] Plan your test data and mocks
- [ ] Consider edge cases and error scenarios

### While Writing Tests
- [ ] Use descriptive test names
- [ ] Follow AAA pattern (Arrange, Act, Assert)
- [ ] Test behavior, not implementation
- [ ] Keep tests focused and isolated
- [ ] Use appropriate matchers

### After Writing Tests
- [ ] Run tests multiple times to ensure consistency
- [ ] Check coverage reports
- [ ] Review for flaky or slow tests
- [ ] Ensure tests pass in CI/CD environment

### Code Review Checklist
- [ ] Tests are readable and maintainable
- [ ] No hard-coded values or magic numbers
- [ ] Proper cleanup in teardown methods
- [ ] Mocks are restored properly
- [ ] Tests cover happy path and edge cases

---

## Useful Jest Matchers

```javascript
// Basic matchers
expect(value).toBe(4);
expect(value).toEqual({ name: 'John' });
expect(value).toBeNull();
expect(value).toBeDefined();
expect(value).toBeTruthy();
expect(value).toBeFalsy();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeCloseTo(3.14, 2);

// Strings
expect(value).toMatch(/pattern/);
expect(value).toContain('substring');

// Arrays
expect(array).toContain(item);
expect(array).toHaveLength(3);

// Objects
expect(object).toHaveProperty('key');
expect(object).toMatchObject({ subset: 'value' });

// Functions
expect(fn).toThrow();
expect(fn).toHaveBeenCalled();
expect(fn).toHaveBeenCalledWith(arg1, arg2);

// Promises
expect(promise).resolves.toBe(value);
expect(promise).rejects.toThrow();
```

---

## Additional Resources

- [Jest Official Documentation](https://jestjs.io/docs/getting-started)
- [React Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/)
- [jest-extended Matchers](https://github.com/jest-community/jest-extended)
- [Testing JavaScript Course](https://testingjavascript.com/)

---

*Remember: Good tests are investments in your codebase's future. They should be readable, maintainable, and provide confidence in your code's behavior.*