# 🌙 **Проект: "Сонный переводчик"**  
### Приложение для анализа и интерпретации снов на Android (Java)

---

## 🔹 1. Общая концепция

Пользователь утром диктует сон в телефон. Приложение:
- записывает голос,
- распознаёт речь,
- превращает в текст,
- анализирует ключевые образы и эмоции,
- предлагает интерпретации на основе психологии, мифологии и символики,
- визуализирует повторяющиеся темы,
- хранит всё локально — **никаких серверов, без облака, без регистрации**.

> Это **личный дневник снов с интеллектуальным анализом**, работающий полностью на устройстве.

---

## 🔹 2. Основные возможности

| Функция | Описание |
|--------|--------|
| 🎤 Голосовой ввод | Нажал кнопку — говоришь сон, отпустил — запись закончилась |
| 📝 Текстовая транскрипция | Голос автоматически превращается в текст |
| 🔍 Анализ символов | Выделение ключевых образов: *вода, падение, лестница, преследование* |
| 💬 Интерпретация | Пояснения к символам: "Вода — символ подавленных эмоций" |
| 🎨 Сонный атлас | Визуализация частоты символов (облако слов) |
| 🗓 История снов | Просмотр всех снов по датам |
| 🧠 Эмоциональный тон | Определение: сон был тревожным, радостным, странным? |
| 🔐 Приватность | Все данные — только на устройстве. Никаких аккаунтов |

---

## 🔹 3. Архитектура приложения (чистый Java)

### 📱 Стек технологий

| Категория | Технология / Библиотека | Обоснование |
|---------|------------------------|------------|
| Язык | **Java 8+** | Требование |
| UI | **Android XML + ViewBinding** | Стабильно, поддерживается, подходит для Java |
| База данных | **Room Persistence Library** | Надёжно, типобезопасно, работает с Java |
| Аудио | **Android MediaRecorder + SpeechRecognizer** | Встроенные API, не требуют интернета |
| NLP | **Локальный анализ на Java** (регулярные выражения, словари) | Без ML-моделей, полностью автономно |
| Визуализация | **Canvas + кастомный View** или **AndroidPlot** | Для облака слов и графиков |
| Хранение | **Internal Storage** (аудио), **Room** (текст, метаданные) | Без внешних серверов |

---

## 🔹 4. Структура приложения

### 📁 Папки проекта (Java-пакеты)

```
com.dreamtranslator
├── activity          // Главные экраны
│   ├── MainActivity.java
│   ├── RecordDreamActivity.java
│   ├── DreamListActivity.java
│   └── AtlasActivity.java
│
├── fragment          // Фрагменты (если используется)
│
├── service           // Фоновые задачи
│   └── AudioRecordService.java (опционально)
│
├── database          // Room: DAO, Entity, Database
│   ├── DreamDao.java
│   ├── DreamEntity.java
│   └── AppDatabase.java
│
├── model             // POJO-модели
│   └── Dream.java (id, text, audioPath, timestamp, symbols, emotion)
│
├── utils             // Вспомогательные классы
│   ├── SpeechHelper.java        // Работа с распознаванием
│   ├── SymbolAnalyzer.java      // Поиск символов
│   ├── EmotionDetector.java     // Определение эмоции
│   └── WordCloudGenerator.java  // Генерация облака
│
├── assets            // JSON-словари
│   └── dream_symbols.json
│
└── ui                // Кастомные View
    └── WordCloudView.java
```

---

## 🔹 5. Ключевые компоненты

### 1. **Голосовой ввод (SpeechRecognizer)**

```java
// SpeechHelper.java
Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, "ru-RU");
intent.putExtra(RecognizerIntent.EXTRA_PROMPT, "Расскажи свой сон...");

SpeechRecognizer speechRecognizer = SpeechRecognizer.createSpeechRecognizer(context);
speechRecognizer.setRecognitionListener(new RecognitionListener() {
    @Override
    public void onResults(Bundle results) {
        ArrayList<String> matches = results.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION);
        String transcript = matches.get(0); // Лучший результат
        // Передать в анализ
    }
});
```

> ⚠️ Требует разрешения:  
> `<uses-permission android:name="android.permission.RECORD_AUDIO" />`

---

### 2. **Хранение — Room (Java)**

```java
// DreamEntity.java
@Entity(tableName = "dreams")
public class DreamEntity {
    @PrimaryKey(autoGenerate = true)
    public int id;
    public String text;
    public String audioPath;
    public long timestamp;
    public String symbols; // JSON: ["water", "falling"]
    public String emotion; // "fear", "joy", "confusion"
}
```

```java
// DreamDao.java
@Dao
public interface DreamDao {
    @Insert
    void insert(DreamEntity dream);

    @Query("SELECT * FROM dreams ORDER BY timestamp DESC")
    List<DreamEntity> getAllDreams();
}
```

---

### 3. **Анализ символов (SymbolAnalyzer.java)**

```java
// Загружаем словарь из assets/dream_symbols.json
public class SymbolAnalyzer {
    private Map<String, Symbol> symbolMap; // слово → символ

    public List<Symbol> findSymbols(String text) {
        List<Symbol> found = new ArrayList<>();
        text = text.toLowerCase();

        for (String key : symbolMap.keySet()) {
            if (text.contains(key)) {
                found.add(symbolMap.get(key));
            }
            // Можно улучшить: через регулярные выражения, лемматизацию (опционально)
        }
        return found;
    }
}
```

**Файл `dream_symbols.json`:**
```json
[
  {
    "word": "вода",
    "symbol": "water",
    "interpretation": "Символ эмоций, подсознания, очищения.",
    "emotion": "emotional_flow"
  },
  {
    "word": "падать",
    "symbol": "falling",
    "interpretation": "Чувство потери контроля, страха неудачи.",
    "emotion": "fear"
  }
]
```

---

### 4. **Определение эмоции (EmotionDetector.java)**

Простой подход — по ключевым словам:

```java
private Map<String, String> emotionKeywords = new HashMap<>();
// Заполняем: "страх" → "fear", "радость" → "joy", "тоска" → "sadness"

public String detectEmotion(String text) {
    text = text.toLowerCase();
    Map<String, Integer> scores = new HashMap<>();

    for (String word : emotionKeywords.keySet()) {
        if (text.contains(word)) {
            String emo = emotionKeywords.get(word);
            scores.put(emo, scores.getOrDefault(emo, 0) + 1);
        }
    }

    return scores.entrySet().stream()
        .max(Map.Entry.comparingByValue())
        .map(Map.Entry::getKey)
        .orElse("neutral");
}
```

---

### 5. **Облако слов (WordCloudView.java)**

Реализуется через `Canvas`:

```java
public class WordCloudView extends View {
    private List<Word> words; // слово, частота, цвет

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        for (Word word : words) {
            float size = 20 + word.frequency * 5; // масштабируем
            paint.setTextSize(size);
            float x = (float) (Math.random() * getWidth());
            float y = (float) (Math.random() * getHeight());
            canvas.drawText(word.text, x, y, paint);
        }
    }
}
```

---

## 🔹 6. Экраны приложения

### 1. **MainActivity**
- Крупная кнопка: 🎤 "Записать сон"
- Список последних снов (дата, первые 50 символов)
- Иконка внизу: 🗺 "Атлас"

### 2. **RecordDreamActivity**
- Анимация звуковой волны
- Кнопка: "Начать запись" → "Отпустите, чтобы остановить"
- После записи — автоматический анализ

### 3. **DreamDetailActivity**
- Текст сна
- Раздел: **Символы**: `#вода`, `#лестница` — с пояснениями
- Раздел: **Эмоция**: "Тревога", "Радость"
- Кнопка: "Удалить", "Назад"

### 4. **AtlasActivity**
- Облако слов (в центре)
- Фильтры: "Только страхи", "Только радостные"
- График: количество снов по неделям

---

## 🔹 7. Приватность и безопасность

- **Нет интернета** — никаких разрешений на `INTERNET`
- **Нет аккаунтов** — пользователь не вводит имя, email, телефон
- **Аудио и текст** — только во внутреннем хранилище (`context.getFilesDir()`)
- **Резервное копирование** — опционально: экспорт в `.txt` или `.json` (через `Intent.ACTION_CREATE_DOCUMENT`)

---

## 🔹 8. Что нужно для старта

### 1. Подготовить
- `dream_symbols.json` — 50–100 символов с интерпретациями
- Дизайн экранов (можно вручную на бумаге или в Figma)
- Иконка приложения (например, глаз с луной)

### 2. Разработка (пошагово)
1. Настроить Room + сущность `DreamEntity`
2. Сделать `MainActivity` с кнопкой и списком
3. Реализовать голосовой ввод через `SpeechRecognizer`
4. Добавить `SymbolAnalyzer` и `EmotionDetector`
5. Сделать экран деталей сна
6. Реализовать `WordCloudView`
7. Протестировать на устройстве

---

## 🔹 9. Пример: как работает анализ

**Пользователь сказал:**  
> "Я падал с высокой башни, вокруг была тьма, и я не мог кричать."

**Результат:**
- Символы: `падать`, `башня`, `тьма`
- Эмоция: `страх`
- Интерпретация:
  - *Падение* — потеря контроля, страх неудачи.
  - *Башня* — амбиции, изоляция.
  - *Тьма* — неизвестность, подавленные страхи.

---

## 🔹 10. Дополнительно (по желанию)

- **Ночной режим** — тёмная тема по умолчанию
- **Экспорт сна** — сохранить в файл
- **Поиск по снам** — "найти все сны с водой"
- **Жест: свайп влево — удалить**
