# TypeScript/NestJS Coding Standards - Norsktorget.no

## File Organization

### Import Order
```typescript
// 1. Node built-ins
import { readFileSync } from 'fs';
import { join } from 'path';

// 2. External packages
import { Injectable, Logger } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

// 3. Internal modules (absolute paths from src/)
import { Ad } from '@/modules/ads/entities/ad.entity';
import { User } from '@/modules/users/entities/user.entity';
import { AppException } from '@/common/exceptions/app.exception';

// 4. Relative imports
import { CreateAdDto } from './dto/create-ad.dto';
import { AdMapper } from './mappers/ad.mapper';
```

## Naming Conventions

### Files
- **kebab-case** for all files
- Suffixes: `.service.ts`, `.controller.ts`, `.entity.ts`, `.dto.ts`, `.guard.ts`

### Classes and Interfaces
- **PascalCase** for classes and interfaces
- No `I` prefix for interfaces (TypeScript convention)
- DTOs end with `Dto`: `CreateAdDto`, `UpdateUserDto`
- Entities end with `Entity` in class name (optional in filename)

### Variables and Functions
- **camelCase** for variables and functions
- Descriptive boolean names: `isActive`, `hasPermission`, `canEdit`
- Private members with `private` keyword (not underscore)

### Constants
- **SCREAMING_SNAKE_CASE** for constants
```typescript
const MAX_FILE_SIZE = 5242880; // 5MB
const API_TIMEOUT_MS = 30000;
const CACHE_TTL_SECONDS = 300;
```

## Architecture

### Module Structure
```
src/
├── common/
│   ├── decorators/
│   ├── exceptions/
│   ├── guards/
│   ├── interceptors/
│   └── pipes/
├── config/
│   ├── database.config.ts
│   └── app.config.ts
└── modules/
    ├── ads/
    │   ├── dto/
    │   ├── entities/
    │   ├── mappers/
    │   ├── ads.controller.ts
    │   ├── ads.service.ts
    │   └── ads.module.ts
    └── users/
```

## Key Rules

1. **Max 300 lines per file** - Split larger files
2. **Dependency Injection** - Use NestJS DI, never `new`
3. **DTOs for validation** - All inputs validated with class-validator
4. **Entities for database** - TypeORM entities, never use DTOs in DB
5. **Result/Exception pattern** - Throw custom exceptions, not generic Error
6. **Async/await** - Never use `.then()` chains

## Controller Pattern

```typescript
@Controller('ads')
@UseGuards(JwtAuthGuard)
@ApiTags('Ads')
export class AdsController {
  constructor(
    private readonly adsService: AdsService,
    private readonly logger: Logger,
  ) {}

  @Post()
  @ApiOperation({ summary: 'Create a new ad' })
  @ApiResponse({ status: 201, type: AdResponseDto })
  async create(
    @Body() createAdDto: CreateAdDto,
    @CurrentUser() user: User,
  ): Promise<AdResponseDto> {
    this.logger.log(`Creating ad for user ${user.id}`);
    
    try {
      const ad = await this.adsService.create(createAdDto, user);
      return AdMapper.toResponse(ad);
    } catch (error) {
      this.logger.error(`Failed to create ad: ${error.message}`);
      throw error;
    }
  }

  @Get(':id')
  @Public() // Custom decorator to bypass auth
  @ApiOperation({ summary: 'Get ad by ID' })
  async findOne(@Param('id', ParseUUIDPipe) id: string): Promise<AdResponseDto> {
    const ad = await this.adsService.findOne(id);
    
    if (!ad) {
      throw new NotFoundException(`Ad with ID ${id} not found`);
    }
    
    return AdMapper.toResponse(ad);
  }
}
```

## Service Pattern

```typescript
@Injectable()
export class AdsService {
  private readonly logger = new Logger(AdsService.name);

  constructor(
    @InjectRepository(Ad)
    private readonly adsRepository: Repository<Ad>,
    private readonly categoriesService: CategoriesService,
    private readonly storageService: StorageService,
  ) {}

  async create(dto: CreateAdDto, user: User): Promise<Ad> {
    // 1. Validate business rules
    const category = await this.categoriesService.findOne(dto.categoryId);
    if (!category) {
      throw new ValidationException('categoryId', 'Category not found');
    }

    // 2. Create entity
    const ad = this.adsRepository.create({
      ...dto,
      userId: user.id,
      status: AdStatus.DRAFT,
      createdAt: new Date(),
    });

    // 3. Save to database
    try {
      return await this.adsRepository.save(ad);
    } catch (error) {
      this.logger.error(`Database error creating ad: ${error.message}`);
      throw new InternalServerErrorException('Failed to create ad');
    }
  }

  async findOne(id: string): Promise<Ad | null> {
    return this.adsRepository.findOne({
      where: { id },
      relations: ['user', 'category'],
    });
  }
}
```

## DTO Validation

```typescript
import { IsString, IsNumber, IsOptional, IsEnum, Min, Max } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateAdDto {
  @ApiProperty({ description: 'Ad title', example: 'Tesla Model 3 2021' })
  @IsString()
  title: string;

  @ApiProperty({ description: 'Price in NOK', example: 350000 })
  @IsNumber()
  @Min(0)
  price: number;

  @ApiPropertyOptional({ description: 'Ad description' })
  @IsOptional()
  @IsString()
  description?: string;

  @ApiProperty({ enum: AdStatus })
  @IsEnum(AdStatus)
  status: AdStatus;
}
```

## Entity Definition

```typescript
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne, CreateDateColumn } from 'typeorm';

@Entity('ads')
export class Ad {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'varchar', length: 200 })
  title: string;

  @Column({ type: 'text', nullable: true })
  description: string | null;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  price: number;

  @Column({ type: 'enum', enum: AdStatus, default: AdStatus.DRAFT })
  status: AdStatus;

  @ManyToOne(() => User, user => user.ads)
  user: User;

  @Column({ type: 'uuid' })
  userId: string;

  @CreateDateColumn()
  createdAt: Date;

  @Column({ type: 'jsonb', nullable: true })
  customFields: Record<string, any> | null;
}
```

## Error Handling

```typescript
// Custom exception base
export class AppException extends HttpException {
  constructor(
    message: string,
    public readonly code: string,
    status: HttpStatus,
  ) {
    super({ message, code }, status);
  }
}

// Specific exceptions
export class ValidationException extends AppException {
  constructor(field: string, message: string) {
    super(
      `Validation failed for ${field}: ${message}`,
      'VALIDATION_ERROR',
      HttpStatus.BAD_REQUEST,
    );
  }
}

export class InsufficientPermissionsException extends AppException {
  constructor(resource: string, action: string) {
    super(
      `Insufficient permissions to ${action} ${resource}`,
      'INSUFFICIENT_PERMISSIONS',
      HttpStatus.FORBIDDEN,
    );
  }
}

// Usage
if (!user.canEditAd(ad)) {
  throw new InsufficientPermissionsException('ad', 'edit');
}
```

## Testing

```typescript
describe('AdsService', () => {
  let service: AdsService;
  let repository: Repository<Ad>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        AdsService,
        {
          provide: getRepositoryToken(Ad),
          useValue: {
            create: jest.fn(),
            save: jest.fn(),
            findOne: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get(AdsService);
    repository = module.get(getRepositoryToken(Ad));
  });

  describe('create', () => {
    it('should create ad successfully', async () => {
      const dto: CreateAdDto = {
        title: 'Test Ad',
        price: 1000,
        categoryId: 'cat-123',
      };
      
      const user = { id: 'user-123' } as User;
      const savedAd = { id: 'ad-123', ...dto } as Ad;

      jest.spyOn(repository, 'create').mockReturnValue(savedAd);
      jest.spyOn(repository, 'save').mockResolvedValue(savedAd);

      const result = await service.create(dto, user);

      expect(result).toEqual(savedAd);
      expect(repository.create).toHaveBeenCalledWith({
        ...dto,
        userId: user.id,
        status: AdStatus.DRAFT,
        createdAt: expect.any(Date),
      });
    });
  });
});
```

## Database Best Practices

1. **Use migrations** - Never modify schema directly
2. **Indexes** - Add indexes for frequently queried fields
3. **Transactions** - Use for multi-step operations
4. **Query optimization** - Use `select`, avoid N+1 queries
5. **JSONB for flexibility** - Use for dynamic fields

## Security

1. **Input validation** - Always validate with DTOs
2. **SQL injection** - Use parameterized queries (TypeORM does this)
3. **Authentication** - JWT with guards
4. **Authorization** - Role-based access control
5. **Rate limiting** - Throttle endpoints
6. **CORS** - Configure for allowed origins only
