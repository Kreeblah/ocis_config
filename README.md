# Configuring OCIS on a Linux system with Nginx and Keycloak

This took me a very long time to figure out, so I wanted to document it in case I need it later, or in case it helps others.

## Setting up Nginx

Setting up Nginx is straightforward, and can be done via other documentation on the Internet.  An example configuration for proxying to OCIS can be found [here](./ocis.nginx).

## Setting up Keycloak

Installing Keycloak is beyond the scope of this document.  However, there is a configuration change that needs to be made in order to get it working with OCIS.

### Creating the required OCIS clients

In order to create the required clients for OCIS in Keycloak, you can visit [this page](https://github.com/owncloud/ocis/tree/master/deployments/examples/ocis_keycloak/config/keycloak/clients) in the OCIS repo and download the JSON payloads for the clients you want.  In order to import them into Keycloak, click on "Clients", and then "Import client".  Make sure you give the web client a unique ID.

**Note that the OCIS clients (Desktop/Mobile) have no way of setting your own client ID or secret for OpenID Connect flows, so if you use those clients, you'll need to use the payloads exactly as is, with the publicly-known values.  Make sure you understand the security implications of this before doing so.**

### Creating the required OCIS roles

OCIS requires specific roles to exist for automatic user creation and permission assignment via OpenID Connect.  These roles are:

1. ocisAdmin
2. ocisSpaceAdmin
3. ocisGuest
4. ocisUser

For the most flexibility, these can be created as realm roles.

### Creating a Client Scope in Keycloak

As an admin user in Keycloak, go to the "Client scopes" section of the realm and create a new one.  Name it whatever you'd like, with any description you'd like, with type "None", protocol "OpenID Connect", don't display it on the consent screen, and don't include it in the token scope.

Once the client scope has been created, click on it and go to the Mappers tab.  Create a new mapper by configuration with the type "User Realm Role".  Set it to be multivalued, give it the token claim name "roles" (without the quotes; this is very important), add it to ID and access tokens as well as userinfo and token introspection.  It may not need to be added to all of these, but I haven't specifically worked to see what the minimum set of things is here.

After saving the mapper, return to the client scope you created and go to the Scope tab.  Assign the OCIS roles you created here.

### Assigning the Client Scope in Keycloak

For each of the clients you created, open it and go to the "Client scopes" tab, and click the "Add client scope" button.  Add the scope you just created with a type of "Default".

### Installing OCIS

At this point, you can install OCIS according to [the directions in the OCIS documentation](https://doc.owncloud.com/ocis/4.0/depl-examples/bare-metal.html), but when you need to create `/etc/ocis/ocis.env`, a working version which just requires values to be filled in is available [here](./ocis.env).
