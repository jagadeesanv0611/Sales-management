# Sales Intelligence Hub

A **Branch-Based Sales Management System** built with **MySQL**, **Python** and **Streamlit**. It tracks branch-wise sales, handles split payments, and automatically keeps received and pending amounts accurate using database triggers and generated columns, all behind a role-based login.

**Domain:** Sales Analytics & Financial Tracking

---


## Problem Statement

Organizations that operate across multiple branches often struggle with tracking sales, collecting payments, monitoring pending amounts and keeping clean financial records. Manual tracking leads to inconsistent data, duplicate entries, incorrect payment calculations and a lack of transparency.

This project provides a structured, automated alternative that:

- Stores branch-wise sales records
- Automatically calculates pending amounts
- Supports split (partial) payments per sale
- Updates received and pending amounts through database triggers
- Offers an interactive Streamlit dashboard for Super Admin and Admin users

## Features

- **Secure login** with session-based authentication and logout
- **Role-based access control** (Super Admin and Admin)
- **Add sales entries** (branch-restricted for Admin)
- **Add payment splits** against existing sales (Cash / UPI / Card)
- **Sales report** with branch filter
- **Pending payments** view
- **Sales status tracking** (Open vs Close)
- **Payment method summary**
- **Financial KPI cards:** total sales, received amount, pending amount
- **Predefined SQL query runner** (20 analysis queries)
- **Automated calculations** via triggers and a generated column, with no manual arithmetic

## Skills

- Python
- MySQL database design
- SQL queries & aggregations
- Triggers & generated columns
- Streamlit web application development
- Payment tracking logic
- Relational database management
- Role-based access control

## Tech Stack

| Layer | Technology |
| --- | --- |
| Language | Python 3.x |
| Database | MySQL |
| Web App | Streamlit |
| Data Handling | pandas, mysql-connector-python (or SQLAlchemy / PyMySQL) |

> Adjust the library names above to match your `requirements.txt`.

## Database Design

**Database name:** `Sales Management System`

### Relationships

```
branches (1) ──► (Many) customer_sales
customer_sales (1) ──► (Many) payment_splits
branches (1) ──► (Many) users
```

### branches:
Stores the business branches.

| Column | Type | Description |
| --- | --- | --- |
| branch_id | INT (PK, AI) | Unique branch identifier |
| branch_name | VARCHAR(100) | Branch name (e.g. Chennai, Delhi) |
| branch_admin_name | VARCHAR(100) | Name of the branch administrator |

### `customer_sales`

Main financial table holding every sales transaction.

| Column | Type | Description |
| --- | --- | --- |
| sale_id | INT (PK, AI) | Unique sale ID |
| branch_id | INT (FK) | References `branches` |
| date | DATE | Date of sale |
| name | VARCHAR(100) | Customer name |
| mobile_number | VARCHAR(15), UNIQUE | Customer mobile number |
| product_name | VARCHAR(30) | Product code: DS / DA / BA / FSD |
| gross_sales | DECIMAL(12,2) | Total sale amount |
| received_amount | DECIMAL(12,2) | Auto-updated via trigger |
| pending_amount | DECIMAL(12,2), **generated** | `gross_sales - received_amount` |
| status | ENUM('Open','Close') | Sale completion status |

### `users`

Login accounts for the application.

| Column | Type | Description |
| --- | --- | --- |
| user_id | INT (PK, AI) | Unique user ID |
| username | VARCHAR(100) | Login username |
| password | VARCHAR(255) | Login password |
| branch_id | INT (FK) | Assigned branch |
| role | ENUM('Super Admin','Admin') | Access level |
| email | VARCHAR(255), UNIQUE | User email |

### `payment_splits`

Handles multiple payments for a single sale.

| Column | Type | Description |
| --- | --- | --- |
| payment_id | INT (PK, AI) | Unique payment ID |
| sale_id | INT (FK) | References `customer_sales` |
| payment_date | DATE | Date of payment |
| amount_paid | DECIMAL(12,2) | Amount paid |
| payment_method | VARCHAR(50) | Cash / UPI / Card |

### Schema Highlights

- Primary keys and foreign keys enforce referential integrity
- `ENUM` constraints on `status` and `role`
- `UNIQUE` constraints on `mobile_number` and `email`
- `pending_amount` is a **generated column**, so it can never drift out of sync

## Automation Logic (Triggers)

An **AFTER INSERT** trigger on `payment_splits` keeps the sales table in sync:

1. Calculates the total `amount_paid` for the sale
2. Updates `received_amount` in `customer_sales`
3. `pending_amount` adjusts automatically through the generated column

```sql
DELIMITER //

CREATE TRIGGER trg_update_received_after_payment
AFTER INSERT ON payment_splits
FOR EACH ROW
BEGIN
    UPDATE customer_sales
    SET received_amount = (
        SELECT COALESCE(SUM(amount_paid), 0)
        FROM payment_splits
        WHERE sale_id = NEW.sale_id
    )
    WHERE sale_id = NEW.sale_id;
END//

DELIMITER ;
```

> Replace this snippet with the exact trigger from your SQL script if the name or logic differs.

**Result:** no manual financial calculation, accurate pending tracking and real-time payment updates.

## Role-Based Access Control

| Capability | Super Admin | Admin |
| --- | :---: | :---: |
| View sales from all branches | ✅ | ❌ (own branch only) |
| Add sales entries | ✅ any branch | ✅ own branch only |
| Add payment splits | ✅ | ✅ own branch sales only |
| View all payment split details | ✅ | ❌ (own branch only) |
| Full business analytics & reports | ✅ | Branch-level reports |
| Track pending payments | ✅ all branches | ✅ own branch |

Authentication details:

- Credentials are validated against the `users` table
- The dashboard is only accessible after a successful login
- Session state maintains the logged-in user
- A logout option ends the session
- Unauthenticated users cannot reach any application feature

## Dashboard & Reporting

**Dashboard pages**

- Login / Logout
- Add Sales Entry
- Add Payment Split
- Sales Report (with branch filter)
- Pending Payments
- Sales Status (Open vs Close)
- Payment Method Summary
- SQL Query Runner (predefined queries)
- Financial KPI Summary

**Insights provided**

- Overall business revenue
- Branch-wise sales comparison
- Total received vs total pending amount
- Pending collection percentage
- Payment method analysis (Cash / UPI / Card)
- Sales trends and growth tracking
- Branch performance summary

## SQL Analysis Queries

Demo data must be loaded into MySQL before running the analysis queries.

**Dataset:** [Sales Management System Datasets](https://drive.google.com/drive/folders/1gLlO1TJzti2CEt8HA0mfKMq8EZM5L23d?usp=sharing)

| Category | Queries |
| --- | --- |
| **Basic** | All records from `customer_sales`, `branches` and `payment_splits`; sales with status `Open`; sales for the Chennai branch |
| **Aggregation** | Total gross sales; total received amount; total pending amount; sales count per branch; average gross sales |
| **Join-based** | Sales with branch name; sales with total payment received; branch-wise gross sales; sales with payment method; sales with branch admin name |
| **Financial tracking** | Sales with pending amount > 5000; top 3 highest gross sales; branch with highest total gross sales |

The queries are available in `[queries file name].sql` and from the **SQL Query Runner** page in the app.

## Project Structure

> Update this tree to match your repository.

```
Sales-Intelligence-Hub/
├── app.py                  # Streamlit application entry point
├── database/
│   ├── schema.sql          # Tables, constraints, generated column
│   ├── triggers.sql        # AFTER INSERT trigger on payment_splits
│   └── sample_data.sql     # Demo data
├── queries/
│   └── analysis_queries.sql
├── requirements.txt
└── README.md
```

## Installation & Setup

### Prerequisites

- Python 3.9 or higher
- MySQL Server 8.x
- pip

### Steps

1. **Clone the repository**

   ```bash
   git clone <your-repo-url>
   cd Sales-Intelligence-Hub
   ```

2. **Create and activate a virtual environment** (optional but recommended)

   ```bash
   python -m venv venv
   source venv/bin/activate      # Windows: venv\Scripts\activate
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **Set up the database**

   ```bash
   mysql -u <user> -p < database/schema.sql
   mysql -u <user> -p < database/triggers.sql
   mysql -u <user> -p < database/sample_data.sql
   ```

5. **Configure the database connection**

   Update the connection settings (host, user, password, database) in `[config file or app.py]`.

6. **Run the application**

   ```bash
   streamlit run app.py
   ```

   The app opens at `http://localhost:8501`.

## Usage

1. Log in with a Super Admin or Admin account.
2. **Admin:** add sales for your branch, then record payments against them.
3. **Super Admin:** switch between branches with the branch filter and review overall analytics.
4. Watch **received** and **pending** amounts update automatically as payments are added.
5. Use the **SQL Query Runner** for the predefined analysis queries.
6. Click **Logout** when finished.

**Demo credentials** *(for evaluation only)*

| Role | Username | Password |
| --- | --- | --- |
| Super Admin | `[username]` | `[password]` |
| Admin | `[username]` | `[password]` |

## Images:
### Login Page:
<img width="1012" height="483" alt="image" src="https://github.com/user-attachments/assets/b9adfcf2-be6a-450c-8145-a29f88a6dd88" />

### Super admin Login:
<img width="1005" height="475" alt="image" src="https://github.com/user-attachments/assets/79af565f-8f0a-4eb9-b132-d592fe5e9b1b" />

### Dashboard Page:
#### Super admin:
<img width="693" height="330" alt="image" src="https://github.com/user-attachments/assets/bb1a09af-b0bb-47d2-94fe-46269c051afe" />

#### Branch admin:
<img width="783" height="372" alt="image" src="https://github.com/user-attachments/assets/aac86e9b-9948-4c96-9737-6c3e15daadbc" />

#### Filter by branch:
<img width="336" height="250" alt="image" src="https://github.com/user-attachments/assets/e3fb202a-80ee-400e-bb53-cb3a3e859afb" />


### Sales Record:
#### Superadmin:
<img width="1018" height="390" alt="image" src="https://github.com/user-attachments/assets/d087a993-0458-4b20-81a2-3fd1cf3e5127" />

#### Branch admin:
<img width="971" height="420" alt="image" src="https://github.com/user-attachments/assets/6fdb56df-3f3f-4752-8751-eebbd4d89cd2" />
















