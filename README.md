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
