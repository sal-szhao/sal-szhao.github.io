---
title: fastapi-tutorial-1
date: 2023-06-15 21:07:16
tags: [fastapi, tech]
---

These series of blogs will introduce basic usage of `FastAPI`. The official tutorial is given as this [site](https://fastapi.tiangolo.com/tutorial/).

## Startings

The first part will be a simple demo of a working site using FastAPI.

``` python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

To see the site, run the following code in the terminal.

``` bash
$ uvicorn main:app --reload
``` 

An alternative way is to add the code to `main.py`.

``` python
import uvicorn

if __name__ == "__main__":
    uvicorn.run(app, host="127.0.0.1", port=8000)
```

Then the file can be run directly using the code.

``` bash
python main.py
```

The web page can be visited at http://127.0.0.1:8000. A useful local debug method is to visit http://127.0.0.1:8000/docs, which has the documentation for all the routes. 


## Path Parameters

### Basics
Path parameters are sent in by variables in URL. One can specify the type of path parameters by `{user_id: str}`.

``` python
@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}

@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

The *path operations* are evaluated in order. Switching the order of the two path operaions will result in not being able to visit `/users/me`. Similary, one cannot redefine a path operation because the first one will always be used.


### Predefined values

It is possible to use Python `enumerations` to serve as the type for the path parameters. For the enumeration class, focus on the new values assigned to the original parameters.

``` python
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    lenet = "lenet"

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}
```

## Query Parameters

### Basics
Path and query parameters can be combined with type specified.

``` python
from typing import Union

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None, short: bool = False, needy: str):
    ...
```

The query parameters will be demonstrated in URL by 

```
http://127.0.0.1:8000/items/foo?q=apple&short=True
```

To set the default value as `None`, use `Union[str, None] = None`. To set other default values, one can assign the default value directly. If the default value is set, the corresponding parameter will be an `optional` one. In this case, `q` and `short` are optional parameters and `needy` is a required parameter.

### Query parameter list
Another thing to notice is the query parameter list.

``` python
from typing import List

async def read_items(q: Annotated[Union[List[str], None], Query()] = None):
    ...
```

Or simply

``` python
async def read_items(q: Annotated[list, Query()] = []):
    ...
```


Then visiting the following website and pass the query parameter list.

```
http://localhost:8000/items/?q=foo&q=bar
```

## Request Body Parameters

The data model can be implemented as a class that inherits from `Pydantic BaseModel`.


``` python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

The attributes in the data model can be directly called like `item.price`. The pydantic model can be converted to dictionary by `dict()` and use `update()` to update the values in the dictionary.


## Validations

### String validation

Additional validation requirements can be added to the query parameters by `Query()`. One can even add `regex` as a validation. `title`, `description` and `deprecated` can be seen in `docs`. `alias` will demonstate the alias name in the URL. Using `Annotated` is preferred when it comes to default values.

``` python
from fastapi import Query
from typing_extensions import Annotated

@app.get("/items/")
async def read_items(
    q: Annotated[
        Union[str, None], 
        Query(
            title="Query string",
            description="Query string for the items to search in the database",
            max_length=50, 
            regex="^fixedquery$",
            alias="item-query",
            deprecated=True
        )
    ] = None
):
    ...
```

### Numeric validation
For path parameters, the validations can be added in a similar way. Number validations can be added as following.

+ `gt`: greater than
+ `le`: less than or equal

``` python
@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=1)]
):
  ...
```

Like query and path parameters, users can add `Body()` to a parameter to declare it as a body parameter.

``` python
@app.put("/items/{item_id}")
async def update_item(
    item_id: int, importance: Annotated[int, Body(gt=0)]
):
```