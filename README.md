# Лабораторная работа №4. Взаимодействие с сервером.
Выполнила: Усова Валентина, ИСП-221С
## Что делает приложение?

Приложение использует API сервиса Dadata для поиска почтовых отделений по адресу. Пользователь вводит адрес, приложение отправляет запрос на сервер Dadata, получает результаты и отображает их в списке. Кроме того, приложение позволяет сохранять найденные адреса в локальной базе данных Room и просматривать историю сохраненных адресов.

## Работа приложения


## 1. Обзор MainActivity

MainActivity отвечает за:

• Взаимодействие с пользователем (ввод адреса, нажатие кнопок).
• Отправку запросов к API Dadata и обработку ответов.
• Взаимодействие с базой данных Room (сохранение и загрузка адресов).
• Обновление пользовательского интерфейса (UI).

## 2. Элементы UI

Экран MainActivity содержит следующие элементы:

• EditText (addressEditText): Поле для ввода адреса.
• Button (searchButton): Кнопка запуска поиска почтовых отделений.
• ListView (postOfficesListView): Список, отображающий результаты поиска и сохраненные адреса.
• Button (savedAddressesButton): Кнопка отображения сохраненных адресов.
• Button (clearButton): Кнопка очистки базы данных.


## 3. Обработка ввода пользователя

При нажатии кнопки "Найти" (searchButton):

1. Из addressEditText извлекается введенный пользователем адрес.
2. Формируется и отправляется запрос к API Dadata с помощью библиотеки Volley. Проверяется, не пустое ли поле ввода.
3. Обрабатывается ответ от сервера (см. раздел 4).

Пример кода:
searchButton.setOnClickListener(v -> {
    String address = addressEditText.getText().toString();
    if (address.isEmpty()) {
        Toast.makeText(this, "Пожалуйста, введите адрес", Toast.LENGTH_SHORT).show();
        return;
    }
    findNearestPostOffice(address);
});

## 4. Обработка ответа сервера

Ответ от API Dadata (JSON) обрабатывается следующим образом:

1. Парсинг JSON: Извлекаются адреса почтовых отделений.
2. Обработка ошибок: Обрабатываются ошибки сети и API, отображаются сообщения пользователю через Toast.
3. Обновление UI: Результаты поиска передаются в PostOfficeAdapter для обновления ListView. Обновление происходит в главном потоке с помощью runOnUiThread.

Пример кода:
JsonObjectRequest request = new JsonObjectRequest(Request.Method.POST, url, jsonBody,
    response -> runOnUiThread(() -> {
        try {
            // Парсинг JSON и обновление адаптера
            JSONArray suggestions = response.getJSONArray("suggestions");
            // ... обработка suggestions ...
            adapter.setSearchResults(addresses); // Обновление адаптера
        } catch (JSONException e) {
            showError("Ошибка парсинга JSON");
        }
    }),
    error -> showError("Ошибка сети или API: " + error.getMessage()));
## 5. Взаимодействие с базой данных Room

Используется RxJava для асинхронного взаимодействия:

• Сохранение адресов (saveAddress): При нажатии кнопки "Сохранить" в ListView, адрес сохраняется в базу данных. Проверка на дубликаты выполняется перед сохранением.
• Загрузка сохраненных адресов (showSavedAddresses): Загружает сохраненные адреса из базы данных и обновляет ListView.
• Очистка базы данных (clearDatabase): Очищает базу данных.

Пример кода (фрагмент - saveAddress):
public void saveAddress(String address) {
    disposables.add(addressDatabase.addressDao().getAddressIfExists(address)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .flatMap(existingAddresses -> {
                // ... проверка на существование и вставка нового адреса ...
            })
            .subscribe(
                result -> showMessage("Адрес сохранен!"),
                error -> showError("Ошибка сохранения адреса!")
            ));
}
## 6. Использование Volley и адаптера

• Volley: Используется для отправки HTTP-запросов к API Dadata.
• PostOfficeAdapter: Отображает данные в ListView. Может отображать как результаты поиска, так и сохраненные адреса.

## 7. Управление состоянием приложения

MainActivity отображает сообщения об ошибках и успешном сохранении адресов через Toast. Ресурсы RxJava освобождаются в onDestroy().

## 8. Обработка событий

Обработка нажатий кнопок осуществляется через setOnClickListener:

• searchButton: Вызывает findNearestPostOffice.
• savedAddressesButton: Вызывает showSavedAddresses.
• clearButton: Вызывает clearDatabase.
• Кнопка "Сохранить" в PostOfficeAdapter: Вызывает saveAddress в MainActivity.

  
## Как собрать проект?
1. Загрузка или клонирование репозитория:
* Скачайте файлы проекта (ZIP-архив) и разархивируйте их в удобную папку или клонируйте репозиторий с помощью команды git clone [URL репозитория].

2. Открытие проекта в Android Studio:
* Откройте Android Studio.
* В главном окне выберите "Open an existing Android Studio project".
* Найдите папку, в которую вы скачали или клонировали проект, и выберите файл build.gradle.

3. Запуск приложения:
* В Android Studio, нажмите кнопку "Run" (зеленый треугольник) на панели инструментов.
* Выберите эмулятор или подключенное Android-устройство, на котором вы хотите запустить приложение.
* Дождитесь завершения сборки и запуска приложения.
