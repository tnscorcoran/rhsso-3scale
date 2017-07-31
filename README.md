Prerequisites
==================================================================================================
Access to a 3scale Account  
Access to a Red Hat SSO installation  
An API client. We assume it to be Postman client for Chrome  
Your RHEL box IP, refered to *rhel-box-ip*  
  

Note: We have inserted whitespace in some URLs as they contain placeholders and are not valid URLs as they appear. Insert the values resolving to the URL fragments and remove whitespace.  

1 - Setup your 3scale Account
==================================================================================================
Go to API - Integration and click *edit integration settings*  
![Edit integration settings](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/01%20edit%20integration%20settings.png)
  
  
For GATEWAY, *choose APICast Self Managed*. For AUTHENTICATION choose *Oauth 2.0*. Then click Update Service  
![edit-integration-settings-details](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/01-2-edit-integration-settings-details.png)  
  
  
Click *add the base URL of your API and save the Configuration*  
![add-base-url](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/02%20Add%20the%20base%20URL%20of%20your%20API.png)  
Keep *Private Base URL* as it is (or enter your desired API Back end URL).  
Set your *Staging Public Base URL* and *Production Public Base URL* both to be http:// apicast-rshsso.*rhel-box-ip*.xip.io  
Click *Update the Staging Envirnonment*
![public-base-urls](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/03%20Set%20Public%20Base%20URLs.png)
Set your credentials location to Headers:  
![update-credentials-location-headers](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/03-update-credentials-location-headers.png)
  
Go to API -> Integration. Click Promote v._x_ to Production
![promote-to-production](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/03-promote-to-prod.png)
  
Choose Developers - Account Developer - 1 Application  
![developer-application](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/04-open-developer-appl.png)

Open your only Application (or choose one in your Oauth API if you have more)
![set-application-details](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/04-set%20your-3scale%20app-details.png)
Add Random Key
Set your Redirect URL to be https://www.getpostman.com/oauth2/callback
![application-details](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/05-get-your-3scale%20application-details.png)
Copy your API Credentials for later. Refered to as  
*client-id*  
*client-secret*  
*redirect-url*  
  
  
  
Next, get your 3scale Access token (used to create gateway). Go to Gear sign - Personal Settings - Tokens  
![05-threescale-accesstoken-1.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/05-threescale-accesstoken-1.png)
  
Add Access token  
![05-threescale-accesstoken-1.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/05-threescale-accesstoken-2.png)
  
Name it, give it read/write access to all scopes and Create Access token. We'll refer to this your *3scale-access-token*   
![05-threescale-accesstoken-1.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/05-threescale-accesstoken-3.png)
  

2 - Initial Setup of Red Hat Single Sign On
==================================================================================================
Login to Master  
Create a Realm for 3scale Oauth. Name it. We'll refer to it as **_3scale-oauth-realm_** 
![6-add-realm-to-rh-sso.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/06-add-realm-to-rhsso.png)  
Test your realm hitting it in a browser  
http:// sso-rhsso.*rhel-box-ip*.xip.io/auth/realms/**_3scale-oauth-realm_**  
  
  
Relax SSL only requirement (for a POC, never in production). As shown under the Login Tab - set it to none and Save.
![6-relax-ssl-requirement.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/07-relax-ssl-requirement.png)
  
  
Choose Initial Access Token (this is a token we will use for a Red Hat SSO API call shortly)  
![initial-access-token](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/08-create-initial-access-tkn.png)
  

Set your expiration and count to be non zero values. 
![9-set-expiration-and-count-999.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/09-set-expiration-and-count-999.png)  
Save it. Your Initial Access Token will appear. Copy it, you won't be able to retrieve it again. We'll refer to it as your *initial-access-token*
  
  
Create a user on RH SSO. For simplicity I'll refer to as *rh-sso-user-id*.
![10-Add-User.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/10-Add-User.png)  

Enter your equivalents to the following and Save:
![11-add-user-initial-details.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/11-add-user-initial-details.png)  

On the Credentials tab, add New Password and Password Confirmation and reset. I'll refer to it as rh-sso-user-password
![12-add-credentials.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/12-add-credentials.png)
  
  

3 - Use Postman to create client on Red Hat Single Sign On
==================================================================================================
In Postman, import the postman JSON *RHSSO_3scaleGist.postman_collection.json* in this repo.
Open RH SSO Add Client request add client to Red Hat SSO (Client credentials on RHSSO need to be the same as Application Credentials set on 3scale above)
Set the realm name in the address (to reflect *rhel-box-ip*) and the Bearer token to your *initial-access-token* retrieved above.
![12-postman-add-cli-1.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/13-postman-add-cli-1.png)
Save it. 
Switch to the Body tab. Enter the *client-id*, *client-secret* and *redirect-url* you set in 3scale above (Redirect URL is called redirectUris here)  
![14-Add-Client-Body.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/14-Add-Cli-Bdy.png)
  
Save and Send it. You should get back some JSON and 201 created HTTP status.  
  


4 - Check and Update your Client in Red Hat SSO  
==================================================================================================
In Red Hat SSO, click on the Clients view on the Left hand side. Select your new client (*client-id*)  
Assuming you want to give the user ability to grant access to the Application to access their data (we do), click Consent Required. Save  
![14-update-client-in-rhsso.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/14-update-client-in-rh-sso.png)  
  
  
5 - Setup your API Gateway on Openshift  
==================================================================================================
Login to Openshift and make the following commands
oc login 
	(default credentials are developer/developer)  
Create your project, e.g. with something like these values:  
oc new-project "3scalegateway-**_3scale-oauth-realm_**" --display-name="3scalegateway-**_3scale-oauth-realm_**" --description="3scalegateway-**_3scale-oauth-realm_**"  
  
oc secret new-basicauth apicast-configuration-url-secret --password=https:// *3scale-access-token* @ **_3scale-oauth-realm_**-admin.3scale.net (remove whitespace)  
  
oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.0.0.GA-redhat-2/apicast-gateway/apicast.yml  
  
Your gateway will deploy in a couple of minutes.  
**TIP** Scale it down to 1 Pod - that way if you want to look up logs, they can only be in the remaining Pod.  
  
Open web Console https:// *openshift-host* :8443 and open 3scalegateway-**_3scale-oauth-realm_**  
Go to Applications -> Deployments -> apicast -> Environment  
Add this ENV variable: RHSSO_ENDPOINT and set it to: http:// sso-rhsso.*rhel-box-ip*.xip.io/auth/realms/**_3scale-oauth-realm_**  
Save  
  
On Openshift web Console ->  Overview -> Create Route. 
Name: 		apicast-rshsso-route  
Hostname:	apicast-rshsso.*rhel-box-ip*.xip.io  
Leave the rest of the defaults and click Create  
![15-create-route.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/15-create-rout.png)  
  
  
6 - In Postman test Oauth flow and API with token  
==================================================================================================  
Open Echo Hello. Set your GET URL to be http:// apicast-rshsso.*rhel-box-ip*.xip.io/hello  
![16-echo-hello-request.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/16-echo-hello-requestt.png)  

Choose Type -> Oauth 2.0
![17-type-Oauth2.0.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/17-type-Oauth20.png)  
  
Set these parameters and  
Click Get New Access Token orange button  
Auth URL:			http:// apicast-rshsso.*rhel-box-ip*.xip.io/authorize  
Access Token URL:	http:// apicast-rshsso.*rhel-box-ip*.xip.io/oauth/token  
Client ID 			*client-id*  
Client Secret 		*client-secret*  
![18-Postman-Request-token.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/18-Postman-Request-tkn.png)  
  
Login as the user you created in RH SSO (rh-sso-user-id/rh-sso-user-password). (styling is always missing so ignore for now)  
![19-login-as-your-user.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/19-login-as-your-user.png)  
After authentication, you'll be prompted to create a new Password. Do this.  
  
A Token (RHSSO Get Token) will be returned to Postman. Click it then click Use Token.  
Click Save then click Send. You should get a successful API Response.    
![20-use-token-save-send.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/20-use-token-savesend.png)  

Go to your 3scale Analytics. Your counts should increment with each call.
![3scale-analytics](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/21-3scale-analytics.png)  
  
  
Return to postman and insert a random characted into the JWT and Use Token and repeat. Authorization should fail.
  
  

7 - Test your JWT on https://jwt.io (optional)  
==================================================================================================  
Go to https://jwt.io  
Paste into the Encoded box on the left the JWT that got inserted into the Bearer token Authorization Header after you selected Use Token.  
On the right, you'll see some decoded payload data representing the user and client.  
  
Go to Red Hat SSO -> Realm Settings -> Keys  
Copy the Public Key  
![21-get-public-key.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/21-get-public-key.png)
Insert Public Key as follows:  
-----BEGIN PUBLIC KEY-----
**_public key goes here_**
-----END PUBLIC KEY-----
			
Back in jwt.io, paste this into the Public Key or Certificate box on the right and the red Invalid Signature banner should turn to a blue *Signature verified* one.
Alter either the payload or the public key and signature validation will fail. This simulates the signature validation that hapeens on the gateway.
  
  
8 - Setup LDAP (if you don't have one, setup a test one using Open LDAP)
==================================================================================================
Go to: User Federation -> Add Provider -> ldap
![22-choose-ldap.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/22-choose-ldap.png)

Enter the following (or your equivalents), testing connection and authentication as you go then Save:
--------------------
Console Display Name:	open-ldap  
Priority				1  
Edit Mode				Read Only  
Vendor					Active Directory  
UUID LDAP attribute		entryUUID  
User Object Classes		shadowaccount, posixaccount  
Connection URL			ldap://*ldap-url*:389  
	***test it  
Users DN				ou=people,dc=example,dc=com  
Authentication Type		Simple  
Bind DN					cn=admin,dc=example,dc=com  
Bind Credential			*ldap-password*  
  
![22-sample-ldap-settings.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/22-sample-ldap-settingz.png)  
  
Retest auth flow and API with token In Postman - this time using credentials stored on LDAP  
  
  
  
9 - SSO into Dev Portal (based on https://support.3scale.net/docs/developer-portal/authentication#rhsso)
==================================================================================================
In 3scale, go to Settings -> Developer Portal -> SSO Integrations.  
![23-settings-dev-portal-sso.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/23-settings-dev-portal-sso.png)  
Click on Red Hat Single Sign On. Enter the following (or your desired values)  
Client			3scale-dev-portal-client  
Client secret	3scale-dev-portal-client-secret  
Realm			http://sso-rhsso.*rhel-box-ip*.xip.io/auth/realms/**_3scale-oauth-realm_**  
![24-3scale-dev-portal-credentials.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/24-3scale-dev-portal-credentials.png)  
  
  
While we're in 3scale we're going to remove the secret access key for easier access (for illustration only - we would normally keep it there until  our Dev Portal is ready to go live)  
In 3scale go to Settings -> Developer Portal. Delete the Developer Portal Access Code and Update Account.   
![25-remove-portal-access-key.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/25-remove-portal-access-key.png)  
  
Next we need to add a client to your Realm in Red Hat SSO that aligns with your Dev Portal Oauth client to 3scale.  
Open Postman and repeat the steps on line 66 above, replacing the data elements in the request JSON body with these (or your equivalents):  
clientId: 		3scale-dev-portal-client  
secret:			3scale-dev-portal-client-secret  
redirectUris:	https://**_3scale-oauth-realm_**.3scale.net  
(note the redirectUris entry is the same as your 3scale admin portal url without the '-admin')  
Click Send.  
  
You'll need to make a couple of mods to the client generated by your Postman call.  
Root URL:				https://**_3scale-oauth-realm_**.3scale.net 	(your dev portal URL) 	
Valid Redirect URIs:	https://**_3scale-oauth-realm_**.3scale.net*	(your dev portal URL with * appended)  
Web Origins:			**empty**							(delete what's there)  
![26-modify-3scale-portal-client-on-rhsso.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/26-modify-3scale-portal-client-on-rhsso.png)  
  
Test it out. Go to Developer Portal -> (in a new Incognito Window) Visit Developer Portal  
![27-dev-portal-visit-dev-portal.png](https://raw.githubusercontent.com/tnscorcoran/rhsso-3scale/master/27-dev-portal-visit-dev-portal.png)  










********************************************************************************************************	
******** SSO into Dev Portal
https://tom-demo-admin.3scale.net
Settings->Developer Portal->RH SSO
	Client			3scale-dev-portal
	Client secret	d5252450-76f5-4ffe-90a2-e1c6dbf5ea9b
	Realm			http://sso-rhsso.*rhel-box-ip*.xip.io/auth/realms/tudor-realm
	
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
 	rh-sso-ldap-user-id or rh-sso-user-id
- Add Custom signup fields
- close window

3scale Admin
- Activate Request

new	INCOGNITO WINDOW 
- https://tnscorcoran.3scale.net
- Sign In 
- Authenticate with Red Hat SSO
			 
			