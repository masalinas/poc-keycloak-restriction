# Description
PoC Keycloak Restriction Extension for role and policy based flow modes

## Steps

- **STEP01**: compile the last Restrict Keycloak Client Extension

```
./keycloak-restrict-client-auth/mvnw clean install
```

- **STEP02**: deployed keycloak 25.0.2 with the new extension configured:

```
docker run \
-d \
--name consum-keycloak-restriction \
-p 8080:8080 \
-e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=password \
-v ./keycloak-restrict-client-auth/target/keycloak-restrict-client-auth.jar:/opt/keycloak/providers/keycloak-restrict-client-auth.jar \
quay.io/keycloak/keycloak:25.0.2 start-dev
```

- **STEP03**: Created a new realm

Created a new realm named **mock**

![Mock Realm](./images/mock-realm.png "Mock Realm")

- **STEP04**: Created a new client

 Created a new client named **mock** with the client authentication activated, inside the previous realms

![Mock Client](./images/mock-client.png "Mock Client")

- **STEP05**: Created two roles
 
 Created two roles inside the previous client, like this:

 - User role with name **user**. This is a generic role to be tested
 - Restricted Access role with name **restricted-access**. This is the default restriction role used by the Authenticator extension to control the access.

![Mock Roles](./images/mock-roles.png "Mock Roles")

- **STEP06**: Created two users
 
 Created two users and asign the previous roles to each one like this:

 - User with name **user** attached to the role **user** of the client **mock**
 - User with name **maintenance** attached to the role **restricted-access** of the client **mock**

![Mock Users](./images/mock-users.png "Mock Users")

 - **STEP07**: Created a restriction flow
 
 Create a new flow named **direct grant maintenance** cloned from **direct grant** default flow adding a new subflow at last, where implement the restriction task **Restric user authentication on clients** (supported by the extension) like this:

![Restriction Flow](./images/restriction-flow.png "Restriction Flow")

- **STEP08**: Configure Restric user authentication on clients task
 
Configure **Restric user authentication on clients** task like this. We must set the same alias as the default restriction role name previously configured as **restricted-access**

![Restrict User Authentication Configuration](./images/restrict-user-authentication-configuration.png "Restrict User Authentication Configuration")

- **STEP09**: Bind Restriction Flow to direct grant default flow
 
After save the **direct grant maintenance** flow bind this flow to default direct grant flow, like this:

![Direct Grant Default Flow](./images/direct-grant-default-flow.png "Direct Grant Default Flow")

- **STEP10**: end-to-end tests
 
From postman we will try to login using the maintenace account. The result will be success because this user has the **restricted-access** role asigned:

![End-to-End maintenance account login](./images/end-to-end-maintenance-login.png "End-to-End maintenance account login")

Now we will login from postman again but using the user account. The result will be an **access denied** error because the **restricted-access** role is not attached to this account:

![End-to-End user account login](./images/end-to-end-user-login.png "End-to-End user account login")

- **STEP11**: Bonus: restrict access to account profile

We can restrict also the access to account creating a new flow cloned from browser default flow like this

![Restriction Account Access](./images/restriction-access-count.png "Restriction Account Access")

We must bind this custom flow to Browser default flow and configure as previuslly the task **Restric user authentication on clients** 

Now if we open the from ui the account access for our realm lije this an use the maintenace the user will logged correctly, but is we use the user we will obtain a **access denied** error

```
http://localhost:8080/realms/mock/account/
```

## Links

- [Ofitial Restrict Keycloak Client Extension](https://github.com/sventorben/keycloak-restrict-client-auth)
