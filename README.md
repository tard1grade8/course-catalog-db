# Course Catalog Database (Lab 6)

## Constraint violation log

Commands were run with `\set VERBOSITY verbose`.

| # | What we violate | SQLSTATE | Constraint | API response |
|---|---|---|---|---|
| 1 | NULL in courses.title | 23502 | title (COLUMN NAME) | 422 Unprocessable Entity (if Pydantic let it through: 500) |
| 2 | courses.credits = 0 | 23514 | courses_credits_check | 422 Unprocessable Entity |
| 3 | courses.likes = -1 | 23514 | courses_likes_check | 500 Internal Server Error (only server code increments likes) |
| 4 | a one-letter student name | 23514 | students_name_check | 422 Unprocessable Entity |
| 5 | a duplicate student email | 23505 | students_email_key | 409 Conflict |
| 6 | a duplicate course id | 23505 | courses_pkey | 409 Conflict |
| 7 | a repeated enrollment in a course | 23505 | enrollments__student_id_course_id_key | 409 Conflict |
| 8 | enrolling in a nonexistent course | 23503 | enrollments_course_id_fkey | 404 Not Found (or 422 Unprocessable Entity) |
| 9 | enrolling a nonexistent student | 23503 | enrollments_student_id_fkey | 404 Not Found (or 422 Unprocessable Entity) |
| 10 | deleting a course that has enrollments | 23503 | enrollments_course_id_fkey | 409 Conflict (course with enrollments cannot be deleted) |
| 11 | deleting a student (ON DELETE CASCADE) | N/A (success) | N/A | Deleting a student deletes their enrollments by cascade (6 -> 3 enrollments left, 6 after ROLLBACK) |
| 12 | inserting a manual ID into students.id | 428C9 | GENERATED ALWAYS AS IDENTITY | 500 Internal Server Error (attempt to insert non-DEFAULT value into identity column) |

### Why repeat Pydantic checks in the database?
Pydantic performs input validation in the application memory, but it only protects the data entering through the API endpoints. Database constraints act as the final line of defense against invalid data introduced by SQL scripts, direct `psql` administration, migration bugs, or race conditions from parallel requests. Constraints ensure long-term data integrity regardless of application code updates.