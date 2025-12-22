# Third-Party Libraries in the Family Project

This document provides a comprehensive breakdown of all third-party libraries used in the Family application, organized by category and purpose. This serves as a reference for developers and can be used for onboarding new team members.

---

## Table of Contents

1. [Ruby/Rails Core](#rubyrails-core)
2. [Authentication & Authorization](#authentication--authorization)
3. [Background Jobs & Queues](#background-jobs--queues)
4. [GraphQL](#graphql)
5. [Frontend JavaScript/React](#frontend-javascriptreact)
6. [File Handling & Storage](#file-handling--storage)
7. [Document Generation & Processing](#document-generation--processing)
8. [Email](#email)
9. [Data Validation & Formatting](#data-validation--formatting)
10. [Internationalization (i18n)](#internationalization-i18n)
11. [Feature Flags](#feature-flags)
12. [Monitoring & Observability](#monitoring--observability)
13. [External Service Integrations](#external-service-integrations)
14. [Search & Indexing](#search--indexing)
15. [Data Processing](#data-processing)
16. [State Management & Business Logic](#state-management--business-logic)
17. [Data Versioning & Auditing](#data-versioning--auditing)
18. [Admin Interface](#admin-interface)
19. [Development & Debugging](#development--debugging)
20. [Testing (Ruby)](#testing-ruby)
21. [Testing (JavaScript)](#testing-javascript)
22. [Build Tools & Bundling](#build-tools--bundling)
23. [Security](#security)
24. [Utilities](#utilities)

---

## Ruby/Rails Core

### Web Framework
- **Rails** (~> 7.2.0) - The main web application framework
- **Puma** (< 6) - High-performance web server for Rails
- **Rack** (~> 2.2.14) - Ruby web server interface, provides middleware

### Database
- **pg** - PostgreSQL database adapter
- **sqlite3** - SQLite database (used in development/testing)
- **activerecord-postgis-adapter** - Adds PostGIS geospatial features to ActiveRecord
- **scenic** - Database views support
- **active_record_union** - UNION queries in ActiveRecord
- **strong_migrations** - Helps prevent unsafe database migrations

---

## Authentication & Authorization

- **devise** (4.9.4) - Flexible authentication solution for Rails
- **devise_masquerade** - Allows admins to impersonate users
- **devise-i18n** - Internationalization for Devise
- **omniauth-saml** - SAML single sign-on authentication
- **omniauth-google-oauth2** - Google OAuth2 authentication
- **pundit** - Authorization framework (policy-based)

---

## Background Jobs & Queues

- **sidekiq-pro** - Professional version of Sidekiq background job processor
- **sidekiq-ent** - Sidekiq Enterprise features
- **sidekiq-unique-jobs** - Ensures jobs run only once
- **sidekiq-status** - Track status of background jobs
- **sidekiq-throttled** (~> 1.5) - Rate limiting for background jobs
- **redis** (~> 4.0) - Redis client for caching and job queues
- **redis-rails** - Redis integration with Rails

---

## GraphQL

- **graphql** (< 2.0.0) - GraphQL server implementation for Ruby
- **apollo_upload_server** - Handles file uploads in GraphQL

---

## Frontend JavaScript/React

### React Ecosystem
- **react** (^17.0.2) - JavaScript library for building user interfaces
- **react-dom** (^17.0.2) - React DOM renderer
- **react-rails** (~> 2.7.0) - React integration with Rails
- **@apollo/client** (^3.5.5) - GraphQL client for React
- **apollo-upload-client** (^13.0.0) - File upload support for Apollo Client

### UI Components
- **react-select** (^5.10.1) - Flexible select dropdown component
- **react-modal** (^3.14.4) - Modal dialogs
- **react-beautiful-dnd** (^12.2.0) - Drag and drop functionality
- **react-table** (^6.10.0) - Data tables
- **react-tabs** (^3.0.0) - Tabbed interfaces
- **react-dropzone** (^14.2.3) - File upload drag-and-drop zones
- **react-loading-overlay** (^1.0.1) - Loading state overlays
- **react-toastify** (^9.1.2) - Toast notifications
- **react-calendar** (^3.7.0) - Calendar component
- **react-chartjs-2** (^5.2.0) - Chart.js React wrapper
- **chart.js** (^4.4.0) - Charting library

### Rich Text & Documents
- **react-quill-abrarhayat** (^2.0.0) - Rich text editor (Quill wrapper)
- **quill-image-uploader** (^1.2.4) - Image upload plugin for Quill
- **react-pdf** (^4.2.0) - PDF viewer component
- **pdfjs-dist** (^2.1.266) - PDF.js library for rendering PDFs
- **mammoth** (^1.9.0) - Converts Word documents to HTML
- **signature_pad** (^3.0.0-beta.3) - Digital signature capture

### Utilities
- **lodash** (^4.17.21) - JavaScript utility library
- **luxon** (^1.28.1) - Modern date/time library
- **moment** (^2.29.4) - Date/time manipulation (legacy, being phased out)
- **classnames** (^2.2.5) - Utility for conditionally joining CSS class names
- **history** (^5.2.0) - Browser history management
- **urijs** (^1.19.11) - URL manipulation library

---

## File Handling & Storage

### File Uploads
- **shrine** - Modern file upload library (preferred, replacing Refile)
- **refile** - File upload library (legacy, custom fork from binti-family)
- **refile-fog** - Cloud storage backend for Refile
- **refile-mini_magick** - Image processing for Refile
- **shrine-google_cloud_storage** - Google Cloud Storage backend for Shrine

### Image Processing
- **mini_magick** - ImageMagick wrapper for Ruby
- **image_processing** - High-level image processing pipeline
- **chunky_png** - PNG manipulation library

### File Validation
- **clamby** - Virus scanning integration
- **ruby-filemagic** - File type detection
- **mime-types** - MIME type detection

### Cloud Storage
- **fog-google** - Google Cloud Storage integration
- **google-cloud-bigquery** - Google BigQuery integration

---

## Document Generation & Processing

### PDF
- **prawn** - PDF generation library
- **pdf-reader** - PDF parsing and reading
- **pdf-forms** (~> 1.5.1) - PDF form filling (PDFTK wrapper)

### Word/Excel
- **docx** (0.8.0) - Word document manipulation
- **caxlsx** - Excel spreadsheet generation
- **rubyXL** - Excel file parsing

### Templates
- **liquid** (~> 4.0) - Template engine for PDF generation

---

## Email

- **premailer** - Inline CSS for HTML emails
- **premailer-rails** - Rails integration for Premailer
- **css_parser** - CSS parsing for email styling

---

## Data Validation & Formatting

- **validates_zipcode** - Zipcode/postal code validation
- **phonelib** - Phone number validation and formatting
- **countries** - Country and subdivision data
- **geocoder** - Address geocoding (converts addresses to coordinates)
- **postcode-validator** - Postal code validation
- **libphonenumber-js** (^1.9.50) - Phone number formatting in JavaScript

---

## Internationalization (i18n)

- **i18n** - Core internationalization framework
- **rails-i18n** (~> 7.0.0) - Rails i18n translations
- **i18n-js** - Exports i18n translations to JavaScript
- **i18n-tasks** - i18n management and missing translation detection
- **json_translate** - JSON column translations
- **http_accept_language** - Detects user's preferred language from browser

---

## Feature Flags

- **flipper** (~> 0.28.1) - Feature flagging system
- **flipper-active_record** - ActiveRecord adapter for Flipper
- **flipper-ui** - Admin UI for managing feature flags
- **flipper-active_support_cache_store** - Cache store for Flipper

---

## Monitoring & Observability

- **honeybadger** - Error tracking and monitoring
- **opentelemetry-sdk** - OpenTelemetry SDK for distributed tracing
- **opentelemetry-exporter-otlp** - OTLP exporter for OpenTelemetry
- **opentelemetry-instrumentation-*** - Instrumentation for various libraries (Rails, GraphQL, Redis, Sidekiq, etc.)
- **prometheus_exporter** (~> 2) - Prometheus metrics exporter
- **coverband** - Runtime code coverage tracking

---

## External Service Integrations

### Communication
- **twilio-ruby** - SMS and voice communication
- **hellosign-ruby-sdk** (~> 3.7.3) - HelloSign e-signature API
- **hellosign-embedded** (^2.6.0) - HelloSign embedded signing widget

### AI Services
- **ruby-openai** - OpenAI API client
- **anthropic** - Anthropic (Claude) API client
- **assemblyai** - AssemblyAI transcription service
- **google-cloud-text_to_speech** - Google Text-to-Speech API
- **google-cloud-translate** - Google Translate API

### Analytics
- **google-apis-analytics_v3** (~> 0.12.0) - Google Analytics API

### Integrations
- **net-sftp** - SFTP client for file transfers
- **net-ssh** - SSH client library
- **faraday** (~> 2) - HTTP client library
- **faraday-multipart** - Multipart request support for Faraday

---

## Search & Indexing

- **elasticsearch-model** (~> 8.0) - Elasticsearch integration for ActiveRecord models
- **elasticsearch-rails** (~> 8.0) - Rails integration for Elasticsearch

---

## Data Processing

- **oj** - Fast JSON parser/serializer
- **json** - JSON handling
- **psych** - Fast YAML library
- **papaparse** (^5.5.2) - CSV parsing library
- **descriptive_statistics** - Statistical calculations

---

## State Management & Business Logic

- **aasm** - State machine library
- **rrule** - Calendar recurrence rule parsing (Ruby)
- **rrule** (^2.7.1) - Calendar recurrence rules (JavaScript)
- **dentaku** - Formula computation engine
- **expr-eval** (^2.0.2) - Mathematical expression evaluation

---

## Data Versioning & Auditing

- **paper_trail** (~> 15.0) - Versioning and audit trail
- **paper_trail-association_tracking** (~> 2.2) - Tracks associations in PaperTrail
- **paranoia** (~> 3.0.1) - Soft deletion (records aren't actually deleted)

---

## Admin Interface

- **activeadmin** (~> 3.2.0) - Admin interface framework
- **formtastic** - Form builder for ActiveAdmin
- **responders** (~> 3.0) - Responder pattern for controllers

---

## Development & Debugging

- **pry-rails** - Interactive debugging console
- **better_errors** - Better error pages in development
- **binding_of_caller** - Call stack inspection
- **annotate** - Auto-generates model annotations
- **amazing_print** - Pretty printing for Ruby objects
- **tapioca** - Sorbet type generation tool
- **sorbet** (0.5.12010) - Static type checker for Ruby
- **sorbet-runtime** - Runtime type checking
- **graphiql-rails** - GraphQL IDE for development

---

## Testing (Ruby)

- **rspec-rails** - RSpec testing framework for Rails
- **factory_bot_rails** - Test data factories
- **capybara** - Integration testing framework
- **selenium-webdriver** - Browser automation for tests
- **shoulda-matchers** - RSpec matchers
- **vcr** - Records HTTP interactions for testing
- **webmock** (3.15.2) - HTTP request stubbing
- **database_cleaner** - Cleans database between tests
- **bullet** - Detects N+1 queries in tests
- **brakeman** - Security vulnerability scanner
- **rubocop** - Ruby code linter
- **simplecov** - Code coverage tool

---

## Testing (JavaScript)

- **jest** (^28.1.3) - JavaScript test runner
- **@testing-library/react** (12.1.2) - React component testing utilities
- **@testing-library/jest-dom** (^5.16.4) - Custom DOM matchers for Jest
- **@playwright/test** (^1.56.0) - End-to-end testing framework
- **cypress** (^13.6.1) - End-to-end testing (legacy, being phased out)
- **@storybook/react** (^6.5.8) - Component development environment

---

## Build Tools & Bundling

### JavaScript Bundling
- **shakapacker** (~> 7.2) - Webpack integration for Rails
- **webpack** (^5.90.0) - JavaScript module bundler
- **babel** - JavaScript transpiler (converts modern JS to compatible versions)
- **terser** (~> 1.2) - JavaScript minification

### CSS/SCSS
- **sassc-rails** - SCSS compilation for Rails
- **autoprefixer-rails** - Automatically adds vendor prefixes to CSS
- **sprockets** (>= 4.0.0) - Asset pipeline for Rails

---

## Security

- **secure_headers** - HTTP security headers
- **jwt** - JSON Web Token handling
- **omniauth-rails_csrf_protection** - CSRF protection for OmniAuth
- **bundler-audit** - Security audit for gem dependencies

---

## Utilities

- **chronic** - Natural language date parsing
- **hashdiff** - Compares hashes and shows differences
- **attribute_normalizer** - Normalizes model attributes
- **auto_strip_attributes** - Automatically strips whitespace from attributes
- **nilify_blanks** - Converts blank strings to nil
- **str_enum** - String-based enums (makes enum columns readable from non-Rails tools)
- **money-rails** - Money/currency handling
- **storext** - Typecasting for JSONB and HStore columns
- **browserslist_useragent** - Browser detection
- **user_agent_parser** - User agent string parsing
- **timecop** - Time mocking for tests
- **faker** - Fake data generation for tests

---

## Key Patterns & Notes

### File Uploads
The application is transitioning from **Refile** (legacy) to **Shrine** (preferred). New file upload features should use Shrine.

### Date/Time Libraries
The application uses both **Luxon** (preferred) and **Moment** (legacy). New code should use Luxon.

### Testing
- **Playwright** is the preferred E2E testing framework (replacing Cypress)
- **Jest** is used for JavaScript unit/component tests
- **RSpec** is used for Ruby unit/integration tests

### Type Checking
The application uses **Sorbet** for static type checking in Ruby. Type annotations are encouraged for new code.

### Background Jobs
All background processing uses **Sidekiq**. Jobs should be idempotent and handle failures gracefully.

### Authorization
All authorization is handled through **Pundit** policies. GraphQL queries and mutations must call `authorize` or use `policy_scope`.

---

## Version Management

- **Ruby**: 3.3.7
- **Rails**: ~> 7.2.0
- **Node.js**: Managed via `.tool-versions` (check file for current version)
- **Package Manager**: Yarn 3.8.7

---

## Additional Resources

- See `AGENTS.md` in the project root for development guidelines
- See `app/lib/services/AGENTS.md` for Service Object patterns
- See `app/graphql/AGENTS.md` for GraphQL patterns
- See `app/javascript/AGENTS.md` for JavaScript/React patterns

---

*Last Updated: Based on current Gemfile and package.json*
*For questions or updates to this document, please contact the development team.*
