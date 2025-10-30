# SSH Key Manager - Development Roadmap

## Project Overview
Transform this repository into a comprehensive SSH Key Manager system that allows remote servers to retrieve, validate, and manage SSH keys through a centralized API service.

## Architecture Goals
- **Centralized Key Management**: Single source of truth for SSH public keys
- **Remote Server Integration**: API endpoints for servers to fetch and validate keys
- **Security**: Secure authentication and authorization for API access
- **Scalability**: Support for multiple users, servers, and key pairs
- **Audit Trail**: Complete logging of all key operations

---

## Phase 1: Foundation & Core Framework

### 1.1 Project Setup
- [ ] Initialize PHP project structure
  - [ ] Create `composer.json` with required dependencies
  - [ ] Set up PSR-4 autoloading
  - [ ] Configure development dependencies (PHPUnit, PHP-CS-Fixer)
- [ ] Set up directory structure:
  ```
  /src
    /Controllers
    /Models
    /Services
    /Middleware
    /Database
  /config
  /public
  /tests
  /storage
    /keys
    /logs
  /vendor
  ```
- [ ] Create `.gitignore` for PHP projects
- [ ] Set up environment configuration (`.env.example`)

### 1.2 Core Dependencies
- [ ] Install Composer packages:
  - [ ] Slim Framework or similar lightweight PHP framework
  - [ ] PDO/Eloquent ORM for database
  - [ ] PHP-JWT for authentication tokens
  - [ ] Monolog for logging
  - [ ] phpseclib for SSH key validation
  - [ ] Guzzle for HTTP client (testing)
  - [ ] PHPUnit for testing

### 1.3 Database Schema Design
- [ ] Design database tables:
  - [ ] `users` - System users who can manage keys
  - [ ] `servers` - Registered remote servers
  - [ ] `ssh_keys` - SSH public/private key pairs
  - [ ] `key_assignments` - Links between keys, users, and servers
  - [ ] `api_tokens` - Authentication tokens for remote servers
  - [ ] `audit_logs` - Activity logging for security
- [ ] Create database migration system
- [ ] Write initial migration files

---

## Phase 2: Core API Development

### 2.1 Authentication System
- [ ] Implement API token authentication
  - [ ] Token generation endpoint
  - [ ] Token validation middleware
  - [ ] Token revocation
  - [ ] Rate limiting per token
- [ ] Implement JWT-based authentication for web UI
- [ ] Create authentication middleware
- [ ] Add role-based access control (RBAC)
  - [ ] Admin role
  - [ ] Server role
  - [ ] User role

### 2.2 SSH Key Management Endpoints
- [ ] **POST /api/keys** - Create new SSH key pair
  - [ ] Generate SSH key pair (RSA, Ed25519)
  - [ ] Store public key in database
  - [ ] Optionally store encrypted private key
  - [ ] Associate key with user
- [ ] **GET /api/keys** - List all keys for authenticated user
- [ ] **GET /api/keys/{id}** - Get specific key details
- [ ] **DELETE /api/keys/{id}** - Revoke/delete a key
- [ ] **PUT /api/keys/{id}** - Update key metadata (name, description)

### 2.3 Server Management Endpoints
- [ ] **POST /api/servers** - Register a new server
  - [ ] Generate unique server identifier
  - [ ] Create API token for server
  - [ ] Store server metadata (hostname, IP, description)
- [ ] **GET /api/servers** - List all registered servers
- [ ] **GET /api/servers/{id}** - Get server details
- [ ] **DELETE /api/servers/{id}** - Unregister a server
- [ ] **PUT /api/servers/{id}/rotate-token** - Rotate server API token

### 2.4 Remote Server API Endpoints
These endpoints are used by remote servers to fetch and validate keys:

- [ ] **GET /api/v1/server/keys** - Get all authorized keys for the requesting server
  - [ ] Authenticate using server API token
  - [ ] Return list of public keys in OpenSSH format
  - [ ] Support filtering by user
  - [ ] Include key metadata (fingerprint, type, expiration)
- [ ] **POST /api/v1/server/validate** - Validate a specific key
  - [ ] Accept key fingerprint or public key
  - [ ] Return validation status and associated user info
  - [ ] Check key expiration and revocation status
- [ ] **GET /api/v1/server/authorized_keys/{username}** - Get authorized_keys file format
  - [ ] Return keys in standard authorized_keys format
  - [ ] Include key options if configured (e.g., `from="10.0.0.1"`)
  - [ ] Support caching headers for performance

---

## Phase 3: Key Assignment & Authorization

### 3.1 Key-to-Server Assignment
- [ ] **POST /api/assignments** - Assign key to server(s)
  - [ ] Link user's key to specific servers
  - [ ] Support multiple server assignment
  - [ ] Set permissions (e.g., sudo access level)
- [ ] **GET /api/assignments** - List all assignments
- [ ] **DELETE /api/assignments/{id}** - Remove key assignment
- [ ] Implement assignment rules and policies
  - [ ] Time-based access (temporary keys)
  - [ ] IP restrictions
  - [ ] Command restrictions

### 3.2 Key Validation Service
- [ ] Create SSH key validation service class
  - [ ] Validate key format (OpenSSH, RFC4716, PKCS8)
  - [ ] Parse key type (RSA, DSA, ECDSA, Ed25519)
  - [ ] Generate key fingerprints (MD5, SHA256)
  - [ ] Check key strength requirements
  - [ ] Detect duplicate keys
- [ ] Implement key expiration checking
- [ ] Implement key revocation checking

---

## Phase 4: Security Features

### 4.1 Audit Logging
- [ ] Log all API requests
  - [ ] Timestamp, user/server ID, endpoint, action
  - [ ] IP address and user agent
  - [ ] Request/response payload (sanitized)
- [ ] Log key operations
  - [ ] Key creation, deletion, assignment
  - [ ] Key downloads by servers
  - [ ] Failed authentication attempts
- [ ] Create audit log query endpoints
- [ ] Implement log retention policies

### 4.2 Security Hardening
- [ ] Implement rate limiting
  - [ ] Per API token
  - [ ] Per IP address
  - [ ] Configurable thresholds
- [ ] Add input validation and sanitization
- [ ] Implement HTTPS enforcement
- [ ] Add CORS configuration
- [ ] Implement request signing for critical operations
- [ ] Add webhook support for security events
- [ ] Implement IP whitelist/blacklist for server access

### 4.3 Key Rotation & Expiration
- [ ] Implement automatic key expiration
- [ ] Create key rotation workflow
  - [ ] Notify before expiration
  - [ ] Grace period for old keys
  - [ ] Automated rotation API
- [ ] Add scheduled cleanup of expired keys

---

## Phase 5: Client Tools

### 5.1 Server-Side Agent Script
Create a bash/Python script for remote servers:
- [ ] Installation script for servers
- [ ] Automated key fetching daemon/cron job
- [ ] Update authorized_keys file automatically
- [ ] Support for multiple users
- [ ] Configuration file for API endpoint and token
- [ ] Logging and error handling
- [ ] Systemd service file for daemon mode

### 5.2 CLI Management Tool
- [ ] Create PHP CLI tool for administrators
  - [ ] User management commands
  - [ ] Key management commands
  - [ ] Server management commands
  - [ ] Bulk operations support
  - [ ] Import/export functionality
- [ ] Add interactive mode for common tasks

---

## Phase 6: Web Interface (Optional but Recommended)

### 6.1 Frontend Setup
- [ ] Choose frontend approach (PHP templates, or separate SPA)
- [ ] Set up routing for web pages
- [ ] Create base layout and templates
- [ ] Implement responsive design

### 6.2 Web UI Features
- [ ] Dashboard
  - [ ] Overview of keys, servers, users
  - [ ] Recent activity feed
  - [ ] Security alerts
- [ ] Key Management UI
  - [ ] List, create, edit, delete keys
  - [ ] Upload existing keys
  - [ ] Download public keys
  - [ ] View key fingerprints
- [ ] Server Management UI
  - [ ] Register/unregister servers
  - [ ] View server status
  - [ ] Manage API tokens
- [ ] Assignment Management UI
  - [ ] Assign keys to servers
  - [ ] Visual representation of assignments
  - [ ] Bulk assignment tools
- [ ] User Management UI
  - [ ] Create/edit users
  - [ ] Assign roles
  - [ ] View user activity
- [ ] Audit Log Viewer
  - [ ] Searchable log interface
  - [ ] Filtering and export capabilities

---

## Phase 7: Testing & Quality Assurance

### 7.1 Unit Tests
- [ ] Write tests for all service classes
- [ ] Write tests for authentication middleware
- [ ] Write tests for key validation logic
- [ ] Write tests for database models
- [ ] Aim for >80% code coverage

### 7.2 Integration Tests
- [ ] Test API endpoints end-to-end
- [ ] Test server authentication flow
- [ ] Test key retrieval by remote servers
- [ ] Test key assignment workflow
- [ ] Test audit logging

### 7.3 Security Testing
- [ ] Test authentication bypass attempts
- [ ] Test SQL injection vulnerabilities
- [ ] Test XSS vulnerabilities (if web UI exists)
- [ ] Test rate limiting effectiveness
- [ ] Test key validation edge cases
- [ ] Perform dependency vulnerability scan

---

## Phase 8: Documentation

### 8.1 API Documentation
- [ ] Generate OpenAPI/Swagger specification
- [ ] Document all endpoints with examples
- [ ] Include authentication requirements
- [ ] Provide code examples (curl, PHP, Python)
- [ ] Document error codes and responses

### 8.2 Deployment Documentation
- [ ] Write installation guide
  - [ ] System requirements
  - [ ] PHP version, extensions
  - [ ] Database setup
  - [ ] Web server configuration (Apache/Nginx)
- [ ] Write configuration guide
  - [ ] Environment variables
  - [ ] Security settings
  - [ ] Performance tuning
- [ ] Write upgrade guide
- [ ] Write backup and restore procedures

### 8.3 User Documentation
- [ ] Create user guide for web interface
- [ ] Create guide for server administrators
  - [ ] Installing the agent script
  - [ ] Configuring servers
  - [ ] Troubleshooting
- [ ] Create developer guide for API integration
- [ ] Create FAQ and troubleshooting guide

### 8.4 Code Documentation
- [ ] Add PHPDoc comments to all classes and methods
- [ ] Create architecture overview document
- [ ] Document database schema
- [ ] Create diagrams (architecture, sequence, ER)

---

## Phase 9: DevOps & Deployment

### 9.1 Containerization
- [ ] Create Dockerfile for application
- [ ] Create docker-compose.yml for local development
  - [ ] PHP application container
  - [ ] Database container (MySQL/PostgreSQL)
  - [ ] Web server container (Nginx)
- [ ] Optimize container images for production

### 9.2 CI/CD Pipeline
- [ ] Set up GitHub Actions or similar CI
- [ ] Automate testing on pull requests
- [ ] Automate code style checking
- [ ] Automate security scanning
- [ ] Automate deployment to staging
- [ ] Create deployment automation for production

### 9.3 Monitoring & Alerting
- [ ] Implement health check endpoint
- [ ] Add Prometheus metrics endpoint (optional)
- [ ] Configure logging to external service
- [ ] Set up alerts for critical errors
- [ ] Monitor API performance and availability

---

## Phase 10: Advanced Features (Future Enhancements)

### 10.1 Multi-Factor Authentication
- [ ] Add 2FA for user login
- [ ] Add confirmation for sensitive operations
- [ ] Implement hardware token support (YubiKey)

### 10.2 Key Backup & Recovery
- [ ] Implement encrypted key backup
- [ ] Create key recovery workflow
- [ ] Add key escrow for critical keys

### 10.3 Integration Features
- [ ] LDAP/Active Directory integration
- [ ] SAML/OAuth2 authentication
- [ ] Slack/Discord notifications
- [ ] Webhook integrations
- [ ] REST API for third-party tools

### 10.4 Advanced Management
- [ ] Key approval workflow
- [ ] Automated compliance reporting
- [ ] Key usage analytics
- [ ] Bulk import from existing systems
- [ ] Multi-tenancy support

### 10.5 High Availability
- [ ] Database replication setup
- [ ] Load balancing configuration
- [ ] Caching layer (Redis/Memcached)
- [ ] Distributed session management

---

## Technology Stack Recommendations

### Backend
- **PHP**: 8.1 or higher
- **Framework**: Slim 4 or Laravel 10+
- **Database**: MySQL 8.0+ or PostgreSQL 14+
- **Cache**: Redis (optional, for sessions and rate limiting)

### Libraries
- **phpseclib/phpseclib**: SSH key operations
- **firebase/php-jwt**: JWT authentication
- **monolog/monolog**: Logging
- **vlucas/phpdotenv**: Environment configuration
- **guzzlehttp/guzzle**: HTTP client
- **phpunit/phpunit**: Testing

### Frontend (if web UI)
- **HTML/CSS/JavaScript**: Plain or with framework (Vue.js, React)
- **Bootstrap** or **Tailwind CSS**: UI framework
- **Alpine.js**: Lightweight JavaScript framework (optional)

### DevOps
- **Docker**: Containerization
- **Nginx** or **Apache**: Web server
- **GitHub Actions**: CI/CD
- **Composer**: Dependency management

---

## Security Considerations Checklist

- [ ] Use HTTPS only (TLS 1.2+)
- [ ] Store API tokens hashed (bcrypt/Argon2)
- [ ] Implement rate limiting
- [ ] Validate and sanitize all inputs
- [ ] Use prepared statements for database queries
- [ ] Implement CSRF protection for web UI
- [ ] Use secure random generators for tokens
- [ ] Regular dependency updates
- [ ] Implement proper error handling (no sensitive data in errors)
- [ ] Use security headers (CSP, HSTS, X-Frame-Options)
- [ ] Implement account lockout after failed attempts
- [ ] Regular security audits
- [ ] Encrypt sensitive data at rest
- [ ] Implement key rotation policies

---

## Initial Implementation Priority

**MVP (Minimum Viable Product) - Phases 1-3:**
1. Basic project setup with PHP framework
2. Database schema and migrations
3. API token authentication
4. Key management endpoints (CRUD)
5. Server registration
6. Key retrieval endpoint for remote servers
7. Basic key assignment functionality

**After MVP - Phases 4-5:**
1. Audit logging
2. Security hardening
3. Server-side agent script

**Polish - Phases 6-10:**
1. Web interface
2. Comprehensive testing
3. Documentation
4. Deployment tools
5. Advanced features

---

## Success Metrics

- [ ] Remote servers can fetch authorized keys via API
- [ ] Keys can be managed centrally through API
- [ ] Complete audit trail of all operations
- [ ] API response time <200ms for key retrieval
- [ ] Support for 100+ servers and 1000+ keys
- [ ] Zero security vulnerabilities in production
- [ ] 100% uptime SLA with proper deployment
- [ ] Complete API and user documentation

---

## Notes

- Start with a simple, working MVP and iterate
- Security is paramount - review code regularly
- Keep backward compatibility for API versions
- Use semantic versioning for releases
- Consider cloud deployment (AWS, GCP, Azure) for scalability
- Regular backups of database and configuration
- Plan for disaster recovery from day one
