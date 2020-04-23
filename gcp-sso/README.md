# How to set an external IDP with Cloud Identity for GCP

To authenticate user to log on GCP console, the Google Cloud Identity is the default setup. It's easy for anybody who has a Google account to login. However for business users, many of them already have their existing authentication systems, the Microsoft Active Directory(AD) is a well-known example. To fulfil the use case, Google Cloud Identity can also integrate 3rd-party solution as its' identity provider(IDP).

In the following article, I'm going to show how to setup an external IDP for Cloud Identity to authenticate users log on to GCP console. The whole demo will all build from the ground as you just opened for business. The external IDP I select here is Active Directory (AD) with Active Directory Federation Service (ADFS), you may be using other IDP solutions like Okta, Keycloak, etc.


![GCP and ADFS](https://cloud.google.com/solutions/images/federating-gcp-with-ad-signing-in-to-gcp-console.svg)

This article is inspired by the [Google Cloud hybrid cloud solution tutorial](https://cloud.google.com/solutions/federating-gcp-with-active-directory-configuring-single-sign-on), you may find more detail explanation there.

## Getting started
Before following the steps to setup the sample environment, please make sure you got all the prerequisites ready on hands.

The installation process will covers:
1. Install and configure an Active Directory (AD) server.
2. Install Google Cloud Directory Sync and configure it to sync AD identities to Google Cloud Identity.
3. Install and configure an Active Directory Federation Servicer(ADFS) server.
4. Configure Cloud Identity to use ADFS for user authentication.
5. Testing on the deployment.

### Prerequisites
- A domain name which can be resolved on internet.
- [Register](https://cloud.google.com/identity/signup/premium/welcome) the Google Cloud Identity using the domain name.
- Register Google Cloud Platform account for hosting the AD & ADFS.

### Install and configure an Active Directory (AD) server.
[Video]
- provision a window server 2016 instance on GCP console.
- RDP to the server
- install AD

### Install Google Cloud Directory Sync and configure it to sync AD identities to Google Cloud Identity.
[Video]
- create an google identity account for AD sync.
- create an AD account for AD sync.
- install and config Google Cloud Directory Sync.
- create a AD user and test sync.

### Install and configure an Active Directory Federation Servicer(ADFS) server.
[Video]
- install ADFS
- configure the server

### Configure Cloud Identity to use ADFS for user authentication.
[Video]
- configure Cloud Identity 

### Testing on the deployment.
[Video]
- Login to GCP console with an AD user account.


doing these steps in Auth0
- create a new **application** (with regular web app as type)
- download **PEM** file under advanced settings.
- SAML URL:
` https://dev-peuew6z0.auth0.com/samlp/0lWWXM8mtjQl3XgFL6H0i5AZAB1EJz9D`
- enable SAML addon in Addon page (fill the Callback URL with `https://console.cloud.google.com/`) and record all required information in the Usage page.


configure SSO in [Cloud Admin](https://admin.google.com/u/4/ac/security/sso?hl=en&c_hn=https://admin.google.com)



### AD installation
- set local ip
- 




### Most useful shortcuts

- `CTRL + O` to open files
- `CTRL + T` to open a new tab
- `CTRL + S` to save the current file or tab

### References
- [Auth0 as Identity Provider](https://auth0.com/docs/protocols/saml/saml-idp-generic#2-configure-auth0-as-idp)
- 