https://habr.com/ru/articles/989608/

Снова здравствуйте!

Сегодня мы рассмотрим обработку ошибок и механизм **Dead Letter Queue**. Это две тесно связанные темы, которые важно понимать для правильной работы распределённых систем в исключительных ситуациях.

В этой статье не будет ничего сложного. Мы не будем лезть в устройство брокера сообщений, так как для этой темы этого не потребуется. Если вы читали все предыдущие статьи этого цикла, то все приёмы вам будут знакомы, так как с этим всем вы уже работали.

Здесь мы будем использовать нашу платформу. Крайний раз она менялась в [данной статье](https://habr.com/ru/articles/965218/).

## Ошибки десериализации

Для начала рассмотрим ошибки десериализации. Они возникают, когда формат данных, используемый продюсером, не соответствует ожиданиям десериализатора консьюмера. Давайте разберёмся на конкретном примере:

Если консьюмер ожидает в качестве тела JSON, а мы отправляем что-то такое: `{ABC}`, то он, очевидно, не сможет десериализовать это в JSON, так как тело не соответствует JSON формату.

Если произойдёт такая ошибка, то консьюмер фактически перестанет работать. Он застрянет на этом сообщении навечно. Это происходит потому, что консьюмер будет пытаться обработать сообщение -> словит ошибку -> не закоммитит оффсет -> будет пытаться обработать сообщение -> ... И этот цикл не закончится. Так что обработка таких ошибок очень важна.

### Как обрабатывать ошибки десериализации?

Наверное, самое первое, что приходит на ум — сделать так, чтобы консьюмер мог пропускать те сообщения, которые ему не удалось десериализовать. То есть он просто скипнет его, переведя оффсет на единицу вперёд. И на самом деле такое поведение настроить можно. Это делается несложно. Нужно прописать следующее в `application.properties` файле консьюмера:

```
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
spring.kafka.consumer.properties.spring.deserializer.value.delegate.class=org.springframework.kafka.support.serializer.JsonDeserializer
```

Как вы можете видеть, мы поменяли десериализатор. Этот десериализатор по сути является некоторой обёрткой, которая позволит нашему консьюмеру пропускать сообщения, которые он не смог обработать. Однако надо указать, какому "настоящему" десериализатору он будет делегировать. В нашем случае это стандартный `JsonDeserializer`.

Отмечу, что весь код я пишу в нашем `inventory-service`.

### Чего-то не хватает...

Не кажется ли вам странным, что мы просто забиваем на сообщение?

Было бы неплохо всё-таки что-то извлечь из этого проблемного сообщения. Такой подход уже ближе к жизни, так как в сообщении может быть что-то важное.

Обычно для этих целей вводят специальный топик, который хранит сообщения, которые не смог обработать консьюмер. Этот паттерн называется **the dead letter topic pattern** или же **the dead letter queue pattern**.

### Применяем паттерн DLT

Для начала надо дать понять нашему консьюмеру, что мы от него хотим тогда, когда он не смог корректно обработать сообщение. Для этого надо добавить в контекст специальный бин. Он должен имплементировать интерфейс `CommonErrorHandler`.

Добавим:

```
@Beanpublic DefaultErrorHandler errorHandler(KafkaTemplate<String, Object> kafkaTemplate) {    DeadLetterPublishingRecoverer recoverer = new DeadLetterPublishingRecoverer(        kafkaTemplate,        (record, e) -> new TopicPartition(            record.topic() + "-dlt",            record.partition()        )    );    DefaultErrorHandler handler = new DefaultErrorHandler(recoverer, new FixedBackOff(3000, 3));    return handler;}
```

В качестве класса-имплементации `CommonErrorHandler` мы берём `DefaultErrorHandler`. В качестве параметра передаём `KafkaTemplate`, чтобы иметь возможность через него отправлять сообщения в DLT. Далее создаём объект типа `DeadLetterPublishingRecoverer`, который будет отвечать за логику отправления в DLT. В его конструктор передаём `KafkaTemplate` и лямбда-функцию, которая принимает проблемное сообщения и исключение, с которым оно упало. Эта лямбда-функция возвращает объект типа `TopicPartition`, который отвечает за то, в какой топик и в какую партицию отправить сообщение. Мы сохраняем название топика как префикс, добавляя `-dlt` в конец. Также мы сохраняем исходную партицию, что бывает полезно для отладки.

Далее давайте добавим 2 новых топика, чтобы консьюмеры нашего `inventory-service` отправляли в них "мёртвые сообщения":

```
@Beanpublic NewTopic inventoryReservedDLTopic() {    return TopicBuilder.name("inventory-reserved-dlt")        .partitions(3)        .replicas(3)        .config(TopicConfig.RETENTION_MS_CONFIG, "86400000")        .config(TopicConfig.RETENTION_BYTES_CONFIG, "524288000")        .build();}@Beanpublic NewTopic inventoryRejectedTopic() {    return TopicBuilder.name("inventory-rejected")        .partitions(3)        .replicas(3)        .config(TopicConfig.RETENTION_MS_CONFIG, "86400000")        .config(TopicConfig.RETENTION_BYTES_CONFIG, "524288000")        .build();}
```

Также прыгнем в наш `order-service` и добавим новый топик для того, чтобы `inventory-service` отправлял в него сообщения, которые он не смог обработать:

```
@Beanpublic NewTopic orderPlacedDLTopic() {    return TopicBuilder.name("order-placed-dlt")        .partitions(3)        .replicas(3)        .config(TopicConfig.RETENTION_MS_CONFIG, "86400000")        .config(TopicConfig.RETENTION_BYTES_CONFIG, "524288000")        .build();}
```

## Другие ошибки

Также в консьюмере могут возникнуть и другие ошибки. При их возникновении поведение консьюмера в основном зависит от вашего кода. Опять же, давайте на конкретном примере.

Далее вспомним часть кода сервиса нашего `inventory-service`:

```
@KafkaListener(topics = "order-placed")public void reserveInventory(OrderPlacedEvent orderPlacedEvent) {    transactionTemplate.executeWithoutResult(status -> {        processOrderInTransaction(orderPlacedEvent);    });}private void processOrderInTransaction(OrderPlacedEvent orderPlacedEvent) {    try {        processedOrderIdRepository.save(new ProcessedOrderId(            orderPlacedEvent.orderId()        ));    } catch (DataIntegrityViolationException e) {        logger.info("Order {} already processed", orderPlacedEvent.orderId());        return;    }    ...
```

Попрошу обратить внимание на 11 строчку. Здесь потенциально может произойти `NPE`. Если он действительно произойдёт, то наш консьюмер не сможет закоммитить оффсет, так как мы не дойдём до конца метода. На всякий случай напомню, что способ коммита в нашем сервисе — `record`_._ Таким образом, это тоже надо учесть.

Вообще, фактически мы это уже учли. Наш обработчик на самом деле срабатывает при любых исключениях, возникающих в листенере.

## Корректен ли наш обработчик на текущий момент?

Чисто в теории в консьюмере мы можем вызывать какой-то внешний сервис. Может быть такое, что при вызове этого сервиса произошёл таймаут. Было бы неплохо добавить ретраи, чтобы была возможность повторить вызов этого сервиса.

## Два типа ошибок

Мы можем разделить ошибки на _Retryable_ и _Non Retryable_. Для первого типа нужно настроить количество ретраев и длительность интервалов между ними. При этом Retryable сообщение после исчерпывания всех попыток всё равно попадёт в DLT топик.

Для этого можно создать 2 кастомных исключения:

```
public class RetryableException extends RuntimeException {    public RetryableException(String message) {        super(message);    }    public RetryableException(Throwable cause) {        super(cause);    }}
```

```
public class NonRetryableException extends RuntimeException {    public NonRetryableException(String message) {        super(message);    }    public NonRetryableException(Throwable cause) {        super(cause);    }}
```

Теперь можно ловить исключения с помощью _try-catch_ конструкции и оборачивать пойманные исключения в одно из вышеописанных исключений.

Отлично. Далее мы "зарегистрируем" эти исключения в конфиге. Также настроим параметры ретраев.

```
@Beanpublic DefaultErrorHandler errorHandler(KafkaTemplate<String, Object> kafkaTemplate) {    DeadLetterPublishingRecoverer recoverer = new DeadLetterPublishingRecoverer(        kafkaTemplate,        (record, e) -> new TopicPartition(            record.topic() + "-dlt",            record.partition()        )    );    DefaultErrorHandler handler = new DefaultErrorHandler(recoverer, new FixedBackOff(3000, 3));    handler.addNotRetryableExceptions(NonRetryableException.class);    handler.addRetryableExceptions(RetryableException.class);    return handler;}
```

Кстати, если вы не оборачиваете исключение в одно из написанных нами ранее, то оно будет считаться Retryable.

## Сервис, который будет обрабатывать сообщения из DLT

Давайте создадим простенький сервис, который будет консьюмером для наших dlt.

Начнём с базы:

![dlt-processor-service](https://habrastorage.org/r/w1560/getpro/habr/upload_files/8aa/482/f8b/8aa482f8b0bdb382b5b84a23afed47d7.png "dlt-processor-service")

dlt-processor-service

Далее сконфигурируем всё, что нужно через `application.properties` файл:

```
# Core application configuration
spring.application.name=dlt-processor-service
server.port=8084

# Kafka properties
spring.kafka.bootstrap-servers=localhost:29092,localhost:29093,localhost:29094
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.group-id=dltProcessorService
spring.kafka.consumer.auto-offset-reset=latest
spring.kafka.consumer.enable-auto-commit=false
spring.kafka.listener.ack-mode=record
```

Десериализуем всё в String по той причине, что Spring Boot через наш бин `DefaultErrorHandler` отправляет сообщения в DLT особым образом. Он не меняет ключи, добавляет хедеры, переводит тело в строчку формата BASE64. Поэтому и такой набор десериализаторов.

Далее напишем простенький сервис:

```
@Servicepublic class DltProcessorService {    private static final Logger logger = LoggerFactory.getLogger(DltProcessorService.class);    @KafkaListener(topicPattern = "^.*-dlt$")    public void processDeadLetters(String deadLetterBase64) {        String deadLetter = new String(Base64.getDecoder().decode(deadLetterBase64.substring(1, deadLetterBase64.length() - 1)));        logger.info("Dead Letter {} successfully processed", deadLetter);    }}
```

Наш listener будет читать все топики с суффиксом `-dlt`. Чтобы вручную не перечислять их все, предлагаю задать формат топика с помощью регулярного выражения.

Далее декодируем строку из BASE64 и логируем полученное сообщение.

## Про другие консьюмеры нашей платформы

Мы всё настроили для `inventory-service`. Остальные консьюмеры предлагаю вам настроить самим. И вы попрактикуетесь, и я не буду дублировать практически то же самое в статье. В любом случае я оставлю ссылку на репозиторий с кодом, который получился в ходе работы над статьёй, так что вы сможете свериться или подсмотреть, если где-то запнётесь.

## Проверяем работу на практике

Давайте теперь поднимем всю нашу инфраструктуру и попрактикуемся.

Зайдём в Kafka-UI и перейдём, например, в топик `order-placed`. Далее с помощью кнопки `Produce Message` произведём сообщение:

![производим некорректное сообщение](https://habrastorage.org/r/w1560/getpro/habr/upload_files/de8/331/b32/de8331b32d07bcf0b0438080e694e49f.png "производим некорректное сообщение")

производим некорректное сообщение

В теле лежит некорректный JSON. Соответственно, `inventory-service` отбросит сообщение в DLT. Давайте зайдём в соответствующий топик и проверим:

![сообщение, попавшее в dead letter topic](https://habrastorage.org/r/w1560/getpro/habr/upload_files/5bc/7aa/37e/5bc7aa37ef5659c37dcb9c411cbe0f1f.png "сообщение, попавшее в dead letter topic")

сообщение, попавшее в dead letter topic

Действительно, сообщение получено. Теперь давайте перейдём в логи консьюмера, который реагирует на сообщения в DLT. Вы должны увидеть следующее:  
`Dead Letter {ABC} successfully processed`

## Заключение

Поздравляю вас! Теперь у вас есть очень важный навык, который позволяет работать с ошибками в Spring Kafka.

Прикрепляю [ссылку](https://github.com/ndymko/kafka-article-6) на GitHub репозиторий с кодом, получившимся в ходе статьи.