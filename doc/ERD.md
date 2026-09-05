# Entity Relationship Diagram (ERD)

This ERD supports the initial Campus Equipment Borrowing & Reservation System scope. It models access control, equipment inventory, reservations, releases, and returns.

```mermaid
erDiagram
    ROLE ||--o{ USER : assigns
    USER ||--o| BORROWER_PROFILE : has
    EQUIPMENT_CATEGORY ||--o{ EQUIPMENT_ITEM : classifies
    BORROWER_PROFILE ||--o{ RESERVATION : creates
    EQUIPMENT_ITEM ||--o{ RESERVATION : is_requested_in
    USER ||--o{ RESERVATION : reviews
    RESERVATION ||--o| RELEASE_RECORD : results_in
    USER ||--o{ RELEASE_RECORD : releases
    RELEASE_RECORD ||--o| RETURN_RECORD : ends_with
    USER ||--o{ RETURN_RECORD : receives

    ROLE {
        int role_id PK
        string name UK
        string description
        boolean is_active
    }

    USER {
        int user_id PK
        int role_id FK
        string full_name
        string email UK
        string password_hash
        boolean is_active
        datetime created_at
        datetime updated_at
    }

    BORROWER_PROFILE {
        int borrower_id PK
        int user_id FK
        string school_id UK
        string department
        string contact_number
        boolean is_eligible
        datetime created_at
        datetime updated_at
    }

    EQUIPMENT_CATEGORY {
        int category_id PK
        string name UK
        string description
        boolean is_active
        datetime created_at
        datetime updated_at
    }

    EQUIPMENT_ITEM {
        int item_id PK
        int category_id FK
        string item_code UK
        string name
        string model
        string serial_number UK
        string current_condition
        string current_status
        string storage_location
        boolean is_active
        datetime created_at
        datetime updated_at
    }

    RESERVATION {
        int reservation_id PK
        int borrower_id FK
        int item_id FK
        int reviewed_by_user_id FK
        string purpose
        datetime requested_release_at
        datetime requested_return_at
        string status
        string reviewer_notes
        datetime reviewed_at
        datetime created_at
        datetime updated_at
    }

    RELEASE_RECORD {
        int release_id PK
        int reservation_id FK_UK
        int released_by_user_id FK
        datetime actual_release_at
        string release_notes
        datetime created_at
    }

    RETURN_RECORD {
        int return_id PK
        int release_id FK_UK
        int received_by_user_id FK
        datetime actual_return_at
        string returned_condition
        string resulting_item_status
        string return_notes
        datetime created_at
    }
```

## Relationship Notes

| Relationship | Cardinality | Meaning |
| --- | --- | --- |
| Role → User | One-to-many | A role can be assigned to many user accounts; each user has one role in the first version. |
| User → Borrower Profile | One-to-zero-or-one | Only student/faculty accounts that borrow equipment need a borrower profile. |
| Equipment Category → Equipment Item | One-to-many | A category groups multiple physical equipment items. |
| Borrower Profile → Reservation | One-to-many | A borrower can make many reservation requests over time. |
| Equipment Item → Reservation | One-to-many | One item can occur in many historical reservations, but date-time conflicts must be prevented. |
| Reservation → Release Record | One-to-zero-or-one | Only an approved reservation that is handed over creates a release record. |
| Release Record → Return Record | One-to-zero-or-one | A released item receives one return record when the loan ends. |

## Data Rules Represented by the ERD

- `email`, `school_id`, `item_code`, category `name`, and role `name` should be unique.
- `serial_number` should be unique when the item has a manufacturer serial number; allow a nullable value for items without one.
- `RELEASE_RECORD.reservation_id` is unique: a reservation can be released only once.
- `RETURN_RECORD.release_id` is unique: a released item can be returned only once.
- `RESERVATION.reviewed_by_user_id` is filled when a custodian or administrator approves or rejects the request.
- Reservation date/time overlap validation is a business rule enforced by the application/database query rather than a simple foreign-key relationship.
