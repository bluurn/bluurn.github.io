# Доклад про SLA, SRP и DIP

## Что такое SLA?

SLA - single level of abstraction. Принцип единого уровня абстракций.

### Что означает?

Чтобы повысить читаемость кода, мы пишем код на том уровне абстракции, который приемлем для его понимания без умственных затрат на вникание в реализацию.

### Когда не SLA?

Например, вот:

``` ruby
# spec/features/user_marks_todo_complete_spec.rb
feature "User marks todo complete" do
  scenario "updates todo as completed" do
    sign_in # Норм
    create_todo "Buy milk" # Норм
    # А ниже уже похуже :)
    find(".todos li", text: "Buy milk").click_on "Mark complete"

    expect(page).to have_css(".todos li.completed", text: "Buy milk")
  end

  def create_todo(name)
    click_on "Add new todo"
    fill_in "Name", with: name
    click_on "Submit"
  end
end
```

### Как добиться?

- Корректно именовать идентификаторы
- Выносить сложную логику в отдельные методы/функции

``` ruby
# spec/features/user_marks_todo_complete_spec.rb
feature "User marks todo complete" do
  scenario "updates todo as completed" do
    sign_in
    create_todo "Buy milk"

    mark_complete "Buy milk"

    expect(page).to have_completed_todo "Buy milk"
  end

  def create_todo(name)
    click_on "Add new todo"
    fill_in "Name", with: name
    click_on "Submit"
  end

  def mark_complete(name)
    find(".todos li", text: name).click_on "Mark complete"
  end

  def have_completed_todo(name)
    have_css(".todos li.completed", text: name)
  end
end
```

## Что такое SRP?

Принцип единственной обязанности (Single responsibility principle)

### Что означает?

Каждый объект должен иметь одну обязанность и эта обязанность должна быть полностью инкапсулирована в класс. Все его сервисы должны быть направлены исключительно на обеспечение этой обязанности.

### Ошибки, нарушающие SRP

- Слишком много логики в контроллере
- Слишком много логики в модели
- Слишком много логики в представлении
- Свалка из общих вспомогательных классов

### Как исправить?

Здесь мы стараемся сокрыть реализацию, извлекая группу методов, не связанных напрямую с бизнес-логикой данного класса.

#### Для контроллеров

Инкапсулировать в отдельные классы все, кроме
  * Обработки сессии и cookies.
  * Выбора модели.
  * Управление параметрами запроса.
  * Рендеринг/редирект.

#### Для моделей

Инкапсулировать в отдельные классы все, кроме
  * Конфигурация ActiveRecord
  * Обертки сеттеров, например, `full_name=`
  * Обертки аксесоров, например, `full_name`
  * Scopes, finds, etc

#### Для представлений

Убрать логику в декораторы или domain models.

#### Что делать со свалкой?

Не захламлять проект хелперами, скорее всего их можно сгруппировать иначе:
  - В декораторы
  - В domain models

## Что такое DIP?

Принцип инверсии зависимостей (англ. Dependency Inversion Principle, DIP) —  принцип объектно-ориентированного программирования, используемый для уменьшения связанности в компьютерных программах.

### А точнее?

- Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба типа модулей должны зависеть от абстракций.
- Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

### Ошибки, нарушающие DIP

- Сильная связанность классов

### Как исправить?

- Ликвидировать связанность :)
