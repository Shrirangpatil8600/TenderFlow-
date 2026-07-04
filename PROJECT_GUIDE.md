# Project Guide: Online Tender Management System

Welcome! This document is a comprehensive guide to understanding, setting up, and deploying the **Online Tender Management System** project. It covers the project architecture, database design, module-level functionalities, runtime setup, and key technical concepts with detailed explanations.

---

## 1. Project Overview

The **Online Tender Management System** is a J2EE (Java Enterprise Edition) web application built to automate and manage the process of publishing tenders, submitting vendor bids, and assigning contracts. It ensures transparency, efficiency, and structured record-keeping for companies seeking merchandise or services.

The application is structured using the **MVC (Model-View-Controller)** design pattern with a **Repository Pattern (DAO Interfaces & Implementations)** for database operations. It includes two key user portals:
1. **Admin Portal**: For administrative control over vendors, notice boards, tenders, and bid decisions.
2. **Vendor Portal**: For registered vendors to browse active tenders, place bids, check assignment status, and view their bid history.

---

## 2. Technical Stack

*   **Front-End (View)**: HTML, CSS, JavaScript, Bootstrap for responsive designs, and JSP (JavaServer Pages) for dynamic server-side rendering.
*   **Back-End Controller**: Java Servlets (utilizing standard URL mappings).
*   **Dependency Management & Build Tool**: Apache Maven (configured via `pom.xml`).
*   **Database (Model)**: MySQL Database.
*   **Database Helper**: `DBUtil.java` using JDBC API and resource bundles for externalized database configurations.
*   **Application Server**: Apache Tomcat 9.0 or older (required because the project uses the classic `javax.servlet` API namespace. It is not compatible with Tomcat 10+ unless the imports are migrated to the `jakarta` namespace).

---

## 3. Directory & File Structure

Here is a guide to the project layout and the role of key files:

```text
Tender-Management-System-master/
├── DataBase/                       # Database scripts and guides
│   ├── tender.sql                  # MySQL SQL Dump containing schema and dummy data
│   ├── mysql_create_tables.md      # DDL script instructions
│   └── how-to-import-sql-dump-file.md # Walkthrough on importing database dumps
├── Presentation/                   # Additional visual presentation assets
└── tendermanagement/               # Maven Web Project Root
    ├── pom.xml                     # Maven project configuration (dependencies, build compiler plugins)
    ├── WebContent/                 # Front-end View pages
    │   ├── css/, js/, fonts/       # Bootstrap, Custom stylesheet, and font resources
    │   ├── images/                 # System UI images
    │   ├── index.jsp               # Main landing portal page
    │   ├── login.jsp               # Common login form for Admin and Vendors
    │   ├── register.jsp            # Vendor sign-up registration form
    │   ├── adminHome.jsp           # Admin dashboard home
    │   ├── createTender.jsp        # Form for admin to create and publish a new tender
    │   ├── viewTender.jsp          # Admin directory showing all published tenders
    │   ├── viewTenderBids.jsp      # Admin view listing bids submitted for a specific tender
    │   ├── addNotice.jsp           # Admin board to post announcements
    │   ├── vendorHome.jsp          # Vendor dashboard home
    │   ├── vendorSearchTender.jsp  # Search directory for vendors to look up active tenders
    │   ├── bidTenderForm.jsp       # Vendor bid submission form
    │   ├── bidHistory.jsp          # Vendor log of historical bids placed
    │   ├── viewProfile.jsp         # Vendor profile details
    │   └── WEB-INF/                # Java web application configuration
    │       └── web.xml             # Deployment descriptor
    └── src/                        # Java Source Code
        ├── dbdetails.properties    # Configuration file for MySQL credentials
        └── com/hit/
            ├── beans/              # Data Transfer Objects (POJOs/Model beans)
            │   ├── TenderBean.java # Represents Tender records
            │   ├── VendorBean.java # Represents Vendor records
            │   ├── BidderBean.java # Represents Bid records
            │   ├── NoticeBean.java # Represents notice announcements
            │   └── TenderStatusBean.java # Represents the assigned status of tenders
            ├── dao/                # Data Access Layer (Repository Pattern)
            │   ├── TenderDao.java / TenderDaoImpl.java
            │   ├── VendorDao.java / VendorDaoImpl.java
            │   ├── BidderDao.java / BidderDaoImpl.java
            │   └── NoticeDao.java / NoticeDaoImpl.java
            ├── srv/                # Controller Servlets
            │   ├── LoginSrv.java   # Processes common authentications
            │   ├── RegisterSrv.java# Registers new vendor accounts
            │   ├── CreateTenderSrv.java # Publishes new tender records
            │   ├── BidTenderSrv.java # Processes vendor bid submissions
            │   ├── AcceptBidSrv.java # Admin logic to assign tender to a vendor
            │   ├── RejectBidSrv.java # Admin logic to decline vendor bids
            │   └── LogoutSrv.java   # Cleans sessions and signs out users
            └── utility/            # Connection Helper utilities
                ├── DBUtil.java     # Establishes connections using properties
                └── IDUtil.java     # Automatically generates unique custom IDs
```

---

## 4. Module-wise Working

### A. Administrator Module
*   **Tender Management**: Admin creates tenders (`createTender.jsp`) by specifying name, type, price, description, deadline, and location. This runs through `CreateTenderSrv.java` which auto-generates a unique Tender ID (using `IDUtil`) and saves it.
*   **Notice Board**: Admin can add (`addNotice.jsp`), update, or delete notifications to keep vendors informed about updates, modifications, or schedule changes.
*   **Vendor Management**: Admin has complete visibility over the empaneled vendors. `adminViewVendor.jsp` list profiles, which helps check vendor details.
*   **Bid Evaluation & Decision**: 
    *   Admin selects a tender to view all submitted bids (`viewTenderBids.jsp` backed by `BidderDao.getAllBiddersByTid()`).
    *   The admin reviews the submitted bidding amounts and can click **Accept** (`AcceptBidSrv.java`) or **Reject** (`RejectBidSrv.java`).
    *   Accepting a bid updates its status to "Accepted", updates other bids for that tender to "Rejected", and writes an entry to `tenderstatus` assigning the tender to that vendor.

### B. Vendor Module
*   **Registration & Login**: Vendors register via `register.jsp` and authenticate through `login.jsp`. The servlet `LoginSrv.java` distinguishes between the roles and redirects vendors to `vendorHome.jsp`.
*   **Tender Search & Browsing**: Vendors search for active tenders by name, type, or location on `vendorSearchTender.jsp`.
*   **Bid Placement**: 
    *   A vendor can select an active tender and bid on it using `bidTenderForm.jsp`. 
    *   Each vendor is allowed to submit only one bid per tender. The system checks if a bid already exists to enforce this rule.
*   **Bid History & Track Status**: Vendors view their historical bids and current status (e.g., Pending, Accepted, or Rejected) on `bidHistory.jsp` (calling `BidderDao.getBidderHistory()`).

---

## 5. Database Design & Relationships

The system uses a relational database schema mapping relationships between tenders, vendors, and bids:

```sql
CREATE DATABASE tender;
USE tender;

-- 1. Notice Table: Admin announcements
CREATE TABLE notice (
    id INT(3) NOT NULL AUTO_INCREMENT,
    title VARCHAR(35),
    info VARCHAR(300),
    PRIMARY KEY(id)
);

-- 2. Vendor Table: Empaneled vendor profiles
CREATE TABLE vendor (
    vid VARCHAR(15) PRIMARY KEY,
    password VARCHAR(20) NOT NULL,
    vname VARCHAR(30) NOT NULL,
    vmob VARCHAR(12) NOT NULL,
    vemail VARCHAR(40) NOT NULL UNIQUE,
    company VARCHAR(15) NOT NULL,
    address VARCHAR(100) NOT NULL
);

-- 3. Tender Table: Advertised projects
CREATE TABLE tender (
    tid VARCHAR(15) PRIMARY KEY,
    tname VARCHAR(40) NOT NULL,
    ttype VARCHAR(20) NOT NULL,
    tprice INT NOT NULL,
    tdesc VARCHAR(300) NOT NULL,
    tdeadline DATE NOT NULL,
    tloc VARCHAR(70) NOT NULL
);

-- 4. Bidder Table: Bids submitted by vendors on tenders
CREATE TABLE bidder (
    bid VARCHAR(15) PRIMARY KEY,
    vid VARCHAR(15),
    tid VARCHAR(15),
    bidamount INT NOT NULL,
    deadline DATE NOT NULL,
    status VARCHAR(10) DEFAULT 'Pending',
    FOREIGN KEY (vid) REFERENCES vendor(vid) ON DELETE CASCADE,
    FOREIGN KEY (tid) REFERENCES tender(tid) ON DELETE CASCADE
);

-- 5. Tender Status Table: Log of assigned tenders
CREATE TABLE tenderstatus (
    tid VARCHAR(15) PRIMARY KEY,
    bid VARCHAR(15),
    status VARCHAR(15) NOT NULL,
    vid VARCHAR(15),
    FOREIGN KEY (tid) REFERENCES tender(tid) ON DELETE CASCADE,
    FOREIGN KEY (bid) REFERENCES bidder(bid) ON DELETE CASCADE,
    FOREIGN KEY (vid) REFERENCES vendor(vid) ON DELETE CASCADE
);
```

---

## 6. How to Set Up the Project

Follow these steps to deploy and run the project locally on your machine.

### Step 1: Install Prerequisites
1.  **Java Development Kit (JDK)**: Install JDK 8 (which matches target compilation settings in Maven) or higher.
2.  **Apache Tomcat Server**: Download and install **Tomcat 8.5 or 9.0**.
    *   *Important Note*: Do not use Tomcat 10+. Tomcat 10 uses the `jakarta.*` packages, but this project is compiled with the `javax.*` servlet namespace.
3.  **MySQL Server**: Ensure MySQL Server is running locally.

### Step 2: Set Up Database
1.  Open your MySQL terminal or GUI client.
2.  Log in to your local account and execute the SQL dump file [tender.sql](file:///c:/Users/Gaurav%20Gawali/Documents/GitHub/Tender-Management-System-master/DataBase/tender.sql) located inside the `DataBase` folder to create the database structure with seed data.

### Step 3: Configure Database Properties
Open [dbdetails.properties](file:///c:/Users/Gaurav%20Gawali/Documents/GitHub/Tender-Management-System-master/tendermanagement/src/dbdetails.properties) and update the credentials to match your local database settings:
```properties
driverName = com.mysql.cj.jdbc.Driver
connectionString = jdbc:mysql://localhost:3306/tender
username = your_mysql_username
password = your_mysql_password
```

### Step 4: Import Project & Compile with Maven
1.  Open Eclipse IDE (Enterprise Edition).
2.  Select `File` -> `Import...` -> `Maven` -> `Existing Maven Projects`.
3.  Choose the root directory `tendermanagement/` (which contains `pom.xml`) and click **Finish**.
4.  Right-click on the project -> `Run As` -> `Maven Build...`
    *   Enter `clean install` in the **Goals** field and click **Run**.
5.  Right-click on the project -> `Maven` -> `Update Project...` -> check `Force Update of Snapshots/Releases` -> Click **OK**.

### Step 5: Deploy on Tomcat Server
1.  Right-click on the project -> `Run As` -> `Run on Server`.
2.  Choose your configured **Tomcat v9.0** server.
3.  Add the project to the server list and click **Finish**.
4.  The application will open in your browser (default: `http://localhost:8080/tendermanagement/` or the configured port).
5.  **Default logins**:
    *   *Admin*: Email = `Admin`, Password = `Admin` (case-sensitive)
    *   *Default Vendor*: Email = `shashi@gmail.com`, Password = `shashi`

---

## 7. Technical FAQ & Deep-Dive

This section provides detailed answers to key technical design and operational questions about the system:

### Q1: Explain how database connections are optimized in `DBUtil.java`?
**Answer**:
> "`DBUtil.java` implements a connection optimization logic by preventing redundant connection initialization. It maintains a static variable `Connection conn` and checks:
> `if(conn == null || conn.isClosed()) { ... }`
> This ensures that the application reuses the established connection instead of generating a new TCP socket handshakes for every query. It also exposes utility methods to safely close resources (`PreparedStatement`, `ResultSet`, and `Connection`) within `try-catch` blocks, preventing resource leaks."

### Q2: What is the benefit of the Repository Pattern (Dao interface + Impl class) used here?
**Answer**:
> "Using interfaces (e.g., `TenderDao.java`) decoupled from their JDBC implementations (e.g., `TenderDaoImpl.java`) provides abstraction and modularity:
> 1.  It defines a strict contract of database operations without tying the controller servlets to raw SQL statements.
> 2.  It allows swapping the underlying database layer (e.g., migrating from JDBC to Hibernate/JPA, or switching from MySQL to Oracle) without altering any servlet controllers. Servlets only invoke methods defined on the interface."

### Q3: How does the unique ID generation system work in `IDUtil.java`?
**Answer**:
> "Instead of relying on database auto-increment values for complex business entities like Tenders (`tid`) or Bids (`bid`), the system uses a custom utility `IDUtil.java`. 
>
> It combines prefix characters representing the entity types (e.g., 'T' for Tender, 'V' for Vendor, 'B' for Bidder) with timestamp sequences generated from the system clock down to the second. This prevents enumeration attacks, yields human-readable IDs, and guarantees uniqueness across distributed databases without database locks."

### Q4: How is the database integrity maintained when deleting a tender or vendor?
**Answer**:
> "Database integrity is maintained at the SQL schema level using foreign keys configured with **`ON DELETE CASCADE`** constraints.
>
> For example, in the `bidder` table, `vid` references `vendor(vid)` and `tid` references `tender(tid)` with `ON DELETE CASCADE`. If an admin deletes a tender from the `tender` table, MySQL automatically purges all corresponding bids submitted for that tender in the `bidder` table and assignments in the `tenderstatus` table. This prevents orphan rows and database inconsistencies."

### Q5: How is authentication state tracked across pages?
**Answer**:
> "Authentication state is tracked using servlet container-managed HTTP sessions (`HttpSession` interface).
>
> Upon successful validation in `LoginSrv.java`, the servlet retrieves the session and stores the email: `session.setAttribute("user", email)`. Every secure menu file (`adminMenu.jsp` and `vendorMenu.jsp`) reads this session attribute. If the attribute is missing, the page forwards the client back to the login page. When logging out, `LogoutSrv.java` invalidates the session using `session.invalidate()`, clearing all session state."

---
*Created by Shrirang Patil.*
