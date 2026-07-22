# API Authentication Overview


## 1 Overview of authentication methods
JumpServer API currently supports the following four authentication methods:

| Method | Use Case | Expiration | Example |
|------|----------|----------|------|
| Session | Called in the browser after logging in | session expired | `session.md` |
| Token | Temporary script/obtained after interactive login | Validity period | `token.md` |
| Private Token | Long-term background tasks/integrations | Does not automatically expire | `private_token.md` |
| Access Key | Third-party systems/signed requests | Manually revoked | `access_key.md` |

## 2 Quick Links
- [Session Authentication](session.md)
- [Token authentication](token.md)
- [Access Key Authentication](access_key.md)
- [Private Token Authentication](private_token.md)

## 3 Swagger UI Online Documentation
After completing JumpServer deployment, you can view and test the complete API by accessing swagger UI.
![api_swagger](../../../img/api_swagger.png)

> For detailed examples and precautions, please go to the corresponding subpage to view. This file is only used as a navigation overview, and the sample code will not be repeated. 




