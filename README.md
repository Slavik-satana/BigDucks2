# BigDucks2
Репозиторий проекта-решения для Хакатона SF Науки о данных. Команда БигДаки
## Состав команды:

*Самаковский Вячеслав (Преобразование текста в класс, с помощью LLM моделей)
*Казиев Владислав (Разработка классификатора (градиентный бустинг))
*Копанева Ольга (Преобразование текста в параметр для классификатора при помощи генеративной модели, обученной на размеченных человеком данных)
*Рыбальченко Елена (Разработка классификатора (нейронные сети))
*Алексеев Арсалан (Визуализация проекта)
*Новиков Дмитрий (Подбор оптимальной модели для форимрования эмбеддингов)
*Гришин Егор (Расширение обучающего датасета / выборки)



## Расширение обучающего датасета


## Генерация новых признаков для обучения
### ["Расстояние между распознанным автоматически текстом и сгенерированной на его базе фразой"](part_sent_gen.ipynb)
Гипотеза: при генерации текста должна получиться фраза ближе к человеческой речи, т.е. если текст распознан изначально хорошо, сгенерированная по его части фраза будет близка к исходной, а если изначальный текст плохой, то сгенерированный текст будет от него дальше
Что сделано: предобработаны исходные данные (лемматизированы и токенизированы до уровня слов); на основании столбца с размеченными человеком фразами (B) обучена GRU RNN для генерации текста; по первой половине исходной фразы (A) сгенерирована вторая половина; рассчитано косинусное расстояние между исходной и сгенерированной фразой
Предстоящие изменения / улучшения: проверить тот же путь на большем объеме данных (на увеличенном датасете); проверить вариант с генерацией по символам (а не словам); проверить вариант с предобученными word2 vec; проверить вариант с Masked Language Modelling

### ["Подбор эмбеддингов"](embeddings-eval.ipynb)
Брали предобученные модели эмбеддингов кодировал первый столбец (model_annotation) и делали классификацию катбустом. Наилучшее качество классификации дают эмбеддинги из Сберовской библиотеки - 0.76 на валидации по метрике AUC. Далее можно дообучить модели эмбеддингов на наши тексты - качество немного вырастет, и заменить катбуст нейронками. Думаю при таком базовом подходе можно будет 0.8 по ROC AUC получить на валидации.

### ["Оценка от GPT модели"](Api_GPT.ipynb)
Гипотиза: GPT модель сможет отличить плохо сгенерированный текст, от текста сказанного человеком.
Что сделано: Подключена API YandexGPT. С помощью нее проставленны оценки от 1 до 10 всем входным фразам из датасета. Так же исследованы такие модели как Mistrall, Copilot, ChatGPT 3.5, ChatGPT 4o. Эти модели хоть и показали более высокий результат, но сложны в подключении или требуют больших материальных ресурсов.


## Обучение модели

### ["LoRA (low rank adaptation)"](LORA_funetune.ipynb)

Тезисы:
-  всю модель обучать вычислительно дорого;
- классификаторы поверх фичей также требуют дополнительных вычислений к основной модели;
- обучение LoRA не требует дополнительных классификаторов и обучает часть весов.

Таким образом предположила, что это может быть оптимальным вариантом.

Взяла за основу модель RuBERT и обучила LoRA, auc валидации 0.737, что выше, чем base line 0.62

Возможные пути для дальнейшего улучшения:
- аугментации;
- попробовать настройку гиперпарамеров LoRA;
- попробовать другие базовые сети;
- попробовать более мощные, но квантованные сети.

### "Классификатор на базе Градиентного бустинга"

После получения всех признаков будет исследованы в том числе модели на основе градиентного бустинга. Сейчас это направление деятельности невозможно, поскольку нет готовых признаков.
