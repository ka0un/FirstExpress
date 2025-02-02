# Express JS

## Folder Structure

MVC (Model-View-Controller) pattern, "Express Generator Structure."
```text
├── src/             # Application source code
│   ├── config/      # Configuration files
│   ├── controllers/ # Route controllers
│   ├── models/      # Database models
│   ├── middleware/  # Custom middleware
│   ├── routes/      # Route definitions
│   ├── services/    # Business logic
│   ├── utils/       # Helper functions and utilities
│   └── views/       # Template files (if using server-side rendering)
```

## Naming conventions

| Type | Example | Convention |
|------|---------|------------|
| File name | `user.controller.ts` | Dot notation, lowercase |
| Interface | `IUserController` | PascalCase with 'I' prefix |
| Class | `UserController` | PascalCase |
| Methods | `getUsers`, `getUserById` | camelCase |
| Variables | `newUser`, `updatedUser` | camelCase |
| Parameters | `req`, `res` | camelCase |
| Response properties | `status`, `data`, `message` | camelCase |
