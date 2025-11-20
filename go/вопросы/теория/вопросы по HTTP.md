## **Ответы на частые вопросы по HTTP на собеседовании**

### **1. "Чем отличается PUT от POST?"**

**Ответ:**
- **POST** используется для создания ресурса, когда клиент не знает ID. Обычно возвращает `201 Created`. Не идемпотентный.
- **PUT** используется для полного обновления/создания ресурса, когда клиент знает ID. Идемпотентный.

**Пример:**
```http
POST /api/users        # Создает нового пользователя
PUT /api/users/123     # Полностью обновляет пользователя с ID 123
```

### **2. "Какая разница между 401 и 403 статусами?"**

**Ответ:**
- **401 Unauthorized** - клиент не аутентифицирован (не предоставил credentials)
- **403 Forbidden** - клиент аутентифицирован, но не имеет прав на действие

**Пример:**
```http
401 - Не передан токен авторизации
403 - Пользователь пытается удалить чужой пост
```

### **3. "Что такое идемпотентные методы?"**

**Ответ:**
Идемпотентность означает, что многократное выполнение одного запроса дает тот же результат, что и однократное.

**Идемпотентные:** GET, PUT, DELETE, HEAD, OPTIONS
**Не идемпотентные:** POST, PATCH

**Пример:**
```go
// PUT /api/users/123 - сколько раз ни выполни, результат одинаков
// POST /api/users - каждый вызов создает нового пользователя
```

### **4. "Как работает CORS?"**

**Ответ:**
CORS (Cross-Origin Resource Sharing) - механизм для доступа к ресурсам с другого домена.

**Простой запрос:**
```http
GET /api/data
Origin: https://example.com
```

**Предварительный запрос (preflight):**
```http
OPTIONS /api/data
Origin: https://example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type

# Ответ сервера
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Content-Type
```

### **5. "Что такое Content-Type и какие основные типы знаешь?"**

**Ответ:**
Content-Type указывает тип передаваемых данных.

**Основные типы:**
- `application/json` - JSON данные
- `application/xml` - XML данные
- `application/x-www-form-urlencoded` - формы
- `multipart/form-data` - загрузка файлов
- `text/html` - HTML страницы
- `text/plain` - простой текст

### **6. "Как бы ты спроектировал REST API для системы блога?"**

**Ответ:**
```http
# Пользователи
GET    /api/users          # Список пользователей
POST   /api/users          # Регистрация
GET    /api/users/{id}     # Профиль пользователя
PUT    /api/users/{id}     # Обновление профиля

# Посты
GET    /api/posts          # Список постов с пагинацией
POST   /api/posts          # Создание поста (только авторизованные)
GET    /api/posts/{id}     # Получение поста
PUT    /api/posts/{id}     # Обновление (только автор)
DELETE /api/posts/{id}     # Удаление (только автор)

# Комментарии
GET    /api/posts/{id}/comments
POST   /api/posts/{id}/comments

# Пагинация и фильтрация
GET /api/posts?page=1&limit=10&category=tech
```

### **7. "Как реализовать пагинацию в API?"**

**Ответ:**
**Оффсетная пагинация:**
```http
GET /api/users?offset=0&limit=20
```
```json
{
  "data": [...],
  "pagination": {
    "offset": 0,
    "limit": 20,
    "total": 150,
    "has_next": true
  }
}
```

**Курсорная пагинация (лучше для больших данных):**
```http
GET /api/users?cursor=abc123&limit=20
```
```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "def456",
    "has_next": true
  }
}
```

### **8. "Какие методы кэширования ты использовал?"**

**Ответ:**
**Серверное кэширование:**
```go
// HTTP заголовки
w.Header().Set("Cache-Control", "public, max-age=3600")
w.Header().Set("ETag", "abc123")

// Условные запросы
if r.Header.Get("If-None-Match") == "abc123" {
    w.WriteHeader(304) // Not Modified
    return
}
```

**Кэширование в БД, Redis, CDN**

### **9. "Как защитить API от DDoS атак?"**

**Ответ:**
- **Rate Limiting** - ограничение запросов в единицу времени
- **API Gateway** с защитой от DDoS
- **CAPTCHA** для подозрительных запросов
- **Валидация входных данных**
- **Firewall** на уровне приложения

**Пример Rate Limiting:**
```go
func rateLimitMiddleware(next http.Handler) http.Handler {
    limiter := rate.NewLimiter(100, 30) // 100 запросов в минуту
    
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !limiter.Allow() {
            http.Error(w, "Rate limit exceeded", 429)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

### **10. "В чем разница между JWT и сессиями?"**

**Ответ:**
**Сессии:**
```go
// Сервер хранит состояние
session := store.Get(r, "session")
session.Values["user_id"] = userID
session.Save(r, w)
```
**Плюсы:** Простая инвалидация, безопасность
**Минусы:** Хранение состояния, масштабируемость

**JWT:**
```go
// Stateless - вся информация в токене
token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
tokenString, _ := token.SignedString(secretKey)
```
**Плюсы:** Stateless, масштабируемость, мобильность
**Минусы:** Сложная инвалидация, размер токена

### **Бонус: "Что такое HATEOAS?"**

**Ответ:**
HATEOAS (Hypermedia as the Engine of Application State) - клиент взаимодействует с API через гипермедиа, предоставляемое сервером.

**Пример:**
```json
{
  "user": {
    "id": 123,
    "name": "John",
    "_links": {
      "self": { "href": "/api/users/123" },
      "posts": { "href": "/api/users/123/posts" },
      "follow": { "href": "/api/users/123/follow" }
    }
  }
}
```

### **Дополнительные советы:**
- Приводите конкретные примеры из своего опыта
- Объясняйте trade-offs разных подходов
- Упоминайте security best practices
- Демонстрируйте понимание производительности

**Подготовка:** Практикуйтесь объяснять эти концепции простыми словами, как если бы объясняли коллеге.