# **ТЕХНИЧЕСКОЕ ЗАДАНИЕ**

**Создание API для информационной системы «СУ Учебный центр» с использованием FastAPI**

---

## **1. Цель задания**

Связать готовую базу данных PostgreSQL с сервером FastAPI, создать REST API для управления данными учебного центра. Студенты должны реализовать серверный уровень трёхуровневого приложения:

1. **База данных** — PostgreSQL (готовое ТЗ выполнено).
2. **Серверный уровень** — FastAPI + SQLAlchemy (ORM).
3. **Клиентский уровень** — тестирование через Postman или веб-браузер.

---

## **2. Требования к API**

### 2.1 Основные ресурсы и эндпоинты

#### **Студенты (`student`)**

* GET `/students` — получить список всех студентов
* GET `/students/{student_id}` — получить студента по ID
* POST `/students` — добавить нового студента
* PUT `/students/{student_id}` — обновить данные студента
* DELETE `/students/{student_id}` — удалить студента

#### **Преподаватели (`teacher`)**

* GET `/teachers` — список преподавателей
* POST `/teachers` — добавить преподавателя

#### **Курсы (`course`)**

* GET `/courses` — список курсов
* POST `/courses` — добавить курс

#### **Группы (`study_group`)**

* GET `/groups` — список групп
* POST `/groups` — создать группу
* GET `/groups/{group_id}/students` — получить список студентов группы

#### **Посещаемость (`attendance`)**

* GET `/lessons/{lesson_id}/attendance` — список посещаемости
* POST `/lessons/{lesson_id}/attendance` — отметить посещаемость

#### **Оценки (`grade`)**

* GET `/students/{student_id}/grades` — оценки студента
* POST `/grades` — добавить оценку

#### **Платежи (`payment`)**

* GET `/students/{student_id}/payments` — платежи студента
* POST `/payments` — добавить платеж

---

### 2.2 Технические требования

1. Использовать **FastAPI** (Python).
2. Подключение к PostgreSQL через **SQLAlchemy + psycopg2**.
3. Использовать Pydantic-модели для валидации данных.
4. Реализовать базовую обработку ошибок (404, 400, 500).
5. Подготовить документацию к API (Swagger автоматически через FastAPI).

---

## **3. Подсказки по структуре проекта**

```
university_api/
│
├── app/
│   ├── main.py            # запуск FastAPI-приложения
│   ├── models.py          # SQLAlchemy модели для всех таблиц
│   ├── schemas.py         # Pydantic модели для сериализации и валидации
│   ├── database.py        # подключение к PostgreSQL
│   ├── crud.py            # функции CRUD для работы с таблицами
│   └── routers/
│       ├── students.py
│       ├── teachers.py
│       ├── courses.py
│       ├── groups.py
│       ├── grades.py
│       ├── payments.py
│       └── attendance.py
│
├── requirements.txt       # зависимости: fastapi, uvicorn, sqlalchemy, psycopg2-binary
└── README.md
```

---

## **4. Подсказки по реализации**

### 4.1 Подключение к PostgreSQL

```python
# app/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql+psycopg2://username:password@localhost:5432/university"

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
```

---

### 4.2 Пример модели и схемы студента

```python
# app/models.py
from sqlalchemy import Column, Integer, String, Date
from .database import Base

class Student(Base):
    __tablename__ = "student"
    student_id = Column(Integer, primary_key=True, index=True)
    first_name = Column(String(50), nullable=False)
    last_name = Column(String(50), nullable=False)
    birth_date = Column(Date, nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    phone = Column(String(20))
    registration_date = Column(Date)
```

```python
# app/schemas.py
from pydantic import BaseModel, EmailStr
from datetime import date

class StudentCreate(BaseModel):
    first_name: str
    last_name: str
    birth_date: date
    email: EmailStr
    phone: str | None = None

class Student(StudentCreate):
    student_id: int
    registration_date: date

    class Config:
        orm_mode = True
```

---

### 4.3 Пример CRUD-функций

```python
# app/crud.py
from sqlalchemy.orm import Session
from . import models, schemas

def get_students(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.Student).offset(skip).limit(limit).all()

def create_student(db: Session, student: schemas.StudentCreate):
    db_student = models.Student(**student.dict())
    db.add(db_student)
    db.commit()
    db.refresh(db_student)
    return db_student
```

---

### 4.4 Пример маршрута

```python
# app/routers/students.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from .. import crud, schemas, database

router = APIRouter(prefix="/students", tags=["students"])

def get_db():
    db = database.SessionLocal()
    try:
        yield db
    finally:
        db.close()

@router.get("/", response_model=list[schemas.Student])
def read_students(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    return crud.get_students(db, skip=skip, limit=limit)

@router.post("/", response_model=schemas.Student)
def create_student(student: schemas.StudentCreate, db: Session = Depends(get_db)):
    return crud.create_student(db, student)
```

---

### 4.5 Запуск приложения

```bash
uvicorn app.main:app --reload
```

После запуска автоматически доступна документация:

* Swagger: `http://127.0.0.1:8000/docs`
* ReDoc: `http://127.0.0.1:8000/redoc`

---

## **5. Рекомендации студентам**

1. Создайте все таблицы и связи по выполненному ТЗ PostgreSQL.
2. Настройте подключение FastAPI к этой базе данных.
3. Создайте Pydantic-схемы для всех таблиц.
4. Реализуйте CRUD-функции для основных таблиц: `student`, `teacher`, `course`, `study_group`.
5. Тестируйте все эндпоинты через Postman или браузер.
6. По желанию: добавьте дополнительные эндпоинты для отчётов, среднего балла, суммы платежей и др.
