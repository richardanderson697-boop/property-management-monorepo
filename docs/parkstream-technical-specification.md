# ParkStream Platform - Technical Specification

## Executive Summary

This document outlines the architecture, implementation plan, and technical details for extracting shared code from RentStream and building ParkStream as a separate but code-sharing platform within the existing monorepo.

---

## Table of Contents

1. [Monorepo Structure](#monorepo-structure)
2. [Shared Packages Architecture](#shared-packages-architecture)
3. [Database Strategy](#database-strategy)
4. [API Architecture](#api-architecture)
5. [Mobile App Structure](#mobile-app-structure)
6. [Implementation Phases](#implementation-phases)
7. [Code Examples](#code-examples)

---

## 1. Monorepo Structure

### Final Directory Layout

```
rentstream-monorepo/
├── apps/
│   ├── api/                          # RentStream API (existing)
│   ├── mobile/                       # RentStream Mobile (existing)
│   ├── web/                          # RentStream Web (existing)
│   ├── parkstream-api/               # NEW: ParkStream API
│   ├── parkstream-mobile/            # NEW: ParkStream Mobile
│   └── parkstream-web/               # NEW: ParkStream Web
│
├── packages/
│   ├── database/                     # EXISTING: Extend for MHP
│   ├── shared-auth/                  # NEW: Authentication & Authorization
│   ├── shared-payments/              # NEW: Stripe & Payment Processing
│   ├── shared-notifications/         # NEW: Email/SMS/Push
│   ├── shared-storage/               # NEW: S3/CloudFront Document Storage
│   ├── shared-ui-components/         # NEW: React/React Native Components
│   ├── shared-types/                 # NEW: TypeScript Interfaces
│   ├── shared-utils/                 # NEW: Common Utilities
│   └── shared-validation/            # NEW: Zod/Class-Validator Schemas
│
├── infrastructure/
│   ├── docker/
│   │   ├── docker-compose.yml        # Update with ParkStream services
│   │   ├── rentstream.Dockerfile
│   │   └── parkstream.Dockerfile
│   └── terraform/                    # Cloud infrastructure (if applicable)
│
├── package.json                      # Root workspace config
├── tsconfig.base.json                # Shared TypeScript config
└── .env.example
```

### Package.json Workspace Configuration

```json
{
  "name": "property-management-monorepo",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "dev:rentstream-api": "npm run dev --workspace=apps/api",
    "dev:parkstream-api": "npm run dev --workspace=apps/parkstream-api",
    "dev:rentstream-mobile": "npm run start --workspace=apps/mobile",
    "dev:parkstream-mobile": "npm run start --workspace=apps/parkstream-mobile",
    "build:all": "npm run build --workspaces",
    "test:all": "npm run test --workspaces",
    "lint:all": "npm run lint --workspaces"
  }
}
```

---

## 2. Shared Packages Architecture

### 2.1 Shared Auth Package

**Location**: `packages/shared-auth/`

**Purpose**: Handle authentication, JWT generation, role-based access control, and session management for both platforms.

**Structure**:
```
packages/shared-auth/
├── src/
│   ├── index.ts
│   ├── auth.service.ts
│   ├── jwt.service.ts
│   ├── guards/
│   │   ├── roles.guard.ts
│   │   └── jwt-auth.guard.ts
│   ├── decorators/
│   │   ├── current-user.decorator.ts
│   │   └── roles.decorator.ts
│   ├── strategies/
│   │   ├── jwt.strategy.ts
│   │   └── local.strategy.ts
│   └── types/
│       ├── user.interface.ts
│       └── auth-config.interface.ts
├── package.json
└── tsconfig.json
```

**Key Exports**:
```typescript
// packages/shared-auth/src/index.ts
export { AuthService } from './auth.service';
export { JwtService } from './jwt.service';
export { JwtAuthGuard } from './guards/jwt-auth.guard';
export { RolesGuard } from './guards/roles.guard';
export { CurrentUser } from './decorators/current-user.decorator';
export { Roles } from './decorators/roles.decorator';
export * from './types';
```

---

### 2.2 Shared Payments Package

**Location**: `packages/shared-payments/`

**Purpose**: Stripe integration, payment processing, subscription management, and invoice generation.

**Structure**:
```
packages/shared-payments/
├── src/
│   ├── index.ts
│   ├── stripe.service.ts
│   ├── payment.service.ts
│   ├── subscription.service.ts
│   ├── webhook.service.ts
│   ├── invoices/
│   │   ├── invoice-generator.service.ts
│   │   └── pdf-generator.service.ts
│   ├── types/
│   │   ├── payment.interface.ts
│   │   └── subscription.interface.ts
│   └── utils/
│       ├── calculate-late-fees.ts
│       └── prorate-charges.ts
├── package.json
└── tsconfig.json
```

**Key Features**:
- ACH/Credit Card processing
- Recurring subscription billing
- Late fee calculations
- Webhook event handling
- Receipt/Invoice PDF generation

---

### 2.3 Shared Notifications Package

**Location**: `packages/shared-notifications/`

**Purpose**: Email (SendGrid/AWS SES), SMS (Twilio), and Push notifications.

**Structure**:
```
packages/shared-notifications/
├── src/
│   ├── index.ts
│   ├── notification.service.ts
│   ├── providers/
│   │   ├── email.provider.ts        # SendGrid/SES
│   │   ├── sms.provider.ts          # Twilio
│   │   └── push.provider.ts         # Firebase/OneSignal
│   ├── templates/
│   │   ├── email/
│   │   │   ├── payment-received.hbs
│   │   │   ├── lease-expiring.hbs
│   │   │   └── maintenance-update.hbs
│   │   └── sms/
│   │       ├── payment-reminder.ts
│   │       └── emergency-alert.ts
│   └── types/
│       └── notification.interface.ts
├── package.json
└── tsconfig.json
```

---

### 2.4 Shared Storage Package

**Location**: `packages/shared-storage/`

**Purpose**: S3 uploads, CloudFront CDN, document signing, and file management.

**Structure**:
```
packages/shared-storage/
├── src/
│   ├── index.ts
│   ├── s3.service.ts
│   ├── cloudfront.service.ts
│   ├── document.service.ts
│   ├── utils/
│   │   ├── file-validation.ts
│   │   ├── image-resize.ts
│   │   └── generate-presigned-url.ts
│   └── types/
│       └── storage.interface.ts
├── package.json
└── tsconfig.json
```

**Key Features**:
- Secure file uploads (leases, photos, IDs)
- Image resizing/optimization
- Pre-signed URL generation
- Document versioning
- Automatic cleanup of old files

---

### 2.5 Shared UI Components Package

**Location**: `packages/shared-ui-components/`

**Purpose**: Reusable React and React Native components for consistent UI across platforms.

**Structure**:
```
packages/shared-ui-components/
├── src/
│   ├── index.ts
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.native.tsx
│   │   │   └── Button.styles.ts
│   │   ├── Input/
│   │   ├── Modal/
│   │   ├── Card/
│   │   ├── DataTable/
│   │   ├── Chart/
│   │   └── PaymentForm/
│   ├── hooks/
│   │   ├── useDebounce.ts
│   │   ├── useAsync.ts
│   │   └── useForm.ts
│   ├── theme/
│   │   ├── colors.ts
│   │   ├── typography.ts
│   │   └── spacing.ts
│   └── utils/
│       └── format-currency.ts
├── package.json
└── tsconfig.json
```

**Platform-Specific Files**:
```typescript
// Button.tsx (Web)
import React from 'react';
export const Button = ({ children, ...props }) => (
  <button className="btn" {...props}>{children}</button>
);

// Button.native.tsx (Mobile)
import React from 'react';
import { TouchableOpacity, Text } from 'react-native';
export const Button = ({ children, ...props }) => (
  <TouchableOpacity {...props}>
    <Text>{children}</Text>
  </TouchableOpacity>
);
```

---

### 2.6 Shared Types Package

**Location**: `packages/shared-types/`

**Purpose**: TypeScript interfaces and types shared across all apps.

**Structure**:
```
packages/shared-types/
├── src/
│   ├── index.ts
│   ├── entities/
│   │   ├── user.interface.ts
│   │   ├── property.interface.ts
│   │   ├── payment.interface.ts
│   │   ├── lease.interface.ts
│   │   └── maintenance.interface.ts
│   ├── dtos/
│   │   ├── create-user.dto.ts
│   │   ├── update-property.dto.ts
│   │   └── payment-request.dto.ts
│   ├── enums/
│   │   ├── user-role.enum.ts
│   │   ├── property-type.enum.ts
│   │   └── payment-status.enum.ts
│   └── api-responses/
│       ├── success.response.ts
│       └── error.response.ts
├── package.json
└── tsconfig.json
```

**Example Enums**:
```typescript
// packages/shared-types/src/enums/property-type.enum.ts
export enum PropertyType {
  APARTMENT = 'APARTMENT',
  HOUSE = 'HOUSE',
  CONDO = 'CONDO',
  MOBILE_HOME_PARK = 'MOBILE_HOME_PARK', // ParkStream specific
  COMMERCIAL = 'COMMERCIAL'
}

// packages/shared-types/src/enums/user-role.enum.ts
export enum UserRole {
  LANDLORD = 'LANDLORD',
  TENANT = 'TENANT',
  MAINTENANCE = 'MAINTENANCE',
  PARK_MANAGER = 'PARK_MANAGER',      // ParkStream specific
  ADMIN = 'ADMIN'
}
```

---

### 2.7 Shared Validation Package

**Location**: `packages/shared-validation/`

**Purpose**: Input validation schemas using Zod or class-validator.

**Structure**:
```
packages/shared-validation/
├── src/
│   ├── index.ts
│   ├── schemas/
│   │   ├── user.schema.ts
│   │   ├── property.schema.ts
│   │   ├── payment.schema.ts
│   │   └── lease.schema.ts
│   └── validators/
│       ├── email.validator.ts
│       ├── phone.validator.ts
│       └── date.validator.ts
├── package.json
└── tsconfig.json
```

**Example Schema**:
```typescript
// packages/shared-validation/src/schemas/payment.schema.ts
import { z } from 'zod';

export const PaymentSchema = z.object({
  amount: z.number().positive(),
  currency: z.enum(['USD', 'CAD']),
  paymentMethod: z.enum(['CARD', 'ACH', 'CASH']),
  description: z.string().optional(),
  metadata: z.record(z.string()).optional()
});

export type PaymentInput = z.infer<typeof PaymentSchema>;
```

---

## 3. Database Strategy

### Option A: Separate Databases (Recommended)

**Approach**: Each platform has its own PostgreSQL database.

**Pros**:
- Complete data isolation
- Independent scaling
- Simpler compliance (different regulations for MHP)
- No risk of cross-contamination

**Cons**:
- Users with accounts on both platforms need separate credentials (mitigated with SSO)
- Slightly more infrastructure

**Schema**:
```
PostgreSQL Instance 1: rentstream_production
  - users
  - properties
  - leases
  - payments
  - maintenance_tickets

PostgreSQL Instance 2: parkstream_production
  - users
  - parks
  - lots
  - leases
  - payments
  - utility_meters
  - utility_bills
  - amenities
```

**Connection Management**:
```typescript
// packages/database/src/config/database.config.ts
export const getDatabaseConfig = (platform: 'rentstream' | 'parkstream') => ({
  type: 'postgres',
  host: process.env[`${platform.toUpperCase()}_DB_HOST`],
  port: parseInt(process.env[`${platform.toUpperCase()}_DB_PORT`]),
  username: process.env[`${platform.toUpperCase()}_DB_USER`],
  password: process.env[`${platform.toUpperCase()}_DB_PASSWORD`],
  database: process.env[`${platform.toUpperCase()}_DB_NAME`],
  entities: [`${__dirname}/../entities/${platform}/**/*.entity.{ts,js}`],
  migrations: [`${__dirname}/../migrations/${platform}/**/*.{ts,js}`]
});
```

---

### Option B: Shared Database with Schemas

**Approach**: Single PostgreSQL instance with separate schemas.

**Structure**:
```sql
-- Database: property_management

-- Shared schema
CREATE SCHEMA shared;
CREATE TABLE shared.users (...);
CREATE TABLE shared.payments (...);

-- RentStream schema
CREATE SCHEMA rentstream;
CREATE TABLE rentstream.properties (...);
CREATE TABLE rentstream.leases (...);

-- ParkStream schema
CREATE SCHEMA parkstream;
CREATE TABLE parkstream.parks (...);
CREATE TABLE parkstream.lots (...);
CREATE TABLE parkstream.utility_meters (...);
```

**When to use**: If you need cross-platform reporting or a single admin dashboard.

---

### Database Migrations

**Location**: `packages/database/`

**Structure**:
```
packages/database/
├── src/
│   ├── migrations/
│   │   ├── rentstream/
│   │   │   ├── 001_create_properties.ts
│   │   │   └── 002_create_leases.ts
│   │   └── parkstream/
│   │       ├── 001_create_parks.ts
│   │       ├── 002_create_lots.ts
│   │       └── 003_create_utility_meters.ts
│   ├── entities/
│   │   ├── rentstream/
│   │   │   ├── property.entity.ts
│   │   │   └── lease.entity.ts
│   │   └── parkstream/
│   │       ├── park.entity.ts
│   │       ├── lot.entity.ts
│   │       └── utility-meter.entity.ts
│   └── seeds/
│       ├── rentstream/
│       └── parkstream/
├── package.json
└── ormconfig.ts
```

---

## 4. API Architecture

### 4.1 ParkStream API Structure

**Location**: `apps/parkstream-api/`

**Base Structure** (mirror RentStream):
```
apps/parkstream-api/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── config/
│   │   └── app.config.ts
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.module.ts
│   │   │   ├── auth.controller.ts
│   │   │   └── auth.service.ts
│   │   ├── parks/
│   │   │   ├── parks.module.ts
│   │   │   ├── parks.controller.ts
│   │   │   ├── parks.service.ts
│   │   │   └── dto/
│   │   ├── lots/
│   │   ├── utilities/                # MHP-SPECIFIC
│   │   │   ├── utilities.module.ts
│   │   │   ├── meters/
│   │   │   │   ├── meters.controller.ts
│   │   │   │   ├── meters.service.ts
│   │   │   │   └── dto/
│   │   │   ├── billing/
│   │   │   │   ├── billing.controller.ts
│   │   │   │   ├── billing.service.ts
│   │   │   │   └── calculators/
│   │   │   │       ├── direct-meter.calculator.ts
│   │   │   │       ├── rubs.calculator.ts
│   │   │   │       └── flat-fee.calculator.ts
│   │   │   └── rate-tables/
│   │   ├── amenities/                # MHP-SPECIFIC
│   │   │   ├── amenities.module.ts
│   │   │   ├── amenities.controller.ts
│   │   │   └── amenities.service.ts
│   │   ├── payments/
│   │   ├── leases/
│   │   └── notifications/
│   ├── common/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── filters/
│   └── cron/
│       ├── utility-billing.cron.ts
│       └── meter-reading-reminders.cron.ts
├── test/
├── package.json
└── tsconfig.json
```

---

### 4.2 Shared Package Integration

**Example: Using Shared Auth in ParkStream API**

```typescript
// apps/parkstream-api/src/modules/auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { AuthService } from '@monorepo/shared-auth';
import { AuthController } from './auth.controller';

@Module({
  imports: [
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: '7d' }
    })
  ],
  providers: [
    {
      provide: AuthService,
      useFactory: (jwtService) => new AuthService(jwtService, {
        platform: 'parkstream',
        sessionDuration: '7d'
      }),
      inject: [JwtModule]
    }
  ],
  controllers: [AuthController],
  exports: [AuthService]
})
export class AuthModule {}
```

**Example: Using Shared Payments**

```typescript
// apps/parkstream-api/src/modules/payments/payment.service.ts
import { Injectable } from '@nestjs/common';
import { StripeService, PaymentService } from '@monorepo/shared-payments';

@Injectable()
export class ParkStreamPaymentService {
  constructor(
    private readonly stripeService: StripeService,
    private readonly paymentService: PaymentService
  ) {}

  async processLotRent(lotId: string, amount: number) {
    // Use shared payment processing
    return this.paymentService.createPayment({
      amount,
      description: `Lot ${lotId} - Monthly Rent`,
      metadata: { lotId, type: 'LOT_RENT' }
    });
  }

  async processUtilityBill(lotId: string, utilityCharges: UtilityCharge[]) {
    const total = utilityCharges.reduce((sum, c) => sum + c.amount, 0);
    
    return this.paymentService.createPayment({
      amount: total,
      description: `Lot ${lotId} - Utility Charges`,
      metadata: {
        lotId,
        type: 'UTILITY_BILL',
        breakdown: JSON.stringify(utilityCharges)
      }
    });
  }
}
```

---

### 4.3 MHP-Specific Modules

#### Utility Billing Module

**Core Functionality**:
- Meter reading capture (manual entry, photo OCR, IoT integration)
- Multiple billing methods (Direct, RUBS, Flat Fee)
- Tiered rate calculations
- Automated billing cycles

**Database Entities**:

```typescript
// packages/database/src/entities/parkstream/utility-meter.entity.ts
import { Entity, Column, ManyToOne } from 'typeorm';

@Entity('utility_meters')
export class UtilityMeter {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'enum', enum: ['WATER', 'ELECTRIC', 'GAS', 'SEWER'] })
  type: UtilityType;

  @Column()
  meterNumber: string;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  currentReading: number;

  @Column({ type: 'timestamp' })
  lastReadingDate: Date;

  @ManyToOne(() => Lot)
  lot: Lot;

  @Column({ type: 'jsonb', nullable: true })
  metadata: {
    manufacturer?: string;
    installDate?: string;
    calibrationDate?: string;
  };
}

// packages/database/src/entities/parkstream/utility-bill.entity.ts
@Entity('utility_bills')
export class UtilityBill {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Lot)
  lot: Lot;

  @Column({ type: 'date' })
  billingPeriodStart: Date;

  @Column({ type: 'date' })
  billingPeriodEnd: Date;

  @Column({ type: 'jsonb' })
  charges: UtilityCharge[]; // [{ type: 'WATER', usage: 1200, rate: 0.05, amount: 60 }]

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  totalAmount: number;

  @Column({ type: 'enum', enum: ['PENDING', 'SENT', 'PAID', 'OVERDUE'] })
  status: BillStatus;

  @Column({ type: 'date' })
  dueDate: Date;
}
```

**Billing Calculators**:

```typescript
// apps/parkstream-api/src/modules/utilities/billing/calculators/direct-meter.calculator.ts
export class DirectMeterCalculator {
  calculate(
    previousReading: number,
    currentReading: number,
    rateTable: RateTable
  ): UtilityCharge {
    const usage = currentReading - previousReading;
    
    // Tiered pricing
    let amount = 0;
    let remainingUsage = usage;
    
    for (const tier of rateTable.tiers) {
      if (remainingUsage <= 0) break;
      
      const tierUsage = Math.min(remainingUsage, tier.maxUsage - tier.minUsage);
      amount += tierUsage * tier.rate;
      remainingUsage -= tierUsage;
    }
    
    return {
      type: rateTable.utilityType,
      usage,
      rate: rateTable.baseRate,
      amount,
      breakdown: this.generateBreakdown(usage, rateTable)
    };
  }
}

// apps/parkstream-api/src/modules/utilities/billing/calculators/rubs.calculator.ts
export class RUBSCalculator {
  calculate(
    totalParkUsage: number,
    totalParkCost: number,
    lot: Lot,
    allLots: Lot[]
  ): UtilityCharge {
    // Ratio Utility Billing System
    // Allocate based on: # of occupants, square footage, or equal split
    
    const allocationFactor = this.calculateAllocationFactor(lot, allLots);
    const lotShare = totalParkCost * allocationFactor;
    
    return {
      type: 'WATER', // or whatever utility
      usage: totalParkUsage * allocationFactor, // Estimated
      amount: lotShare,
      breakdown: {
        method: 'RUBS',
        allocationFactor,
        totalParkCost,
        totalParkUsage
      }
    };
  }
  
  private calculateAllocationFactor(lot: Lot, allLots: Lot[]): number {
    // Example: Equal split
    return 1 / allLots.length;
    
    // Or occupancy-based:
    // const totalOccupants = allLots.reduce((sum, l) => sum + l.occupants, 0);
    // return lot.occupants / totalOccupants;
  }
}
```

**Cron Job for Auto-Billing**:

```typescript
// apps/parkstream-api/src/cron/utility-billing.cron.ts
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';

@Injectable()
export class UtilityBillingCron {
  constructor(
    private readonly utilityBillingService: UtilityBillingService,
    private readonly notificationService: NotificationService
  ) {}

  @Cron('0 0 1 * *') // First of every month
  async generateMonthlyBills() {
    const parks = await this.parkRepository.find();
    
    for (const park of parks) {
      const lots = await this.lotRepository.find({ where: { parkId: park.id } });
      
      for (const lot of lots) {
        const bill = await this.utilityBillingService.generateBill(
          lot,
          new Date(),
          park.billingMethod
        );
        
        await this.notificationService.send({
          to: lot.tenant.email,
          template: 'utility-bill-ready',
          data: { bill, lot }
        });
      }
    }
  }

  @Cron('0 9 25 * *') // 25th of each month at 9am
  async sendMeterReadingReminders() {
    const parks = await this.parkRepository.find({
      where: { requiresManualMeterReading: true }
    });
    
    for (const park of parks) {
      await this.notificationService.sendToManagers(park, {
        template: 'meter-reading-reminder',
        data: { dueDate: 'end of month' }
      });
    }
  }
}
```

---

#### Amenity Booking Module

**Features**:
- Reserve clubhouse, laundry, pool, etc.
- Calendar view of availability
- Booking limits per tenant
- Automated reminders

**Database Entity**:

```typescript
// packages/database/src/entities/parkstream/amenity.entity.ts
@Entity('amenities')
export class Amenity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Park)
  park: Park;

  @Column()
  name: string; // "Clubhouse", "Pool", "Laundry Room"

  @Column({ type: 'text', nullable: true })
  description: string;

  @Column({ type: 'int', default: 1 })
  capacity: number;

  @Column({ type: 'jsonb' })
  availability: {
    monday: { start: string; end: string }[];
    tuesday: { start: string; end: string }[];
    // ... etc
  };

  @Column({ type: 'boolean', default: true })
  requiresReservation: boolean;
}

@Entity('amenity_bookings')
export class AmenityBooking {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Amenity)
  amenity: Amenity;

  @ManyToOne(() => User)
  bookedBy: User;

  @Column({ type: 'timestamp' })
  startTime: Date;

  @Column({ type: 'timestamp' })
  endTime: Date;

  @Column({ type: 'enum', enum: ['CONFIRMED', 'CANCELLED'] })
  status: string;
}
```

---

### 5.3 Shared Components Usage

**Example: Using shared UI components in ParkStream Mobile**

```typescript
// apps/parkstream-mobile/src/screens/tenant/UtilityUsageScreen.tsx
import React, { useEffect } from 'react';
import { ScrollView } from 'react-native';
import { Card, Chart } from '@monorepo/shared-ui-components';
import { useUtilityStore } from '../../store/utilityStore';

export const UtilityUsageScreen = ({ lotId }) => {
  const { usage, fetchUsage } = useUtilityStore();
  
  useEffect(() => {
    fetchUsage(lotId);
  }, [lotId]);
  
  return (
    <ScrollView>
      <Card title="Water Usage (Last 12 Months)">
        <Chart
          type="line"
          data={usage.water}
          xAxis="month"
          yAxis="gallons"
        />
      </Card>
      
      <Card title="Electric Usage (Last 12 Months)">
        <Chart
          type="bar"
          data={usage.electric}
          xAxis="month"
          yAxis="kWh"
        />
      </Card>
    </ScrollView>
  );
};
```

---

## 5. Mobile App Structure

### 5.1 ParkStream Mobile App

**Location**: `apps/parkstream-mobile/`

**Key Screens** (unique to ParkStream):

```
apps/parkstream-mobile/
├── src/
│   ├── navigation/
│   │   ├── AppNavigator.tsx
│   │   ├── TenantStack.tsx
│   │   └── ManagerStack.tsx
│   ├── screens/
│   │   ├── tenant/
│   │   │   ├── LotDetailsScreen.tsx
│   │   │   ├── UtilityUsageScreen.tsx
│   │   │   ├── MeterReadingScreen.tsx       # Photo capture
│   │   │   ├── AmenityBookingScreen.tsx
│   │   │   └── ParkDirectoryScreen.tsx
│   │   └── manager/
│   │       ├── ParkDashboardScreen.tsx
│   │       ├── LotsManagementScreen.tsx
│   │       ├── MeterReadingsScreen.tsx
│   │       ├── UtilityBillingScreen.tsx
│   │       └── BulkNotificationsScreen.tsx
│   ├── components/
│   │   ├── MeterReadingCamera.tsx
│   │   ├── UtilityChart.tsx
│   │   ├── LotCard.tsx
│   │   └── AmenityCalendar.tsx
│   ├── services/
│   │   ├── api/
│   │   │   ├── parkApi.ts
│   │   │   ├── utilityApi.ts
│   │   │   └── amenityApi.ts
│   │   └── storage/
│   ├── store/                                 # Zustand
│   │   ├── parkStore.ts
│   │   ├── utilityStore.ts
│   │   └── amenityStore.ts
│   └── utils/
│       ├── meterOCR.ts                       # Text recognition from photos
│       └── utilityCalculations.ts
├── package.json
└── app.json
```

---

### 5.2 Meter Reading Photo Capture

**Component Example**:

```typescript
// apps/parkstream-mobile/src/components/MeterReadingCamera.tsx
import React, { useState } from 'react';
import { View, Text, Image, TouchableOpacity } from 'react-native';
import { Camera } from 'react-native-vision-camera';
import { processImage } from '../utils/meterOCR';

export const MeterReadingCamera = ({ meterId, onReadingCaptured }) => {
  const [photo, setPhoto] = useState(null);
  const [extractedReading, setExtractedReading] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const handleTakePhoto = async () => {
    const camera = useRef<Camera>(null);
    const photo = await camera.current.takePhoto({
      qualityPrioritization: 'quality',
      enableAutoStabilization: true
    });
    
    setPhoto(photo);
    setLoading(true);
    
    // OCR Processing
    const reading = await processImage(photo.path);
    setExtractedReading(reading);
    setLoading(false);
  };
  
  const handleConfirm = async () => {
    await onReadingCaptured({
      meterId,
      reading: extractedReading,
      photoUri: photo.path,
      timestamp: new Date()
    });
  };
  
  return (
    <View>
      {!photo ? (
        <Camera ref={camera} />
      ) : (
        <View>
          <Image source={{ uri: photo.path }} />
          {loading ? (
            <Text>Processing...</Text>
          ) : (
            <>
              <Text>Detected Reading: {extractedReading}</Text>
              <TouchableOpacity onPress={handleConfirm}>
                <Text>Confirm</Text>
              </TouchableOpacity>
              <TouchableOpacity onPress={() => setPhoto(null)}>
                <Text>Retake</Text>
              </TouchableOpacity>
            </>
          )}
        </View>
      )}
    </View>
  );
};

// apps/parkstream-mobile/src/utils/meterOCR.ts
import TextRecognition from '@react-native-ml-kit/text-recognition';

export const processImage = async (imagePath: string): Promise<number> => {
  const result = await TextRecognition.recognize(imagePath);
  
  // Extract numbers from recognized text
  const numbers = result.text.match(/\d+/g);
  
  if (!numbers || numbers.length === 0) {
    throw new Error('No numbers detected in image');
  }
  
  // Find the longest number sequence (usually the meter reading)
  const longestNumber = numbers.reduce((a, b) => 
    a.length > b.length ? a : b
  );
  
  return parseFloat(longestNumber);
};