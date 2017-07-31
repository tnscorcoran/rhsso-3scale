Prerequisites
==================================================================================================
Access to 3scale Account  
Access to Red Hat SSO On Openshift  
Your AWS box IP 52.15.120.30  

1 - Setup your 3scale Account
==================================================================================================
Go to API - Integration and click *edit integration settings*  
![Edit integration settings](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/01%20edit%20integration%20settings.png)
  
  
For GATEWAY, *choose APICast Self Managed*. For AUTHENTICATION choose *Oauth 2.0*. Then click Update Service  
![](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/01-2-edit-integration-settings-details.png)  
  
  
![](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/02%20Add%20the%20base%20URL%20of%20your%20API.png)  
Click *add the base URL of your API and save the Configuration*  
Keep *Private Base URL* as it is
Set your *Staging Public Base URL* and *Production Public Base URL* both to be http://apicast-rshsso.52.15.120.30.xip.io
Update the Staging Envirnonment
[3 Set Public Base URLs.png]
Set your credentials location to Headers:
[3-update-credentials-location-headers.png]
  
Go to API -> Integration. Click Promote v.<x> to Production
[3-promote-to-production.png]

Open the only Application (or choose one in your Oauth API if you have more)
[4-get-set your-3scale application-details]
Add Random Key
Set your Redirect URL to be https://www.getpostman.com/oauth2/callback
[5-get-your-3scale application-details.png]
Copy your API Credentials for later. Mine are
63e3f8c9
5d102db67c1723cdfa966d831a390101
https://www.getpostman.com/oauth2/callback


2 Initial Setup Red Hat Single Sign On
==================================================================================================
Login to Master
[6-add-realm-to-rh-sso.png]
My new realm is tom-oauth-1
Relax SSL only requirement (for a POC, never in production). As shown under the Login Tab - set it to none and Save.
[6-relax-ssl-requirement.png]
[9-set-expiration-and-count-999.png]
Save it. Your Initial Access Token will appear. Copy it, you won;t be able to retrieve it again.
eyJhbGciOiJSUzI1NiJ9.eyJqdGkiOiIzZmYyMjk1ZS1mOTBiLTQwZjQtYTRhNC0yN2IxYzJhYWI0YzQiLCJleHAiOjE1ODc0NTYxNDcsIm5iZiI6MCwiaWF0IjoxNTAxMTQyNTQ3LCJpc3MiOiJodHRwOi8vc3NvLXJoc3NvLjUyLjE1LjEyMC4zMC54aXAuaW8vYXV0aC9yZWFsbXMvdG9tLW9hdXRoLTEiLCJhdWQiOiJodHRwOi8vc3NvLXJoc3NvLjUyLjE1LjEyMC4zMC54aXAuaW8vYXV0aC9yZWFsbXMvdG9tLW9hdXRoLTEiLCJ0eXAiOiJJbml0aWFsQWNjZXNzVG9rZW4ifQ.hV5wKhCK6Db3QzB25kXsNtiLmQxeFwmcN3jmjuMRe_EArJ7D7T3RWJyBpJNBnVJOmW43ii7lQjAQ1YjnUqOVURH09uR6KB8iq5ZiSmi6A2ZZkokKxuQyVt_eKuOI4yI8f9aNp8HOjwSoitFUf0RRhtCdx66Axm2psKbjFMJEP5ponLsKsM2_HBnWb2vYbdGxvBtA0tRNDzZcqzTqg2pF0h8yxuOCAMtw5KVo_hunZKx1m0PBFMbivQzUJi4zeyG1pEHrogc5HVzjM54-K6HQbX-7xK902kgHdspjVUvCgAaE3mPGIc0XB4oOJcjD7loKZevmkjSgbszc1op_1S5Cog

Test your realm hitting it in a browser
http://sso-rhsso.52.15.120.30.xip.io/auth/realms/tom-oauth-1


Create a user on RH SSO cal-client
[10-Add-User.png]
Enter something like the following and Save:
[11-add-user-initial-details.png]
On the Credentials tab, add New Password and Password Confirmation and reset
[12-add-credentials.png]


Use Postman to create client on Red Hat Single Sign On
==================================================================================================
Import the postman JSON spec that accompanies this (RHSSO_3scaleGist.postman_collection.json)
Open RH SSO Add Client request add client to Red Hat SSO (Client credentials on RHSSO need to be the same as Application Credentials set on 3scale above)
Set the realm name in the address and the Bearer token to your Initial Access Token retrieved above.
[12-postman-add-client-1.png]
Save it. 
Switch to the Body tab. Enter the clientId, secret and URL you set in 3scale above (Redirect URL is called redirectUris here)
[14-Add-Client-Body.png]
Save and Send it. You should get back some JSON and 201 created HTTP status.

2 Check and Update your Client in Red Hat SSO
==================================================================================================
In Red Hat SSO, click on the Clients view on the Left hand side. Select your new client (63e3f8c9)
Assuming you want to give the user ability to grant access to the Application to access their data (we do), click Consent Required. Save
[14-update-client-in-rhsso.png]


Setup your API Gateway on Openshift
==================================================================================================
Login to Openshift and make the following commands
oc login 
	(use credentials developer/developer)
oc new-project "3scalegateway-tom-oauth-1" --display-name="3scalegateway-tom-oauth-1" --description="3scalegateway-tom-oauth-1"

oc secret new-basicauth apicast-configuration-url-secret --password=https://9800f60ff34d25e7bc5b22c25287e94da130f6fb0fe9b77c05bd6a6d993b0614@tom-oauth-1-admin.3scale.net

oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.0.0.GA-redhat-2/apicast-gateway/apicast.yml

Your gateway will deploy in a couple of minutes.
**TIP** Scale it down to 1 Pod - that way if you want to look up logs, they can only be in the remaining Pod.

Open web Console https://ec2-52-15-120-30.us-east-2.compute.amazonaws.com:8443 and open 3scalegateway-tom-oauth-1
Go to Applications -> Deployments -> apicast -> Environment 
Add this ENV variable: RHSSO_ENDPOINT and set it to: http://sso-rhsso.52.15.120.30.xip.io/auth/realms/tom-oauth-1
Save

On Openshift web Console ->  Overview -> Create Route. 
Name: 		apicast-rshsso-route
Hostname:	apicast-rshsso.52.15.120.30.xip.io
Leave the rest of the defaults and click Create
[15-create-route.png]


In Postman test Oauth flow and API with token
==================================================================================================
Open Echo Hello. Set your GET URL to be http://apicast-rshsso.52.15.120.30.xip.io/hello
[16-echo-hello-request.png]

Choose Type -> Oauth 2.0
[17-type-Oauth2.0.png]

Set these parameters and 
Click Get New Access Token orange button
Auth URL:			http://apicast-rshsso.52.15.120.30.xip.io/authorize
Access Token URL:	http://apicast-rshsso.52.15.120.30.xip.io/oauth/token
Client ID 			63e3f8c9
Client Secret 		5d102db67c1723cdfa966d831a390101
[18-Postman-Request-token.png]

Login as the user you created in RH SSO (cal-client). (styling is always missing so ignore for now)
[19-login-as-your-user.png]
After authentication, you'll be prompted to create a new Password. Do this.

A Token (RHSSO Get Token) will be returned to Postman. Click it then click Use Token.
Click Save then click Send.
[20-use-token-save-send.png]

Go to your 3scale Analytics. Your counts should increment with each call.

Insert a random characted into the JWT and Use Token and repeat. Authorization should fail.


NICE TO DEMO: Test your JWT on JWT.io
==================================================================================================
Go to JWT.io
Paste into the Encoded box on the left the JWT that got inserted into the Bearer token Authorization Header after you selected Use Token.
On the right, you'll see some payload data representing the user and client.

Go to Red Hat SSO -> Realm Settings -> Keys			
Copy the Public Key 
[21-get-public-key.png]
(insert JWT here - between -----BEGIN PUBLIC KEY----- and -----END PUBLIC KEY-----) 	
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxlgJ4D/Wo5gPAzoNiihZ3mQsMLELx1yhZUCsvmeP2DJbIlCHSOGdVz8m/XVsxeA1B90mYE79i/UD1ZVUYEZgFZNB5b5AQE2veQfnnrOLqCkRbtOEMRoLuQIOLt5/Hf/xeGv3iuL/PvZs/pVDTa1Dkr0fFLjNAcHeDH9p3XR5JQuvokv6nnh8QXnOYPRKSE4f5pvlN1HfKK0AIrHihBegOHYu7MOWF06FaYvTdQozjXEALNebqKxBVbF7+aFz43GCRVWqcYcY88dlfHhuCd/7kaXHr6Ib6bTZYvktOJSHw05GwuVwhS23b8t0UoLal4Xvq3Vv2JgwV66Vz4jAbyKG1wIDAQAB
-----END PUBLIC KEY-----
			
Paste this into the Public Key or Certificate box on the right and the red Invalid Signature banner should turn to a blue Signature verified one.
Alter either the payload or the public key and signature should fail. This simulates the signature validion that hapeens on the gateway.


Setup LDAP (if you don't have set one up using Open LDAP for example)
Anand and Kavitha, THESE CREDENTIALS WILL BE REMOVED WHEN GIST IS TESTED
==================================================================================================
Go to: User Federation -> Add Provider -> ldap
[22-choose-ldap.png]

Enter the following, testing connection and authentication as you go then Save:
--------------------
Console Display Name:	open-ldap
Priority				1
Edit Mode				Read Only
Vendor					Active Directory
UUID LDAP attribute		entryUUID
User Object Classes		shadowaccount, posixaccount
Connection URL			ldap://ec2-107-22-51-90.compute-1.amazonaws.com:389
	***test it
Users DN				ou=people,dc=example,dc=com
Authentication Type		Simple
Bind DN					cn=admin,dc=example,dc=com
Bind Credential			admin123

[22-sample-ldap-settings.png]

Retest auth flow and API with token In Postman - this time using credentials stored on LDAP
Anand and Kavitha, use adam/adam123



SSO into Dev Portal (based on https://support.3scale.net/docs/developer-portal/authentication#rhsso)
==================================================================================================
In 3scale, go to Settings -> Developer Portal -> SSO Integrations.
[23-settings-dev-portal-sso.png]
Click on Red Hat Single Sign On. Enter the following (or your desired values)
Client			3scale-dev-portal
Client secret	d5252450-76f5-4ffe-90a2-e1c6dbf5ea9b
Realm			http://sso-rhsso.52.15.120.30.xip.io/auth/realms/tom-oauth-1
[24-3scale-dev-portal-credentials.png]
While we're in 3scale we're going to remove the secret access key for easier access (for illustration only - we would normally keep it there until our Dev Portal is ready to go live)
In 3scale go to Settings -> Developer Portal. Delete the Developer Portal Access Code and Update Account.
[25-remove-portal-access-key.png]

Next we need to add a client to your Realm in Red Hat SSO that aligns with your Dev Portal Oauth client to 3scale.
Open Postman and repeat the steps on line 66 above, replacing the data elements in the request JSON body with these (or your equivalents):
clientId: 		3scale-dev-portal
secret:			d5252450-76f5-4ffe-90a2-e1c6dbf5ea9b
redirectUris:	https://tom-oauth-1.3scale.net
(note the redirectUris entry is the same as your 3scale admin portal url without the '-admin')
Click Send.

You'll need to make a couple of mods to the client generated by your Postman call.
Root URL:				https://tom-oauth-1.3scale.net 	(your dev portal URL) 	
Valid Redirect URIs:	https://tom-oauth-1.3scale.net*	(your dev portal URL with * appended)
Web Origins:			<empty>							(delete what's there)
[26-modify-3scale-portal-client-on-rhsso.png]

Test it out. Go to Developer Portal -> (in a new Incognito Window) Visit Developer Portal
[27-dev-portal-visit-dev-portal.png]










********************************************************************************************************	
******** SSO into Dev Portal
https://tom-demo-admin.3scale.net
Settings->Developer Portal->RH SSO
	Client			3scale-dev-portal
	Client secret	d5252450-76f5-4ffe-90a2-e1c6dbf5ea9b
	Realm			http://sso-rhsso.52.15.120.30.xip.io/auth/realms/tudor-realm
	
Initial Access Token
	eyJhbGciOiJSUzI1NiJ9.eyJqdGkiOiJjYmNkOThlMC0yNzAzLTQ4ZTgtODc2Mi1hZTM2NzU1NmNjOTMiLCJleHAiOjE1MDIwMDg3NDQsIm5iZiI6MCwiaWF0IjoxNTAxMjMxMTQ0LCJpc3MiOiJodHRwOi8vc3NvLXJoc3NvLjUyLjE1LjEyMC4zMC54aXAuaW8vYXV0aC9yZWFsbXMvdG9tLW9hdXRoLTEiLCJhdWQiOiJodHRwOi8vc3NvLXJoc3NvLjUyLjE1LjEyMC4zMC54aXAuaW8vYXV0aC9yZWFsbXMvdG9tLW9hdXRoLTEiLCJ0eXAiOiJJbml0aWFsQWNjZXNzVG9rZW4ifQ.GNOBauH7xtWikZ4hOTSr0rHjRWtQ2oDitvcYdePoMc5thnVtcE8r_YkGJPtjK_a-GN-mnEydHZdNgJduZWy0-LCdZ40Spd3D0Nw7K_ZFCS3QlkcEmC8KL0LylqxMzCXANEEJHkT-UCzoqGyRQHiizWsKMRyBhXBoZdVl4iwecb9M2mSELx0GLxeUMW4--WVR7Nuf4WUlggDi5XJU7XqHMzoPzPYCpIvOuHTLL689j5lK6Emi6P3n4-Ipd4tQALC8Lqm9YXhcMwu55wcBGt6ix9JDiKpex5kDvXDGi9Is0DQanPL1PLkzBzDt2wcpzh5E_nRmoa0xVkbg0zZEyIAnTQ

	 
Go to the Create Client in Postman (like line 38 above) - create a client with the client id and secret and then add these
Root URL				 https://3scale.amp.52.15.150.46.nip.io
Valid Redirect URIs		 https://3scale.amp.52.15.150.46.nip.io*
	
In RHSSO, add Mappers to org_name and email like here: https://support.3scale.net/docs/developer-portal/authentication#rhsso 	
	
Delete any Accounts besides Demo-Org.

INCOGNITO WINDOW 
- https://tom-demo.3scale.net
- Sign In 
- Authenticate with Red Hat SSO
 	adam or cal-client
- Add Custom signup fields
- close window

3scale Admin
- Activate Request

new	INCOGNITO WINDOW 
- https://tnscorcoran.3scale.net
- Sign In 
- Authenticate with Red Hat SSO
			 
			