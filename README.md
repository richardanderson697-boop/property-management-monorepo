# Property Management Monorepo

This repository contains the codebase for the RentStream and ParkStream platforms, designed to manage property rentals and parking spaces, respectively. It is structured as a monorepo to facilitate code sharing and maintainability.

## Project Structure

- **apps/**: Contains the application-specific code for both RentStream and ParkStream.
  - **api/**: RentStream API.
  - **mobile/**: RentStream Mobile application.
  - **web/**: RentStream Web application.
  - **parkstream-api/**: ParkStream API.
  - **parkstream-mobile/**: ParkStream Mobile application.
  - **parkstream-web/**: ParkStream Web application.

- **packages/**: Contains shared code packages used across applications.
  - **shared-auth/**: Authentication and authorization.
  - **shared-payments/**: Payment processing with Stripe.
  - **shared-notifications/**: Email, SMS, and push notifications.
  - **shared-storage/**: Document storage using S3/CloudFront.
  - **shared-ui-components/**: UI components for React/React Native.
  - **shared-types/**: TypeScript interfaces.
  - **shared-utils/**: Common utilities.
  - **shared-validation/**: Validation schemas using Zod/Class-Validator.

- **infrastructure/**: Contains infrastructure-related configurations.
  - **docker/**: Docker configurations.
  - **terraform/**: Cloud infrastructure setup (if applicable).

- **package.json**: Root workspace configuration.
- **tsconfig.base.json**: Shared TypeScript configuration.
- **.env.example**: Example environment configuration file.

## Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/property-management-monorepo.git
   cd property-management-monorepo
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Run applications:**
   - RentStream API: `npm run dev:rentstream-api`
   - ParkStream API: `npm run dev:parkstream-api`
   - RentStream Mobile: `npm run dev:rentstream-mobile`
   - ParkStream Mobile: `npm run dev:parkstream-mobile`

## Usage

Use the provided scripts in `package.json` to build, test, and lint the entire workspace or individual applications.

## Contributing

Please read the contribution guidelines before making a pull request.

## License

This project is licensed under the MIT License.