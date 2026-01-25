# SSH Key Manager API - Implementation To-Do

## Overview
Transform the V PHP Framework into an SSH Key Manager API that allows servers to authenticate users via SSH public keys. The API will integrate with OpenSSH's `AuthorizedKeysCommand` configuration to dynamically provide public keys for user authentication.

---

## Phase 1: Database Schema Design

### 1.1 Create SSH Keys Table
- [ ] Add `ssh_keys` table to `root/install/install.sql`:
  - `id` (INT, AUTO_INCREMENT, PRIMARY KEY)
  - `username` (VARCHAR(255), NOT NULL, INDEX)
  - `key_name` (VARCHAR(255), NOT NULL) - Friendly name for the key
  - `public_key` (TEXT, NOT NULL) - The actual SSH public key
  - `key_type` (VARCHAR(50), NOT NULL) - e.g., 'ssh-rsa', 'ssh-ed25519', 'ecdsa-sha2-nistp256'
  - `fingerprint` (VARCHAR(255), UNIQUE) - MD5 or SHA256 fingerprint for uniqueness
  - `is_active` (BOOLEAN, DEFAULT TRUE) - Enable/disable keys without deletion
  - `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
  - `last_used` (TIMESTAMP, NULL) - Track when key was last used for login
  - `expires_at` (TIMESTAMP, NULL) - Optional expiration date
  - `created_by` (VARCHAR(255), NULL) - Admin who added the key
  - Foreign key constraint: `username` references `users(username)` ON DELETE CASCADE

### 1.2 Create Server Tokens Table
- [ ] Add `server_tokens` table for API authentication:
  - `id` (INT, AUTO_INCREMENT, PRIMARY KEY)
  - `server_name` (VARCHAR(255), NOT NULL) - Friendly name for the server
  - `token` (VARCHAR(255), UNIQUE, NOT NULL) - API authentication token
  - `ip_address` (VARCHAR(45), NULL) - Optional IP restriction
  - `is_active` (BOOLEAN, DEFAULT TRUE)
  - `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
  - `last_used` (TIMESTAMP, NULL)
  - `created_by` (VARCHAR(255), NULL) - Admin who created the token

### 1.3 Create API Access Logs Table
- [ ] Add `api_access_logs` table for audit trail:
  - `id` (INT, AUTO_INCREMENT, PRIMARY KEY)
  - `server_token_id` (INT, NULL) - References server_tokens(id)
  - `username` (VARCHAR(255), NULL) - Username queried
  - `ip_address` (VARCHAR(45), NOT NULL)
  - `endpoint` (VARCHAR(255), NOT NULL)
  - `success` (BOOLEAN, NOT NULL)
  - `timestamp` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
  - INDEX on `timestamp` for efficient log queries

---

## Phase 2: Model Layer

### 2.1 Create SSHKey Model
- [ ] Create `root/app/Models/SSHKey.php` with methods:
  - `getActiveKeysByUsername(string $username): array` - Get all active keys for a user
  - `addKey(string $username, string $keyName, string $publicKey): bool` - Add new SSH key
  - `validatePublicKey(string $publicKey): array` - Validate key format and extract metadata
  - `generateFingerprint(string $publicKey): string` - Generate unique fingerprint
  - `deleteKey(int $keyId): bool` - Delete a key
  - `toggleKeyStatus(int $keyId, bool $isActive): bool` - Enable/disable a key
  - `updateLastUsed(int $keyId): bool` - Update last_used timestamp
  - `getKeysByUser(string $username): array` - Get all keys for a user (admin view)
  - `checkExpiration(): int` - Clean up expired keys (for cron)

### 2.2 Create ServerToken Model
- [ ] Create `root/app/Models/ServerToken.php` with methods:
  - `validateToken(string $token, string $ipAddress = null): bool` - Validate API token
  - `getServerByToken(string $token): ?object` - Get server info by token
  - `createToken(string $serverName, string $createdBy, string $ipAddress = null): string` - Generate new token
  - `revokeToken(int $tokenId): bool` - Revoke server access
  - `listActiveTokens(): array` - List all active server tokens
  - `updateLastUsed(int $tokenId): bool` - Track token usage

### 2.3 Create APILog Model
- [ ] Create `root/app/Models/APILog.php` with methods:
  - `logAccess(int $serverTokenId, string $username, string $ipAddress, string $endpoint, bool $success): bool`
  - `getRecentLogs(int $limit = 100): array` - Get recent API access logs
  - `getLogsByServer(int $serverTokenId, int $limit = 100): array`
  - `getLogsByUsername(string $username, int $limit = 100): array`
  - `cleanOldLogs(int $daysToKeep = 90): int` - Cron job to clean old logs

### 2.4 Enhance Users Model
- [ ] Update `root/app/Models/Users.php`:
  - Implement actual `getUserInfo()` method to fetch user data from database
  - Add `getUserWithKeys(string $username): ?object` - Get user with associated SSH keys
  - Add `isUserActive(string $username): bool` - Check if user account is active

---

## Phase 3: API Controllers

### 3.1 Create API Keys Controller
- [ ] Create `root/app/Controllers/API/KeysController.php`:
  - `handleGetKeys()` - GET endpoint for AuthorizedKeysCommand
    - Accept `?username=<username>` parameter
    - Validate server token in Authorization header or query param
    - Return public keys in OpenSSH authorized_keys format (one per line)
    - Log access attempt
    - Return HTTP 200 with keys, 404 if user not found, 401 if token invalid
  - `handleGetKeysJson()` - GET endpoint for JSON format
    - Return keys in JSON format for API clients
    - Include metadata (key_name, fingerprint, created_at, etc.)
  - Add rate limiting to prevent abuse

### 3.2 Create Admin SSH Keys Controller
- [ ] Create `root/app/Controllers/SSHKeysController.php` for web UI:
  - `handleRequest()` - GET: Display user's SSH keys management page
  - `handleSubmission()` - POST: Process key additions/deletions
  - `handleAddKey()` - Add new SSH key form submission
  - `handleDeleteKey()` - Delete key action
  - `handleToggleKey()` - Enable/disable key action
  - Implement CSRF protection using existing `Csrf` class
  - Require authentication via SessionManager

### 3.3 Create Admin Server Tokens Controller
- [ ] Create `root/app/Controllers/ServerTokensController.php`:
  - `handleRequest()` - GET: Display server tokens management page
  - `handleSubmission()` - POST: Create new server token
  - `handleRevoke()` - Revoke a server token
  - `handleViewLogs()` - View access logs for a specific token
  - Admin-only access control

### 3.4 Create API Logs Controller
- [ ] Create `root/app/Controllers/APILogsController.php`:
  - `handleRequest()` - GET: Display API access logs
  - Add filtering by date, server, username
  - Pagination for large log sets
  - Admin-only access

---

## Phase 4: Routing Configuration

### 4.1 Add API Routes
- [ ] Update `root/app/Core/Router.php`:
  - Add route: `GET /api/v1/keys` → `API\KeysController::handleGetKeys`
  - Add route: `GET /api/v1/keys.json` → `API\KeysController::handleGetKeysJson`
  - Bypass session authentication for API routes (use token auth instead)
  - Add route: `POST /api/v1/validate` → Validate token endpoint (optional)

### 4.2 Add Web UI Routes
- [ ] Add management interface routes:
  - `GET /ssh-keys` → `SSHKeysController::handleRequest`
  - `POST /ssh-keys` → `SSHKeysController::handleSubmission`
  - `POST /ssh-keys/add` → `SSHKeysController::handleAddKey`
  - `POST /ssh-keys/delete` → `SSHKeysController::handleDeleteKey`
  - `POST /ssh-keys/toggle` → `SSHKeysController::handleToggleKey`
  - `GET /server-tokens` → `ServerTokensController::handleRequest`
  - `POST /server-tokens` → `ServerTokensController::handleSubmission`
  - `POST /server-tokens/revoke` → `ServerTokensController::handleRevoke`
  - `GET /api-logs` → `APILogsController::handleRequest`

### 4.3 Update Router Logic
- [ ] Modify `Router::dispatch()` method:
  - Detect API routes (starting with `/api/`)
  - Skip session authentication for API routes
  - Implement token-based authentication for API routes
  - Keep session authentication for web UI routes

---

## Phase 5: View Templates (Web UI)

### 5.1 Create SSH Keys Management View
- [ ] Create `root/app/Views/ssh-keys.php`:
  - Display list of user's SSH keys in a table
  - Show key name, fingerprint, type, created date, last used, status
  - Add form to upload/paste new SSH key
  - Add delete and toggle active/inactive buttons
  - Use SPECTRE.CSS for styling (required by AGENTS.md)
  - Include CSRF token in all forms

### 5.2 Create Server Tokens Management View
- [ ] Create `root/app/Views/server-tokens.php`:
  - Display list of server tokens (admin only)
  - Show server name, token (partially masked), IP restriction, created date, last used
  - Form to create new token
  - Revoke token action
  - Link to view access logs for each token
  - Use SPECTRE.CSS for styling

### 5.3 Create API Logs View
- [ ] Create `root/app/Views/api-logs.php`:
  - Display table of API access logs
  - Show timestamp, server name, username, endpoint, success status, IP
  - Add filters for date range, server, username
  - Implement pagination
  - Use SPECTRE.CSS for styling

### 5.4 Update Navigation
- [ ] Update `root/app/Views/partials/header.php`:
  - Add navigation links to SSH Keys, Server Tokens (admin), API Logs (admin)
  - Conditional display based on admin status

---

## Phase 6: API Authentication & Security

### 6.1 Implement Token-Based Authentication
- [ ] Create `root/app/Core/APIAuth.php`:
  - `validateRequest(): bool` - Validate incoming API requests
  - Support Bearer token in Authorization header: `Authorization: Bearer <token>`
  - Support token in query parameter: `?token=<token>` (fallback)
  - Verify token against `server_tokens` table
  - Optional IP address validation
  - Rate limiting per token

### 6.2 Rate Limiting
- [ ] Create `root/app/Core/RateLimiter.php`:
  - Implement token bucket or sliding window algorithm
  - Store rate limit data in database or memory (session/cache)
  - Default: 100 requests per minute per token
  - Return HTTP 429 (Too Many Requests) when limit exceeded
  - Configurable limits in `config.php`

### 6.3 Input Validation & Sanitization
- [ ] Validate all API inputs:
  - Username format validation (alphanumeric, dots, dashes, underscores)
  - SSH public key format validation
  - Token format validation
  - Prevent SQL injection (use prepared statements)
  - Prevent XSS in web UI (use htmlspecialchars)

### 6.4 Logging & Monitoring
- [ ] Implement comprehensive logging:
  - Log all API requests to `api_access_logs` table
  - Log authentication failures
  - Log key additions/deletions/modifications
  - Use existing `ErrorManager` for error logging

---

## Phase 7: Configuration

### 7.1 Update Config File
- [ ] Add to `root/config.php`:
  - `API_RATE_LIMIT_ENABLED` (bool) - Enable/disable rate limiting
  - `API_RATE_LIMIT_REQUESTS` (int) - Requests per minute
  - `API_LOG_RETENTION_DAYS` (int) - Days to keep API logs
  - `SSH_KEY_MAX_PER_USER` (int) - Maximum keys per user
  - `SSH_KEY_EXPIRATION_ENABLED` (bool) - Enable key expiration
  - `SSH_KEY_DEFAULT_EXPIRATION_DAYS` (int) - Default expiration period
  - `REQUIRE_KEY_APPROVAL` (bool) - Admin must approve new keys

### 7.2 Environment Variables
- [ ] Document environment variables in README:
  - Database connection details
  - SMTP settings (for notifications)
  - API configuration options

---

## Phase 8: Client Scripts

### 8.1 Create AuthorizedKeysCommand Script
- [ ] Create `scripts/get-ssh-keys.sh`:
  - Bash script to be used by OpenSSH
  - Accept username as first argument: `$1`
  - Make HTTPS request to API endpoint
  - Include server token in Authorization header
  - Handle SSL certificate verification
  - Output keys in OpenSSH format (one per line)
  - Handle errors gracefully (return empty on error)
  - Add logging to syslog for troubleshooting

### 8.2 Create Installation Script
- [ ] Create `scripts/install-client.sh`:
  - Install dependencies (curl or wget)
  - Copy `get-ssh-keys.sh` to `/usr/local/bin/`
  - Set proper permissions (755)
  - Configure sshd_config:
    - Set `AuthorizedKeysCommand /usr/local/bin/get-ssh-keys.sh`
    - Set `AuthorizedKeysCommandUser nobody`
  - Restart SSH service
  - Test script execution

### 8.3 Create Token Configuration Helper
- [ ] Create `scripts/configure-server.sh`:
  - Interactive script to configure server token
  - Store token securely in `/etc/ssh/api-token`
  - Set proper file permissions (600)
  - Test API connectivity

---

## Phase 9: Scheduled Tasks (Cron Jobs)

### 9.1 Update Cron Script
- [ ] Update `root/cron.php`:
  - Add `daily` task: Clean expired SSH keys
  - Add `daily` task: Clean old API access logs
  - Add `hourly` task: Send expiration warnings (optional)
  - Add `hourly` task: Update key usage statistics

### 9.2 Key Expiration Job
- [ ] Implement expiration logic:
  - Query keys with `expires_at < NOW()`
  - Disable or delete expired keys
  - Send email notifications to users (using existing Mailer)
  - Log expiration actions

### 9.3 Log Cleanup Job
- [ ] Implement log cleanup:
  - Delete logs older than `API_LOG_RETENTION_DAYS`
  - Keep summary statistics
  - Log cleanup action

---

## Phase 10: Documentation

### 10.1 Update README.md
- [ ] Add sections:
  - SSH Key Manager API overview
  - Installation instructions
  - Database setup
  - API endpoint documentation
  - Web UI usage guide
  - Server integration guide
  - Security best practices

### 10.2 Create API Documentation
- [ ] Create `docs/API.md`:
  - Endpoint reference
  - Authentication methods
  - Request/response examples
  - Error codes and messages
  - Rate limiting details

### 10.3 Create Server Integration Guide
- [ ] Create `docs/SERVER-SETUP.md`:
  - Prerequisites
  - Installing client script
  - Configuring OpenSSH
  - Testing setup
  - Troubleshooting common issues
  - Example sshd_config snippet:
    ```
    AuthorizedKeysCommand /usr/local/bin/get-ssh-keys.sh
    AuthorizedKeysCommandUser nobody
    ```

### 10.4 Create Admin Guide
- [ ] Create `docs/ADMIN-GUIDE.md`:
  - User management
  - SSH key management
  - Server token management
  - Viewing audit logs
  - Security recommendations
  - Backup and recovery

### 10.5 Update CHANGELOG.md
- [ ] Document all changes:
  - New features
  - Database schema changes
  - API endpoints
  - Breaking changes
  - Migration guide

---

## Phase 11: Testing

### 11.1 Unit Tests
- [ ] Create `unit/SSHKeyModelTest.php`:
  - Test key validation
  - Test fingerprint generation
  - Test CRUD operations
  - Test expiration logic

- [ ] Create `unit/ServerTokenModelTest.php`:
  - Test token validation
  - Test token generation
  - Test IP restriction logic

- [ ] Create `unit/APIAuthTest.php`:
  - Test token authentication
  - Test rate limiting
  - Test authorization failures

### 11.2 Integration Tests
- [ ] Create `unit/APIEndpointTest.php`:
  - Test `/api/v1/keys` endpoint
  - Test with valid token
  - Test with invalid token
  - Test with missing user
  - Test response format

### 11.3 Manual Testing
- [ ] Test web UI:
  - User login
  - Add SSH key
  - Delete SSH key
  - Toggle key status
  - Admin token management
  - View logs

- [ ] Test API:
  - Use curl to test endpoints
  - Verify output format matches OpenSSH
  - Test rate limiting
  - Test error responses

- [ ] Test SSH integration:
  - Configure test server
  - Test SSH login with managed keys
  - Verify logging
  - Test key revocation

---

## Phase 12: Security Hardening

### 12.1 Code Review
- [ ] Review all code for security issues:
  - SQL injection vulnerabilities
  - XSS vulnerabilities
  - CSRF protection
  - Authentication bypass
  - Information disclosure

### 12.2 Input Validation
- [ ] Validate all inputs:
  - SSH key format validation
  - Username sanitization
  - Token format validation
  - File upload validation (if applicable)

### 12.3 HTTPS Enforcement
- [ ] Add to `root/public/index.php`:
  - Redirect HTTP to HTTPS
  - Set security headers:
    - `Strict-Transport-Security`
    - `X-Content-Type-Options`
    - `X-Frame-Options`
    - `X-XSS-Protection`

### 12.4 Token Security
- [ ] Implement secure token generation:
  - Use cryptographically secure random generation
  - Minimum 32 characters
  - Store hashed in database (optional, but recommended)
  - Implement token rotation

### 12.5 Audit Trail
- [ ] Ensure all actions are logged:
  - User login/logout
  - Key additions/deletions
  - Token creations/revocations
  - Configuration changes
  - Failed authentication attempts

---

## Phase 13: Deployment

### 13.1 Production Checklist
- [ ] Set proper file permissions:
  - Application files: 644
  - Directories: 755
  - Config file: 640 (readable by web server only)
  - Logs directory: writable by web server

- [ ] Configure web server:
  - Apache/Nginx virtual host configuration
  - SSL/TLS certificate installation
  - Redirect to HTTPS
  - Set proper document root to `root/public`

- [ ] Database security:
  - Create dedicated database user with minimal privileges
  - Use strong passwords
  - Enable SSL for database connections (if remote)

- [ ] Backup strategy:
  - Regular database backups
  - Backup server tokens securely
  - Document restore procedure

### 13.2 Monitoring
- [ ] Set up monitoring:
  - API endpoint availability
  - Response time monitoring
  - Error rate monitoring
  - Disk space for logs
  - Alert on authentication failures spike

### 13.3 Documentation
- [ ] Create deployment guide
- [ ] Document server requirements
- [ ] Create rollback procedure
- [ ] Document troubleshooting steps

---

## Phase 14: Maintenance & Future Enhancements

### 14.1 Monitoring & Alerts
- [ ] Implement alerts for:
  - High API error rate
  - Rate limit violations
  - Expired keys
  - Low server token usage (unused tokens)

### 14.2 Future Enhancements
- [ ] Multi-factor authentication for web UI
- [ ] Key rotation reminders
- [ ] Bulk key import from file
- [ ] Key usage analytics dashboard
- [ ] Integration with LDAP/Active Directory
- [ ] Support for SSH certificate authorities
- [ ] API versioning for backward compatibility
- [ ] Webhook notifications for key events
- [ ] Export audit logs in various formats
- [ ] Support for key comments/descriptions

---

## Implementation Priority

### High Priority (MVP)
1. Database schema (Phase 1)
2. Core models (Phase 2)
3. API controller with basic authentication (Phase 3.1, 6.1)
4. API routes (Phase 4.1)
5. Client script (Phase 8.1)
6. Basic documentation (Phase 10.1, 10.3)

### Medium Priority
7. Web UI for key management (Phase 3.2, 5.1)
8. Server token management (Phase 3.3, 5.2)
9. Rate limiting (Phase 6.2)
10. Cron jobs (Phase 9)
11. Testing (Phase 11)

### Lower Priority
12. API logs UI (Phase 3.4, 5.3)
13. Advanced security features (Phase 12)
14. Complete documentation (Phase 10.2, 10.4)
15. Future enhancements (Phase 14.2)

---

## Notes

- **Follow AGENTS.md guidelines**: Use SPECTRE.CSS for all UI, maintain existing architecture
- **Minimal changes**: Build on existing framework without major refactoring
- **Security first**: Validate all inputs, use prepared statements, implement proper authentication
- **Backward compatibility**: Ensure existing framework features remain functional
- **Documentation**: Keep README.md and other docs up to date
- **Testing**: Write tests before implementation where possible (TDD)

---

## OpenSSH Integration Example

### sshd_config Configuration
```bash
# Add to /etc/ssh/sshd_config
AuthorizedKeysCommand /usr/local/bin/get-ssh-keys.sh
AuthorizedKeysCommandUser nobody
```

### Example API Request
```bash
curl -H "Authorization: Bearer YOUR_SERVER_TOKEN" \
     "https://keys.example.com/api/v1/keys?username=john"
```

### Expected Response Format
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB... john@laptop
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... john@desktop
```

---

## Success Criteria

- [ ] Servers can successfully authenticate SSH users via API
- [ ] Web UI allows users to manage their SSH keys
- [ ] Admins can manage server tokens and view logs
- [ ] All tests pass
- [ ] All code passes linting (phpcs)
- [ ] Documentation is complete and accurate
- [ ] Security audit completed with no critical issues
- [ ] Production deployment successful
