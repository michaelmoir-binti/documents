# Setup the `family` project

## Prerequisites

<!-- Add prerequisites here if any -->

## Installation Options

There are two primary ways to set up the `family` project for development:

### [Onboarding: Setting Up Your Mac For Local Development](Local-installation-and-setup.md)

Local installation runs the application and all services directly on your development machine. This approach offers:

- **Faster iteration**: No network latency when accessing services
- **Full control**: Direct access to all services and databases
- **Offline development**: Can work without network connectivity
- **Easier debugging**: Direct access to logs and processes
- **Resource efficiency**: No need to maintain a separate VM

**Best for**: Developers who want the fastest development experience and have the necessary tools installed locally (Docker, PostgreSQL, Redis, etc.).

### [Onboarding: Setting Up Your Mac For VM Development](VM-installation-and-setup.md)

VM installation runs the application in a virtualized environment that closely mirrors production. This approach offers:

- **Production-like environment**: Matches production infrastructure more closely
- **Isolation**: Keeps development environment separate from your local machine
- **Consistency**: Same environment across team members
- **Easier onboarding**: Less local setup required
- **Resource isolation**: Development work doesn't impact your local machine's resources

**Best for**: Developers who prefer a containerized approach, want production parity, or have limited local resources.

