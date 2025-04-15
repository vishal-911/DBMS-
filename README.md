# Merchant Navy Portal Database Theory

## Database Overview

The Merchant Navy Portal is a comprehensive system designed to serve the maritime industry by connecting seafarers, recruiting companies, and administrative personnel. The database facilitates job searching, application management, ship tracking, training course administration, and communication between various stakeholders in the merchant navy ecosystem.

This relational database employs a MySQL architecture to manage the complex relationships between users (seafarers, recruiters, administrators), companies, ships, job postings, applications, training courses, and inter-user communications. The database design follows normalization principles to minimize redundancy while maintaining data integrity through primary and foreign key relationships.


## Tables Description

1. **Users**: Central entity that stores information about all system users with a discriminator field (UserType) to distinguish between seafarers, administrators, and recruiters.

2. **Companies**: Stores shipping company details linked to recruiter users who manage their profiles.

3. **Ships**: Contains information about vessels owned by companies, including specifications and operational details.

4. **ShipSchedules**: Tracks the movement of ships between ports with departure and arrival information.

5. **Jobs**: Stores job postings from companies for positions on specific ships, including requirements, compensation, and status.

6. **Applications**: Manages the job application process, linking seafarers to job postings with application materials and status tracking.

7. **TrainingCourses**: Contains information about maritime courses available for certification and skill development.

8. **CourseEnrollments**: Tracks seafarer enrollment in training courses with payment and enrollment status.

9. **Messages**: Facilitates communication between users with message content, read status, and threading capabilities.

## Implementation Details

The database implementation uses:

1. **Data Integrity Constraints**:
   - Primary keys for unique identification (e.g., UserID, CompanyID)
   - Foreign keys for relationship enforcement
   - CHECK constraints (e.g., UserType values limited to 'Seafarer', 'Admin', 'Recruiter')
   - UNIQUE constraints (e.g., Username, Email)

2. **Data Types**:
   - VARCHAR for variable-length text with reasonable limits
   - TEXT for longer content (descriptions, requirements)
   - DECIMAL for precise numeric values (salaries, fees)
   - DATE/DATETIME for temporal data
   - BOOLEAN for binary states (IsRead)

3. **Default Values**:
   - Current timestamp for creation dates
   - Default status values (e.g., 'Pending', 'Scheduled', 'Open')

4. **Indexing**:
   - Primary keys are automatically indexed
   - Foreign keys should be indexed for performance in join operations

## Sample Queries

Here are some practical SQL queries for common operations in the Merchant Navy Portal:

### 1. Find all open jobs for a specific ship type with salary above a threshold:

```sql
SELECT j.JobID, j.JobTitle, c.CompanyName, s.ShipName, s.ShipType, j.Salary
FROM Jobs j
JOIN Companies c ON j.CompanyID = c.CompanyID
JOIN Ships s ON j.ShipID = s.ShipID
WHERE s.ShipType = 'Container Ship' 
  AND j.Salary > 5000.00
  AND j.Status = 'Open'
ORDER BY j.Salary DESC;
```

### 2. Get application statistics by status for a company:

```sql
SELECT j.CompanyID, c.CompanyName, a.Status, COUNT(*) AS ApplicationCount
FROM Applications a
JOIN Jobs j ON a.JobID = j.JobID
JOIN Companies c ON j.CompanyID = c.CompanyID
WHERE j.CompanyID = 1
GROUP BY j.CompanyID, c.CompanyName, a.Status;
```

### 3. Find available training courses in a date range:

```sql
SELECT CourseID, CourseName, StartDate, EndDate, Location, 
       Fees, MaxCapacity,
       (SELECT COUNT(*) FROM CourseEnrollments WHERE CourseID = TrainingCourses.CourseID) AS CurrentEnrollments
FROM TrainingCourses
WHERE StartDate BETWEEN '2023-07-01' AND '2023-08-31'
  AND (SELECT COUNT(*) FROM CourseEnrollments WHERE CourseID = TrainingCourses.CourseID) < MaxCapacity;
```

### 4. Retrieve a seafarer's job application history with status:

```sql
SELECT a.ApplicationID, j.JobTitle, c.CompanyName, s.ShipName, 
       a.ApplicationDate, a.Status
FROM Applications a
JOIN Jobs j ON a.JobID = j.JobID
JOIN Companies c ON j.CompanyID = c.CompanyID
JOIN Ships s ON j.ShipID = s.ShipID
WHERE a.UserID = 4
ORDER BY a.ApplicationDate DESC;
```

### 5. Find ships scheduled to arrive at a specific port in the next month:

```sql
SELECT s.ShipID, s.ShipName, c.CompanyName, ss.DeparturePort, 
       ss.ArrivalPort, ss.DepartureDate, ss.ArrivalDate
FROM ShipSchedules ss
JOIN Ships s ON ss.ShipID = s.ShipID
JOIN Companies c ON s.CompanyID = c.CompanyID
WHERE ss.ArrivalPort = 'Singapore'
  AND ss.ArrivalDate BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 30 DAY)
ORDER BY ss.ArrivalDate;
```

### 6. Get unread messages for a user:

```sql
SELECT m.MessageID, u.FullName AS Sender, m.Subject, 
       m.SentDate, m.MessageContent
FROM Messages m
JOIN Users u ON m.SenderID = u.UserID
WHERE m.ReceiverID = 4
  AND m.IsRead = 0
ORDER BY m.SentDate DESC;
```

### 7. Find seafarers with specific qualifications (via course enrollments):

```sql
SELECT u.UserID, u.FullName, u.Email, u.PhoneNumber
FROM Users u
JOIN CourseEnrollments ce ON u.UserID = ce.UserID
JOIN TrainingCourses tc ON ce.CourseID = tc.CourseID
WHERE u.UserType = 'Seafarer'
  AND tc.CourseName IN ('Advanced Firefighting', 'GMDSS Operator Course')
  AND ce.Status = 'Enrolled'
GROUP BY u.UserID
HAVING COUNT(DISTINCT tc.CourseID) = 2;
```

### 8. Generate a report of ship activity by company:

```sql
SELECT c.CompanyID, c.CompanyName, COUNT(s.ShipID) AS TotalShips,
       COUNT(ss.ScheduleID) AS TotalSchedules,
       COUNT(CASE WHEN ss.Status = 'Completed' THEN 1 END) AS CompletedVoyages,
       COUNT(CASE WHEN ss.Status = 'In Transit' THEN 1 END) AS InTransitVoyages,
       COUNT(CASE WHEN ss.Status = 'Scheduled' THEN 1 END) AS ScheduledVoyages
FROM Companies c
LEFT JOIN Ships s ON c.CompanyID = s.CompanyID
LEFT JOIN ShipSchedules ss ON s.ShipID = ss.ShipID
GROUP BY c.CompanyID, c.CompanyName
ORDER BY TotalShips DESC;
```

These queries demonstrate the database's capability to support complex operations needed for the Merchant Navy Portal's functionality, from job searching and application tracking to course management and communication.
