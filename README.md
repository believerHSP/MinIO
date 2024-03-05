# MinIO

Ref: Upload to MinIO through CMD: [ https://gist.github.com/PhilipSchmid/1fd2688ace9f51ecaca2788a91fec133 ]

# Keycloak-MinIO Integration & PBAC:
Configure Keycloak for PBAC
1. Create a Realm
   
2. Create a client with following configurations:
   client ID: <minio>
   Always display in UI: Toggle On 
   Valid redirect URIs: http://192.168.122.1:9001/oauth_callback
   web origins: *
   client Authentication: Toggle On
   Authentication Flow: Toggle On Standard Flow & Direct Access Grants

3. Client Scopes: allow Keycloak to map user attributes as part of the JSON Web Token (JWT) returned in authentication requests. This allows MinIO to reference 
                  those attributes when assigning policies to the user.
   ** This step creates the necessary client scope to support MinIO authorization after successful Keycloak authentication.

   Navigate to Client Scopes and create a New client scope for MInIO authorization.
   Name: <minio_acl>
   Display on consent screen: Toggle On
   Include in token space: Toggle On
 ￼
4. Once created, select the scope from the list and navigate to mappers.
   Select configure a new mapper to create a new mapping:
   Mapper Type: User Attribute
   Name: <minio-policy-mapper>
   User Attribute: policy
   Token Claim Name: policy
   Claim JSON Type: String
   Toggle On following fields:
   Add to ID token
   Add to access token
   Add to userinfo
   Add to token introspection

5. Once created, assign the Client Scope to the MinIO client.
   Navigate to clients and select the MinIO client.

   Select client scopes, then select add client scope.
   Select the previously created scope and set the assigned type to default.

6. Apply the Necessary Attribute to Keycloak Groups:
   Create a User & Set credentials for it.
   Create a attribute with policy as key and value with a custom policy created in minIO.

7. By configuring above options keycloak will add an attribute in JWT.
   You can test the configured policies of a user by using the Keycloak API:
   
    curl -d "client_id=minio" \
     -d "client_secret=up1oFkakcYoT3aiglnRI97ALYEd8ZzbO" \
     -d "grant_type=password" \
     -d "username=himansu" \
     -d "password=himansu@1234" \
     http://keycloak-url.example.net:8080/realms/REALM/protocol/openid-connect/token

     Use a JWT decoder to review the payload and ensure it contains the policy key with a MinIO policy.
   
9. Configure MinIO for Keycloak Authentication:

    podman run --name minio -dt \
   -p 9000:9000 \
   -p 9001:9001 \
   -e MINIO_ROOT_USER="admin" \
   -e MINIO_ROOT_PASSWORD="admin@123" \
   -e MINIO_IDENTITY_OPENID_CONFIG_URL_PRIMARY_IAM="http://192.168.122.1:9080/realms/minio/.well-known/openid-configuration" \
   -e MINIO_IDENTITY_OPENID_CLIENT_ID_PRIMARY_IAM="minio" \
   -e MINIO_IDENTITY_OPENID_CLIENT_SECRET_PRIMARY_IAM="up1oFkakcYoT3aiglnRI97ALYEd8ZzbO" \
   -e MINIO_IDENTITY_OPENID_CLAIM_NAME_PRIMARY_IAM="policy" \
   -e MINIO_IDENTITY_OPENID_DISPLAY_NAME_PRIMARY_IAM="OAuth_Login" \
   -e MINIO_IDENTITY_OPENID_SCOPES_PRIMARY_IAM="openid" \
   -e MINIO_IDENTITY_OPENID_REDIRECT_URI_DYNAMIC_PRIMARY_IAM="on" \
   -e MINIO_BROWSER_REDIRECT="http://192.168.122.1:9001/oauth_callback" \
      quay.io/minio/minio server /data  --console-address ":9001"

10. Created a Custom policy at Folder level:
    #keenable
     {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::b11"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::b11/f2b1/*",
                "arn:aws:s3:::b11/f2b1"
            ]
        },
        {
            "Effect": "Deny",
            "Action": [
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::b11/f1b1",
                "arn:aws:s3:::b11/f1b1/*"
            ]
        }
    ]
}

11. Open MinIO using http://192.168.122.1:9001
    Login using OAuth_Login tab. It will redirect you to Keycloak Authentication page. Fill the keycloak user and you will be redirected to MinIO web page.
    You will be having the access as per the policy assigned to user in keycloak.



















    
