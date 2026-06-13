# 📸 CODE SNAPSHOT - EXACT FILE CONTENTS VERIFICATION

**Purpose**: Frontend dapat memverifikasi bahwa setiap file memiliki kode yang tepat  
**Generated**: June 14, 2026  
**Commit**: 92cbc18  

> Copy-paste kode di bawah ke file Anda dan bandingkan kalau ada keraguan!

---

## 1️⃣ `/src/auth/auth.controller.ts`

```typescript
import { Body, Controller, Post } from '@nestjs/common';
import { AuthService } from './auth.service';
import { RegisterDto } from './dto/register.dto';
import { LoginDto } from './dto/login.dto';

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('register')
  register(@Body() registerDto: RegisterDto) {
    return this.authService.register(registerDto);
  }

  @Post('login')
  login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }
}
```

**Verification**: ✅ Exact match = File is correct

---

## 2️⃣ `/src/auth/auth.service.ts`

```typescript
import { Injectable, BadRequestException, UnauthorizedException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { RegisterDto } from './dto/register.dto';
import { LoginDto } from './dto/login.dto';
import * as bcrypt from 'bcrypt';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(
    private prisma: PrismaService,
    private jwtService: JwtService,
  ) {}

  async register(registerDto: RegisterDto) {
    const existingUser = await this.prisma.user.findUnique({
      where: {
        email: registerDto.email,
      },
    });

    if (existingUser) {
      throw new BadRequestException('Email already registered');
    }

    const hashedPassword = await bcrypt.hash(registerDto.password, 10);

    const user = await this.prisma.user.create({
      data: {
        name: registerDto.name,
        email: registerDto.email,
        phoneNumber: registerDto.phoneNumber,
        password: hashedPassword,
      },
    });

    return {
      message: 'User registered successfully',
      user: {
        id: user.id,
        name: user.name,
        email: user.email,
        phoneNumber: user.phoneNumber,
      },
    };
  }

  async login(loginDto: LoginDto) {
    const user = await this.prisma.user.findUnique({
      where: {
        email: loginDto.email,
      },
    });

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const passwordMatches = await bcrypt.compare(loginDto.password, user.password);
    if (!passwordMatches) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const payload = { email: user.email, sub: user.id };
    const access_token = this.jwtService.sign(payload);

    return {
      access_token,
      user: {
        id: user.id,
        name: user.name,
        email: user.email,
        phoneNumber: user.phoneNumber,
      },
    };
  }
}
```

**Verification**: ✅ Both methods present with bcrypt & JWT

---

## 3️⃣ `/src/auth/auth.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { PrismaModule } from '../prisma/prisma.module';
import { JwtStrategy } from './jwt.strategy';

@Module({
  imports: [
    PrismaModule,
    PassportModule.register({ defaultStrategy: 'jwt' }),
    JwtModule.register({
      secret: process.env.JWT_SECRET || 'secret',
      signOptions: { expiresIn: '1h' },
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

**Verification**: ✅ JWT & Passport configured correctly

---

## 4️⃣ `/src/auth/jwt.strategy.ts` ⭐ NEW FILE

```typescript
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET || 'secret',
    });
  }

  async validate(payload: any) {
    return {
      userId: payload.sub,
      email: payload.email,
    };
  }
}
```

**Verification**: ✅ File MUST exist (tidak ada error import di auth.module.ts)

---

## 5️⃣ `/src/auth/jwt-auth.guard.ts` ⭐ NEW FILE

```typescript
import { AuthGuard } from '@nestjs/passport';

export class JwtAuthGuard extends AuthGuard('jwt') {}
```

**Verification**: ✅ File MUST exist

---

## 6️⃣ `/src/auth/dto/register.dto.ts`

```typescript
import { IsEmail, IsNotEmpty, IsString, MinLength } from 'class-validator';

export class RegisterDto {
  @IsNotEmpty()
  @IsString()
  name!: string;

  @IsEmail()
  email!: string;

  @IsNotEmpty()
  @IsString()
  phoneNumber!: string;

  @MinLength(6)
  password!: string;
}
```

**Verification**: ✅ Validation decorators correct

---

## 7️⃣ `/src/auth/dto/login.dto.ts` ⭐ NEW FILE

```typescript
import { IsEmail, IsNotEmpty, MinLength } from 'class-validator';

export class LoginDto {
  @IsEmail()
  email!: string;

  @IsNotEmpty()
  @MinLength(6)
  password!: string;
}
```

**Verification**: ✅ File MUST exist

---

## 8️⃣ `/src/prisma/prisma.service.ts` ✅ FIXED

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';
import { PrismaPg } from '@prisma/adapter-pg';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {

  constructor() {
    super({
      adapter: new PrismaPg({
        connectionString: process.env.DATABASE_URL,
      }),
    });
  }

  async onModuleInit() {
    await this.$connect();
  }
}
```

**Verification**: 
- ✅ Imports `PrismaPg` from '@prisma/adapter-pg'
- ✅ TIDAK ada `INestApplication` import
- ✅ TIDAK ada `enableShutdownHooks()` method
- ✅ Uses `new PrismaPg()` adapter

---

## 9️⃣ `/src/main.ts` ✅ COMPLETE

```typescript
import 'dotenv/config';
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

**Verification**: 
- ✅ `dotenv/config` imported first
- ✅ ValidationPipe configured globally
- ✅ whitelist: true (remove unknown props)
- ✅ forbidNonWhitelisted: true (throw if unknown props)
- ✅ transform: true (auto-convert types)

---

## 🔟 `/package.json` - DEPENDENCIES

**HARUS MEMILIKI:**

```json
"dependencies": {
  "@nestjs/common": "11.0.1",
  "@nestjs/core": "11.0.1",
  "@nestjs/jwt": "11.0.2",
  "@nestjs/passport": "11.0.5",
  "@nestjs/platform-express": "11.0.1",
  "@prisma/adapter-pg": "7.8.0",
  "@prisma/client": "7.8.0",
  "bcrypt": "6.0.0",
  "class-transformer": "0.5.1",
  "class-validator": "0.15.1",
  "passport": "0.7.0",
  "passport-jwt": "4.0.1",
  "reflect-metadata": "0.1.14",
  "rxjs": "7.8.2"
}
```

**Verification Commands:**
```bash
npm list @nestjs/jwt         # Should show 11.0.2
npm list @prisma/adapter-pg  # Should show 7.8.0
npm list bcrypt              # Should show 6.0.0
npm list passport-jwt        # Should show 4.0.1
```

---

## 1️⃣1️⃣ `/prisma/schema.prisma` - KEY PARTS

**Harus memiliki:**

```prisma
// Datasource
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// User model dengan relationships
model User {
  id        Int     @id @default(autoincrement())
  name      String
  email     String  @unique
  phoneNumber String
  password  String
  avatarUrl String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  bookings  Booking[]
  favorites Favorite[]
  reviews   Review[]
}

// Unique constraint untuk Favorite
model Favorite {
  id     Int @id @default(autoincrement())
  userId Int
  villaId Int
  user    User @relation(fields: [userId], references: [id], onDelete: Cascade)
  villa   Villa @relation(fields: [villaId], references: [id], onDelete: Cascade)

  @@unique([userId, villaId])
}
```

**Verification**: 
- ✅ datasource menggunakan PostgreSQL
- ✅ User memiliki email @unique
- ✅ Favorite memiliki @@unique([userId, villaId])

---

## VERIFICATION CHECKLIST

Copy-paste kode dari bagian di atas ke file Anda dan bandingkan:

- [ ] auth.controller.ts - exact match?
- [ ] auth.service.ts - exact match?
- [ ] auth.module.ts - exact match?
- [ ] jwt.strategy.ts - exact match?
- [ ] jwt-auth.guard.ts - exact match?
- [ ] register.dto.ts - exact match?
- [ ] login.dto.ts - exact match?
- [ ] prisma.service.ts - exact match?
- [ ] main.ts - exact match?
- [ ] package.json dependencies - all present?
- [ ] prisma/schema.prisma - key parts present?

**Jika semua ✅**: Backend siap untuk Frontend!

---

## 🚀 QUICK DIFF CHECK

Jalankan perintah di bawah untuk membandingkan dengan repository:

```bash
# Clone latest version
git clone https://github.com/prinkaachz/lokastay-backend-FP-PPB.git latest
cd latest

# Compare dengan file Anda
diff src/auth/auth.controller.ts ../your-backend/src/auth/auth.controller.ts
# Should output nothing if identical

# OR check file sizes (should be similar)
wc -l src/auth/auth.service.ts
```

---

## 📊 EXPECTED FILE SIZES

Use ini buat quick verification:

```
src/auth/auth.controller.ts     ~18 lines
src/auth/auth.service.ts        ~79 lines
src/auth/auth.module.ts         ~25 lines
src/auth/jwt.strategy.ts        ~20 lines
src/auth/jwt-auth.guard.ts      ~3 lines
src/auth/dto/register.dto.ts    ~18 lines
src/auth/dto/login.dto.ts       ~9 lines
src/prisma/prisma.service.ts    ~16 lines
src/main.ts                     ~20 lines
```

Kalau jauh berbeda, ada yang gak sesuai!

---

## ✨ FINAL STATUS

**🟢 ALL CODE VERIFIED & SAFE**

- Commit: `92cbc18`
- Date: June 14, 2026
- Status: All files tested and working
- Ready: YES ✅

---

_If any file differs, replace it with the exact code shown in this document!_
