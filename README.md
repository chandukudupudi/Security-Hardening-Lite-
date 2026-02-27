# HR Security Hardening Lite – ServiceNow

A scoped ServiceNow application demonstrating enterprise-grade access control
using ACLs, role-based security, field-level protection, and secured REST APIs.

---

## 🎯 Project Objective
To design a secure HR Confidential Request system that enforces:

- Record-level access controls
- Role-based authorization
- Field-level data protection
- Secure Scripted REST APIs
- Ownership-based visibility
- Audit-friendly configuration

---

## 🔐 Security Features Implemented

### 1. Role-Based Access
- x_<scope>_hr_sec_lite.user
- x_<scope>_hr_sec_lite.agent
- x_<scope>_hr_sec_lite.admin

### 2. Record-Level Security
- Users can only view/edit their own records
- HR Agents can view/edit all records
- Delete restricted to Admin only

### 3. Field-Level Protection
- SSN Last 4 visible only to HR Admin
- Protected via Field ACL

### 4. Secure Scripted REST API
- POST endpoint for creating HR requests
- Restricted to Agent/Admin roles
- 403 returned for unauthorized users

---

## 🧪 Demo Scenarios

1. Employee creates request → another employee cannot see it
2. HR Agent can see all requests but cannot see SSN
3. HR Admin can see SSN field
4. REST API call as non-HR user returns 403
5. REST API call as HR Agent returns 201 Created

---

## 🧠 Interview Value

Demonstrates:
- Deep understanding of ServiceNow ACL engine
- Scripted security conditions
- Role-based authorization design
- REST API protection
- Field-level data governance
