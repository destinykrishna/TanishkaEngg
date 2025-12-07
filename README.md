# Tanishka Engineering Platform

A full‑stack monorepo powering the public Tanishka Engineering website and a secure admin operations panel with advanced photo management and carousel features.

> Backend: Node.js (Express 4.19.2) + MongoDB + ImageKit + Email Service  \
> Admin Frontend: React 18.3.1 + Vite + Tailwind (port 5174)  \
> User Frontend: React 18.3.1 + Vite + Tailwind (port 5173)

---
## ✨ Key Features
- **Advanced Photo Management**: Multi-image upload with selective removal and smart validation (up to 4 images per item).
- **Interactive Carousels**: Beautiful image galleries with drag/swipe support, keyboard navigation, and fullscreen viewing.
- **Content & Portfolio Management**: CRUD for Services, Projects, and About content with rich media support.
- **Email Notifications**: Automated email system for contact form submissions with dual notifications (user confirmation + admin notification).
- **Inquiry Capture**: Rate‑limited contact form endpoint with email integration to mitigate spam.
- **Secure Admin Auth**: Cookie (httpOnly) based session with robust authentication middleware.
- **Media Handling**: Multer in‑memory upload + ImageKit integration with incremental photo management.
- **Granular Access Control**: Middleware‑gated create/update/delete routes.
- **Responsive Admin UI**: Collapsible sidebar, protected routes, dashboard metrics, and intuitive photo management.
- **Optimized User Experience**: Smooth animations, loading states, and error handling throughout.
- **Scalable Structure**: Clear separation of controllers, models, routes, middlewares, and services.
- **Environment Isolation**: Frontends consume API via configurable base URL / port.
- **Featured Content System**: Admin control to select which services, projects, and testimonials appear on home page.
- **Dynamic Home Page**: Services, projects, and testimonials pulled from API with real-time filtering and sorting.
- **Admin Profile Management**: Admin photo upload with ImageKit integration and profile customization.
- **Testimonial Management**: Dynamic testimonials with featured flag and display order control.
- **Tag System**: Content tagging for services and projects with admin interface.
- **Backward Compatibility**: Smart database queries that work with existing data missing new fields.
- **AI-Powered Chatbot**: Real-time chat support using Groq AI with multi-language support (EN, AR, HI, DE, ES) and intelligent fallback mode.

---
## 🗂️ Monorepo Structure
```
TanishkaEngineering/
├─ backend/                # Express API & MongoDB models
│  ├─ src/
│  │  ├─ controllers/      # Business logic per resource
│  │  ├─ models/           # Mongoose schemas
│  │  ├─ routes/           # REST route modules
│  │  ├─ middlewares/      # Auth & rate limiting
│  │  ├─ services/         # External integrations (e.g., storage)
│  │  ├─ db/               # DB connection bootstrap
│  │  └─ app.js / server.js
│  └─ .env.example         # Backend environment template
│
├─ frontend-admin/         # Admin control panel (React + Vite)
│  └─ src/
│     ├─ context/          # Auth provider (cookie-aware)
│     ├─ components/       # UI components & layout
│     ├─ pages/            # Feature pages (CRUD, Dashboard, etc.)
│     └─ App.jsx / main.jsx
│
├─ frontend-user/          # Public website UI with advanced carousel features
│  └─ src/
│     ├─ components/       # Reusable UI & layout (includes ImageCarousel)
│     ├─ pages/            # Public-facing pages with optimized galleries
│     └─ routes/           # App route wiring
│
└─ README.md               # This document
```

---
## 🔐 Authentication Flow
| Step | Action | Notes |
|------|--------|-------|
| 1 | Admin submits credentials (`POST /api/auth/admin/login`) | Server sets httpOnly session cookie on success. |
| 2 | Frontend stores lightweight `adminSession` flag (no token required) | Enables stateful UI without reading cookie directly. |
| 3 | Protected actions send cookie automatically (`withCredentials: true`) | Axios instance configured accordingly. |
| 4 | Logout (`GET /api/auth/admin/logout`) | Server clears cookie; client resets auth context. |

Future enhancement: add `/api/auth/admin/me` to hydrate admin profile on refresh (optional).

---
## 🧪 Core API Endpoints (Summary)
Base API Prefix: `http://localhost:3000/api`

### Auth
| Method | Path | Purpose |
|--------|------|---------|
| POST | /auth/admin/register | (Optional seed) Create first admin. |
| POST | /auth/admin/login | Login & set session cookie. |
| GET  | /auth/admin/logout | Destroy session. |

### Services
| Method | Path | Description |
|--------|------|-------------|
| GET | /get-all-data | List all services (public). Supports query params: `featured=true`, `limit=3`, `search=query`, `tags=tagId` |
| POST | /create | Add service (admin, multipart field: `photo[]`, up to 4 images). Supports `isActive`, `isFeatured`, `order` fields. |
| PUT | /update/:id | Update service with incremental photo management and selective removal. Supports featured/active/order updates. |
| DELETE | /delete/:id | Remove service. |

### Projects
| Method | Path | Description |
|--------|------|-------------|
| GET | /projects | List all projects (public). Supports query params: `featured=true`, `limit=3`, `search=query`, `tags=tagId` |
| POST | /add/projects | Create project (admin, field: `photos[]`, up to 4 images). Supports `isActive`, `isFeatured`, `order` fields. |
| PUT | /update-projects/:id | Update project with advanced photo management and selective removal. Supports featured/active/order updates. |
| DELETE | /delete-projects/:id | Remove project. |

### Testimonials
| Method | Path | Description |
|--------|------|-------------|
| GET | /testimonials | List active testimonials (public). Supports `featured=true` and `limit=2` for home page display. |
| GET | /testimonials/admin/all | List all testimonials (admin). |
| POST | /testimonials | Add testimonial (admin). Supports `isActive`, `isFeatured`, `order` fields. |
| PUT | /testimonials/:id | Update testimonial including featured flag and order. |
| DELETE | /testimonials/:id | Remove testimonial. |

### Admin Profile
| Method | Path | Description |
|--------|------|-------------|
| GET | /profile | Get admin profile data. |
| PUT | /profile | Update admin profile with photo upload support (multipart field: `profilePhoto`). |
| PUT | /profile/change-password | Change admin password. |

### About
| Method | Path | Description |
|--------|------|-------------|
| GET | /aboutUs | Fetch about content. |
| PUT | /update | Update about content (admin). |

### Inquiries
| Method | Path | Description |
|--------|------|-------------|
| POST | /contact-form/submit | Submit inquiry (rate limited, sends confirmation email to user and notification to admin). |
| GET | /contact-form/all-data | Admin list of inquiries. |

### Chatbot
| Method | Path | Description |
|--------|------|-------------|
| POST | /chatbot/chat | Send message to AI chatbot. Request body: `{ "message": "...", "conversationHistory": [], "language": "en" }`. Response: `{ "success": true, "message": "...", "isDemoMode": false }` |
| GET | /chatbot/suggestions | Get suggested questions for the user. Query param: `language=en` (default). Response: `{ "suggestions": [...] }` |

> Authentication middleware: `authAdminMiddleware` protects all mutating admin routes.

---
## 🤖 AI-Powered Chatbot System
The platform includes an intelligent, multi-language AI chatbot powered by Groq AI's `llama-3.3-70b-versatile` model for fast, reliable responses.

### Features
- **Real-time AI Responses**: Instant conversational replies using Groq's latest LLM (free & unlimited)
- **Multi-Language Support**: Automatic response generation in English, Arabic, Hindi, German, and Spanish
- **Conversation History**: Maintains context across messages with intelligent history management (last 10 messages)
- **Suggested Questions**: Smart suggestions updated dynamically based on language selection
- **Intelligent Fallback**: Demo mode with pre-configured responses if API is unavailable
- **Company-Specific Knowledge**: System prompts configured with Tanishka Engineering services and details
- **Service Information**: Chatbot provides details about HVAC, cooling systems, clean room solutions
- **Lead Generation**: Handles consultation requests and contact information capture
- **Rate Limiting**: Chatbot-specific rate limiting to prevent abuse

### Configuration
Backend requires Groq API key in `.env`:
```env
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxx
```

Get a free API key at [Groq Console](https://console.groq.com/keys)

### How It Works
1. **Frontend Request**: User sends message via chat UI
2. **Vite Proxy**: Frontend proxies request to `/api/chatbot/chat` (configured in `vite.config.js`)
3. **Backend Processing**: 
   - Validates message and language
   - Builds conversation context with system prompt
   - Calls Groq API with `llama-3.3-70b-versatile` model
   - Falls back to demo response if Groq unavailable
4. **Response**: Returns AI-generated message with conversation ID

### Response Examples
```json
{
  "success": true,
  "message": "Hello! Thank you for reaching out to Tanishka Engineering...",
  "conversationId": null,
  "isDemoMode": false
}
```

### Supported Languages
- `en` - English
- `ar` - Arabic (العربية)
- `hi` - Hindi (हिन्दी)
- `de` - German (Deutsch)
- `es` - Spanish (Español)

### Technical Details
- **Model**: `llama-3.3-70b-versatile` (Latest Groq model, 70B parameters)
- **Temperature**: 0.7 (balanced creativity/accuracy)
- **Max Tokens**: 1024 (response length limit)
- **Initialization**: Lazy-loaded after environment variables are ready
- **Error Handling**: Graceful fallback to demo mode with company contact info

### Demo Mode
When Groq API key is missing or API is unavailable, the chatbot automatically responds with pre-configured messages about:
- HVAC services and capabilities
- Cooling systems and infrastructure
- Clean room solutions
- Project consultation inquiries
- Company contact information

This ensures the chatbot is always functional, even during development or API outages.

### Integration Points
- **Frontend**: `frontend-user/src/components/ChatBot.jsx`
- **Backend Controller**: `backend/src/controllers/chatbot.controller.js`
- **Backend Routes**: `backend/src/routes/chatbot.routes.js`
- **Rate Limiter**: `backend/src/middlewares/chatbotRateLimiter.middlewares.js`
- **Frontend Proxy**: `frontend-user/vite.config.js` (API proxy configuration)

---
## 🎯 Featured Content System
The platform includes a powerful featured content management system that allows admins to control what appears on the home page without requiring code changes.

### How It Works
1. **Admin Panel Selection**: Mark services, projects, or testimonials as "Featured" via checkbox in the respective admin managers.
2. **Ordering**: Set an `order` field to control display sequence (lower numbers appear first).
3. **Status Control**: Use `isActive` flag to show/hide content without deletion.
4. **Home Page Fetching**: The public home page automatically fetches and displays featured items:
   - Services: `/api/get-all-data?featured=true&limit=3`
   - Projects: `/api/projects?featured=true&limit=3`
   - Testimonials: `/api/testimonials?featured=true&limit=2`

### Database Fields
All content types (services, projects, testimonials) support:
- `isActive` (Boolean): Controls visibility across the site
- `isFeatured` (Boolean): Marks content for home page display
- `order` (Number): Controls display order (1, 2, 3... ascending)

### Backward Compatibility
The system intelligently handles existing content:
- Documents without `isActive` field are treated as `isActive: true`
- Featured filtering requires explicit `isFeatured: true` in database
- No migration needed; gradual adoption as you update content

---
## 📋 Recent Updates (Latest Session)

### ✅ Completed Features
1. **AI-Powered Chatbot Integration**
   - Integrated Groq AI (`llama-3.3-70b-versatile` model) for unlimited free chatbot responses
   - Multi-language support (EN, AR, HI, DE, ES) with language-specific system prompts
   - Conversation history management with context preservation (last 10 messages)
   - Intelligent fallback demo mode for development/API unavailability
   - Chatbot-specific rate limiting to prevent abuse
   - Real-time suggested questions updated per language
   - Vite proxy configuration for seamless frontend-backend communication
   - Lazy client initialization to ensure environment variables load correctly

2. **Featured Content System**
   - Added `isActive`, `isFeatured`, and `order` fields to services and projects models
   - Admin managers (ServiceManager, ProjectManager) now include featured/active/order controls
   - Backend controllers support `?featured=true&limit=N` query filtering
   - Home page dynamically fetches featured items with proper sorting

2. **Dynamic Home Page**
   - Services, projects, and testimonials now pull from API instead of mock data
   - Proper data extraction handling both paginated and direct array responses
   - Real-time updates reflecting admin panel changes

3. **Admin Profile Photo Upload**
   - Admin can upload profile photo via camera icon in profile page
   - Photo preview displayed before upload
   - ImageKit integration with automatic old photo cleanup
   - Full multipart form-data support with file validation

4. **Route & Navigation Fixes**
   - Separated service routes: `/services/:id` for HVAC, `/service-types/:type/:id` for consulting/contracting
   - Fixed project navigation to use routing instead of modals
   - Proper route ordering to prevent catch-all route conflicts

### 🔧 Technical Improvements
- Simplified MongoDB filter logic to match testimonials pattern for consistency
- Enhanced error handling and logging in profile controller
- Better form data handling with explicit field appending
- Improved file validation (type and size checks)
- Fixed multipart form-data header handling in axios requests

---
## 📸 Photo Management System
The platform features an advanced photo management system with the following capabilities:

### Admin Panel Features
- **Multi-Image Upload**: Select and upload multiple images simultaneously (up to 4 per item)
- **Incremental Addition**: Add new photos to existing collections without replacing current ones
- **Selective Removal**: Remove individual photos while keeping others intact
- **Smart Validation**: 
  - Maximum 4 images per service/project
  - File size limits (2MB for services, 3MB for projects)
  - Image format validation
- **Real-time Preview**: Preview images before upload with remove functionality
- **Batch Operations**: Handle multiple file operations efficiently

### User Experience Features
- **Interactive Carousel**: 
  - Touch/swipe navigation on mobile devices
  - Keyboard arrow key support
  - Drag gesture support with momentum
  - Smooth transitions and animations
- **Fullscreen Gallery**: Click any image to view in fullscreen mode
- **Auto-play**: Optional auto-advancing slideshow for services
- **Navigation Indicators**: 
  - Dot indicators for easy slide jumping
  - Image counter display (e.g., "2 / 4")
  - Thumbnail navigation for galleries with 3+ images
- **Responsive Design**: Optimized viewing on all device sizes
- **Loading States**: Skeleton loading and error handling

### Technical Implementation
- **Backend**: Incremental photo management with index-based removal
- **Frontend**: React components with Framer Motion animations
- **Storage**: ImageKit integration for optimized delivery
- **Validation**: Client and server-side validation for robust operation

---
## ⚙️ Environment Configuration
### Backend (`backend/.env`)
Copy from `.env.example` and fill values:
```
PORT=3000
MONGODB_URI=mongodb://localhost:27017/tanishka-consult-backend
JWT_SECRET=REPLACE_WITH_STRONG_SECRET
IMAGEKIT_PUBLIC_KEY=your_public_key
IMAGEKIT_PRIVATE_KEY=your_private_key
IMAGEKIT_URL_ENDPOINT=https://ik.imagekit.io/your_endpoint_id

# Email Configuration (Gmail SMTP)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_SECURE=false
EMAIL_USER=your_email@gmail.com
EMAIL_PASS=your_app_specific_password
EMAIL_FROM=your_email@gmail.com

# Groq AI Configuration (Chatbot)
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxx
```
> Never commit real secrets. The root `.gitignore` already protects `.env`.
> Get free Groq API key at https://console.groq.com/keys (unlimited free tier)

### Admin Frontend (`frontend-admin/.env`)
```
VITE_API_PORT=3000        # OR use VITE_API_URL=http://localhost:3000/api
```

### User Frontend (`frontend-user/.env`)
```
VITE_API_PORT=3000        # Aligns with backend; switch to VITE_API_URL if deploying elsewhere
```

If deploying behind a reverse proxy / domain, set `VITE_API_URL=https://api.example.com/api` and remove the port variable.

---
## 🚀 Local Development
### 1. Install Dependencies
```powershell
# From repo root
cd backend; npm install; cd ..
cd frontend-admin; npm install; cd ..
cd frontend-user; npm install; cd ..
```

### 2. Run Services (three terminals)
```powershell
# Terminal 1 - Backend
cd backend; npm run dev

# Terminal 2 - Admin Frontend
cd frontend-admin; npm run dev  # => http://localhost:5174

# Terminal 3 - User Frontend
cd frontend-user; npm run dev   # => http://localhost:5173
```

### 3. Login Flow
1. (If no admin) Manually enable and call `POST /api/auth/admin/register` or insert admin in DB.
2. Visit `http://localhost:5174/login` and sign in.
3. Perform CRUD operations & observe public site reflect changes.

---
## 🛠️ Notable Implementation Details
- **AI Chatbot Integration**:
  - Groq API integration with lazy-loading client initialization
  - Multi-language system prompts for company-specific responses
  - Fallback demo mode with intelligent keyword matching for offline functionality
  - Conversation history limited to last 10 messages to manage token usage
  - Rate limiting middleware specific to chatbot endpoints
  - Vite proxy configuration (`/api` → `http://localhost:3000`) for cross-origin requests
  - Frontend uses relative URLs (`/api/chatbot/chat`) for automatic proxy routing
- **Featured Content Management**: 
  - Admin checkboxes to mark services, projects, and testimonials as featured
  - Order field for custom display sequencing
  - API endpoints support `?featured=true&limit=X` filtering
  - Home page fetches featured items with proper sorting (by order, then recency)
  - Backward compatible: existing documents without new fields work seamlessly
- **Admin Profile Management**:
  - Admin can upload and update profile photo
  - Photo stored in ImageKit with old photo cleanup
  - Profile page displays preview of selected photo before upload
  - Full multipart/form-data support with FormData API
- **Dynamic Home Page**: 
  - Real-time content fetching from API with featured flag filtering
  - Proper data extraction from paginated and non-paginated responses
  - Sorting by custom order field for featured items
  - Fallback support for items without new fields
- **Email Service Integration**: 
  - Automated dual-email system for contact form submissions
  - User receives confirmation email with submission details
  - Admin receives notification email for new inquiries
  - Gmail SMTP integration with app-specific password support
  - Rate limiting to prevent email spam
- **Advanced Photo Management**: 
  - Multiple image uploads with batch selection
  - Selective photo removal without affecting other images
  - Smart validation (max 4 images, file size limits)
  - Incremental photo addition (add to existing without replacement)
- **Interactive Image Carousel**:
  - Drag/swipe navigation with touch support
  - Keyboard arrow key navigation
  - Fullscreen modal with zoom functionality
  - Auto-play option for service galleries
  - Thumbnail navigation for galleries with 3+ images
  - Smooth animations using Framer Motion
- **Axios Instance**: Centralized in `frontend-admin/src/context/AuthContext.jsx` with `withCredentials=true`.
- **Multer Memory Storage**: Files processed in memory then handed to ImageKit with optimized batch handling.
- **Enhanced Rate Limiting**: Applied to contact form and authentication endpoints to reduce abuse.
- **Field Normalization**: Services use form field `photo[]`, projects use `photos[]` (UI accounts for differences).
- **CORS Configuration**: Properly configured with credentials support for cross-origin requests.
- **UI/UX Enhancements**: Loading states, error handling, and responsive design throughout.
- **MongoDB Query Optimization**: Uses `$or` and `$and` operators for flexible filtering combining featured, search, and other criteria.

---
## 🔒 Security Checklist (Quick Pass)
| Item | Status | Recommendation |
|------|--------|---------------|
| HTTP Only Cookie | ✔ | Add `Secure` + `SameSite=strict` in production. |
| Rate Limiting | ✔ (contact form + auth) | Extend to more mutation endpoints. |
| Email Security | ✔ (app passwords) | Use OAuth2 for enhanced email security. |
| Input Validation | Partial | Add schema validation (Zod / Joi) server-side. |
| Error Handling | Basic | Centralize an error middleware for consistent responses. |
| Logging | Minimal | Introduce structured logger (pino / winston). |
| Secrets | .env | Rotate and store in a vault for production. |

---
## 🧱 Suggested Next Enhancements
- Email template customization with HTML formatting and branding.
- Email delivery status tracking and bounce handling.
- `/api/auth/admin/me` endpoint + automatic session hydration on page refresh.
- Toast/notification system for admin actions (partially implemented).
- Advanced image optimization and lazy loading.
- Bulk photo operations (select multiple for deletion).
- Image reordering via drag-and-drop in admin panel.
- Pagination & filtering enhancements for Inquiries / Projects.
- Role-based extension (multi-admin with permissions).
- Image metadata extraction (EXIF, dimensions, etc.).
- Progressive Web App (PWA) features for mobile experience.
- Unit & integration tests (Jest + Supertest backend, React Testing Library frontend).
- Content scheduling (schedule services/projects to appear on specific dates).
- Analytics dashboard for admin (view metrics, popular items, etc.).
- Multi-language support (i18n) for front-end content.

---
## 📦 Deployment Notes
| Layer | Suggestion |
|-------|------------|
| Backend | Containerize (Node 20 LTS base, healthcheck endpoint). |
| DB | Managed MongoDB (Atlas / DocumentDB). |
| Media | Continue ImageKit or migrate to S3 + CDN. |
| Frontends | Static hosting (Vercel / Netlify) with environment injection. |
| Reverse Proxy | Nginx/Caddy consolidating SSL + compression. |

Enable CORS only for approved origins in production (whitelist admin + public domains).

---
## 📝 Scripts Reference
| Location | Script | Purpose |
|----------|--------|---------|
| backend | `npm run dev` | Start API with nodemon. |
| backend | `npm start` | Start API (production mode). |
| frontend-* | `npm run dev` | Vite dev server. |
| frontend-* | `npm run build` | Production build (outputs `dist`). |
| frontend-* | `npm run preview` | Preview built assets locally. |
| frontend-* | `npm run lint` | Lint source code. |

---
## 🤝 Contribution Guide
1. Create a feature branch: `git checkout -b feat/short-feature-name`  
2. Make changes & ensure lint passes.  
3. Commit with conventional message: `feat(services): add partial update option`  
4. Open PR & request review.

---
## 📄 License
Add your chosen license (MIT recommended for open collaboration). Create a `LICENSE` file at root.

https://github.com/user-attachments/assets/f41bfacd-1b03-4a0a-a24c-e0b86903cb5b

---
## 🙋 Support / Contact
For internal use: reach the platform maintainer or open a GitHub issue.

---
> Built with clarity, scalability, and maintainability in mind.
