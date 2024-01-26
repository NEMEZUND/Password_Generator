import PySimpleGUI as sg  # Импорт библиотеки PySimpleGUI для создания графического интерфейса
import itertools  # Импорт модуля itertools для работы с итераторами и комбинаторикой
import string  # Импорт модуля string для работы со строками
import threading  # Импорт модуля threading для создания потоков

def generate_passwords(params, file_path, status_text, progress_bar, stop_event):
    # Сбор символов в зависимости от выбранных параметров
    symbols = ''
    if params['digits']:
        symbols += string.digits
    if params['special_chars']:
        symbols += '@#$%^&*<>_=+-'
    if params['latin_letters']:
        symbols += string.ascii_lowercase
    if params['russian_letters']:
        symbols += 'абвгдеёжзийклмнопрстуфхцчшщъыьэюя'
    if params['uppercase_letters']:
        symbols += symbols.upper()

    # Определение длины пароля, по умолчанию 2, если длина не указана или не является числом
    length = int(params['length']) if params['length'] and params['length'].isdigit() else 2
    total_combinations = len(symbols) ** length  # Вычисление общего числа возможных комбинаций

    # Открытие файла для записи паролей
    with open(file_path, 'w', encoding='utf-8') as outfile:
        combinations = itertools.product(symbols, repeat=length)  # Получение всех комбинаций символов заданной длины
        for i, combination in enumerate(combinations):
            if stop_event.is_set():
                break  # Прерывание генерации, если установлено событие stop_event
            password = ''.join(combination)
            outfile.write(password + '\n')  # Запись пароля в файл
            progress_value = int((i + 1) / total_combinations * 100)
            progress_bar.update_bar(progress_value)  # Обновление значения прогресс-бара

    status_text.update(f'Количество возможных комбинаций: {total_combinations}')  # Обновление информации о количестве возможных комбинаций
    progress_bar.update_bar(100)  # Установка прогресс-бара на 100%

def update_total_combinations(params, status_text):
    # Обновление информации о количестве возможных комбинаций в зависимости от выбранных параметров
    if params['length'] and params['length'].isdigit():
        total_combinations = len(get_combinations(params)) ** int(params['length'])
        status_text.update(f'Количество возможных комбинаций: {total_combinations}')
    else:
        status_text.update(f'Количество возможных комбинаций: 0')

def main():
    stop_event = threading.Event()  # Создание объекта события для остановки генерации

    # Определение макета графического интерфейса (GUI)
    layout = [
        [sg.Checkbox('Цифры', key='digits', enable_events=True), sg.Checkbox('Спецсимволы', key='special_chars', enable_events=True)],
        [sg.Checkbox('Латинские буквы', key='latin_letters', enable_events=True), sg.Checkbox('Русские буквы', key='russian_letters', enable_events=True)],
        [sg.Checkbox('Заглавные буквы', key='uppercase_letters', enable_events=True)],
        [sg.Text('Длина пароля:'), sg.InputText(key='length', enable_events=True)],
        [sg.Button('Сгенерировать'), sg.Button('Прервать')],
        [sg.Button('Выход')],
        [sg.Text('Количество возможных комбинаций: 0', key='status_text')],
        [sg.ProgressBar(100, orientation='h', size=(20, 20), key='progress_bar')]
    ]

    window = sg.Window('Генератор паролей', layout)  # Создание окна GUI

    while True:
        event, values = window.read()  # Ожидание событий в окне

        # Обработка событий закрытия окна и кнопки "Выход"
        if event == sg.WIN_CLOSED or event == 'Выход':
            stop_event.set()  # Установка события stop_event перед выходом
            break

        # Обработка события кнопки "Прервать"
        if event == 'Прервать':
            stop_event.set()  # Установка события stop_event для прерывания генерации

        # Обработка события кнопки "Сгенерировать"
        if event == 'Сгенерировать':
            params = {key: values[key] for key in ['digits', 'special_chars', 'latin_letters', 'russian_letters', 'uppercase_letters', 'length']}
            file_path_txt = sg.popup_get_file('Выберите место для сохранения файла (txt)', save_as=True, default_extension=".txt")

            if file_path_txt:
                progress_bar = window['progress_bar']
                progress_bar.update_bar(0)
                stop_event.clear()  # Сброс события stop_event перед началом генерации

                # Использование threading для выполнения генерации в отдельном потоке
                thread = threading.Thread(target=generate_passwords, args=(params, file_path_txt, window['status_text'], progress_bar, stop_event))
                thread.start()

        # Обработка событий изменения состояния чекбоксов и поля "Длина пароля"
        if event in ['digits', 'special_chars', 'latin_letters', 'russian_letters', 'uppercase_letters']:
            params = {key: values[key] for key in ['digits', 'special_chars', 'latin_letters', 'russian_letters', 'uppercase_letters', 'length']}
            update_total_combinations(params, window['status_text'])

        if event == 'length':
            update_total_combinations({key: values[key] for key in ['digits', 'special_chars', 'latin_letters', 'russian_letters', 'uppercase_letters', 'length']}, window['status_text'])

    window.close()  # Закрытие окна GUI по завершении работы

def get_combinations(params):
    # Получение символов в зависимости от выбранных параметров
    symbols = ''
    if params['digits']:
        symbols += string.digits
    if params['special_chars']:
        symbols += '@#$%^&*<>_=+-'
    if params['latin_letters']:
        symbols += string.ascii_lowercase
    if params['russian_letters']:
        symbols += 'абвгдеёжзийклмнопрстуфхцчшщъыьэюя'
    if params['uppercase_letters']:
        symbols += symbols.upper()

    return symbols

if __name__ == '__main__':
    main()  # Вызов функции main() при запуске скрипта

