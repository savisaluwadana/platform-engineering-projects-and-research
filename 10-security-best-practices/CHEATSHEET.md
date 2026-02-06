# Security Best Practices Cheatsheet

## Quick Reference Guide for Platform Engineers

### Security Scanning Tools

```bash
# SAST (Static Application Security Testing)
# SonarQube
sonar-scanner \
  -Dsonar.projectKey=myproject \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=TOKEN

# Semgrep
semgrep --config=auto .
semgrep --config=p/owasp-top-ten .
semgrep --config=p/security-audit .
semgrep --json --output=results.json .

# Bandit (Python)
bandit -r ./src                           # Scan recursively
bandit -r ./src -f json -o report.json    # JSON output
bandit -r ./src -ll                       # Low severity and above

# ESLint with security plugin (JavaScript)
npm install --save-dev eslint-plugin-security
eslint --plugin security --rule security/detect-object-injection:error .

# Brakeman (Ruby on Rails)
brakeman -A -q                            # All checks, quiet
brakeman -o report.html                   # HTML report

# DAST (Dynamic Application Security Testing)
# OWASP ZAP
zap-cli quick-scan --self-contained http://localhost:3000
zap-cli active-scan http://localhost:3000
zap-cli report -o zap-report.html -f html

# Dependency Scanning
# npm audit (Node.js)
npm audit                                 # Check vulnerabilities
npm audit fix                             # Auto-fix vulnerabilities
npm audit --json                          # JSON output
npm audit --audit-level=moderate          # Severity threshold

# Snyk
snyk test                                 # Test for vulnerabilities
snyk monitor                              # Monitor project
snyk test --severity-threshold=high       # Fail on high severity
snyk container test nginx:latest          # Scan container image

# OWASP Dependency-Check
dependency-check --project myapp --scan ./src

# Safety (Python)
safety check                              # Check requirements.txt
safety check --json                       # JSON output
safety check --file requirements.txt

# Container Scanning
# Trivy
trivy image nginx:latest                  # Scan image
trivy image --severity HIGH,CRITICAL nginx:latest
trivy image --format json -o results.json nginx:latest
trivy fs --security-checks vuln,config .  # Scan filesystem
trivy repo https://github.com/user/repo   # Scan repository

# Grype
grype nginx:latest                        # Scan image
grype dir:.                               # Scan directory
grype sbom:./sbom.json                    # Scan SBOM

# Clair
clairctl analyze nginx:latest
clairctl report nginx:latest

# Anchore
anchore-cli image add nginx:latest
anchore-cli image wait nginx:latest
anchore-cli image vuln nginx:latest all

# Infrastructure Scanning
# Checkov (IaC Security)
checkov -d .                              # Scan directory
checkov -f main.tf                        # Scan file
checkov --framework terraform             # Terraform only
checkov --framework kubernetes            # Kubernetes only
checkov --output json                     # JSON output

# tfsec (Terraform)
tfsec .                                   # Scan current directory
tfsec --format json .                     # JSON output
tfsec --minimum-severity HIGH .           # High severity only

# Kube-bench (Kubernetes CIS Benchmark)
kube-bench run --targets master,node      # Run all checks
kube-bench run --check 1.2.3              # Specific check
kube-bench run --json                     # JSON output

# Kube-hunter (Kubernetes penetration testing)
kube-hunter --remote some.node.com        # Remote scan
kube-hunter --cidr 192.168.0.0/24        # Network scan

# Secret Scanning
# TruffleHog
trufflehog git https://github.com/user/repo --only-verified
trufflehog filesystem ./src               # Scan filesystem
trufflehog git file://. --since-commit HEAD~100

# GitGuardian
ggshield secret scan path ./src           # Scan directory
ggshield secret scan ci                   # CI/CD integration
ggshield secret scan docker nginx:latest  # Docker image

# Gitleaks
gitleaks detect --source . --verbose      # Scan repository
gitleaks protect --staged                 # Pre-commit hook
gitleaks detect --report-format json --report-path report.json

# detect-secrets
detect-secrets scan                       # Generate baseline
detect-secrets audit .secrets.baseline    # Audit findings
```

### Secrets Management

```bash
# HashiCorp Vault
# Start Vault dev server
vault server -dev

# Set Vault address
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='root'

# Write secrets
vault kv put secret/myapp/config api_key="12345" db_password="secret"
vault kv put secret/database/config username="admin" password="password"

# Read secrets
vault kv get secret/myapp/config
vault kv get -field=api_key secret/myapp/config
vault kv get -format=json secret/myapp/config

# List secrets
vault kv list secret/

# Delete secrets
vault kv delete secret/myapp/config

# Enable secrets engine
vault secrets enable -path=secret kv-v2
vault secrets enable database
vault secrets enable aws

# Policies
vault policy write myapp-policy - <<EOF
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}
EOF

vault policy list
vault policy read myapp-policy

# Authentication
vault login
vault login -method=userpass username=myuser password=mypass
vault auth enable userpass
vault write auth/userpass/users/myuser password=mypass policies=myapp-policy

# Tokens
vault token create -policy=myapp-policy
vault token lookup
vault token revoke TOKEN

# AWS Secrets Manager
# Create secret
aws secretsmanager create-secret \
  --name myapp/database \
  --secret-string '{"username":"admin","password":"secret"}'

# Get secret
aws secretsmanager get-secret-value --secret-id myapp/database
aws secretsmanager get-secret-value --secret-id myapp/database --query SecretString --output text

# Update secret
aws secretsmanager update-secret \
  --secret-id myapp/database \
  --secret-string '{"username":"admin","password":"newsecret"}'

# Rotate secret
aws secretsmanager rotate-secret \
  --secret-id myapp/database \
  --rotation-lambda-arn arn:aws:lambda:region:account:function:MyRotationFunction

# List secrets
aws secretsmanager list-secrets

# Delete secret
aws secretsmanager delete-secret --secret-id myapp/database --force-delete-without-recovery

# Azure Key Vault
# Create key vault
az keyvault create --name myKeyVault --resource-group myRG --location eastus

# Set secret
az keyvault secret set --vault-name myKeyVault --name mySecret --value "secretValue"

# Get secret
az keyvault secret show --vault-name myKeyVault --name mySecret
az keyvault secret show --vault-name myKeyVault --name mySecret --query value -o tsv

# List secrets
az keyvault secret list --vault-name myKeyVault

# Delete secret
az keyvault secret delete --vault-name myKeyVault --name mySecret

# GCP Secret Manager
# Create secret
echo -n "my-secret-value" | gcloud secrets create my-secret --data-file=-

# Access secret
gcloud secrets versions access latest --secret="my-secret"

# List secrets
gcloud secrets list

# Delete secret
gcloud secrets delete my-secret

# Kubernetes Secrets
# Create secret from literal
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret

# Create secret from file
kubectl create secret generic my-secret --from-file=./secret.txt

# Create TLS secret
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key

# Get secret
kubectl get secret my-secret -o yaml
kubectl get secret my-secret -o jsonpath='{.data.username}' | base64 -d

# Use with External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: my-k8s-secret
  data:
    - secretKey: password
      remoteRef:
        key: secret/data/myapp/config
        property: db_password
```

### SSL/TLS Best Practices

```bash
# Generate SSL Certificate with OpenSSL
# Generate private key
openssl genrsa -out private.key 2048

# Generate CSR (Certificate Signing Request)
openssl req -new -key private.key -out request.csr

# Generate self-signed certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# View certificate
openssl x509 -in cert.pem -text -noout
openssl x509 -in cert.pem -noout -dates
openssl x509 -in cert.pem -noout -subject

# Test SSL/TLS connection
openssl s_client -connect example.com:443
openssl s_client -connect example.com:443 -servername example.com  # SNI

# Check certificate expiration
openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates

# Let's Encrypt with Certbot
# Install certificate
certbot certonly --standalone -d example.com -d www.example.com
certbot certonly --webroot -w /var/www/html -d example.com
certbot certonly --dns-cloudflare -d example.com

# List certificates
certbot certificates

# Renew certificates
certbot renew
certbot renew --dry-run  # Test renewal

# Revoke certificate
certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem

# Auto-renewal with cron
0 0,12 * * * certbot renew --quiet

# Nginx SSL Configuration
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    # Modern configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/nginx/ssl/chain.pem;
    
    # Session cache
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
}

# Test SSL configuration
# SSL Labs
curl -s "https://api.ssllabs.com/api/v3/analyze?host=example.com" | jq

# testssl.sh
./testssl.sh https://example.com

# nmap
nmap --script ssl-enum-ciphers -p 443 example.com
```

### Security Hardening Checklist

```yaml
# System Hardening
- [ ] Keep system updated
  - apt update && apt upgrade (Debian/Ubuntu)
  - yum update (CentOS/RHEL)
  
- [ ] Disable root login
  - PermitRootLogin no in /etc/ssh/sshd_config
  
- [ ] Use SSH keys instead of passwords
  - PasswordAuthentication no in /etc/ssh/sshd_config
  
- [ ] Configure firewall
  - ufw enable (Ubuntu)
  - firewall-cmd --permanent --add-service=ssh (CentOS)
  
- [ ] Install fail2ban
  - apt install fail2ban
  - Configure /etc/fail2ban/jail.local
  
- [ ] Disable unnecessary services
  - systemctl disable <service>
  
- [ ] Set proper file permissions
  - chmod 600 ~/.ssh/authorized_keys
  - chmod 700 ~/.ssh
  
- [ ] Enable audit logging
  - auditd service
  - Configure /etc/audit/auditd.conf
  
- [ ] Harden kernel parameters
  - Edit /etc/sysctl.conf:
    - net.ipv4.conf.all.rp_filter = 1
    - net.ipv4.conf.default.rp_filter = 1
    - net.ipv4.icmp_echo_ignore_broadcasts = 1
    - net.ipv4.conf.all.accept_source_route = 0
    - net.ipv6.conf.all.accept_source_route = 0
    - net.ipv4.conf.all.log_martians = 1

# Application Hardening
- [ ] Use HTTPS everywhere
- [ ] Implement authentication and authorization
- [ ] Validate and sanitize all inputs
- [ ] Use parameterized queries (prevent SQL injection)
- [ ] Implement rate limiting
- [ ] Use security headers:
  - Content-Security-Policy
  - X-Frame-Options
  - X-Content-Type-Options
  - Strict-Transport-Security
  
- [ ] Implement CORS properly
- [ ] Use secure session management
- [ ] Encrypt sensitive data at rest
- [ ] Encrypt data in transit (TLS)
- [ ] Implement proper error handling (don't leak info)
- [ ] Regular dependency updates
- [ ] Code reviews and security testing
- [ ] Implement logging and monitoring

# Docker Hardening
- [ ] Use official base images
- [ ] Scan images for vulnerabilities
- [ ] Use minimal base images (alpine, distroless)
- [ ] Don't run as root
  - USER non-root-user
  
- [ ] Use read-only filesystem
  - docker run --read-only
  
- [ ] Limit resources
  - docker run --memory="256m" --cpus="1.0"
  
- [ ] Use secrets management
  - docker secret create
  
- [ ] Enable Content Trust
  - export DOCKER_CONTENT_TRUST=1
  
- [ ] Use .dockerignore
- [ ] Multi-stage builds
- [ ] Don't store secrets in images
- [ ] Regular image updates

# Kubernetes Hardening
- [ ] Enable RBAC
- [ ] Use Network Policies
- [ ] Enable Pod Security Standards/Admission
- [ ] Scan images before deployment
- [ ] Use secrets for sensitive data
- [ ] Limit resource usage (requests/limits)
- [ ] Run containers as non-root
- [ ] Use read-only root filesystem
- [ ] Disable automounting service account tokens
- [ ] Enable audit logging
- [ ] Keep cluster updated
- [ ] Use private container registry
- [ ] Implement network segmentation
- [ ] Regular security audits
```

### Security Headers

```nginx
# Nginx Security Headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

# Remove server header
server_tokens off;
more_clear_headers Server;
```

```apache
# Apache Security Headers (.htaccess or VirtualHost)
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-Content-Type-Options "nosniff"
Header always set X-XSS-Protection "1; mode=block"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Content-Security-Policy "default-src 'self'"
Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"
Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"

# Remove server header
ServerTokens Prod
ServerSignature Off
```

### Compliance Basics

```bash
# GDPR (General Data Protection Regulation)
- Data minimization
- Right to erasure
- Data portability
- Privacy by design
- Consent management
- Data breach notification (72 hours)
- Data Protection Impact Assessment (DPIA)

# HIPAA (Health Insurance Portability and Accountability Act)
- Access controls
- Audit controls
- Encryption
- Integrity controls
- Transmission security

# PCI DSS (Payment Card Industry Data Security Standard)
- Install and maintain firewall
- Don't use default passwords
- Protect stored cardholder data
- Encrypt transmission of cardholder data
- Use and update antivirus software
- Develop and maintain secure systems
- Restrict access to cardholder data
- Assign unique ID to each person with access
- Restrict physical access to cardholder data
- Track and monitor network access
- Regularly test security systems
- Maintain information security policy

# SOC 2 (Service Organization Control 2)
- Security
- Availability
- Processing integrity
- Confidentiality
- Privacy

# ISO 27001
- Information security management system (ISMS)
- Risk assessment
- Security controls
- Continuous improvement

# CIS Benchmarks
# Download and apply CIS benchmarks
# https://www.cisecurity.org/cis-benchmarks/
```

### Common Vulnerabilities and Fixes

```yaml
# OWASP Top 10 (2021)
1. Broken Access Control
   Fix:
   - Implement proper authorization checks
   - Use least privilege principle
   - Deny by default
   
   # Example (Node.js)
   if (req.user.id !== resource.ownerId) {
     return res.status(403).json({ error: 'Forbidden' });
   }

2. Cryptographic Failures
   Fix:
   - Use strong encryption algorithms
   - Encrypt data at rest and in transit
   - Proper key management
   
   # Example (Python)
   from cryptography.fernet import Fernet
   key = Fernet.generate_key()
   cipher = Fernet(key)
   encrypted = cipher.encrypt(b"Secret data")

3. Injection
   Fix:
   - Use parameterized queries
   - Input validation
   - Use ORM frameworks
   
   # Bad (SQL Injection)
   query = f"SELECT * FROM users WHERE id = {user_id}"
   
   # Good (Parameterized Query)
   query = "SELECT * FROM users WHERE id = ?"
   cursor.execute(query, (user_id,))

4. Insecure Design
   Fix:
   - Threat modeling
   - Secure design patterns
   - Security requirements in SDLC

5. Security Misconfiguration
   Fix:
   - Remove default accounts
   - Disable unnecessary features
   - Keep software updated
   - Security hardening guides
   
   # Example: Disable directory listing (Nginx)
   autoindex off;

6. Vulnerable and Outdated Components
   Fix:
   - Regular dependency updates
   - Automated vulnerability scanning
   - Remove unused dependencies
   
   # Check and update
   npm audit
   npm audit fix
   pip list --outdated

7. Identification and Authentication Failures
   Fix:
   - Multi-factor authentication
   - Strong password policies
   - Session management
   - Rate limiting
   
   # Example: Rate limiting (Express.js)
   const rateLimit = require('express-rate-limit');
   const limiter = rateLimit({
     windowMs: 15 * 60 * 1000, // 15 minutes
     max: 100 // limit each IP to 100 requests per windowMs
   });
   app.use(limiter);

8. Software and Data Integrity Failures
   Fix:
   - Verify software signatures
   - Use SRI (Subresource Integrity)
   - Secure CI/CD pipeline
   
   # Example: SRI
   <script src="https://cdn.example.com/script.js"
           integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
           crossorigin="anonymous"></script>

9. Security Logging and Monitoring Failures
   Fix:
   - Log security events
   - Monitor logs
   - Alert on suspicious activity
   - Centralized logging
   
   # Example: Security logging
   logger.warn('Failed login attempt', {
     username: username,
     ip: req.ip,
     timestamp: new Date()
   });

10. Server-Side Request Forgery (SSRF)
    Fix:
    - Validate and sanitize URLs
    - Use allowlist for permitted domains
    - Network segmentation
    
    # Example: URL validation
    from urllib.parse import urlparse
    
    def is_safe_url(url):
        parsed = urlparse(url)
        return parsed.hostname in ALLOWED_HOSTS

# Additional Common Vulnerabilities

# XSS (Cross-Site Scripting)
# Bad
document.innerHTML = userInput;

# Good
document.textContent = userInput;
# Or use DOMPurify
const clean = DOMPurify.sanitize(userInput);

# CSRF (Cross-Site Request Forgery)
# Use CSRF tokens
<input type="hidden" name="csrf_token" value="{{ csrf_token }}">

# Command Injection
# Bad
os.system(f"ping {user_input}")

# Good
import subprocess
subprocess.run(["ping", "-c", "1", user_input], check=True)

# Path Traversal
# Bad
file_path = base_path + user_input

# Good
import os
file_path = os.path.normpath(os.path.join(base_path, user_input))
if not file_path.startswith(base_path):
    raise ValueError("Invalid path")

# XXE (XML External Entity)
# Disable external entities
import xml.etree.ElementTree as ET
parser = ET.XMLParser()
parser.entity = {}  # Disable entities
tree = ET.parse(xml_file, parser=parser)
```

### Security Testing

```bash
# Penetration Testing Tools
# Nmap (Network scanning)
nmap -sV -sC target.com                   # Version detection + scripts
nmap -p- target.com                       # All ports
nmap -sU target.com                       # UDP scan
nmap --script vuln target.com             # Vulnerability scan

# Nikto (Web server scanner)
nikto -h http://target.com                # Basic scan
nikto -h http://target.com -Tuning 123    # Specific tests

# Burp Suite (Web application testing)
# Commercial tool - use GUI

# SQLMap (SQL injection testing)
sqlmap -u "http://target.com/page?id=1" --dbs
sqlmap -u "http://target.com/page?id=1" -D database --tables
sqlmap -u "http://target.com/page?id=1" -D database -T users --dump

# Metasploit (Exploitation framework)
msfconsole
search exploit windows
use exploit/windows/smb/ms17_010_eternalblue
set RHOST target.com
exploit

# John the Ripper (Password cracking)
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt

# Hydra (Brute force)
hydra -l admin -P passwords.txt ssh://target.com
hydra -L users.txt -P passwords.txt http-post-form "http://target.com/login:username=^USER^&password=^PASS^:F=incorrect"
```

### Incident Response

```yaml
# Incident Response Steps
1. Preparation
   - Incident response plan
   - Tools and resources ready
   - Team training
   - Contact lists updated

2. Identification
   - Monitor alerts
   - Verify incident
   - Determine scope
   - Classify severity

3. Containment
   - Short-term containment (isolate affected systems)
   - Long-term containment (temporary fixes)
   - Preserve evidence

4. Eradication
   - Remove threat
   - Identify root cause
   - Improve security posture

5. Recovery
   - Restore systems
   - Verify functionality
   - Monitor for issues
   - Return to normal operations

6. Lessons Learned
   - Document incident
   - Review response
   - Update procedures
   - Implement improvements

# Incident Response Commands
# Isolate compromised system
iptables -A INPUT -j DROP
iptables -A OUTPUT -j DROP

# Capture memory dump
sudo dd if=/dev/mem of=/tmp/memory.dump

# Capture disk image
sudo dd if=/dev/sda of=/tmp/disk.img bs=4M

# Check for suspicious processes
ps aux | grep -v grep | sort -nrk 3,3 | head -10
lsof -i -P -n | grep LISTEN

# Check network connections
netstat -antp
ss -tulpn

# Check for unauthorized users
who
last
lastlog

# Check for privilege escalation
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null

# Check logs
journalctl -xe
tail -f /var/log/auth.log
tail -f /var/log/syslog
```

## Best Practices Summary

1. **Defense in Depth** - Multiple layers of security
2. **Least Privilege** - Minimum necessary access
3. **Fail Secure** - Fail in a secure state
4. **Separation of Duties** - No single point of failure
5. **Security by Design** - Build security in from the start
6. **Regular Updates** - Keep everything patched
7. **Encrypt Everything** - Data at rest and in transit
8. **Monitor Continuously** - Detect threats early
9. **Test Security** - Regular security assessments
10. **Incident Response Plan** - Be prepared for breaches
11. **Security Training** - Educate your team
12. **Document Everything** - Procedures and configurations

## Quick Security Checks

```bash
# Daily checks
- Monitor security alerts
- Review failed login attempts
- Check for unusual network activity
- Verify backup completion

# Weekly checks
- Review access logs
- Update security patches
- Scan for vulnerabilities
- Review firewall rules

# Monthly checks
- Update dependencies
- Review user accounts
- Test incident response
- Security awareness training

# Quarterly checks
- Penetration testing
- Security audit
- Review and update policies
- Disaster recovery testing
```
