# 🛡️ Broken Access Control Checklist

<aside>
<img src="notion://custom_emoji/1522c6c7-7767-4cfe-857c-db41ff62c82c/2389044a-0c22-80c4-b114-007abf85da8b" alt="icon" width="40px" />

**Made by @Hunt3r_Rocky**

💬 DM me on Twitter or leave suggestions in comments.
</aside>

---

### 🔄 Horizontal Privilege Escalation

- **Parameter Tampering**:
    - Modify `userID`, `orderID`, etc., in URLs/APIs (e.g., `/api/orders?userID=123` → `userID=124`).
    - **Expected Result**: Access denied unless resource belongs to the user.

- Attempt to access another user's resources at the same privilege level  
  View/edit/delete another user's profile, messages, files

- **API Role Enforcement**:
    - Use a low-privilege token to access another user’s data via API (e.g., `GET /api/users/456`).
    - **Expected Result**: 403 Forbidden.

- **Impersonation Prevention**:
    - Test actions like transferring funds on behalf of another user.
    - **Expected Result**: Action fails unless explicitly authorized.

- Test GraphQL object ID manipulation (e.g., `user(id: 2)`)

- Interact with objects (carts, profiles) of other users via API

---

### 🔼 Vertical Privilege Escalation

- **Admin Endpoint Access**:
    - Access `/admin` or `/api/admin` as a regular user.
    - **Expected Result**: 403 Forbidden.

- **High-Privilege Actions**:
    - Attempt admin actions (e.g., user deletion) as a non-admin.
    - **Expected Result**: Action blocked.

- **Hardcoded Parameters**:
    - Add `isAdmin=true` to requests.
    - **Expected Result**: Server ignores unauthorized parameters.

---

### 🧩 IDOR

- **Direct Object References**:
    - Access resources via URLs like `/file/12345` without ownership.
    - **Expected Result**: Access denied.

- **Enumeration**:
    - Increment object IDs (e.g., `12345` → `12346`) to test unauthorized access.
    - **Expected Result**: Objects not returned unless owned.

- **API Data Filtering**:
    - Call `GET /api/files` to check if only user-owned files are listed.
    - **Expected Result**: No cross-user data leakage.

---

### 📁 File and Directory Access

- **Unrestricted Uploads**:
    - Upload malicious files (e.g., `.php`, `.exe`).
    - **Expected Result**: Blocked by file type/content validation.

- **Directory Traversal**:
    - Request `../../etc/passwd` in file paths.
    - **Expected Result**: Path traversal blocked.

- **Cross-User File Access**:
    - Access User A’s uploaded file as User B via direct URL.
    - **Expected Result**: Access denied.

---

### 🌐 URL/Query Parameter Manipulation

- **URL Bypass**:
    - Navigate to `/admin` as a non-admin user.
    - **Expected Result**: Redirect or denial.

- **Query Tampering**:
    - Modify `?isAdmin=false` → `true` in requests.
    - **Expected Result**: Server-side validation rejects changes.

---

### 🧬 API / GraphQL Access Control

- **Role Enforcement**:
    - Access role-restricted endpoints (e.g., `/api/admin`) with a non-admin token.
    - **Expected Result**: 403 Forbidden.

- **GraphQL Introspection**:
    - Run `__schema` queries to check for sensitive data exposure.
    - **Expected Result**: Introspection disabled or sanitized.

- **Mass Assignment**:
    - Send extra parameters (e.g., `"role": "admin"`) in user update requests.
    - **Expected Result**: Unauthorized fields ignored.

- Profile update → Try modifying `isStaff`, `org_id`, `permissions`

- **Access Control for CRUD Operations:**
    - Validate Create, Read, Update, and Delete (CRUD) permissions for all roles.
    - **Test:** Ensure restricted roles cannot perform unauthorized actions (e.g., Viewer modifying data).
  
  **Steps:**
  1. Log in as a Viewer.
  2. Attempt to:
      - Create a resource (e.g., `/api/createResource`).
      - Update a resource (e.g., `/api/updateResource`).
      - Delete a resource (e.g., `/api/deleteResource`)
  3. Log in as an Admin and repeat the actions.

- GraphQL: mutate fields not shown in UI (e.g., `verified:true`)

- Test old API versions (`/v1/`, `/v2/`) for weaker checks

---

### 🔁 Multi-Step Workflows & Multi-Tenant Access

- **State Skipping**:
    - Jump to the final step (e.g., payment confirmation) without completing prior steps.
    - **Expected Result**: Server validates workflow state; request denied.

---

### 🏢 SaaS-Specific Tests

- **Invite Leakage**:
    - Check if pending invites expose emails, full name or org details.
    - **Expected Result**: Data masked (e.g., `us***@example.com`).

- **User Enumeration via Invites:**
    - Test if sending invites to invalid or non-existent emails reveals user existence in other organizations.
    - **Expected Behavior:** Generic error messages should not confirm whether a user exists in the system.

- **Invite Tampering**:
    - Modify `org_id` in invitation links (e.g., `invite?org_id=100` → `101`).
    - **Expected Result**: Tampered invites rejected.

    - Switch `org_id`, `tenant_id` in URL/body to access other orgs  
    - Test if invite from Org A accepted while logged into Org B  
    - Try accessing data from Org A using a user of Org B  
    - Upload files in Org A → download in Org B

---

### 🔐 Account Takeover (ATO) via Invitation

- **Enforcement of User Acceptance for Invitations:**
    - Verify that users cannot be added to an organization without explicitly accepting the invite.

- **Email and Password Manipulation:**
    - Test if the inviter can change the email or reset the password of an invited user after adding them to the org.

  **Steps:**
  1. Add a user to your org without requiring invite acceptance.
  2. Attempt to modify their email or reset their password.
  - **Expected Behavior:** Email/password changes should require user consent or authentication.

---

### 🧨 Hijacking an Organization via Org ID Manipulation

- **Org ID Tampering in Invitation Links:**
    - Analyze the invitation link for predictable or sequential `org_id` or `invite_id`.
    - **Expected Behavior:** Org IDs should be non-iterable, and tampered invites should fail validation.

- **Backend Validation of Org ID:**
    - Test if the backend strictly validates that the invite corresponds to the correct organization.

---

### ⌛ Invite Expiration Handling

- **Validation of Expiry for Canceled or Removed Invites:**
    - Invite a user, then cancel the invite or remove the user from the org.
    - **Expected Behavior:** Invite links should expire or become invalid upon cancellation/removal.

- **Time-Limited Invite Tokens:**
    - Check if invite tokens expire after a set period (e.g., 24 hours).

  **Steps:**
  1. Generate an invite and capture the token.
  2. Use the token after the expected expiry time.
  - **Expected Behavior:** Expired tokens should return a "Token Expired" error.

---

### 🔧 Role Modification on Invitation

- **Role Escalation Post Invitation:**
    - Test if an invited user can escalate their role after accepting the invite.

  **Steps:**
  1. Accept an invite with a low privilege role (e.g., Viewer).
  2. Use parameter manipulation or API calls to upgrade the role (e.g., Viewer → Admin).
  - **Expected Behavior:** Role changes should only be performed by authorized users (e.g., org admins).

---

### 🧠 Additional Scenarios to Test

- **Mass Invites Abuse:**
    - Test if a user can send unlimited invites without rate-limiting or approval.
    - **Expected Behavior:** Implement rate limits and require admin approval for bulk invites.

- **Cross-Org Notifications:**
    - Test if users receive unauthorized notifications when part of multiple organizations.
    - **Expected Behavior:** Notifications should only be sent to relevant org members.

- **Audit Logs and Monitoring:**
    - Check if all invite-related actions (e.g., sending, canceling, accepting invites) are logged.
    - **Expected Behavior:** The system should log invite actions with timestamps and actor details.

---

### 🎭 Role-Based Testing

- **Unauthorized Role Assignment**:
    - Use a Viewer account to assign Admin roles via API tampering.
    - **Expected Result**: 403 Forbidden.

- **Session Invalidation**:
    - Downgrade a user’s role and test active session permissions.
    - **Expected Result**: Session invalidated or privileges revoked.

- **Failure to Revoke Permissions:**
    - Check if permissions are correctly revoked after role downgrades or user removal.

- **Data Ownership**:
    - Access another user’s resource via API (e.g., `GET /api/docs/567`).
    - **Expected Result**: 403 Forbidden.

- **Overly Permissive Roles:**
    - Verify that roles are not granted more permissions than necessary.
    - **Test:** Compare permissions granted to each role with the expected access matrix.

- **Session Hijacking for Role Misuse:**
    - Test if a session from a downgraded user retains elevated permissions.

- **Access Control in Multi-Tenant Applications:**
    - Ensure users with the same role in one tenant cannot access data from another tenant.

- **Default Role Assignment:**
    - Test the default role assigned to new users upon registration or invitation.
    - **Expected Behavior:** Default role should have the least privileges (e.g., Viewer).
