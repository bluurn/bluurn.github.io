# Доклад про SLA, SRP и DIP

## Что такое SLA?

SLA - single level of abstraction. Принцип единого уровня абстракций.

### Что означает?

Чтобы повысить читаемость кода, мы пишем код на том уровне абстракции, который приемлем для его понимания без умственных затрат на вникание в реализацию.

### Когда не SLA?

Когда уровни абстракций смешаны.

### Пример

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

### Как исправить?

- Корректно именовать идентификаторы
- Выносить сложную логику в отдельные методы/функции

### Исправленный пример

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

### Пример без SRP:

``` ruby
# app/domains/student.rb
class Student
  attr_accessor :first_term_home_work, :first_term_test,
    :first_term_paper
  attr_accessor :second_term_home_work, :second_term_test,
    :second_term_paper

  def first_term_grade
    (first_term_home_work + first_term_test + first_term_paper) / 3
  end

  def second_term_grade
    (second_term_home_work + second_term_test + second_term_paper) / 3
  end

end
```

### Почему не SRP?

Класс `Student` содержит логику, которую можно вынести в отдельный класс `Grade`

### Исправленный пример:

``` ruby
# app/domains/student.rb
class Student
  def initialize
    @terms = [
      Grade.new(:first),
      Grade.new(:second)
      ]
  end

  def first_term_grade
    term(:first).grade
  end

  def second_term_grade
    term(:second).grade
  end

  private

  def term reference
    @terms.find {|term| term.name == reference}
  end
end

# app/domains/grade.rb
class Grade
  attr_reader :name, :home_work, :test, :paper

  def initialize(name)
    @name      = name
    @home_work = 0
    @test      = 0
    @paper     = 0
  end

  def grade
    (home_work + test + paper) / 3
  end
end
```


### SRP и Rails

- Слишком много логики в модели
- Слишком много логики в контроллере
- Слишком много логики в представлении
- Свалка из общих вспомогательных классов


### Как исправить?

Инкапсулируем реализацию, извлекая группу методов, не связанных напрямую с логикой данного класса в отдельные классы.


#### Для моделей

Инкапсулировать в отдельные классы все, кроме
  * Конфигурация ActiveRecord
  * Обертки сеттеров
  * Обертки геттеров
  * Scopes, finds, etc

#### Исправленная модель


#### Для контроллеров

Инкапсулировать в отдельные классы все, кроме
  * Обработки сессии и cookies.
  * Выбора модели.
  * Управление параметрами запроса.
  * Рендеринг/редирект.

#### Для представлений

Убрать логику в декораторы или service models.

#### Что делать со свалкой?

Не захламлять проект хелперами, скорее всего их можно сгруппировать иначе:
  - В декораторы
  - В service models


## Что такое DIP?

Принцип инверсии зависимостей (англ. Dependency Inversion Principle, DIP) —  принцип объектно-ориентированного программирования, используемый для уменьшения связанности в компьютерных программах.

### А точнее?

- Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба типа модулей должны зависеть от абстракций.
- Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

### Ошибки, нарушающие DIP

- Сильная связанность классов

### Пример

``` ruby
# app/domains/report.rb
class Report  
  def initialize
    @body = "whatever"
  end

  def print
    XmlFormatter.new.generate @body
  end
end
# app/domains/xml_formatter.rb
class XmlFormatter  
  def generate(body)
    # convert the body argument into XML
  end
end  

```
### Как исправить?

- Ликвидировать связанность :)

### Исправленный пример

``` ruby
# app/domains/report.rb
class Report  
  def initialize
    @body = "whatever"
  end

  def print(formatter)
    formatter.generate @body
  end
end

# app/domains/xml_formatter.rb
class XmlFormatter  
  def generate(body)
    # convert the body argument into XML
  end
end  
```
