# Campus Equipment Borrowing & Reservation System

## 1. Project Overview

Campus Equipment is a web-based system for managing school-owned equipment such as laptops, projectors, cameras, speakers, and laboratory tools. It gives students and faculty a way to check availability and request equipment, while custodians and administrators manage approvals, releases, returns, records, and access.

The initial release is intentionally focused on the core borrowing workflow and does not require advanced third-party integrations.

## 2. User Roles

| Role | Main purpose | Primary access |
| --- | --- | --- |
| Borrower | Student or faculty member requesting equipment | Browse availability, submit and view reservations, view own borrowing history |
| Custodian | Staff member responsible for equipment handover and return checks | Review requests, approve/reject, release items, process returns and condition checks |
| Administrator | System manager | Manage users, roles, categories, equipment records, and reports |

## 3. Functional Breakdown

### 3.1 Authentication

#### User Login
- Users sign in with their registered email/student-faculty ID and password.
- The system validates credentials before granting access.
- Unauthenticated users cannot use protected system features.

#### Role-Based Dashboard
- After sign-in, the system opens the dashboard appropriate to the user’s role.
- Borrowers see available equipment, their requests, and their current borrowings.
- Custodians see pending approval, release, and return tasks.
- Administrators see management and reporting functions.

#### Password Reset Request
- A user can request a password-reset link or temporary code using their registered email address.
- The reset process verifies the user before allowing a new password to be created.

### 3.2 Master Data Management

#### Equipment Category Management
- Administrators can create, view, edit, and deactivate equipment categories.
- Example categories: Laptop, Projector, Camera, Speaker, Laboratory Tool.
- Every equipment item belongs to one category so inventory can be organized and filtered consistently.

#### Equipment Item Management
- Administrators can register and maintain each borrowable item.
- Each record should include: item code, category, name/model, serial number, condition, current status, and storage/assigned location.
- An item can be marked unavailable when under maintenance, lost, retired, or otherwise not borrowable.

#### Borrower Profile Management
- Administrators maintain borrower records for students and faculty.
- Profile details include name, ID number, department, contact number, email address, and eligibility status.
- Ineligible or inactive borrowers cannot place new requests.

### 3.3 Borrowing Transactions

#### Equipment Reservation
- Borrowers search and filter available equipment by category, name, and date.
- They select an item, intended borrowing date/time, return date/time, and purpose of use.
- The system creates a reservation request and prevents conflicting reservations for the same item and time period.
- Borrowers can view the status of their submitted requests.

#### Approval and Release
- Custodians review pending reservations and approve or reject each request.
- A rejection should store a reason visible to the borrower.
- For approved requests, the custodian records the actual release date/time and marks the item as released/borrowed.
- Releasing an item updates its current availability so it cannot be reserved by another borrower during the active loan.

#### Return and Condition Check
- Custodians record the actual return date/time when an item is returned.
- They inspect the item and record its returned condition.
- The item status is updated to available, under maintenance, or another appropriate status.
- Late returns and damaged items are flagged for follow-up and reporting.

### 3.4 Reports and Visibility

#### Borrowing History Report
- Authorized staff can view filterable borrowing history.
- Filters include borrower, equipment item/category, date range, transaction status, and late-return status.
- The report supports accountability by showing request, approval, release, and return records.

#### Availability Calendar
- Users can view a daily or weekly calendar of equipment availability.
- The calendar distinguishes available, reserved, borrowed, and under-maintenance items.
- It helps borrowers select dates that do not conflict with existing reservations.

### 3.5 Administration

#### User and Role Management
- Administrators create, edit, activate/deactivate, and assign roles to user accounts.
- Roles control which dashboard, pages, actions, and data each user can access.
- Deactivating an account prevents login while retaining transaction history for audit purposes.

## 4. Core Workflow

1. An administrator configures users, borrower profiles, equipment categories, and equipment items.
2. A borrower logs in, checks the availability calendar, and submits a reservation request.
3. A custodian reviews the request and approves or rejects it.
4. When the borrower collects an approved item, the custodian records the release.
5. When the item is returned, the custodian records the return and condition check.
6. The system updates the item’s availability and preserves the full transaction history for reports.

## 5. Main Statuses

| Area | Suggested statuses |
| --- | --- |
| Reservation | Pending, Approved, Rejected, Cancelled, Expired |
| Loan | Awaiting Release, Borrowed, Returned, Overdue |
| Equipment | Available, Reserved, Borrowed, Under Maintenance, Unavailable |
| Return condition | Good, Damaged, Needs Inspection, Under Repair |

## 6. Key Business Rules

- Only active, eligible borrowers may create reservations.
- A reservation requires an available item and a valid return date/time after its release date/time.
- One item cannot have overlapping approved reservations or active loans.
- Only custodians and administrators can approve, reject, release, or complete a return.
- Equipment marked as unavailable or under maintenance cannot be reserved or released.
- Every approval, rejection, release, return, and status change should retain the acting user and timestamp for audit history.
- A returned item with a damaged condition must not automatically become available until its status is reviewed.

## 7. Initial Data Entities

| Entity | Purpose | Important fields |
| --- | --- | --- |
| User | Login and access control | User ID, name, email, password hash, role, active status |
| Borrower Profile | Student/faculty eligibility record | Borrower ID, user ID, school ID, department, contact details, eligibility |
| Role | Permission group | Role ID, name, permissions |
| Equipment Category | Equipment grouping | Category ID, name, active status |
| Equipment Item | Individual trackable item | Item ID/code, category, name/model, serial number, condition, status, location |
| Reservation | Requested schedule for an item | Reservation ID, borrower, item, purpose, requested release/return dates, status |
| Release Record | Confirmation of handover | Release ID, reservation, custodian, actual release time, notes |
| Return Record | Confirmation of return and inspection | Return ID, release/reservation, custodian, actual return time, condition, notes |

## 8. Suggested Screens

| Screen | Users | Purpose |
| --- | --- | --- |
| Login / Password Reset | All users | Authenticate and regain account access |
| Borrower Dashboard | Borrower | View available items, current requests, and active loans |
| Custodian Dashboard | Custodian | Act on pending requests, releases, and returns |
| Administrator Dashboard | Administrator | Monitor inventory, users, and reporting shortcuts |
| Equipment Catalog | Borrower, Custodian, Administrator | Search and view item availability |
| Reservation Form | Borrower | Submit an equipment request |
| Approval & Release Queue | Custodian, Administrator | Approve/reject requests and record handover |
| Return & Condition Check | Custodian, Administrator | Record return and item condition |
| Categories and Equipment | Administrator | Maintain inventory master data |
| Borrower Profiles | Administrator | Maintain borrower records and eligibility |
| Users and Roles | Administrator | Maintain accounts and permissions |
| Reports / Availability Calendar | Authorized users | Review history and scheduling availability |

## 9. Scope for the First Website Version

The first website should implement the functions documented above with responsive web pages, role-based access, and a database-backed audit trail. Out of scope unless later requested: online payment/penalties, external school-information-system synchronization, barcode scanning, email/SMS notifications, and multi-campus inventory transfers.
