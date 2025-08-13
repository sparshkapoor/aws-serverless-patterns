# Serverless Users API

A production-ready serverless REST API built with AWS SAM that provides comprehensive user management with JWT authentication via Amazon Cognito and role-based access control.

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   API Gateway   │────│ Lambda Authorizer │────│ Amazon Cognito  │
│                 │    │                  │    │   User Pool     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │
         ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ Lambda Function │────│   Amazon X-Ray   │    │  Amazon DynamoDB│
│  (Users CRUD)   │    │    (Tracing)     │    │   Users Table   │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

**Core Services:**
- **AWS Lambda** - Serverless compute for API operations and JWT authorization
- **Amazon API Gateway** - RESTful API with custom authorizer integration
- **Amazon DynamoDB** - NoSQL database with on-demand billing
- **Amazon Cognito** - User authentication and JWT token management
- **AWS X-Ray** - Distributed tracing and performance monitoring

## ✨ Features

- 🔐 **JWT Authentication** - Secure token-based auth with Cognito
- 👥 **Role-Based Access** - Admin vs User permissions
- 🛡️ **Custom Authorizer** - Fine-grained resource-level authorization
- 📊 **Full CRUD Operations** - Complete user lifecycle management
- 🔍 **AWS X-Ray Tracing** - Request monitoring and debugging
- 🌐 **CORS Enabled** - Ready for web application integration
- ⚡ **Serverless** - Auto-scaling with pay-per-use pricing

## 🔌 API Endpoints

| Method | Endpoint | Description | Access Level |
|--------|----------|-------------|--------------|
| `GET` | `/users` | List all users | 🔴 Admin Only |
| `POST` | `/users` | Create new user | 🔴 Admin Only |
| `GET` | `/users/{userid}` | Get user details | 🟡 User/Admin |
| `PUT` | `/users/{userid}` | Update user | 🟡 User/Admin |
| `DELETE` | `/users/{userid}` | Delete user | 🟡 User/Admin |

## 🚀 Quick Start

### Prerequisites
```bash
# Required tools
aws --version        # AWS CLI v2
sam --version        # SAM CLI
python3 --version    # Python 3.10+
docker --version     # Docker for local testing
```

### Installation
```bash
# 1. Clone and navigate
git clone <repository-url>
cd users

# 2. Build application
sam build

# 3. Deploy with guided setup
sam deploy --guided
```

### Configuration
After deployment, update `env.json` for local testing:
```json
{
    "UsersFunction": {
        "USERS_TABLE": "ws-serverless-patterns-users-Users"
    }
}
```

## 🔑 Authentication Setup

### 1. Create Cognito User
```bash
aws cognito-idp admin-create-user \
  --user-pool-id <USER_POOL_ID> \
  --username user@example.com \
  --user-attributes Name=email,Value=user@example.com Name=name,Value="John Doe" \
  --temporary-password TempPass123!
```

### 2. Get JWT Token
```bash
export ID_TOKEN=$(aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id <CLIENT_ID> \
  --auth-parameters USERNAME=user@example.com,PASSWORD=<PASSWORD> \
  --query 'AuthenticationResult.IdToken' \
  --output text)
```

### 3. Set API Endpoint
```bash
export API_ENDPOINT=https://<API_ID>.execute-api.us-east-1.amazonaws.com/Prod
```

## 📡 API Usage Examples

### List Users (Admin Only)
```bash
curl -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/users
```

### Create User (Admin Only)
```bash
curl -X POST \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane Smith","email":"jane@example.com"}' \
  $API_ENDPOINT/users
```

### Get User by ID
```bash
curl -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/users/64c8e4a8-1001-7067-6373-7afe3e367291
```

### Update User
```bash
curl -X PUT \
  -H "Authorization: Bearer $ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane Doe","email":"jane.doe@example.com"}' \
  $API_ENDPOINT/users/64c8e4a8-1001-7067-6373-7afe3e367291
```

### Delete User
```bash
curl -X DELETE \
  -H "Authorization: Bearer $ID_TOKEN" \
  $API_ENDPOINT/users/64c8e4a8-1001-7067-6373-7afe3e367291
```

## 🧪 Local Development

### Start Local API
```bash
sam local start-api -n env.json --region us-east-1
```

### Test Individual Functions
```bash
# Test user creation
sam local invoke -e events/event-post-user.json -n env.json --region us-east-1

# Test user retrieval
sam local invoke -e events/event-get-user-by-id.json -n env.json --region us-east-1

# Test user deletion
sam local invoke -e events/event-delete-user-by-id.json -n env.json --region us-east-1
```

### Available Test Events
- `event-get-all-users.json` - List all users
- `event-post-user.json` - Create user
- `event-get-user-by-id.json` - Get specific user
- `event-put-user.json` - Update user
- `event-delete-user-by-id.json` - Delete user

## 🔐 Authorization Model

### User Permissions
- ✅ Access own user resource (`/users/{their-userid}`)
- ✅ CRUD operations on own data only
- ❌ Cannot access other users' data
- ❌ Cannot list all users

### Admin Permissions (`apiAdmins` group)
- ✅ Full access to all user resources
- ✅ List all users
- ✅ Create new users
- ✅ CRUD operations on any user
- ✅ Administrative privileges

### Authorization Flow
1. **JWT Validation** - Token signature and expiration verified
2. **User Identification** - Extract user ID from token claims
3. **Permission Check** - Validate resource access based on user/admin role
4. **Policy Generation** - Create IAM policy for API Gateway

## 📁 Project Structure

```
users/
├── src/api/
│   ├── users.py              # Main CRUD operations handler
│   └── authorizer.py         # JWT validation & authorization
├── events/                   # Local testing event files
│   ├── event-post-user.json
│   ├── event-get-user-by-id.json
│   └── ...
├── tests/
│   ├── unit/                 # Unit test files
│   └── integration/          # Integration test files
├── template.yaml             # SAM infrastructure template
├── requirements.txt          # Python dependencies
├── env.json                 # Local environment variables
└── README.md                # This documentation
```

## 🔧 Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `USERS_TABLE` | DynamoDB table name | `ws-serverless-patterns-users-Users` |
| `USER_POOL_ID` | Cognito User Pool ID | `us-east-1_bHyposPHv` |
| `APPLICATION_CLIENT_ID` | Cognito Client ID | `tel4b5g96a7e358o4r841hlcg` |
| `ADMIN_GROUP_NAME` | Admin group name | `apiAdmins` |

## 📊 Monitoring & Observability

### AWS X-Ray Tracing
- **Enabled** for all Lambda functions
- **Request tracking** across service boundaries
- **Performance insights** and bottleneck identification

### CloudWatch Integration
- **Function logs** for debugging
- **API Gateway metrics** for performance monitoring
- **Custom metrics** for business insights

### Error Handling
- **Structured error responses** with appropriate HTTP status codes
- **Detailed logging** for troubleshooting
- **Graceful degradation** for service failures

## 🚀 Deployment Outputs

Post-deployment, you'll receive these key outputs:

```yaml
APIEndpoint: https://ds82kllhfh.execute-api.us-east-1.amazonaws.com/Prod
UserPool: us-east-1_bHyposPHv
UserPoolClient: tel4b5g96a7e358o4r841hlcg
UsersTable: ws-serverless-patterns-users-Users
CognitoLoginURL: https://tel4b5g96a7e358o4r841hlcg.auth.us-east-1.amazoncognito.com/login...
CognitoAuthCommand: aws cognito-idp initiate-auth --auth-flow USER_PASSWORD_AUTH...
```

## 🛡️ Security Features

- **JWT Token Validation** - Cryptographic signature verification
- **Resource-Level Authorization** - Fine-grained access control
- **CORS Configuration** - Secure cross-origin requests
- **Input Validation** - Request payload sanitization
- **Error Sanitization** - No sensitive data in error responses

## 🤝 Contributing

1. **Fork** the repository
2. **Create** feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** changes (`git commit -m 'Add amazing feature'`)
4. **Push** to branch (`git push origin feature/amazing-feature`)
5. **Open** Pull Request

## 📄 License

This project is licensed under the **MIT-0 License** - see the [LICENSE](LICENSE) file for details.

## 🆘 Troubleshooting

### Common Issues

**Authentication Errors**
```bash
# Verify token format
echo $ID_TOKEN | cut -d'.' -f2 | base64 -d

# Check token expiration
aws cognito-idp get-user --access-token $ACCESS_TOKEN
```

**Local Testing Issues**
```bash
# Ensure Docker is running
docker ps

# Verify DynamoDB table exists
aws dynamodb describe-table --table-name ws-serverless-patterns-users-Users
```

**Authorization Failures**
- Check user group membership in Cognito
- Verify API Gateway authorizer configuration
- Review CloudWatch logs for detailed error messages

### Support Resources
- 📚 [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/)
- 🔐 [Amazon Cognito User Guide](https://docs.aws.amazon.com/cognito/)
- 🚀 [API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/)

---

**Built with ❤️ using AWS Serverless Technologies**