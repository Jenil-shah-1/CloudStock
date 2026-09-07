# ☁️📦 CloudStock — E-Commerce Microservices

> Full-featured microservices e-commerce app on AWS ECS Fargate with Stripe payments, admin dashboard, category management, customer portal, and CloudWatch monitoring. 1-click deploy via GitHub Actions.

---

## Live Demo

![CloudStock Live Demo](https://raw.githubusercontent.com/Jenil-shah-1/CloudStock/main/docs/demo.jpg)

---

## Architecture

![CloudStock Architecture](https://raw.githubusercontent.com/Jenil-shah-1/CloudStock/main/docs/CloudStock_Architecture.png)

---

## Services (1 EC2 Instance - t2.micro Free Tier)

| Service | Port | Handles | Tech |
|---------|------|---------|------|
| Frontend | 80 | UI — shop, cart, checkout, admin panel, customer portal | React + Recharts + Stripe Elements + Nginx |
| Product Service | 4001 | Products CRUD + Cart + Categories | Node.js/Express |
| Order Service | 4002 | Orders + Payments + Auth + Analytics | Node.js/Express + Stripe SDK |

---

## Features

### Customer Experience
- Hero banner, category navigation (scrollable icons)
- Product catalog — 4-column grid, star ratings, search by name/category
- Hot Deals, Trending slider, product detail modal with reviews
- Shopping cart → Checkout (name, email, phone, address + Stripe)
- My Orders — Email login, order progress tracker, printable receipts
- Trust bar, promo banner, mobile responsive (hamburger menu), back to top

### Admin Panel (Protected — no header, full-screen layout)
- **Login** — Username/password authentication (`admin` / `CloudStock2026`)
- **Dashboard** — Stats cards (📊 Total Orders, 💰 Paid, 🚨 Failed, 🚀 Products Live)
- **Analytics charts** — Revenue Over Time (area chart) + Revenue Breakdown (bar chart)
- **Revenue** includes paid + shipped + delivered orders
- **Time range selector** — 10m, 1h, 4h, 6h, 12h, 1d, 3d
- **Products CRUD** — Add/edit/delete products on dedicated form page, category dropdown
- **Categories management** — Add/edit/delete categories with custom icon URLs, product count per category
- **Orders management** — Filter by status (All/Paid/Pending/Failed/Shipped/Delivered), update status
- **Logout** — Session-based admin auth

---

## 1-Click Deploy to AWS

### Prerequisites
- AWS account with `AdministratorAccess` IAM user
- GitHub repo forked/cloned

### Setup (once)

Add secrets to your GitHub repo → Settings → Secrets → Actions:

| Secret | Value |
|--------|-------|
| `AWS_ACCESS_KEY_ID` | Your IAM access key |
| `AWS_SECRET_ACCESS_KEY` | Your IAM secret key |
| `DB_PASSWORD` | Any password — letters + numbers only (e.g. `CloudStock2026Strong`) |
| `STRIPE_SECRET_KEY` | Stripe test secret key (`sk_test_...`) |
| `STRIPE_PUBLISHABLE_KEY` | Stripe test publishable key (`pk_test_...`) |

### Deploy

1. Go to **Actions** → **🚀 Deploy CloudStock**
2. Click **Run workflow** → select `deploy`
3. Wait ~15 min → get ALB URL in the summary ✅

### What happens automatically:
```
Step 1: Provisions AWS infra (VPC, ALB, ECS, RDS, ECR, Cloud Map, CloudWatch)
Step 2: Builds Docker images (linux/amd64)
Step 3: Pushes images to ECR
Step 4: Runs db-init ECS task (loads schema + seed data)
Step 5: Deploys services sequentially (waits for each to be stable)
Step 6: Verifies /products, /orders/stats/summary
Step 7: Outputs ALB URL ✅
```

### Destroy

Same workflow → select `destroy` → all resources deleted.

---

## Run Locally

```bash
# Set Stripe test keys in .env
cat > .env << EOF
STRIPE_SECRET_KEY=sk_test_your_key
REACT_APP_STRIPE_PUBLISHABLE_KEY=pk_test_your_key
EOF

docker compose up --build
```

Open http://localhost:3000

- **Admin Panel:** Click Admin → Login with `admin` / `CloudStock2026`
- **My Orders:** Click My Orders → Enter the email used during checkout
- **Test card:** `4242 4242 4242 4242` | Any future expiry | Any CVC

---

## Test Cards (Stripe)

| Card Number | Result |
|-------------|--------|
| `4242 4242 4242 4242` | Payment succeeds ✅ |
| `4000 0000 0000 0002` | Card declined ❌ |
| `4000 0000 0000 9995` | Insufficient funds ❌ |
| `4000 0000 0000 0069` | Expired card ❌ |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, Recharts, Stripe Elements, Nginx |
| Backend | Node.js, Express, Stripe SDK |
| Database | MySQL 8.0 (RDS) |
| Payments | Stripe (test mode) |
| Charts | Recharts (admin panel) |
| Icons | Icons8 Fluency (CDN) |
| Monitoring | CloudWatch |
| Containers | Docker, 1 EC2 Instance |
| Networking | VPC, ALB, NAT Gateway, ECS Service Connect (Cloud Map) |
| Registry | Amazon ECR |
| CI/CD | GitHub Actions |

---

## Project Structure

```
CloudStock/
├── frontend/              # React SPA + Nginx (shop, admin, customer portal)
├── product-service/       # Products CRUD + Cart + Categories
├── order-service/         # Orders + Payments + Auth + Analytics
├── db-init/               # DB migration container (runs once)
├── database/              # SQL schema + seed data (15 products, 11 categories)
├── .github/workflows/     # CI/CD pipeline
├── docs/                  # Architecture diagrams + documentation
├── docker-compose.yml     # Local development
└── .env                   # Local Stripe keys (gitignored)
```

---

## Database Schema

| Table | Purpose |
|-------|---------|
| `categories` | Category name, icon, image URL |
| `products` | Name, description, price, image, category, stock |
| `users` | Email, name |
| `cart_items` | User cart (user_id, product_id, quantity) |
| `orders` | Order with shipping details + status |
| `order_items` | Products in each order |
| `payments` | Payment records (amount, status, method) |

---

## API Endpoints

### Product Service (port 4001)
| Method | Path | Description |
|--------|------|-------------|
| GET | /products | List all products |
| GET | /products/:id | Get product |
| POST | /products | Create product (admin) |
| PUT | /products/:id | Update product (admin) |
| DELETE | /products/:id | Safe delete (soft-delete if has orders) |
| GET | /categories | List all categories |
| POST | /categories | Create category (admin) |
| PUT | /categories/:id | Update category (admin) |
| DELETE | /categories/:id | Delete category (admin) |
| GET | /cart/:userId | Get cart items |
| POST | /cart | Add to cart |
| DELETE | /cart/:id | Remove from cart |

### Order Service (port 4002)
| Method | Path | Description |
|--------|------|-------------|
| POST | /auth/admin | Admin login |
| GET | /orders/stats/summary | Dashboard stats |
| GET | /orders/stats/timeseries | Revenue chart data (query: ?minutes=60) |
| GET | /orders/all | All orders (admin) |
| GET | /orders/by-email/:email | Customer orders |
| GET | /orders/:userId | User orders |
| POST | /orders | Create order |
| PUT | /orders/:id/status | Update order status (validates: pending/paid/failed/shipped/delivered) |
| POST | /payments/create-intent | Stripe payment intent |
| POST | /payments/confirm | Confirm payment |
| POST | /payments/failed | Log failed payment |

---

## User Flow

1. **Browse** — Hero banner → Category strip → Product grid with search
2. **Filter** — Click category or search by name/category
3. **View** — Click product → Modal with details, ratings, reviews
4. **Cart** — Add items, view cart, proceed to checkout
5. **Pay** — Fill shipping details + Stripe card → Payment processed
6. **Track** — My Orders → Email login → Order progress tracker + receipts
7. **Admin** — Login → Dashboard → Manage products, categories, orders
8. **Monitor** — CloudWatch dashboard → Real-time order monitoring

---

## Default Categories

Mobile, Laptop, Television, Earpods, Kitchen, Accessories, Cameras, Fans, Grooming, Storage, Air Conditioners (11 total — manageable from admin panel)

---

## Cost (~$0/month on Free Tier)

| Resource | Cost |
|----------|------|
| EC2 t2.micro (1 Instance) | ~$0 (Free Tier) |
| RDS db.t3.micro | ~$0 (Free Tier) |
| ALB | ~$0 (Free Tier allowance) |
| ECR + S3 | ~$0 (Free Tier) |
| **Total** | **~$0/month** |

---

## Monitoring (CloudWatch)

### CloudWatch Dashboard
```
https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#dashboards:name=cloudstock-orders
```

### Structured Log Events

| Event | Trigger | Fields |
|-------|---------|--------|
| `ORDER_PENDING` | Order created | order_id, user_id, amount, customer, email, reason |
| `ORDER_BOOKED` | Payment succeeded | order_id, user_id, amount, customer, email, reason |
| `ORDER_FAILED` | Payment failed | order_id, user_id, amount, reason, stripe_status |
| `ORDER_ERROR` | Exception | order_id, error |

---

## Security

- ECS tasks in **private subnets** — no public IPs
- RDS in **private subnets** (`publicly_accessible = false`)
- Deployed in Public Subnets to avoid expensive NAT Gateway costs (Student-friendly)
- ALB is the only internet-facing resource (public subnets, port 80)
- ECS security group allows inbound only from ALB
- **Request body size limit** — 1MB max on all endpoints
- **Input validation** — Order status whitelist (pending/paid/failed/shipped/delivered)
- **Input sanitization** — All user inputs trimmed and length-capped
- **Safe product delete** — Soft-delete (stock=0) if product has order history
- Admin panel protected by username/password authentication
- DB password stored as GitHub Secret — never in code
- Stripe keys stored as GitHub Secrets — never in code
- Stripe test mode — no real charges
- **DB connection hardening** — 30s connect timeout + keepAlive enabled

---

## Credits

© 2026 CloudStock. Proudly built by [**Jenil Shah**](https://github.com/Jenil-shah-1)
