# TenderFlow – Tender Management System

TenderFlow is an enterprise-grade web application built using Java (Servlets, JSP, JDBC) and MySQL, implementing the Model-View-Controller (MVC) architecture. It is designed to manage and streamline the process of floating tenders, logging empaneled vendor lists, submitting competitive bids, and tracking tender allocations.

## Technical Stack
*   **Frontend Client:** HTML5, CSS3, JavaScript, Bootstrap, JSP (JavaServer Pages)
*   **Backend logic:** Java J2EE, Servlets, JDBC, Maven
*   **Database:** MySQL

## System Features
*   **Dual-Role Portals:** Separate workspaces and security constraints for Administrators and empaneled Vendors.
*   **Tender Control (Admin):** Float new tenders (with titles, prices, descriptions, deadlines, locations) and view details of all submissions.
*   **Bidding Control (Vendor):** Empaneled vendors can view floating tenders, submit competitive bidding prices, and track allocation statuses.
*   **Tender Allocation Workflow:** Admin reviews all bids placed against a tender and selects the most suitable bid, updating the allocation state in MySQL.
*   **Security & Database Integrity:** Employs JDBC Prepared Statements to prevent SQL injection and transaction checks to ensure vendors can submit only one bid per tender.

---

## How to Set Up and Run

For complete setup guides, step-by-step instructions, schema diagrams, and developer Q&A, please refer to the **[PROJECT_GUIDE.md](PROJECT_GUIDE.md)** file.

### 1. Database Setup
1. Create a MySQL database named `tender_db`.
2. Import and run the `db.sql` database script.

### 2. Configure Database Connection
Open `src/main/resources/application.properties` and update variables to match your local database settings:
```properties
driverName = com.mysql.cj.jdbc.Driver
connectionString = jdbc:mysql://localhost:3306/tender_db
username = your_mysql_username
password = your_mysql_password
```

### 3. Deploy
1. Open Eclipse IDE (Enterprise Edition) and import the Maven project.
2. Build the project using:
   ```bash
   mvn clean install
   ```
3. Run the project on an Apache Tomcat Server (v8.5 or v9.0).
4. Access the web client at: `http://localhost:8080/tendermanagement/`

### 4. Default Credentials
*   **Admin Login:** `Admin` / `Admin`
*   **Vendor Login:** `shashi@gmail.com` / `shashi`

---

## Developer
- **Created by:** Shrirang Patil
- **GitHub:** [Shrirangpatil8600](https://github.com/Shrirangpatil8600)
