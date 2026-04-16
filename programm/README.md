# 📂 Папка с кодом программы

Здесь находится главный файл приложения:
- `password_generator.py` - основной исполняемый файл# password_generator.py
import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext
import random
import string
import datetime
import os
from pathlib import Path

class SecurePassGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("SecurePass Generator - Генератор и анализатор паролей")
        self.root.geometry("900x700")
        self.root.resizable(True, True)
        
        # Настройка логов
        self.setup_logs()
        
        # Создание интерфейса
        self.setup_ui()
        
        # Логируем запуск
        self.log_activity("Приложение запущено")
        
    def setup_logs(self):
        """Создание директории для логов"""
        self.logs_dir = Path("logs")
        self.logs_dir.mkdir(exist_ok=True)
        
        self.log_file = self.logs_dir / "activity.log"
        self.history_file = self.logs_dir / "password_history.txt"
        
    def log_activity(self, message):
        """Запись в журнал действий"""
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        log_entry = f"[{timestamp}] - {message}\n"
        
        try:
            with open(self.log_file, 'a', encoding='utf-8') as f:
                f.write(log_entry)
        except Exception as e:
            print(f"Ошибка записи лога: {e}")
    
    def save_password_history(self, password, strength, action_type="generated"):
        """Сохранение истории паролей (без самих паролей для безопасности)"""
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        # Сохраняем только метаданные, не сам пароль
        history_entry = f"[{timestamp}] - {action_type}: длина={len(password)}, надежность={strength}\n"
        
        try:
            with open(self.history_file, 'a', encoding='utf-8') as f:
                f.write(history_entry)
        except Exception as e:
            print(f"Ошибка сохранения истории: {e}")
    
    def setup_ui(self):
        """Создание графического интерфейса"""
        # Создаем вкладки
        self.notebook = ttk.Notebook(self.root)
        self.notebook.pack(fill='both', expand=True, padx=10, pady=5)
        
        # Вкладка генератора
        self.generator_frame = ttk.Frame(self.notebook)
        self.notebook.add(self.generator_frame, text="🔐 Генератор паролей")
        self.setup_generator_tab()
        
        # Вкладка анализатора
        self.analyzer_frame = ttk.Frame(self.notebook)
        self.notebook.add(self.analyzer_frame, text="📊 Анализатор надежности")
        self.setup_analyzer_tab()
        
        # Вкладка истории
        self.history_frame = ttk.Frame(self.notebook)
        self.notebook.add(self.history_frame, text="📜 История операций")
        self.setup_history_tab()
        
    def setup_generator_tab(self):
        """Настройка вкладки генератора"""
        # Фрейм для настроек
        settings_frame = ttk.LabelFrame(self.generator_frame, text="Настройки генерации", padding=10)
        settings_frame.pack(fill='x', padx=10, pady=10)
        
        # Длина пароля
        ttk.Label(settings_frame, text="Длина пароля:").grid(row=0, column=0, sticky='w', pady=5)
        self.password_length = tk.IntVar(value=12)
        self.length_scale = ttk.Scale(settings_frame, from_=8, to=32, variable=self.password_length, 
                                      orient='horizontal', command=self.update_length_label)
        self.length_scale.grid(row=0, column=1, sticky='ew', padx=10, pady=5)
        self.length_label = ttk.Label(settings_frame, text="12")
        self.length_label.grid(row=0, column=2, padx=5)
        
        # Чекбоксы для типов символов
        self.use_lowercase = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame, text="Строчные буквы (a-z)", 
                       variable=self.use_lowercase).grid(row=1, column=0, columnspan=2, sticky='w', pady=5)
        
        self.use_uppercase = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame, text="Заглавные буквы (A-Z)", 
                       variable=self.use_uppercase).grid(row=2, column=0, columnspan=2, sticky='w', pady=5)
        
        self.use_digits = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame, text="Цифры (0-9)", 
                       variable=self.use_digits).grid(row=3, column=0, columnspan=2, sticky='w', pady=5)
        
        self.use_symbols = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame, text="Специальные символы (!@#$%^&*()_+-=[]{}|;:,.<>?)", 
                       variable=self.use_symbols).grid(row=4, column=0, columnspan=2, sticky='w', pady=5)
        
        # Кнопка генерации
        ttk.Button(settings_frame, text="🔑 Сгенерировать пароль", 
                  command=self.generate_password).grid(row=5, column=0, columnspan=3, pady=15)
        
        # Результат генерации
        result_frame = ttk.LabelFrame(self.generator_frame, text="Результат", padding=10)
        result_frame.pack(fill='both', expand=True, padx=10, pady=10)
        
        self.generated_password = tk.StringVar()
        password_entry = ttk.Entry(result_frame, textvariable=self.generated_password, 
                                   font=('Courier', 12), state='readonly')
        password_entry.pack(fill='x', padx=10, pady=10)
        
        # Кнопки действий
        button_frame = ttk.Frame(result_frame)
        button_frame.pack(fill='x', padx=10, pady=5)
        
        ttk.Button(button_frame, text="📋 Копировать в буфер", 
                  command=self.copy_to_clipboard).pack(side='left', padx=5)
        
        ttk.Button(button_frame, text="🔄 Очистить", 
                  command=lambda: self.generated_password.set("")).pack(side='left', padx=5)
        
        # Оценка надежности
        self.strength_label = ttk.Label(result_frame, text="", font=('Arial', 10, 'bold'))
        self.strength_label.pack(pady=10)
        
    def setup_analyzer_tab(self):
        """Настройка вкладки анализатора"""
        # Поле для ввода пароля
        input_frame = ttk.LabelFrame(self.analyzer_frame, text="Введите пароль для проверки", padding=10)
        input_frame.pack(fill='x', padx=10, pady=10)
        
        self.password_to_check = tk.StringVar()
        password_entry = ttk.Entry(input_frame, textvariable=self.password_to_check, 
                                   font=('Courier', 11), show="*")
        password_entry.pack(fill='x', padx=10, pady=10)
        
        # Кнопка показать/скрыть
        self.show_password = tk.BooleanVar(value=False)
        ttk.Checkbutton(input_frame, text="Показать пароль", 
                       variable=self.show_password, command=self.toggle_password_visibility).pack(pady=5)
        
        ttk.Button(input_frame, text="🔍 Проверить надежность", 
                  command=self.analyze_password).pack(pady=10)
        
        # Результаты анализа
        result_frame = ttk.LabelFrame(self.analyzer_frame, text="Результат анализа", padding=10)
        result_frame.pack(fill='both', expand=True, padx=10, pady=10)
        
        self.analysis_result = scrolledtext.ScrolledText(result_frame, height=10, width=60, wrap=tk.WORD)
        self.analysis_result.pack(fill='both', expand=True, padx=10, pady=10)
        
    def setup_history_tab(self):
        """Настройка вкладки истории"""
        control_frame = ttk.Frame(self.history_frame)
        control_frame.pack(fill='x', padx=10, pady=5)
        
        ttk.Button(control_frame, text="🔄 Обновить историю", 
                  command=self.load_history).pack(side='left', padx=5)
        
        ttk.Button(control_frame, text="💾 Экспортировать историю", 
                  command=self.export_history).pack(side='left', padx=5)
        
        ttk.Button(control_frame, text="🗑️ Очистить историю", 
                  command=self.clear_history).pack(side='left', padx=5)
        
        # Текстовое поле для истории
        self.history_text = scrolledtext.ScrolledText(self.history_frame, wrap=tk.WORD, font=('Courier', 9))
        self.history_text.pack(fill='both', expand=True, padx=10, pady=10)
        
        # Загружаем историю
        self.load_history()
        
    def update_length_label(self, *args):
        """Обновление отображения длины пароля"""
        self.length_label.config(text=str(int(self.password_length.get())))
    
    def generate_password(self):
        """Генерация пароля"""
        length = int(self.password_length.get())
        
        # Формируем пул символов
        chars = ""
        if self.use_lowercase.get():
            chars += string.ascii_lowercase
        if self.use_uppercase.get():
            chars += string.ascii_uppercase
        if self.use_digits.get():
            chars += string.digits
        if self.use_symbols.get():
            chars += "!@#$%^&*()_+-=[]{}|;:,.<>?"
        
        if not chars:
            messagebox.showwarning("Ошибка", "Выберите хотя бы один тип символов!")
            return
        
        # Генерация пароля
        password = ''.join(random.choice(chars) for _ in range(length))
        self.generated_password.set(password)
        
        # Оценка надежности
        strength, score = self.check_password_strength(password)
        
        # Обновление метки надежности
        colors = {"Слабый": "red", "Средний": "orange", "Сильный": "green"}
        self.strength_label.config(text=f"Надежность: {strength} (баллов: {score}/10)", 
                                   foreground=colors.get(strength, "black"))
        
        # Логирование
        self.log_activity(f"Сгенерирован пароль: длина={length}, надежность={strength}")
        self.save_password_history(password, strength, "generated")
    
    def check_password_strength(self, password):
        """Проверка надежности пароля (возвращает уровень и баллы)"""
        score = 0
        recommendations = []
        
        # Длина пароля
        if len(password) >= 12:
            score += 3
            recommendations.append("✓ Отличная длина пароля (12+ символов)")
        elif len(password) >= 8:
            score += 2
            recommendations.append("⚠️ Хорошая длина, но рекомендуем 12+ символов")
        else:
            recommendations.append("❌ Пароль слишком короткий (минимум 8 символов)")
        
        # Строчные буквы
        if any(c.islower() for c in password):
            score += 1
            recommendations.append("✓ Есть строчные буквы")
        else:
            recommendations.append("⚠️ Добавьте строчные буквы")
        
        # Заглавные буквы
        if any(c.isupper() for c in password):
            score += 1
            recommendations.append("✓ Есть заглавные буквы")
        else:
            recommendations.append("⚠️ Добавьте заглавные буквы")
        
        # Цифры
        if any(c.isdigit() for c in password):
            score += 1
            recommendations.append("✓ Есть цифры")
        else:
            recommendations.append("⚠️ Добавьте цифры")
        
        # Специальные символы
        special_chars = "!@#$%^&*()_+-=[]{}|;:,.<>?"
        if any(c in special_chars for c in password):
            score += 2
            recommendations.append("✓ Есть специальные символы")
        else:
            recommendations.append("⚠️ Добавьте специальные символы")
        
        # Разнообразие символов
        unique_chars = len(set(password))
        if unique_chars > len(password) * 0.7:
            score += 2
            recommendations.append("✓ Хорошее разнообразие символов")
        
        # Определение уровня надежности
        if score >= 7:
            strength = "Сильный"
        elif score >= 4:
            strength = "Средний"
        else:
            strength = "Слабый"
        
        return strength, min(score, 10)
    
    def analyze_password(self):
        """Анализ введенного пароля"""
        password = self.password_to_check.get()
        
        if not password:
            messagebox.showwarning("Ошибка", "Введите пароль для проверки!")
            return
        
        strength, score = self.check_password_strength(password)
        
        # Формирование результата анализа
        self.analysis_result.delete(1.0, tk.END)
        
        result_text = f"🔍 РЕЗУЛЬТАТ АНАЛИЗА ПАРОЛЯ\n"
        result_text += f"{'='*50}\n\n"
        result_text += f"Длина пароля: {len(password)} символов\n"
        result_text += f"Уровень надежности: {strength.upper()}\n"
        result_text += f"Балльная оценка: {score}/10\n\n"
        result_text += f"📊 ДЕТАЛЬНЫЙ АНАЛИЗ:\n"
        result_text += f"{'-'*50}\n"
        
        # Детальный анализ
        if len(password) >= 12:
            result_text += "✅ Длина: Отлично (12+ символов)\n"
        elif len(password) >= 8:
            result_text += "⚠️ Длина: Хорошо, но рекомендуем 12+ символов\n"
        else:
            result_text += "❌ Длина: Слишком короткий\n"
        
        result_text += "✅ Строчные буквы: " + ("Есть\n" if any(c.islower() for c in password) else "Нет\n")
        result_text += "✅ Заглавные буквы: " + ("Есть\n" if any(c.isupper() for c in password) else "Нет\n")
        result_text += "✅ Цифры: " + ("Есть\n" if any(c.isdigit() for c in password) else "Нет\n")
        
        special_chars = "!@#$%^&*()_+-=[]{}|;:,.<>?"
        result_text += "✅ Спецсимволы: " + ("Есть\n" if any(c in special_chars for c in password) else "Нет\n")
        
        result_text += f"\n💡 РЕКОМЕНДАЦИИ ПО УЛУЧШЕНИЮ:\n"
        result_text += f"{'-'*50}\n"
        
        if score < 7:
            if len(password) < 12:
                result_text += "• Увеличьте длину пароля до 12+ символов\n"
            if not any(c.isupper() for c in password):
                result_text += "• Добавьте заглавные буквы (A-Z)\n"
            if not any(c.isdigit() for c in password):
                result_text += "• Добавьте цифры (0-9)\n"
            if not any(c in special_chars for c in password):
                result_text += "• Добавьте специальные символы (!@#$%^&*)\n"
        else:
            result_text += "✓ Ваш пароль соответствует современным стандартам безопасности!\n"
        
        self.analysis_result.insert(1.0, result_text)
        
        # Логирование
        self.log_activity(f"Проверен пароль: надежность={strength}, оценка={score}/10")
    
    def toggle_password_visibility(self):
        """Переключение видимости пароля в анализаторе"""
        if self.show_password.get():
            self.analyzer_frame.winfo_children()[0].winfo_children()[1].config(show="")
        else:
            self.analyzer_frame.winfo_children()[0].winfo_children()[1].config(show="*")
    
    def copy_to_clipboard(self):
        """Копирование пароля в буфер обмена"""
        password = self.generated_password.get()
        if password:
            self.root.clipboard_clear()
            self.root.clipboard_append(password)
            messagebox.showinfo("Успех", "Пароль скопирован в буфер обмена!")
            self.log_activity("Пароль скопирован в буфер обмена")
        else:
            messagebox.showwarning("Ошибка", "Нет пароля для копирования!")
    
    def load_history(self):
        """Загрузка истории операций"""
        self.history_text.delete(1.0, tk.END)
        
        try:
            if self.log_file.exists():
                with open(self.log_file, 'r', encoding='utf-8') as f:
                    logs = f.read()
                    self.history_text.insert(1.0, logs)
            else:
                self.history_text.insert(1.0, "История операций пуста.")
        except Exception as e:
            self.history_text.insert(1.0, f"Ошибка загрузки истории: {e}")
    
    def export_history(self):
        """Экспорт истории в файл"""
        timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
        export_file = self.logs_dir / f"history_export_{timestamp}.txt"
        
        try:
            if self.log_file.exists():
                with open(self.log_file, 'r', encoding='utf-8') as src:
                    content = src.read()
                with open(export_file, 'w', encoding='utf-8') as dst:
                    dst.write(content)
                messagebox.showinfo("Успех", f"История экспортирована в файл:\n{export_file}")
                self.log_activity(f"История экспортирована в {export_file.name}")
            else:
                messagebox.showwarning("Ошибка", "Нет данных для экспорта!")
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось экспортировать историю: {e}")
    
    def clear_history(self):
        """Очистка истории (с подтверждением)"""
        if messagebox.askyesno("Подтверждение", "Вы уверены, что хотите очистить всю историю?"):
            try:
                # Создаем резервную копию
                backup_file = self.logs_dir / f"history_backup_{datetime.datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"
                if self.log_file.exists():
                    import shutil
                    shutil.copy(self.log_file, backup_file)
                
                # Очищаем файл
                with open(self.log_file, 'w', encoding='utf-8') as f:
                    f.write("")
                
                # Записываем новую запись об очистке
                self.log_activity("История операций очищена пользователем")
                self.load_history()
                messagebox.showinfo("Успех", "История очищена. Резервная копия сохранена.")
            except Exception as e:
                messagebox.showerror("Ошибка", f"Не удалось очистить историю: {e}")

def main():
    root = tk.Tk()
    app = SecurePassGenerator(root)
    root.mainloop()

if __name__ == "__main__":
    main()

## Как запустить:
1. Откройте командную строку/терминал
2. Перейдите в эту папку: `cd programm`
3. Запустите: `python password_generator.py`
