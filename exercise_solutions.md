
# Solutions and Explanations for Advanced SQLite3 Exercises

This document contains the solutions to the advanced SQLite3 exercises along with detailed explanations of the code and concepts used. These solutions are intended to help students understand how to approach and solve complex database-related tasks using SQLite and Python.

## Table of Contents

1. [Exercise 1: Creating a Projects Table and Relating it to Students and Courses](#exercise-1-creating-a-projects-table-and-relating-it-to-students-and-courses)
2. [Exercise 2: Analysis of Attendance and Academic Performance](#exercise-2-analysis-of-attendance-and-academic-performance)
3. [Exercise 3: Creating a Course Recommendation System](#exercise-3-creating-a-course-recommendation-system)
4. [Conclusion](#conclusion)

---

## Exercise 1: Creating a Projects Table and Relating it to Students and Courses

### Objective

Create a new table called `proyectos` that stores information about projects completed by students in various courses. The projects should be linked to both a course and a student, and should record the project name, submission date, and the grade received.

### Solution

1. **Creating the `proyectos` Table:**

   ```python
   import sqlite3

   # Connect to the SQLite database
   conexion = sqlite3.connect('escuela.db')
   cursor = conexion.cursor()

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

   conexion.commit()
   conexion.close()
   ```

   **Explanation:**
   - **SQLite Database Connection:** We first connect to the SQLite database (`escuela.db`). If this database doesn't exist, it will be created automatically.
   - **Table Creation:** The `proyectos` table is created with fields to store details such as project name, submission date, and grade. The `alumno_id` and `curso_id` fields are foreign keys that link the projects to the `alumnos` and `cursos` tables, respectively.

2. **Populating the `proyectos` Table:**

   ```python
   import sqlite3
   import random
   from faker import Faker

   # Connect to SQLite database
   conexion = sqlite3.connect('escuela.db')
   cursor = conexion.cursor()

   # Create an instance of Faker
   fake = Faker()

   # Populate the projects table
   alumnos_ids = cursor.execute("SELECT id FROM alumnos").fetchall()
   cursos_ids = cursor.execute("SELECT id FROM cursos").fetchall()

   for _ in range(1000):  # Create 1000 projects
       alumno_id = random.choice(alumnos_ids)[0]
       curso_id = random.choice(cursos_ids)[0]
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

   **Explanation:**
   - **Faker Library:** We use the Faker library to generate realistic project names and submission dates. Faker helps in creating data that mimics real-world entries.
   - **Random Selection:** `random.choice()` is used to randomly assign each project to a student (`alumno_id`) and a course (`curso_id`). 
   - **Inserting Data:** The `INSERT INTO` SQL command adds each project to the `proyectos` table. Some projects have missing grades (`None`) to simulate incomplete data.

3. **Required Queries:**

   - **Average Project Grades per Course:**

     ```python
     cursor.execute('''
     SELECT c.nombre_curso, AVG(p.calificacion) as promedio_calificacion
     FROM proyectos p
     JOIN cursos c ON p.curso_id = c.id
     WHERE p.calificacion IS NOT NULL
     GROUP BY c.nombre_curso
     ''')
     resultados = cursor.fetchall()
     for resultado in resultados:
         print(resultado)
     ```

     **Explanation:**
     - This query calculates the average grade (`AVG`) for projects in each course, excluding projects with missing grades (`IS NOT NULL`).

   - **Students with Multiple Projects in a Course:**

     ```python
     cursor.execute('''
     SELECT p.alumno_id, a.nombre, p.curso_id, c.nombre_curso, COUNT(*) as num_proyectos
     FROM proyectos p
     JOIN alumnos a ON p.alumno_id = a.id
     JOIN cursos c ON p.curso_id = c.id
     GROUP BY p.alumno_id, p.curso_id
     HAVING COUNT(*) > 1
     ''')
     resultados = cursor.fetchall()
     for resultado in resultados:
         print(resultado)
     ```

     **Explanation:**
     - This query identifies students who have submitted more than one project in the same course. The `HAVING COUNT(*) > 1` clause filters out students with only one project per course.

   - **Projects Submitted Late:**

     ```python
     deadline = '2023-12-01'  # Example deadline
     cursor.execute('''
     SELECT p.nombre_proyecto, p.fecha_entrega, a.nombre as alumno, c.nombre_curso
     FROM proyectos p
     JOIN alumnos a ON p.alumno_id = a.id
     JOIN cursos c ON p.curso_id = c.id
     WHERE p.fecha_entrega > ?
     ''', (deadline,))
     resultados = cursor.fetchall()
     for resultado in resultados:
         print(resultado)
     ```

     **Explanation:**
     - This query finds projects that were submitted after a specified deadline. The `WHERE p.fecha_entrega > ?` clause is used to filter the results.

---

## Exercise 2: Analysis of Attendance and Academic Performance

### Objective

Analyze the correlation between students' attendance in classes and their grades in courses. Additionally, students should identify patterns that may suggest how influential attendance is on academic performance.

### Solution

1. **Calculating Correlation:**

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

   **Explanation:**
   - **Correlation Calculation:** We use the `pandas` library to calculate the correlation between attendance and grades for each course. The `corr()` function calculates the Pearson correlation coefficient, showing how strongly attendance is related to academic performance.
   - **Data Querying:** The `JOIN` clause combines data from `asistencias` and `calificaciones` tables based on student and course IDs.

2. **Identifying Patterns:**

   - **High Attendance, Low Grades:**

     ```python
     df_high_attendance_low_grades = df[(df['asistencias'] > 15) & (df['calificacion'] < 5)]
     print(df_high_attendance_low_grades)
     ```

     **Explanation:**
     - This filter identifies students with high attendance (more than 15 classes) but low grades (below 5). It helps highlight cases where students attend regularly but do not perform well academically.

   - **Low Attendance, High Grades:**

     ```python
     df_low_attendance_high_grades = df[(df['asistencias'] < 5) & (df['calificacion'] > 8)]
     print(df_low_attendance_high_grades)
     ```

     **Explanation:**
     - This filter finds students with low attendance (fewer than 5 classes) but high grades (above 8). These cases may indicate students who perform well despite attending fewer classes.

3. **Reporting Results:**

   - **Visualizing Correlation:**

     ```python
     import matplotlib.pyplot as plt

     df_grouped = df.groupby('curso_id')[['asistencias', 'calificacion']].mean()
     plt.scatter(df_grouped['asistencias'], df_grouped['calificacion'])
     plt.xlabel('Average Attendance')
     plt.ylabel('Average Grade')
     plt.title('Attendance vs. Grade Correlation')
     plt.show()
     ```

     **Explanation:**
     - A scatter plot is used to visualize the relationship between attendance and grades across different courses. This helps in identifying trends and understanding how attendance impacts academic performance.

---

## Exercise 3: Creating a Course Recommendation System

### Objective

Create a basic course recommendation system for students based on their past performance and interest in certain topics.

### Solution

1. **Creating and Populating the `intereses` Table:**

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
   alumnos_ids = cursor.execute("SELECT id FROM alumnos").fetchall()

   for alumno_id in alumnos_ids:
       alumno_id = alumno_id[0]
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

   **Explanation:**
   - **Creating the Interests Table:** This script creates the `intereses` table to store students' interests in various topics. The `nivel_interes` field records how interested a student is in a particular topic, on a scale of 1 to 5.
   - **Populating the Table:** The `Faker` library is used to generate data for students' interests. Each student can have interests in multiple topics, making the data realistic and varied.

2. **Developing the Recommendation System:**

   ```python
   import sqlite3

   # Connect to SQLite database
   conexion = sqlite3.connect('escuela.db')
   cursor = conexion.cursor()

   # Recommendation system based on interests and performance
   def recomendar_cursos(alumno_id):
       cursor.execute('''
       SELECT i.tema, c.nombre_curso, AVG(ca.calificacion) as promedio_calificacion
       FROM intereses i
       JOIN cursos c ON i.tema LIKE '%' || c.nombre_curso || '%'
       JOIN calificaciones ca ON ca.curso_id = c.id
       WHERE i.alumno_id = ? AND ca.alumno_id = ?
       GROUP BY c.nombre_curso
       HAVING promedio_calificacion >= 7
       ORDER BY i.nivel_interes DESC, promedio_calificacion DESC
       ''', (alumno_id, alumno_id))
       
       recomendaciones = cursor.fetchall()
       return recomendaciones

   # Example usage
   for alumno_id in alumnos_ids[:10]:  # Show recommendations for the first 10 students
       print(f"Recommendations for student {alumno_id[0]}:")
       recomendaciones = recomendar_cursos(alumno_id[0])
       for curso in recomendaciones:
           print(curso)

   conexion.close()
   ```

   **Explanation:**
   - **Recommendation Algorithm:** This function recommends courses to students based on their interests and past academic performance. Courses related to topics where students have shown a high level of interest and achieved good grades are prioritized.
   - **Querying the Data:** The query matches student interests with courses and calculates the average grade for each course. Only courses where the student has performed well (average grade >= 7) are recommended.

---

## Congratulations on making it this far!

You’re tackling advanced exercises that not only challenge your technical skills but also strengthen your ability to solve complex problems. Remember, every mistake you encounter and correct is an opportunity to learn. It’s not about reaching the destination as quickly as possible, but about enjoying the journey of discovery.

These exercises are more than just tasks; they are the building blocks of your future in development and data analysis. With every line of code, every query you refine, you’re one step closer to mastering the art of managing databases and effectively manipulating data.

### Keep pushing forward with confidence, persevere through the challenges, and never underestimate the power of your curiosity. You are capable of achieving great things!
