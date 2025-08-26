# SmartCare Clinic: Database Schema Design

This document outlines the proposed database schema for the SmartCare Clinic management system. The design uses a hybrid approach, leveraging both MySQL for structured, relational data and MongoDB for flexible, document-based data.

---

## MySQL Database Design

The MySQL database will store the core operational data of the clinic, which is well-defined and relational in nature.

### Table: `users`

Stores common information for all user types (Admins, Doctors, Patients) and manages roles.

| Column          | Data Type                            | Constraints                                             | Description                      |
| :-------------- | :----------------------------------- | :------------------------------------------------------ | :------------------------------- |
| `id`            | `INT`                                | `Primary Key`, `Auto Increment`                         | Unique identifier for each user. |
| `first_name`    | `VARCHAR(100)`                       | `Not Null`                                              | User's first name.               |
| `last_name`     | `VARCHAR(100)`                       | `Not Null`                                              | User's last name.                |
| `email`         | `VARCHAR(255)`                       | `Not Null`, `Unique`                                    | User's email address for login.  |
| `password_hash` | `VARCHAR(255)`                       | `Not Null`                                              | Hashed password for security.    |
| `phone_number`  | `VARCHAR(20)`                        |                                                         | User's contact phone number.     |
| `role`          | `ENUM('ADMIN', 'DOCTOR', 'PATIENT')` | `Not Null`                                              | User's role in the system.       |
| `created_at`    | `DATETIME`                           | `Default CURRENT_TIMESTAMP`                             | Timestamp of user creation.      |
| `updated_at`    | `DATETIME`                           | `Default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | Timestamp of last update.        |

### Table: `doctors`

Stores information specific to doctors, linked to the `users` table.

| Column         | Data Type | Constraints                     | Description                            |
| :------------- | :-------- | :------------------------------ | :------------------------------------- |
| `id`           | `INT`     | `Primary Key`, `Auto Increment` | Unique identifier for doctor profile.  |
| `user_id`      | `INT`     | `Foreign Key users(id)`         | Links to the corresponding user entry. |
| `specialty_id` | `INT`     | `Foreign Key specialties(id)`   | Links to the doctor's specialty.       |
| `bio`          | `TEXT`    |                                 | A short biography of the doctor.       |

### Table: `patients`

Stores information specific to patients, linked to the `users` table.

| Column          | Data Type      | Constraints                     | Description                            |
| :-------------- | :------------- | :------------------------------ | :------------------------------------- |
| `id`            | `INT`          | `Primary Key`, `Auto Increment` | Unique identifier for patient profile. |
| `user_id`       | `INT`          | `Foreign Key users(id)`         | Links to the corresponding user entry. |
| `date_of_birth` | `DATE`         | `Not Null`                      | Patient's date of birth.               |
| `address`       | `VARCHAR(255)` |                                 | Patient's mailing address.             |

### Table: `specialties`

Stores the list of medical specialties available at the clinic.

| Column | Data Type      | Constraints                     | Description                          |
| :----- | :------------- | :------------------------------ | :----------------------------------- |
| `id`   | `INT`          | `Primary Key`, `Auto Increment` | Unique identifier for the specialty. |
| `name` | `VARCHAR(100)` | `Not Null`, `Unique`            | The name of the medical specialty.   |

### Table: `appointments`

Stores information about scheduled appointments between doctors and patients.

| Column             | Data Type                                     | Constraints                       | Description                             |
| :----------------- | :-------------------------------------------- | :-------------------------------- | :-------------------------------------- |
| `id`               | `INT`                                         | `Primary Key`, `Auto Increment`   | Unique identifier for each appointment. |
| `doctor_id`        | `INT`                                         | `Foreign Key doctors(id)`         | The doctor for the appointment.         |
| `patient_id`       | `INT`                                         | `Foreign Key patients(id)`        | The patient for the appointment.        |
| `appointment_time` | `DATETIME`                                    | `Not Null`                        | The scheduled date and time.            |
| `status`           | `ENUM('SCHEDULED', 'COMPLETED', 'CANCELLED')` | `Not Null`, `Default 'SCHEDULED'` | Current status of the appointment.      |
| `created_at`       | `DATETIME`                                    | `Default CURRENT_TIMESTAMP`       | Timestamp of appointment creation.      |

### Table: `doctor_availability`

Stores the available time slots for each doctor.

| Column       | Data Type  | Constraints                     | Description                            |
| :----------- | :--------- | :------------------------------ | :------------------------------------- |
| `id`         | `INT`      | `Primary Key`, `Auto Increment` | Unique identifier for the time slot.   |
| `doctor_id`  | `INT`      | `Foreign Key doctors(id)`       | The doctor whose availability this is. |
| `start_time` | `DATETIME` | `Not Null`                      | The start of the available slot.       |
| `end_time`   | `DATETIME` | `Not Null`                      | The end of the available slot.         |
| `is_booked`  | `BOOLEAN`  | `Not Null`, `Default false`     | Indicates if the slot is taken.        |

---

## MongoDB Collection Design

The MongoDB database will store data that is less structured or benefits from a flexible schema, such as prescriptions and clinical notes.

### Collection: `prescriptions`

This collection stores prescription details issued by doctors during appointments. The structure can vary, for example, some medications might require refills while others don't.

```json
{
  "_id": "ObjectId('64c8a9f3b4d3e2c1f8a7b1c2')",
  "appointment_id": 101,
  "patient_id": 1,
  "doctor_id": 1,
  "issue_date": "2025-08-26T10:30:00Z",
  "medications": [
    {
      "name": "Lisinopril",
      "dosage": "10mg",
      "frequency": "Once daily",
      "instructions": "Take in the morning with food."
    },
    {
      "name": "Atorvastatin",
      "dosage": "20mg",
      "frequency": "Once daily at bedtime"
    }
  ],
  "refill_information": {
    "is_refillable": true,
    "refill_count": 3,
    "next_refill_date": "2025-09-26"
  },
  "doctor_notes": "Patient to monitor blood pressure daily. Follow-up in 3 months."
}
```

### Collection: `clinical_notes`

This collection stores clinical notes and observations made by doctors for a patient's record. The content is free-form and can be updated over time.

```json
{
  "_id": "ObjectId('64c8aa67b4d3e2c1f8a7b1c3')",
  "appointment_id": 101,
  "patient_id": 1,
  "doctor_id": 1,
  "note_timestamp": "2025-08-26T11:00:00Z",
  "subjective_notes": "Patient reports feeling well, no complaints of chest pain or shortness of breath.",
  "objective_notes": {
    "blood_pressure": "120/80 mmHg",
    "heart_rate": "72 bpm",
    "weight_kg": 85
  },
  "assessment": "Hypertension, well-controlled on current medication.",
  "plan": "Continue current medication regimen. Encourage continued diet and exercise. Schedule follow-up appointment in 3 months."
}
```
