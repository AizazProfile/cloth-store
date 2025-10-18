# Online Cloth Store Backend

A highly concurrent, scalable backend API for an online clothing store built with Node.js, Express, and MongoDB. Features robust concurrency control, event-driven architecture, and real-time capabilities.

## 🌟 Features

### Core Features
- **Product Management**: Complete CRUD operations for clothing products
- **Shopping Cart**: Session-based cart management with real-time updates
- **Order Processing**: Full order lifecycle with payment integration
- **User Management**: Authentication and profile management
- **Real-time Updates**: WebSocket integration for live updates

### Concurrency Features
- **Race Condition Prevention**: Atomic stock operations prevent overselling
- **Deadlock Avoidance**: Smart resource locking with timeout mechanisms
- **Starvation Prevention**: FIFO queue system ensures fairness
- **Non-blocking I/O**: Asynchronous payment processing
- **Event-driven Architecture**: Real-time payment and order status updates
- **Thread Safety**: Optimistic locking and versioning for data consistency

### Technical Features
- **Comprehensive Monitoring**: System health and performance tracking
- **Structured Logging**: Detailed operation logs with rotation
- **Error Handling**: Robust error handling with proper HTTP status codes
- **Input Validation**: Schema validation using Joi
- **Security**: Helmet, rate limiting, and CORS protection

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   API Gateway   │    │   Database      │
│   (Client)      │◄──►│   (Express)     │◄──►│   (MongoDB)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │   Services      │
                       │   Layer         │
                       └─────────────────┘
                              │
                       ┌──────┼──────┐
                       ▼      ▼      ▼
               ┌──────────┐ ┌────────┐ ┌──────────┐
               │Concurrency│ │Payment │ │Monitoring│
               │ Manager   │ │Service │ │ Service  │
               └──────────┘ └────────┘ └──────────┘
```

## 🚀 Getting Started

### Prerequisites
- Node.js (v14 or higher)
- MongoDB (v4.4 or higher)
- npm or yarn

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd cloth-store/backend
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   ```bash
   cp .env .env.local
   # Edit .env.local with your configurations
   ```

4. **Start MongoDB**
   ```bash
   # Using MongoDB service
   sudo service mongod start
   
   # Or using Docker
   docker run -d -p 27017:27017 --name mongodb mongo:latest
   ```

5. **Seed the database**
   ```bash
   npm run seed
   ```

6. **Start the development server**
   ```bash
   npm run dev
   ```

The server will start on `http://localhost:5000`

## 📊 API Endpoints

### Products
- `GET /api/products` - Get all products with pagination and filtering
- `GET /api/products/:id` - Get single product by ID
- `GET /api/products/featured` - Get featured products
- `GET /api/products/categories` - Get product categories with counts
- `GET /api/products/:id/stock` - Get real-time stock information
- `POST /api/products/:id/reserve-stock` - Reserve stock atomically
- `POST /api/products/:id/release-stock` - Release reserved stock

### Shopping Cart
- `GET /api/cart` - Get current user's cart
- `POST /api/cart/add` - Add item to cart
- `PUT /api/cart/update/:itemId` - Update cart item quantity
- `DELETE /api/cart/remove/:itemId` - Remove item from cart
- `DELETE /api/cart/clear` - Clear entire cart
- `POST /api/cart/validate` - Validate cart items and stock
- `POST /api/cart/reserve` - Reserve cart items for checkout

### Orders
- `GET /api/orders` - Get orders for current session
- `GET /api/orders/:orderId` - Get single order details
- `POST /api/orders/create` - Create order from cart
- `GET /api/orders/:orderId/status` - Get order status
- `POST /api/orders/:orderId/cancel` - Cancel an order

### Users
- `POST /api/users/register` - Register new user
- `POST /api/users/login` - User login
- `GET /api/users/profile` - Get user profile (authenticated)
- `PUT /api/users/profile` - Update user profile
- `GET /api/users/addresses` - Get user addresses
- `POST /api/users/addresses` - Add new address

### Monitoring
- `GET /health` - System health check
- `GET /api/monitoring/report` - Comprehensive system report
- `GET /api/monitoring/performance/:operation?` - Performance statistics
- `GET /api/monitoring/concurrency` - Concurrency statistics

## 🧪 Testing

### Run All Tests
```bash
# Run comprehensive concurrency tests
npm run test:concurrency

# Run specific test suites
npm test
```

### Test Coverage
The test suite covers:
- **Race Condition Prevention**: Multiple users competing for limited stock
- **Deadlock Prevention**: Cross-resource locking scenarios
- **Atomic Operations**: Multi-step transaction rollback
- **Non-blocking I/O**: Asynchronous payment processing
- **Starvation Prevention**: Fair resource allocation
- **Performance Under Load**: High concurrent operation handling

### Load Testing
```bash
# Install artillery for load testing
npm install -g artillery

# Run load tests
artillery run tests/load-test.yml
```

## 🔧 Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | Server port | `5000` |
| `NODE_ENV` | Environment | `development` |
| `MONGODB_URI` | MongoDB connection string | `mongodb://localhost:27017/clothstore` |
| `JWT_SECRET` | JWT signing secret | Required |
| `SESSION_SECRET` | Session signing secret | Required |
| `LOG_LEVEL` | Logging level (error/warn/info/debug) | `info` |
| `RATE_LIMIT_WINDOW_MS` | Rate limiting window | `900000` (15 min) |
| `RATE_LIMIT_MAX_REQUESTS` | Max requests per window | `100` |

### Database Configuration

MongoDB is configured with:
- Connection pooling (max 10 connections)
- Optimized timeouts for concurrency
- Proper indexes for performance
- Automatic reconnection handling

## 🔒 Concurrency Implementation

### Stock Management
```javascript
// Atomic stock reservation
const result = await Product.reserveStock(productId, quantity, session);

// Within transaction for consistency
await concurrencyManager.executeTransaction([
  async (session) => {
    await Product.reserveStock(productId, quantity, session);
    await Cart.addItem(sessionId, itemData, session);
  }
]);
```

### Locking Mechanism
```javascript
// Resource locking with timeout
const lockId = await concurrencyManager.acquireLock(resourceId);
try {
  // Critical section
  await performOperation();
} finally {
  concurrencyManager.releaseLock(resourceId, lockId);
}
```

### Event-Driven Payment
```javascript
// Non-blocking payment initiation
eventEmitter.on('payment:completed', async (paymentData) => {
  await Order.updatePaymentStatus(orderId, 'completed');
  await confirmStockReservation(orderId);
});
```

## 📈 Monitoring & Observability

### System Health
Monitor key metrics:
- Memory usage and CPU utilization
- Active database connections
- Request throughput and error rates
- Concurrency operation statistics

### Logging
Structured JSON logs include:
- Request/response details
- Performance metrics
- Error traces with context
- Concurrency operation tracking

### Performance Monitoring
Track performance of:
- HTTP requests
- Database operations
- Stock operations
- Payment processing

## 🛡️ Security Features

- **Rate Limiting**: Prevents abuse with configurable limits
- **CORS Protection**: Configurable cross-origin policies
- **Helmet Integration**: Security headers for common vulnerabilities
- **Input Validation**: Comprehensive request validation
- **Session Security**: Secure session management
- **Error Sanitization**: Prevents information leakage

## 🚀 Deployment

### Production Setup
1. **Environment Configuration**
   ```bash
   NODE_ENV=production
   MONGODB_URI=your-production-mongodb-uri
   JWT_SECRET=your-strong-jwt-secret
   ```

2. **Process Management**
   ```bash
   # Using PM2
   npm install -g pm2
   pm2 start server.js --name cloth-store-api
   ```

3. **Reverse Proxy (Nginx)**
   ```nginx
   server {
       listen 80;
       location / {
           proxy_pass http://localhost:5000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```

### Docker Deployment
```bash
# Build image
docker build -t cloth-store-backend .

# Run container
docker run -p 5000:5000 --env-file .env cloth-store-backend
```

## 🤝 Development Guidelines

### Code Style
- Use ESLint configuration
- Follow async/await patterns
- Implement proper error handling
- Write comprehensive tests

### Database Guidelines
- Use transactions for multi-step operations
- Implement proper indexing
- Handle connection failures gracefully
- Use optimistic locking for concurrency

### API Design
- RESTful endpoint structure
- Consistent response formats
- Proper HTTP status codes
- Comprehensive error messages

## 📝 Scripts Reference

| Script | Description |
|--------|-------------|
| `npm start` | Start production server |
| `npm run dev` | Start development server with hot reload |
| `npm test` | Run test suite |
| `npm run test:concurrency` | Run concurrency tests |
| `npm run seed` | Seed database with sample data |
| `npm run seed:clear` | Clear all products from database |
| `npm run seed:stats` | Show database statistics |

## 🐛 Troubleshooting

### Common Issues

1. **MongoDB Connection Failed**
   - Check MongoDB service is running
   - Verify connection string in `.env`
   - Check network connectivity

2. **High Memory Usage**
   - Monitor for memory leaks
   - Adjust connection pool size
   - Review caching strategies

3. **Concurrency Issues**
   - Check lock timeout configurations
   - Review transaction isolation levels
   - Monitor deadlock detection logs

### Debug Mode
```bash
LOG_LEVEL=debug npm run dev
```

### Performance Profiling
```bash
node --prof server.js
node --prof-process isolate-*.log > processed.txt
```

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙋‍♂️ Support

For questions and support:
1. Check the troubleshooting section
2. Review system logs
3. Create an issue with detailed information

## 🔄 Contributing

1. Fork the repository
2. Create a feature branch
3. Write tests for new features
4. Ensure all tests pass
5. Submit a pull request

---

**Built with ❤️ for high-concurrency e-commerce applications**