# HireSphere — Project Description

**An AI-Powered Career Development and Recruitment Management System**

Document basis: Software Requirements Specification v1.1 (IEEE 830 / ISO 29148), dated 31 October 2026.

---

## Table of Contents

- [1. Purpose](#1-purpose)
- [2. Scope](#2-scope)
- [3. Product Perspective](#3-product-perspective)
- [4. Product Functions (Summary)](#4-product-functions-summary)
- [5. User Classes and Characteristics](#5-user-classes-and-characteristics)
- [6. Operating Environment](#6-operating-environment)
- [7. Design and Implementation Constraints](#7-design-and-implementation-constraints)
- [8. Assumptions and Dependencies](#8-assumptions-and-dependencies)
- [9. Out of Scope (Initial Version)](#9-out-of-scope-initial-version)
- [10. System Features and Functional Requirements](#10-system-features-and-functional-requirements)
- [11. Use Case Specifications](#11-use-case-specifications)
- [12. Business Rules and Calculation Methods](#12-business-rules-and-calculation-methods)
- [13. Non-Functional Requirements Summary](#13-non-functional-requirements-summary)
- [14. MVP Acceptance Criteria](#14-mvp-acceptance-criteria)
- [15. Open Issues (To Be Decided)](#15-open-issues-to-be-decided)

---

## 1. Purpose

HireSphere is a web application that connects students with companies through one platform. It combines career preparation (CV analysis, skill-gap reports, practice tests, mock interviews, an AI career chatbot) with recruitment management (job posting, applications, online assessments, shortlisting, analytics). It builds trust between the two sides through company verification, fake-job protection, and transparent company hiring history. It has three portals: Student, Company, and Admin.

The system consists of a React single-page frontend, a Java Spring Boot REST backend, and a MySQL database, with AI features provided by an external AI service.

Requirements are marked **MVP** (to be delivered in the first release) or **Later** (deferred to a subsequent release). Requirement IDs follow the pattern `FR-<AREA>-<NN>` (functional), `NFR-<AREA>-<NN>` (non-functional), and `BR-<NN>` (business rules). The word "shall" marks a mandatory requirement.

---

## 2. Scope

HireSphere does not guarantee ATS results, job eligibility, selection, or placement. AI outputs are decision-support recommendations, and final hiring decisions remain with human recruiters. No automated check can prove that a company is genuine; HireSphere reduces risk through layered checks and human review but does not guarantee that every company or job is genuine.

---

## 3. Product Perspective

HireSphere is a new, self-contained system using a decoupled three-tier architecture:

- The **React SPA** runs in the user's browser and communicates with the Spring Boot backend only through secured REST APIs (and a Server-Sent Events stream for long AI responses).
- The **Spring Boot backend** stores data in MySQL, keeps CV files and company verification documents in file storage (metadata in MySQL), and calls an external AI provider for language-based features.
- An **email service** is needed in the MVP only to verify company email addresses.

---

## 4. Product Functions (Summary)

- Registration, login, and role-based access for students, companies, and administrators.
- Job posting, editing, closing, and browsing.
- Smart CV Analyzer: CV + job description analysis and CV-only job matching, with ATS check, job-match %, eligibility checklist, and skill-gap report.
- Application submission and status tracking.
- Timed MCQ assessments and practice mode with automatic scoring.
- AI mock interviews and an AI career chatbot, with long AI operations run asynchronously and streamed.
- Applicant review, shortlisting, and recruitment status management.
- Company profile with platform-verified hiring history (students hired, average and highest package) and career growth paths.
- Following companies with a priority level so high-priority companies produce priority notifications.
- Student activity streaks and points on the dashboard (Later).
- Fake company and fake job protection: stronger company verification, job screening, student reports, and administrator controls.
- Dashboards for students, companies, and administrators, and notifications.
- Administration: user management, company verification, job moderation, report handling, audit log, and question bank management.

---

## 5. User Classes and Characteristics

| User class | Description | Technical skill | Main needs |
|---|---|---|---|
| **Student** | Undergraduate or graduate preparing for internships and placements | Basic web use | Improve CV, find suitable jobs, practise, apply and track; trust that companies are real; see company hiring history |
| **Company / Recruiter** | HR or hiring staff of a verified organisation | Basic web use | Publish jobs, assess and shortlist candidates, view analytics, present the company and its placement record |
| **Administrator** | Platform operator | Moderate | Manage users, verify companies, moderate jobs and reports, maintain question bank, review audit log |

---

## 6. Operating Environment

- **Client:** modern web browsers (latest two versions of Chrome, Edge, Firefox, and Safari) on desktop, tablet, and mobile.
- **Server:** Java 17 or later with Spring Boot, on Windows, Linux, or macOS.
- **Database:** MySQL 8.x.
- **External services:** an AI provider API reachable over HTTPS, and an email (SMTP) service for company email verification.

---

## 7. Design and Implementation Constraints

- The frontend shall be built with React; the backend with Java and Spring Boot; MySQL is the primary database.
- All client-server communication shall use REST over HTTPS with JSON payloads and JWT authentication. Streaming AI output uses Server-Sent Events over HTTPS.
- AI features shall use an external AI service; no custom model is trained in the initial version.
- The first release shall be a modular monolith, not a microservice system.
- Uploaded CV text extraction depends on text-based PDFs; scanned image-only CVs may not be analysable.
- The MVP accepts PDF CVs only; DOCX support is deferred (TBD-05).

---

## 8. Assumptions and Dependencies

- Companies will provide complete job descriptions and eligibility criteria; matching quality depends on this data.
- Students will provide accurate profiles and readable CVs.
- Companies will provide truthful registration details and documents; administrators verify them manually.
- The external AI service is available; when it is not, only AI features are affected.
- Job recommendations and matching initially use only jobs stored in the HireSphere database.
- Hiring history on a company profile reflects only offers recorded through HireSphere unless the company adds approved self-reported data.
- Users have internet access and a supported browser.

---

## 9. Out of Scope (Initial Version)

- A custom-trained AI model; facial emotion or personality analysis.
- Automatic final hiring decisions without human review.
- Live video proctoring and advanced anti-cheating systems.
- Guaranteed integration with external job portals.
- Automatic verification against government company registries (manual administrator review is used).
- Native Android and iOS apps; enterprise-scale microservices.
- Guaranteed ATS scores or guaranteed placement.

---

## 10. System Features and Functional Requirements

### 10.1 Authentication and User Management

Users register, log in, and are restricted to the functions of their role.

| ID | Requirement | Priority |
|---|---|---|
| FR-AUTH-01 | Allow a student to register with name, email, password, and basic academic details. | MVP |
| FR-AUTH-02 | Allow a company to register with company name, contact person, official work email, password, contact details, registration number, website, and a proof document (see FR-ADM-08); the account remains Pending until verified. | MVP |
| FR-AUTH-03 | Authenticate users by email and password and return a JWT on success. | MVP |
| FR-AUTH-04 | Store passwords only as salted hashes (BCrypt); never return them in any response. | MVP |
| FR-AUTH-05 | Enforce role-based access (STUDENT, COMPANY, ADMIN) on every backend endpoint and every frontend route. | MVP |
| FR-AUTH-06 | Reject duplicate email registrations and invalid input with a clear message. | MVP |
| FR-AUTH-07 | Treat an expired or invalid token as unauthenticated and redirect to the login page. | MVP |
| FR-AUTH-08 | An administrator can activate or deactivate any account; a deactivated user cannot log in. | MVP |
| FR-AUTH-09 | Support password reset through an email link. | Later |

### 10.2 Student Profile

Students maintain the academic and professional information used for eligibility checks and matching.

| ID | Requirement | Priority |
|---|---|---|
| FR-STU-01 | View and edit personal details, college, degree, graduation year, and CGPA. | MVP |
| FR-STU-02 | Add, edit, and remove skills, projects, and certifications. | MVP |
| FR-STU-03 | Set preferred job roles and locations. | MVP |
| FR-STU-04 | Upload, replace, and delete a CV (PDF only in the MVP; DOCX is Later), maximum 5 MB. | MVP |
| FR-STU-05 | Set notification preferences. | Later |
| FR-STU-06 | Withdrawn: merged into FR-UI-05 (dark / light mode). The ID is kept so earlier references stay traceable. | — |

### 10.3 Job Management and Browsing

Verified companies publish and manage vacancies; students browse them. Job statuses are defined in [Section 12.4](#84-job-status-rules).

| ID | Requirement | Priority |
|---|---|---|
| FR-JOB-01 | A verified company can create a job with title, description, required skills (at least one), eligibility criteria (min CGPA, degree, graduation year), salary or range, and application deadline. | MVP |
| FR-JOB-02 | A company can add optional details: preferred skills, experience, working hours and days, leave policy, benefits, work-from-home or hybrid option, and selection rounds. | Later |
| FR-JOB-03 | A company can edit and close its own vacancies. | MVP |
| FR-JOB-04 | Automatically stop accepting applications when the deadline passes or the job is closed. | MVP |
| FR-JOB-05 | Prevent companies whose verification status is not Verified (Pending, Rejected, Suspended) from publishing jobs. | MVP |
| FR-JOB-06 | Students can browse published jobs and filter by company, location, salary, and work mode. | MVP |
| FR-JOB-07 | Students can view full job details including eligibility, deadline, and the company's Verified badge. | MVP |
| FR-JOB-08 | An administrator can flag, hide, or remove invalid or reported job postings (statuses Pending_Review, Hidden, Removed). | MVP |
| FR-JOB-09 | The first 3 jobs of every newly verified company are held as Pending_Review until an administrator approves them. | MVP |
| FR-JOB-10 | Scan job text for scam indicators (Section 12.6). Flagged jobs are held as Pending_Review and placed in the administrator review queue. | MVP |
| FR-JOB-11 | Students can report a job or company with a reason. When reports on a job reach a threshold (default 3 from different students, configurable), the job is hidden automatically until an administrator reviews it. | MVP |
| FR-JOB-12 | Limit the number of job postings per company per day (configurable). | Later |
| FR-JOB-13 | Jobs and company profiles show a Verified badge, and job pages show the warning "Never pay money to apply for a job. Report suspicious jobs." | MVP |

### 10.4 Smart CV Analyzer

The analyzer compares a student's CV with a job description (Case 1) or with all published jobs (Case 2) and produces a structured report. ATS compatibility, job-match percentage, and eligibility are reported separately.

| ID | Requirement | Priority |
|---|---|---|
| FR-CV-01 | Validate an uploaded CV for allowed file type (PDF in the MVP), size, and readable content before analysis. | MVP |
| FR-CV-02 | Extract text from the CV and identify skills, education, projects, certifications, and experience. | MVP |
| FR-CV-03 | Case 1: accept a job description (pasted text or file) or an existing vacancy selected by the student. | MVP |
| FR-CV-04 | Case 2: compare the CV with all published jobs and show a separate match result for each. | MVP |
| FR-CV-05 | Calculate a job-match percentage using the weighted method in [Section 12.2](#82-job-match-calculation). | MVP |
| FR-CV-06 | The report lists matching skills and missing skills (skill gap). | MVP |
| FR-CV-07 | The report includes an eligibility checklist; where required student data is unavailable it shows "cannot be determined" instead of passing or failing. | MVP |
| FR-CV-08 | The report includes an indicative ATS compatibility result based on readability, structure, and keyword coverage. | MVP |
| FR-CV-09 | Generate suggestions for skills, projects, certifications, and CV improvements using the AI service. This call runs asynchronously (NFR-PERF-04) and rule-based results are shown first. | MVP |
| FR-CV-10 | In Case 2, results are ordered by match percentage, with links to job details and the Apply action. | MVP |
| FR-CV-11 | Store each analysis result and skill-gap report so the student can view past reports. | MVP |
| FR-CV-12 | Every report states that results are indicative and do not guarantee any ATS result, eligibility, or selection. | MVP |
| FR-CV-13 | If text cannot be extracted (scanned image CV), tell the student and suggest uploading a text-based PDF. | MVP |
| FR-CV-14 | If the AI service is unavailable, still show the rule-based match, skills, and eligibility results and indicate that AI suggestions are temporarily unavailable. | MVP |
| FR-CV-15 | Show a consent prompt before the student's first use of any AI feature and store the consent with its date. Without consent, only rule-based results are shown and no CV or interview content is sent to the AI service. | MVP |

### 10.5 Job Recommendations and Alerts

Students are told about relevant opportunities.

| ID | Requirement | Priority |
|---|---|---|
| FR-REC-01 | Notify students when a newly published job matches their preferred roles or skills (basic rule). | MVP |
| FR-REC-02 | Show a list of recommended jobs on the student dashboard based on skills, education, and preferred roles. | MVP |
| FR-REC-03 | Support advanced alert rules and configurable alert preferences. | Later |

### 10.6 Application Tracker

Students apply to jobs and follow progress; companies update progress.

| ID | Requirement | Priority |
|---|---|---|
| FR-APP-01 | A student can apply once to an open job before its deadline. An application cannot be resubmitted after withdrawal (BR-02). | MVP |
| FR-APP-02 | Record the submission date and set the status to Applied. | MVP |
| FR-APP-03 | The student sees a list of applications with company, job, submission date, and current status. | MVP |
| FR-APP-04 | Allowed statuses: Applied, Assessment, Shortlisted, Interview, Selected, Rejected, Withdrawn, following the transitions in [Section 12.1](#81-application-status-lifecycle). | MVP |
| FR-APP-05 | Every status change is saved in a status history with time and the user who made it. | MVP |
| FR-APP-06 | A student can withdraw an application until a final decision is recorded. Withdrawal is final. | MVP |
| FR-APP-07 | Create a notification for the student on every status change. | MVP |
| FR-APP-08 | If the student does not meet explicit eligibility criteria, warn before applying; blocking is configurable per job (default: warn only). | MVP |
| FR-APP-09 | The tracker shows assessment and interview schedules and upcoming deadlines. | Later |

### 10.7 Assessments and Practice

Companies assess candidates with timed MCQ tests; students practise.

| ID | Requirement | Priority |
|---|---|---|
| FR-TST-01 | A company can create a test for a job by selecting questions from the question bank or adding its own, and setting a duration. | MVP |
| FR-TST-02 | The company can assign a test to selected applicants. | MVP |
| FR-TST-03 | A student can attempt an assigned test within the time limit; the attempt auto-submits when time ends. | MVP |
| FR-TST-04 | Automatically evaluate MCQ answers and store score and answers in MySQL. | MVP |
| FR-TST-05 | A student has one attempt per assigned test unless the company allows a retake. | MVP |
| FR-TST-06 | The company sees results for each candidate and can sort by score. | MVP |
| FR-TST-07 | Practice mode lets students attempt aptitude and technical MCQs by topic, company, and difficulty and see correct answers with explanations. | MVP |
| FR-TST-08 | Keep a practice performance history and show comparison with aggregate results. | Later |
| FR-TST-09 | Support coding problems with execution in a secure sandbox. | Later |
| FR-TST-10 | Support advanced online proctoring. | Later |

### 10.8 AI Interview Agent

Students practise interviews; companies later run preliminary AI interviews.

| ID | Requirement | Priority |
|---|---|---|
| FR-INT-01 | A student can start a text-based mock interview by choosing type (HR or technical) and a target role. | MVP |
| FR-INT-02 | Generate interview questions relevant to the chosen role and the student's skills. | MVP |
| FR-INT-03 | The student submits answers as text. | MVP |
| FR-INT-04 | Return AI feedback on relevance, completeness, and clarity with improvement suggestions. Feedback is streamed to the UI. | MVP |
| FR-INT-05 | At the end of a session, present a summary report and keep the session in the student's history. | MVP |
| FR-INT-06 | If the AI service fails or times out, send an error event, show an error, keep answers and any partial output already received, and allow retry. | MVP |
| FR-INT-07 | A company can configure a job-specific preliminary interview with question sets and an evaluation rubric. | Later |
| FR-INT-08 | Produce an AI assessment summary for recruiter review, clearly labelled as a recommendation. | Later |
| FR-INT-09 | Support voice and video interviews. | Later |

### 10.9 AI Career Chatbot

A conversational guide for career questions.

| ID | Requirement | Priority |
|---|---|---|
| FR-BOT-01 | Provide a chat interface where a student can ask career, CV, interview, skill, and job-search questions. Answers are streamed. | MVP |
| FR-BOT-02 | Store chat history per user and show it on return. | MVP |
| FR-BOT-03 | Display a notice that responses are AI-generated guidance. | MVP |
| FR-BOT-04 | If the AI service is unavailable or times out, show a clear message, keep any partial answer, and allow retry. | MVP |
| FR-BOT-05 | With the student's consent, use profile and CV information for personalised answers. | Later |

### 10.10 Applicant Management and Shortlisting

Companies review applicants and manage recruitment.

| ID | Requirement | Priority |
|---|---|---|
| FR-CAM-01 | A company sees all applicants for its own jobs with profile, CV, eligibility information, and CV match result. | MVP |
| FR-CAM-02 | The company can filter and sort applicants by status, test score, and eligibility. | MVP |
| FR-CAM-03 | The company can change an applicant's status and shortlist candidates. Setting the status to Selected requires the offered role and package (FR-COM-05). | MVP |
| FR-CAM-04 | The company can view test results and, later, AI interview feedback. | MVP |
| FR-CAM-05 | A company cannot see applicants or data of another company's jobs. | MVP |
| FR-CAM-06 | The system does not automatically select or reject any candidate; every final status change is made by a company user. | MVP |

### 10.11 Company Profile

Companies present themselves to students, including their hiring track record.

| ID | Requirement | Priority |
|---|---|---|
| FR-COM-01 | A company can edit its description, industry, contact details, and work arrangements. | MVP |
| FR-COM-02 | Students can view a company's profile from a job page, including its Verified badge and aggregated hiring history. | MVP |
| FR-COM-03 | A company can upload and manage workplace photographs and event images (image types only, size-limited). | Later |
| FR-COM-04 | The company profile shows hiring history per academic year: number of students hired, average package, highest package, and roles offered. | MVP |
| FR-COM-05 | When a recruiter sets an application to Selected, the system requires the offered role and package (CTC in LPA). Hiring history is computed automatically from these records and labelled "Platform-verified". | MVP |
| FR-COM-06 | A company may add past hiring data from before it joined HireSphere. It is labelled "Self-reported" and visible only after administrator approval. | Later |
| FR-COM-07 | Students see only aggregated figures. Individual offers are never shown, and averages and maxima are hidden when fewer than 3 hires exist (BR-14). | MVP |
| FR-COM-08 | A company can describe career growth paths per role (e.g. Trainee → Engineer → Senior Engineer) with typical time per level and package growth. Shown after administrator approval. | Later |

### 10.12 Dashboards and Analytics

Role-specific overviews.

| ID | Requirement | Priority |
|---|---|---|
| FR-DSH-01 | The student dashboard shows available jobs, application statuses, CV analysis summaries, and test results. | MVP |
| FR-DSH-02 | The company dashboard shows published jobs, applicant counts, assessment results, and shortlisted candidates. | MVP |
| FR-DSH-03 | The admin dashboard shows total students, companies, active jobs, and applications, the number of pending company verifications and open job reports, with links to management pages. | MVP |
| FR-DSH-04 | The Career Readiness Dashboard combines practice results, CV analysis, and mock interview history and suggests areas for improvement. | Later |
| FR-DSH-05 | The Recruitment Analytics Dashboard shows applicants per role, stage conversion rates, time-to-hire, and graphical reports. | Later |
| FR-DSH-06 | Record one activity day per student per calendar day. | Later |
| FR-DSH-07 | The student dashboard shows total active days, current streak, longest streak (one grace day per week keeps a streak alive), and a monthly calendar view. | Later |
| FR-DSH-08 | Award points for meaningful activities per [Section 12.5](#85-engagement-points-later), with a daily cap (default 50, configurable). | Later |
| FR-DSH-09 | The dashboard shows total points, level or badge, and a weekly points trend. Points are for motivation only (BR-12). | Later |

### 10.13 Notifications

Users are informed of relevant events. Students can also decide which companies matter most to them.

| ID | Requirement | Priority |
|---|---|---|
| FR-NOT-01 | Create in-app notifications for status changes, test assignments, company verification outcome, job review outcome (approved or removed), company suspension, and new matching jobs. | MVP |
| FR-NOT-02 | Users can view notifications and mark them as read. | MVP |
| FR-NOT-03 | Send email notifications for key events. | Later |
| FR-NOT-04 | A student can follow companies and set a priority for each: High, Normal, or Muted. | MVP |
| FR-NOT-05 | When a followed company publishes a job (status Open), the student gets a notification. For High priority the notification is marked "Priority" and listed first. Muted companies produce no notification. | MVP |
| FR-NOT-06 | For High-priority companies, notify the student when a job deadline is within 48 hours. | MVP |
| FR-NOT-07 | Jobs from followed High-priority companies are highlighted in the job list and shown earlier in recommendations. This does not change the match %. | Later |
| FR-NOT-08 | The number of High-priority companies per student is limited (default 10, configurable) to avoid notification overload. | MVP |
| FR-NOT-09 | High-priority alerts are also sent by email. | Later |
| FR-NOT-10 | A student does not receive two notifications for the same job; when a job matches both FR-REC-01 and a followed company, only the higher-priority notification is kept. | MVP |

### 10.14 Administration

Platform operators maintain users, companies, jobs, and content, and protect students from fake companies and fake jobs. Protection is layered and ends with human review.

**A. Users, content, and audit**

| ID | Requirement | Priority |
|---|---|---|
| FR-ADM-01 | An administrator can list, search, activate, and deactivate student and company accounts. | MVP |
| FR-ADM-02 | An administrator can review company registrations and approve or reject them with a reason, after completing the checklist in FR-ADM-10. | MVP |
| FR-ADM-03 | An administrator can monitor and moderate job postings, including the Pending_Review queue and flagged jobs. | MVP |
| FR-ADM-04 | An administrator can add, edit, and delete aptitude and technical MCQs, organised by topic, company, and difficulty. | MVP |
| FR-ADM-05 | Administrator accounts are not created through public registration. | MVP |
| FR-ADM-06 | An administrator can manage announcements and FAQs. | Later |
| FR-ADM-07 | The system keeps an append-only audit log of administrative actions and allows administrators to review it. | MVP |

**B. Stronger company verification**

| ID | Requirement | Priority |
|---|---|---|
| FR-ADM-08 | Company registration also requires an official registration number (e.g. CIN or GSTIN), company website, official work email, and at least one proof document (PDF or image, size-limited). | MVP |
| FR-ADM-09 | Verify email ownership by link or OTP. Free-mail domains (Gmail, Yahoo, and similar) are flagged for extra review. | MVP |
| FR-ADM-10 | The administrator completes a checklist before approval: registration number checked, email domain matches website, website reachable, document reviewed, and no duplicate registration number, domain, or phone on another account. The decision and reason are logged. | MVP |
| FR-ADM-11 | Detect duplicate registration numbers, domains, and phone numbers across accounts and warn the administrator. | MVP |

**C. Controls and trust**

| ID | Requirement | Priority |
|---|---|---|
| FR-ADM-12 | An administrator can suspend a company or revoke its verification. All its jobs are hidden and affected applicants notified. | MVP |
| FR-ADM-13 | Every verification, rejection, suspension, report decision, and moderation action is written to the audit log with administrator, time, target, decision, and reason. | MVP |
| FR-ADM-14 | Companies have a trust level: Verified, then Trusted after a configurable number of jobs with no upheld reports. | Later |
| FR-ADM-15 | An administrator can view the reports queue, see report history per job and company, and uphold or dismiss each report with a reason. | MVP |

### 10.15 User Interface

| ID | Requirement | Priority |
|---|---|---|
| FR-UI-01 | The UI is a responsive React application usable from 360 px to 1920 px screen widths. | MVP |
| FR-UI-02 | Each role sees only its own navigation and pages; unauthorised routes redirect to a login or "unauthorised" page. | MVP |
| FR-UI-03 | Forms show clear validation messages next to the invalid field. | MVP |
| FR-UI-04 | Long operations (CV analysis, AI responses) show a loading indicator or skeleton with progress and a readable error message on failure. Operations that can exceed 5 seconds run asynchronously; the UI never blocks while waiting. | MVP |
| FR-UI-05 | The UI supports dark and light themes, and the student's choice is remembered across sessions. (Merges the former FR-STU-06.) | Later |

---

## 11. Use Case Specifications

The system defines 13 key use cases across the three portals:

| UC | Title | Actor | Priority |
|---|---|---|---|
| UC-01 | Register and Log In | Student, Company | MVP |
| UC-02 | Verify Company Registration | Administrator | MVP |
| UC-03 | Post Job Vacancy | Company (verified) | MVP |
| UC-04 | Analyze CV against a Job Description (Case 1) | Student | MVP |
| UC-05 | Match CV against Available Jobs (Case 2) | Student | MVP |
| UC-06 | Apply for Job and Track Application | Student | MVP |
| UC-07 | Review Applicants and Update Status | Company | MVP |
| UC-08 | Attempt Timed Test | Student | MVP |
| UC-09 | Take AI Mock Interview | Student | MVP |
| UC-10 | Chat with AI Career Bot | Student | MVP |
| UC-11 | Follow Company and Receive Priority Alerts | Student | MVP |
| UC-12 | Report Suspicious Job or Company | Student | MVP |
| UC-13 | Handle Reports and Suspend Company | Administrator | MVP |

Each use case has defined preconditions, a main flow, alternate/exception flows, postconditions, and related requirement IDs. See the SRS (Section 5) for full specifications.

---

## 12. Business Rules and Calculation Methods

### 12.1 Application Status Lifecycle

An application starts as **Applied** and moves through the statuses below. Selected, Rejected, and Withdrawn are final. Every transition is stored in the status history. A withdrawn application cannot be resubmitted. Selecting a candidate requires the offered role and package.

| From | Allowed next status |
|---|---|
| Applied | Assessment, Shortlisted, Rejected, Withdrawn |
| Assessment | Shortlisted, Rejected, Withdrawn |
| Shortlisted | Interview, Rejected, Withdrawn |
| Interview | Selected, Rejected, Withdrawn |
| Selected / Rejected / Withdrawn | None (final) |

### 12.2 Job-Match Calculation

The job-match percentage is **rule-based and reproducible**. The AI service explains gaps and suggests improvements but does not produce the number. The weights are proposed starting values and are configurable.

| Component | Weight | Measurement |
|---|---|---|
| Required skills coverage | 55% | Share of the job's required skills found in the CV (exact and synonym match). A job always has at least one required skill. |
| Preferred skills coverage | 20% | Share of the job's preferred skills found in the CV |
| Projects, certifications, and experience relevance | 25% | Keyword and role overlap between CV content and the job description |

**Formula:**

```
Match % = 0.55 × required_coverage + 0.20 × preferred_coverage + 0.25 × relevance_score
```

Eligibility (CGPA, degree, graduation year) and the ATS readability result are reported **separately** and are not included in the percentage.

**No preferred skills:** if a job lists no preferred skills, the 20% weight is redistributed:

```
Match % = 0.70 × required_coverage + 0.30 × relevance_score
```

This avoids a division by zero and keeps the maximum at 100%.

### 12.3 Business Rules

| ID | Rule |
|---|---|
| BR-01 | Only companies whose verification status is Verified may publish jobs. Pending, Rejected, and Suspended companies cannot. |
| BR-02 | A student may apply to a given job only once. A withdrawn application is final and cannot be resubmitted. |
| BR-03 | Applications are not accepted after the deadline or after the job is closed. |
| BR-04 | Only the company that owns a job may view its applicants and change application statuses. |
| BR-05 | Status changes must follow the lifecycle in Section 12.1. |
| BR-06 | ATS result, job-match percentage, and eligibility are always shown as three separate results. |
| BR-07 | CV uploads are limited to PDF (DOCX is Later) and a maximum of 5 MB. |
| BR-08 | A timed test auto-submits when its time limit ends; answers given so far are scored. |
| BR-09 | AI output is advisory; final hiring decisions are made by human recruiters. |
| BR-10 | Closed jobs remain visible in the tracker of students who applied. |
| BR-11 | Administrator accounts are created only by existing administrators or deployment seeding, never by public registration. |
| BR-12 | Points and streaks are for student motivation only. They are not visible to companies and do not affect match %, ranking, or any hiring decision. |
| BR-13 | Jobs that ask applicants for payment are prohibited and shall be removed. |
| BR-14 | Company hiring statistics are shown only as aggregates and only when at least 3 hires exist for the period; individual offers are never shown to students. |

### 12.4 Job Status Rules

| From | Allowed next status | Trigger |
|---|---|---|
| Draft | Open, Pending_Review | Verified company publishes. Pending_Review if it is one of the first 3 jobs or is flagged by the scam screening. |
| Pending_Review | Open, Removed | Administrator approves or rejects (reason logged). |
| Open | Closed, Hidden, Removed | Company closes or deadline passes (Closed); administrator or report threshold hides (Hidden); administrator removes (Removed). |
| Hidden | Open, Removed | Administrator restores or removes after review. |
| Closed / Removed | None (final) | A new job must be created. When a company is suspended, all its Open jobs become Hidden. |

### 12.5 Engagement Points (Later)

Points reward meaningful activity, not logins alone. A daily cap applies (default 50, configurable). A streak may skip one day per week without breaking. Final values are open (TBD-09).

| Activity | Points |
|---|---|
| Daily login (once per day) | 5 |
| CV analysis completed | 10 |
| Practice test completed | 10, plus up to 5 bonus for score |
| Mock interview completed | 15 |
| Job application submitted | 5 |
| Profile 100% complete (one time) | 20 |
| 7-day streak bonus | 25 |

### 12.6 Initial Scam Indicators (FR-JOB-10)

- **Payment requests:** "registration fee", "security deposit", "pay to apply", "training fee", "refundable deposit".
- **Unrealistic promises:** "guaranteed job", "100% placement", very high salary for no skills or experience.
- **Unofficial contact:** contact only through WhatsApp or Telegram, or a personal email address (e.g. Gmail) instead of a company domain.
- The list is configurable and tuned with sample postings (TBD-10). A flag never removes a job by itself; an administrator decides.

---

## 13. Non-Functional Requirements Summary

### 13.1 Performance

| ID | Requirement | Target |
|---|---|---|
| NFR-PERF-01 | LCP of landing, login, and dashboard pages | ≤ 3 s on throttled Fast 4G (95th pct); ≤ 2 s unthrottled |
| NFR-PERF-02 | Non-AI API calls response time | ≤ 2 s for 95% of requests (up to 100 concurrent users) |
| NFR-PERF-03 | Rule-based CV analysis (excluding AI call) | ≤ 5 s for a CV up to 5 MB |
| NFR-PERF-04a–f | Asynchronous AI operations via SSE or task polling | API responds ≤ 1 s; first SSE chunk ≤ 5 s; hard limit 30 s; UI never blocks; results retrievable by task ID |
| NFR-PERF-05 | Timed-test submission evaluation and storage | ≤ 3 s |

### 13.2 Security

| ID | Requirement |
|---|---|
| NFR-SEC-01 | Passwords hashed with BCrypt; plain text never stored or logged. |
| NFR-SEC-02 | JWTs expire (default 24 h, configurable) and are validated on every protected request. |
| NFR-SEC-03 | Role-based authorisation enforced on the server for every endpoint; hidden UI controls are not the only protection. |
| NFR-SEC-04 | Uploaded files checked for type, size, and content; not executable or served from the web root. |
| NFR-SEC-05 | Protection against SQL injection, XSS, and CSRF through parameterised queries, input validation, output encoding, and CORS. |
| NFR-SEC-06 | All production traffic uses HTTPS. |
| NFR-SEC-07 | CVs, assessment results, and interview records accessible only to their owner, companies the student applied to, and administrators where necessary. |
| NFR-SEC-08 | Secrets supplied through environment configuration, not source code. |
| NFR-SEC-09 | Repeated failed logins rate-limited: after 5 failed attempts per email or IP within 15 minutes, blocked for 15 minutes (configurable). |
| NFR-SEC-10 | Company verification documents accessible only to the owning company and administrators. |
| NFR-SEC-11 | Task and SSE endpoints require authentication, verify task ownership, and do not place long-lived tokens in URLs. |

### 13.3 Reliability and Availability

| ID | Requirement |
|---|---|
| NFR-REL-01 | A failure or timeout of the AI service does not affect non-AI features. |
| NFR-REL-02 | Applications, test submissions, and status changes saved transactionally. |
| NFR-REL-03 | Target 99% availability during a pilot period. |
| NFR-REL-04 | Errors logged with enough detail for diagnosis without exposing internals to users. |

### 13.4 Usability and Accessibility

| ID | Requirement |
|---|---|
| NFR-USE-01 | A new student can start a CV analysis from the dashboard within 3 clicks. |
| NFR-USE-02 | UI is consistent across portals and responsive from 360 px to 1920 px. |
| NFR-USE-03 | Form errors shown in plain language next to the relevant field. |
| NFR-USE-04 | UI aims to meet WCAG 2.1 AA basics: labelled inputs, sufficient contrast, keyboard navigation. |

### 13.5 Maintainability, Portability, and Scalability

| ID | Requirement |
|---|---|
| NFR-MNT-01 | Backend organised in feature modules with controller, service, repository, entity, and DTO layers. |
| NFR-MNT-02 | All REST endpoints documented with OpenAPI (Swagger UI). |
| NFR-MNT-03 | Core services (matching, scoring, authentication) have automated tests; target at least 60% line coverage. |
| NFR-MNT-04 | Configuration externalised so the same build runs in development and production. |
| NFR-POR-01 | Frontend works on the latest two versions of Chrome, Edge, Firefox, and Safari. |
| NFR-SCL-01 | Backend is stateless for horizontal scaling; queries use indexes on foreign keys and search columns. Async task state kept in MySQL, not server memory. |

### 13.6 Privacy and Responsible AI

| ID | Requirement |
|---|---|
| NFR-PRV-01 | Obtain user consent before sending CV or interview content to the external AI service. |
| NFR-PRV-02 | Only data needed for the AI task is sent; passwords, tokens, and unrelated personal data are not sent. |
| NFR-PRV-03 | AI-generated content is labelled as a recommendation, not a guarantee. |
| NFR-PRV-04 | Consequential recruitment decisions are made by humans; AI scores do not trigger automatic rejection or selection. |
| NFR-PRV-05 | Points, streaks, and activity data are not visible to companies and not used in matching, ranking, or any hiring decision. |
| NFR-PRV-06 | Company hiring statistics shown only as aggregates, never as individual offers, and hidden when fewer than 3 hires exist. |

---

## 14. MVP Acceptance Criteria

The MVP is accepted when all criteria below pass.

| ID | Acceptance test | Requirements |
|---|---|---|
| AC-01 | Given valid details, a student and a company can register and log in and each sees only their own portal. | FR-AUTH-01 to 05 |
| AC-02 | A verified company can publish a job with a complete description, skills, eligibility, salary, and deadline; a Pending, Rejected, or Suspended company cannot. | FR-JOB-01, 03, 05 |
| AC-03 | A student can upload a PDF CV and receive a CV-to-job analysis report. | FR-CV-01 to 03, 05 to 09, 15 |
| AC-04 | A student can compare one CV with multiple published jobs and see a separate result for each. | FR-CV-04, 10 |
| AC-05 | Reports show matching skills, missing skills, and recommendations. | FR-CV-06, 09 |
| AC-06 | A student can apply for a job and track its status through changes; a withdrawn application cannot be resubmitted. | FR-APP-01 to 07, BR-02 |
| AC-07 | A company can review applicants, view test results, and shortlist candidates. | FR-CAM-01 to 04, FR-TST-06 |
| AC-08 | The chatbot and mock interview work when the AI service is available and show a clear error when it is not. | FR-BOT-01 to 04, FR-INT-01 to 06 |
| AC-09 | Application, job, profile, and assessment data are stored and retrieved correctly from MySQL. | DR-01, FR-TST-04 |
| AC-10 | Unauthorised users cannot access restricted student or company data or company verification documents. | FR-AUTH-05, FR-CAM-05, NFR-SEC-03, 07, 10 |
| AC-11 | Fake-company and fake-job protection works: registration without registration number or proof is rejected; an administrator cannot approve before completing the checklist; a duplicate registration number raises a warning; the first 3 jobs of a new company are held as Pending_Review; a job containing "registration fee" goes to the review queue; 3 student reports hide a job; suspending a company hides all its jobs and notifies applicants. | FR-ADM-08 to 13, FR-ADM-15, FR-JOB-09 to 11, FR-JOB-13 |
| AC-12 | A student can follow a company and set High, Normal, or Muted. A new job from a High-priority company gives a Priority notification listed first; Muted gives none; an 11th High-priority company is refused when the limit is 10; no duplicate notification is created for one job. | FR-NOT-04 to 06, 08, 10 |
| AC-13 | Setting Selected without role and package is refused. After Selected, the company profile shows per-year aggregates labelled Platform-verified; with fewer than 3 hires averages are hidden; no individual offer is visible to students. | FR-COM-04, 05, 07, FR-CAM-03 |
| AC-14 | A CV analysis shows rule-based results within 5 seconds while AI suggestions load afterwards; chatbot streaming starts within 5 seconds; at the 30-second limit an error with retry appears and partial output is kept; after a page refresh the result is retrieved by task ID; AI is blocked until consent is given. | NFR-PERF-03, 04a–f, FR-CV-09, 14, 15, FR-BOT-04, FR-INT-06 |
| AC-15 | The 6th failed login within the rate-limit window is blocked, and every administrator verification, rejection, suspension, and report decision appears in the audit log. | NFR-SEC-09, FR-ADM-07, FR-ADM-13 |

---

## 15. Open Issues (To Be Decided)

| ID | Issue | Decision needed | Status |
|---|---|---|---|
| TBD-01 | AI provider and model | Choose provider, cost limit, and rate limits | Open |
| TBD-02 | Hosting and deployment environment | Choose host, domain, and backup method | Open |
| TBD-03 | Email provider (SMTP) | Needed in the MVP for company email verification; choose service | Open (changed) |
| TBD-04 | Final matching weights and skill synonym list | Tune using sample CVs and jobs | Open |
| TBD-05 | DOCX CV support in the first release | Decided: PDF only in the MVP; DOCX is Later | Resolved in v1.1 |
| TBD-06 | Data retention period for applications, assessments, documents, and logs | Agree with institution or policy | Open |
| TBD-07 | Standard list of degrees and eligibility fields | Define dropdown values for job and profile forms | Open |
| TBD-08 | Registration-number types to accept and validate | Decide which of CIN, GSTIN, or others are accepted and what format checks to apply | Open (new) |
| TBD-09 | Final point values and streak grace rule | Tune with student feedback (Later release) | Open (new) |
| TBD-10 | Scam indicator list and thresholds | Confirm keyword list, first-3-jobs rule, report threshold of 3, and daily posting limit | Open (new) |
