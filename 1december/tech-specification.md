# **ТЕХНИЧЕСКОЕ ЗАДАНИЕ

на разработку базы данных
Информационной системы «СУ Учебный центр»**

---

## **1. Назначение системы**

База данных предназначена для хранения, обработки и обеспечения целостности данных, связанных с деятельностью учебного центра: управлением курсами, студентами, преподавателями, группами, расписанием, посещаемостью и оплатами.

---

## **2. Требования к системе**

Каждый студент обязан выполнить данное ТЗ в полном объёме и представить PostgreSQL-скрипт, создающий все таблицы, связи, ограничения, индексы, триггеры и функции, указанные ниже.

Все названия таблиц и полей — **строго обязательные**.

---

# **3. Структура базы данных**

База данных должна содержать следующие таблицы:

---

# **3.1. Таблица `student` (Студенты)**

| Поле              | Тип          | Ограничения                                                       | Описание         |
| ----------------- | ------------ | ----------------------------------------------------------------- | ---------------- |
| student_id        | SERIAL       | PRIMARY KEY                                                       | Идентификатор    |
| first_name        | VARCHAR(50)  | NOT NULL                                                          | Имя              |
| last_name         | VARCHAR(50)  | NOT NULL                                                          | Фамилия          |
| birth_date        | DATE         | NOT NULL, CHECK (birth_date < CURRENT_DATE - INTERVAL '14 years') | Дата рождения    |
| email             | VARCHAR(100) | NOT NULL, UNIQUE                                                  | Email            |
| phone             | VARCHAR(20)  | NULL                                                              | Телефон          |
| registration_date | DATE         | NOT NULL DEFAULT CURRENT_DATE                                     | Дата регистрации |

---

# **3.2. Таблица `teacher` (Преподаватели)**

| Поле           | Тип          | Ограничения     |
| -------------- | ------------ | --------------- |
| teacher_id     | SERIAL       | PRIMARY KEY     |
| first_name     | VARCHAR(50)  | NOT NULL        |
| last_name      | VARCHAR(50)  | NOT NULL        |
| email          | VARCHAR(100) | NOT NULL UNIQUE |
| phone          | VARCHAR(20)  | NULL            |
| specialization | VARCHAR(100) | NOT NULL        |

---

# **3.3. Таблица `course` (Курсы)**

| Поле           | Тип           | Ограничения                         |
| -------------- | ------------- | ----------------------------------- |
| course_id      | SERIAL        | PRIMARY KEY                         |
| title          | VARCHAR(100)  | NOT NULL                            |
| description    | TEXT          | NULL                                |
| duration_hours | INT           | NOT NULL CHECK (duration_hours > 0) |
| price          | NUMERIC(10,2) | NOT NULL CHECK (price >= 0)         |

---

# **3.4. Таблица `group` (Учебные группы)**

*(название таблицы обязательно `study_group`, так как group — зарезервированное слово)*

| Поле         | Тип    | Ограничения                                                |
| ------------ | ------ | ---------------------------------------------------------- |
| group_id     | SERIAL | PRIMARY KEY                                                |
| course_id    | INT    | NOT NULL REFERENCES course(course_id) ON DELETE CASCADE    |
| teacher_id   | INT    | NOT NULL REFERENCES teacher(teacher_id) ON DELETE RESTRICT |
| start_date   | DATE   | NOT NULL                                                   |
| end_date     | DATE   | NOT NULL, CHECK(end_date > start_date)                     |
| max_students | INT    | NOT NULL CHECK(max_students BETWEEN 5 AND 50)              |

---

# **3.5. Таблица `group_student` (Запись студента в группу)**

**Связь M:N между student и study_group**

| Поле                               | Тип  | Ограничения                                                 |
| ---------------------------------- | ---- | ----------------------------------------------------------- |
| group_id                           | INT  | NOT NULL REFERENCES study_group(group_id) ON DELETE CASCADE |
| student_id                         | INT  | NOT NULL REFERENCES student(student_id) ON DELETE CASCADE   |
| enrollment_date                    | DATE | NOT NULL DEFAULT CURRENT_DATE                               |
| PRIMARY KEY (group_id, student_id) |      |                                                             |

Доп. ограничение:

* студент **не может быть записан** в группу, если в ней уже достигнут лимит max_students.

---

# **3.6. Таблица `lesson` (Занятия)**

| Поле        | Тип          | Ограничения                                                 |
| ----------- | ------------ | ----------------------------------------------------------- |
| lesson_id   | SERIAL       | PRIMARY KEY                                                 |
| group_id    | INT          | NOT NULL REFERENCES study_group(group_id) ON DELETE CASCADE |
| lesson_date | DATE         | NOT NULL CHECK (lesson_date >= start_date(group_id))        |
| start_time  | TIME         | NOT NULL                                                    |
| end_time    | TIME         | NOT NULL CHECK (end_time > start_time)                      |
| topic       | VARCHAR(200) | NOT NULL                                                    |

Доп. правило:

* преподаватель **не может вести два занятия в одно время**.

---

# **3.7. Таблица `attendance` (Посещаемость)**

| Поле                                | Тип         | Ограничения                                               |
| ----------------------------------- | ----------- | --------------------------------------------------------- |
| lesson_id                           | INT         | NOT NULL REFERENCES lesson(lesson_id) ON DELETE CASCADE   |
| student_id                          | INT         | NOT NULL REFERENCES student(student_id) ON DELETE CASCADE |
| status                              | VARCHAR(10) | NOT NULL CHECK (status IN ('present', 'absent'))          |
| PRIMARY KEY (lesson_id, student_id) |             |                                                           |

---

# **3.8. Таблица `grade` (Оценки)**

| Поле        | Тип    | Ограничения                                    |
| ----------- | ------ | ---------------------------------------------- |
| grade_id    | SERIAL | PRIMARY KEY                                    |
| student_id  | INT    | NOT NULL REFERENCES student(student_id)        |
| course_id   | INT    | NOT NULL REFERENCES course(course_id)          |
| grade_value | INT    | NOT NULL CHECK (grade_value BETWEEN 0 AND 100) |
| grade_date  | DATE   | NOT NULL DEFAULT CURRENT_DATE                  |

---

# **3.9. Таблица `payment` (Платежи студентов)**

| Поле         | Тип           | Ограничения                                          |
| ------------ | ------------- | ---------------------------------------------------- |
| payment_id   | SERIAL        | PRIMARY KEY                                          |
| student_id   | INT           | NOT NULL REFERENCES student(student_id)              |
| course_id    | INT           | NOT NULL REFERENCES course(course_id)                |
| amount       | NUMERIC(10,2) | NOT NULL CHECK (amount > 0)                          |
| payment_date | TIMESTAMP     | NOT NULL DEFAULT CURRENT_TIMESTAMP                   |
| method       | VARCHAR(20)   | NOT NULL CHECK(method IN ('cash', 'card', 'online')) |

---

# **4. Требуемые связи между таблицами**

Все связи обязаны быть реализованы:

* **student 1:N payment**
* **student 1:N grade**
* **course 1:N grade**
* **course 1:N study_group**
* **teacher 1:N study_group**
* **study_group 1:N lesson**
* **student M:N study_group → group_student**
* **lesson 1:N attendance**

---

# **5. Требования к обеспечению целостности**

### **5.1 Доменная целостность**

Обязательные условия уже указаны в CHECK-ограничениях.

### **5.2 Ссылочная целостность**

Все внешние ключи должны иметь корректные ON DELETE и ON UPDATE.

### **5.3 Бизнес-ограничения (реализуются триггерами)**

Каждый студент обязан реализовать следующие триггеры:

---

## **Триггер 1: Ограничение количества студентов в группе**

**group_student BEFORE INSERT**

Проверить:

```
SELECT COUNT(*) FROM group_student WHERE group_id = NEW.group_id
```

Если значение ≥ max_students (из study_group) → запретить вставку.

---

## **Триггер 2: Преподаватель не может вести два занятия одновременно**

**lesson BEFORE INSERT OR UPDATE**

Проверить, есть ли у того же teacher_id другое занятие, время которого пересекается.

---

## **Триггер 3: Оценка может быть выставлена только студенту, записанному на курс**

**grade BEFORE INSERT**

Проверить, что student_id присутствует в group_student для групп этого course_id.

---

# **6. Индексы (обязательные)**

Каждый студент обязан создать индексы:

1. **Индекс для поиска студентов по email**

   ```
   CREATE UNIQUE INDEX idx_student_email ON student(email);
   ```

2. **Индекс по lesson (group_id, lesson_date)**
   для ускорения расписания.

3. **Индекс по payment (student_id, payment_date)**

4. **Индекс по grade (student_id, course_id)**

---

# **7. Требуемые функции**

Каждый студент обязан реализовать **минимум две функции**:

---

### **Функция 1: Получить среднюю оценку студента по курсу**

**get_student_avg_grade(student_id INT, course_id INT)**
Возвращает NUMERIC.

---

### **Функция 2: Получить сумму всех платежей студента**

**get_total_payments(student_id INT)**
Возвращает NUMERIC.

---

# **8. Формат сдачи**

Студент сдаёт:

1. **SQL-скрипт PostgreSQL** создания всех объектов:

   * CREATE TABLE
   * ALTER TABLE
   * CREATE INDEX
   * CREATE FUNCTION
   * CREATE TRIGGER
2. Скрипт должен выполняться ПОЛНОСТЬЮ и без ошибок.
3. Скрипт должен соответствовать 100% требований ТЗ.

---
