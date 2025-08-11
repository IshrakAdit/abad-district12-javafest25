## **1. Auth Service (Port 8081)**

**Purpose:** Authentication, authorization, farmer and admin accounts.

**Tables:**

### `users`

| Column        | Type                               | Notes             |
| ------------- | ---------------------------------- | ----------------- |
| user_id       | UUID / BIGINT                      | Primary Key       |
| username      | VARCHAR(100)                       | Unique            |
| email         | VARCHAR(150)                       | Unique            |
| password_hash | TEXT                               | bcrypt/argon hash |
| role          | ENUM('ANONYMOUS','FARMER','ADMIN') | Role-based access |
| created_at    | TIMESTAMP                          | Default now       |

### `farmer_profiles`

| Column          | Type          | Notes                   |
| --------------- | ------------- | ----------------------- |
| farmer_id       | UUID          | Matches `users.user_id` |
| full_name       | VARCHAR(150)  |                         |
| phone_number    | VARCHAR(20)   |                         |
| region          | VARCHAR(100)  |                         |
| farm_size       | DECIMAL(10,2) | In acres/hectares       |
| preferred_crops | TEXT          | JSON or CSV list        |
| created_at      | TIMESTAMP     |                         |
| updated_at      | TIMESTAMP     |                         |

---

## **2. Location Service (Port 8082)**

**Purpose:** Track farmer location for alerts, recommendations.

### `farmer_locations`

| Column       | Type          | Notes         |
| ------------ | ------------- | ------------- |
| location_id  | UUID          | PK            |
| farmer_id    | UUID          | Links to Auth |
| latitude     | DECIMAL(10,6) |               |
| longitude    | DECIMAL(10,6) |               |
| village      | VARCHAR(100)  |               |
| district     | VARCHAR(100)  |               |
| last_updated | TIMESTAMP     |               |

---

## **3. Alert Service (Port 8084)**

**Purpose:** Send personalized or regional alerts via SSE.

### `alerts`

| Column      | Type                                      | Notes           |
| ----------- | ----------------------------------------- | --------------- |
| alert_id    | UUID                                      | PK              |
| title       | VARCHAR(200)                              |                 |
| message     | TEXT                                      |                 |
| type        | ENUM('WEATHER','CROP','HEALTH','GENERAL') |                 |
| region      | VARCHAR(100)                              | Target region   |
| created_by  | UUID                                      | Admin ID        |
| created_at  | TIMESTAMP                                 |                 |
| valid_until | TIMESTAMP                                 | Optional expiry |

### `alert_reads`

| Column   | Type      | Notes             |
| -------- | --------- | ----------------- |
| alert_id | UUID      | PK part           |
| user_id  | UUID      | PK part           |
| read_at  | TIMESTAMP | When alert viewed |

---

## **4. Diary Service (8085 Spring Boot / 8095 Flask)**

**Purpose:** Store daily farming logs and AI summaries.

### `diary_entries`

| Column       | Type      | Notes                 |
| ------------ | --------- | --------------------- |
| entry_id     | UUID      | PK                    |
| farmer_id    | UUID      |                       |
| date         | DATE      | Farming day           |
| activities   | TEXT      | Original farmer log   |
| ai_summary   | TEXT      | Generated summary     |
| weather_info | JSON      | Optional weather data |
| created_at   | TIMESTAMP |                       |

---

## **5. Assistant Service (8086 Spring Boot / 8096 Flask)**

**Purpose:** AI-powered FAQ & conversational support.

### `assistant_queries`

| Column      | Type         | Notes                         |
| ----------- | ------------ | ----------------------------- |
| query_id    | UUID         | PK                            |
| farmer_id   | UUID         | Nullable for anonymous        |
| query_text  | TEXT         |                               |
| ai_response | TEXT         |                               |
| category    | VARCHAR(100) | e.g., Crop care, Market price |
| created_at  | TIMESTAMP    |                               |

---

## **6. Vision Service (8087 Spring Boot / 8097 Flask)**

**Purpose:** AI-based plant health scans.

### `vision_scans`

| Column          | Type         | Notes                  |
| --------------- | ------------ | ---------------------- |
| scan_id         | UUID         | PK                     |
| farmer_id       | UUID         | Nullable for anonymous |
| image_url       | TEXT         | File storage location  |
| detected_issue  | VARCHAR(200) | Disease name if found  |
| ai_confidence   | DECIMAL(5,2) | % confidence           |
| recommendations | TEXT         | AI-generated guidance  |
| created_at      | TIMESTAMP    |                        |

---

## **7. Content Service (8083 Spring Boot)**

**Purpose:** Learning materials & offline resources.

### `resources`

| Column      | Type         | Notes                    |
| ----------- | ------------ | ------------------------ |
| resource_id | UUID         | PK                       |
| title       | VARCHAR(200) |                          |
| description | TEXT         |                          |
| file_url    | TEXT         |                          |
| category    | VARCHAR(100) | e.g., PDF, Video         |
| is_public   | BOOLEAN      | Anonymous access if true |
| created_by  | UUID         | Admin ID                 |
| created_at  | TIMESTAMP    |                          |

---

## **8. Analytics Service (8088 Spring Boot / 8098 Flask)**

**Purpose:** Logs, trends, anomaly detection.

### `user_activity_logs`

| Column       | Type         | Notes                              |
| ------------ | ------------ | ---------------------------------- |
| log_id       | UUID         | PK                                 |
| user_id      | UUID         | Nullable for anonymous             |
| service_name | VARCHAR(50)  | e.g., "diary", "assistant"         |
| action       | VARCHAR(200) | e.g., "create_entry", "view_alert" |
| metadata     | JSON         | Extra info                         |
| created_at   | TIMESTAMP    |                                    |

---

## **9. Admin Service (8089 Spring Boot)**

**Purpose:** Central admin dashboard to manage services.

### `admin_actions`

| Column       | Type        | Notes |
| ------------ | ----------- | ----- |
| action_id    | UUID        | PK    |
| admin_id     | UUID        |       |
| service_name | VARCHAR(50) |       |
| action       | TEXT        |       |
| created_at   | TIMESTAMP   |       |
