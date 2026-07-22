# Change log

v4.10.9
------------------------
September 24, 2025

!!! summary "Function optimization 🚀"
    - perf: Lion file management supports uploading multiple files at the same time

!!! success "Bug fix 🐛"
    - fix: Fix the problem of excessive memory usage in KoKo



v4.10.8
------------------------
September 18, 2025

!!! info "New features 🌱"
    - feat: Added global resource search function
    - feat: Asset authorization supports reverse account authorization
    - feat: Added Vietnamese (Tiếng Việt) language support
    - feat: KoKo supports connecting to Redis clusters
    - feat: KoKo supports SSL connections to SQL Server
    - feat: supports configuring all email templates
    - feat: supports displaying announcements in pop-up windows

!!! summary "Function optimization 🚀"
    - perf: Improve cloud synchronization to avoid releasing assets in the following situations [Enterprise Edition]:
        - Cloud account is invalid
        - No assets found under area
        - Region has been removed from the update task
    - perf: Improved KoKo direct connection to return the correct exit code after exiting
    - perf: Support automatically starting VNC client to connect to assets [Enterprise Edition]
    - perf: Improve RDP true color (24-bit) display

!!! success "Bug fix 🐛"
    - fix: Fixed the issue where the access key is still valid after the user expires
    - fix: Fixed an error when changing user login rules to approval [Enterprise Edition]
    - fix: Fixed an issue where inactive session tabs would be disconnected due to not automatically renewing when there are multiple session tabs.
    - fix: Fixed the issue where Chen’s complex SQL query results failed to be exported

v4.10.7
------------------------
September 4, 2025

!!! summary "Function optimization 🚀"
    - perf: encrypted storage of key fields in AccessKey table (migration completed)

!!! success "Bug fix 🐛"
    - fix: Fixed an issue where user MFA reset failed

v4.10.6
------------------------
August 29, 2025

!!! success "Bug fix 🐛"
    - fix: Solve the problem of all components going offline after upgrade
    - fix: Resolved an issue where domain users were unable to log in to Windows assets
    - fix: fixed the error of scheduled task to clear session logs
    - fix: Fixed the issue of Magnus becoming unresponsive when executing long SQL statements
    - fix: Fixed the issue where the Nec component would get stuck when connecting to the RealVNC server using password-only authentication.

v4.10.5
------------------------
August 22, 2025

!!! info "New features 🌱"
    - feat: Add reporting functions to support visual data analysis and export [Enterprise Edition]
    - feat: Improved command logging and filtering for greater accuracy
    - feat: Cloud synchronization supports ProxmoxVE [Enterprise Edition]
    - feat: Added character search in KoKo to speed up information lookup

!!! summary "Function optimization 🚀"
    - perf: Store user AccessKey in encrypted form for improved security
    - perf: allow OTP reuse for ease of use when SAFE_MODE is off
    - perf: Disable Passkey as MFA when SAFE_MODE is on to enhance security

v4.10.4
------------------------
July 16, 2025

!!! summary "Function optimization 🚀" 
    
    - perf: increased recording file size in session recording
    - perf: endpoint rules now support matching by host domain name
    - perf: Added fuzzy search support for Elasticsearch command logging
    - perf: Optimized the operation log of downloading FTP log files
    - perf: Optimized the operation log for viewing recordings
    - perf: Improved user asset session details page and backend logic
    - perf: Added work order operation audit [Enterprise Edition]
    - perf: Added support for enabling or disabling SQL Server 2008 TLS encryption [Enterprise Edition]
    - perf: Cloud synchronization task now supports switching to automatically update host information [Enterprise Edition]

v4.10.3
------------------------
July 1, 2025

!!! success "Bug fix 🐛"
    - fix: Fixed an issue where the command record count would show 0 after configuring Elasticsearch for the command store.
    - fix: Fix "Ctrl + C" key combination to function correctly when connecting to Kubernetes.
    - fix: Fixed "remember password" functionality when connecting to an asset.
    - fix: Fixed an issue with KoKo sessions showing WebSocket disconnect warnings.

v4.10.2
------------------------
June 20, 2025

!!! info "New features 🌱"
    - feat: Support users to set personal language preferences
    - feat: Added connection support for MongoDB database in Magnus [Enterprise Edition]
    - feat: Supports automatic change of account password after successfully logging in to the asset [Enterprise Edition]
    - feat: Cloud synchronization now supports SmartX cloud platform [Enterprise Edition]
    - feat: SSO single sign-on now supports MFA [Enterprise Edition]
    - feat: The Chrome RemoteApp interface can now automatically switch display languages based on the current user's language
    - feat: Added support for periodic cleanup of expired connection tokens and temporary tokens
    - feat: Chat AI supports smart replies and command insertion based on character session (SSH, Telnet) context
    - feat: supports batch deletion of weak password sets
    - feat: The number of Celery Workers can be configured through CELERY_WORKER_COUNT
    - feat: When safe mode SAFE_MODE=true is enabled, shortcut commands in Adhoc will hide the account name prompt

!!! success "Bug fix 🐛"
    - fix: Fix the problem of remote application publishing failure
    - fix: Fixed the problem of abnormal nesting display of asset type tree

v4.10.1
------------------------
May 19, 2025

!!! success "Bug fix 🐛"
    - fix: Fix the problem of client download failure
    - fix: Fix the problem of mouse scrolling when Web CLI connects to Linux
    - fix: Fix the problem of abnormal activation of watermark in community version

v4.10.0
------------------------
May 15, 2025

!!! success "Major update ⚡️" 

    - feat: New Privileged Account Management (PAM): Enhanced security and flexibility of account management
    - feat: Face recognition authentication: providing users with a safer and more convenient authentication method [Enterprise Edition]
    - feat: Multi-language support: including English, Chinese (Simplified), Chinese (Traditional), Japanese, Portuguese (Brazil), Spanish, Russian and Korean

!!! info "New features 🌱"
    - feat: Added support for network device and directory service integration
    - feat: Added custom watermark display support
    - feat: Enabled Passkey as a multi-factor authentication (MFA) method
    - feat: Cloud synchronization supports Alibaba Cloud RDS [Enterprise Edition]
    - feat: supports drag-and-drop reordering of columns in the table
    - feat: Supports the file transfer and Chinese character copy functions of connecting the Web GUI to virtual applications (Linux application publishing) [Enterprise Edition]
    - feat: Added support for connecting to virtual applications (Linux application publishing) via VNC client [Enterprise Edition]
    - perf: Added separate settings for default expiry days for newly created users and authorizations
    
!!! summary "Function optimization 🚀" 
    
    - perf: Added support for collecting CPU and GPU model information from assets
    - perf: Users logging in with Passkey no longer need to complete MFA again
    - perf: Show detailed error message when asset or account connection fails
    - perf: Optimize the logic that the city's login alert will not be triggered if the user has logged in from that city within the past seven days
    - perf: option to manually set weak passwords for users

!!! success "Bug fix 🐛"
    - fix: Fixed asset session reconnect failure issue.
