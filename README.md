# NestJS Dockerized Backend Boilerplate

A production-ready NestJS application with PostgreSQL integration, Docker containerization, and enterprise-grade architecture patterns. This boilerplate demonstrates best practices for building scalable REST APIs with proper separation of concerns, dependency injection, and repository patterns.

## 📋 Overview

This is a fully dockerized NestJS blog application that provides a solid foundation for building enterprise applications. It includes:

- **NestJS Framework** — Progressive Node.js framework for building efficient server-side applications
- **PostgreSQL Database** — Robust relational database with environment-based configuration
- **Docker & Docker Compose** — Containerized deployment for consistent environments
- **Repository Pattern** — Clean data access layer abstraction
- **Dependency Injection** — Inversion of Control (IoC) container for loose coupling
- **Module-based Architecture** — Feature-based module organization
- **TypeORM** — Object-Relational Mapping with type safety

## 🏗️ Architecture & Design Patterns

### 1. **Dependency Injection (IoC Container)**

NestJS implements the Inversion of Control (IoC) principle, allowing dependencies to be injected rather than created within classes. This promotes loose coupling and easier testing.

**Example from the codebase:**

```typescript
@Injectable()
export class BlogService {
  constructor(private blogRepository: BlogRepository) {}
  
  async getArticles(): Promise<any> {
    return await this.blogRepository.getArticles();
  }
}
```

The `BlogRepository` is injected through the constructor, not instantiated within the service. The NestJS DI container manages the lifecycle and resolution of dependencies.

### 2. **Repository Pattern**

The repository pattern abstracts the data access layer, providing a clean interface between services and the database. This separation allows:

- Easy testing with mock repositories
- Database implementation changes without affecting business logic
- Consistent data access methods across the application

**Repository Layer:**

```typescript
@Injectable()
export class BlogRepository {
  constructor(
    @InjectRepository(Blog)
    private blogRepository: Repository<Blog>
  ) {}

  async getArticles(): Promise<any> {
    return this.blogRepository.find();
  }

  async getArticleById(blogid: number): Promise<any> {
    return this.blogRepository.findOne({
      where: { id: blogid }
    });
  }

  async postArticle(data: any): Promise<any> {
    const article = this.blogRepository.create(data);
    return this.blogRepository.save(article);
  }
}
```

### 3. **Service Layer**

Services contain business logic and orchestrate between controllers and repositories. They handle validation, transformation, and complex operations.

```typescript
@Injectable()
export class BlogService {
  constructor(private blogRepository: BlogRepository) {}

  async getArticles(): Promise<any> {
    const articles = await this.blogRepository.getArticles();
    return articles;
  }

  async postArticle(data: any): Promise<any> {
    const article = await this.blogRepository.postArticle(data);
    return article;
  }
}
```

### 4. **Controller Layer**

Controllers handle HTTP requests and responses. They delegate business logic to services and format responses for clients.

```typescript
@Controller('blog')
export class BlogController {
  constructor(private readonly blogService: BlogService) {}

  @Get('')
  async getArticles() {
    const articles = await this.blogService.getArticles();
    return {
      success: true,
      articles: articles
    };
  }

  @Get(':id')
  async getSingleArticle(@Param('id') id: number) {
    const article = await this.blogService.getArticlebyID(id);
    return {
      success: true,
      article: article
    };
  }

  @Post('')
  async postArticle(@Body() createArticleDto: createArticleDto) {
    const response = await this.blogService.postArticle(createArticleDto);
    return {
      success: true,
      response: response
    };
  }
}
```

### 5. **Module Organization**

The application is organized into feature-based modules, each encapsulating related components:

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([Blog])],
  providers: [BlogService, BlogRepository],
  controllers: [BlogController],
  exports: [BlogService, BlogRepository],
})
export class BlogModule {}
```

Each module:
- **Imports** — Dependencies from other modules
- **Providers** — Services and repositories (DI tokens)
- **Controllers** — HTTP endpoints
- **Exports** — Public API for other modules

### 6. **Separation of Concerns**

The application strictly separates:

| Layer | Responsibility | Example |
|-------|-----------------|---------|
| **Controller** | HTTP handling, request/response | Route definitions, parameter extraction |
| **Service** | Business logic, validation | Article creation logic, data transformation |
| **Repository** | Data persistence, queries | Database operations, entity management |
| **Entity** | Domain model, database schema | Blog entity with columns and relationships |

This layering ensures each component has a single responsibility and can be tested independently.

## 📁 Project Structure

```
src/
├── app.module.ts              # Root module
├── app.controller.ts          # Root controller
├── app.service.ts             # Root service
├── blog/
│   ├── blog.module.ts         # Blog feature module
│   ├── blog.controller.ts     # Blog HTTP endpoints
│   ├── blog.service.ts        # Blog business logic
│   ├── blog.repository.ts     # Blog data access
│   ├── request.dto.ts         # Data transfer objects
│   └── entities/
│       └── blog.entity.ts     # Blog database entity
└── database/
    └── entities/
        └── blog.entity.ts     # Shared entities

Dockerfile                      # Container configuration
docker-compose.yml             # Multi-container orchestration
.env                           # Environment variables
package.json                   # Dependencies
tsconfig.json                  # TypeScript configuration
```

## 🚀 Getting Started

### Prerequisites

- Node.js 18+ or Docker
- npm or yarn
- PostgreSQL (or use Docker)

### Installation

1. **Clone the repository:**

```bash
git clone <repository-url>
cd NestPractice/prac
```

2. **Install dependencies:**

```bash
npm install
```

3. **Create environment file:**

```bash
cp .env.example .env
```

4. **Configure `.env` file:**

```env
PGHOST=your-database-host
PGPORT=5432
PGUSER=your-username
PGPASSWORD=your-password
PGDATABASE=blogs
NODE_ENV=development
```

5. **Build the application:**

```bash
npm run build
```

6. **Start the development server:**

```bash
npm run start:dev
```

The API will be available at `http://localhost:3000`

## 🐳 Docker Setup

### Using Docker Compose (Recommended)

```bash
docker-compose up --build
```

This starts both the NestJS application and PostgreSQL database.

### Manual Docker Build

1. **Build the image:**

```bash
docker build -t nestjs-blog-api .
```

2. **Run the container:**

```bash
docker run -e PGHOST=your-host \
  -e PGPORT=5432 \
  -e PGUSER=your-user \
  -e PGPASSWORD=your-password \
  -e PGDATABASE=blogs \
  -p 8080:8080 \
  nestjs-blog-api
```

## 📚 API Endpoints

### Get All Articles
```
GET /blog
```

Response:
```json
{
  "success": true,
  "articles": [
    {
      "id": 1,
      "name": "Article Title",
      "created_at": "2025-10-25T04:51:33.736Z",
      "updated_at": "2025-10-25T04:51:33.736Z"
    }
  ]
}
```

### Get Single Article
```
GET /blog/:id
```

### Create Article
```
POST /blog
Content-Type: application/json

{
  "name": "New Article",
  "description": "Article description"
}
```

### Update Article
```
PUT /blog/:id
Content-Type: application/json

{
  "name": "Updated Title"
}
```

### Delete Article
```
DELETE /blog/:id
```

## 🔧 Available Scripts

```bash
# Development
npm run start:dev          # Run with hot reload

# Production
npm run build             # Compile TypeScript
npm run start            # Run compiled application
npm run start:prod       # Run in production mode

# Testing
npm run test             # Run unit tests
npm run test:e2e         # Run end-to-end tests
npm run test:cov         # Generate coverage report
```

## 🗄️ Database Configuration

TypeORM is configured with environment variables for flexibility:

```typescript
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: process.env.PGHOST,
      port: parseInt(process.env.PGPORT || '5432'),
      username: process.env.PGUSER,
      password: process.env.PGPASSWORD,
      database: process.env.PGDATABASE,
      entities: [Blog],
      synchronize: true,
      logging: true,
      ssl: true,
    }),
  ],
})
export class AppModule {}
```

**Key Features:**
- Auto-synchronization of database schema
- SQL query logging for debugging
- SSL support for remote connections
- Environment-based configuration

## 🧪 Testing

The repository pattern makes unit testing straightforward:

```typescript
describe('BlogService', () => {
  let service: BlogService;
  let repository: BlogRepository;

  beforeEach(async () => {
    const mockRepository = {
      getArticles: jest.fn().mockResolvedValue([]),
    };

    const module = await Test.createTestingModule({
      providers: [
        BlogService,
        { provide: BlogRepository, useValue: mockRepository },
      ],
    }).compile();

    service = module.get<BlogService>(BlogService);
    repository = module.get<BlogRepository>(BlogRepository);
  });

  it('should call repository.getArticles()', async () => {
    await service.getArticles();
    expect(repository.getArticles).toHaveBeenCalled();
  });
});
```

## 📦 Dependencies

- **@nestjs/common** — NestJS core utilities
- **@nestjs/core** — NestJS framework core
- **@nestjs/typeorm** — TypeORM integration
- **typeorm** — ORM for database operations
- **pg** — PostgreSQL client
- **@nestjs/config** — Configuration management
- **class-validator** — DTO validation
- **class-transformer** — Object transformation

## 🔐 Security Considerations

1. **Environment Variables** — Sensitive data stored in `.env`, not in code
2. **SSL Connection** — Database connections use SSL encryption
3. **Validation** — DTOs validate incoming data
4. **Error Handling** — Avoid exposing internal errors to clients

## 📈 Extending the Application

To add a new feature:

1. **Create a new module:**

```bash
nest generate module features/users
nest generate service features/users
nest generate controller features/users
```

2. **Create repository:**

```typescript
@Injectable()
export class UserRepository {
  constructor(@InjectRepository(User) private repo: Repository<User>) {}
  // Add methods
}
```

3. **Update module imports:**

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UserService, UserRepository],
  controllers: [UserController],
})
export class UserModule {}
```

4. **Import in AppModule:**

```typescript
imports: [UserModule, BlogModule]
```

## 🐛 Troubleshooting

**Docker connection refused:**
- Ensure database host is correct for Docker (container name or network)
- Check environment variables are passed to container

**TypeORM migrations not running:**
- Verify `synchronize: true` in TypeOrmModule config
- Check entity imports in AppModule

**Port already in use:**
```bash
lsof -i :8080
kill -9 <PID>
```

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

## 👥 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

## 📧 Support

For issues and questions, please open a GitHub issue or contact the maintainers.

---

**Happy Coding! 🚀**
