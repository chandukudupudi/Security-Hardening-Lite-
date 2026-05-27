# HR Security Hardening Lite – ServiceNow

## Project Overview

HR Security Hardening Lite is a security-focused ServiceNow scoped application developed to demonstrate enterprise-grade access control, record-level security, field-level protection, and secure API design using ACLs, roles, Business Rules, and Scripted REST APIs.

The project simulates a confidential HR request management system where employees can submit sensitive HR requests while ensuring strict access governance and data protection.

This implementation demonstrates real-world ServiceNow security architecture concepts commonly used in HRSD, security governance, and enterprise compliance environments.

---

# Business Requirement

The organization required a secure HR request management solution capable of:

* Restricting users to view only their own HR requests
* Allowing HR agents to manage all HR requests
* Protecting sensitive fields such as SSN data
* Securing REST API access based on roles
* Enforcing ownership and governance controls
* Supporting audit-friendly access management

---

# Technologies Used

| Technology            | Purpose                         |
| --------------------- | ------------------------------- |
| ACLs                  | Record and field-level security |
| Scoped Roles          | Role-based access control       |
| Business Rules        | Ownership defaulting            |
| Scripted REST APIs    | Secure API integration          |
| GlideRecord           | Data operations                 |
| Scoped Applications   | Modular development             |
| GitHub Source Control | Version management              |

---

# Application Details

| Item              | Value                      |
| ----------------- | -------------------------- |
| Application Name  | HR Security Hardening Lite |
| Scope             | x_ns_hr_sec_lite           |
| Version           | 1.0.0                      |
| Development Style | Scoped Application         |
| Source Control    | GitHub Integrated          |

---

# Project Architecture

The project enforces layered ServiceNow security controls across records, fields, and APIs.

---

# Security Architecture

```text id="ylm3s8"
Employee User
      ↓
ACL Validation
      ↓
HR Confidential Request Table
      ↓
Field-Level Security Checks
      ↓
Business Rule Governance
      ↓
Secure Scripted REST API
```

---

# Core Components Implemented

# 1. HR Confidential Request Table

A standalone HR request table was created to store confidential employee requests securely.

---

# Table Name

```text id="4wmp1k"
u_hr_conf_request
```

---

# Fields Created

| Field Label   | Field Name      | Type                 |
| ------------- | --------------- | -------------------- |
| Title         | u_title         | String               |
| Description   | u_description   | String               |
| SSN Last 4    | u_ssn_last4     | String               |
| Owner         | u_owner         | Reference → sys_user |
| Status        | u_status        | Choice               |
| HR Notes      | u_hr_notes      | String               |
| Requested For | u_requested_for | Reference → sys_user |

---

# 2. Scoped Roles

Implemented scoped application roles to support role-based security governance.

---

# Roles Created

| Role                   | Purpose           |
| ---------------------- | ----------------- |
| x_ns_hr_sec_lite.user  | Employee access   |
| x_ns_hr_sec_lite.agent | HR support team   |
| x_ns_hr_sec_lite.admin | HR administrators |

---

# Access Model

| Role  | Access                         |
| ----- | ------------------------------ |
| user  | View/edit own records          |
| agent | View/edit all records          |
| admin | Full access + sensitive fields |

---

# 3. Table-Level ACLs

Implemented scripted ACLs to enforce ownership-based access control.

---

# READ ACL Logic

## Functionalities

* HR agents/admins can read all records
* Employees can only read:

  * Their own records
  * Records requested for them

---

# READ ACL Script

```javascript id="6w8u2k"
(function() {

  if (
      gs.hasRole('x_ns_hr_sec_lite.admin') ||
      gs.hasRole('x_ns_hr_sec_lite.agent')
  )
      return true;

  var me = gs.getUserID();

  return (
      current.u_owner == me ||
      current.u_requested_for == me
  );

})();
```

---

# WRITE ACL Logic

## Functionalities

* HR agents/admins can edit all records
* Employees can edit only:

  * Their own records
  * Non-closed requests

---

# WRITE ACL Script

```javascript id="lm5zxn"
(function() {

  if (
      gs.hasRole('x_ns_hr_sec_lite.admin') ||
      gs.hasRole('x_ns_hr_sec_lite.agent')
  )
      return true;

  var me = gs.getUserID();

  if (current.u_status == 'Closed')
      return false;

  return (
      current.u_owner == me ||
      current.u_requested_for == me
  );

})();
```

---

# DELETE ACL

| Operation | Access     |
| --------- | ---------- |
| Delete    | Admin only |

---

# 4. Field-Level Security

Implemented field-level ACLs to protect sensitive HR information.

---

# Protected Field

```text id="fqlx0d"
u_ssn_last4
```

---

# Security Behavior

| Role  | SSN Visibility |
| ----- | -------------- |
| user  | Hidden         |
| agent | Hidden         |
| admin | Visible        |

---

# Field ACL Script

```javascript id="0v1q6d"
(function() {

  return gs.hasRole(
      'x_ns_hr_sec_lite.admin'
  );

})();
```

---

# 5. Ownership Governance

A Business Rule was implemented to automatically assign ownership and default values.

---

# Functionalities

* Automatically sets Owner
* Automatically sets Requested For
* Automatically defaults Status to New

---

# Business Rule Script

```javascript id="72efrr"
(function executeRule(current, previous) {

  if (!current.u_owner)
      current.u_owner = gs.getUserID();

  if (!current.u_requested_for)
      current.u_requested_for = gs.getUserID();

  if (!current.u_status)
      current.u_status = 'New';

})();
```

---

# 6. Secure Scripted REST API

A role-protected Scripted REST API was implemented to securely create HR requests.

---

# API Details

| Property      | Value                       |
| ------------- | --------------------------- |
| API Name      | HR Confidential Request API |
| API ID        | hr_conf_req_api             |
| Resource Path | /create                     |
| Method        | POST                        |

---

# API Security Features

| Feature                   | Behavior                  |
| ------------------------- | ------------------------- |
| Role Validation           | Only HR roles allowed     |
| Payload Validation        | Validates required fields |
| Sensitive Data Protection | Only admin can set SSN    |
| Error Handling            | Returns proper HTTP codes |

---

# Scripted REST API Script

```javascript id="m2zyfi"
(function process(request, response) {

  try {

    if (!(
        gs.hasRole('x_ns_hr_sec_lite.agent') ||
        gs.hasRole('x_ns_hr_sec_lite.admin')
    )) {

      response.setStatus(403);

      response.setBody({
        error:
        'Forbidden: HR agent/admin role required'
      });

      return;
    }

    var body = request.body.data || {};

    if (!body.title) {

      response.setStatus(400);

      response.setBody({
        error: 'title is required'
      });

      return;
    }

    var gr = new GlideRecord(
      'x_scope_u_hr_conf_request'
    );

    gr.initialize();

    gr.u_title = body.title;

    gr.u_description =
        body.description || '';

    gr.u_requested_for =
        body.requested_for ||
        gs.getUserID();

    gr.u_owner =
        body.owner ||
        gs.getUserID();

    if (
        body.ssn_last4 &&
        gs.hasRole(
          'x_ns_hr_sec_lite.admin'
        )
    ) {

      gr.u_ssn_last4 =
          body.ssn_last4;
    }

    var sysId = gr.insert();

    response.setStatus(201);

    response.setBody({
      message: 'Created',
      sys_id: sysId
    });

  } catch (e) {

    response.setStatus(500);

    response.setBody({
      error: e.message
    });
  }

})(request, response);
```

---

# API Security Validation

| User Type     | Result                 |
| ------------- | ---------------------- |
| Employee User | HTTP 403               |
| HR Agent      | HTTP 201               |
| HR Admin      | HTTP 201 + SSN allowed |

---

# Testing Scenarios

# Scenario 1 – Employee Isolation

### Test

emp1 creates HR request.

emp2 attempts to open record.

### Expected Result

* Access denied

---

# Scenario 2 – HR Agent Access

### Test

HR agent opens employee records.

### Expected Result

* Records visible
* SSN field hidden

---

# Scenario 3 – HR Admin Access

### Test

HR admin opens employee record.

### Expected Result

* Full field visibility including SSN

---

# Scenario 4 – REST API Security

### Test

Call API as non-HR user.

### Expected Result

* HTTP 403 returned

---

# End-to-End Workflow

```text id="z16tkn"
Employee Creates HR Request
            ↓
Business Rule Sets Ownership
            ↓
ACL Validation Executes
            ↓
Field-Level Security Applied
            ↓
HR Agents Process Requests
            ↓
REST API Access Controlled by Roles
```

---

# Enterprise Concepts Demonstrated

| Concept                   | Implemented |
| ------------------------- | ----------- |
| Record-Level ACLs         | Yes         |
| Field-Level ACLs          | Yes         |
| Role-Based Access         | Yes         |
| Ownership Validation      | Yes         |
| Secure APIs               | Yes         |
| Sensitive Data Protection | Yes         |
| Governance Controls       | Yes         |
| Audit-Friendly Design     | Yes         |

---

# Technical Challenges Solved

## Challenge 1

Restricting employees to their own records.

### Solution

Implemented scripted READ and WRITE ACLs.

---

## Challenge 2

Protecting sensitive SSN data.

### Solution

Implemented field-level ACLs.

---

## Challenge 3

Securing API access.

### Solution

Added role validation in Scripted REST API.

---

## Challenge 4

Ensuring ownership consistency.

### Solution

Implemented Business Rule defaulting logic.

---

# GitHub Repository Structure

```text id="gfgsqk"
10-hr-security-hardening-lite/
│
├── README.md
│
├── docs/
│   ├── security-model.md
│   ├── acl-design.md
│   ├── api-security.md
│   ├── governance.md
│   └── testing.md
│
├── media/
│
└── update_sets/
```

---

# Real-World Enterprise Benefits

| Benefit                   | Impact                 |
| ------------------------- | ---------------------- |
| Sensitive Data Protection | Improved compliance    |
| Role-Based Security       | Controlled access      |
| Secure APIs               | Reduced exposure risk  |
| Ownership Governance      | Improved auditability  |
| Field Protection          | Better confidentiality |

---

# Project Outcome

The HR Security Hardening Lite implementation successfully demonstrated enterprise-grade ServiceNow security architecture using ACLs, scoped roles, Business Rules, and secure Scripted REST APIs.

The project improved:

* Data confidentiality
* Access governance
* API security
* Audit readiness
* HR request protection
