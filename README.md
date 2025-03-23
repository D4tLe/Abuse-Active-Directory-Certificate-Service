# Abuse-Active-Directory-Certificate-Service

This document focuses on understanding and exploiting misconfigurations in Active Directory Certificate Services (ADCS) that can lead to privilege escalation in a Windows domain. ADCS is used for certificate-based authentication, but weaknesses in its configuration can allow attackers to gain higher privileges, including domain administrator access.

## Lab environment
- Domain Controller
- Root Certificate Authority (Standalone)
- Subordinate Certificate Authority (Enterprise)
- Kali Linux machine

Details about lab installation and configuration [Domain Controller] and [Certificate Authority] 

## Tools
- Ceritpy

## ESC1

### Prerequisites:
- Manager approval is not required
- Security descriptors of CA allow low-privileged user to enroll certificate
- No signature required
- Security descriptors on certificate templates allow low-privileged to enroll certificate
- The certificate template defines an EKU that allows for domain authentication:
    - Client authentication
    - PKINIT Client Authentication
    - Smart Card Logon
    - Any Purpose ([ESC2](#ESC2))
    - No EKU
- Requester has the ability to specify subjectAltName (SAN) in the CSR
  
![alt text](/images/ESC1/Prerequisites/CA.png)

![alt text](/images/ESC1/Prerequisites/Template.png)

### Exploit

![alt text](/images/ESC1/Exploit/1.png)

![alt text](/images/ESC1/Exploit/2.png)

### Detections & Mitigations
- Monitor Certificate Enrollment Events
  
![alt text](/images/ESC1/Detections%20&%20Mitigations/CertEnroll.png)

- Monitor Certificate Authentication Events
  
![alt text](/images/ESC1/Detections%20&%20Mitigations/AuthenEvent.png)

- Harden Certificate Template Settings

|![alt text](/images/ESC1/Detections%20&%20Mitigations/Harden1.png)|![alt text](/images/ESC1/Detections%20&%20Mitigations/Harden2.png)|
|-|-|
- Enforce Strict User Mappings (more details in [here][Paper])

## ESC2

ESC2 is the same as ESC1, exclude EKUs is Any Purpose

## ESC3

### Prerequisites

In this scenerio, at least 2 certificate templates matching conditions below:

- Template 1:
  - Manager approval is not required
  - Security descriptors of CA allow low-privileged user to enroll certificate
  - No signature required
  - Security descriptors on certificate templates allow low-privileged to enroll certificate
  - The certificate template defines the Certificate Request Agent EKU
  
![alt-text](/images/ESC3/Prerequisites/ESC3-Condition1.png)

- Template 2:
  - Manager approval is not required
  - Security descriptors of CA allow low-privileged user to enroll certificate
  - No signature required
  - The template schema version 1 or is greater than 2 and specifies an Application Policy Issuance Requirement requiring the Certificate Request Agent EKU.
  - The certificate template defines an EKU that allows for domain authentication
  - Enrollment agent restrictions are not implemented on the CA
  
![alt text](/images/ESC3/Prerequisites/SchemaVersion.png)

|![alt text](/images/ESC3/Prerequisites/ESC3-Condition2.png)|![alt text](/images/ESC3/Prerequisites/ApplicationPolicy.png)|
|-|-|

![alt text](/images/ESC3/Prerequisites/User.png)

![alt text](/images/ESC3/Prerequisites/CA%20restriction.png)



### Exploit

![alt text](/images/ESC3/Exploit/1.png)

![alt text](/images/ESC3/Exploit/2.png)

![alt text](/images/ESC3/Exploit/3.png)


### Detections & Mitigations

- Monitor certificate authentication events (Same as [ESC1](#ESC1))
- Harden certificate template settings (Same as [ESC1](#ESC1))
- Harden CA settings (Same as [ESC1](#ESC1))



## ESC4

### Prerequisites

Any of the rights below included in the misconfigured certificate template:
- Owner
- WriteOwner
- WriteDacl
- WriteProperty
- FullControll

![alt text](/images/ESC4/Prerequisites/WriteProperty.png)

### Exploit

![alt text](/images/ESC4/Exploit/1.png)

![alt text](/images/ESC4/Exploit/ModificatedTemplate.png)

![alt text](/images/ESC4/Exploit/2.png)

![alt text](/images/ESC4/Exploit/3.png)

![alt text](/images/ESC4/Exploit/4.png)

### Prevention & Mitigation
- Monitor certificate enrollments
- Monitor certificate authentication events
- Monitor certificate template modifications
  
![alt text](/images/ESC4/Detections%20&%20Mitigations/Event4662.png)

- Harden certificate template settings

![alt text](/images/ESC4/Detections%20&%20Mitigations/HardenTemplate.png)

## ESC5

### Prerequisites

The PKI system’s security is at risk if an attacker with limited privileges gains control over any of these critical components. Potential risks include, but are not limited to:

- The AD computer object of the CA server
- The CA server’s RPC/DCOM server
- Any descendant AD object or container within the path CN=Public Key Services, CN=Services, CN=Configuration, DC=, DC= (including the Certificate Templates container, Certification Authorities container, NTAuthCertificates object, Enrollment Services container, etc.)

![alt text](/images/ESC5/Prerequisites/LocalAdminCA.png)

### Exploit

![alt text](/images/ESC5/Exploit/1.png)

![alt text](/images/ESC5/Exploit/2.png)

![alt text](/images/ESC5/Exploit/3.png)

![alt text](/images/ESC5/Exploit/4.png)

### Detections & Mitigations

- More details in [here][Paper]

## ESC6

### Prerequisites

- CA specifies the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag

![alt text](/images/ESC6/Prerequisites/EDIF_ATTRIBUTESUBJECTALTNAME.png)

### Exploit

![alt text](/images/ESC6/Exploit/1.png)

![alt text](/images/ESC6/Exploit/2.png)

### Detections & Mitigations

- Monitor certificate enrollments events (Same as [ESC1](#ESC1))
- Harden CA Settings

![alt text](/images/ESC6/Detections%20&%20Mitigations/HardenCA.png)

## ESC7

### Prerequisites

- User account has Manage CA access right

![alt text](/images/ESC7/Prerequisites/ManageCA.png)

### Exploit

![alt text](/images/ESC7/Exploit/1.png)

![alt text](/images/ESC7/Exploit/2.png)

![alt text](/images/ESC7/Exploit/3.png)

### Detections & Mitigations

- Miscellaneous (more details in [here][Paper])

## ESC8


### Prerequisites

- Security descriptors of CA allow low-privileged user to enroll certificate
- Request Disposition: Issue
- Web Enrollment: Enabled
- Web Enrollment Services base on HTTP

![alt text](/images/ESC8/Prerequisites/Condition.png)

![alt text](/images/ESC8/Prerequisites/HTTP.png)
### Exploit

![alt text](/images/ESC8/Exploit/1.png)

![alt text](/images/ESC8/Exploit/2.png)

![alt text](/images/ESC8/Exploit/3.png)

![alt text](/images/ESC8/Exploit/4.png)

### Detections & Mitigations

- Harden AD CS HTTP Endpoints (more details in [here][Paper])


## References

[1] <https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf>

[2] <https://www.rbtsec.com/blog/active-directory-certificate-services-adcs-esc5/>

[3] <https://www.rbtsec.com/blog/active-directory-certificate-attack-esc8-adcs-web-enrollment/>

[4] <https://github.com/ly4k/Certipy>

[5] <https://us.informatiweb-pro.net/system-admin/win-server/ws-2016-ad-cs-install-and-configure-a-root-ca-and-a-secondary-ca.html>

[6] <https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-3/>

[Domain Controller]: https://sec.vnpt.vn/2022/10/build-a-basic-active-directory-lab-for-penetration-testing/
[Certificate Authority]:https://us.informatiweb-pro.net/system-admin/win-server/ws-2016-ad-cs-install-and-configure-a-root-ca-and-a-secondary-ca.html

[Paper]:https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf
