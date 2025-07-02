# Mtix-Lms
# School Management System - Professional LMS

A comprehensive Learning Management System with role-based access control for administrators, teachers, and students.

## Features

### Admin Module
- **User Management**: Create/update/disable all user accounts
- **Role Assignment**: Assign teacher/student roles with permissions
- **System Configuration**: Set academic terms, grading scales, fee structures
- **Reporting**: Generate financial, attendance, and performance reports
- **Audit Logs**: Track all system activities and changes
- **Bulk Operations**: Import/export student data, mass fee updates

### Teacher Module
- **Course Management**: Create and organize course materials
- **Lecture Upload**: Post lectures (videos, PDFs, assignments) with metadata
- **Gradebook**: Record and calculate student grades
- **Attendance**: Track and report student attendance
- **Analytics**: View student performance metrics
- **Communication**: Announcements and messaging to students

### Student Module
- **Dashboard**: Overview of courses, deadlines, announcements
- **Content Access**: View lectures, download materials, submit assignments
- **Grades**: View current grades and feedback
- **Fee Portal**: View and pay fees, download vouchers/receipts
- **Progress Tracking**: Visualize performance across courses

## Technical Architecture

### Frontend
- Next.js 14 (App Router)
- React 18 with TypeScript
- Tailwind CSS + ShadCN UI components
- Zustand for state management
- React Query for data fetching

### Backend
- Node.js with NestJS (or Express)
- REST API (consider GraphQL for complex queries)
- JWT authentication with refresh tokens
- Rate limiting and request validation

### Database
- PostgreSQL with Prisma ORM
- Redis for caching (queries, sessions)
- AWS S3 for file storage

### DevOps
- Docker containers
- CI/CD with GitHub Actions
- Monitoring with Prometheus/Grafana
- Error tracking with Sentry
- Log management

## Security Measures
- Role-based access control (RBAC)
- Password hashing with bcrypt
- CSRF protection
- Rate limiting
- Regular security audits
- Automated dependency updates

## Deployment
- Frontend: Vercel
- Backend: AWS ECS/EKS
- Database: AWS RDS (PostgreSQL)
- File Storage: AWS S3
- DNS & CDN: Cloudflare

# POSTGRES SCHEMA 
* enum Role {
  ADMIN
  TEACHER
  STUDENT
}

* enum FeeStatus {
  PENDING
  PAID
  OVERDUE
  CANCELLED
}

* model User {
-  id            String     @id @default(uuid())
-  email         String     @unique
-  passwordHash  String
-  role          Role
-  firstName     String
-  lastName      String
-  phone         String?
-  avatarUrl     String?
-  createdAt     DateTime   @default(now())
-  updatedAt     DateTime   @updatedAt
-  refreshTokens RefreshToken[]
-  student       Student?
-  teacher       Teacher?
-  notifications Notification[]
-  auditLogs     AuditLog[]
}

* model RefreshToken {
-  id        String   @id @default(uuid())
-  token     String   @unique
-  userId    String
-  user      User     @relation(fields: [userId], references: [id])
-  expiresAt DateTime
-  createdAt DateTime @default(now())
}

* model Student {
-  id          String     @id @default(uuid())
-  userId      String     @unique
-  user        User       @relation(fields: [userId], references: [id])
-  studentId   String     @unique
-  parentName  String
-  parentPhone String?
-  address     String?
-  classId     String
-  class       Class      @relation(fields: [classId], references: [id])
-  enrollments Enrollment[]
-  fees        Fee[]
-  attendances Attendance[]
-  submissions Submission[]
}

* model Teacher {
-  id           String     @id @default(uuid())
-  userId       String     @unique
-  user         User       @relation(fields: [userId], references: [id])
-  teacherId    String     @unique
-  qualifications String?
-  joinDate     DateTime   @default(now())
-  subjects     Subject[]
-  classes      Class[]
-  lectures     Lecture[]
-  assignments  Assignment[]
}

* model Class {
-  id          String     @id @default(uuid())
 - name        String
-  gradeLevel  Int
-  academicYear String
-  teacherId   String?
-  teacher     Teacher?   @relation(fields: [teacherId], references: [id])
-  students    Student[]
-  subjects    Subject[]
-  lectures    Lecture[]
-  assignments Assignment[]
}

* model Subject {
-  id          String     @id @default(uuid())
-  name        String
-  code        String     @unique
-  description String?
-  classes     Class[]
-  teachers    Teacher[]
-  lectures    Lecture[]
-  assignments Assignment[]
}

* model Lecture {
-  id          String     @id @default(uuid())
-  title       String
-  description String?
-  contentUrl  String
-  thumbnailUrl String?
-  subjectId   String
-  subject     Subject    @relation(fields: [subjectId], references: [id])
-  classId     String
-  class       Class      @relation(fields: [classId], references: [id])
-  teacherId   String
-  teacher     Teacher    @relation(fields: [teacherId], references: [id])
-  createdAt   DateTime   @default(now())
-  updatedAt   DateTime   @updatedAt
-  assignments Assignment[]
}

* model Assignment {
-  id          String     @id @default(uuid())
-  title       String
 - description String
-  dueDate     DateTime
-  lectureId   String?
-  lecture     Lecture?   @relation(fields: [lectureId], references: [id])
-  subjectId   String
-  subject     Subject    @relation(fields: [subjectId], references: [id])
-  classId     String
-  class       Class      @relation(fields: [classId], references: [id])
-  teacherId   String
-  teacher     Teacher    @relation(fields: [teacherId], references: [id])
-  submissions Submission[]
-  createdAt   DateTime   @default(now())
}

* model Submission {
-  id           String     @id @default(uuid())
-  assignmentId String
-  assignment   Assignment @relation(fields: [assignmentId], references: [id])
-  studentId    String
-  student      Student    @relation(fields: [studentId], references: [id])
-  contentUrl   String?
-  grade        Float?
-  feedback     String?
-  submittedAt  DateTime   @default(now())
-  gradedAt     DateTime?
}

* model Fee {
-  id          String     @id @default(uuid())
-  studentId   String
-  student     Student    @relation(fields: [studentId], references: [id])
-  amount      Float
-  description String
-  dueDate     DateTime
-  status      FeeStatus  @default(PENDING)
-  voucherUrl  String?
-  paymentDate DateTime?
-  receiptUrl  String?
-  createdAt   DateTime   @default(now())
-  updatedAt   DateTime   @updatedAt
}

* model Attendance {
-  id         String   @id @default(uuid())
-  studentId  String
-  student    Student  @relation(fields: [studentId], references: [id])
-  date       DateTime
-  status     String   // PRESENT, ABSENT, LATE, etc.
-  recordedBy String   // Teacher ID
-  remarks    String?
}

* model Notification {
-  id        String   @id @default(uuid())
-  userId    String
-  user      User     @relation(fields: [userId], references: [id])
-  title     String
-  message   String
-  isRead    Boolean  @default(false)
-  createdAt DateTime @default(now())
-  link      String?
}

* model AuditLog {
-  id          String   @id @default(uuid())
-  userId      String?
-  user        User?    @relation(fields: [userId], references: [id])
-  action      String
-  entityType  String?
-  entityId    String?
-  description String
-  ipAddress   String?
-  userAgent   String?
-  createdAt   DateTime @default(now())
}

* model SystemSetting {
-  id          String   @id @default(uuid())
-  key         String   @unique
-  value       String
-  description String?
-  updatedAt   DateTime @updatedAt
-  updatedBy   String   // User ID
}
