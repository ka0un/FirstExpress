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


## Components, Responsibilities and Examples

Each component has a specific responsibility:
- **Config**: Manages environment-specific settings
- **Models**: Defines data structure and database schema
- **Controllers**: Handles HTTP requests/responses
- **Services**: Contains business logic
- **Middleware**: Processes requests before reaching routes
- **Routes**: Defines API endpoints and their handlers
- **Utils**: Provides reusable helper functions
- **App Entry**: Sets up the application and middleware

1. **Config (src/config/database.config.ts)**
```typescript
// Handles all configuration settings
export const dbConfig = {
  mongodb: {
    url: process.env.MONGODB_URI || 'mongodb://localhost:27017/myapp',
    options: {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      maxPoolSize: 10
    }
  },
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT || '6379')
  }
};
```

2. **Model (src/models/user.model.ts)**
```typescript
// Defines data structure and database interactions
import mongoose, { Schema, Document } from 'mongoose';

export interface IUser extends Document {
  email: string;
  password: string;
  name: string;
  createdAt: Date;
}

const UserSchema = new Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  name: { type: String, required: true },
  createdAt: { type: Date, default: Date.now }
});

export const User = mongoose.model<IUser>('User', UserSchema);
```

3. **Controller (src/controllers/auth.controller.ts)**
```typescript
// Handles request/response logic
import { Request, Response } from 'express';
import { AuthService } from '../services/auth.service';

export class AuthController {
  private authService: AuthService;

  constructor() {
    this.authService = new AuthService();
  }

  public async login(req: Request, res: Response): Promise<void> {
    try {
      const { email, password } = req.body;
      const token = await this.authService.loginUser(email, password);
      res.status(200).json({ token });
    } catch (error) {
      res.status(401).json({ message: 'Invalid credentials' });
    }
  }
}
```

4. **Service (src/services/auth.service.ts)**
```typescript
// Contains business logic
import { User } from '../models/user.model';
import { comparePasswords, generateToken } from '../utils/auth.utils';

export class AuthService {
  public async loginUser(email: string, password: string): Promise<string> {
    const user = await User.findOne({ email });
    if (!user || !await comparePasswords(password, user.password)) {
      throw new Error('Invalid credentials');
    }
    return generateToken(user._id);
  }
}
```

5. **Middleware (src/middleware/auth.middleware.ts)**
```typescript
// Handles intermediate processing of requests
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

export const authMiddleware = async (
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) throw new Error();
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ message: 'Unauthorized' });
  }
};
```

6. **Route (src/routes/auth.routes.ts)**
```typescript
// Defines API endpoints
import { Router } from 'express';
import { AuthController } from '../controllers/auth.controller';
import { validateLogin } from '../middleware/validation.middleware';

const router = Router();
const authController = new AuthController();

router.post('/login', validateLogin, authController.login.bind(authController));
router.post('/logout', authController.logout.bind(authController));

export const authRoutes = router;
```

7. **Utils (src/utils/auth.utils.ts)**
```typescript
// Helper functions and utilities
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';

export const comparePasswords = async (
  password: string,
  hash: string
): Promise<boolean> => {
  return bcrypt.compare(password, hash);
};

export const generateToken = (userId: string): string => {
  return jwt.sign({ userId }, process.env.JWT_SECRET!, {
    expiresIn: '24h'
  });
};
```

8. **App Entry (src/app.ts)**
```typescript
// Main application setup
import express from 'express';
import mongoose from 'mongoose';
import { dbConfig } from './config/database.config';
import { authRoutes } from './routes/auth.routes';
import { errorHandler } from './middleware/error.middleware';

const app = express();

mongoose.connect(dbConfig.mongodb.url, dbConfig.mongodb.options);

app.use(express.json());
app.use('/api/auth', authRoutes);
app.use(errorHandler);

export { app };
```

