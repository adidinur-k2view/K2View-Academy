# **Fabric Web Services Security** 

## Tokenization

Fabric provides a secured mechanism for web-services to access to the data it exposes - such as LUI data or Reference Tables.
This is done by means of authentication mechanism that support Web Service calls using either of the following credentials:
- username and password 
- secured token based on JSON Web Tokens (JWT) solution. 

JWT is an open industry standard method (RFC 7519 ) that securely represents claims between two parties. 

Tokens can be used in two modes:

- Secured mode, using a digital signature on the client side. The secret key will be shared as an output only once to be used by the client to generate the digital signature
- Unsecured mode signed by Fabric.

Fabric supports backward capability via Token Authentication and an enhanced "Create Token" command for secured tokens. 


## Tokenization in the Admin Panel

In order to generate a new token for accessing Fabric API, open the Admin Panel web page:

Step 1: Select the *Admin* -> *Security* -> *API keys* tab.

Step 2: Select the *Add API Key +* button on the upper right of the window.

Step 3: Fill in the following details:
- Name (Mandatory)
- Secured (Optional)

Step 4: Click on the *Save* button


If the secured option was selected, the secret key will appear in a pop=up window, with the ability to select and copy it.
```f151c40f-fede-4fb3-8010-398ffbc02329```


<img src="/articles/26_fabric_security/images/07_fabric_webToken.PNG">


Note that in cases where the secured option was not selected the token value is the token name itself.



## Tokenization from the Command Line

The creation of a token from the command line is described [here](/articles/17_fabric_credentials/02_fabric_credentials_commands.md#create-token)

## Granting Web-services Permissions to Roles 

When assigning a role to a user, different types of methods can be attributed. 
The roles supported are described [here](/articles/17_fabric_credentials/02_fabric_credentials_commands.md#grant-ws_name-to-role-)


### Web Service authorization using the API Key:

#### Project Web Services

Permission is granted to the role in the Web Service or to all Web Services and the API Key is assigned to the role.


#### Product Web Services

The API key is assigned to a user. The permissions for Product Web Services are defined by combining the API Key assigned to the user and the permissions of the roles assigned to the user.

Example:

``` 
    create user 'greg';
    create role 'writeRole';
    grant WRITE on * to 'writeRole';
    assign role 'writeRole' to user 'greg';
    create token 'test_token' user 'greg';
```

This sniplet shows how the WRITE permission (granted to the writeRole) role was assigned to the user; and how a token (test_token) reflecting this role/permission was generated for the user.

When trying to invoke the web service with the DELETE verb, using the 'test_token' token, Fabric will throw the following error as the delete permission was not granted to the specific token: 

``` "Com.k2view.cdbms.exceptions.UnauthorizedException: test_read is not allowed to perform [DELETE_INSTANCE]" ```



