# Oracle APEX – Dynamic Authorization-Based Menu

## 🔹 Overview

This repository demonstrates a **simple and effective authorization pattern** in Oracle APEX where **authorization and navigation menu visibility** are managed together using a shared data model.

By using **two database tables only**, the solution ensures that:

- When a user is authorized for a page  
- The corresponding menu item **appears automatically**  

No extra configuration is required.

This approach eliminates duplicated admin steps and keeps **security and UI fully synchronized**.

---

## 🔹 Why This Approach?

In many APEX applications:
- Page authorization is managed separately
- Menu visibility is configured manually

This often leads to:
- Inconsistencies
- Extra admin work
- Security gaps

👉 This solution solves that by making **authorization the single source of truth**.

---

## 🔹 Architecture Summary

- Authorization is stored in database tables
- A **PL/SQL Authorization Scheme** controls page access
- The **Navigation Menu** is generated dynamically based on the same authorization data

Result:

> Grant access → Page opens + Menu item appears  
> Revoke access → Page blocked + Menu item hidden  

---

## 🔹 Database Tables

### 1️⃣ `AUTH_CUSTOM_MENU`

Stores the application menu structure.

Typical columns:
- `ID` – Menu ID
- `PID` – Parent menu ID
- `NAME` – Menu label
- `TAG` – APEX Page ID
- `ICON` – Menu icon
- `APP_ID` – Application ID
- `ACTIVE` – Y / N
- `SORT` – Display order

---

### 2️⃣ `AUTH_USERS_PAGES`

Stores user authorization per page.

Typical columns:
- `USER_ID`
- `PAGE_ID`
- `APP_ID_V`
- `ALLOWED` (Y / N)

---

## 🔹 Authorization Scheme (PL/SQL)

Create an **Authorization Scheme** of type:

> **PL/SQL Function Returning Boolean**

```sql
DECLARE
    a NUMBER;
BEGIN
    SELECT COUNT(*)
    INTO a
    FROM auth_users_pages up
    INNER JOIN auth_custom_menu cm
        ON up.page_id = cm.id
    WHERE up.user_id   = get_user_id(:APP_USER)
      AND up.app_id_v  = :APP_ID
      AND cm.tag       = :APP_PAGE_ID;

    IF a > 0 THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;
END;


END;
```
📌 This ensures:

The page is accessible only if the user is authorized

Authorization is evaluated dynamically at runtime

🔹 Dynamic Navigation Menu Query
Use the following SQL for a Dynamic Navigation Menu in Oracle APEX:


```sql
SELECT
    LEVEL,
    name AS label,
    'f?p=' || :APP_ID || ':' || tag || ':' || :APP_SESSION || ':::::' AS target,
    name AS is_current,
    icon AS image
FROM auth_custom_menu
WHERE id IN (
    SELECT a.page_id
    FROM auth_users_pages a
    WHERE a.user_id = get_user_id(:APP_USER)
      AND a.allowed = 'Y'
)
  AND app_id = :APP_ID
  AND active = 'Y'
START WITH pid IS NULL
CONNECT BY PRIOR id = pid
ORDER SIBLINGS BY sort;
```
📌 Result:

Each user sees only the pages they are authorized for

Menu structure is built dynamically

## 🔹 Key Benefits

✅ Single authorization action

✅ Automatic menu visibility

✅ No duplicated admin steps

✅ Clean and scalable design

✅ Ideal for enterprise APEX applications

## 🔹 Use Cases
Role-based access control

Admin panels

Large multi-user APEX systems

Applications with frequent authorization changes

## 🔹 Notes
TAG column must match the APEX PAGE_ID

Authorization is evaluated per request

Works with any custom user management logic

## 🔹 Author
Designed as a lightweight, database-driven authorization model for Oracle APEX applications where simplicity, security, and maintainability are required.





















