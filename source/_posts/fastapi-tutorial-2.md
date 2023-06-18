---
title: fastapi-tutorial-2
date: 2023-06-18 06:14:48
tags: [fastapi, tech]
---


## Example Data

You can declare an `example` for a Pydantic model using `Config` and `schema_extra`, which will be displayed in docs. 

``` python
class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

    class Config:
        schema_extra = {
            "example": {
                "name": "Foo",
                "description": "A very nice Item",
                "price": 35.4,
                "tax": 3.2,
            }
        }
```

Multiple examples can also be added to each individual attribute in the class using keyword `examples`.

``` python
async def update_item(
    item_id: int,
    item: Annotated[
        Item,
        Body(
            examples={
                "normal": {
                    "summary": "A normal example",
                    "description": "A **normal** item works correctly.",
                    "value": {
                        "name": "Foo",
                        "description": "A very nice Item",
                    },
                },
                "converted": {
                    "summary": "An example with converted data",
                    "description": "FastAPI can convert price `strings` to actual `numbers`",
                    "value": {
                        "name": "Bar",
                        "price": "35.4",
                    }
                }
                ...
```


## Return Type

### Response model
`response_model` is used to define return type of a path operation. It is often preferred becuase it is more flexible than type annotation when dealing with dictionaries.

``` python
from typing import Any

@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item
```

### Redirect response

A way to realize `redirect` is by defining a reponse type.

``` python
from fastapi.responses import RedirectResponse

@app.get("/teleport", response_model=RedirectResponse)
async def get_teleport():
    return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```


## Dict

`**dict` will unwrap the values in a dictionary when passing into a function. The unwrapped output will be key-value arguments.

``` python
UserInDB(**user_dict)
```

## Response Status Code

`status_code` keyword can be used to generate a status code for the page.

``` python
@app.post("/items/", status_code=201)
```

A short cut for more detailed status is as following,

``` python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
```


## Handling Errors

Users can raise exceptions inside the path operations with customized headers.

``` python
from fastapi import HTTPException

items = {"foo": "The Foo Wrestlers"}

@app.get("/items-header/{item_id}")
async def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "There goes my error"},
        )
    return {"item": items[item_id]}
```

Or define customer exceptions and handlers.

``` python
class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name

@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )

@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

It is allowed to override default exception handlers.

``` python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)
```

## Tags & Summary

There are several keywords for configuring path operations.

``` python
@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    description="Create an item with all the information, name, description, price, tax and a set of unique tags",
    tags=["items"],
)
async def create_item(item: Item):
    return item
```

These attributes will be displayed on docs page. For `description`, there is an alternative way to write by markdown.

``` python
@app.post("/items/", response_model=Item, summary="Create an item", tags=["items"])
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```