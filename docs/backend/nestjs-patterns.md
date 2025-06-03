# 🔧 Patrones NestJS para Microservicios

Guía completa de patrones específicos de NestJS para arquitectura de microservicios escalable, incluyendo decoradores personalizados, interceptors, guards y pipes avanzados.

## 🎯 ¿Por qué Patrones Específicos de NestJS?

NestJS ofrece un ecosistema robusto basado en TypeScript y decoradores que permite implementar patrones enterprise de manera elegante y mantenible:

- **🏗️ Arquitectura modular**: Cada microservicio es un módulo independiente
- **🔌 Dependency Injection**: IoC container integrado para mejor testabilidad
- **🛡️ Guards y Interceptors**: Separación clara de responsabilidades
- **📡 Múltiples transportes**: HTTP, TCP, Redis, RabbitMQ, gRPC
- **🧪 Testing integrado**: Framework de testing built-in

## 📦 Estructura de Microservicio NestJS

### Plantilla Base de Microservicio

```typescript
// apps/auth-service/src/main.ts
import { NestFactory } from "@nestjs/core";
import { MicroserviceOptions, Transport } from "@nestjs/microservices";
import { AppModule } from "./app.module";
import { ValidationPipe } from "@nestjs/common";
import { SwaggerModule, DocumentBuilder } from "@nestjs/swagger";

async function bootstrap() {
  // Crear aplicación híbrida (HTTP + Microservice)
  const app = await NestFactory.create(AppModule);

  // Configurar validación global
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    })
  );

  // Configurar microservicio (para comunicación interna)
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.TCP,
    options: {
      host: "0.0.0.0",
      port: process.env.MICROSERVICE_PORT || 3001,
    },
  });

  // Configurar Swagger
  const config = new DocumentBuilder()
    .setTitle("Auth Service API")
    .setDescription("Authentication and authorization microservice")
    .setVersion("1.0")
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup("api", app, document);

  // Iniciar microservicio
  await app.startAllMicroservices();

  // Iniciar servidor HTTP
  await app.listen(process.env.HTTP_PORT || 3000);

  console.log(`🚀 Auth Service running on: ${await app.getUrl()}`);
  console.log(
    `📡 Microservice listening on port: ${
      process.env.MICROSERVICE_PORT || 3001
    }`
  );
}

bootstrap();
```

### Estructura de Módulos

```typescript
// apps/auth-service/src/app.module.ts
import { Module } from "@nestjs/common";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { TypeOrmModule } from "@nestjs/typeorm";
import { JwtModule } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
import { RedisModule } from "@nestjs-modules/ioredis";

import { AuthModule } from "./auth/auth.module";
import { UsersModule } from "./users/users.module";
import { RolesModule } from "./roles/roles.module";
import { SessionsModule } from "./sessions/sessions.module";
import { SharedModule } from "@ecommerce/shared";

@Module({
  imports: [
    // Configuración
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: [".env.local", ".env"],
    }),

    // Base de datos
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: "postgres",
        host: configService.get("DB_HOST"),
        port: configService.get("DB_PORT"),
        username: configService.get("DB_USERNAME"),
        password: configService.get("DB_PASSWORD"),
        database: configService.get("DB_NAME"),
        entities: [__dirname + "/**/*.entity{.ts,.js}"],
        synchronize: configService.get("NODE_ENV") === "development",
        logging: configService.get("NODE_ENV") === "development",
      }),
      inject: [ConfigService],
    }),

    // Redis para cache y sesiones
    RedisModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        config: {
          host: configService.get("REDIS_HOST"),
          port: configService.get("REDIS_PORT"),
          password: configService.get("REDIS_PASSWORD"),
          retryDelayOnFailover: 100,
          maxRetriesPerRequest: 3,
        },
      }),
      inject: [ConfigService],
    }),

    // JWT
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get("JWT_SECRET"),
        signOptions: {
          expiresIn: configService.get("JWT_EXPIRES_IN", "1h"),
        },
      }),
      inject: [ConfigService],
    }),

    // Passport para autenticación
    PassportModule.register({ defaultStrategy: "jwt" }),

    // Módulos de negocio
    AuthModule,
    UsersModule,
    RolesModule,
    SessionsModule,
    SharedModule,
  ],
})
export class AppModule {}
```

## 🔒 Patrones de Autenticación y Autorización

### JWT Strategy con RBAC

```typescript
// apps/auth-service/src/auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { Strategy, ExtractJwt } from "passport-jwt";
import { ConfigService } from "@nestjs/config";
import { InjectRedis, Redis } from "@nestjs-modules/ioredis";

import { UsersService } from "../users/users.service";
import { JwtPayload } from "../interfaces/jwt-payload.interface";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private readonly usersService: UsersService,
    private readonly configService: ConfigService,
    @InjectRedis() private readonly redis: Redis
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: configService.get("JWT_SECRET"),
      passReqToCallback: true,
    });
  }

  async validate(req: Request, payload: JwtPayload) {
    const { userId, sessionId } = payload;

    // Verificar que la sesión esté activa en Redis
    const sessionData = await this.redis.get(`session:${sessionId}`);
    if (!sessionData) {
      throw new UnauthorizedException("Session expired or invalid");
    }

    // Obtener usuario con roles
    const user = await this.usersService.findByIdWithRoles(userId);
    if (!user || !user.isActive) {
      throw new UnauthorizedException("User not found or inactive");
    }

    // Verificar token blacklist
    const token = ExtractJwt.fromAuthHeaderAsBearerToken()(req);
    const isBlacklisted = await this.redis.get(`blacklist:${token}`);
    if (isBlacklisted) {
      throw new UnauthorizedException("Token has been revoked");
    }

    // Retornar usuario con permisos
    return {
      ...user,
      sessionId,
      permissions: user.roles.flatMap((role) => role.permissions),
    };
  }
}
```

### Guards Personalizados

```typescript
// libs/shared/src/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { ROLES_KEY } from "../decorators/roles.decorator";
import { Role } from "../enums/role.enum";

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();

    return requiredRoles.some((role) =>
      user.roles?.some((userRole) => userRole.name === role)
    );
  }
}

// Permissions Guard más granular
@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredPermissions = this.reflector.getAllAndOverride<string[]>(
      "permissions",
      [context.getHandler(), context.getClass()]
    );

    if (!requiredPermissions) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();

    return requiredPermissions.every((permission) =>
      user.permissions?.includes(permission)
    );
  }
}
```

### Decoradores Personalizados

```typescript
// libs/shared/src/decorators/roles.decorator.ts
import { SetMetadata } from "@nestjs/common";
import { Role } from "../enums/role.enum";

export const ROLES_KEY = "roles";
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// libs/shared/src/decorators/permissions.decorator.ts
import { SetMetadata } from "@nestjs/common";

export const Permissions = (...permissions: string[]) =>
  SetMetadata("permissions", permissions);

// libs/shared/src/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from "@nestjs/common";

export const CurrentUser = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  }
);

// libs/shared/src/decorators/public.decorator.ts
import { SetMetadata } from "@nestjs/common";

export const IS_PUBLIC_KEY = "isPublic";
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

## 📡 Patrones de Comunicación entre Microservicios

### Cliente de Microservicio

```typescript
// libs/shared/src/services/microservice-client.service.ts
import { Injectable, OnModuleInit } from "@nestjs/common";
import {
  ClientProxy,
  ClientProxyFactory,
  Transport,
} from "@nestjs/microservices";
import { ConfigService } from "@nestjs/config";
import { timeout, catchError, retry } from "rxjs/operators";
import { throwError, of } from "rxjs";
import { Logger } from "@nestjs/common";

@Injectable()
export class MicroserviceClientService implements OnModuleInit {
  private readonly logger = new Logger(MicroserviceClientService.name);
  private clients: Map<string, ClientProxy> = new Map();

  constructor(private configService: ConfigService) {}

  async onModuleInit() {
    // Configurar clientes para cada microservicio
    this.setupClient("auth", "AUTH_SERVICE_HOST", "AUTH_SERVICE_PORT");
    this.setupClient(
      "products",
      "PRODUCTS_SERVICE_HOST",
      "PRODUCTS_SERVICE_PORT"
    );
    this.setupClient("orders", "ORDERS_SERVICE_HOST", "ORDERS_SERVICE_PORT");
    this.setupClient(
      "payments",
      "PAYMENTS_SERVICE_HOST",
      "PAYMENTS_SERVICE_PORT"
    );

    // Conectar todos los clientes
    await Promise.all(
      Array.from(this.clients.values()).map((client) => client.connect())
    );
  }

  private setupClient(serviceName: string, hostEnv: string, portEnv: string) {
    const client = ClientProxyFactory.create({
      transport: Transport.TCP,
      options: {
        host: this.configService.get(hostEnv),
        port: this.configService.get(portEnv),
      },
    });

    this.clients.set(serviceName, client);
  }

  async send<T>(service: string, pattern: string, data: any): Promise<T> {
    const client = this.clients.get(service);
    if (!client) {
      throw new Error(`Service ${service} not found`);
    }

    return client
      .send<T>(pattern, data)
      .pipe(
        timeout(5000), // 5 seconds timeout
        retry(3), // Retry 3 times
        catchError((error) => {
          this.logger.error(
            `Failed to communicate with ${service} service: ${error.message}`,
            error.stack
          );
          return throwError(() => error);
        })
      )
      .toPromise();
  }

  async emit(service: string, pattern: string, data: any): Promise<void> {
    const client = this.clients.get(service);
    if (!client) {
      throw new Error(`Service ${service} not found`);
    }

    client.emit(pattern, data);
  }
}
```

### Patrón de Mensaje y Respuesta

```typescript
// apps/products-service/src/products/products.controller.ts
import { Controller } from "@nestjs/common";
import { MessagePattern, Payload } from "@nestjs/microservices";
import { ProductsService } from "./products.service";
import { CreateProductDto, UpdateProductDto, FindProductDto } from "./dto";

@Controller()
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @MessagePattern("products.create")
  async createProduct(@Payload() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }

  @MessagePattern("products.findAll")
  async findAllProducts(@Payload() query: FindProductDto) {
    return this.productsService.findAll(query);
  }

  @MessagePattern("products.findOne")
  async findOneProduct(@Payload() id: string) {
    return this.productsService.findOne(id);
  }

  @MessagePattern("products.update")
  async updateProduct(
    @Payload() data: { id: string; updateProductDto: UpdateProductDto }
  ) {
    return this.productsService.update(data.id, data.updateProductDto);
  }

  @MessagePattern("products.remove")
  async removeProduct(@Payload() id: string) {
    return this.productsService.remove(id);
  }

  @MessagePattern("products.updateStock")
  async updateStock(
    @Payload()
    data: {
      id: string;
      quantity: number;
      operation: "add" | "subtract";
    }
  ) {
    return this.productsService.updateStock(
      data.id,
      data.quantity,
      data.operation
    );
  }

  @MessagePattern("products.validateStock")
  async validateStock(
    @Payload() items: Array<{ productId: string; quantity: number }>
  ) {
    return this.productsService.validateStock(items);
  }
}
```

### Service Layer con Comunicación entre Microservicios

```typescript
// apps/orders-service/src/orders/orders.service.ts
import {
  Injectable,
  BadRequestException,
  NotFoundException,
} from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";
import { InjectRedis, Redis } from "@nestjs-modules/ioredis";

import { Order } from "./entities/order.entity";
import { CreateOrderDto } from "./dto/create-order.dto";
import { MicroserviceClientService } from "@ecommerce/shared";
import { OrderStatus } from "./enums/order-status.enum";

@Injectable()
export class OrdersService {
  constructor(
    @InjectRepository(Order)
    private ordersRepository: Repository<Order>,
    private microserviceClient: MicroserviceClientService,
    @InjectRedis() private readonly redis: Redis
  ) {}

  async create(createOrderDto: CreateOrderDto, userId: string): Promise<Order> {
    // 1. Validar stock de productos
    const stockValidation = await this.microserviceClient.send(
      "products",
      "products.validateStock",
      createOrderDto.items
    );

    if (!stockValidation.isValid) {
      throw new BadRequestException(
        `Insufficient stock for products: ${stockValidation.invalidItems.join(
          ", "
        )}`
      );
    }

    // 2. Crear orden en estado pending
    const order = this.ordersRepository.create({
      ...createOrderDto,
      userId,
      status: OrderStatus.PENDING,
      createdAt: new Date(),
    });

    const savedOrder = await this.ordersRepository.save(order);

    // 3. Reservar stock (compensable transaction)
    try {
      await this.microserviceClient.send("products", "products.reserveStock", {
        orderId: savedOrder.id,
        items: createOrderDto.items,
      });

      // 4. Emitir evento de orden creada
      await this.microserviceClient.emit("orders", "order.created", {
        orderId: savedOrder.id,
        userId,
        totalAmount: savedOrder.totalAmount,
        items: savedOrder.items,
      });

      return savedOrder;
    } catch (error) {
      // Compensar: eliminar orden si no se pudo reservar stock
      await this.ordersRepository.delete(savedOrder.id);
      throw new BadRequestException("Failed to reserve stock for order");
    }
  }

  async processPayment(orderId: string): Promise<Order> {
    const order = await this.findOne(orderId);

    if (order.status !== OrderStatus.PENDING) {
      throw new BadRequestException("Order is not in pending status");
    }

    try {
      // Procesar pago
      const paymentResult = await this.microserviceClient.send(
        "payments",
        "payments.process",
        {
          orderId: order.id,
          amount: order.totalAmount,
          currency: order.currency,
          paymentMethod: order.paymentMethod,
        }
      );

      if (paymentResult.success) {
        // Actualizar orden a pagada
        order.status = OrderStatus.PAID;
        order.paymentId = paymentResult.paymentId;
        await this.ordersRepository.save(order);

        // Confirmar stock
        await this.microserviceClient.send(
          "products",
          "products.confirmStock",
          { orderId: order.id }
        );

        // Emitir evento de pago exitoso
        await this.microserviceClient.emit("orders", "order.paid", {
          orderId: order.id,
          paymentId: paymentResult.paymentId,
          amount: order.totalAmount,
        });

        return order;
      } else {
        // Liberar stock si el pago falló
        await this.microserviceClient.send(
          "products",
          "products.releaseStock",
          { orderId: order.id }
        );

        throw new BadRequestException("Payment processing failed");
      }
    } catch (error) {
      // Liberar stock en caso de error
      await this.microserviceClient.send("products", "products.releaseStock", {
        orderId: order.id,
      });

      throw error;
    }
  }

  async findOne(id: string): Promise<Order> {
    const order = await this.ordersRepository.findOne({
      where: { id },
      relations: ["items", "user"],
    });

    if (!order) {
      throw new NotFoundException(`Order with ID ${id} not found`);
    }

    return order;
  }
}
```

## 🔄 Interceptors Avanzados

### Logging Interceptor

```typescript
// libs/shared/src/interceptors/logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { tap } from "rxjs/operators";

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const { method, url, body, query, params, headers } = request;

    const userAgent = headers["user-agent"] || "";
    const ip = headers["x-forwarded-for"] || request.connection.remoteAddress;

    const now = Date.now();

    this.logger.log(
      `📥 ${method} ${url} - ${ip} - ${userAgent}`,
      JSON.stringify({ body, query, params })
    );

    return next.handle().pipe(
      tap((responseData) => {
        const duration = Date.now() - now;
        const statusCode = response.statusCode;

        this.logger.log(
          `📤 ${method} ${url} - ${statusCode} - ${duration}ms`,
          JSON.stringify({ response: responseData })
        );
      })
    );
  }
}
```

### Cache Interceptor

```typescript
// libs/shared/src/interceptors/cache.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable, of } from "rxjs";
import { tap } from "rxjs/operators";
import { InjectRedis, Redis } from "@nestjs-modules/ioredis";
import { Reflector } from "@nestjs/core";

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(
    @InjectRedis() private readonly redis: Redis,
    private reflector: Reflector
  ) {}

  async intercept(
    context: ExecutionContext,
    next: CallHandler
  ): Promise<Observable<any>> {
    const cacheTTL = this.reflector.get<number>(
      "cache-ttl",
      context.getHandler()
    );

    if (!cacheTTL) {
      return next.handle();
    }

    const request = context.switchToHttp().getRequest();
    const cacheKey = this.generateCacheKey(request);

    // Intentar obtener del cache
    const cachedResult = await this.redis.get(cacheKey);
    if (cachedResult) {
      return of(JSON.parse(cachedResult));
    }

    // Si no está en cache, ejecutar y guardar resultado
    return next.handle().pipe(
      tap(async (result) => {
        await this.redis.setex(cacheKey, cacheTTL, JSON.stringify(result));
      })
    );
  }

  private generateCacheKey(request: any): string {
    const { method, url, query, params, user } = request;
    const keyData = {
      method,
      url,
      query,
      params,
      userId: user?.id,
    };

    return `cache:${Buffer.from(JSON.stringify(keyData)).toString("base64")}`;
  }
}

// Decorador para configurar cache
export const CacheTTL = (ttl: number) => SetMetadata("cache-ttl", ttl);
```

### Transform Response Interceptor

```typescript
// libs/shared/src/interceptors/transform.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";

export interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
  timestamp: string;
  path: string;
  statusCode: number;
}

@Injectable()
export class TransformInterceptor<T>
  implements NestInterceptor<T, ApiResponse<T>>
{
  intercept(
    context: ExecutionContext,
    next: CallHandler
  ): Observable<ApiResponse<T>> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();

    return next.handle().pipe(
      map((data) => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
        path: request.url,
        statusCode: response.statusCode,
      }))
    );
  }
}
```

## 🧪 Testing Patterns

### Unit Testing

```typescript
// apps/auth-service/src/auth/auth.service.spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { JwtService } from "@nestjs/jwt";
import { getRepositoryToken } from "@nestjs/typeorm";
import { Repository } from "typeorm";
import { Redis } from "ioredis";
import { getRedisToken } from "@nestjs-modules/ioredis";

import { AuthService } from "./auth.service";
import { User } from "../users/entities/user.entity";
import { UsersService } from "../users/users.service";

describe("AuthService", () => {
  let service: AuthService;
  let usersService: UsersService;
  let jwtService: JwtService;
  let redis: Redis;

  const mockUserRepository = {
    findOne: jest.fn(),
    save: jest.fn(),
    create: jest.fn(),
  };

  const mockJwtService = {
    sign: jest.fn(),
    verify: jest.fn(),
  };

  const mockRedis = {
    set: jest.fn(),
    get: jest.fn(),
    del: jest.fn(),
    setex: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        AuthService,
        {
          provide: UsersService,
          useValue: {
            findByEmail: jest.fn(),
            create: jest.fn(),
          },
        },
        {
          provide: JwtService,
          useValue: mockJwtService,
        },
        {
          provide: getRedisToken(),
          useValue: mockRedis,
        },
      ],
    }).compile();

    service = module.get<AuthService>(AuthService);
    usersService = module.get<UsersService>(UsersService);
    jwtService = module.get<JwtService>(JwtService);
    redis = module.get<Redis>(getRedisToken());
  });

  describe("login", () => {
    it("should return access token for valid credentials", async () => {
      const loginDto = { email: "test@example.com", password: "password123" };
      const user = {
        id: "1",
        email: "test@example.com",
        password: "$2b$10$hashedpassword",
        isActive: true,
      };

      jest.spyOn(usersService, "findByEmail").mockResolvedValue(user as any);
      jest.spyOn(service as any, "validatePassword").mockResolvedValue(true);
      jest.spyOn(jwtService, "sign").mockReturnValue("mock-jwt-token");
      jest.spyOn(redis, "setex").mockResolvedValue("OK");

      const result = await service.login(loginDto);

      expect(result).toHaveProperty("accessToken");
      expect(result.accessToken).toBe("mock-jwt-token");
      expect(usersService.findByEmail).toHaveBeenCalledWith(loginDto.email);
    });

    it("should throw UnauthorizedException for invalid credentials", async () => {
      const loginDto = { email: "test@example.com", password: "wrongpassword" };

      jest.spyOn(usersService, "findByEmail").mockResolvedValue(null);

      await expect(service.login(loginDto)).rejects.toThrow();
    });
  });
});
```

### Integration Testing

```typescript
// apps/auth-service/test/auth.e2e-spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { INestApplication } from "@nestjs/common";
import * as request from "supertest";
import { TypeOrmModule } from "@nestjs/typeorm";
import { ConfigModule } from "@nestjs/config";

import { AppModule } from "../src/app.module";

describe("AuthController (e2e)", () => {
  let app: INestApplication;
  let authToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [
        ConfigModule.forRoot({
          envFilePath: ".env.test",
        }),
        AppModule,
      ],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  describe("/auth/register (POST)", () => {
    it("should register a new user", () => {
      return request(app.getHttpServer())
        .post("/auth/register")
        .send({
          email: "test@example.com",
          password: "password123",
          firstName: "John",
          lastName: "Doe",
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty("accessToken");
          expect(res.body.user).toHaveProperty("email", "test@example.com");
        });
    });
  });

  describe("/auth/login (POST)", () => {
    it("should login with valid credentials", () => {
      return request(app.getHttpServer())
        .post("/auth/login")
        .send({
          email: "test@example.com",
          password: "password123",
        })
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty("accessToken");
          authToken = res.body.accessToken;
        });
    });
  });

  describe("/auth/profile (GET)", () => {
    it("should get user profile with valid token", () => {
      return request(app.getHttpServer())
        .get("/auth/profile")
        .set("Authorization", `Bearer ${authToken}`)
        .expect(200)
        .expect((res) => {
          expect(res.body.user).toHaveProperty("email", "test@example.com");
        });
    });

    it("should reject access without token", () => {
      return request(app.getHttpServer()).get("/auth/profile").expect(401);
    });
  });
});
```

## 📊 Métricas y Observabilidad

### Metrics Interceptor

```typescript
// libs/shared/src/interceptors/metrics.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { tap, catchError } from "rxjs/operators";
import { PrometheusService } from "../services/prometheus.service";

@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private prometheusService: PrometheusService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, route } = request;

    const timer = this.prometheusService.httpDuration.startTimer({
      method,
      route: route?.path || "unknown",
    });

    this.prometheusService.httpRequests.inc({
      method,
      route: route?.path || "unknown",
    });

    return next.handle().pipe(
      tap(() => {
        timer({ status_code: "2xx" });
      }),
      catchError((error) => {
        timer({ status_code: "5xx" });
        this.prometheusService.httpErrors.inc({
          method,
          route: route?.path || "unknown",
          error_type: error.constructor.name,
        });
        throw error;
      })
    );
  }
}
```

## 🎯 Checklist de Implementación

### ✅ Configuración Base

- [ ] Configurar estructura de módulos NestJS
- [ ] Implementar configuración con ConfigModule
- [ ] Configurar base de datos con TypeORM
- [ ] Añadir Redis para cache y sesiones
- [ ] Configurar JWT y Passport

### ✅ Patrones de Autenticación

- [ ] Implementar JWT Strategy
- [ ] Crear Guards personalizados (Roles, Permissions)
- [ ] Añadir decoradores (@CurrentUser, @Roles, @Public)
- [ ] Configurar RBAC completo

### ✅ Comunicación entre Servicios

- [ ] Configurar TCP transport para microservicios
- [ ] Implementar MicroserviceClientService
- [ ] Crear patrones de mensaje (@MessagePattern)
- [ ] Añadir manejo de errores y timeouts

### ✅ Interceptors y Middleware

- [ ] Logging interceptor para auditoría
- [ ] Cache interceptor con Redis
- [ ] Transform interceptor para respuestas consistentes
- [ ] Metrics interceptor para Prometheus

### ✅ Testing

- [ ] Unit tests para servicios críticos
- [ ] Integration tests para controllers
- [ ] E2E tests para flujos completos
- [ ] Mock de dependencias externas

---

**🎯 Próximo paso**: Implementa la [API Gateway](./api-gateway.md) para centralizar el routing entre microservicios.
