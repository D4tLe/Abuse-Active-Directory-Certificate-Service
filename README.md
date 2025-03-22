# Abuse-Active-Directory-Certificate-Service

This document focuses on understanding and exploiting misconfigurations in Active Directory Certificate Services (ADCS) that can lead to privilege escalation in a Windows domain. ADCS is used for certificate-based authentication, but weaknesses in its configuration can allow attackers to gain higher privileges, including domain administrator access.

## Lab environment
- Domain Controller
- Root Certificate Authority (Standalone)
- Subordinate Certificate Authority (Enterprise)
- Kali Linux machine

Details about lab installation and configuration [Domain Controller] and [Certificate Authority] 

[Domain Controller]: https://sec.vnpt.vn/2022/10/build-a-basic-active-directory-lab-for-penetration-testing/
[Certificate Authority]:https://us.informatiweb-pro.net/system-admin/win-server/ws-2016-ad-cs-install-and-configure-a-root-ca-and-a-secondary-ca.html

## Tools
- Ceritpy

## ESC1

### Prerequisites:
- Manager approval no required
- Security descriptors of CA allow low-privileged user to enroll certificate
- No signature required
- Security descriptors on certificate templates allow low-privileged to enroll certificate
- EKUs facilitate authentication:
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

### Preventions & Mitigations
- Monitor Certificate Enrollment Events
  
![alt text](/images/ESC1/Detections%20&%20Mitigations/CertEnroll.png)

- Monitor Certificate Authentication Events
  
![alt text](/images/ESC1/Detections%20&%20Mitigations/AuthenEvent.png)

- Harden Certificate Template Settings

|![alt text](/images/ESC1/Detections%20&%20Mitigations/Harden1.png)|![alt text](/images/ESC1/Detections%20&%20Mitigations/Harden2.png)|
|-|-|
- Enforce Strict User Mappings

## ESC2

ESC2 is the same as ESC1, exclude EKUs is Any Purpose