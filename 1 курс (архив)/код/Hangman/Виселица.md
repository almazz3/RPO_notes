#Виселица #Python #code #программирование
```python
#Импорт библиотек в самом начале кода
#случайные величины, время, система
import random, time, os
import read_words as tws

class Hangman:

    #Конструктор класса
    def __init__(self, words_param):
        self.words = words_param
        self.mainloop()

    def _new_game(self):
        self.attempts = 12
        self.selected_word = random.choice(self.words).lower()
        self.unguessed_string = "_ " * len(self.selected_word)
        self.guessed_string = self.unguessed_string 

    def mainloop(self):

        os.system("cls")
        is_running = True
        while is_running == True:
            self._new_game()
            print("Добро пожаловать в игру Висельник!")
            print("Выйти из игры можно только после её окончания.")
            user_input = input("Enter для продолжения или \"exit\" для выхода. \n")
            if user_input == "exit":
                is_running = False
                break

            while self.attempts != 0:

                if self.guessed_string.count("_ ") == 0:
                    print("Победа! Загаданное слово - ", self.selected_word)
                    break

                os.system("cls")
                print("Осталось попыток:", self.attempts)
                print(self.guessed_string)
                letter = input("Введите букву: ")

                if self.selected_word.count(letter) != 0:

                    self.unguessed_string = ""
                    for i in range(len(self.selected_word)):
                        if self.selected_word[i] == letter or self.guessed_string[i] != "_":
                            self.unguessed_string += self.selected_word[i]
                        else:
                            self.unguessed_string += "_"
                    self.guessed_string = self.unguessed_string

                else:
                    self.attempts -= 1

    def test(self):
        os.system("cls")
        print(self.words)
        print(self.attempts)
        print(self.selected_word)
        print(self.guessed_string)

words_list = tws.read_words("words.txt")
s = Hangman(words_list)
s.test()
s.mainloop()
```

Пример игры **"Виселица"** **(Hangman)**, реализованный на Python в объектно-ориентированном стиле. Код использует два файла: `read_words.py` для загрузки слов и основной файл с классом `Hangman`. #Hangman #ООП

# `read_words.py` (Модуль для чтения слов)

```python
def read_words(filename: str):
    words_out = []
    with open(filename, "r", encoding='utf-8') as myfile:
        words_raw = myfile.readlines()

        for word in words_raw:
            words_out.append(word[:-1])

        myfile.close()
    return words_out
```

Этот модуль предназначен для чтения списка слов из текстового файла.

## Функция `read_words(filename: str)`
 * `words_out = []`: Создается пустой список, куда будут добавлены слова.
 * `with open(filename, "r", encoding='utf-8') as myfile:`: Открывается файл с именем *filename* в режиме чтения `"r"` с кодировкой *UTF-8*. Использование конструкции `with open(...)` гарантирует, что файл будет автоматически закрыт, даже если произойдет ошибка. 
 * `words_raw = myfile.readlines()`: Считываются все строки из файла и сохраняются в список `words_raw`. Каждая строка в этом списке будет содержать символ перевода строки `\n` в конце.
 * `for word in words_raw:`: Перебор всех считанных строк.
 * `words_out.append(word[:-1])`: Каждая строка добавляется в итоговый список `words_out`, но при этом используется срез `[:-1]`, который удаляет последний символ (который, предположительно, является символом новой строки `\n`.
 * `myfile.close()`: Эта строка избыточна, так как `with open(...)` уже закрывает файл, но не является ошибкой.
 * `return words_out`: Функция возвращает очищенный список слов.
 
# Разбор Основного Кода (Класс Hangman)
Основная логика игры содержится в классе Hangman.

## 1. Импорты

```python
import random, time, os
import read_words as tws
```
 
 * `random`: Используется для выбора случайного слова `random.choice`.
 * `time`: Импортирован, но не используется в предоставленном коде.
 * `os`: Используется для очистки консоли `os.system("cls")`.
 * `read_words as tws`: Импортирует модуль для чтения слов и присваивает ему короткий псевдоним `tws`.
 
## 2. Класс Hangman

### 2.1. Конструктор (init)

```python
def init(self, words_param):
    self.words = words_param
    self.mainloop()
```

 * При создании объекта `Hangman` (например, `s = Hangman(words_list)`), конструктор принимает список слов `words_param`.
 * Он сохраняет этот список в атрибуте экземпляра `self.words`.
 * Сразу после инициализации он запускает основной цикл игры вызовом `self.mainloop()`.

### 2.2. Начать новую игру `new_game`

```Python
def _new_game(self):
    self.attempts = 12
    self.selected_word = random.choice(self.words).lower()
    self.unguessed_string = "_ " * len(self.selected_word)
    self.guessed_string = self.unguessed_string 
```

Эта служебная функция (начинается с `_` для обозначения внутреннего использования) сбрасывает состояние игры:
 * `self.attempts = 12`: Устанавливает начальное количество попыток (12).
 * `self.selected_word = random.choice(self.words).lower()`: Случайным образом выбирает слово из списка и переводит его в нижний регистр.
 * `self.unguessed_string = "_ " * len(self.selected_word)`: Создает строку, отображающую прогресс угадывания, например, для слова из 5 букв будет `_ _ _ _ _`.
 * `self.guessed_string = self.unguessed_string`: Инициализирует строку, которую видит пользователь.

### 2.3. Основной цикл игры (mainloop)

```python
def mainloop(self):
    os.system("cls")
    is_running = True
    while is_running == True:
        # ... Логика начала игры и основной цикл угадывания ...
```

Это главный цикл, который позволяет играть в игру несколько раз подряд.

#### Цикл выбора "Играть/Выход"

```python
    while is_running == True:
        self._new_game()
        print("Добро пожаловать в игру Висельник!")
        # ...
        user_input = input("Enter для продолжения или \"exit\" для выхода. \n")
        if user_input == "exit":
            is_running = False
            break
```

 * Очищает экран (`os.system("cls")`).
 * В начале каждой итерации вызывает `self._new_game()`, чтобы начать новую игру.
 * Спрашивает пользователя, хочет ли он начать или выйти (`"exit"`). Если пользователь вводит `"exit"`, внешний цикл прерывается `break`. 
 
#### Цикл угадывания слова (внутренний цикл)

```python
        while self.attempts != 0:
            if self.guessed_string.count("_ ") == 0:
                print("Победа! Загаданное слово - ", self.selected_word)
                break
            
            os.system("cls")
            print("Осталось попыток:", self.attempts)
            print(self.guessed_string)
            letter = input("Введите букву: ")
            
            # ... Логика проверки буквы ...
```

Этот цикл продолжается до тех пор, пока:
 * Победа: Строка `self.guessed_string` не содержит больше `_` (т.е. слово полностью угадано). В этом случае выводится сообщение о победе и цикл прерывается.
 * Поражение: Количество попыток `self.attempts` становится равно 0 (происходит в конце цикла).
 
### 2.4. Логика проверки буквы

Проверка происходит внутри внутреннего цикла:

```python
            if self.selected_word.count(letter) != 0:
                # ... Буква найдена ...
            else:
                self.attempts -= 1
```

* Если буква есть в слове `self.selected_word.count(letter) != 0`:
	* Создается новая строка `self.unguessed_string`.
	* Происходит итерация по всем индексам загаданного слова `for i in range(len(self.selected_word))`.
	* Проверяется условие:
	     * `if self.selected_word[i] == letter`: Текущая буква угадана в этом месте, ИЛИ
	     * `self.guessed_string[i] != "_"`: Эта позиция уже была угадана ранее.
	* Если условие истинно, в новую строку добавляется реальная буква. Иначе добавляется `_`.
	* После цикла `self.guessed_string` обновляется новой, более полной строкой.
	* Если буква не найдена `else`:
		* Количество попыток уменьшается: `self.attempts -= 1`.

### 2.5. Функция test

```python
def test(self):
    os.system("cls")
    print(self.words)
    print(self.attempts)
    print(self.selected_word)
    print(self.guessed_string)
```

Эта функция была добавлена для отладки. Она выводит текущее состояние игры (список слов, попытки, выбранное слово и строку угадывания). В финальном коде она вызывается сразу после инициализации, но перед запуском `s.mainloop()`, что позволяет разработчику увидеть начальные параметры.

### 3. Запуск Кода

```python
words_list = tws.read_words("words.txt")
s = Hangman(words_list)
s.test()
s.mainloop()
```

 * `words_list = tws.read_words("words.txt")`: Считывается список слов из файла *words.txt*.
 * `s = Hangman(words_list)`: Создается экземпляр класса `Hangman`, и конструктор автоматически запускает `s.mainloop()`.
 * `s.test()`: Вызывается функция отладки.
 * `s.mainloop()`: В этом месте код фактически попытается запустить `mainloop()` во второй раз, но поскольку `s.mainloop()` уже был вызван в конструкторе, эта строка избыточна, а в некоторых случаях может привести к неожиданному поведению, если конструктор не завершил свою работу. Обычно, конструктор не должен вызывать `mainloop`.
 
# Общая структура игры

* Инициализация: Загрузка слов, создание объекта `Hangman`.
* Внешний цикл: Спрашивает "Играть снова?"
* Начало игры: Выбор случайного слова, сброс попыток до 12.
* Внутренний цикл: Цикл угадывания. Пока попытки > 0 и слово не угадано:
* Отображение состояния (попытки, угаданная строка).
* Ввод буквы пользователем.
* Проверка:
	* Если буква есть: Обновление угаданной строки.
    * Если буквы нет: попытки - 1.
* Проверка окончания:
	* Если слово угадано: Победа! (Выход из внутреннего цикла)
	* Если попытки == 0: Поражение! (Внутренний цикл завершается).
* Конец игры: Возврат к Внешнему циклу ("Играть снова?").