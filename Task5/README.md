## 1. Анализ текущего REST API (из task5.yaml)

REST-контракт содержит следующие ресурсы и операции:

### Эндпоинты
- `GET /clients/{id}` → объект `Client`
- `GET /clients/{id}/documents` → список `[Document]`
- `GET /clients/{id}/relatives` → список `[Relative]`

### Модели данных
**Client**
- id: string  
- name: string  
- age: integer  

**Document**
- id: string  
- type: string  
- number: string  
- issueDate: string  
- expiryDate: string  

**Relative**
- id: string  
- relationType: string  
- name: string  
- age: integer  

Проблема: для формирования одной «карточки клиента» потребителю приходится делать несколько REST-запросов, что увеличивает RPS и задержки. Передать все данные одним большим объектом нельзя из-за объема (до ~500 атрибутов).

---

## 2. Предлагаемая GraphQL-схема (SDL)

GraphQL позволяет запрашивать только нужные поля и объединять связанные данные в одном запросе.

```graphql
schema {
  query: Query
}

type Query {
  client(id: ID!): Client
  clients(ids: [ID!]!): [Client!]!
}

type Client {
  id: ID!
  name: String
  age: Int

  documents: [Document!]!
  relatives: [Relative!]!
}

type Document {
  id: ID!
  type: String
  number: String
  issueDate: String
  expiryDate: String
}

type Relative {
  id: ID!
  relationType: String
  name: String
  age: Int
}
```
## 3. Маппинг REST → GraphQL

GET /clients/{id}
→ Query.client(id)
Поля id, name, age.

GET /clients/{id}/documents
→ Client.documents
Резолвер этого поля вызывает REST-эндпоинт.

GET /clients/{id}/relatives
→ Client.relatives
Аналогично вызывается REST-эндпоинт.

Таким образом, все операции REST полностью покрыты GraphQL.

## 4. Как GraphQL решает проблему
   Было (REST)- чтобы получить данные клиента + документы + родственников:

3 отдельных HTTP-запроса\
больше RPS\
больше сетевых задержек

Стало (GraphQL) - один запрос:
```
query GetClient($id: ID!) {
  client(id: $id) {
    id
    name
    age
    documents {
      id
      type
      number
    }
    relatives {
      id
      relationType
      name
    }
  }
}

```
