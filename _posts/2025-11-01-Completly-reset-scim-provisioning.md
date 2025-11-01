---
title: Completely Resetting SCIM Provisioning for an Application
date: 2025-11-01 20:00 +0100
categories: [Microsoft, Entra]
tags: [Microsoft, Entra, SCIM]
---

When working with SCIM provisioning in Entra ID (formerly Azure AD), you might encounter sync issues that persist even after retries or mapping fixes.  
I recently ran into such a case where a SCIM-provisioned user was mapped to the wrong user account.  
After updating the mapping, restarting provisioning multiple times, and even removing and re-adding all users, the sync still kept linking to the wrong account.

At that point, there were two options:
1. Delete the application and start from scratch  
2. Figure out how to completely reset the provisioning

Since the app was already in use for single sign-on and I wanted to understand how to solve the issue, I went with option 2.

After digging a bit longer on the internet, I finally found a method that worked.

---

### How to Fully Reset SCIM Provisioning

If you need to completely reset SCIM provisioning for an application (for example, to fix incorrect user mappings), follow these steps.

#### 1. Open Graph Explorer
Go to [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer)  
Log in with a **Global Admin** account and make sure the correct tenant is selected.

![Desktop View](/assets/img/posts/2025-11-01-Completly-reset-scim-provisioning/ms-tenant.png){: width="500"}

#### 2. Grant the required permissions
Click your profile and grant the following permissions:
```
Synchronization.ReadWrite.All
Application.Read.All
Directory.Read.All
User.Read
```

#### 3. Create the POST request
Once you’ve granted the permissions, create a **POST** request to the following URL, replacing `APPLICATIONID` and `JOBID` with the appropriate values for your application (the service principal ID and the provisioning job ID):
```
https://graph.microsoft.com/beta/servicePrincipals/APPLICATIONID/synchronization/jobs/JOBID/restart
```

#### 4. Use the reset body
In my case, the request body looked like this, though [other reset scopes](https://learn.microsoft.com/en-us/graph/api/resources/synchronization-synchronizationjobrestartcriteria?view=graph-rest-1.0) are also available.
```
{
  "criteria": {
    "resetScope": "Full"
  }
}
```

A **Full** reset clears all synchronization-related state, including caches, watermarks, quarantines, and pending deletions. It effectively forces the service to start a complete reprovisioning cycle for all users and objects.

![Desktop View](/assets/img/posts/2025-11-01-Completly-reset-scim-provisioning/post-request.png){: width="1000"}

---

### Conclusion
This approach saved me from having to delete and recreate the entire application setup.  
It’s a quick and effective way to recover from broken or stale SCIM mappings without losing your SSO configuration.

---

**Further Reading**
- [Microsoft Graph – Synchronization Job Restart Criteria](https://learn.microsoft.com/en-us/graph/api/resources/synchronization-synchronizationjobrestartcriteria?view=graph-rest-1.0)
- [Microsoft Entra SCIM Provisioning Documentation](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/)

