---
layout: default
title: How do I generate and install Domain Controller certificates?
collection: networkconfig
permalink: networkconfig/2b_domaincontrollers/
---
# How do I generate and install Domain Controller certificates?

To use smartcards and PIV credentials for network authentication, all Domain Controllers must have Domain Controller authentication certificates.

## Issuing Domain Controller certificates

Each U.S. Federal Government's Civilian agency has information-security policies that guide its decision-making about whether its Domain Controller's <!-- More than one DC? More than one certificate possible? -->certificate should be issued by the agency's local enterprise Certificate Authority (CA) or whether a Federal Public Key Infrastructure (FPKI)-managed and certified CA should issue it. Providing a common guide and recommendation is challenging, as each agency should follow its own policies; however, we do not recommend that agencies set up a local enterprise CA just to issue Domain Controller certificates. The best option is to collaborate with your agency's Chief Information Security Officer (CISO) (or the Information Security Office), who can give you definitive direction and has oversight for managing CAs and ensuring that security protections are in place.

## Generate and install Domain Controller certificate

_**Use Case:** We would like to use a local enterprise Microsoft CA to issue a Domain Controller certificate to the Domain Controller server. The server must have a certificate installed with the appropriate fields/values as a prerequisite to enabling authorized users with PIV credentials (i.e., PIV cards) to log into domain-connected devices._

  > **Note:** These procedures are accurate for using Microsoft 2012 Server, Standard Edition, for CA and Domain Controller servers (as of March 2017).

### Prerequisite

  * The server that hosts the CA must be on the domain

### Install CA role

  1. Log into the **CA server** as a member of the **Enterprise Administrators** group.
  2. Open the **Server Manager**.
  3. Click on **Manage**, and then click on **Add Roles and Features**.
  4. Proceed through the **Add Roles and Features Wizard** options. Choose the following:
     1. Server Roles: **_Active Directory Certificate Services_**
     2. AD CS Roles Services: **_Certification Authority_** 
  5. On the **Results** page, click on **Configure Active Directory Certificate Services on the destination server**.
  6. Proceed through the **AD CS Configuration** options. Choose the following, as necessary:
     1. Role Service: **_Certification Authority_** 
     2. Setup Type: **_Enterprise CA_** 
     3. CA Type: **_Root CA_**
     4. Private Key: **_Create a new private key_** 
     5. Cryptography: **_RSA#Microsoft Software Key Storage Provider, 2048 bit, SHA-256 6e_**
     6. CA Name: Use the recommended naming convention:
        **dc=[_AD suffix_], dc=[_AD domain_], cn=[_certification authority name_]** 
        (e.g., dc=_gov_, dc=_[AgencyName]_, cn=_[AgencyName]_ _NPE_ _CA1_) 
     7. Validity Period: **_6 years_** 
     8. Certificate Database: **_&lt;your preference&gt;_** 

## Configure CA template for Domain Controller

  > **Note:** Certificate templates are available on enterprise CAs.

  1. Log into the CA server as a member of the **Enterprise Administrators** group.
  2. Open the certificate template's **MMC snap-in** (i.e., **certtmpl.msc**). 
  3. Right-click on the **Domain Controller Authentication** template, and then click on **Duplicate Template**.
  4. Under the **Compatibility** tab, modify the **Compatibility Settings** for both the _CA_ and _certificate recipients_ to the highest version compatibility as possible (e.g., **Windows Server 2012 R2** or **Windows 7 2008 R2**).
  5. Under the **General** tab, we recommend the following settings:
     1. Template Name:  rename to:  **_&lt;Your organization&gt; - Domain Controller Authentication_**.
     2. Validity Period:  **_3 years_**.
     3. Renewal Period:  **_6 weeks_**.
  6. Under the **Cryptography** tab, set the following values:
     1. Minimum Key Size:  **_2048_**.
     2. Request Hash:  **_SHA256_** (if possible).
  7. Open the **CA console** (i.e., **certsrv.msc**).
  8. In the **console tree**, click on the **_[CA's name]_**.
  9. In the **details** pane, double-click on **Certificate Templates**.
 10. In the **console tree**, right-click on **Certificate Templates** and click on **New**. Then, click on **Certificate Template To Issue**.
 11. Select and enable the **_certificate template_** that was created, and then click on **OK**

## Auto-enroll Domain Controller certificate using Group Policy Object (GPO)**

  1. Log into the **Domain Controller server** as a member of the **Enterprise Administrators** group.
  2. Open the **GPMC** (i.e. **gpmc.msc** ).
  3. Within the appropriate **GPO**, navigate to **_Computer Configuration\Policies\Windows Settings\Security Settings\Public Key Policies\**_
  4. Configure **Certificate Services Client – Auto-Enrollment** with the following options:
     1. Configuration Model: **_Enabled_**.
     2. For Renew Expired Certificates, Update Pending Certificates, Remove Revoked Certificates: **_Check_all checkboxes_**.
     3. Update Certificates That Use Certificate Templates: **_Check the checkbox_**.
  5. At the command line, you can now force the group policy to update via the command: **_gpupdate /force_** or wait for the group policy to update on its own.
  6. If successful, you will see a new Domain Controller certificate in the **_Certificate (Local Computer) -&gt; Personal -&gt; Certificates folder_** (e.g., **_open MMC.exe -&gt; File -&gt; Add/Remove Snap-in -&gt;Certificates -&gt;Computer account -&gt;Local computer)_**.
  
  > At the **Certificate Template** tab, you should see a certificate generated with the custom certificate template.
