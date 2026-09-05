# Technology Stack

This stack applies to the **entire Campus Equipment Borrowing & Reservation System**: authentication, user roles, equipment inventory, reservations, approvals, releases, returns, reports, availability calendar, and administration.

## 1. Selected Stack

| Layer | Technology | Purpose |
| --- | --- | --- |
| Backend web framework | ASP.NET Core (C#) | Builds the web application, dashboards, APIs/controllers, authorization rules, validation, and all business logic. |
| Authentication and authorization | ASP.NET Core Identity | Manages user accounts, password hashing, sign-in, role-based access, password-reset tokens, and account security. |
| Database | PostgreSQL | Stores users, roles, borrower profiles, inventory, reservations, releases, returns, and audit data. |
| Database access | Entity Framework Core + `Npgsql.EntityFrameworkCore.PostgreSQL` | Maps C# entities to PostgreSQL tables and manages database queries and migrations. |
| Email library | MailKit | Sends password-reset messages from the C# application through SMTP. |
| Email delivery | School SMTP server | Delivers reset emails from an approved school address such as `noreply@school.edu`. |
| User interface | ASP.NET Core Razor Pages | Provides responsive browser pages, forms, tables, dashboards, and role-specific screens. Razor Pages is the selected UI approach for the first version. |
| Styling | Bootstrap 5 + custom CSS | Creates a responsive, consistent interface for desktop and mobile users without requiring a separate frontend application. |
| Client-side interaction | Vanilla JavaScript | Supports confirmation dialogs, table filtering, calendar interactions, and small asynchronous UI updates where needed. |
| Calendar | FullCalendar JavaScript library | Displays daily/weekly equipment availability using reservation and loan data provided by ASP.NET Core. |
| Reporting | EF Core queries + Razor Pages | Produces filterable borrowing-history, overdue, availability, and inventory reports. PDF/Excel export can be added later if required. |
| Validation | ASP.NET Core model validation + FluentValidation (optional) | Validates forms and business rules on the server, including required fields and valid date ranges. |
| Logging | ASP.NET Core logging | Records application errors and operational events; critical business actions are also retained in PostgreSQL audit fields. |
| Deployment | IIS on a school Windows Server or Azure App Service | Hosts the ASP.NET Core application. The final hosting choice depends on the school infrastructure. |

## 2. Password Recovery Architecture

```text
Browser
  │  1. User enters registered email
  ▼
ASP.NET Core application
  │  2. Finds active account and creates a secure reset token
  ▼
PostgreSQL                         MailKit
  │  Stores user/password hash       │  3. Sends reset email
  │  and reset-related data          ▼
  └───────────────────────────── School SMTP server
                                      │
                                      ▼
                                User's registered email inbox
```

PostgreSQL stores application data; it does **not** send email. MailKit connects to the school SMTP server, which delivers the reset message.

## 3. Whole-System Architecture

```text
Borrower / Custodian / Administrator browser
                  │
                  ▼
Razor Pages + Bootstrap + JavaScript + FullCalendar
                  │ HTTPS
                  ▼
ASP.NET Core application
 ├─ ASP.NET Core Identity: sign-in, roles, password recovery
 ├─ Business services: reservations, conflict checks, approval, release, return
 ├─ Reporting services: history, overdue, availability, inventory
 ├─ Entity Framework Core + Npgsql
 └─ MailKit: password-reset email delivery
                  │
                  ▼
             PostgreSQL database
  Users | Roles | Profiles | Categories | Items | Reservations
  Releases | Returns | Status history / audit information
                  │
                  └── School SMTP server (password-reset email only)
```

## 4. Technology Use by System Function

| System function | Technologies used | How they support the feature |
| --- | --- | --- |
| Login and role-based dashboards | ASP.NET Core Identity, Razor Pages, PostgreSQL | Authenticates the account, loads its role, and directs the user to the Borrower, Custodian, or Administrator dashboard. |
| Forgot Password | ASP.NET Core Identity, MailKit, school SMTP, PostgreSQL | Generates a protected reset token, sends the reset link, and securely updates the password hash. |
| User and role management | ASP.NET Core Identity, EF Core, PostgreSQL | Creates/deactivates accounts and assigns system roles. |
| Borrower profiles | Razor Pages, EF Core, PostgreSQL | Maintains school ID, department, contact information, and borrowing eligibility. |
| Category and equipment management | Razor Pages, EF Core, PostgreSQL | Creates and maintains categories, individual equipment records, conditions, locations, and statuses. |
| Equipment search and availability | Razor Pages, EF Core, PostgreSQL | Filters items by category/status and determines whether an item is borrowable. |
| Reservation requests | Razor Pages, server-side validation, EF Core, PostgreSQL | Accepts a requested schedule and stores the request only when the borrower and item are eligible. |
| Reservation conflict prevention | C# business service, PostgreSQL transaction/query | Checks approved reservations and active loans to prevent overlapping bookings for the same item. |
| Approval and release | Razor Pages, role authorization, EF Core, PostgreSQL | Allows custodians/administrators to approve or reject requests and record actual handover. |
| Return and condition check | Razor Pages, EF Core, PostgreSQL | Records actual return time and condition, then updates the equipment status to available or under maintenance. |
| Availability calendar | FullCalendar, JavaScript, ASP.NET Core endpoint, PostgreSQL | Shows reserved, borrowed, available, and maintenance periods in daily/weekly views. |
| Borrowing history and reports | EF Core queries, Razor Pages, PostgreSQL | Provides filterable reports by borrower, item, date range, status, and overdue/late-return state. |
| Audit trail | PostgreSQL audit columns, ASP.NET Core logging | Retains who performed important actions and when, while application logs capture technical errors. |

## 5. Forgot Password Implementation Decision

Use an expiring **password-reset link**, generated by ASP.NET Core Identity, instead of a custom numeric OTP for the first version.

1. The user selects **Forgot Password** and submits their registered email.
2. The application checks for an active account without revealing whether the email exists.
3. ASP.NET Core Identity generates a secure, time-limited password-reset token.
4. The application creates a reset URL containing the encoded token.
5. MailKit sends the URL through the school SMTP server.
6. The user opens the link and enters a new password.
7. ASP.NET Core Identity validates the token and updates the stored password hash in PostgreSQL.

If the project later requires a numeric OTP, store only its hash, expiry timestamp, used flag, and failed-attempt count in PostgreSQL. Do not store the plain OTP.

## 6. Core Application Dependencies

| Package | Use |
| --- | --- |
| `Microsoft.AspNetCore.Identity.EntityFrameworkCore` | ASP.NET Core Identity persistence through Entity Framework Core. |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | PostgreSQL provider for Entity Framework Core. |
| `MailKit` | SMTP email sending with modern TLS and authentication support. |
| `Microsoft.EntityFrameworkCore.Design` | Enables Entity Framework Core migration tooling during development. |
| FullCalendar | Frontend JavaScript dependency used for the equipment availability calendar. |
| Bootstrap 5 | Frontend CSS/JavaScript dependency for responsive UI components. |

## 7. Configuration and Secrets

Keep secrets out of source code and version control. Use development user secrets or environment variables locally, and secure deployment configuration in production.

Required configuration values:

| Setting | Example purpose |
| --- | --- |
| `ConnectionStrings:DefaultConnection` | PostgreSQL connection string. |
| `Smtp:Host` | School SMTP hostname. |
| `Smtp:Port` | SMTP port, commonly `587` for STARTTLS. |
| `Smtp:Username` | SMTP account name. |
| `Smtp:Password` | SMTP password, app password, or protected credential. |
| `Smtp:FromAddress` | Approved sender address, such as `noreply@school.edu`. |
| `Smtp:FromName` | Sender display name, such as Campus Equipment. |
| `Application:BaseUrl` | Public URL used when generating password-reset links. |

## 8. Security Requirements

- Use HTTPS for the website and TLS/STARTTLS for SMTP.
- Never store plaintext passwords, plaintext OTPs, or SMTP credentials in the database or source code.
- Let ASP.NET Core Identity hash passwords and generate password-reset tokens.
- Return the same response for known and unknown email addresses on the Forgot Password page to reduce account enumeration.
- Expire reset links after a short period and invalidate them after successful password reset.
- Apply rate limiting to password-reset requests and sign-in attempts.
- Restrict administration pages with role-based authorization.
- Record critical transaction actions—approval, release, return, and account changes—with user and timestamp.

## 9. Non-Required Third-Party Services for Version 1

The first version does not require SendGrid, Firebase Authentication, Twilio SMS, Azure Communication Services, or a payment service. The school SMTP server plus MailKit is sufficient, assuming the school provides valid SMTP access.
