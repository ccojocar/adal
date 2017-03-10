# Azure Active Directory library for Go

This project provides a stand alone Azure Active Directory library for Go. The code was extracted
from [go-autorest](https://github.com/Azure/go-autorest/) project, which is used as a base for
[azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go).

[![Build Status](https://travis-ci.org/cosmincojocar/adal.svg?branch=master)](https://travis-ci.org/cosmincojocar/adal) [![Go Report Card](https://goreportcard.com/badge/github.com/cosmincojocar/adal)](https://goreportcard.com/report/github.com/cosmincojocar/adal)oauth2


## Installation

```
go get -u github.com/cosmincojocar/adal
```

## Usage

An Active Directory application is required in order to use this library. An application can be registered in the [Azure Portal](https://portal.azure.com/) follow this [guidelines](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications) or using the [Azure CLI](https://github.com/Azure/azure-cli).

### Register an Azure AD Application with secret


1. Register a new application with a `secret` credential

   ```
   az ad app create \
      --display-name example-app \
      --homepage https://example-app/home \
      --identifier-uris https://example-app/app \
      --password secret
   ```

2. Create a service principal using the `Application ID` from previous step

   ```
   az ad sp create --id "Application ID"
   ```

   * Replace `Application ID` with `appId` from step 1.

### Register an Azure AD Application with certificate

1. Create a private key

   ```
   openssl genrsa -out "example-app.key" 2048
   ```

2. Create the certificate

   ```
   openssl req -new -key "example-app.key" -subj "/CN=example-app" -out "example-app.csr"
   openssl x509 -req -in "example-app.csr" -signkey "example-app.key" -out "example-app.crt" -days 10000
   ```

3. Create the PKCS12 version of the certificate containing also the private key

   ```
   openssl pkcs12 -export -out "example-app.pfx" -inkey "example-app.key" -in "example-app.crt" -passout pass:

   ```

4. Register a new application with the certificate content form `example-app.crt`

   ```
   certificateContents="$(tail -n+2 "example-app.crt" | head -n-1)"

   az ad app create \
      --display-name example-app \
      --homepage https://example-app/home \
      --identifier-uris https://example-app/app \
      --key-usage Verify --end-date 2018-01-01 \
      --key-value "${certificateContents}"
  ```

5. Create a service principal using the `Application ID` from previous step

   ```
   az ad sp create --id "APPLICATION_ID"
   ```

   * Replace `APPLICATION_ID` with `appId` from step 4.


### Grant the necessary permissions

Azure relies on a Role-Based Access Control (RBAC) model to manage the access to resources at a fine-grained
level. There is a set of [pre-defined roles](https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-built-in-roles)
which can be assigned to a service principal of an Azure AD application depending of your needs.

```
az role assignment create --assignee "SERVICE_PRINCIPAL_ID" --role "ROLE_NAME"
```

* Replace the `SERVICE_PRINCIPAL_ID` with the `appId` from previous step.
* Replace the `ROLE_NAME` with a role name of your choice.

It is also possible to define custom role definitions.

```
az role definition create --role-definition role-definition.json
```

* Check [custom roles](https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-control-custom-roles) for more details regarding the content of role-definition.json file.
