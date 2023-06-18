---
title: fastapi-tutorial-4
date: 2023-06-18 10:31:27
tags: [fastapi, tech]
---

## Database

In this part, we will see how to use `FastAPI` to work with `SQLAlchemy ORM`. 

### database.py
First we will have the `database.py` to store basic parameters for the whole database.

``` python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./sql_app.db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()
```

Notice that `check_same_thread` is only set when `sqlite` is being used.


### models.py

Details of the data models (tables) in the database will be stored in `models.py`. These models are all `SQLAlchemy` models.

``` python
from sqlalchemy import Boolean, Column, ForeignKey, Integer, String
from sqlalchemy.orm import relationship

from .database import Base

class User(Base):
    __tablename__ = "users"
    ...
    items = relationship("Item", back_populates="owner")


class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String, index=True)
    owner_id = Column(Integer, ForeignKey("users.id"))

    owner = relationship("User", back_populates="items")
```

To relate the two tables, a `ForeignKey` needs to be defined in of the two tables. Then in each table, there needs to be a row defined by `relationship`.

### schemas.py

The file will store models that will be used in `FastAPI` files instead of the database side. These models are all `Pydantic` models.

``` python
from pydantic import BaseModel

class ItemBase(BaseModel):
    title: str
    description: Union[str, None] = None

class ItemCreate(ItemBase):
    pass

class Item(ItemBase):
    id: int
    owner_id: int

    class Config:
        orm_mode = True
```

`orm_mode` needs to be set to `True`. It will allow Pydantic model to read ORM models, enabling relationship between Pydantic models.

### crud.py

In this part, we will store resuable utility functions to interact the data in the database. CRUD comes from: Create, Read, Update, and Delete.

To read and create data,

``` python
from sqlalchemy.orm import Session
from . import models, schemas

def get_user_by_email(db: Session, email: str):
    return db.query(models.User).filter(models.User.email == email).first()

def create_user(db: Session, user: schemas.UserCreate):
    fake_hashed_password = user.password + "notreallyhashed"
    db_user = models.User(email=user.email, hashed_password=fake_hashed_password)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

`refresh` refreshes your instance (so that it contains any new data from the database, like the generated ID).


### main.py

We then combine everything inside `main.py`.

``` python
from sqlalchemy.orm import Session
from . import crud, models, schemas
from .database import SessionLocal, engine

models.Base.metadata.create_all(bind=engine)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.post("/users/", response_model=schemas.User)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    db_user = crud.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return crud.create_user(db=db, user=user)
```

`models.Base.metadata.create_all(bind=engine)` is used to create all the tables in the database. 

`get_db()` is used to create an independent database session/connection (SessionLocal) per request, use the same session through all the request and then close it after the request is finished.


## Run the database file structure

Then one can run the command in the top directory.

``` bash
$ uvicorn sqlapp.main:app --reload
```

Then the code will automatically generate a database file in the top directory.