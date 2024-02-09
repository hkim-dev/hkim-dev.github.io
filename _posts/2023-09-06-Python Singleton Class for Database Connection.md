---
title: "Python Singleton Class for Database Connection"
date: 2023-09-06 08:26:28 -0400
categories: "design_pattern"
header:
  teaser: "/assets/images/singleton.png"
tags:
  - Python
# classes: wide
---
![source: https://www.javatpoint.com/singleton-design-pattern-in-python](/assets/images/singleton.png)

A lot of times, we find ourselves in a situation where we make connections to external services while coding in object-oriented languages. While I was not aware of such design pattern that enables the reuse of a single connection, (obviously) turned to Google, and came across a design pattern called **singleton**. 

Wikipedia’s definition of the singleton pattern is as follows:

> In software engineering, the **singleton pattern** is a software design pattern that restricts the instantiation of a class to a singular instance.
> 

It basically means that this software design pattern prevents the creation of multiple instances. Especially, for example, when we work with a database and need to perform operations such as write and delete, we can keep using the single connection object with this pattern. It also comes in handy for logging as it requires a single logger instance. 

# Implementation

To implement the singleton pattern for my use case, a static method was used to return the instance to be shared. 

```python
from pymongo import MongoClient

from .access_secrets_manager import get_secret_value

class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]

class MongoDBConnection(metaclass=Singleton):
    def __init__(self):
        self.client = self.create_mongo_client()

    @staticmethod
    def create_mongo_client():
        try:
            # retrieve secret values
            mongodb_name = get_secret_value('--------', 'user_name')
            mongodb_pwd = get_secret_value('--------', 'user_password')
            cluster_name = get_secret_value('--------', 'cluster_name')

            # construct MongoDB URI 
            mongodb_uri = f"mongodb+srv://{mongodb_name}:{mongodb_pwd}@{cluster_name}.fpxkpcs.mongodb.net/"

            # create and return MongoDB client
            return MongoClient(mongodb_uri)
        except Exception as e:
            print(f"Error creating MongoDB client: {e}")
            raise RuntimeError("Failed to create MongoDB client.") from e
```

The **`Singleton`** class is a metaclass, and it overrides the **`__call__`** method. This method is called when an instance of a class is created. The **`_instances`** class attribute is a dictionary that stores instances of classes that use this metaclass. It ensures that only one instance of each class exists.