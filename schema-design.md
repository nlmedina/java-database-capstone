# SmartCare Clinic: Database Schema

This document defines the authoritative database schema for SmartCare Clinic. The system uses:

- MySQL for relational data (users, profiles, appointments).
- MongoDB for document-oriented data (prescriptions).

All tables, columns, and constraints below are expressed in database terms, independent of any application framework.

---

## MySQL Relational Schema

### Table: `admin`

Stores administrator credentials.

| Column     | Type         | Constraints                 | Notes                        |
| :--------- | :----------- | :-------------------------- | :--------------------------- |
| `id`       | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |                              |
| `username` | VARCHAR(255) | NOT NULL, UNIQUE            |                              |
| `password` | VARCHAR(255) | NOT NULL                    | Stored hashed in production. |

---

### Table: `doctor`

Stores doctor profile and login details.

| Column      | Type         | Constraints                 | Notes                                 |
| :---------- | :----------- | :-------------------------- | :------------------------------------ |
| `id`        | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |                                       |
| `name`      | VARCHAR(100) | NOT NULL                    |                                       |
| `specialty` | VARCHAR(50)  | NOT NULL                    |                                       |
| `email`     | VARCHAR(255) | NOT NULL, UNIQUE            |                                       |
| `password`  | VARCHAR(255) |                             | Store hashed; enforce min length app. |
| `phone`     | VARCHAR(10)  | NOT NULL                    | Exactly 10 digits.                    |

### Table: `doctor_available_times`

Stores available time slots for each doctor.

| Column           | Type         | Constraints                      | Notes                                     |
| :--------------- | :----------- | :------------------------------- | :---------------------------------------- |
| `doctor_id`      | BIGINT       | NOT NULL, FOREIGN KEY doctor(id) |                                           |
| `available_time` | VARCHAR(255) | NOT NULL                         | Time window string (e.g., "09:00-10:00"). |

PRIMARY KEY (`doctor_id`, `available_time`) to prevent duplicates.

---

### Table: `patient`

Stores patient profile and login details.

| Column      | Type         | Constraints                 | Notes                                 |
| :---------- | :----------- | :-------------------------- | :------------------------------------ |
| `id`        | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |                                       |
| `name`      | VARCHAR(100) | NOT NULL                    |                                       |
| `email`     | VARCHAR(255) | NOT NULL, UNIQUE            |                                       |
| `specialty` | VARCHAR(50)  | NOT NULL                    |                                       |
| `password`  | VARCHAR(255) |                             | Store hashed; enforce min length app. |
| `phone`     | VARCHAR(10)  | NOT NULL                    | Exactly 10 digits.                    |
| `address`   | VARCHAR(255) | NOT NULL                    |                                       |

Note: The presence of `specialty` on `patient` mirrors the current code and may be revisited if unintended.

---

### Table: `appointment`

Stores appointments between a doctor and a patient.

| Column             | Type     | Constraints                       | Notes                                 |
| :----------------- | :------- | :-------------------------------- | :------------------------------------ |
| `id`               | BIGINT   | PRIMARY KEY, AUTO_INCREMENT       |                                       |
| `doctor_id`        | BIGINT   | NOT NULL, FOREIGN KEY doctor(id)  |                                       |
| `patient_id`       | BIGINT   | NOT NULL, FOREIGN KEY patient(id) |                                       |
| `appointment_time` | DATETIME |                                   | Should be in the future when created. |
| `status`           | INT      | NOT NULL                          | 0 = Scheduled, 1 = Completed.         |

---

## MongoDB Documents

### Collection: `prescriptions`

Stores prescription details issued for an appointment.

Document structure:

```json
{
  "_id": "<ObjectId|string>",   // Unique identifier
  "patientName": "<string>",    // required, 3–100 chars
  "appointmentId": <number>,     // required, references MySQL appointment.id
  "medication": "<string>",     // required, 3–100 chars
  "doctorNotes": "<string>"     // optional, up to 200 chars
}
```

---

## Notes

- Consider adding UNIQUE constraints where appropriate (e.g., `doctor.email`, `patient.email`).
- Store passwords hashed using a strong algorithm (e.g., bcrypt/argon2); length allows for hashes and salts.
- Foreign key actions (ON DELETE/UPDATE) can be defined based on business rules (e.g., RESTRICT or CASCADE for appointments).
