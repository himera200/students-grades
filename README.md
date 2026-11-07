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







from src.students_grades.database.database import get_connection
from src.students_grades.models.teacher import Teacher


def _map(row) -> Teacher:
    return Teacher(*row)


_SELECT = """
    SELECT t.teacher_id, t.user_id, t.department,
           u.surname || ' ' || u.name
    FROM teachers t
    JOIN users u ON u.user_id = t.user_id
"""


def create(user_id: int, department: str) -> int:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        INSERT INTO teachers (user_id, department)
        VALUES (?, ?)
    """, (user_id, department))
    conn.commit()
    new_id = cur.lastrowid
    conn.close()
    return new_id


def get_by_id(teacher_id: int) -> Teacher | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " WHERE t.teacher_id = ?", (teacher_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_by_user_id(user_id: int) -> Teacher | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " WHERE t.user_id = ?", (user_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_all() -> list[Teacher]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " ORDER BY u.surname, u.name")
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def update(teacher_id: int, department: str):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        UPDATE teachers
        SET department = ?
        WHERE teacher_id = ?
    """, (department, teacher_id))
    conn.commit()
    conn.close()


def delete(teacher_id: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        DELETE FROM teachers
        WHERE teacher_id = ?
    """, (teacher_id,))
    conn.commit()
    conn.close()



from src.students_grades.database.database import get_connection
from src.students_grades.models.grade import Grade


def _map(row) -> Grade:
    return Grade(*row)


_SELECT = """
    SELECT g.grade_id, g.student_id, g.subject_id, g.teacher_id,
           g.value, g.grade_date, g.comment,
           su.surname || ' ' || su.name,
           s.name,
           tu.surname || ' ' || tu.name
    FROM grades g
    JOIN students st ON st.student_id = g.student_id
    JOIN users su ON su.user_id = st.user_id
    JOIN subjects s ON s.subject_id = g.subject_id
    JOIN teachers t ON t.teacher_id = g.teacher_id
    JOIN users tu ON tu.user_id = t.user_id
"""


def create(
    student_id: int,
    subject_id: int,
    teacher_id: int,
    value: int,
    comment: str | None
) -> int:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        INSERT INTO grades (
            student_id, subject_id, teacher_id,
            value, grade_date, comment
        )
        VALUES (?, ?, ?, ?, date('now'), ?)
    """, (student_id, subject_id, teacher_id, value, comment))
    conn.commit()
    new_id = cur.lastrowid
    conn.close()
    return new_id


def get_by_id(grade_id: int) -> Grade | None:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " WHERE g.grade_id = ?", (grade_id,))
    row = cur.fetchone()
    conn.close()
    return _map(row) if row else None


def get_by_student(student_id: int) -> list[Grade]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " WHERE g.student_id = ? ORDER BY g.grade_date DESC",
                (student_id,))
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def get_by_student_and_subject(student_id: int, subject_id: int) -> list[Grade]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + """
        WHERE g.student_id = ? AND g.subject_id = ?
        ORDER BY g.grade_date DESC
    """, (student_id, subject_id))
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def get_by_group_and_subject(group_id: int, subject_id: int) -> list[Grade]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + """
        WHERE st.group_id = ? AND g.subject_id = ?
        ORDER BY su.surname, g.grade_date
    """, (group_id, subject_id))
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def get_all() -> list[Grade]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute(_SELECT + " ORDER BY g.grade_date DESC")
    rows = cur.fetchall()
    conn.close()
    return [_map(r) for r in rows]


def update(grade_id: int, value: int, comment: str | None):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        UPDATE grades
        SET value = ?, comment = ?
        WHERE grade_id = ?
    """, (value, comment, grade_id))
    conn.commit()
    conn.close()


def delete(grade_id: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        DELETE FROM grades
        WHERE grade_id = ?
    """, (grade_id,))
    conn.commit()
    conn.close()

from src.students_grades.database.database import get_connection
from src.students_grades.models.subject import Subject
from src.students_grades.models.teacher import Teacher


def assign(teacher_id: int, subject_id: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        INSERT INTO teacher_subjects (teacher_id, subject_id)
        VALUES (?, ?)
    """, (teacher_id, subject_id))
    conn.commit()
    conn.close()


def unassign(teacher_id: int, subject_id: int):
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        DELETE FROM teacher_subjects
        WHERE teacher_id = ? AND subject_id = ?
    """, (teacher_id, subject_id))
    conn.commit()
    conn.close()


def is_assigned(teacher_id: int, subject_id: int) -> bool:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT 1
        FROM teacher_subjects
        WHERE teacher_id = ? AND subject_id = ?
    """, (teacher_id, subject_id))
    row = cur.fetchone()
    conn.close()
    return row is not None


def get_subjects_by_teacher(teacher_id: int) -> list[Subject]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT s.subject_id, s.name, s.hours, s.semester
        FROM teacher_subjects ts
        JOIN subjects s ON s.subject_id = ts.subject_id
        WHERE ts.teacher_id = ?
        ORDER BY s.name
    """, (teacher_id,))
    rows = cur.fetchall()
    conn.close()
    return [Subject(*r) for r in rows]


def get_teachers_by_subject(subject_id: int) -> list[Teacher]:
    conn = get_connection()
    cur = conn.cursor()
    cur.execute("""
        SELECT t.teacher_id, t.user_id, t.department,
               u.surname || ' ' || u.name
        FROM teacher_subjects ts
        JOIN teachers t ON t.teacher_id = ts.teacher_id
        JOIN users u ON u.user_id = t.user_id
        WHERE ts.subject_id = ?
        ORDER BY u.surname, u.name
    """, (subject_id,))
    rows = cur.fetchall()
    conn.close()
    return [Teacher(*r) for r in rows]






import hashlib

from src.students_grades.models.user import User
from src.students_grades.repositories import (
    role_repository,
    student_group_repository,
    student_repository,
    teacher_repository,
    user_repository,
)

MIN_PASSWORD_LENGTH = 6


def hash_password(password: str) -> str:
    return hashlib.sha256(password.encode("utf-8")).hexdigest()


def _validate_user_data(
    surname: str,
    name: str,
    email: str,
    login: str,
    password: str
):
    if not surname.strip():
        raise ValueError("Фамилия не может быть пустой")
    if not name.strip():
        raise ValueError("Имя не может быть пустым")
    if "@" not in email or "." not in email:
        raise ValueError("Некорректный email")
    if not login.strip():
        raise ValueError("Логин не может быть пустым")
    if len(password) < MIN_PASSWORD_LENGTH:
        raise ValueError(
            f"Пароль должен быть не короче {MIN_PASSWORD_LENGTH} символов"
        )
    if user_repository.get_by_login(login):
        raise ValueError("Логин уже занят")
    if user_repository.get_by_email(email):
        raise ValueError("Email уже зарегистрирован")


def _create_user(
    surname: str,
    name: str,
    patronymic: str | None,
    email: str,
    login: str,
    password: str,
    role_name: str
) -> int:
    role = role_repository.get_by_name(role_name)
    if not role:
        raise ValueError("Роль не найдена")
    return user_repository.create(
        surname.strip(),
        name.strip(),
        patronymic.strip() if patronymic and patronymic.strip() else None,
        email.strip(),
        login.strip(),
        hash_password(password),
        role.role_id
    )


def register_student(
    surname: str,
    name: str,
    patronymic: str | None,
    email: str,
    login: str,
    password: str,
    group_id: int,
    record_book_number: str
):
    _validate_user_data(surname, name, email, login, password)

    if not student_group_repository.get_by_id(group_id):
        raise ValueError("Группа не найдена")
    if not record_book_number.strip():
        raise ValueError("Номер зачётной книжки не может быть пустым")
    for s in student_repository.get_all():
        if s.record_book_number == record_book_number.strip():
            raise ValueError("Студент с таким номером зачётки уже есть")

    user_id = _create_user(
        surname, name, patronymic, email, login, password, "Student"
    )
    student_repository.create(user_id, group_id, record_book_number.strip())


def register_teacher(
    surname: str,
    name: str,
    patronymic: str | None,
    email: str,
    login: str,
    password: str,
    department: str
):
    _validate_user_data(surname, name, email, login, password)

    if not department.strip():
        raise ValueError("Кафедра не может быть пустой")

    user_id = _create_user(
        surname, name, patronymic, email, login, password, "Teacher"
    )
    teacher_repository.create(user_id, department.strip())


def authenticate(login: str, password: str) -> User | None:
    user = user_repository.get_by_login(login.strip())
    if not user:
        return None
    if user.password_hash != hash_password(password):
        return None
    return user
'8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92'


from src.students_grades.models.grade import Grade
from src.students_grades.repositories import (
    grade_repository,
    student_group_repository,
    student_repository,
    subject_repository,
    teacher_repository,
    teacher_subject_repository,
)

MIN_GRADE = 2
MAX_GRADE = 5
DEBT_THRESHOLD = 3.0


def _check_value(value: int):
    if not isinstance(value, int) or not MIN_GRADE <= value <= MAX_GRADE:
        raise ValueError(
            f"Оценка должна быть целым числом от {MIN_GRADE} до {MAX_GRADE}"
        )


def add_grade(
    teacher_id: int,
    student_id: int,
    subject_id: int,
    value: int,
    comment: str | None
):
    _check_value(value)

    if not teacher_repository.get_by_id(teacher_id):
        raise ValueError("Преподаватель не найден")
    if not student_repository.get_by_id(student_id):
        raise ValueError("Студент не найден")
    if not subject_repository.get_by_id(subject_id):
        raise ValueError("Предмет не найден")
    if not teacher_subject_repository.is_assigned(teacher_id, subject_id):
        raise ValueError("Преподаватель не ведёт этот предмет")

    comment = comment.strip() if comment and comment.strip() else None
    grade_repository.create(
        student_id, subject_id, teacher_id, value, comment
    )


def edit_grade(
    teacher_id: int,
    grade_id: int,
    value: int,
    comment: str | None
):
    _check_value(value)

    grade = grade_repository.get_by_id(grade_id)
    if not grade:
        raise ValueError("Оценка не найдена")
    if grade.teacher_id != teacher_id:
        raise ValueError("Можно изменять только свои оценки")

    comment = comment.strip() if comment and comment.strip() else None
    grade_repository.update(grade_id, value, comment)


def delete_grade(teacher_id: int, grade_id: int):
    grade = grade_repository.get_by_id(grade_id)
    if not grade:
        raise ValueError("Оценка не найдена")
    if grade.teacher_id != teacher_id:
        raise ValueError("Можно удалять только свои оценки")
    grade_repository.delete(grade_id)


def get_student_grades(student_id: int) -> list[Grade]:
    if not student_repository.get_by_id(student_id):
        raise ValueError("Студент не найден")
    return grade_repository.get_by_student(student_id)


def calculate_average(grades: list[Grade]) -> float | None:
    if not grades:
        return None
    return sum(g.value for g in grades) / len(grades)


def get_student_average(student_id: int) -> float | None:
    return calculate_average(get_student_grades(student_id))


def get_student_debts(student_id: int) -> list[tuple[str, float]]:
    grades = get_student_grades(student_id)
    by_subject: dict[str, list[Grade]] = {}
    for g in grades:
        by_subject.setdefault(g.subject_name, []).append(g)

    debts = []
    for subject_name, subject_grades in by_subject.items():
        avg = calculate_average(subject_grades)
        if avg < DEBT_THRESHOLD:
            debts.append((subject_name, avg))
    return debts


def get_group_report(group_id: int, subject_id: int) -> list[dict]:
    if not student_group_repository.get_by_id(group_id):
        raise ValueError("Группа не найдена")
    if not subject_repository.get_by_id(subject_id):
        raise ValueError("Предмет не найден")

    report = []
    for student in student_repository.get_by_group(group_id):
        grades = grade_repository.get_by_student_and_subject(
            student.student_id, subject_id
        )
        avg = calculate_average(grades)
        if avg is None:
            status = "нет оценок"
        elif avg < DEBT_THRESHOLD:
            status = "на риске отчисления"
        else:
            status = "успевает"
        report.append({
            "student": student.full_name,
            "grades": [g.value for g in grades],
            "average": avg,
            "status": status,
        })
    return report
