VaultSharp
==========

A cross-platform .NET Library for HashiCorp's Vault - A Secret Management System.

**VaultSharp NuGet:** [![NuGet](https://img.shields.io/nuget/dt/VaultSharp.svg?style=flat)](https://www.nuget.org/packages/VaultSharp)	

**VaultSharp Latest Documentation:** Inline Below and also at: http://rajanadar.github.io/VaultSharp/

**VaultSharp Gitter Lobby:** [Gitter Lobby](https://gitter.im/rajanadar-VaultSharp/Lobby)

**Older VaultSharp 0.6.x Documentation:** [0.6.x Docs](https://github.com/rajanadar/VaultSharp/blob/master/README-0.6.x.md)

**Report Issues/Feedback:** [Create a VaultSharp GitHub issue](https://github.com/rajanadar/VaultSharp/issues/new)

[![NuGet](https://img.shields.io/nuget/dt/VaultSharp.svg?style=flat)](https://www.nuget.org/packages/VaultSharp)	
[![Join the chat at https://gitter.im/rajanadar-VaultSharp/Lobby](https://badges.gitter.im/rajanadar-VaultSharp/Lobby.svg)](https://gitter.im/rajanadar-VaultSharp/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)	
[![License](https://img.shields.io/:license-apache%202.0-brightgreen.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)	
[![Build status](https://ci.appveyor.com/api/projects/status/aldh4a6n2t7hthdv?svg=true)](https://ci.appveyor.com/project/rajanadar/vaultsharp)

### What is VaultSharp?	

* VaultSharp is a .NET Standard 1.3 (and .NET 4.5) cross-platform C# Library that can be used in any .NET application to interact with Hashicorp's Vault.	
* The Vault system is a secret management system built as an Http Service by Hashicorp.

VaultSharp has been re-designed ground up, to give a structured user experience across the various auth methods, secrets engines & system apis.
Also, the Intellisense on IVaultClient class should help. I have tried to add a lot of documentation.

### Give me a quick snippet for use!

 * Add a Nuget reference to VaultSharp as follows ```Install-Package VaultSharp -Version <latest_version>```
 * Instantiate a IVaultClient as follows:

 ```cs	
// Initialize one of the several auth methods.
IAuthMethodInfo authMethod = new TokenAuthMethodInfo("MY_VAULT_TOKEN");

// Initialize settings. You can also set proxies, custom delegates etc. here.
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// Use client to read a key-value secret.
var kv2Secret = await vaultClient.V1.Secrets.KeyValue.V2.ReadSecretAsync("secret-name");

// Generate a dynamic Consul credential
var consulCreds = await vaultClient.V1.Secrets.Consul.GetCredentialsAsync(consulRole, consulMount);	
var consulToken = consulCredentials.Data.Token;
```

### How to reuse Vault Client Token?

* **TLDR:** The same Vault Client Token will be intelligently **reused** for the lifetime of the client.
* **NOTE:** The Vault Client Token is lazy loaded; it becomes available via the ReturnedLoginAuthInfo method once the first Vault API is requested.
Example:
```cs
 // Setup Vault Client via Github Auth Method
 IAuthMethodInfo authMethod = new GitHubAuthMethodInfo(personalAccessToken);
 var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);
 IVaultClient vaultClient = new VaultClient(vaultClientSettings);
 
 // If the token is attempted to be read at this time; it is null as it is lazy loaded on first Vault API call
 // var token = authMethod.ReturnedLoginAuthInfo.ClientToken; -->null!

 // Use client to read a key-value secret and trigger the load of ClientToken
var kv2Secret = await vaultClient.V1.Secrets.KeyValue.V2.ReadSecretAsync("secret-name");
 // now ReturnedLoginAuthInfo method will provide a valid ClientToken for further use
 var token = authMethod.ReturnedLoginAuthInfo.ClientToken;
```
* The Vault Client Token can be persisted and shared across clients via a store like Redis. Redis TTL capabilities can be leveraged so that the token will be expired automatically using the returned LeaseDurationSeconds value.
**Pseudo-code** flow could be like below:
```cs
 // Start Vault Client initialization
 if VaultClient initialized => continue 
 // Vault Client Token not found in Redis
 // or within 1 hour of expiring
 if NotExistOrNearlyExpireToken(appName) {
    settingFromGithubAuth <= LoginWithPersonalToken(pToken)
    NewVaultClient(settingFromGithubAuth)
 } else {
    vToken <= ReadFromRedis(appName)
    settingFromToken <= LoginWithClientToken(vToken)
    NewVaultClient(settingFromToken)
 }
 // Dummy operation so token will be loaded
 ReadSecretsKV("secret-name")
 // Persist token with TTL into Redis
 token <= authMethod.ReturnedLoginAuthInfo.ClientToken
 ttls <= authMethod.ReturnedLoginAuthInfo.LeaseDurationSeconds
 StoreToRedis(token, ttls)
 // End Vault Client initialization
```

### Gist of the features

 * VaultSharp 0.10.x supports 
   - All the Auth Methods for Logging  into Vault. (AppRole, AWS, Azure, GitHub, Google Cloud, JWT/OIDC, Kubernetes, LDAP, Okta, RADIUS, TLS, Tokens & UserPass)
   - All the secret engines to get dynamic credentials. (AD, AWS EC2 and IAM, Consul, Cubbyhole, Databases, Google Cloud, Key-Value, Nomad, PKI, RabbitMQ, SSH and TOTP)
   - Several system APIs including enterprise vault apis
 * You can also bring your own "Auth Method" by providing a custom delegate to fetch a token from anywhere.
 * VaultSharp has first class support for Consul engine.
 * KeyValue engine supports both v1 and v2 apis.
 * Abundant intellisense.
 * Provides hooks into http-clients to set custom proxy settings etc.

### VaultSharp - Supported .NET Platforms

VaultSharp is built on **.NET Standard 1.3** & **.NET Framework 4.5**. This makes it highly compatible and cross-platform.

The following platforms are supported due to that.

 * .NET Core 1.0 and above including .NET Core 2.0
 * .NET Framework 4.5 and above
 * Mono 4.6 and above
 * Xamarin.iOS 10.0 and above
 * Xamarin Mac 3.0 and above
 * Xamarin.Android 7.0 and above
 * UWP 10.0 and above
 
 Source: https://github.com/dotnet/standard/blob/master/docs/versions.md

### VaultSharp and Consul Support

* VaultSharp supports dynamic Consul credential generation.
* Please look at the API usage in the 'Consul' section of 'Secrets Engines' below, to see all the Consul related methods in action.

### Auth Methods

* VaultSharp supports all authentication methods supported by the Vault Service
* Here is a sample to instantiate the vault client with each of the authentication backends.

#### AliCloud Auth Method

```cs
// setup the AliCloud based auth to get the right token.

IAuthMethodInfo authMethod = new AliCloudAuthMethodInfo(roleName, base64EncodedIdentityRequestUrl, base64EncodedIdentityRequestHeaders); 
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the AliCloud jwt
```

#### App Role Auth Method

```cs
// setup the AppRole based auth to get the right token.

IAuthMethodInfo authMethod = new AppRoleAuthMethodInfo(roleId, secretId);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the app role and secret id.
```

#### AWS Auth Method

AWS Auth method has 2 flavors. An EC2 way and an IAM way. Here are examples for both.

##### AWS Auth Method - EC2

```cs
// setup the AWS-EC2 based auth to get the right token.

IAuthMethodInfo authMethod = new EC2AWSAuthMethodInfo(pkcs7, null, null, nonce, roleName);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the aws-ec2 role
```

```cs
// setup the AWS-EC2 based auth to get the right token.

IAuthMethodInfo authMethod = new EC2AWSAuthMethodInfo(null, identity, signature, nonce, roleName);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the aws-ec2 role
```

##### AWS Auth Method - IAM

```cs
// setup the AWS-IAM based auth to get the right token.

IAuthMethodInfo authMethod = new IAMAWSAuthMethodInfo(nonce, roleName); // uses default requestHeaders
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the aws-iam role
```

#### Azure Auth Method

```cs
// setup the Azure based auth to get the right token.

IAuthMethodInfo authMethod = new AzureAuthMethodInfo(roleName, jwt); 
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the azure jwt
```

#### GitHub Auth Method

```cs
IAuthMethodInfo authMethod = new GitHubAuthMethodInfo(personalAccessToken);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the github token.
```

#### Google Cloud Auth Method

```cs
// setup the Google Cloud based auth to get the right token.

IAuthMethodInfo authMethod = new GoogleCloudAuthMethodInfo(roleName, jwt); 
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the Google Cloud jwt
```

#### JWT/OIDC Auth Method

```cs
// setup the JWT/OIDC based auth to get the right token.

IAuthMethodInfo authMethod = new JWTAuthMethodInfo(roleName, jwt); 
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the jwt
```

#### Kubernetes Auth Method

```cs
// setup the Kubernetes based auth to get the right token.

IAuthMethodInfo authMethod = new KubernetesAuthMethodInfo(roleName, jwt); 
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the Kubernetes jwt
```

#### LDAP Authentication Backend

```cs
IAuthMethodInfo authMethod = new LDAPAuthMethodInfo(userName, password);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the LDAP username and password.
```

#### Okta Auth Method

```cs
IAuthMethodInfo authMethod = new OktaAuthMethodInfo(userName, password);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the Okta username and password.
```

#### RADIUS Auth Method

```cs
IAuthMethodInfo authMethod = new RADIUSAuthMethodInfo(userName, password);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
// vault token/policies mapped to the RADIUS username and password.
```

#### Certificate (TLS) Auth Method

```cs
var clientCertificate = new X509Certificate2(certificatePath, certificatePassword, 
                            X509KeyStorageFlags.Exportable | X509KeyStorageFlags.PersistKeySet);

IAuthMethodInfo authMethod = new CertAuthMethodInfo(clientCertificate);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
vault token/policies mapped to the client certificate.
```

#### Token Auth Method

```cs
IAuthMethodInfo authMethod = new TokenAuthMethodInfo(vaultToken);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
vault token/policies mapped to the vault token.
```

#### Username and Password Auth Method

```cs
IAuthMethodInfo authMethod = new UserPassAuthMethodInfo(username, password);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);

// any operations done using the vaultClient will use the 
vault token/policies mapped to the username/password.
```

#### Custom Auth Method - Bring your own Vault Token

 * VaultSharp also supports a custom way to provide the Vault auth token to VaultSharp.
 * In this approach, you are free to provide any delegate that returns the Vault token.
 * The token can be retrieved from a database, another secret engine, from a file, etc.

```cs
// Func<Task<String>> getTokenAsync = a custom async method to return the vault token.
IAuthMethodInfo authMethod = new CustomAuthMethodInfo("my-own-token-auth-method", getTokenAsync);
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);
```

#### App Id Auth Method (DEPRECATED)

* Please note that the app-id auth backend has been deprecated by Vault. They recommend us to use the AppRole backend.
* So VaultSharp doesn't support App Id natively. 
* If you are in dire need of the App Id support, please raise an issue.

#### MFA (LEGACY/UNSUPPORTED)

* Please note that this legacy Auth Method is not supported by Vault anymore.
* Instead Vault Enterprise contains a fully-supported MFA system.
* It is significantly more complete and flexible and which can be used throughout Vault's API. 
* Please see the *System Backend* section of the docs for the Enterprise MFA apis.

### Secrets Engines

* VaultSharp supports all secrets engines supported by the Vault Service
* Here is a sample to instantiate the vault client with each of the secrets engine

All of the below examples assume that you have a vault client instance ready. e.g.

 ```cs	
// Initialize one of the several auth methods.
IAuthMethodInfo authMethod = new TokenAuthMethodInfo("MY_VAULT_TOKEN");

// Initialize settings. You can also set proxies, custom delegates etc. here.
var vaultClientSettings = new VaultClientSettings("https://MY_VAULT_SERVER:8200", authMethod);

IVaultClient vaultClient = new VaultClient(vaultClientSettings);
```

#### Active Directory Secrets Engine

##### Retrieving Passwords (offering credentials)

 * This method offers the credential information for a given role.

 ```cs	
var adCreds = await vaultClient.V1.Secrets.ActiveDirectory.GetCredentialsAsync(role);
var currentPassword = adCreds.Data.CurrentPassword;
```

#### AWS Secrets Engine

##### Generate IAM Credentials

 * This endpoint generates dynamic IAM credentials based on the named role.

 ```cs	
var awsCreds = await vaultClient.V1.Secrets.AWS.GetCredentialsAsync(role);

var accessKey = awsCreds.Data.AccessKey;
var secretKey = awsCreds.Data.SecretKey;
var securityToken = awsCreds.Data.SecurityToken;
```

##### Generate IAM Credentials with STS

 * This generates a dynamic IAM credential with an STS token based on the named role.

 ```cs	
var awsCreds = await vaultClient.V1.Secrets.AWS.GenerateSTSCredentialsAsync(role, ttl);

var accessKey = awsCreds.Data.AccessKey;
var secretKey = awsCreds.Data.SecretKey;
var securityToken = awsCreds.Data.SecurityToken;
```

#### Azure Secrets Engine

##### Generate dynamic Azure credentials

 * This endpoint generates a new service principal based on the named role.

```cs
var azureCredentials = await vaultClient.V1.Secrets.Azure.GetCredentialsAsync(roleName);
var clientId = azureCredentials.Data.ClientId;
var clientSecret = azureCredentials.Data.ClientSecret;
```

#### Consul Secrets Engine

 * This endpoint generates a dynamic Consul token based on the given role definition.

```cs
// Generate a dynamic Consul credential
var consulCreds = await vaultClient.V1.Secrets.Consul.GetCredentialsAsync(consulRole);	
var consulToken = consulCredentials.Data.Token;
```

#### Cubbyhole Secrets Engine

##### Read Secret

 * This endpoint retrieves the secret at the specified location.

 ```cs	
var secret = await vaultClient.V1.Secrets.Cubbyhole.ReadSecretAsync(secretPath);
var secretValues = secret.Data;
```

##### List Secrets

 * This endpoint returns a list of secret entries at the specified location. 
 * Folders are suffixed with /. The input must be a folder; list on a file will not return a value. 
 * The values themselves are not accessible via this command.

 ```cs	
var secret = await vaultClient.V1.Secrets.Cubbyhole.ReadSecretPathsAsync(folderPath);
var paths = secret.Data;
```

##### Create/Update Secret

 * This endpoint stores a secret at the specified location.

 ```cs	
var value = new Dictionary<string, object> { { "key1", "val1" }, { "key2", 2 } };
await vaultClient.V1.Secrets.Cubbyhole.WriteSecretAsync(secretPath, value);
```

##### Delete Secret

 * This endpoint deletes the secret at the specified location.

 ```cs	
await vaultClient.V1.Secrets.Cubbyhole.DeleteSecretAsync(secretPath);
```

#### Databases Secrets Engine

##### Generate dynamic DB credentials

 * This endpoint generates a new set of dynamic credentials based on the named role.

 ```cs	
var dbCreds = await vaultClient.V1.Secrets.Database.GetCredentialsAsync(role);
var username = dbCreds.Data.Username;
var password = dbCreds.Data.Password;
```

#### Google Cloud Secrets Engine

##### Generate Secret (IAM Service Account Creds): OAuth2 Access Token

 * Generates an OAuth2 token with the scopes defined on the roleset. This OAuth access token can be used in GCP API calls

 ```cs	
var oauthSecret = await vaultClient.V1.Secrets.GoogleCloud.GenerateOAuth2TokenAsync(roleset);
var token = oauthSecret.Data.Token;
```
##### Generate Secret (IAM Service Account Creds): Service Account Key

 * Generates a service account key.

 ```cs	
var privateKeySecret = await vaultClient.V1.Secrets.GoogleCloud.GenerateServiceAccountKeyAsync(roleset, keyAlgorithm, privateKeyType);
var privateKeyData = privateKeySecret.Data.Base64EncodedPrivateKeyData;
```

#### Key Value Secrets Engine

 * VaultSharp supports both v1 and v2 of the Key Value Secrets Engine.
 * Here are examples for both.

##### Key Value Secrets Engine - V1

###### Create/Update Secret

 * This endpoint stores a secret at the specified location. 
 * If the value does not yet exist, the calling token must have an ACL policy granting the create capability. 
 * If the value already exists, the calling token must have an ACL policy granting the update capability.

 ```cs	
var value = new Dictionary<string, object> { { "key1", "val1" }, { "key2", 2 } };
await vaultClient.V1.Secrets.KeyValue.V1.WriteSecretAsync(secretPath, value);
```

###### Read Secret

 * Reads the secret at the specified location returning data.

```cs
// Use client to read a v1 key-value secret.
var kv1Secret = await vaultClient.V1.Secrets.KeyValue.V1.ReadSecretAsync("v1-secret-name");
Dictionary<string, object> dataDictionary = kv1Secret.Data;
```

###### List Secrets

 * This endpoint returns a list of key names at the specified location. 
 * Folders are suffixed with /. The input must be a folder; list on a file will not return a value. 
 * Note that no policy-based filtering is performed on keys; do not encode sensitive information in key names. 
 * The values themselves are not accessible via this command.

 ```cs	
var secret = await vaultClient.V1.Secrets.KeyValue.V1.ReadSecretsAsync(path);
var paths = secret.Data;
```

###### Delete Secret

 * This endpoint deletes the secret at the specified location.

 ```cs	
await vaultClient.V1.Secrets.KeyValue.V1.DeleteSecretAsync(secretPath);
```

##### Key Value Secrets Engine - V2

###### Create/Update Secret

 * This endpoint stores a secret at the specified location. 
 * If the value does not yet exist, the calling token must have an ACL policy granting the create capability. 
 * If the value already exists, the calling token must have an ACL policy granting the update capability.

 ```cs	
var value = new Dictionary<string, object> { { "key1", "val1" }, { "key2", 2 } };
await vaultClient.V1.Secrets.KeyValue.V2.WriteSecretAsync(secretPath, value, checkAndSet);
```

###### Read Secret

 * Reads the secret at the specified location returning data and metadata.

```cs
// Use client to read a v2 key-value secret.
var kv2Secret = await vaultClient.V1.Secrets.KeyValue.V2.ReadSecretAsync("v2-secret-name");
Dictionary<string, object> dataDictionary = kv2Secret.Data;
```

###### Read Metadata

 * Reads the secret metadata at the specified location returning.

```cs
var kv2SecretMetadata = await vaultClient.V1.Secrets.KeyValue.V2.ReadSecretMetadataAsync("v1-secret-name");
```

###### List Secrets

 * This endpoint returns a list of key names at the specified location. 
 * Folders are suffixed with /. The input must be a folder; list on a file will not return a value. 
 * Note that no policy-based filtering is performed on keys; do not encode sensitive information in key names. 
 * The values themselves are not accessible via this command.

 ```cs	
var secret = await vaultClient.V1.Secrets.KeyValue.V2.ReadSecretsAsync(path);
var paths = secret.Data;
```

###### Destroy Secret

 * This endpoint destroys the secret at the specified location for the given versions.

 ```cs	
await vaultClient.V1.Secrets.KeyValue.V2.DestroySecretAsync(secretPath, new List<int> { 1, 2 });
```

#### Identity Secrets Engine

Coming soon...

#### Nomad Secrets Engine

##### Generate dynamic DB credentials

 * Generates a dynamic Nomad token based on the given role definition.

```cs
var nomadCredentials = await vaultClient.V1.Secrets.Nomad.GetCredentialsAsync(roleName);
var accessorId = nomadCredentials.Data.AccessorId;
var secretId = nomadCredentials.Data.SecretId;
```

#### PKI (Cerificates) Secrets Engine

```cs
var certificateCredentialsRequestOptions = new CertificateCredentialsRequestOptions { // initialize };
var certSecret = await vaultClient.V1.Secrets.PKI.GetCredentialsAsync(pkiRoleName, certificateCredentialsRequestOptions);

var privateKeyContent = certSecret.Data.PrivateKeyContent;
```

#### RabbitMQ Secrets Engine

##### Generate dynamic DB credentials

 * This endpoint generates a new set of dynamic credentials based on the named role.

 ```cs	
var secret = await vaultClient.V1.Secrets.RabbitMQ.GetCredentialsAsync(role);
var username = secret.Data.Username;
var password = secret.Data.Password;
```

#### SSH Secrets Engine

##### Generate SSH credentials

 * This endpoint creates credentials for a specific username and IP with the parameters defined in the given role.

 ```cs	
var sshCreds = await vaultClient.V1.Secrets.SSH.GetCredentialsAsync(role, ipAddress, username);
var sshKey = sshCreds.Data.Key;
```

#### TOTP Secrets Engine

##### Generate Code

This endpoint generates a new time-based one-time use password based on the named key.

```cs
var totpSecret = await vaultClient.V1.Secrets.TOTP.GetCodeAsync(keyName);
var code = totpSecret.Data.Code;
```

##### Validate Code

This endpoint validates a time-based one-time use password generated from the named key.

```cs
var totpValidity = await vaultClient.V1.Secrets.TOTP.ValidateCodeAsync(keyName, code);
var valid = totpValidity.Data.Valid;
```

#### Transit Secrets Engine

##### Encrypt Method

###### Encrypt Single Item

```cs
var keyName = "test_key";

var context = "context1";
var plainText = "raja";
var encodedPlainText = Convert.ToBase64String(Encoding.UTF8.GetBytes(plainText));
var encodedContext = Convert.ToBase64String(Encoding.UTF8.GetBytes(context));

var encryptOptions = new EncryptRequestOptions
{
    Base64EncodedPlainText = encodedPlainText,
    Base64EncodedContext = encodedContext,
};

var encryptionResponse = await _authenticatedVaultClient.V1.Secrets.Transit.EncryptAsync(keyName, encryptOptions);
var cipherText = encryptionResponse.Data.CipherText;
```

###### Encrypt Batched Items

```cs
var encryptOptions = new EncryptRequestOptions
{
    BatchedEncryptionItems = new List<EncryptionItem>
    {
        new EncryptionItem { Base64EncodedContext = encodedContext1, Base64EncodedPlainText = encodedPlainText1 },
        new EncryptionItem { Base64EncodedContext = encodedContext2, Base64EncodedPlainText = encodedPlainText2 },
        new EncryptionItem { Base64EncodedContext = encodedContext3, Base64EncodedPlainText = encodedPlainText3 },
    }
};

var encryptionResponse = await _authenticatedVaultClient.V1.Secrets.Transit.EncryptAsync(keyName, encryptOptions);
var firstCipherText = encryptionResponse.Data.BatchedResults.First().CipherText;
```

##### Decrypt Method

###### Decrypt Single Item

```cs
var decryptOptions = new DecryptRequestOptions
{
    CipherText = cipherText,
    Base64EncodedContext = encodedContext,
};

var decryptionResponse = await _authenticatedVaultClient.V1.Secrets.Transit.DecryptAsync(keyName, decryptOptions);
var encodedPlainText = decryptionResponse.Data.Base64EncodedPlainText;
```

###### Decrypt Batched Item

```cs
var decryptOptions = new DecryptRequestOptions
{
    BatchedDecryptionItems = new List<DecryptionItem>
    {
        new DecryptionItem { Base64EncodedContext = encodedContext1, CipherText = cipherText1 },
        new DecryptionItem { Base64EncodedContext = encodedContext2, CipherText = cipherText2 },
        new DecryptionItem { Base64EncodedContext = encodedContext3, CipherText = cipherText3 },
    }
};

var decryptionResponse = await _authenticatedVaultClient.V1.Secrets.Transit.DecryptAsync(keyName, decryptOptions);
var firstEncodedPlainText = decryptionResponse.Data.BatchedResults.First().Base64EncodedPlainText;
```

### System Backend

 * The system backend is a default backend in Vault that is mounted at the /sys endpoint. 
 * This endpoint cannot be disabled or moved, and is used to configure Vault and interact with many of Vault's internal features.

VaultSharp already supports several of the System backend features.

```cs
// vaultClient.V1.System.<method> The method you are looking for.
```

Additional documentation coming soon...

### What is the deal with the Versioning of VaultSharp?

* This library is written for Hashicorp's Vault Service
* The Vault service is evolving constantly and the Hashicorp team is rapidly working on it.
* Pretty soon, they should have an 1.0.0 version of the Vault Service from Hashicorp.
* Because this client library is intended to facilititate the Vault Service operations, this library makes it easier for its consumers to relate to the Vault service it supports.
* Hence a version of 0.11.x denotes that this library will support the Vault 0.11.x Service Apis.
* Tomorrow when Vault Service gets upgraded to 1.0.0, this library will be modified accordingly and versioned as 1.0.0

### Can I use it in my PowerShell Automation?

* Absolutely. VaultSharp is a .NET Library. 
* This means, apart from using it in your C#, VB.NET, J#.NET and any .NET application, you can use it in PowerShell automation as well.
* Load up the DLL in your PowerShell code and execute the methods. PowerShell can totally work with .NET Dlls.

### All the methods are async. How do I use them synchronously?

* The methods are async as the defacto implementation. The recommended usage.
* However, there are innumerable scenarios where you would continue to want to use it synchronously.
* For all those cases, there are various options available to you.
* There is a lot of discussion around the right usage, avoiding deadlocks etc.
* This library allows you to set the 'continueAsyncTasksOnCapturedContext' option when you initialize the client.
* It is an optional parameter and defaults to 'false'
* Setting it to false, allows you to access the .Result property of the task with reduced/zero deadlock issues.
* There are other ways as well to invoke it synchronously, and  I leave it to the users of the library. (Task.Run etc.) 
* But please note that as much as possible, use it in an async manner. 

### In Conclusion

* If the above documentation doesn't help you, feel free to create an issue or email me. https://github.com/rajanadar/VaultSharp/issues/new

### Happy Coding folks!
