# Hashicorp Vault plugin for Quorum

The Hashicorp Vault plugin for Quorum enables the storage of Quorum account private keys in a Hashicorp Vault as an alternative to storing as `keystore` files on the node.  It is an implementation of Quorum's [`account` plugin interface](https://docs.goquorum.com/en/latest/PluggableArchitecture/Plugins/account/interface/).

Managing Quorum accounts in a Hashicorp Vault offers several benefits over using the standard `geth` `keystore` files:

* Your account private keys are stored in a Hashicorp Vault which can be deployed on separate infrastructure to your Quorum node  

* Vault enables you to configure permissions on a per-secret basis to ensure account private keys can only be accessed by authorised users/applications 

## Plugin Configuration

```json
{
    "vault": "https://localhost:8200",
    "accountDirectory": "file:///path/to/accts",
    "unlock": ["1a31744b4a6ee9f3c3d1550beb56d53d2a4fa454"],
    "authentication": {
        "roleId": "env://HASHICORP_ROLE_ID",
        "secretId": "env://HASHICORP_SECRET_ID",
        "approlePath": "approle"
    },
    "tls": {
        "caCert": "file:///path/to/ca.pem",
        "clientCert": "file:///path/to/client.pem",
        "clientKey": "file:///path/to/client.key"
    }
}
```

| Field | Description |
| --- | --- |
| `vault` | Vault server URL |
| `accountDirectory` | Path to directory containing [account config files](#account-configuration) |
| `unlock` | List of accounts to retrieve from Vault at startup and store in memory |
| `authentication` | See [authentication](#authentication) |
| `tls` | See [tls](#tls) |

### authentication

The recommended way to authenticate the plugin with Vault is with the approle authentication method.  This requires the following configuration: 

| Field | Description |
| --- | --- |
| `roleId` | Environment variable URL for Approle role ID |
| `secretId` | Environment variable URL for Approle secret ID |
| `approlePath` | Approle API path the plugin will attempt to login with using the provided credentials |

Alternatively, an authentication token (e.g. root token or token obtained by logging in separately using the HTTP API) can be provided directly.  This is not recommended for production use but can be helpful during testing.

| Field | Description |
| --- | --- |
| `token` | Environment variable URL for Vault token |

### tls

For production use it is recommended to configure TLS on Vault.  If enabled, the plugin will require the following configuration:

| Field | Description |
| --- | --- |
| `caCert` | PEM-encoded CA certificate file URL |
| `clientCert` | PEM-encoded client certificate file URL |
| `clientKey` | PEM-encoded client key file URL |

## Account Creation Configuration
```json
{
    "secretEnginePath": "kv",
    "secretPath": "myacct",
    "CasValue": 4
}
```

| Field | Description |
| --- | --- |
| `secretEnginePath` | K/V v2 API path the plugin will store the new account in |
| `secretPath` | Secret name API path the plugin will store the new account at |
| `CasValue` / `insecureSkipCas` | See [Check-And-Set overwrite protection](#check-and-set-overwrite-protection) |

### Check-And-Set overwrite protection

The plugin makes use of the Vault API's Check-And-Set (CAS) feature to prevent accidental overwriting of data.

If a secret already exists at the specified location, `CasValue` must be provided and must equal the current version number of the secret.  

The CAS check can be skipped by setting `"insecureSkipCas": "true"`.  

**TODO below TODO below TODO below TODO below TODO below TODO below TODO below**

> Overwriting data may result in permanent data loss.  See **TODO** for more information on versioning

### What gets stored in Vault when creating accounts?
The string hex representations of the account address and private key are stored in the Vault as a key/value pair:

```json
{
  ...
  "data" : {
      "bcc328f4679fcc781d983da1c8be3d3baa6e5ae5" : "dfe8b73d2771380d3f36bd78ce537715e812d7797c0b055fe944cd42cc750853"
  },
  ...
}
```

## Account Configuration

Account configuration files are stored in an `accountDirectory`.  These configuration files specify the location of the account secret in the Vault.  

Typically these files do not have to be created or edited manually; the `geth account` CLI and API can be used to create new accounts when needed.

```json
{
   "Address" : "1a31744b4a6ee9f3c3d1550beb56d53d2a4fa454",
   "VaultAccount" : {
      "SecretEnginePath" : "kv",
      "SecretPath" : "myacct",
      "SecretVersion" : 13
   },
   "Version" : 1
}
```
