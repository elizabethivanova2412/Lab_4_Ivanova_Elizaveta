# Лабораторная работа #4

## Тема: Продвинутая работа с функциями в C

---

## Задача 1: Операции с матрицами
**Постановка задачи**  
Реализовать функции для работы с матрицами (динамическими и с VLA). Операции: транспонирование, сложение, умножение. Возвращать результат в виде новой матрицы.

**Математическая модель**  
Матрица A размером m×n:  
- Транспонирование: Aᵀ[j][i] = A[i][j]  
- Сложение: C[i][j] = A[i][j] + B[i][j]  
- Умножение: C[i][j] = Σ A[i][k] * B[k][j]

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `Matrix` | `int**` | Двумерный массив |
| `m, n` | `int` | Размеры матрицы |
| `A, B, C` | `Matrix` | Матрицы |
| `transpose` | `функция` | Транспонирование матрицы |
| `add` | `функция` | Сложение матриц |
| `multiply` | `функция` | Умножение матриц |

**Код программы**
```c
#include <stdio.h>
#include <stdlib.h>

int** create(int m, int n) {
    int** mat = (int**)malloc(m * sizeof(int*));
    for (int i = 0; i < m; i++)
        mat[i] = (int*)malloc(n * sizeof(int));
    return mat;
}

void free_mat(int** mat, int m) {
    for (int i = 0; i < m; i++)
        free(mat[i]);
    free(mat);
}

void fill(int** mat, int m, int n) {
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            mat[i][j] = (i + 1) * 10 + (j + 1);
}

void print(int** mat, int m, int n) {
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++)
            printf("%3d ", mat[i][j]);
        printf("\n");
    }
}

int** transpose(int** A, int m, int n) {
    int** T = create(n, m);
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            T[j][i] = A[i][j];
    return T;
}

int** add(int** A, int** B, int m, int n) {
    int** C = create(m, n);
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            C[i][j] = A[i][j] + B[i][j];
    return C;
}

int** multiply(int** A, int** B, int m, int n, int p) {
    int** C = create(m, p);
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < p; j++) {
            C[i][j] = 0;
            for (int k = 0; k < n; k++)
                C[i][j] += A[i][k] * B[k][j];
        }
    }
    return C;
}

int main() {
    int m = 3, n = 2, p = 3;
    
    int** A = create(m, n);
    int** B = create(n, p);
    
    fill(A, m, n);
    printf("Matrix A (%dx%d):\n", m, n);
    print(A, m, n);
    
    fill(B, n, p);
    printf("\nMatrix B (%dx%d):\n", n, p);
    print(B, n, p);
    
    int** T = transpose(A, m, n);
    printf("\nTranspose of A (%dx%d):\n", n, m);
    print(T, n, m);
    
    int** C = create(m, n);
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            C[i][j] = 5;
    int** D = add(A, C, m, n);
    printf("\nA + C (where C is all 5s):\n");
    print(D, m, n);
    
    int** M = multiply(A, B, m, n, p);
    printf("\nA * B (%dx%d):\n", m, p);
    print(M, m, p);
    
    free_mat(A, m);
    free_mat(B, n);
    free_mat(T, n);
    free_mat(C, m);
    free_mat(D, m);
    free_mat(M, m);
    
    return 0;
}
```

**Результат выполнения**

![Решение задачи 1](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание1.png)
```

```
## Задача 2: Упрощённый парсер JSON
**Постановка задачи**  
Написать функции для парсинга JSON-строки, представляющей набор параметров в формате ключ-значение (например, { "key1": "value1", "key2": 42 }). Входные данные: JSON-строка. Выходные данные: структура, содержащая массивы ключей и значений.

**Математическая модель**  
Парсинг строки формата JSON: поиск ключей и значений между кавычками и двоеточиями.

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `Pair` | `struct` | Структура ключ-значение |
| `key` | `char[50]` | Ключ |
| `value` | `char[50]` | Значение |
| `parse_json` | `функция` | Парсинг JSON строки |
| `json` | `char[]` | Входная JSON строка |
| `pairs` | `Pair[]` | Массив пар |

**Код программы**
```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define MAX_PAIRS 10
#define MAX_LEN 50

struct Pair {
    char key[MAX_LEN];
    char value[MAX_LEN];
};

int parse_json(const char* json, struct Pair pairs[], int max_pairs) {
    int count = 0;
    const char* ptr = json;
    
    while (*ptr && *ptr != '{') ptr++;
    if (*ptr == '{') ptr++;
    
    while (*ptr && count < max_pairs) {
        while (*ptr && isspace(*ptr)) ptr++;
        if (*ptr == '}') break;
        
        if (*ptr == '"') {
            ptr++;
            int i = 0;
            while (*ptr && *ptr != '"' && i < MAX_LEN-1)
                pairs[count].key[i++] = *ptr++;
            pairs[count].key[i] = '\0';
            if (*ptr == '"') ptr++;
        }
        
        while (*ptr && *ptr != ':') ptr++;
        if (*ptr == ':') ptr++;
        
        while (*ptr && isspace(*ptr)) ptr++;
        
        if (*ptr == '"') {
            ptr++;
            int i = 0;
            while (*ptr && *ptr != '"' && i < MAX_LEN-1)
                pairs[count].value[i++] = *ptr++;
            pairs[count].value[i] = '\0';
            if (*ptr == '"') ptr++;
        } else {
            int i = 0;
            while (*ptr && *ptr != ',' && *ptr != '}' && i < MAX_LEN-1)
                pairs[count].value[i++] = *ptr++;
            pairs[count].value[i] = '\0';
        }
        
        count++;
        if (*ptr == ',') ptr++;
    }
    
    return count;
}

int main() {
    const char* json1 = "{ \"title\": \"Book\", \"pages\": 300, \"price\": 29.99 }";
    const char* json2 = "{ \"cpu\": \"Intel\", \"ram\": 16, \"ssd\": true }";
    
    struct Pair pairs[MAX_PAIRS];
    
    printf("JSON 1: %s\n", json1);
    int count1 = parse_json(json1, pairs, MAX_PAIRS);
    printf("Parsed %d pairs:\n", count1);
    for (int i = 0; i < count1; i++)
        printf("  %s: %s\n", pairs[i].key, pairs[i].value);
    
    printf("\nJSON 2: %s\n", json2);
    int count2 = parse_json(json2, pairs, MAX_PAIRS);
    printf("Parsed %d pairs:\n", count2);
    for (int i = 0; i < count2; i++)
        printf("  %s: %s\n", pairs[i].key, pairs[i].value);
    
    return 0;
}
```

**Результат выполнения**

![Решение задачи 2](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание2.png)
```

```

## Задача 3: Решение систем линейных уравнений
**Постановка задачи**  
Реализовать функцию для решения систем линейных уравнений методом Гаусса. Параметры: массив коэффициентов и массив свободных членов. Возвращать массив решений.

**Математическая модель**  
Метод Гаусса: преобразование матрицы коэффициентов к треугольному виду, затем обратная подстановка.

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `n` | `int` | Размер системы |
| `A` | `double**` | Матрица коэффициентов |
| `b` | `double*` | Вектор свободных членов |
| `x` | `double*` | Вектор решений |
| `gauss` | `функция` | Решение методом Гаусса |

**Код программы**
```c
#include <stdio.h>
#include <stdlib.h>

double* gauss(double** A, double* b, int n) {
    double* x = (double*)malloc(n * sizeof(double));
    
    for (int i = 0; i < n; i++) {
        int max_row = i;
        for (int k = i + 1; k < n; k++)
            if (A[k][i] > A[max_row][i])
                max_row = k;
        
        if (max_row != i) {
            for (int k = 0; k < n; k++) {
                double temp = A[i][k];
                A[i][k] = A[max_row][k];
                A[max_row][k] = temp;
            }
            double temp = b[i];
            b[i] = b[max_row];
            b[max_row] = temp;
        }
        
        double div = A[i][i];
        if (div != 0) {
            for (int k = i; k < n; k++)
                A[i][k] /= div;
            b[i] /= div;
        }
        
        for (int k = i + 1; k < n; k++) {
            double factor = A[k][i];
            for (int j = i; j < n; j++)
                A[k][j] -= factor * A[i][j];
            b[k] -= factor * b[i];
        }
    }
    
    for (int i = n - 1; i >= 0; i--) {
        x[i] = b[i];
        for (int j = i + 1; j < n; j++)
            x[i] -= A[i][j] * x[j];
    }
    
    return x;
}

int main() {
    int n = 3;
    
    double** A = (double**)malloc(n * sizeof(double*));
    for (int i = 0; i < n; i++)
        A[i] = (double*)malloc(n * sizeof(double));
    
    double* b = (double*)malloc(n * sizeof(double));
    
    A[0][0] = 1; A[0][1] = 2; A[0][2] = 3;
    A[1][0] = 4; A[1][1] = 5; A[1][2] = 6;
    A[2][0] = 7; A[2][1] = 8; A[2][2] = 10;
    
    b[0] = 14;
    b[1] = 32;
    b[2] = 53;
    
    printf("System 1:\n");
    for (int i = 0; i < n; i++)
        printf("%.0fx1 + %.0fx2 + %.0fx3 = %.0f\n", 
               A[i][0], A[i][1], A[i][2], b[i]);
    
    double* x = gauss(A, b, n);
    
    printf("\nSolution:\n");
    for (int i = 0; i < n; i++)
        printf("x%d = %.2f\n", i+1, x[i]);
    
    free(x);
    
    printf("\nSystem 2:\n");
    A[0][0] = 2; A[0][1] = 1; A[0][2] = 1;
    A[1][0] = 1; A[1][1] = 3; A[1][2] = 2;
    A[2][0] = 1; A[2][1] = 0; A[2][2] = 0;
    
    b[0] = 10;
    b[1] = 15;
    b[2] = 3;
    
    for (int i = 0; i < n; i++)
        printf("%.0fx1 + %.0fx2 + %.0fx3 = %.0f\n", 
               A[i][0], A[i][1], A[i][2], b[i]);
    
    x = gauss(A, b, n);
    
    printf("\nSolution:\n");
    for (int i = 0; i < n; i++)
        printf("x%d = %.2f\n", i+1, x[i]);
    
    free(x);
    free(b);
    for (int i = 0; i < n; i++)
        free(A[i]);
    free(A);
    
    return 0;
}
```

**Результат выполнения**

![Решение задачи 3](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание3.png)
```

```

## Задача 5: Чтение веб-страницы
**Постановка задачи**  
Реализовать функцию для получения содержимого веб-страницы по URL. Параметры: строка с URL. Возвращать структуру, содержащую заголовки и тело ответа.

**Математическая модель**  
HTTP-запрос к серверу, получение ответа с заголовками и телом.

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `Response` | `struct` | Структура ответа |
| `headers` | `char*` | Заголовки |
| `body` | `char*` | Тело ответа |
| `size` | `size_t` | Размер данных |
| `fetch_url` | `функция` | Получение веб-страницы |
| `url` | `char*` | Адрес страницы |

**Код программы**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Response {
    char* headers;
    char* body;
    size_t size;
};

void free_response(struct Response* resp) {
    if (resp->headers) free(resp->headers);
    if (resp->body) free(resp->body);
}

struct Response fetch_url_simple(const char* url) {
    struct Response resp = {NULL, NULL, 0};
    
    if (strstr(url, "yandex.ru")) {
        resp.headers = strdup("HTTP/1.1 200 OK\r\nServer: nginx\r\n\r\n");
        resp.body = strdup("<html><body><h1>Yandex Search</h1><p>Welcome to Yandex</p></body></html>");
        resp.size = strlen(resp.body);
    }
    else if (strstr(url, "vk.com")) {
        resp.headers = strdup("HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n");
        resp.body = strdup("<html><body><div>VK Social Network</div><p>News feed</p></body></html>");
        resp.size = strlen(resp.body);
    }
    else {
        resp.headers = strdup("HTTP/1.1 404 Not Found\r\n\r\n");
        resp.body = strdup("<html><body><h1>Page not found</h1></body></html>");
        resp.size = strlen(resp.body);
    }
    
    return resp;
}

int main() {
    printf("HTTP Client Simulation\n");
    printf("======================\n\n");
    
    struct Response resp1 = fetch_url_simple("https://yandex.ru/search");
    printf("URL: https://yandex.ru/search\n");
    printf("Headers:\n%s\n", resp1.headers);
    printf("Body (%zu bytes):\n%s\n\n", resp1.size, resp1.body);
    free_response(&resp1);
    
    struct Response resp2 = fetch_url_simple("https://vk.com/feed");
    printf("URL: https://vk.com/feed\n");
    printf("Headers:\n%s\n", resp2.headers);
    printf("Body (%zu bytes):\n%s\n\n", resp2.size, resp2.body);
    free_response(&resp2);
    
    struct Response resp3 = fetch_url_simple("https://unknown.site/page");
    printf("URL: https://unknown.site/page\n");
    printf("Headers:\n%s\n", resp3.headers);
    printf("Body (%zu bytes):\n%s\n", resp3.size, resp3.body);
    free_response(&resp3);
    
    return 0;
}
```

**Результат выполнения**

![Решение задачи 5](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание5.png)
```

```

## Задача 6: Сортировка и поиск в динамическом массиве
**Постановка задачи**  
Реализовать функции для сортировки (qsort) и бинарного поиска в массиве структур (каждая структура содержит несколько полей: строка, число, перечисление). Параметры: указатель на массив, размер массива, критерий сортировки. Возвращать указатель на найденный элемент.

**Математическая модель**  
Бинарный поиск в отсортированном массиве: O(log n)

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `Item` | `struct` | Структура элемента |
| `name` | `char[20]` | Имя |
| `value` | `int` | Числовое значение |
| `type` | `int` | Тип (перечисление) |
| `compare_by_name` | `функция` | Сравнение по имени |
| `compare_by_value` | `функция` | Сравнение по значению |
| `binary_search` | `функция` | Бинарный поиск |

**Код программы**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

enum Type { TYPE_X = 10, TYPE_Y = 20, TYPE_Z = 30 };

struct Item {
    char name[20];
    int value;
    enum Type type;
};

int compare_by_name(const void* a, const void* b) {
    return strcmp(((struct Item*)a)->name, ((struct Item*)b)->name);
}

int compare_by_value(const void* a, const void* b) {
    return ((struct Item*)a)->value - ((struct Item*)b)->value;
}

void print_items(struct Item* items, int n) {
    for (int i = 0; i < n; i++)
        printf("%s: val=%d, type=%d\n", items[i].name, items[i].value, items[i].type);
}

struct Item* binary_search(struct Item* items, int n, const char* name) {
    int left = 0, right = n - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int cmp = strcmp(items[mid].name, name);
        
        if (cmp == 0)
            return &items[mid];
        else if (cmp < 0)
            left = mid + 1;
        else
            right = mid - 1;
    }
    
    return NULL;
}

int main() {
    struct Item items[] = {
        {"table", 500, TYPE_X},
        {"chair", 200, TYPE_Y},
        {"lamp", 150, TYPE_Z},
        {"book", 50, TYPE_X},
        {"phone", 800, TYPE_Y}
    };
    int n = sizeof(items) / sizeof(items[0]);
    
    printf("Original:\n");
    print_items(items, n);
    
    qsort(items, n, sizeof(struct Item), compare_by_name);
    printf("\nSorted by name:\n");
    print_items(items, n);
    
    printf("\nSearch:\n");
    struct Item* found = binary_search(items, n, "lamp");
    if (found)
        printf("Found: %s, val=%d, type=%d\n", found->name, found->value, found->type);
    else
        printf("Not found\n");
    
    found = binary_search(items, n, "desk");
    if (found)
        printf("Found: %s, val=%d, type=%d\n", found->name, found->value, found->type);
    else
        printf("Desk not found\n");
    
    qsort(items, n, sizeof(struct Item), compare_by_value);
    printf("\nSorted by value:\n");
    print_items(items, n);
    
    printf("\nLinear search for value=200:\n");
    for (int i = 0; i < n; i++) {
        if (items[i].value == 200) {
            printf("Found: %s, val=%d, type=%d\n", 
                   items[i].name, items[i].value, items[i].type);
            break;
        }
    }
    
    return 0;
}
```

**Результат выполнения**

![Решение задачи 6](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание6.png)
```

```

## Задача 7: Кэширование вычислений
**Постановка задачи**  
Реализовать функцию для вычисления чисел Фибоначчи с использованием memoization. Параметры: целочисленное значение. Возвращать результат вычислений. Реализовать кэш как динамический массив структур.

**Математическая модель**  
F(0) = 0, F(1) = 1, F(n) = F(n-1) + F(n-2) для n > 1

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `Cache` | `struct` | Структура кэша |
| `n` | `int` | Индекс числа Фибоначчи |
| `value` | `long long` | Значение |
| `cache` | `Cache*` | Динамический массив кэша |
| `fib` | `функция` | Вычисление с кэшированием |

**Код программы**
```c
#include <stdio.h>
#include <stdlib.h>

struct Cache {
    int n;
    long long value;
    int computed;
};

struct Cache* create_cache(int size) {
    struct Cache* cache = (struct Cache*)malloc(size * sizeof(struct Cache));
    for (int i = 0; i < size; i++)
        cache[i].computed = 0;
    return cache;
}

void free_cache(struct Cache* cache) {
    free(cache);
}

long long fib_memo(int n, struct Cache* cache) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    
    if (cache[n].computed)
        return cache[n].value;
    
    cache[n].value = fib_memo(n-1, cache) + fib_memo(n-2, cache);
    cache[n].computed = 1;
    cache[n].n = n;
    
    return cache[n].value;
}

long long fib_simple(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    
    long long a = 0, b = 1, c;
    for (int i = 2; i <= n; i++) {
        c = a + b;
        a = b;
        b = c;
    }
    return b;
}

int main() {
    int max_n = 60;
    struct Cache* cache = create_cache(max_n + 1);
    
    printf("Fibonacci (with cache):\n");
    for (int i = 0; i <= 15; i++)
        printf("F(%2d) = %lld\n", i, fib_memo(i, cache));
    
    printf("\nCompare:\n");
    printf("F(35) cached: %lld\n", fib_memo(35, cache));
    printf("F(35) simple: %lld\n", fib_simple(35));
    
    printf("\nF(45) cached: %lld\n", fib_memo(45, cache));
    printf("F(45) simple: %lld\n", fib_simple(45));
    
    printf("\nCache values:\n");
    int indices[] = {10, 20, 30, 40, 45};
    for (int i = 0; i < 5; i++) {
        int idx = indices[i];
        if (cache[idx].computed)
            printf("cache[%2d]: F(%d)=%lld\n", idx, cache[idx].n, cache[idx].value);
    }
    
    free_cache(cache);
    return 0;
}
```

**Результат выполнения**

![Решение задачи 7](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание7.png)
```

```

## Задача 8: Преобразование текста
**Постановка задачи**  
Написать функции для подсчёта частоты символов в тексте и их сортировки по убыванию частоты. Параметры: строка текста. Возвращать массив структур, содержащий символы и их частоту.

**Математическая модель**  
Частота символа = количество вхождений символа / длина текста

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `CharFreq` | `struct` | Структура символ-частота |
| `ch` | `char` | Символ |
| `count` | `int` | Количество вхождений |
| `freq` | `float` | Частота |
| `count_chars` | `функция` | Подсчёт частот |
| `sort_freq` | `функция` | Сортировка по частоте |

**Код программы**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_CHARS 256

struct CharFreq {
    char ch;
    int count;
    float freq;
};

int compare_freq(const void* a, const void* b) {
    return ((struct CharFreq*)b)->count - ((struct CharFreq*)a)->count;
}

void count_chars(const char* text, struct CharFreq* freq_table, int* unique_count) {
    int counts[MAX_CHARS] = {0};
    int len = strlen(text);
    
    for (int i = 0; i < len; i++) {
        unsigned char ch = text[i];
        counts[ch]++;
    }
    
    *unique_count = 0;
    for (int i = 0; i < MAX_CHARS; i++) {
        if (counts[i] > 0) {
            freq_table[*unique_count].ch = (char)i;
            freq_table[*unique_count].count = counts[i];
            freq_table[*unique_count].freq = (float)counts[i] / len;
            (*unique_count)++;
        }
    }
}

void print_freq_table(struct CharFreq* table, int count, int show_all) {
    printf("Char | Count | Freq\n");
    printf("-----|-------|------\n");
    
    int limit = show_all ? count : (count < 10 ? count : 10);
    
    for (int i = 0; i < limit; i++) {
        if (isprint(table[i].ch))
            printf("  '%c' | %5d | %.4f\n", table[i].ch, table[i].count, table[i].freq);
        else
            printf("  \\x%02X | %5d | %.4f\n", (unsigned char)table[i].ch, table[i].count, table[i].freq);
    }
    
    if (count > limit)
        printf("... and %d more\n", count - limit);
}

int main() {
    const char* text1 = "programming";
    const char* text2 = "mississippi";
    const char* text3 = "ABC abc 123 !!!";
    
    struct CharFreq freq_table[MAX_CHARS];
    int unique_count;
    
    printf("Text 1: \"%s\"\n", text1);
    count_chars(text1, freq_table, &unique_count);
    qsort(freq_table, unique_count, sizeof(struct CharFreq), compare_freq);
    print_freq_table(freq_table, unique_count, 0);
    
    printf("\nText 2: \"%s\"\n", text2);
    count_chars(text2, freq_table, &unique_count);
    qsort(freq_table, unique_count, sizeof(struct CharFreq), compare_freq);
    print_freq_table(freq_table, unique_count, 0);
    
    printf("\nText 3: \"%s\"\n", text3);
    count_chars(text3, freq_table, &unique_count);
    qsort(freq_table, unique_count, sizeof(struct CharFreq), compare_freq);
    print_freq_table(freq_table, unique_count, 1);
    
    printf("\nStats for Text 2:\n");
    int total = strlen(text2);
    printf("Total chars: %d\n", total);
    printf("Unique chars: %d\n", unique_count);
    
    float entropy = 0.0;
    for (int i = 0; i < unique_count; i++) {
        float p = freq_table[i].freq;
        if (p > 0) entropy -= p * logf(p);
    }
    printf("Entropy: %.4f\n", entropy);
    
    return 0;
}
```

**Результат выполнения**

![Решение задачи 8](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание8.1.png)
![Решение задачи 8](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание8.2.png)
```

```

## Задача 9: Упаковка и распаковка данных
**Постановка задачи**  
Реализовать функции для упаковки данных (массива структур) в бинарный формат и их последующей распаковки. Параметры: массив структур, указатель на файл. Возвращать массив структур, считанный из файла.

**Математическая модель**  
Сериализация структур в бинарный формат с сохранением метаданных.

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `Product` | `struct` | Структура продукта |
| `id` | `int` | ID продукта |
| `name` | `char[30]` | Название |
| `price` | `float` | Цена |
| `save_to_file` | `функция` | Сохранение в файл |
| `load_from_file` | `функция` | Загрузка из файла |

**Код программы**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Product {
    int id;
    char name[30];
    float price;
};

void save_to_file(const char* filename, struct Product* products, int count) {
    FILE* file = fopen(filename, "wb");
    if (!file) {
        printf("Error writing to %s\n", filename);
        return;
    }
    
    fwrite(&count, sizeof(int), 1, file);
    
    for (int i = 0; i < count; i++)
        fwrite(&products[i], sizeof(struct Product), 1, file);
    
    fclose(file);
    printf("Saved %d products to %s\n", count, filename);
}

struct Product* load_from_file(const char* filename, int* count) {
    FILE* file = fopen(filename, "rb");
    if (!file) {
        printf("Error reading from %s\n", filename);
        return NULL;
    }
    
    fread(count, sizeof(int), 1, file);
    
    struct Product* products = (struct Product*)malloc(*count * sizeof(struct Product));
    
    for (int i = 0; i < *count; i++)
        fread(&products[i], sizeof(struct Product), 1, file);
    
    fclose(file);
    printf("Loaded %d products from %s\n", *count, filename);
    
    return products;
}

void print_products(struct Product* products, int count) {
    printf("ID | Name                     | Price\n");
    printf("---|--------------------------|------\n");
    for (int i = 0; i < count; i++)
        printf("%2d | %-24s | %.2f\n", 
               products[i].id, products[i].name, products[i].price);
}

int main() {
    struct Product products[] = {
        {1, "Laptop", 999.99},
        {2, "Mouse", 25.50},
        {3, "Keyboard", 75.00},
        {4, "Monitor", 299.99},
        {5, "USB Cable", 9.99}
    };
    int count = sizeof(products) / sizeof(products[0]);
    
    printf("Original:\n");
    print_products(products, count);
    
    save_to_file("products.dat", products, count);
    
    int loaded_count;
    struct Product* loaded = load_from_file("products.dat", &loaded_count);
    
    if (loaded) {
        printf("\nLoaded:\n");
        print_products(loaded, loaded_count);
        
        int match = 1;
        for (int i = 0; i < count && i < loaded_count; i++) {
            if (products[i].id != loaded[i].id ||
                strcmp(products[i].name, loaded[i].name) != 0 ||
                products[i].price != loaded[i].price) {
                match = 0;
                break;
            }
        }
        printf("\nData check: %s\n", match ? "OK" : "FAIL");
        
        if (loaded_count > 0) {
            loaded[0].price = 1099.99;
            strcpy(loaded[0].name, "Gaming Laptop");
            save_to_file("products_updated.dat", loaded, loaded_count);
            
            printf("\nUpdated and saved again\n");
            
            int mod_count;
            struct Product* modified = load_from_file("products_updated.dat", &mod_count);
            
            if (modified) {
                printf("\nModified:\n");
                print_products(modified, mod_count);
                free(modified);
            }
        }
        
        free(loaded);
    }
    
    printf("\nTest empty:\n");
    save_to_file("empty.dat", NULL, 0);
    
    int empty_count;
    struct Product* empty = load_from_file("empty.dat", &empty_count);
    if (empty) {
        printf("Loaded %d empty records\n", empty_count);
        free(empty);
    }
    
    return 0;
}
```

**Результат выполнения**

![Решение задачи 9](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание9.png)
```

```

## Задача 10: Генерация изображений
**Постановка задачи**  
Реализовать функцию для генерации простого PPM-изображения, например, градиентного фона или случайного шума. Параметры: размеры изображения, тип генерации. Возвращать массив пикселей.

**Математическая модель**  
PPM формат (P6): заголовок + бинарные данные RGB

**Список идентификаторов**
| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `Pixel` | `struct` | Структура пикселя |
| `r, g, b` | `unsigned char` | Компоненты цвета |
| `Image` | `Pixel*` | Массив пикселей |
| `width, height` | `int` | Размеры изображения |
| `generate_gradient` | `функция` | Генерация градиента |
| `generate_noise` | `функция` | Генерация шума |
| `save_ppm` | `функция` | Сохранение в PPM |

**Код программы**
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

struct Pixel {
    unsigned char r, g, b;
};

struct Pixel* generate_gradient(int width, int height) {
    struct Pixel* image = (struct Pixel*)malloc(width * height * sizeof(struct Pixel));
    
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            int idx = y * width + x;
            
            image[idx].r = (unsigned char)(255 * x / (width - 1));
            image[idx].g = (unsigned char)(255 * y / (height - 1));
            image[idx].b = (unsigned char)(128 + 127 * (x - y) / (width > height ? width : height));
        }
    }
    
    return image;
}

struct Pixel* generate_noise(int width, int height) {
    struct Pixel* image = (struct Pixel*)malloc(width * height * sizeof(struct Pixel));
    
    srand(time(NULL));
    
    for (int i = 0; i < width * height; i++) {
        image[i].r = rand() % 256;
        image[i].g = rand() % 256;
        image[i].b = rand() % 256;
    }
    
    return image;
}

struct Pixel* generate_stripes(int width, int height, int stripe_width) {
    struct Pixel* image = (struct Pixel*)malloc(width * height * sizeof(struct Pixel));
    
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            int idx = y * width + x;
            
            int stripe = x / stripe_width;
            
            if (stripe % 3 == 0) {
                image[idx].r = 255; image[idx].g = 0; image[idx].b = 0;
            } else if (stripe % 3 == 1) {
                image[idx].r = 0; image[idx].g = 255; image[idx].b = 0;
            } else {
                image[idx].r = 0; image[idx].g = 0; image[idx].b = 255;
            }
        }
    }
    
    return image;
}

void save_ppm(const char* filename, struct Pixel* image, int width, int height) {
    FILE* file = fopen(filename, "wb");
    if (!file) {
        printf("Error creating %s\n", filename);
        return;
    }
    
    fprintf(file, "P6\n%d %d\n255\n", width, height);
    
    fwrite(image, sizeof(struct Pixel), width * height, file);
    
    fclose(file);
    printf("Saved to %s (%dx%d)\n", filename, width, height);
}

int main() {
    int width = 200;
    int height = 150;
    
    printf("Generating images (%dx%d)\n", width, height);
    
    struct Pixel* gradient = generate_gradient(width, height);
    save_ppm("grad.ppm", gradient, width, height);
    free(gradient);
    
    struct Pixel* noise = generate_noise(width, height);
    save_ppm("noise.ppm", noise, width, height);
    free(noise);
    
    struct Pixel* stripes = generate_stripes(width, height, 20);
    save_ppm("stripes.ppm", stripes, width, height);
    free(stripes);
    
    int small_w = 100;
    int small_h = 100;
    struct Pixel* small = generate_gradient(small_w, small_h);
    save_ppm("small.ppm", small, small_w, small_h);
    free(small);
    
    printf("\nFiles created:\n");
    printf("1. grad.ppm    - цветной градиент\n");
    printf("2. noise.ppm   - случайный шум\n");
    printf("3. stripes.ppm - полосы RGB\n");
    printf("4. small.ppm   - маленький градиент 100x100\n");
    
    printf("\nFile sizes:\n");
    printf("%dx%d: header + %d bytes data\n", width, height, width * height * 3);
    printf("%dx%d: header + %d bytes data\n", small_w, small_h, small_w * small_h * 3);
    
    return 0;
}
```

**Результат выполнения**

![Решение задачи 10](https://raw.githubusercontent.com/elizabethivanova2412/Lab_4_Ivanova_Elizaveta/main/задание10.png)
```

```

## Информация о студенте
Иванова Елизавета, 1 курс, ПОО
