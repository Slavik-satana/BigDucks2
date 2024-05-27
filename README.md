# BigDucks2
Репозиторий проекта-решения для Хакатона SF Науки о данных. Команда БигДаки
## Состав команды:

* Самаковский Вячеслав (Преобразование текста в класс, с помощью LLM моделей)
* Казиев Владислав (Разработка классификатора (градиентный бустинг))
* Копанева Ольга (Преобразование текста в параметр для классификатора при помощи генеративной модели, обученной на размеченных человеком данных)
* Рыбальченко Елена (Разработка классификатора (нейронные сети))
* Алексеев Арсалан (Визуализация проекта)
* Новиков Дмитрий (Подбор оптимальной модели для форимрования эмбеддингов)
* Гришин Егор (Расширение обучающего датасета / выборки)



## [Расширение обучающего датасета](gpt_ru.ipynb)
Идея: использовать GPT2 на русском языке(модель ai-forever/rugptsmall_based_on_gpt2) после fine-tuning(а) на основе столбца “human_markup” из датасета. Далее необходимо научить еще одну модель “портить”(corrupt) данные подобно столбцу “model_annotation”. В итоге у должны получиться оба столбца, на основе которых нужно провести проверку соответствия распределения WER для оригинальных данных и сгенерированных.Если всё будет выполнено верно, распределение получиться схожим с оригинальным и эти данные можно будет использовать дальше.

Что пробовали: Пробовал 3 подхода: GPT2(на основе английского оригинала от openai и русскоязычного от Сбера), Seq2Seq(t5-russian-spell), а также проект ai-forever/SAGE (Spelling correction, corruption and evaluation for multiple languages) https://github.com/ai-forever/sage

Что получилось: Первая часть идеи(генерация столбца “human_markup”) получилась успешно с высоким качеством. Вторая часть(генерация столбца “model_annotation” на основе 1 этапа) вызвала затруднения, так как различными способами и моделями не смог получить точное соответствие WER оригиналу. ["Результаты"](corrupts_2hyps.csv)


## Генерация новых признаков для обучения
### ["Расстояние между распознанным автоматически текстом и сгенерированной на его базе фразой"](part_sent_gen.ipynb)
Идея: при генерации текста должна получиться фраза ближе к человеческой речи, т.е. если текст распознан изначально хорошо, сгенерированная по его части фраза будет близка к исходной, а если изначальный текст плохой, то сгенерированный текст будет от него дальше

Что пробовали:
["Вариант 1"](part_sent_gen.ipynb): на основании размеченных человеком фразам обучена GRU RNN для генерации текста; по первой половине исходной фразы сгенерирована вторая;

["Вариант 2"](part_sent_gen_bert.ipynb): применена идея восстановления маскированных токенов - в исходной фразе маскируем отдельные слова и восстанавливаем с помощью модели ruRoBERTa.
В обоих вариантах рассчитано косинусное расстояние между исходной и сгенерированной фразой

Что получилось: в первом варианте корреляции с label не обнаружено; во втором варианте корреляция составила около 10%. ["Результаты"](result.xlsx)

### ["Подбор эмбеддингов"](embeddings-eval.ipynb)
Идея: 
Классический подход для классификации текстов - это преобразование с помощью эмбеддинг модели текста в вектор, который затем подаётся в модель классификатор
Что пробовали:
Протестировали следующий набор эмбеддингов: 
- sbert_large_nlu_ru от Сбера
- multilingual-e5-large 
- LaBSE
- rubert-tiny2
- rubert-base-cased-sentence от DeepPavlov
- fasttext от Facebook

Провели ['дообучение модели'](ft-embed.ipynb) эмбеддингов, что позволилось улучшить качество ['классификации'](sbert-classifier.ipynb) на 6+ процентов.

При подборе классификатора тестировались модели:
- Catboost classifier
- CNN EfficientNet-v2
- Perceptron

Что получилось:
1. Наилучшее качество классификации было получено с эмбеддингами sbert_large_nlu_ru. AUC score 0.758
2. Дообучение модели эмбеддингов позволило добиться AUC score 0.807
3. Наилучшее качество классификации дала Perceptron модель AUC score 0.811


### ["Оценка от GPT модели"](Api_GPT.ipynb)
Идея: GPT модель сможет отличить плохо сгенерированный текст, от текста сказанного человеком. 

Что пробовали: Были попытки получить оценки от следующих моделей: YandexGPT, Mistrall, Copilot, ChatGPT 3.5, ChatGPT 4o.

Что получилось: Была подключена API YandexGPT. Оценки получаемые в автоматическом режиме не коррелировали с реальными классами. Зачастую чат возвращал меньшее количество оценок, чем у него запрашивалось. При большом окне контекста модель выставляла всем фразам минимальные оценки. Остальные же чаты были опробованы в веб режиме. Оценки были более разнообразны, относительно Яндекса, но корреляции с реальными классами также не наблюдалось. Как и у Яндекса часто были проблемы с количеством оценок.



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


