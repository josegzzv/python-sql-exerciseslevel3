# Advanced SQLite3 Exercises with Faker

This document provides a set of advanced exercises for working with SQLite3 databases populated with fictional data using the Faker library. These exercises are designed to challenge students and allow them to apply advanced SQL and Python techniques over a 3 to 4-hour period.

## Table of Contents

1. [Exercise 1: Creating a Projects Table and Relating it to Students and Courses](#exercise-1-creating-a-projects-table-and-relating-it-to-students-and-courses)
2. [Exercise 2: Analysis of Attendance and Academic Performance](#exercise-2-analysis-of-attendance-and-academic-performance)
3. [Exercise 3: Creating a Course Recommendation System](#exercise-3-creating-a-course-recommendation-system)

---

## Exercise 1: Creating a Projects Table and Relating it to Students and Courses

### Objective

Create a new table called `proyectos` that stores information about projects completed by students in various courses. The projects should be linked to both a course and a student, and should record the project name, submission date, and the grade received.

### Instructions

1. **Create the `proyectos` Table:**
   - The table should have the following fields:
     - `id` (INTEGER, PRIMARY KEY, AUTOINCREMENT)
     - `alumno_id` (INTEGER, FOREIGN KEY to the `alumnos` table)
     - `curso_id` (INTEGER, FOREIGN KEY to the `cursos` table)
     - `nombre_proyecto` (TEXT)
     - `fecha_entrega` (DATE)
     - `calificacion` (REAL)

2. **Populate the `proyectos` Table:**
   - Use `Faker` to generate fictional project data for students.
   - Ensure that some students have multiple projects in different courses, and that some projects have missing grades.

3. **Required Queries:**
   - Find the average project grades per course.
   - Identify students who have submitted more than one project in a specific course.
   - Identify projects that were submitted late (submission date after a defined deadline).

### Example Code to Create and Populate the Table

```python
import sqlite3
import random
from faker import Faker

# Connect to SQLite database
conexion = sqlite3.connect('escuela.db')
cursor = conexion.cursor()

# Create an instance of Faker
fake = Faker()

# Create the projects table
cursor.execute('''
CREATE TABLE IF NOT EXISTS proyectos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    alumno_id INTEGER,
    curso_id INTEGER,
    nombre_proyecto TEXT NOT NULL,
    fecha_entrega DATE,
    calificacion REAL,
    FOREIGN KEY(alumno_id) REFERENCES alumnos(id),
    FOREIGN KEY(curso_id) REFERENCES cursos(id)
)
''')

# Populate the projects table
for _ in range(1000):  # Create 1000 projects
    alumno_id = random.choice(alumnos_ids)
    curso_id = random.choice(cursos_ids)
    nombre_proyecto = fake.catch_phrase()
    fecha_entrega = fake.date_this_year()
    calificacion = random.choice([random.uniform(0, 10), None])  # Some missing grades
    cursor.execute('''
    INSERT INTO proyectos (alumno_id, curso_id, nombre_proyecto, fecha_entrega, calificacion)
    VALUES (?, ?, ?, ?, ?)
    ''', (alumno_id, curso_id, nombre_proyecto, fecha_entrega, calificacion))

conexion.commit()
conexion.close()
```



## Exercise 2: Analysis of Attendance and Academic Performance

### Objective

Analyze the correlation between students' attendance in classes and their grades in courses. Additionally, students should identify patterns that may suggest how influential attendance is on academic performance.

### Instructions

1. **Calculate Correlation:**
    - Use SQL to calculate the correlation between the number of attendances and the grades received by students in each course.
    - Consider using statistical methods in Python, such as the corr() function from pandas, if needed.
2. **Identify Patterns:**
    - Identify courses where high attendance does not seem to correlate with high grades.
    - Find students who have high attendance but low grades, and vice versa.
3. **Report Results:**
    - Generate a report that shows courses with the highest and lowest correlation between attendance and performance.
    - Visualize these results using charts, such as scatter plots, if necessary.

### Example Code to Calculate Correlation

```python
import sqlite3
import pandas as pd

# Connect to SQLite database
conexion = sqlite3.connect('escuela.db')

# Query attendance and grades data
query = '''
SELECT a.alumno_id, a.curso_id, a.asistencias, c.calificacion
FROM asistencias a
JOIN calificaciones c ON a.alumno_id = c.alumno_id AND a.curso_id = c.curso_id
WHERE a.asistencias IS NOT NULL AND c.calificacion IS NOT NULL
'''

df = pd.read_sql_query(query, conexion)

# Calculate the correlation between attendance and grades
correlation = df.groupby('curso_id')[['asistencias', 'calificacion']].corr().iloc[0::2, -1]
print(correlation)

conexion.close()

```

## Exercise 3: Creating a Course Recommendation System

### Objective

Create a basic course recommendation system for students based on their past performance and interest in certain topics.

### Instructions

1. **Create intereses Table:**
    - The table should store students' interests in different topics related to the courses.
    - Suggested fields:
        - id (INTEGER, PRIMARY KEY, AUTOINCREMENT)
        - alumno_id (INTEGER, FOREIGN KEY to the alumnos table)
        - tema (TEXT)
        - nivel_interes (INTEGER) # A value between 1 and 5
2. **Populate the intereses Table:**
    - Use Faker to generate topics and interest levels for students.
    - Each student should have multiple interests in different topics.
3. **Recommendation System:**
    - Develop an algorithm that recommends courses to students based on:
        - Courses where they have performed well.
        - Declared interests in topics related to those courses.
    - The recommendation should prioritize courses related to the student's interests and where they have shown good performance.
4. **Report Results:**
    - Generate a list of course recommendations for each student.
    - Explain how interests and past performance influence the recommendations.

### Example Code to Create and Populate the intereses Table

```python
import sqlite3
import random
from faker import Faker

# Connect to SQLite database
conexion = sqlite3.connect('escuela.db')
cursor = conexion.cursor()

# Create an instance of Faker
fake = Faker()

# Create the interests table
cursor.execute('''
CREATE TABLE IF NOT EXISTS intereses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    alumno_id INTEGER,
    tema TEXT NOT NULL,
    nivel_interes INTEGER,
    FOREIGN KEY(alumno_id) REFERENCES alumnos(id)
)
''')

# Populate the interests table
temas = ['Matemáticas', 'Historia', 'Ciencias', 'Literatura', 'Arte', 'Tecnología', 'Deporte', 'Música']
for alumno_id in alumnos_ids:
    for _ in range(random.randint(1, 5)):  # Each student has between 1 and 5 interests
        tema = random.choice(temas)
        nivel_interes = random.randint(1, 5)
        cursor.execute('''
        INSERT INTO intereses (alumno_id, tema, nivel_interes)
        VALUES (?, ?, ?)
        ''', (alumno_id, tema, nivel_interes))

conexion.commit()
conexion.close()

```

## Conclusion

These exercises are designed to challenge students with more complex SQL and Python tasks, requiring them to work with multiple related tables, perform advanced data analysis, and create basic recommendation systems. Students should aim to spend ~~3 to 4 hours~~ working through these exercises, applying their knowledge and extending their skills in database management and data manipulation. But since you are all incredibly sharp, you’ll probably finish them in under 2 hours!
**Let's go Group 103! You got this!**

---
## Disclaimer

These exercises have been created for educational purposes and are designed to challenge and enhance students' skills in database management and data analysis using SQLite and Python. The data used in these exercises is fictional and generated automatically using the Faker library.

The use of this material in production environments is not recommended without thorough review and additional modifications. The author is not responsible for any damage, data loss, or misunderstandings that may arise from the use of this material.
---

## License

This project is licensed under the MIT License. For more details, please see the [LICENSE](./LICENSE) file.
