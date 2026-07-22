# Security advice

## 1 Basic safety requirements
!!! tip ""
    - JumpServer needs to open ports 80 443 2222 to the outside world at least.
    - The operating system of the server where JumpServer is located should be upgraded to the latest one.
    - The software that JumpServer depends on should be upgraded to the latest version.
    - Please do not use weak passwords for dependent components such as servers, databases, and Redis. 
    - Turning off Firewalld and SELinux is not recommended.
    - Only open necessary ports. If necessary, please access JumpServer through VPN or SSLVPN.
    - If it must be opened to the outside world, you should deploy a web application firewall for security filtering.
    - Please deploy an SSL certificate to access JumpServer via HTTPS protocol.
    - JumpServer should set strong password rules in security to disable users from using weak passwords.
    - The JumpServer MFA authentication function should be turned on to avoid security issues caused by password leaks.

!!! warning "attention"
    - If you find any security issues with JumpServer, please give us feedback at ibuler@fit2cloud.com

## 2 Security configuration recommendations
!!! tip ""
    - [Summary of common high-risk commands in Linux](https://kb.fit2cloud.com/?p=173){:target="_blank"}
    - [Set an asset to only allow connections after logging into JumpServer through a certain IP](https://kb.fit2cloud.com/?p=199){:target="_blank"}
    - [JumpServer is accessed using the user's own SSL certificate](https://kb.fit2cloud.com/?p=152){:target="_blank"}
    - [JumpServer enhances user login security](https://kb.fit2cloud.com/?p=71){:target="_blank"}
    - [JumpServer login asset user switching](https://kb.fit2cloud.com/?p=65){:target="_blank"}
    - [JumpServer high-risk command restrictions](https://kb.fit2cloud.com/?p=63){:target="_blank"}
    - [Restrict source IP to log in to JumpServer bastion host](https://kb.fit2cloud.com/?p=43){:target="_blank"}
    - [Commonly used MFA tools for JumpServer](https://kb.fit2cloud.com/?p=6){:target="_blank"}
    - [JumpServer set session expiration time](https://kb.fit2cloud.com/?p=5){:target="_blank"}
