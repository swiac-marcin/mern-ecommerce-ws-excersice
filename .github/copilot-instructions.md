# Copilot Instructions for MERN Ecommerce

## Quick Start

### Installation
Both frontend and backend must be installed separately:
```bash
# Backend
cd backend && npm install

# Frontend  
cd frontend && npm install
```

### Environment Setup
Create `.env` files in both directories:
- **Backend** (`backend/.env`): Requires `MONGO_URI`, `ORIGIN`, `EMAIL`, `PASSWORD`, `LOGIN_TOKEN_EXPIRATION`, `OTP_EXPIRATION_TIME`, `PASSWORD_RESET_TOKEN_EXPIRATION`, `COOKIE_EXPIRATION_DAYS`, `SECRET_KEY`, `PRODUCTION`
- **Frontend** (`frontend/.env`): Requires `REACT_APP_BASE_URL="http://localhost:8000"`

### Running Development Servers
Use separate terminals or split terminal:
```bash
# Terminal 1 - Backend
cd backend && npm run dev

# Terminal 2 - Frontend
cd frontend && npm start
```

### Testing & Building
```bash
# Frontend
npm run test          # Run Jest tests
npm run build         # Production build

# Backend
npm run seed          # Populate database with sample data
npm start             # Production run (use npm run dev for development)
```

## Architecture

### MERN Stack
- **MongoDB**: Document database (local connection required)
- **Express.js**: Backend API server (port 8000)
- **React 18**: Frontend UI with Material UI
- **Node.js**: Backend runtime

### Backend Architecture (`/backend`)
**Pattern**: Controllers → Models → Routes

- **routes/**: Express route handlers mounting to `/auth`, `/users`, `/products`, `/orders`, `/cart`, `/brands`, `/categories`, `/address`, `/reviews`, `/wishlist`
- **controllers/**: Request handlers for each feature (Auth.js, Product.js, Order.js, etc.)
- **models/**: Mongoose schemas (User.js, Product.js, Cart.js, Order.js, Review.js, Wishlist.js, etc.)
- **middleware/VerifyToken.js**: JWT authentication middleware - extracts token from cookies and validates before passing `req.user` to protected routes
- **utils/**: Helper functions like `SanitizeUser` (strips sensitive fields), `GenerateToken` (JWT creation), `GenerateOtp` (OTP generation), `Emails` (nodemailer integration)
- **database/db.js**: MongoDB connection initialization
- **seed/**: Sample data population script

### Frontend Architecture (`/frontend`)
**Pattern**: Redux Toolkit → API slices → Components/Pages

- **features/**: Redux slices for state management
  - Each feature (auth, products, cart, order, etc.) has `*Slice.jsx` (Redux Toolkit) and `*Api.jsx` (API calls)
  - State pattern: `status` (idle/pending/fulfilled/rejected), `error`, and domain-specific state
  - API calls use axios with `REACT_APP_BASE_URL` base URL
- **pages/**: Page components (routed views)
- **layout/**: Reusable layout wrappers
- **components/**: Feature-specific components
- **hooks/**: Custom React hooks
- **theme/**: Material UI theme configuration
- **assets/**: Images and static files
- **config/**: Configuration files
- **constants/**: Constants used across the app

## Key Conventions

### Backend
- **Error Handling**: Controllers catch errors with try/catch and return status codes with JSON messages
- **Cookie Management**: JWT tokens stored in httpOnly cookies with sameSite/secure settings based on `PRODUCTION` env var
- **Password Security**: Passwords hashed with bcryptjs (10 salt rounds)
- **Email Verification**: OTP-based verification for signup, password reset via nodemailer
- **User Sanitization**: Always use `sanitizeUser()` before returning user objects to strip passwords
- **Environment-Based Logic**: Check `process.env.PRODUCTION === 'true'` for production-specific behavior (cookie flags, CORS settings)

### Frontend
- **Redux State Structure**: Consistent pattern with `status`, `*Status`, `errors`, `successMessage`
- **API Integration**: `AuthApi.jsx`, `ProductApi.jsx`, etc. handle all API calls, while `*Slice.jsx` manages state
- **Async Thunks**: Use `createAsyncThunk` for async operations, mapped to slice actions
- **Material UI**: All components use `@mui/material` with Material UI theming
- **Form Handling**: `react-hook-form` for form validation and management
- **Toast Notifications**: `react-toastify` for user feedback
- **Route Structure**: `react-router-dom` v6 with nested routes in pages/

### Authentication Flow
- JWT tokens stored in httpOnly cookies (not localStorage)
- `verifyToken` middleware checks cookies on protected backend routes
- Frontend checks `loggedInUser` in Redux auth state
- Token expiration handled client-side with checkAuth endpoint

### CORS & API Communication
- Backend CORS configured to accept `process.env.ORIGIN` with credentials
- Frontend base URL: `process.env.REACT_APP_BASE_URL`
- Cookies included in all requests (axios + Redux middleware setup)
- Custom header exposed: `X-Total-Count` (for pagination)

## Database

### MongoDB Connection
Requires local MongoDB instance running at `MONGO_URI` (default: `mongodb://localhost:27017/your-database-name`)

### Collections
- **users**: Stores user accounts with email, password (hashed), verification status, admin flag
- **products**: Product catalog with attributes (name, stock, etc.)
- **orders**: Order history with user reference and status
- **carts**: Shopping carts with product items
- **reviews**: Product reviews with ratings
- **wishlists**: User wish lists with product notes
- **otps**: Temporary OTP records for email verification
- **passwordresettokens**: Temporary tokens for password reset flow

## Testing & Debugging

### Manual Testing
Use demo account after seeding:
- Email: `demo@gmail.com`
- Password: `helloWorld@123`
- Note: Password reset/OTP features don't work with demo account (non-real email)

### Development Tips
- Use `npm run seed` to populate fresh test data
- Check backend logs with Morgan (configured as "tiny" format)
- Frontend Redux DevTools compatible for state debugging
- JWT token in cookies viewable in browser DevTools under Application → Cookies
