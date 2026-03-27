# Authentication & Security

## Current Authentication Status

⚠️ **WARNING:** The current implementation has **NO AUTHENTICATION** on the webhook endpoint.

### Current State

| Endpoint | Authentication | Authorization | Status |
|----------|---------------|---------------|--------|
| `POST /api/V1/webhooks/order/` | ❌ None | ❌ None | **INSECURE** |
| `GET /admin/` | ✅ Django Auth | ✅ Staff/Superuser | **SECURE** |

---

## Security Vulnerabilities

### 1. No Authentication on Webhook Endpoint

**Risk:** Anyone can send webhook events to the endpoint.

**Impact:**
- Fake sales notifications
- System abuse and spam
- Data pollution in database
- Unwanted notifications to administrators

**Current Code:**
```python
class WebhookOrderView(APIView):
    # No authentication_classes
    # No permission_classes
    def post(self, request):
        # Processes any request
        pass
```

### 2. No Rate Limiting

**Risk:** Endpoint can be flooded with requests.

**Impact:**
- Denial of Service (DoS)
- Excessive notification costs
- Database bloat

### 3. No Request Validation

**Risk:** Malformed or malicious payloads accepted.

**Impact:**
- Data integrity issues
- Potential injection attacks

### 4. No HTTPS Enforcement

**Risk:** Data transmitted in plain text.

**Impact:**
- Credential interception
- Data exposure

---

## Recommended Security Implementation

### 1. API Key Authentication

Implement API key authentication for the webhook endpoint:

```python
# authentication.py
from rest_framework import authentication
from rest_framework import exceptions
from django.conf import settings

class APIKeyAuthentication(authentication.BaseAuthentication):
    """Custom API Key authentication for webhooks."""
    
    def authenticate(self, request):
        api_key = request.headers.get('X-API-Key')
        
        if not api_key:
            return None  # No credentials provided
        
        if api_key != settings.WEBHOOK_API_KEY:
            raise exceptions.AuthenticationFailed('Invalid API key')
        
        return (None, None)  # Authentication successful
```

**Settings Configuration:**
```python
# settings.py
WEBHOOK_API_KEY = config('WEBHOOK_API_KEY', default='')
```

**Environment Variable:**
```env
WEBHOOK_API_KEY=your-secure-random-api-key-here
```

**View Implementation:**
```python
# views.py
from rest_framework.views import APIView
from rest_framework.authentication import SessionAuthentication
from .authentication import APIKeyAuthentication

class WebhookOrderView(APIView):
    authentication_classes = [APIKeyAuthentication, SessionAuthentication]
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        # Only authenticated requests reach here
        pass
```

### 2. Rate Limiting

Implement rate limiting using `django-ratelimit`:

**Installation:**
```bash
pip install django-ratelimit
```

**Configuration:**
```python
# settings.py
INSTALLED_APPS = [
    # ...
    'ratelimit',
]

RATELIMIT_ENABLE = True
RATELIMIT_VIEW = 'webhooks.views.rate_limit_exceeded'
```

**View Decorator:**
```python
from ratelimit.decorators import ratelimit
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class WebhookOrderView(APIView):
    @ratelimit(key='ip', rate='100/h', block=True)
    def post(self, request):
        # Limited to 100 requests per hour per IP
        pass

def rate_limit_exceeded(request, exception):
    return Response(
        {'error': 'Rate limit exceeded. Try again later.'},
        status=status.HTTP_429_TOO_MANY_REQUESTS
    )
```

### 3. Request Validation

Implement request validation with serializers:

```python
# serializers.py
from rest_framework import serializers

class WebhookOrderSerializer(serializers.Serializer):
    event_type = serializers.CharField(max_length=50, required=False)
    product = serializers.CharField(max_length=255, required=True)
    quantity = serializers.IntegerField(min_value=1, required=True)
    product_cost_price = serializers.DecimalField(
        max_digits=10, 
        decimal_places=2, 
        min_value=0,
        required=True
    )
    product_selling_price = serializers.DecimalField(
        max_digits=10, 
        decimal_places=2, 
        min_value=0,
        required=True
    )
    
    def validate(self, data):
        """Validate that selling price >= cost price."""
        if data['product_selling_price'] < data['product_cost_price']:
            raise serializers.ValidationError(
                "Selling price cannot be less than cost price"
            )
        return data
```

**View Usage:**
```python
from .serializers import WebhookOrderSerializer

class WebhookOrderView(APIView):
    def post(self, request):
        serializer = WebhookOrderSerializer(data=request.data)
        
        if not serializer.is_valid():
            return Response(
                serializer.errors,
                status=status.HTTP_400_BAD_REQUEST
            )
        
        validated_data = serializer.validated_data
        # Process validated data
```

### 4. HTTPS Enforcement

**Django Settings:**
```python
# settings.py
SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

**Web Server Configuration (Nginx):**
```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    # ... rest of configuration
}
```

---

## Security Best Practices

### Environment Variables

**✅ DO:**
```python
# settings.py
from decouple import config

SECRET_KEY = config('SECRET_KEY')
DATABASE_PASSWORD = config('DATABASE_PASSWORD')
API_KEY = config('API_KEY')
```

```env
# .env (never commit this file)
SECRET_KEY=super-secret-key-xyz123
DATABASE_PASSWORD=db-password-abc456
API_KEY=api-key-def789
```

**❌ DON'T:**
```python
# NEVER hardcode secrets
SECRET_KEY = 'my-secret-key-123'
API_KEY = 'my-api-key-456'
```

### Logging Security

**✅ DO:**
```python
import logging

logger = logging.getLogger(__name__)

def process_webhook(request):
    logger.info(f'Webhook received: event_type={request.data.get("event_type")}')
    # Log non-sensitive information only
```

**❌ DON'T:**
```python
# NEVER log sensitive data
logger.info(f'API Key: {api_key}')
logger.info(f'Password: {password}')
logger.info(f'Request headers: {request.headers}')  # May contain auth tokens
```

### Error Handling

**✅ DO:**
```python
from rest_framework.response import Response
from rest_framework import status
import logging

logger = logging.getLogger(__name__)

def post(self, request):
    try:
        # Process request
        pass
    except Exception as e:
        logger.error(f'Webhook processing error: {str(e)}')
        # Return generic error to client
        return Response(
            {'error': 'Processing failed'},
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )
```

**❌ DON'T:**
```python
# NEVER expose stack traces
def post(self, request):
    try:
        # Process request
        pass
    except Exception as e:
        return Response({'error': str(e), 'traceback': traceback.format_exc()})
```

---

## Admin Security

### Current Admin Security

The Django admin interface uses Django's built-in authentication:

**Features:**
- Username/password authentication
- Session management
- Permission-based access control
- Staff status requirement

**Configuration:**
```python
# admin.py
@admin.register(Webhook)
class WebhookAdmin(admin.ModelAdmin):
    # Only staff users can access
    pass
```

### Admin Security Best Practices

1. **Use Strong Passwords:**
   ```bash
   python manage.py createsuperuser
   # Use strong, unique password
   ```

2. **Enable Two-Factor Authentication:**
   ```bash
   pip install django-two-factor-auth
   ```

3. **Limit Admin Access by IP:**
   ```python
   # urls.py
   from django.contrib import admin
   from django.http import HttpResponseForbidden
   
   def admin_view(request):
       if request.META['REMOTE_ADDR'] not in ['127.0.0.1']:
           return HttpResponseForbidden()
       return admin.site.urls(request)
   ```

4. **Custom Admin URL:**
   ```python
   # urls.py
   urlpatterns = [
       path('secure-admin-xyz/', admin.site.urls),  # Non-standard URL
   ]
   ```

---

## Security Checklist

### Pre-Deployment Checklist

- [ ] Implement API key authentication
- [ ] Enable rate limiting
- [ ] Configure HTTPS/SSL
- [ ] Set up request validation
- [ ] Configure secure environment variables
- [ ] Enable security logging
- [ ] Set up admin two-factor authentication
- [ ] Configure firewall rules
- [ ] Set up intrusion detection
- [ ] Review and update CORS settings

### Ongoing Security

- [ ] Regular dependency updates
- [ ] Security audit logs review
- [ ] Penetration testing
- [ ] API key rotation
- [ ] Backup verification
- [ ] Monitor for suspicious activity

---

## Security Headers

### Django Middleware Configuration

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # ... other middleware
]

# Security headers
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'
```

### Recommended Headers

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```

---

## Incident Response

### If Security Breach Detected

1. **Immediate Actions:**
   - Rotate all API keys
   - Change all passwords
   - Review access logs
   - Identify affected data

2. **Investigation:**
   - Analyze logs for attack vector
   - Determine scope of breach
   - Document timeline

3. **Recovery:**
   - Patch vulnerability
   - Restore from clean backup if needed
   - Notify affected parties if required

4. **Post-Incident:**
   - Document lessons learned
   - Update security procedures
   - Implement additional controls

---

## Compliance Considerations

### Data Protection

- **GDPR:** Ensure proper data handling for EU users
- **LGPD:** Brazilian data protection compliance
- **CCPA:** California consumer privacy

### Recommendations

1. **Data Minimization:** Only collect necessary data
2. **Purpose Limitation:** Use data only for intended purpose
3. **Storage Limitation:** Implement data retention policies
4. **Access Control:** Restrict data access to authorized personnel

---

## Next Steps

- Review [System Modeling](system-modeling.md) for security flow diagrams
- Check [Development Guide](development.md) for secure coding practices
- See [Deploy Guide](deploy.md) for production security configuration
