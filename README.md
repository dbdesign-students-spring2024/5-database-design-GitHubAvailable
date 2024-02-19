# Data Normalization and Entity-Relationship Diagramming

An assignment to normalize the structure of data and establish a set of Entity-Relationship Diagrams for the data.

<!-- The contents of this file will be deleted and replaced with the content described in the [instructions](./instructions.md) -->

## Original Dataset
| assignment_id | student_id | due_date | professor | assignment_topic                | classroom | grade | relevant_reading    | professor_email   |
| :------------ | :--------- | :------- | :-------- | :------------------------------ | :-------- | :---- | :------------------ | :---------------- |
| 1             | 1          | 23.02.21 | Melvin    | Data normalization              | WWH 101   | 80    | Deumlich Chapter 3  | l.melvin@foo.edu  |
| 2             | 7          | 18.11.21 | Logston   | Single table queries            | 60FA 314  | 25    | Dümmlers Chapter 11 | e.logston@foo.edu |
| 1             | 4          | 23.02.21 | Melvin    | Data normalization              | WWH 101   | 75    | Deumlich Chapter 3  | l.melvin@foo.edu  |
| 5             | 2          | 05.05.21 | Logston   | Python and pandas               | 60FA 314  | 92    | Dümmlers Chapter 14 | e.logston@foo.edu |
| 4             | 2          | 04.07.21 | Nevarez   | Spreadsheet aggregate functions | WWH 201   | 65    | Zehnder Page 87     | i.nevarez@foo.edu |
| ...           | ...        | ...      | ...       | ...                             | ...       | ...   | ...                 | ...               |

## Problems
1. To safitfy 4NF, this table must satisfy 3NF. However, it does not because does not have a column that has unique values. In addition, even if surrogate keys are given, some fields still only relate to non-key fields. For instance, `classroom` is only related to the course but not the students' grade on an assignment.
2. Suppose the table uses `(assignment_id, student_id)` as composite key. But `assignment_topic` is only related to `assignment_id`. This violates 2NF, which is the prerequiste of 3NF.
3. There are two multi-valued columns - `due_date` and `professor`, since one professor can teach multiple sections and give the same homework with different due dates and the same assignment with the same due date could be given by different professors.

## Solution
We split data into following tables:
### Table `score`
Primary Key: `(assignment_id, student_id)`
| assignment_id | student_id | grade |
| :------------ | :--------- | :---- |
| 1             | 1          | 80    |
| 2             | 7          | 25    |
| 1             | 4          | 75    |
| 5             | 2          | 92    |
| 4             | 2          | 65    |
| ...           | ...        | ...   |

### Table `class_task`
Primary Key: `(assignment_id, class_id)`
| class_id | assignment_id | due_date | relevant_reading    |
| :------- | :------------ | :------- | :------------------ |
| DATA-11  | 1             | 23.02.21 | Deumlich Chapter 3  |
| COMP-11  | 2             | 18.11.21 | Dümmlers Chapter 11 |
| COMP-12  | 5             | 05.05.21 | Dümmlers Chapter 14 |
| DATA-21  | 4             | 04.07.21 | Zehnder Page 87     |
| ...      | ...           | ...      | ...                 |

### Table `assignment`
Primary Key: `assignment_id`
| id  | topic                           |
| :-- | :------------------------------ |
| 1   | Data normalization              |
| 2   | Single table queries            |
| 5   | Python and pandas               |
| 4   | Spreadsheet aggregate functions |
| ... | ...                             |

### Table `class`
Primary Key: `id`
| id      | course_id | professor_id | section | classroom |
| :------ | :-------- | :----------- | :------ | :-------- |
| DATA-11 | DS-10     | 1            | A       | WWH 101   |
| COMP-11 | CS-20     | 2            | A       | 60FA 314  |
| COMP-12 | CS-20     | 2            | B       | 60FA 314  |
| DATA-21 | DS-20     | 3            | A       | WWH 201   |
| ...     | ...       | ...          | ...     | ...       |

### Table `professor`
Primary Key: `id`
| id  | full_name | email             |
| :-- | :-------- | :---------------- |
| 1   | E.Logston | e.logston@foo.edu |
| 2   | I.Melvin  | i.melvin@foo.edu  |
| 3   | I.Nevarez | i.nevarez@foo.edu |
| ... | ...       | ...               |

## ER Diagram
The ER diagram of these tables are given below:
![ER Diagram of 4NF Dataset](./images/normalized_er_diagram.svg)

## Description
The new dataset is 4NF-compliant because each table is 4NF-compliant.

### Change: Create table `score` and `task`
Table `score` and `task` are 4NF-compliant because:
1. They are both **2NF-complaint** because every non-key field provides facts about the primary key as a whole: 
   + `score`: A value of `grade` represents the grade of **a specific student** for **a specific assignment**. 
   + `task`: Each non-key field describes a fact about **an assignment** in **a specific class** (notice that `relevant_reading` is included because two classes could use identical homework questions but with different relevant reading material due to different progress).
2. They are both **3NF-complaint** because no non-key field is another fact about another non-key field:
   + `score`: This is automatically satisfied because `grade` is the only non-key field. 
   + `class_task`: `due_date` and `relevant_reading` are unrelated.
3. They have **at most 1 multi-valued field**:
   + `score`: 0 multi-valued field, since one student can have at most one score for each assignment.
   + `class_task`: Even though an assignment can have more than one `relevant_reading`, it is the only possible multi-valued field as one assignment at a given class cannot have 2 different deadlines.

### Change: Create table `assignment`, `class`, and `professor`
Table `assignment`, `class`, and `professor` are 4NF-compliant because:
1. They are all **3NF-complaint** because no non-key field is another fact about another non-key field: 
   + `assignment`: This is automatically satisfied because `topic` is the only non-key field.
   + `class`: Any two non-key fields are unrelated. 
   + `professor`: `full_name` is unrelated to `email` (as a professor could also use an email ending in domain of specific department).
2. They have **at most 1 multi-valued field**.
   + `assignment`: At most 1, since `assignment_topic` is the only non-key field. 
   + `class`: each record represents a specific section of a specific course, and by assumption "each section meets in a specific classroom with a specific professor," each non-key field can only have 1 possible value.
   + `professor`: While a professor can have multiple email, he or she can only have one possible `full_name` at any given time.