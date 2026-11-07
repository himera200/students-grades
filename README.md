INSERT INTO users (surname, name, patronymic, email, login, password_hash, role_id)
VALUES ('Иванов', 'Иван', 'Иванович', 'ivanov@mail.ru', 'ivanov', 'hash1', 1),
       ('Петрова', 'Анна', 'Сергеевна', 'petrova@mail.ru', 'petrova', 'hash2', 1),
       ('Сидоров', 'Пётр', 'Алексеевич', 'sidorov@mail.ru', 'sidorov', 'hash3', 1),
       ('Смирнова', 'Ольга', 'Николаевна', 'smirnova@mail.ru', 'smirnova', 'hash4', 2),
       ('Козлов', 'Дмитрий', 'Андреевич', 'kozlov@mail.ru', 'kozlov', 'hash5', 2),
       ('Админов', 'Админ', NULL, 'admin@mail.ru', 'admin', 'hash6', 3);

INSERT INTO students (user_id, group_id, record_book_number)
VALUES (1, 1, 'ЗК-1001'),
       (2, 1, 'ЗК-1002'),
       (3, 2, 'ЗК-1003');

INSERT INTO teachers (user_id, department)
VALUES (4, 'Кафедра информатики'),
       (5, 'Кафедра математики');

INSERT INTO teacher_subjects (teacher_id, subject_id)
VALUES (1, 1), (1, 3), (1, 4),
       (2, 2);

INSERT INTO grades (student_id, subject_id, teacher_id, value, grade_date, comment)
VALUES (1, 1, 1, 5, '2026-10-01', 'Отлично'),
       (1, 3, 1, 4, '2026-10-10', NULL),
       (2, 1, 1, 4, '2026-10-01', NULL),
       (2, 2, 2, 3, '2026-10-05', 'Нужно подтянуть'),
       (3, 2, 2, 5, '2026-10-05', NULL),
       (3, 4, 1, 4, '2026-10-12', NULL);

## 3. Проектирование базы данных

База данных реализована в SQLite. Структура описана в файле `src/students_grades/database/schema.sql`.

![ER-диаграмма](docs/diagrams/ER.png)

import sqlite3
from pathlib import Path


DB_NAME = "students_grades.db"


def get_connection():
    conn = sqlite3.connect(DB_NAME)
    conn.execute("PRAGMA foreign_keys = ON")
    return conn


def init_db():
    if Path(DB_NAME).exists():
        return

    conn = get_connection()

    sql_path = Path(__file__).parent / "schema.sql"

    with open(sql_path, "r", encoding="utf-8") as file:
        sql_script = file.read()

    conn.executescript(sql_script)
    conn.commit()
    conn.close()

    print("База данных создана.")

    from src.students_grades.database.database import init_db


def main():
    init_db()
    print("===== Система учета оценок студента =====")


if __name__ == "__main__":
    main()

## 3. Проектирование базы данных
База данных реализована в SQLite. Структура и тестовые данные описаны в файле `src/students_grades/database/schema.sql`. При запуске программы база создаётся автоматически функцией `init_db()` из файла `src/students_grades/database/database.py`.

![ER-диаграмма](docs/diagrams/ER.png)

Таблицы базы данных:
* `user_roles` — роли пользователей;
* `users` — пользователи системы;
* `students` — студенты (связь 1:1 с пользователями);
* `teachers` — преподаватели (связь 1:1 с пользователями);
* `student_groups` — учебные группы;
* `subjects` — предметы;
* `teacher_subjects` — связь «преподаватель — предмет» (многие-ко-многим);
* `grades` — оценки.

## 4. Запуск


from src.students_grades.database.database import get_connection
from src.students_grades.models.subject import Subject


def _map(row) -> Subject:
    return Subject(*row)


def create(name: str, hours: int, semester: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        INSERT INTO subjects (name, hours, semester)
        VALUES (?, ?, ?)
    """, (name, hours, semester))
    conn.commit()
    conn.close()


def get_by_id(subject_id: int) -> Subject | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT subject_id, name, hours, semester
        FROM subjects
        WHERE subject_id = ?
    """, (subject_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_all() -> list[Subject]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT subject_id, name, hours, semester
        FROM subjects
        ORDER BY name
    """)
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def update(subject_id: int, name: str, hours: int, semester: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        UPDATE subjects
        SET name = ?, hours = ?, semester = ?
        WHERE subject_id = ?
    """, (name, hours, semester, subject_id))
    conn.commit()
    conn.close()


def delete(subject_id: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        DELETE FROM subjects
        WHERE subject_id = ?
    """, (subject_id,))
    conn.commit()
    conn.close()

from src.students_grades.database.database import get_connection
from src.students_grades.models.student_group import StudentGroup


def _map(row) -> StudentGroup:
    return StudentGroup(*row)


def create(name: str, course: int, faculty: str):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        INSERT INTO student_groups (name, course, faculty)
        VALUES (?, ?, ?)
    """, (name, course, faculty))
    conn.commit()
    conn.close()


def get_by_id(group_id: int) -> StudentGroup | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT group_id, name, course, faculty
        FROM student_groups
        WHERE group_id = ?
    """, (group_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_all() -> list[StudentGroup]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT group_id, name, course, faculty
        FROM student_groups
        ORDER BY name
    """)
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def update(group_id: int, name: str, course: int, faculty: str):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        UPDATE student_groups
        SET name = ?, course = ?, faculty = ?
        WHERE group_id = ?
    """, (name, course, faculty, group_id))
    conn.commit()
    conn.close()


def delete(group_id: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        DELETE FROM student_groups
        WHERE group_id = ?
    """, (group_id,))
    conn.commit()
    conn.close()

    from src.students_grades.database.database import get_connection
from src.students_grades.models.user_role import UserRole


def _map(row) -> UserRole:
    return UserRole(*row)


def get_by_id(role_id: int) -> UserRole | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT role_id, name
        FROM user_roles
        WHERE role_id = ?
    """, (role_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_by_name(name: str) -> UserRole | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT role_id, name
        FROM user_roles
        WHERE name = ?
    """, (name,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_all() -> list[UserRole]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT role_id, name
        FROM user_roles
        ORDER BY role_id
    """)
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]



    from src.students_grades.database.database import get_connection
from src.students_grades.models.user import User


def _map(row) -> User:
    return User(*row)


def create(
    surname: str,
    name: str,
    patronymic: str | None,
    email: str,
    login: str,
    password_hash: str,
    role_id: int
) -> int:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        INSERT INTO users (
            surname, name, patronymic,
            email, login, password_hash, role_id
        )
        VALUES (?, ?, ?, ?, ?, ?, ?)
    """, (surname, name, patronymic, email, login, password_hash, role_id))
    conn.commit()
    new_id = cur.lastrowid
    conn.close()
    return new_id


def get_by_id(user_id: int) -> User | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT u.user_id, u.surname, u.name, u.patronymic,
               u.email, u.login, u.password_hash, u.role_id,
               r.name
        FROM users u
        JOIN user_roles r ON r.role_id = u.role_id
        WHERE u.user_id = ?
    """, (user_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_by_login(login: str) -> User | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT u.user_id, u.surname, u.name, u.patronymic,
               u.email, u.login, u.password_hash, u.role_id,
               r.name
        FROM users u
        JOIN user_roles r ON r.role_id = u.role_id
        WHERE u.login = ?
    """, (login,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_by_email(email: str) -> User | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT u.user_id, u.surname, u.name, u.patronymic,
               u.email, u.login, u.password_hash, u.role_id,
               r.name
        FROM users u
        JOIN user_roles r ON r.role_id = u.role_id
        WHERE u.email = ?
    """, (email,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_all() -> list[User]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT u.user_id, u.surname, u.name, u.patronymic,
               u.email, u.login, u.password_hash, u.role_id,
               r.name
        FROM users u
        JOIN user_roles r ON r.role_id = u.role_id
        ORDER BY u.surname, u.name
    """)
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def update(
    user_id: int,
    surname: str,
    name: str,
    patronymic: str | None,
    email: str
):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        UPDATE users
        SET surname = ?, name = ?, patronymic = ?, email = ?
        WHERE user_id = ?
    """, (surname, name, patronymic, email, user_id))
    conn.commit()
    conn.close()


def delete(user_id: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        DELETE FROM users
        WHERE user_id = ?
    """, (user_id,))
    conn.commit()
    conn.close()





from src.students_grades.database.database import get_connection
from src.students_grades.models.student import Student


def _map(row) -> Student:
    return Student(*row)


_SELECT = """
    SELECT s.student_id, s.user_id, s.group_id, s.record_book_number,
           u.surname || ' ' || u.name,
           g.name
    FROM students s
    JOIN users u ON u.user_id = s.user_id
    JOIN student_groups g ON g.group_id = s.group_id
"""


def create(user_id: int, group_id: int, record_book_number: str) -> int:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        INSERT INTO students (user_id, group_id, record_book_number)
        VALUES (?, ?, ?)
    """, (user_id, group_id, record_book_number))
    conn.commit()
    new_id = cur.lastrowid
    conn.close()
    return new_id


def get_by_id(student_id: int) -> Student | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " WHERE s.student_id = ?", (student_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_by_user_id(user_id: int) -> Student | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " WHERE s.user_id = ?", (user_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_all() -> list[Student]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " ORDER BY u.surname, u.name")
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def get_by_group(group_id: int) -> list[Student]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " WHERE s.group_id = ? ORDER BY u.surname, u.name",
                (group_id,))
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def update(student_id: int, group_id: int, record_book_number: str):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        UPDATE students
        SET group_id = ?, record_book_number = ?
        WHERE student_id = ?
    """, (group_id, record_book_number, student_id))
    conn.commit()
    conn.close()


def delete(student_id: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        DELETE FROM students
        WHERE student_id = ?
    """, (student_id,))
    conn.commit()
    conn.close()
