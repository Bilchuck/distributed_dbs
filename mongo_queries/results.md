# 1
## 1). Створіть декілька товарів з різним набором властивостей Phone/TV/Smart Watch
```
2:51:28 PM	Info: db.getCollection('items').save({
	"category" : "Phone",
	"model" : "iPhone 6",
	"producer" : "Apple",
	"price" : 600
})
```
```
db.getCollection('items').save([
    {
	"category" : "TV",
	"model" : "Samsung super 1",
	"producer" : "Samsung",
	"price" : 1600,
        "OS": "Web OS"
    },
    {
	"category" : "Phone",
	"model" : "Samsung Galaxy 13",
	"producer" : "Samsung",
	"price" : 900
    },
    {
	"category" : "TV",
	"model" : "LG model 1",
	"producer" : "LG",
	"price" : 1600,
        "OS": "Androind TV"
    },
    {
	"category" : "Phone",
	"model" : "Xiaomi mi 6",
	"producer" : "Xiaomi",
	"price" : 300
    },
    {
	"category" : "Smart Watch",
	"model" : "Apple Watch 3",
	"producer" : "Apple",
	"price" : 400
    },
    {
	"category" : "Smart Watch",
	"model" : "Samsung Watch",
	"producer" : "Samsung",
	"price" : 400
    },
    
])
```

## 2) Напишіть запит, який виводіть усі товари (відображення у JSON)
```
db.getCollection('items').find({})
```
result:
```
/* 1 */
{
    "_id" : ObjectId("5c7293507cd3913141d94d69"),
    "category" : "Phone",
    "model" : "iPhone 6",
    "producer" : "Apple",
    "price" : 600.0
}

/* 2 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6b"),
    "category" : "TV",
    "model" : "Samsung super 1",
    "producer" : "Samsung",
    "price" : 1600.0,
    "OS" : "Web OS"
}

/* 3 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6c"),
    "category" : "Phone",
    "model" : "Samsung Galaxy 13",
    "producer" : "Samsung",
    "price" : 900.0
}

/* 4 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6d"),
    "category" : "TV",
    "model" : "LG model 1",
    "producer" : "LG",
    "price" : 1600.0,
    "OS" : "Androind TV"
}

/* 5 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6e"),
    "category" : "Phone",
    "model" : "Xiaomi mi 6",
    "producer" : "Xiaomi",
    "price" : 300.0
}

/* 6 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6f"),
    "category" : "Smart Watch",
    "model" : "Apple Watch 3",
    "producer" : "Apple",
    "price" : 400.0
}

/* 7 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d70"),
    "category" : "Smart Watch",
    "model" : "Samsung Watch",
    "producer" : "Samsung",
    "price" : 400.0
}
```

## 3) Підрахуйте скільки товарів у певної категорії
db.getCollection('items').find({ category: "Smart Watch" }).count()
```
result: 2
```

## 4) Підрахуйте скільки є різних категорій товарів
```
db.getCollection('items').aggregate([{
  $group: {
    _id: "$category",
    count: { $sum: 1 } }
  }
])
```
```
/* 1 */
{
    "_id" : "TV",
    "count" : 2.0
}

/* 2 */
{
    "_id" : "Smart Watch",
    "count" : 2.0
}

/* 3 */
{
    "_id" : "Phone",
    "count" : 3.0
}
```

## 5)Напишіть запити, які вибирають товари за різними критеріям і їх сукупності: 
 ### a) категорія та ціна (в проміжку)
  ```
  db.getCollection('items').find({
    category: "Phone",
    price: {
      $gt: 200,
      $lt: 700
    }
  })
  ```
  ```
  /* 1 */
  {
      "_id" : ObjectId("5c7293507cd3913141d94d69"),
      "category" : "Phone",
      "model" : "iPhone 6",
      "producer" : "Apple",
      "price" : 600.0
  }

  /* 2 */
  {
      "_id" : ObjectId("5c72969b7cd3913141d94d6e"),
      "category" : "Phone",
      "model" : "Xiaomi mi 6",
      "producer" : "Xiaomi",
      "price" : 300.0
  }
  ```

  ### b) Розмір (наприклад розмір взуття або діагональ екрану) або модель,
  ```
  # add new items
  db.getCollection('items').save([
    { category: "shoe", model: "Adidas model 1", size: 12 },
    { category: "shoe", model: "Adidas model 1", size: 13 },
    { category: "shoe", model: "Reebok model 1", size: 12 },
    { category: "shoe", model: "Reebok model 1", size: 13 },
])
  ```
  ```
  db.getCollection('items').find({
    $or: [
        { model: "Adidas model 1" },
        { size: 12 }
    ]
  })
```
```
result:
/* 1 */
{
    "_id" : ObjectId("5c729daf7cd3913141d94d72"),
    "category" : "shoe",
    "model" : "Adidas model 1",
    "size" : 12.0
}

/* 2 */
{
    "_id" : ObjectId("5c729daf7cd3913141d94d73"),
    "category" : "shoe",
    "model" : "Adidas model 1",
    "size" : 13.0
}

/* 3 */
{
    "_id" : ObjectId("5c729daf7cd3913141d94d74"),
    "category" : "shoe",
    "model" : "Reebok model 1",
    "size" : 12.0
}
```

### c) конструкція з використанням in
```
db.getCollection('items').find({
  category: "Phone",
  producer: { $in: ["Samsung", "Xiaomi"] }
})
```
```
result:
/* 1 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6c"),
    "category" : "Phone",
    "model" : "Samsung Galaxy 13",
    "producer" : "Samsung",
    "price" : 900.0
}

/* 2 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6e"),
    "category" : "Phone",
    "model" : "Xiaomi mi 6",
    "producer" : "Xiaomi",
    "price" : 300.0
}
```
## 6) Виведіть список всіх виробників товарів без повторів
```
db.getCollection('items').distinct('producer')
```

```
/* 1 */
[
    "Apple",
    "Samsung",
    "LG",
    "Xiaomi"
]
```

## 7) Оновить певні товари, змінивши існуючі значення і додайте нові властивості (характеристики) товару за певним критерієм
```
db.getCollection('items').update({
  category: "Phone",
  producer: { $in: ["Samsung", "Xiaomi"] }
}, {
  $set: { OS: "Android" },
  $inc: { price: 100 }
}, { multi: true })
```

```
Result after update:
/* 1 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6c"),
    "category" : "Phone",
    "model" : "Samsung Galaxy 13",
    "producer" : "Samsung",
    "price" : 1000.0,
    "OS" : "Android"
}

/* 2 */
{
    "_id" : ObjectId("5c72969b7cd3913141d94d6e"),
    "category" : "Phone",
    "model" : "Xiaomi mi 6",
    "producer" : "Xiaomi",
    "price" : 400.0,
    "OS" : "Android"
}
```
## 8) Знайдіть товари у яких є (присутнє поле) певні властивості
```
db.getCollection('items').find({ size: { $exists: true } })
```
```
/* 1 */
{
    "_id" : ObjectId("5c729daf7cd3913141d94d72"),
    "category" : "shoe",
    "model" : "Adidas model 1",
    "size" : 12.0
}

/* 2 */
{
    "_id" : ObjectId("5c729daf7cd3913141d94d73"),
    "category" : "shoe",
    "model" : "Adidas model 1",
    "size" : 13.0
}

/* 3 */
{
    "_id" : ObjectId("5c729daf7cd3913141d94d74"),
    "category" : "shoe",
    "model" : "Reebok model 1",
    "size" : 12.0
}

/* 4 */
{
    "_id" : ObjectId("5c729daf7cd3913141d94d75"),
    "category" : "shoe",
    "model" : "Reebok model 1",
    "size" : 13.0
}
```

## 9) Для знайдених товарів збільшіть їх вартість на певну суму

```
db.getCollection('items').update(
    { producer: "Apple" },
    { $inc: { price: 200 }},
    { multi: true }
)
```

# 2 orders
## 1) Створіть кілька замовлень з різними наборами товарів, але так щоб один з товарів був у декількох замовленнях
```
db.getCollection('orders').save([
{    
	"order_number" : 201513,
	"date" : ISODate("2015-04-14"),
	"total_sum" : 1923.4,
	"customer" : {
    	"name" : "Andrii",
    	"surname" : "Rodinov",
    	"phones" : [ 9876543, 1234567],
    	"address" : "PTI, Peremohy 37, Kyiv, UA"
	},
	"payment" : {
    	"card_owner" : "Andrii Rodionov",
    	"cardId" : 12345678
	},
	"order_items_id" : [
    	{
        	"$ref" : "items",
        	"$id" : ObjectId("5c7293507cd3913141d94d69")
    	},
    	{
        	"$ref" : "items",
        	"$id" : ObjectId("5c72969b7cd3913141d94d6b")
    	}
	]
},
{    
	"order_number" : 201514,
	"date" : ISODate("2015-04-15"),
	"total_sum" : 1923.4,
	"customer" : {
    	"name" : "Anton",
    	"surname" : "Bilchuk",
    	"phones" : [ 9876543, 1234567],
    	"address" : "PTI, Peremohy 37, Kyiv, UA"
	},
	"payment" : {
    	"card_owner" : "Anton Bilchuk",
    	"cardId" : 12345678
	},
	"order_items_id" : [
    	{
        	"$ref" : "items",
        	"$id" : ObjectId("5c7293507cd3913141d94d69")
    	},
    	{
        	"$ref" : "items",
        	"$id" : ObjectId("5c72969b7cd3913141d94d6d")
    	}
	]
}


])
```
## 2) Виведіть всі замовлення
```
db.getCollection('orders').find({});
result: 
/* 1 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d77"),
    "order_number" : 201513.0,
    "date" : ISODate("2015-04-14T00:00:00.000Z"),
    "total_sum" : 1923.4,
    "customer" : {
        "name" : "Andrii",
        "surname" : "Rodinov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Andrii Rodionov",
        "cardId" : 12345678.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c7293507cd3913141d94d69")
        }, 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c72969b7cd3913141d94d6b")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d78"),
    "order_number" : 201514.0,
    "date" : ISODate("2015-04-15T00:00:00.000Z"),
    "total_sum" : 1923.4,
    "customer" : {
        "name" : "Anton",
        "surname" : "Bilchuk",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Anton Bilchuk",
        "cardId" : 12345678.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c7293507cd3913141d94d69")
        }, 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c72969b7cd3913141d94d6d")
        }
    ]
}
```
## 3) Виведіть замовлення з вартістю більше певного значення

```
db.getCollection('orders').find({ total_sum: { $gt: 1923 } })
```

```
/* 1 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d77"),
    "order_number" : 201513.0,
    "date" : ISODate("2015-04-14T00:00:00.000Z"),
    "total_sum" : 1923.4,
    "customer" : {
        "name" : "Andrii",
        "surname" : "Rodinov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Andrii Rodionov",
        "cardId" : 12345678.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c7293507cd3913141d94d69")
        }, 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c72969b7cd3913141d94d6b")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d78"),
    "order_number" : 201514.0,
    "date" : ISODate("2015-04-15T00:00:00.000Z"),
    "total_sum" : 1923.4,
    "customer" : {
        "name" : "Anton",
        "surname" : "Bilchuk",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Anton Bilchuk",
        "cardId" : 12345678.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c7293507cd3913141d94d69")
        }, 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c72969b7cd3913141d94d6d")
        }
    ]
}
```

## 4) Знайдіть замовлення зроблені одним замовником

Let's add more orders..

```
db.getCollection('orders').save([
{    
	"order_number" : 201513,
	"date" : ISODate("2015-04-14"),
	"total_sum" : 100,
	"customer" : {
    	"name" : "Andrii",
    	"surname" : "Rodinov",
    	"phones" : [ 9876543, 1234567],
    	"address" : "PTI, Peremohy 37, Kyiv, UA"
	},
	"payment" : {
    	"card_owner" : "Andrii Rodionov",
    	"cardId" : 12345678
	},
	"order_items_id" : [
    	{
        	"$ref" : "items",
        	"$id" : ObjectId("5c729daf7cd3913141d94d72")
    	}
	]
}
])
```

And look for Andrii's orders. I decided to query only by first and second name as we don;t have any customer/user id's.
```
db.getCollection('orders').find({
    "customer.name": 'Andrii',
    "customer.surname" : "Rodinov"
})
```
```
Result:
/* 1 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d77"),
    "order_number" : 201513.0,
    "date" : ISODate("2015-04-14T00:00:00.000Z"),
    "total_sum" : 1923.4,
    "customer" : {
        "name" : "Andrii",
        "surname" : "Rodinov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Andrii Rodionov",
        "cardId" : 12345678.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c7293507cd3913141d94d69")
        }, 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c72969b7cd3913141d94d6b")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5c72a4a27cd3913141d94d7a"),
    "order_number" : 201513.0,
    "date" : ISODate("2015-04-14T00:00:00.000Z"),
    "total_sum" : 100.0,
    "customer" : {
        "name" : "Andrii",
        "surname" : "Rodinov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Andrii Rodionov",
        "cardId" : 12345678.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c729daf7cd3913141d94d72")
        }
    ]
}
```
## 5) Знайдіть всі замовлення з певним товаром (товарами) (шукати можна по ObjectId)

Iphone object id = ObjectId("5c7293507cd3913141d94d69").
Ref of irder item should be "item". Maybe in future we would have another refs.
```
db.getCollection('orders').find({
    "order_items_id.$ref": "items",
   "order_items_id.$id": ObjectId("5c7293507cd3913141d94d69")
})
```
```
Result:
/* 1 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d77"),
    "order_number" : 201513.0,
    "date" : ISODate("2015-04-14T00:00:00.000Z"),
    "total_sum" : 1923.4,
    "customer" : {
        "name" : "Andrii",
        "surname" : "Rodinov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Andrii Rodionov",
        "cardId" : 12345678.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c7293507cd3913141d94d69")
        }, 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c72969b7cd3913141d94d6b")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d78"),
    "order_number" : 201514.0,
    "date" : ISODate("2015-04-15T00:00:00.000Z"),
    "total_sum" : 1923.4,
    "customer" : {
        "name" : "Anton",
        "surname" : "Bilchuk",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Anton Bilchuk",
        "cardId" : 12345678.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c7293507cd3913141d94d69")
        }, 
        {
            "$ref" : "items",
            "$id" : ObjectId("5c72969b7cd3913141d94d6d")
        }
    ]
}
```
## 6) Додайте в усі замовлення з певним товаром ще один товар і збільште існуючу вартість замовлення на деяке значення Х
Add to every order with iPhone(prev query) new item with Apple watch and increment total_sum by apple watch price
```
db.getCollection('orders').update({
    "order_items_id.$ref": "items",
   "order_items_id.$id": ObjectId("5c7293507cd3913141d94d69")
}, {
    $push: {
        "order_items_id": {
            "$ref" : "items",
            "$id": ObjectId("5c72969b7cd3913141d94d6f")
         }
    },
    $inc: {
        "total_sum": 600
    }
}, { multi: true })
```

## 7) Виведіть кількість товарів в певному замовленні
Maybe it can be done without aggregation, but it is the simpliest way for me.
ObjectId("5c72a3bc7cd3913141d94d77") - Order with iPhone, apple watch and TV
```
db.getCollection('orders').aggregate([
    { $match: { _id: ObjectId("5c72a3bc7cd3913141d94d77") }},
    { $project: { numberOfItems: { $size: "$order_items_id" } } }
])

Result:
/* 1 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d77"),
    "numberOfItems" : 3
}
```

## 8) Виведіть тільки інформацію про кастомера і номери кредитної карт, для замовлень вартість яких перевищує певну суму

```
db.getCollection('orders').aggregate([
    { $match: { total_sum: { $gt: 100 } }},
    { $project: { customer: "$customer", payment: "$payment" }}
])
```

```
/* 1 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d77"),
    "customer" : {
        "name" : "Andrii",
        "surname" : "Rodinov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Andrii Rodionov",
        "cardId" : 12345678.0
    }
}

/* 2 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d78"),
    "customer" : {
        "name" : "Anton",
        "surname" : "Bilchuk",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Anton Bilchuk",
        "cardId" : 12345678.0
    }
}
```

## 9) Видаліть товар з замовлень, зроблених за певний період дат
```
db.getCollection('orders').find({}).count()
3
--
db.getCollection('orders').remove({
  date: {
    $gt: ISODate("2015-04-14"),
    $lt: ISODate("2015-04-31")
  }
})
--
db.getCollection('orders').find({}).count()
2
```
## 10) Перейменуйте у всіх замовлення ім'я (прізвище) замовника

## 11) Знайдіть замовлення зроблені одним замовником, і виведіть тільки інформацію про кастомера та товари у замовлені підставивши замість ObjectId("***") назви товарів та їх вартість (аналог join-а між таблицями orders та items).
Змінив `$id` на `id`
```
db.getCollection('orders').aggregate([
    { $project: { customer: "$customer", order_items_id: "$order_items_id.id" } },
    { $lookup: {
        from: 'items',
        localField: 'order_items_id',
        foreignField: '_id',
        as: 'items'
       }
    }, {
      $project: {
        customer: 1,
        "items.model": 1,
        "items.price": 1
      }
    }
])
```

```
Result: 
/* 1 */
{
    "_id" : ObjectId("5c72a3bc7cd3913141d94d77"),
    "customer" : {
        "name" : "Andrii",
        "surname" : "Rodinov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "items" : [ 
        {
            "model" : "iPhone 6",
            "price" : 800.0
        }, 
        {
            "model" : "Samsung super 1",
            "price" : 1600.0
        }, 
        {
            "model" : "Apple Watch 3",
            "price" : 600.0
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5c72a4a27cd3913141d94d7a"),
    "customer" : {
        "name" : "Andrii",
        "surname" : "Rodinov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "PTI, Peremohy 37, Kyiv, UA"
    },
    "items" : [ 
        {
            "model" : "Adidas model 1"
        }
    ]
}
```