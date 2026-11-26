/* lab4_var12.cpp – Листинг программы для 4-ой лабораторной работы
*******************************************************************
* Filename : lab4_var12.cpp
* Abstract : Лабораторная работа 4, вариант 12
* Description : Заданы матрицы A (N x M), B (M x L), C (L x K).
*               Программа находит максимальные элементы amax, bmax,
*               cmax и значение beta = min{|amax|, |bmax|, |cmax|}.
*               Данные вводятся вручную или генерируются ЛКГ.
*               Размер каждой матрицы не более 300 элементов.
*               Весь код находится в одном файле.
* Creation Date : 2026 / 11 / 22
* Author : <ФИО студента>, группа <___>
* Notes / Platform / Copyrights : ISO C++17, любая ОС
******************************************************************/
#include <iostream>
#include <iomanip>
#include <sstream>
#include <string>
#include <cstdlib>
#include <cstdint>
#include <new>

#ifdef _WIN32
#include <windows.h>
#endif

/* Размер каждой матрицы – не более 300 элементов (по заданию)   */
const int MAX_MATRIX_SIZE = 300;

/* Границы значений при ручном вводе                              */
const double ELEM_MIN = -10000.0;
const double ELEM_MAX = 10000.0;

/*=================================================================*/
/*   МОДУЛЬ 1. Ввод данных                                         */
/*=================================================================*/
/* Ограничения на запись числа */
static const std::size_t MAX_INT_DIGITS      = 9; /* до 9 цифр         */
static const int         MAX_INTEGER_DIGITS  = 9; /* цифр до точки     */
static const int         MAX_FRACTION_DIGITS = 3; /* цифр после точки  */

static bool ReadLine(std::string* line) {
    bool isRead = static_cast<bool>(std::getline(std::cin, *line));
    if (isRead) {
        const char* spaces = " \t\r\n";
        std::size_t first = line->find_first_not_of(spaces);
        if (first == std::string::npos) {
            line->clear();               /* пустая строка */
        } else {
            std::size_t last = line->find_last_not_of(spaces);
            *line = line->substr(first, last - first + 1);
        }
    }
    return isRead;
}

static bool ParseInt(const std::string& text, int minVal, int maxVal,
                     int* value) {
    bool isOk = true;
    bool isNegative = false;
    std::size_t pos = 0;
    long long acc = 0;

    if (!text.empty() && (text[0] == '+' || text[0] == '-')) {
        isNegative = (text[0] == '-');
        pos = 1;
    }
    if (pos >= text.size() || text.size() - pos > MAX_INT_DIGITS) {
        isOk = false;                    /* нет цифр или слишком длинно */
    }
    for (std::size_t i = pos; isOk && i < text.size(); ++i) {
        if (text[i] < '0' || text[i] > '9') {
            isOk = false;                /* лишний символ */
        } else {
            acc = acc * 10 + (text[i] - '0');
        }
    }
    if (isOk) {
        if (isNegative) {
            acc = -acc;
        }
        if (acc < minVal || acc > maxVal) {
            isOk = false;                /* число вне границ */
        } else {
            *value = static_cast<int>(acc);
        }
    }
    return isOk;
}

static bool ParseDouble(const std::string& text, double minVal,
                        double maxVal, double* value) {
    bool isOk = true;
    bool isPointSeen = false;
    int intDigits = 0;
    int fracDigits = 0;
    std::size_t pos = 0;
    std::string clean;                   /* число с точкой вместо запятой */

    if (!text.empty() && (text[0] == '+' || text[0] == '-')) {
        clean += text[0];
        pos = 1;
    }
    for (std::size_t i = pos; isOk && i < text.size(); ++i) {
        char ch = text[i];
        if (ch >= '0' && ch <= '9') {
            clean += ch;
            if (isPointSeen) {
                ++fracDigits;
            } else {
                ++intDigits;
            }
        } else if ((ch == '.' || ch == ',') && !isPointSeen) {
            isPointSeen = true;
            clean += '.';
        } else {
            isOk = false;                /* лишний символ */
        }
    }
    if (isOk) {
        isOk = (intDigits >= 1 && intDigits <= MAX_INTEGER_DIGITS)
            && (fracDigits <= MAX_FRACTION_DIGITS)
            && (!isPointSeen || fracDigits >= 1);
    }
    if (isOk) {
        double number = std::strtod(clean.c_str(), nullptr);
        if (number == 0.0) {
            number = 0.0;                /* чтобы не было «-0» */
        }
        if (number < minVal || number > maxVal) {
            isOk = false;
        } else {
            *value = number;
        }
    }
    return isOk;
}

bool InputInt(const char* prompt, int minVal, int maxVal, int* result) {
    bool isDone = false;
    bool isAlive = true;
    std::string line;

    while (!isDone && isAlive) {
        std::cout << prompt;
        isAlive = ReadLine(&line);
        if (isAlive) {
            isDone = ParseInt(line, minVal, maxVal, result);
            if (!isDone) {
                std::cout << "  Ошибка: нужно ввести одно целое число от "
                          << minVal << " до " << maxVal << ".\n";
            }
        }
    }
    return isDone;
}

bool InputRow(const char* prompt, int count, double minVal,
              double maxVal, double* row) {
    bool isDone = false;
    bool isAlive = true;
    std::string line;

    while (!isDone && isAlive) {
        std::cout << prompt;
        isAlive = ReadLine(&line);
        if (isAlive) {
            std::istringstream stream(line);
            std::string token;
            int got = 0;
            bool isValid = true;

            while (isValid && (stream >> token)) {
                if (got >= count) {
                    isValid = false;     /* чисел больше, чем нужно */
                } else {
                    isValid = ParseDouble(token, minVal, maxVal, &row[got]);
                    if (isValid) {
                        ++got;
                    }
                }
            }
            isDone = isValid && (got == count);
            if (!isDone) {
                std::cout << "  Ошибка: нужно ввести ровно " << count
                          << " чис. через пробел, каждое от " << minVal
                          << " до " << maxVal
                          << ", не более 3 знаков после запятой.\n";
            }
        }
    }
    return isDone;
}

void WaitForEnter(const char* prompt) {
    std::string line;
    std::cout << prompt;
    ReadLine(&line);
}

/*=================================================================*/
/*   МОДУЛЬ 2. Работа с матрицами                                 */
/*=================================================================*/
/* Числа для генератора случайных чисел (ЛКГ):
   x = (A * x + C) mod 2^32 – «mod» получается сам (переполнение).  */
static const std::uint32_t LCG_A = 1664525u;
static const std::uint32_t LCG_C = 1013904223u;

/* Сгенерированные числа: от -100.000 до 100.000 с шагом 0.001     */
static const std::uint32_t GEN_STEPS = 200001u;
static const int GEN_MIN_THOUSANDTHS = -100000;

double* CreateMatrix(int rows, int cols) {
    double* matr = nullptr;
    if (rows > 0 && cols > 0) {
        std::size_t size = static_cast<std::size_t>(rows)
                         * static_cast<std::size_t>(cols);
        matr = new (std::nothrow) double[size]();
    }
    return matr;
}

void DestroyMatrix(double** matr) {
    if (matr != nullptr && *matr != nullptr) {
        delete[] *matr;
        *matr = nullptr;
    }
}

std::uint32_t NextRandom(std::uint32_t* state) {
    *state = LCG_A * (*state) + LCG_C;
    return *state;
}

void FillMatrixRandom(double* matr, int rows, int cols,
                      std::uint32_t* state) {
    int size = rows * cols;
    for (int idx = 0; idx < size; ++idx) {
        /* берём старшие биты: они «случайнее» младших */
        std::uint32_t highBits = NextRandom(state) >> 8;
        int thousandths = static_cast<int>(highBits % GEN_STEPS)
                        + GEN_MIN_THOUSANDTHS;
        matr[idx] = thousandths / 1000.0;
    }
}

void PrintMatrix(const char* name, const double* matr, int rows, int cols) {
    std::ios_base::fmtflags oldFlags = std::cout.flags();
    std::streamsize oldPrecision = std::cout.precision();

    std::cout << "Матрица " << name << " (" << rows << " x " << cols
              << "):\n";
    std::cout << std::fixed << std::setprecision(3);
    for (int line = 0; line < rows; ++line) {
        for (int col = 0; col < cols; ++col) {
            std::cout << std::setw(11) << matr[line * cols + col];
        }
        std::cout << '\n';
    }
    std::cout.flags(oldFlags);
    std::cout.precision(oldPrecision);
}

double AbsValue(double value) {
    double result = value;
    if (value < 0.0) {
        result = -value;
    }
    return result;
}

double MinOfThree(double first, double second, double third) {
    double result = first;
    if (second < result) {
        result = second;
    }
    if (third < result) {
        result = third;
    }
    return result;
}

double FindMaxElement(const double* matr, int rows, int cols) {
    double maxItem = 0.0;
    int size = rows * cols;
    if (matr != nullptr && size > 0) {
        maxItem = matr[0];
        for (int idx = 1; idx < size; ++idx) {
            if (matr[idx] > maxItem) {
                maxItem = matr[idx];
            }
        }
    }
    return maxItem;
}

/*=================================================================*/
/*   МОДУЛЬ 3. Главная программа                                  */
/*=================================================================*/
/* Данные студента – заменить на свои */
static const char* const STUDENT_INFO =
    "Выполнил: <ФИО студента>, группа <___>";

static bool InputDimensions(int* n, int* m, int* l, int* k) {
    std::string prompt;

    std::cout << "Матрицы: A имеет размер N x M, B - M x L, C - L x K.\n"
              << "Размер каждой матрицы (число элементов) не более "
              << MAX_MATRIX_SIZE << ".\n";

    prompt = "N - число строк матрицы A [1.." +
             std::to_string(MAX_MATRIX_SIZE) + "]: ";
    bool isOk = InputInt(prompt.c_str(), 1, MAX_MATRIX_SIZE, n);

    if (isOk) {
        prompt = "M - число столбцов A и строк B [1.." +
                 std::to_string(MAX_MATRIX_SIZE / *n) + "]: ";
        isOk = InputInt(prompt.c_str(), 1, MAX_MATRIX_SIZE / *n, m);
    }
    if (isOk) {
        prompt = "L - число столбцов B и строк C [1.." +
                 std::to_string(MAX_MATRIX_SIZE / *m) + "]: ";
        isOk = InputInt(prompt.c_str(), 1, MAX_MATRIX_SIZE / *m, l);
    }
    if (isOk) {
        prompt = "K - число столбцов матрицы C [1.." +
                 std::to_string(MAX_MATRIX_SIZE / *l) + "]: ";
        isOk = InputInt(prompt.c_str(), 1, MAX_MATRIX_SIZE / *l, k);
    }
    return isOk;
}

static bool InputMatrixManual(const char* name, double* matr,
                              int rows, int cols) {
    bool isOk = true;

    std::cout << "\nВвод матрицы " << name << " (" << rows << " x "
              << cols << "): по " << cols << " чис. в строке через пробел,\n"
              << "значения от " << ELEM_MIN << " до " << ELEM_MAX
              << ", не более 3 знаков после запятой.\n";
    for (int line = 0; isOk && line < rows; ++line) {
        std::string prompt = std::string(name) + ", строка " +
            std::to_string(line + 1) + " из " + std::to_string(rows) + ": ";
        isOk = InputRow(prompt.c_str(), cols, ELEM_MIN, ELEM_MAX,
                        &matr[line * cols]);
    }
    return isOk;
}

static bool RunSession(void) {
    int n = 0, m = 0, l = 0, k = 0;
    int mode = 0;
    double* matrA = nullptr;
    double* matrB = nullptr;
    double* matrC = nullptr;

    /* --- 1. Ввод размеров --- */
    bool isOk = InputDimensions(&n, &m, &l, &k);

    /* --- 2. Вывод размеров и выделение памяти --- */
    if (isOk) {
        std::cout << "\nВведены размеры: A = " << n << " x " << m
                  << ", B = " << m << " x " << l
                  << ", C = " << l << " x " << k << "\n";
        matrA = CreateMatrix(n, m);
        matrB = CreateMatrix(m, l);
        matrC = CreateMatrix(l, k);
        if (matrA == nullptr || matrB == nullptr || matrC == nullptr) {
            std::cout << "Ошибка: нет доступной динамической памяти!\n";
            isOk = false;
        }
    }

    /* --- 3. Выбор: ручной ввод или генерация --- */
    if (isOk) {
        isOk = InputInt("\nСпособ подготовки данных: 1 - ручной ввод, "
                        "2 - автоматическая генерация (ЛКГ).\nВаш выбор: ",
                        1, 2, &mode);
    }

    /* --- 4. Заполнение матриц --- */
    if (isOk && mode == 1) {
        isOk = InputMatrixManual("A", matrA, n, m);
        if (isOk) {
            isOk = InputMatrixManual("B", matrB, m, l);
        }
        if (isOk) {
            isOk = InputMatrixManual("C", matrC, l, k);
        }
    } else if (isOk) {
        int seed = 0;
        isOk = InputInt("Начальное значение генератора (seed), "
                        "целое от 0 до 999999999: ", 0, 999999999, &seed);
        if (isOk) {
            std::uint32_t state = static_cast<std::uint32_t>(seed);
            FillMatrixRandom(matrA, n, m, &state);
            FillMatrixRandom(matrB, m, l, &state);
            FillMatrixRandom(matrC, l, k, &state);
            std::cout << "Матрицы сгенерированы (значения от -100 до 100, "
                         "3 знака после запятой).\n";
        }
    }

    /* --- 5. Вывод матриц, расчёт и результат --- */
    if (isOk) {
        std::cout << "\n========== Исходные данные ==========\n";
        PrintMatrix("A", matrA, n, m);
        std::cout << '\n';
        PrintMatrix("B", matrB, m, l);
        std::cout << '\n';
        PrintMatrix("C", matrC, l, k);

        double aMax = FindMaxElement(matrA, n, m);
        double bMax = FindMaxElement(matrB, m, l);
        double cMax = FindMaxElement(matrC, l, k);
        double beta = MinOfThree(AbsValue(aMax), AbsValue(bMax),
                                 AbsValue(cMax));

        std::cout << "\n============ Результат ============\n"
                  << std::fixed << std::setprecision(3)
                  << "amax = " << aMax << "\n"
                  << "bmax = " << bMax << "\n"
                  << "cmax = " << cMax << "\n"
                  << "beta = min{|amax|, |bmax|, |cmax|} = "
                  << beta << "\n";
        std::cout.unsetf(std::ios_base::floatfield);
    }

    /* --- 6. Освобождение памяти (делается всегда) --- */
    DestroyMatrix(&matrA);
    DestroyMatrix(&matrB);
    DestroyMatrix(&matrC);
    return isOk;
}

/* *****************   Main function   *************************** */
int main(void) { /* *** BEGIN ********************************** */
/******************   Local variables   **************************/
    bool isGoOn = true;
    int again = 0;

/******************     Code Segment    **************************/
#ifdef _WIN32
    SetConsoleOutputCP(CP_UTF8);         /* русские буквы в консоли */
#endif
    std::cout << "Нижегородский государственный технический университет\n"
              << "Лабораторная работа 4. Вариант 12.\n"
              << "Функции, указатели, динамическая память.\n"
              << STUDENT_INFO << "\n\n";

    while (isGoOn) {
        isGoOn = RunSession();
        if (isGoOn) {
            isGoOn = InputInt("\nПовторить расчёт? 1 - да, 0 - нет: ",
                              0, 1, &again) && (again == 1);
        }
        std::cout << '\n';
    }
    std::cout << "========== Конец работы программы ==========\n";
    WaitForEnter("Нажмите Enter для выхода...");
    return 0;
} /* **** END ************************************************ */

















/* Лабораторная работа 5. Вариант 19.
 * Определить количество слов в тексте и поменять местами
 * первое и пятое слова.
 * Конец ввода: Ctrl+Z, затем Enter (Windows) / Ctrl+D (Linux). */
#include <cctype>
#include <cstddef>
#include <fstream>
#include <iostream>
#include <iterator>
#include <string>
#include <vector>

#ifdef _WIN32
#include <windows.h>
#endif

const unsigned int CONSOLE_CODEPAGE = 65001U;  // UTF-8 (для KOI-8R: 20866)

using std::cout;
using std::cin;
using std::string;
using std::vector;

const char *const INPUT_FILE  = "input.txt";
const char *const OUTPUT_FILE = "output.txt";

struct Word {
    std::size_t begin;
    std::size_t end;   // не включается
};

static bool isSpace(char c) {
    return std::isspace(static_cast<unsigned char>(c)) != 0;
}

static vector<Word> findWords(const string &text) {
    vector<Word> words;
    std::size_t i = 0U, n = text.size();
    while (i < n) {
        while (i < n && isSpace(text[i])) i++;
        if (i >= n) break;
        Word w;
        w.begin = i;
        while (i < n && !isSpace(text[i])) i++;
        w.end = i;
        words.push_back(w);
    }
    return words;
}

static string swapWords(const string &text, const Word &a, const Word &b) {
    string result;
    result += text.substr(0, a.begin);
    result += text.substr(b.begin, b.end - b.begin);
    result += text.substr(a.end, b.begin - a.end);
    result += text.substr(a.begin, a.end - a.begin);
    result += text.substr(b.end);
    return result;
}

static bool enterTextFromKeyboard() {
    cout << "Введите текст. Конец ввода: Ctrl+Z и Enter (Windows) "
            "или Ctrl+D (Linux).\n";
    string text, line;
    while (std::getline(cin, line)) {
        text += line;
        text += '\n';
    }
    std::ofstream out(INPUT_FILE, std::ios::binary);
    if (!out) return false;
    out << text;
    return static_cast<bool>(out);
}

static bool readFile(const char *name, string &text) {
    std::ifstream in(name, std::ios::binary);
    if (!in) return false;
    text.assign(std::istreambuf_iterator<char>(in),
                std::istreambuf_iterator<char>());
    return true;
}

int main() {
#ifdef _WIN32
    SetConsoleCP(CONSOLE_CODEPAGE);
    SetConsoleOutputCP(CONSOLE_CODEPAGE);
#endif
    int choice = 0;
    cout << "Лабораторная работа 5, вариант 19\n"
            "1 - ввести текст с клавиатуры (будет сохранен в "
         << INPUT_FILE << ")\n"
            "2 - использовать уже существующий файл " << INPUT_FILE << "\n"
            "Ваш выбор: ";
    cin >> choice;
    cin.ignore(10000, '\n');

    if (choice == 1) {
        if (!enterTextFromKeyboard()) {
            cout << "Ошибка: не удалось сохранить файл " << INPUT_FILE << "\n";
            return 1;
        }
        cout << "\nТекст сохранен в файл " << INPUT_FILE << "\n";
    }

    string text;
    if (!readFile(INPUT_FILE, text)) {
        cout << "Ошибка: не удалось открыть файл " << INPUT_FILE << "\n";
        return 1;
    }

    vector<Word> words = findWords(text);
    cout << "\nКоличество слов в тексте: " << words.size() << "\n";

    string result = text;
    if (words.size() >= 5U) {
        result = swapWords(text, words[0], words[4]);
        cout << "Первое и пятое слова поменяны местами.\n";
    } else {
        cout << "В тексте меньше пяти слов - обмен невозможен, "
                "текст не изменен.\n";
    }

    cout << "\nРезультат:\n" << result << "\n";

    std::ofstream out(OUTPUT_FILE, std::ios::binary);
    if (!out) {
        cout << "Ошибка: не удалось создать файл " << OUTPUT_FILE << "\n";
        return 1;
    }
    out << result;
    cout << "Результат сохранен в файл " << OUTPUT_FILE << "\n";
    return 0;
}





from src.students_grades.auth import login, register
from src.students_grades.database.database import init_db
from src.students_grades.menu import admin_menu, student_menu, teacher_menu

STUDENT_ROLE_ID = 1
TEACHER_ROLE_ID = 2
ADMIN_ROLE_ID = 3


def main():
    init_db()

    while True:
        print("\n===== Система учёта оценок студентов =====")
        print("1. Вход")
        print("2. Регистрация")
        print("0. Выход")

        choice = input("Выберите действие: ").strip()

        if choice == "1":
            user = login()
            if not user:
                continue

            if user.role_id == STUDENT_ROLE_ID:
                student_menu(user)
            elif user.role_id == TEACHER_ROLE_ID:
                teacher_menu(user)
            elif user.role_id == ADMIN_ROLE_ID:
                admin_menu(user)
        elif choice == "2":
            register()
        elif choice == "0":
            print("До свидания!")
            break
        else:
            print("Неверный выбор")


if __name__ == "__main__":
    main()








# ---------- Администратор ----------

def _groups_menu():
    while True:
        print_header("Группы")
        print("1. Показать")
        print("2. Добавить")
        print("3. Удалить")
        print("0. Назад")
        choice = ask_choice()

        if choice == "1":
            _show_groups()
        elif choice == "2":
            name = ask("Название")
            course = ask_int("Курс", 1, 6)
            faculty = ask("Факультет")
            do(admin_service.create_group, "Группа добавлена",
               name, course, faculty)
        elif choice == "3":
            _show_groups()
            group_id = ask_int("ID группы")
            do(admin_service.delete_group, "Группа удалена", group_id)
        elif choice == "0":
            break
        else:
            print("Неверный выбор")


def _subjects_menu():
    while True:
        print_header("Предметы")
        print("1. Показать")
        print("2. Добавить")
        print("3. Удалить")
        print("0. Назад")
        choice = ask_choice()

        if choice == "1":
            _show_subjects()
        elif choice == "2":
            name = ask("Название")
            hours = ask_int("Количество часов", 1)
            semester = ask_int("Семестр", 1, 12)
            do(admin_service.create_subject, "Предмет добавлен",
               name, hours, semester)
        elif choice == "3":
            _show_subjects()
            subject_id = ask_int("ID предмета")
            do(admin_service.delete_subject, "Предмет удалён", subject_id)
        elif choice == "0":
            break
        else:
            print("Неверный выбор")


def _students_menu():
    while True:
        print_header("Студенты")
        print("1. Показать")
        print("2. Удалить")
        print("0. Назад")
        choice = ask_choice()

        if choice == "1":
            _show_students()
        elif choice == "2":
            _show_students()
            student_id = ask_int("ID студента")
            if confirm("Удалить студента"):
                do(admin_service.delete_student, "Студент удалён", student_id)
        elif choice == "0":
            break
        else:
            print("Неверный выбор")


def _teachers_menu():
    while True:
        print_header("Преподаватели")
        print("1. Показать")
        print("2. Назначить на предмет")
        print("3. Снять с предмета")
        print("4. Удалить")
        print("0. Назад")
        choice = ask_choice()

        if choice == "1":
            _show_teachers()
        elif choice == "2":
            _show_teachers()
            teacher_id = ask_int("ID преподавателя")
            _show_subjects()
            subject_id = ask_int("ID предмета")
            do(admin_service.assign_teacher, "Преподаватель назначен",
               teacher_id, subject_id)
        elif choice == "3":
            _show_teachers()
            teacher_id = ask_int("ID преподавателя")
            subjects = run(admin_service.get_teacher_subjects, teacher_id)
            if subjects is not None:
                print_table(
                    ["ID", "Предмет"],
                    [[s.subject_id, s.name] for s in subjects]
                )
            subject_id = ask_int("ID предмета")
            do(admin_service.unassign_teacher, "Назначение снято",
               teacher_id, subject_id)
        elif choice == "4":
            _show_teachers()
            teacher_id = ask_int("ID преподавателя")
            if confirm("Удалить преподавателя"):
                do(admin_service.delete_teacher, "Преподаватель удалён",
                   teacher_id)
        elif choice == "0":
            break
        else:
            print("Неверный выбор")


def admin_menu(user):
    while True:
        print_header(f"Администратор: {user.login}")
        print("1. Группы")
        print("2. Предметы")
        print("3. Студенты")
        print("4. Преподаватели")
        print("0. Выход")
        choice = ask_choice()

        if choice == "1":
            _groups_menu()
        elif choice == "2":
            _subjects_menu()
        elif choice == "3":
            _students_menu()
        elif choice == "4":
            _teachers_menu()
        elif choice == "0":
            break
        else:
            print("Неверный выбор")










# ---------- Преподаватель ----------

def _pick_student_grade(teacher_id: int):
    """Выбор студента и показ его оценок. Возвращает student_id или None."""
    _show_students()
    student_id = ask_int("ID студента")
    grades = run(grade_service.get_student_grades, student_id)
    if grades is None:
        return None
    _show_grades(grades)
    return student_id if grades else None


def _add_grade(teacher):
    _show_students()
    student_id = ask_int("ID студента")

    subjects = admin_service.get_teacher_subjects(teacher.teacher_id)
    print_table(
        ["ID", "Предмет", "Часы", "Семестр"],
        [[s.subject_id, s.name, s.hours, s.semester] for s in subjects]
    )
    subject_id = ask_int("ID предмета")
    value = ask_int("Оценка (2-5)")
    comment = ask_optional("Комментарий")
    do(
        grade_service.add_grade,
        "Оценка выставлена",
        teacher.teacher_id, student_id, subject_id, value, comment
    )


def _edit_grade(teacher):
    if _pick_student_grade(teacher.teacher_id) is None:
        return
    grade_id = ask_int("ID оценки")
    value = ask_int("Новая оценка (2-5)")
    comment = ask_optional("Комментарий")
    do(
        grade_service.edit_grade,
        "Оценка изменена",
        teacher.teacher_id, grade_id, value, comment
    )


def _delete_grade(teacher):
    if _pick_student_grade(teacher.teacher_id) is None:
        return
    grade_id = ask_int("ID оценки")
    if confirm("Удалить оценку"):
        do(
            grade_service.delete_grade,
            "Оценка удалена",
            teacher.teacher_id, grade_id
        )


def _group_report():
    _show_groups()
    group_id = ask_int("ID группы")
    _show_subjects()
    subject_id = ask_int("ID предмета")
    report = run(grade_service.get_group_report, group_id, subject_id)
    if report is None:
        return
    print_table(
        ["Студент", "Оценки", "Средний балл", "Статус"],
        [[r["student"], ", ".join(map(str, r["grades"])) or "-",
          _avg(r["average"]), r["status"]] for r in report]
    )


def teacher_menu(user):
    teacher = run(auth_service.get_teacher_profile, user.user_id)
    if not teacher:
        return

    while True:
        print_header(f"Преподаватель: {user.surname} {user.name}")
        print("1. Мои предметы")
        print("2. Выставить оценку")
        print("3. Изменить оценку")
        print("4. Удалить оценку")
        print("5. Ведомость группы")
        print("0. Выход")
        choice = ask_choice()

        if choice == "1":
            subjects = admin_service.get_teacher_subjects(teacher.teacher_id)
            print_table(
                ["ID", "Предмет", "Часы", "Семестр"],
                [[s.subject_id, s.name, s.hours, s.semester]
                 for s in subjects]
            )
        elif choice == "2":
            _add_grade(teacher)
        elif choice == "3":
            _edit_grade(teacher)
        elif choice == "4":
            _delete_grade(teacher)
        elif choice == "5":
            _group_report()
        elif choice == "0":
            break
        else:
            print("Неверный выбор")









from src.students_grades.services import (
    admin_service,
    auth_service,
    grade_service,
)
from src.students_grades.ui_utils import (
    ask,
    ask_choice,
    ask_int,
    ask_optional,
    confirm,
    do,
    print_header,
    print_table,
    run,
)


def _avg(value) -> str:
    return "-" if value is None else f"{value:.2f}"


# ---------- Общие показы ----------

def _show_groups():
    print_table(
        ["ID", "Группа", "Курс", "Факультет"],
        [[g.group_id, g.name, g.course, g.faculty]
         for g in admin_service.get_groups()]
    )


def _show_subjects():
    print_table(
        ["ID", "Предмет", "Часы", "Семестр"],
        [[s.subject_id, s.name, s.hours, s.semester]
         for s in admin_service.get_subjects()]
    )


def _show_students():
    print_table(
        ["ID", "ФИО", "Группа", "Зачётка"],
        [[s.student_id, s.full_name, s.group_name, s.record_book_number]
         for s in admin_service.get_students()]
    )


def _show_teachers():
    print_table(
        ["ID", "ФИО", "Кафедра"],
        [[t.teacher_id, t.full_name, t.department]
         for t in admin_service.get_teachers()]
    )


def _show_grades(grades):
    print_table(
        ["ID", "Предмет", "Оценка", "Дата", "Преподаватель", "Комментарий"],
        [[g.grade_id, g.subject_name, g.value, g.grade_date,
          g.teacher_name, g.comment] for g in grades]
    )


# ---------- Студент ----------

def student_menu(user):
    student = run(auth_service.get_student_profile, user.user_id)
    if not student:
        return

    while True:
        print_header(f"Студент: {user.surname} {user.name}")
        print("1. Мои оценки")
        print("2. Средний балл")
        print("3. Задолженности")
        print("0. Выход")
        choice = ask_choice()

        if choice == "1":
            grades = run(grade_service.get_student_grades, student.student_id)
            if grades is not None:
                _show_grades(grades)
        elif choice == "2":
            grades = run(grade_service.get_student_grades, student.student_id)
            if grades is not None:
                avg = grade_service.calculate_average(grades)
                print(f"\nСредний балл: {_avg(avg)}")
        elif choice == "3":
            debts = run(grade_service.get_student_debts, student.student_id)
            if debts is not None:
                print_table(
                    ["Предмет", "Средний балл"],
                    [[name, _avg(avg)] for name, avg in debts]
                )
        elif choice == "0":
            break
        else:
            print("Неверный выбор")








from src.students_grades.services import admin_service, auth_service
from src.students_grades.ui_utils import (
    ask, ask_choice, ask_int, ask_optional, do, print_header, print_table,
)


def login():
    print_header("Авторизация")
    login_value = ask("Логин")
    password_value = ask("Пароль")

    user = auth_service.authenticate(login_value, password_value)
    if not user:
        print("\nНеверный логин или пароль")
        return None

    print(f"\nДобро пожаловать, {user.name} {user.surname}")
    return user


def register():
    print_header("Регистрация")
    print("1. Студент")
    print("2. Преподаватель")
    print("0. Назад")
    role = ask_choice("Кто вы")

    if role not in ("1", "2"):
        return

    surname = ask("Фамилия")
    name = ask("Имя")
    patronymic = ask_optional("Отчество")
    email = ask("Email")
    login_value = ask("Логин")
    password_value = ask("Пароль (не короче 6 символов)")

    if role == "1":
        print_table(
            ["ID", "Группа", "Курс", "Факультет"],
            [[g.group_id, g.name, g.course, g.faculty]
             for g in admin_service.get_groups()]
        )
        group_id = ask_int("ID группы")
        record_book = ask("Номер зачётной книжки")
        do(auth_service.register_student, "Регистрация прошла успешно",
           surname, name, patronymic, email,
           login_value, password_value, group_id, record_book)
    else:
        department = ask("Кафедра")
        do(auth_service.register_teacher, "Регистрация прошла успешно",
           surname, name, patronymic, email,
           login_value, password_value, department)







def ask(prompt: str, allow_empty: bool = False) -> str:
    while True:
        value = input(f"{prompt}: ").strip()
        if value or allow_empty:
            return value
        print("Поле не может быть пустым")


def ask_optional(prompt: str) -> str | None:
    value = input(f"{prompt} (Enter - пропустить): ").strip()
    return value or None


def ask_int(prompt: str, min_value: int | None = None,
            max_value: int | None = None) -> int:
    while True:
        raw = input(f"{prompt}: ").strip()
        try:
            value = int(raw)
        except ValueError:
            print("Нужно ввести целое число")
            continue
        if min_value is not None and value < min_value:
            print(f"Число должно быть не меньше {min_value}")
            continue
        if max_value is not None and value > max_value:
            print(f"Число должно быть не больше {max_value}")
            continue
        return value


def ask_choice(prompt: str = "Выберите действие") -> str:
    return input(f"{prompt}: ").strip()


def confirm(prompt: str) -> bool:
    while True:
        answer = input(f"{prompt} (д/н): ").strip().lower()
        if answer in ("д", "да", "y", "yes"):
            return True
        if answer in ("н", "нет", "n", "no"):
            return False
        print("Введите «д» или «н»")


def print_header(title: str) -> None:
    print(f"\n===== {title} =====")


def print_table(headers: list[str], rows: list[list]) -> None:
    if not rows:
        print("Нет данных")
        return
    text_rows = [["" if c is None else str(c) for c in row] for row in rows]
    widths = [max(len(headers[i]), *(len(r[i]) for r in text_rows))
              for i in range(len(headers))]

    def line(cells):
        return " | ".join(c.ljust(widths[i]) for i, c in enumerate(cells))

    print(line(headers))
    print("-+-".join("-" * w for w in widths))
    for row in text_rows:
        print(line(row))


def run(action, *args):
    """Вызывает сервис и возвращает результат; при ValueError печатает ошибку."""
    try:
        return action(*args)
    except ValueError as e:
        print(f"\nОшибка: {e}")
        return None


def do(action, success_message: str, *args) -> bool:
    """Для действий без результата: печатает успех или ошибку."""
    try:
        action(*args)
    except ValueError as e:
        print(f"\nОшибка: {e}")
        return False
    print(f"\n{success_message}")
    return True







#include <iostream>
#include <fstream>
#include <iomanip>
#include <cmath>
using namespace std;

const double A  = 0.0;
const double B  = 3.0;
const double DX = 0.1;
const int N = 31;        // (B - A) / DX + 1

double f(double x)
{
    return log(x * x - 2 * x + 2);
}

// вывод таблицы в любой поток (экран или файл)
void printTable(ostream& out, double T[2][N])
{
    const string line = "+----+--------+--------+----------------+----------------+";

    out << line << "\n";
    out << "| " << setw(2) << "i" << " | " << setw(6) << "X[i]" << " | " << setw(6) << "Y[i]"
        << " | " << setw(14) << "&X[i]" << " | " << setw(14) << "&Y[i]" << " |\n";
    out << line << "\n";

    out << fixed << setprecision(4);
    for (int i = 0; i < N; i++)
    {
        out << "| " << setw(2) << i
            << " | " << setw(6) << T[0][i]
            << " | " << setw(6) << T[1][i]
            << " | " << setw(14) << (void*)&T[0][i]
            << " | " << setw(14) << (void*)&T[1][i] << " |\n";
    }
    out << line << "\n";
}

int main()
{
    double T[2][N];

    for (int i = 0; i < N; i++)
    {
        T[0][i] = A + i * DX;
        T[1][i] = f(T[0][i]);
    }

    double ymax = T[1][0], ymin = T[1][0];
    int imax = 0, imin = 0;
    for (int i = 1; i < N; i++)
    {
        if (T[1][i] > ymax) { ymax = T[1][i]; imax = i; }
        if (T[1][i] < ymin) { ymin = T[1][i]; imin = i; }
    }

    ofstream fout("result.txt");
    ostream* streams[2] = { &cout, &fout };
    for (int k = 0; k < 2; k++)
    {
        ostream& out = *streams[k];
        printTable(out, T);
        out << fixed << setprecision(4);
        out << "Наибольшее: y = " << ymax << " при x = " << T[0][imax] << "\n";
        out << "Наименьшее: y = " << ymin << " при x = " << T[0][imin] << "\n";
    }
    fout.close();
    return 0;
}






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











    from src.students_grades.models.student import Student
from src.students_grades.models.student_group import StudentGroup
from src.students_grades.models.subject import Subject
from src.students_grades.models.teacher import Teacher
from src.students_grades.repositories import (
    grade_repository,
    student_group_repository,
    student_repository,
    subject_repository,
    teacher_repository,
    teacher_subject_repository,
    user_repository,
)


# ---------- Справочник групп ----------

def get_groups() -> list[StudentGroup]:
    return student_group_repository.get_all()


def create_group(name: str, course: int, faculty: str):
    if not name.strip():
        raise ValueError("Название группы не может быть пустым")
    if not isinstance(course, int) or not 1 <= course <= 6:
        raise ValueError("Курс должен быть числом от 1 до 6")
    if not faculty.strip():
        raise ValueError("Факультет не может быть пустым")
    for g in student_group_repository.get_all():
        if g.name.lower() == name.strip().lower():
            raise ValueError("Группа с таким названием уже существует")
    student_group_repository.create(name.strip(), course, faculty.strip())


def delete_group(group_id: int):
    if not student_group_repository.get_by_id(group_id):
        raise ValueError("Группа не найдена")
    if student_repository.get_by_group(group_id):
        raise ValueError("Нельзя удалить группу, в которой есть студенты")
    student_group_repository.delete(group_id)


# ---------- Справочник предметов ----------

def get_subjects() -> list[Subject]:
    return subject_repository.get_all()


def create_subject(name: str, hours: int, semester: int):
    if not name.strip():
        raise ValueError("Название предмета не может быть пустым")
    if not isinstance(hours, int) or hours <= 0:
        raise ValueError("Количество часов должно быть положительным числом")
    if not isinstance(semester, int) or not 1 <= semester <= 12:
        raise ValueError("Семестр должен быть числом от 1 до 12")
    for s in subject_repository.get_all():
        if s.name.lower() == name.strip().lower():
            raise ValueError("Предмет с таким названием уже существует")
    subject_repository.create(name.strip(), hours, semester)


def delete_subject(subject_id: int):
    if not subject_repository.get_by_id(subject_id):
        raise ValueError("Предмет не найден")
    for g in grade_repository.get_all():
        if g.subject_id == subject_id:
            raise ValueError("Нельзя удалить предмет, по которому есть оценки")
    for t in teacher_subject_repository.get_teachers_by_subject(subject_id):
        teacher_subject_repository.unassign(t.teacher_id, subject_id)
    subject_repository.delete(subject_id)


# ---------- Назначение преподавателей ----------

def get_teachers() -> list[Teacher]:
    return teacher_repository.get_all()


def get_teacher_subjects(teacher_id: int) -> list[Subject]:
    return teacher_subject_repository.get_subjects_by_teacher(teacher_id)


def assign_teacher(teacher_id: int, subject_id: int):
    if not teacher_repository.get_by_id(teacher_id):
        raise ValueError("Преподаватель не найден")
    if not subject_repository.get_by_id(subject_id):
        raise ValueError("Предмет не найден")
    if teacher_subject_repository.is_assigned(teacher_id, subject_id):
        raise ValueError("Преподаватель уже назначен на этот предмет")
    teacher_subject_repository.assign(teacher_id, subject_id)


def unassign_teacher(teacher_id: int, subject_id: int):
    if not teacher_subject_repository.is_assigned(teacher_id, subject_id):
        raise ValueError("Такого назначения нет")
    teacher_subject_repository.unassign(teacher_id, subject_id)


# ---------- Учётные записи ----------

def get_students() -> list[Student]:
    return student_repository.get_all()


def delete_student(student_id: int):
    student = student_repository.get_by_id(student_id)
    if not student:
        raise ValueError("Студент не найден")
    if grade_repository.get_by_student(student_id):
        raise ValueError("Нельзя удалить студента, у которого есть оценки")
    student_repository.delete(student_id)
    user_repository.delete(student.user_id)


def delete_teacher(teacher_id: int):
    teacher = teacher_repository.get_by_id(teacher_id)
    if not teacher:
        raise ValueError("Преподаватель не найден")
    for g in grade_repository.get_all():
        if g.teacher_id == teacher_id:
            raise ValueError(
                "Нельзя удалить преподавателя, который выставлял оценки"
            )
    for s in teacher_subject_repository.get_subjects_by_teacher(teacher_id):
        teacher_subject_repository.unassign(teacher_id, s.subject_id)
    teacher_repository.delete(teacher_id)
    user_repository.delete(teacher.user_id)
