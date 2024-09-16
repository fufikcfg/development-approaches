# Domain Driven Design

## DDD

Domain-Driven Design (далее DDD) - это архитектурный подход, который структурирует приложение вокруг следующих основных слоев: Domain Layer, Application Layer, Infrastructure Layer и Presentation Layer в случаях, когда есть визуализация ответов.

## Предметная область

Предметная область (Domain) в контексте DDD представляет собой область знаний, проблем и задач, которые решаются в рамках конкретной предметной области или бизнес-домена.

Например:

Электронная коммерция:

* Пользователи (User.php): Содержит информацию о зарегистрированных пользователях.

* Товары (Product.php): Представляет товары, которые можно приобрести на платформе.

Бронирование отелей:

* Отели (Hotel.php): Содержит информацию о доступных отелях, категориях номеров, ценах и услугах.

* Отзывы (Review.php): Содержит отзывы и рейтинги от посетителей отелей, комментарии и оценки различных аспектов сервиса.

## Общая предметная область (Shared)

Shared тносится к компонентам, классам или модулям, которые не принадлежат непосредственно ни к одному из четырех слоев DDD (Domain, Infrastructure, Application, Presentation), а являются общими и могут использоваться в разных частях приложения.

Основное применение:

* Повторное использование кода: В слое Shared могут быть размещены общие утилиты, библиотеки, сервисы и функции, которые могут использоваться многократно в разных частях приложения.

Пример:

```
src/
├── Ecommerce/
│   └── ...
├── HotelReservation/
│   └── ...
├── ProjectManagement/
│   └── ...
└── Shared/
    ├── Application/
    │   └── ...
    ├── Domain/
    │   └── ...
    └── Infrastructure/
        ├── Responder/
        ├── Service/
        ├── Storage/
        └── Serializer/
```

## Слои:

### Domain Layer

Доменный слой - это центральный слой в DDD, где располагается бизнес-логика и модели предметной области. Здесь выделяются основные сущности.

Цель выделения сущностей - это набор их свойств, но не поведения. Сущность задает логическую структуру данных, которая используется в бизнес-действиях.

* Не зависим от внешних библиотек, Domain не привязан к БД и никогда ничего не знает про детали реализации приложения (например, какую БД использовать, вид кеша и тд).

* Не привязан к фреймворку. Например, доменный слой предметной области в приложении на Symfony, который перенесен на приложение Laravel должен будет сохранить свою работоспособность.

#### Пример Domain Layer:

```
src/
└── Users/
    ├── Application/
    │   └── ...
    ├── Domain/
    │   ├── Entity/
    │   │   └── User.php
    │   └── Factory/
    │       └── UserFactory.php
    └── Infrastructure/
        └── ...
```

##### Entity

User.php представляет собой сущность, которая отражает один из объектов предметной области.
 Сущности отражают концепции, ключевые для предметной области. Они содержат состояние и основные операции, которые могут быть выполнены над этим состоянием. Реализация сущности хранится в доменном слое, так как она прямо связана с предметной областью и бизнес-правилами.

```php
namespace App\Users\Domain\Entity;

class User
{
    readonly int $id;

    public function __construct(
        private string $username,
        private array $projects,
        private readonly \DateTime $createdAt,
    ) {
    }

    public function getUsername(): string
    {
        return $this->username;
    }

    public function setUsername(string $username): void
    {
        $this->username = $username;
    }

    public function addProject(Project $project): void
    {
        $this->projects[] = $project;
    }

    public function getProjects()
    {
        return $this->projects;
    }

    public function getId(): int
    {
        return $this->id;
    }

    public function getCreatedAt(): \DateTime
    {
        return $this->createdAt;
    }
}
```

##### Factory

UserFactory.php фабрика, которая отвечает за создание объектов сущности User. Реализация фабрики хранится в доменном слое вместе с сущностью, так как она связана с созданием и инициализацией объектов сущностей, что прямо относится к предметной области.

```php
namespace App\Users\Domain\Factory;

use App\Users\Domain\Entity\User;

class UserFactory
{
    public function __construct(
        private readonly \DateTime $dateTime = new \DateTime()
    ) {
    }

    public function create(string $username): User
    {
        return new User(
            $username,
            $this->dateTime,
        );
    }
}
```

### Application Layer

Слой Приложения служит в качестве прослойки между представлением (UI) и доменным слоем. Он содержит службы приложения, которые координируют взаимодействие между представлением (Infrastructure Layer, или в некоторых случаях Presentation Layer) и доменным слоем, а также ведут работу по управлению транзакциями, аутентификации, авторизации и другими аспектами, не относящимися непосредственно к бизнес-логике.

* Этот слой содержит логику координации и управления выполнением бизнес-операций.

* Здесь располагаются службы приложения (Application Services), которые используют доменные объекты для выполнения конкретных задач.

* Является прослойкой между пользовательским интерфейсом и доменным слоем.

#### Пример Application Layer:

```
src/
└── Users/
    ├── Application/
    │   ├── DTO/
    │   │   └── UserDTO.php
    │   └── UseCase/
    │       └── Query/
    │           └── GetUserById/
    │               ├── GetUserByIdHandler.php
    │               └── GetUserByIdQuery.php
    ├── Domain/
    │   └── ...
    └── Infrastructure/
        └── ...

```

##### DTO

Объекты передачи данных (Data Transfer Objects) используются для переноса данных между слоями приложения. Например, UserDTO может содержать данные о пользователе, которые должны быть переданы между слоями или компонентами приложения.

```php
readonly class UserDTO implements ArrayableInterface
{
    public function __construct(
        private int $id,
        private string $username,
        private \DateTime $createdAt,
    ) {
    }

    public function getId(): int
    {
        return $this->id;
    }

    public function getUsername(): string
    {
        return $this->username;
    }

    public function getCreatedAt(): \DateTime
    {
        return $this->createdAt;
    }


    public function toArray(): array
    {
        return [
            'id' => $this->getId(),
            'username' => $this->getUsername(),
            'createdAt' => $this->getCreatedAt(),
        ];
    }
}
```

##### UseCase

Представляет определенное использование в системе, которое часто охватывает определенную функциональность или операцию бизнес-логики. Процессы, без которых не может существовать бизнес. 

Так же содержит обработчик (Handler), - представляет класс, который фактически выполняет операцию, связанную с определенным UseCase и Query (запрос), который содержит необходимые параметры и логику для запроса данных о пользователе по его идентификатору.

Примеры UseCase:

* RegisterUserHandler: операция регистрации нового пользователя в системе. UseCase может включать в себя валидацию пользовательских данных, создание учетной записи, отправку подтверждения по почте, генерацию токена доступа.

* AuthenticateUserHandler: аутентификация пользователя в системе. UseCase может проверять логин и пароль, генерацию токена доступа, управление сеансом пользователя и другие действия, связанные с аутентификацией.

* ProcessPaymentHandler: обработка платежа пользователя. UseCase может включать в себя взаимодействие с платежным шлюзом, проверку статуса платежа, журналирование транзакций и обновление информации о заказе.

```php
namespace App\Users\Application\UseCase\Query\GetUserById;

use App\Shared\Application\Query\Query;

readonly class GetUserByIdQuery extends Query
{
    public function __construct(
        private int $userId
    ) {
    }

    public function getUserId(): int
    {
        return $this->userId;
    }
}
```

```php
namespace App\Users\Application\UseCase\Query\GetUserById;

use App\Shared\Application\Query\QueryInterface;
use App\Users\Application\DTO\UserDTO;
use App\Users\Infrastructure\Repository\UserRepository;

readonly class GetUserByIdHandler implements QueryInterface
{
    public function __construct(
        private UserRepository $userRepository,

    ) {
    }

    public function handle(GetUserByIdQuery $query): array
    {
        $user = $this->userRepository->findById($query->getUserId());

        return (new UserDTO(
            $user->getId(),
            $user->getUsername(),
            $user->getCreatedAt(),
        ))->toArray();
    }
}
```

#### Infrastructure Layer:

Предназначен для хранения инфраструктурного кода, который связан с внешними зависимостями системы, такими как базы данных, фреймворки, внешние API и другие технические аспекты.

#### Пример Infrastructure Layer:
```
src/
└── Users/
    ├── Application/
    │   └── ...
    ├── Domain/
    │   └── ...
    └── Infrastructure/
        ├── Controller/
        │   └── GetUserByIdAction.php
        ├── Doctrine/
        │   └── User.orm.xml
        └── Repository/
            └── UserRepository.php

```

##### Doctrine

Файл используется для маппинга объектов (ORM) в базе данных с помощью Doctrine или других ORM-решений. Содержит информацию о том, как сущности доменной модели отображаются в базу данных. Поэтому он относится к слою Infrastructure, поскольку это детали реализации, специфичные для работы с базой данных и ORM.

##### Repository

Репозиторий отвечает за выполнение операций с базой данных. Хранится в слое Infrastructure поскольку может включать в себя интеграцию с внешними сервисами или API, которые используются для получения, сохранения или обновления данных, использование конкретных библиотек для работы с базой данных и управлением соединения.

```php
namespace App\Users\Infrastructure\Repository;

class UserRepository extends ServiceEntityRepository
{
    private EntityManagerInterface $_em;

    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, User::class);
        $this->_em = $this->getEntityManager();
    }

    public function findById(int $id): User
    {
        return $this->find($id);
    }

    public function create(User $user): User
    {
        $this->_em->persist($user);
        $this->_em->flush();

        return $user;
    }
}
```

##### Controller

Контроллер отвечает за обработку входящих запросов от клиента, передачу данных между слоями и визуализацию ответов. Но так же, контроллер может рассматриваться как часть слоя Infrastructure, так как обрабатывает входящие запросы и далее диспетчирует их между слоями. Может относиться к слою Представления (Presentation Layer).

```php
namespace App\Users\Infrastructure\Controller;

#[AsController]
#[Route('/users/{id}', methods: ['GET'])]
class GetUserByIdAction
{
    public function __construct(
        readonly private GetUserByIdHandler $getUserByIdHandler,
    ) {
    }

    public function __invoke(int $id): UserResponder
    {
        return new UserResponder($this->getUserByIdHandler->handle(new GetUserByIdQuery($id));
    }
}
```

#### Presentation Layer:

Слой для работы с фронтом приложения. Не требуется, когда REST API, но можно.

#### Пример Presentation Layer:

```php
src
  Users
    Application/
      ...
    Domain/
      ...
    Infrastructure/
      ...
    Presentation/
      Controller/
        GetUserByIdAction.php
views/
  user.html.twig
```

##### Controller

```php
namespace App\Users\Presentation\Controller;

#[AsController]
#[Route('/users/{id}', methods: ['GET'])]
class GetUserByIdAction
{
    public function __construct(
        readonly private GetUserByIdHandler $getUserByIdHandler,
    ) {
    }

    public function __invoke(int $id): Response
    {
        $user = $this->getUserByIdHandler->handle(new GetUserByIdQuery($id);

        return $this->render('@views/user.html.twig', [
            'username' => $user->getUsername(),
            'createdAt' => $user->getCreatedAt()
        ]);
    }
}
```

# Плюсы и минусы:

## Минусы:

1. Сложность: Это сложно, требует времени и усилий от команды, некоторые проекты могут не нуждаться в таком уровне абстракции, который предлагает DDD
2. Дорого: Дороговизна в разработке, на построение архитектуры и выполнение задач требуется больше времени
3. Много однотипного кода: Много кода схожего друг с другом
4. Постоянный самоконтроль: Нужно постоянно себя контролировать, чтобы не использовать классы/интерфейсы из других слоев/фич, минуя api

## Плюсы:

1. Понимание и один язык с менеджерами: Помогает лучше понять бизнес-процессы и потребности пользователей и дает возможность говорить на одном языке, что ведет к более качественному и соответствующему продукту.
2. Гибкость, масштабируемость и слабая зависимость: Позволяет создавать гибкие и расширяемые архитектуры приложений, что облегчает поддержку и доработку системы в будущем в которой слои не зависят друг от друга
3. Легко тестировать: За счет изолирования, кода по слоям, при изменении кода не нужно переписывать большое количество тестов
4. Дешево поддерживать: В перспективе

# Вывод:

## Это инструмент для создания расширяемого и стабильного приложения и образ мышления, но не все клиенты, у которых проекты с маленьким жизненным циклом, готовы платить больше за хорошую архитектуру. Для проектов с бОльшим жизненным циклом подойдет Porto, который унаследовал концепции из DDD - более простой в понимании и требует меньше усилий на написание кода.

# Источники:

* https://habr.com/ru/articles/718916/ - статью читать не полностью, прочитать комментарии

* https://www.youtube.com/watch?v=ahKDDQWXzWw - норм объяснение и примеры предметной области

* https://github.com/alejandro-yakovlev/symfony-docker - пример реализации (с ошибками)

* Эрик Эванс, Предметно-ориентированное проектирование (DDD) - не более глубокое погружение в тему 