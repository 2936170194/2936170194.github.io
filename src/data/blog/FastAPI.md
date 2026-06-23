---
title: FastAPI
author: 李杰
pubDatetime: 2026-06-14T00:00:00Z
featured: false
draft: false
description: FastAPI、掘金头条项目笔记
tags:
 - 后端
 - FastAPI
---

# FastAPI 复习笔记

这份笔记按照资料顺序提炼：先从 FastAPI 的基础程序、路由、请求与响应进入，再学习中间件、依赖注入和异步 ORM，最后落到“AI 掘金头条”项目的新闻、用户、收藏、浏览历史、缓存和模型调用。复习时不要先背 API 清单，而要抓住一条主线：**HTTP 请求进入应用后，被路由分发，被参数系统校验，被依赖系统补齐上下文，被 CRUD 访问数据库，最后通过统一响应或异常处理返回前端。**

## 首部记忆导图（Mermaid）

```mermaid
flowchart TB
    Start["FastAPI 抗遗忘总线：请求进来 -> 校验参数 -> 执行业务 -> 访问数据 -> 统一返回"]:::root

    Start --> Basic["1 基础：路由、参数、响应"]:::basic
    Basic --> Route["路由 = URL + HTTP 方法 + 处理函数"]:::basic
    Basic --> Params["参数三分法：Path 指唯一资源；Query 筛选集合；Body 创建/更新"]:::basic
    Params --> Pydantic["类型提示 + Pydantic：少写手动校验，多得自动文档"]:::basic
    Pydantic --> Response["response_model / 响应类：约束输出，隐藏多余字段"]:::basic

    Start --> Advanced["2 进阶：中间件、依赖注入、ORM"]:::advanced
    Advanced --> Middleware["Middleware：所有请求都经过，适合 CORS/日志/耗时统计"]:::advanced
    Advanced --> Depends["Depends：按需注入，适合 DB 会话/当前用户/公共参数"]:::advanced
    Depends --> Session["get_db：yield 会话 -> 成功 commit -> 异常 rollback"]:::advanced
    Session --> ORM["SQLAlchemy 异步 ORM：select/add/update/delete"]:::advanced

    Start --> News["3 新闻模块：项目分层与列表详情"]:::news
    News --> Router["APIRouter 分模块：main.py 只负责挂载"]:::news
    News --> CORS["CORS：协议/域名/端口不同就跨域，需要后端授权"]:::news
    News --> List["分类/列表：分页 + 总量 + hasMore"]:::news
    News --> Detail["详情：查新闻 -> 浏览量 +1 -> 同类相关新闻"]:::news

    Start --> User["4 用户模块：认证与统一边界"]:::user
    User --> Hash["passlib bcrypt：存哈希，不存明文"]:::user
    User --> Token["Token：解决 HTTP 无状态，后续请求放 Authorization"]:::user
    Token --> Current["get_current_user：Header -> token -> user"]:::user
    User --> Unified["success_response + 全局异常：成功和失败都统一格式"]:::user

    Start --> Project["5 收藏/历史/缓存/模型调用"]:::project
    Project --> Favorite["收藏：唯一约束防重复，联表取新闻"]:::project
    Project --> History["历史：重复浏览更新时间，列表按 view_time 倒序"]:::project
    Project --> Redis["Redis：Cache-Aside，先查缓存，未命中查库再写缓存"]:::project
    Project --> AI["模型调用：QWen 地址 + Key + 模型名，生产环境应走后端代理"]:::project

    Middleware -.对所有接口生效.-> Router
    Depends -.按路由声明生效.-> Current
    ORM -.支撑.-> List
    ORM -.支撑.-> Favorite
    Redis -.减轻数据库压力.-> List

    classDef root fill:#263238,stroke:#111,color:#fff,stroke-width:2px;
    classDef basic fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20;
    classDef advanced fill:#E3F2FD,stroke:#1565C0,color:#0D47A1;
    classDef news fill:#FFF3E0,stroke:#EF6C00,color:#E65100;
    classDef user fill:#F3E5F5,stroke:#7B1FA2,color:#4A148C;
    classDef project fill:#E0F2F1,stroke:#00796B,color:#004D40;
```

## 一、FastAPI 基础：先把请求和响应跑通

### 本部分记忆导图

```mermaid
flowchart LR
    A["基础入口：FastAPI 是快速构建 API 的 Python Web 框架"]:::root
    A --> B["异步优势：I/O 等待时释放执行权"]:::node
    A --> C["路由：装饰器把 URL 映射到函数"]:::node
    C --> D["Path 参数：资源身份，如 /book/{id}"]:::path
    C --> E["Query 参数：集合筛选/分页，如 ?page=1"]:::query
    C --> F["Body 参数：创建/更新资源，用 Pydantic 模型"]:::body
    D --> G["Path/Query/Field：补充必填、范围、长度、描述"]:::check
    E --> G
    F --> G
    G --> H["自动校验 + 自动 /docs 文档"]:::check
    H --> I["响应：默认 JSONResponse；特殊内容用响应类"]:::resp
    I --> J["response_model：输出白名单，也是安全边界"]:::resp
    I --> K["HTTPException：客户端错误用标准异常中断"]:::warn

    classDef root fill:#263238,stroke:#111,color:#fff;
    classDef node fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20;
    classDef path fill:#E3F2FD,stroke:#1565C0,color:#0D47A1;
    classDef query fill:#FFF3E0,stroke:#EF6C00,color:#E65100;
    classDef body fill:#F3E5F5,stroke:#7B1FA2,color:#4A148C;
    classDef check fill:#E0F2F1,stroke:#00796B,color:#004D40;
    classDef resp fill:#FCE4EC,stroke:#C2185B,color:#880E4F;
    classDef warn fill:#FFEBEE,stroke:#C62828,color:#B71C1C;
```

FastAPI 的核心价值可以记成三句话：**异步性能高、类型驱动开发效率高、自动生成交互式文档。**第一个程序通常只需要创建 `FastAPI()` 实例，定义路由，再用 `uvicorn main:app --reload` 启动；`--reload` 表示代码改动后自动重启，开发调试时很常用。

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "hello world"}
```

路由就是 URL 地址和处理函数之间的映射关系。`@app.get("/")` 表示接收 `GET /` 请求，`root()` 是真正执行业务的函数，返回的字典会被 FastAPI 自动转成 JSON。

> **易忘点：** `async def` 的优势主要出现在 I/O 密集场景，例如数据库、Redis、HTTP 调用、文件读写。不要在异步函数里直接写 `time.sleep()`、同步数据库驱动这类阻塞操作，否则事件循环会被卡住。

参数是 FastAPI 的第一道“理解请求”的入口。复习时按位置记，不要按装饰器死背：路径参数在 URL 路径里，查询参数在 `?` 后面，请求体在 HTTP body 中。

```python
from fastapi import FastAPI, Path, Query
from pydantic import BaseModel, Field

app = FastAPI()

@app.get("/book/{id}")
async def get_book(id: int = Path(..., ge=1, le=100)):
    return {"id": id, "title": f"这是第{id}本书"}

@app.get("/news/list")
async def get_news_list(
    page: int = Query(1, ge=1),
    page_size: int = Query(10, ge=1, le=100, alias="pageSize"),
):
    return {"page": page, "pageSize": page_size}

class BookCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=20)
    author: str = Field(..., min_length=2, max_length=10)
    publisher: str = Field("黑马出版社")
    price: float = Field(..., gt=0)

@app.post("/book")
async def create_book(book: BookCreate):
    return book
```

`Path`、`Query`、`Field` 的使用场景很清楚：`Path` 管路径参数，`Query` 管查询参数，`Field` 管 Pydantic 模型字段。它们都可以设置 `gt/ge/lt/le`、`min_length/max_length`、`description` 等校验信息。`...` 表示必填；有默认值则通常是可选。

> **易错点：** 函数参数如果没有出现在路径模板里，且不是 Pydantic 模型，FastAPI 通常会把它当作查询参数。例如 `/news/list` 中的 `page`、`page_size` 就来自 `?page=1&pageSize=10`。

响应方面，FastAPI 默认会把字典、列表、Pydantic 模型等 Python 对象通过 `jsonable_encoder` 转成 JSON 兼容格式，再包装成 `JSONResponse`。如果要返回 HTML、文件、纯文本、流式数据或重定向，就使用对应响应类。

```python
from fastapi.responses import HTMLResponse, FileResponse

@app.get("/html", response_class=HTMLResponse)
async def get_html():
    return "<h1>Hello World</h1>"

@app.get("/file")
async def get_file():
    return FileResponse("./files/1.jpeg")
```

`response_model` 是输出约束，不只是文档装饰。它会把返回数据限制在指定 Pydantic 模型内，多余字段不会出现在响应中，这对隐藏密码、内部状态、数据库冗余字段尤其重要。

```python
from pydantic import BaseModel

class NewsResponse(BaseModel):
    id: int
    title: str
    content: str

@app.get("/news/{id}", response_model=NewsResponse)
async def get_news(id: int):
    return {
        "id": id,
        "title": f"这是第{id}条新闻",
        "content": "新闻正文",
        "internal_flag": "不会出现在响应里",
    }
```

当错误是客户端引起的，例如资源不存在、登录失败、参数不合法，应主动抛出 `HTTPException`，让接口以标准 HTTP 状态码返回。

```python
from fastapi import HTTPException

@app.get("/news/{id}")
async def get_news(id: int):
    if id not in [1, 2, 3]:
        raise HTTPException(status_code=404, detail="当前 id 不存在")
    return {"id": id}
```

## 二、进阶机制：中间件、依赖注入和异步 ORM

### 本部分记忆导图

```mermaid
flowchart TB
    A["进阶主线：公共逻辑放到正确的位置"]:::root
    A --> B["Middleware：每个请求都经过"]:::mid
    B --> B1["适合：CORS、日志、耗时、响应头"]:::mid
    B --> B2["执行：请求前一次，响应前一次；多个中间件按注册栈执行"]:::warn

    A --> C["Depends：当前接口需要才注入"]:::dep
    C --> C1["公共参数：skip/limit/page"]:::dep
    C --> C2["数据库会话：AsyncSession"]:::dep
    C --> C3["安全认证：当前登录用户"]:::dep

    A --> D["ORM：对象关系映射"]:::orm
    D --> D1["配置：create_async_engine + async_sessionmaker"]:::orm
    D --> D2["模型：DeclarativeBase + Mapped + mapped_column"]:::orm
    D --> D3["建表：Base.metadata.create_all 或 SQL 初始化"]:::orm
    D --> D4["CRUD：select/add/赋值更新/delete"]:::orm

    D4 --> E["查询套路：select -> db.execute -> scalars/scalar"]:::query
    E --> E1["where 条件：== like in_ & | ~"]:::query
    E --> E2["func 聚合：count/avg/max/min/sum"]:::query
    E --> E3["分页：offset=(page-1)*size；limit=size"]:::query

    classDef root fill:#263238,stroke:#111,color:#fff;
    classDef mid fill:#E3F2FD,stroke:#1565C0,color:#0D47A1;
    classDef dep fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20;
    classDef orm fill:#FFF3E0,stroke:#EF6C00,color:#E65100;
    classDef query fill:#E0F2F1,stroke:#00796B,color:#004D40;
    classDef warn fill:#FFEBEE,stroke:#C62828,color:#B71C1C;
```

中间件适合处理“所有请求都要统一经历”的逻辑。它在请求进入路由处理函数之前执行一次，在响应返回客户端之前再执行一次。因此，日志、性能监控、跨域、统一响应头等天然适合放到中间件。

```python
@app.middleware("http")
async def timer_middleware(request, call_next):
    print("中间件开始处理")
    response = await call_next(request)
    print("中间件处理完成")
    return response
```

依赖注入解决的是“多个接口都要用，但不是每个请求都无条件需要”的逻辑。它的记忆公式是：**创建依赖项 -> 导入 Depends -> 在路由参数中声明依赖项。**

```python
from fastapi import Depends, Query

async def common_pagination(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=60),
):
    return {"skip": skip, "limit": limit}

@app.get("/news/list")
async def list_news(page = Depends(common_pagination)):
    return page
```

> **对比记忆：** 中间件像“门禁大厅”，所有请求都要经过；依赖注入像“按需领工具”，只有声明了 `Depends(...)` 的接口才会执行。

SQLAlchemy ORM 让 Python 对象和关系型数据库表建立映射。资料中使用的是异步版本：`sqlalchemy[asyncio]` + `aiomysql`。核心配置包括异步引擎、异步会话工厂和数据库会话依赖。

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

ASYNC_DATABASE_URL = "mysql+aiomysql://root:root@localhost:3306/news_app?charset=utf8mb4"

async_engine = create_async_engine(
    ASYNC_DATABASE_URL,
    echo=True,
    pool_size=10,
    max_overflow=20,
)

AsyncSessionLocal = async_sessionmaker(
    bind=async_engine,
    class_=AsyncSession,
    expire_on_commit=False,
)

async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

这段代码对应示例项目中的 `config/db_conf.py`。复习时重点记住 `yield session` 的位置：路由函数执行期间拿到同一个 `AsyncSession`，执行成功后提交，出异常则回滚并继续抛出。

> **重难点：** 如果 `get_db()` 已经统一负责提交事务，CRUD 层最好不要到处手动 `commit()`，否则事务边界会变得混乱。教学项目中有些函数会显式 `commit()`，复习时要知道这是可运行写法，但正式项目应统一事务归属。

模型类用 `DeclarativeBase` 作为基类，用 `Mapped[...]` 和 `mapped_column(...)` 描述字段映射。新闻项目中的 `News` 模型包含索引、外键和时间字段，适合记忆 ORM 模型写法。

```python
from datetime import datetime
from typing import Optional
from sqlalchemy import DateTime, ForeignKey, Index, Integer, String, Text
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
    created_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.now)
    updated_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.now, onupdate=datetime.now)

class News(Base):
    __tablename__ = "news"
    __table_args__ = (
        Index("fk_news_category_idx", "category_id"),
        Index("idx_publish_time", "publish_time"),
    )

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    title: Mapped[str] = mapped_column(String(255), nullable=False)
    description: Mapped[Optional[str]] = mapped_column(String(500))
    content: Mapped[str] = mapped_column(Text, nullable=False)
    image: Mapped[Optional[str]] = mapped_column(String(255))
    author: Mapped[Optional[str]] = mapped_column(String(50))
    category_id: Mapped[int] = mapped_column(Integer, ForeignKey("news_category.id"), nullable=False)
    views: Mapped[int] = mapped_column(Integer, default=0, nullable=False)
    publish_time: Mapped[datetime] = mapped_column(DateTime, default=datetime.now)
```

CRUD 记忆时按“查、新、改、删”四类即可。

```python
from sqlalchemy import func, select, update

# 查询列表
stmt = select(News).where(News.category_id == category_id).offset(skip).limit(limit)
result = await db.execute(stmt)
news_list = result.scalars().all()

# 聚合统计
stmt = select(func.count(News.id)).where(News.category_id == category_id)
total = (await db.execute(stmt)).scalar_one()

# 更新字段
stmt = update(News).where(News.id == news_id).values(views=News.views + 1)
result = await db.execute(stmt)
ok = result.rowcount > 0
```

`scalars().all()` 取多个 ORM 对象，`scalar_one_or_none()` 取一个或 `None`，`scalar()`/`scalar_one()` 常配合聚合查询。分页公式永远是：`offset = (page - 1) * page_size`。

## 三、AI 掘金头条新闻模块：从工程结构到列表详情

### 本部分记忆导图

```mermaid
flowchart TB
    A["新闻模块主线：项目分层让业务不挤在 main.py"]:::root
    A --> B["main.py：创建 app、注册异常、CORS、include_router"]:::main
    A --> C["routers：只处理 HTTP 入参和响应"]:::router
    A --> D["schemas：Pydantic 请求/响应模型"]:::schema
    A --> E["models：SQLAlchemy 表结构"]:::model
    A --> F["crud：封装数据库查询和更新"]:::crud
    A --> G["config/utils/cache：基础设施能力"]:::infra

    C --> H["分类接口：GET /api/news/categories"]:::case
    H --> H1["查缓存/查库 -> 返回分类"]:::case
    C --> I["列表接口：GET /api/news/list"]:::case
    I --> I1["categoryId + page + pageSize"]:::case
    I1 --> I2["offset/limit 查列表 + count 查总量 + hasMore"]:::case
    C --> J["详情接口：GET /api/news/detail"]:::case
    J --> J1["id 查详情 -> views +1 -> 同类相关新闻"]:::case

    B --> K["CORS：前端 localhost:5173 调后端 127.0.0.1:8000"]:::warn
    E --> L["索引：category_id/publish_time 提升筛选排序"]:::model

    classDef root fill:#263238,stroke:#111,color:#fff;
    classDef main fill:#E3F2FD,stroke:#1565C0,color:#0D47A1;
    classDef router fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20;
    classDef schema fill:#F3E5F5,stroke:#7B1FA2,color:#4A148C;
    classDef model fill:#FFF3E0,stroke:#EF6C00,color:#E65100;
    classDef crud fill:#E0F2F1,stroke:#00796B,color:#004D40;
    classDef infra fill:#FCE4EC,stroke:#C2185B,color:#880E4F;
    classDef case fill:#E1F5FE,stroke:#0277BD,color:#01579B;
    classDef warn fill:#FFEBEE,stroke:#C62828,color:#B71C1C;
```

“AI 掘金头条”后端采用典型分层结构：`routers` 放接口，`crud` 放数据库操作，`models` 放 ORM 模型，`schemas` 放 Pydantic 模型，`config` 放数据库和缓存配置，`utils` 放响应、认证、异常、加密等通用能力。这样做的直接好处是：**main.py 不堆业务代码，路由只处理 HTTP 边界，数据库细节被 CRUD 封装。**

示例项目入口大致如下：

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from routers import news, users, favorite, history
from utils.exception_handlers import register_exception_handlers

app = FastAPI()

register_exception_handlers(app)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(news.router)
app.include_router(users.router)
app.include_router(favorite.router)
app.include_router(history.router)
```

模块化路由的写法可以记成两步：每个模块自己创建 `APIRouter`，然后在 `main.py` 中 `include_router()`。

```python
from fastapi import APIRouter

router = APIRouter(prefix="/api/news", tags=["news"])

@router.get("/categories")
async def get_categories():
    return {"msg": "获取分类成功"}
```

CORS 是新闻项目接前端时必须理解的点。浏览器判断同源看三个条件：协议、域名、端口。前端常跑在 `http://localhost:5173/`，后端跑在 `http://127.0.0.1:8000/`，域名和端口都不同，所以浏览器会按跨域请求处理。后端通过 `CORSMiddleware` 告诉浏览器“这个来源允许访问”。

> **易错点：** 教学阶段常用 `allow_origins=["*"]` 放开所有来源。生产环境不要这样配置，尤其在允许携带 Cookie/凭证时，应明确写出可信前端域名。

新闻分类接口的流程很简单：路由接收 `skip`、`limit`，通过 `Depends(get_db)` 拿数据库会话，调用 CRUD 查询分类，返回统一结构。项目中分类接口已经接入缓存，因此实际调用的是 `news_cache.get_categories(...)`。

```python
@router.get("/categories")
async def get_categories(
    skip: int = 0,
    limit: int = 100,
    db: AsyncSession = Depends(get_db),
):
    categories = await news_cache.get_categories(db, skip, limit)
    return {
        "code": 200,
        "message": "获取分类成功",
        "data": categories,
    }
```

新闻列表接口是分页接口的标准模板。它从前端接收 `categoryId`、`page`、`pageSize`，换算成 `offset`，再分别查询列表和总量，最后计算 `hasMore`。

```python
@router.get("/list")
async def get_news_list(
    category_id: int = Query(..., alias="categoryId"),
    page: int = Query(1, gt=0),
    page_size: int = Query(10, alias="pageSize"),
    db: AsyncSession = Depends(get_db),
):
    offset = (page - 1) * page_size
    news_list = await news_cache.get_news_list(db, category_id, offset, page_size)
    total = await news.get_news_count(db, category_id)
    has_more = total > page_size + offset
    return {"code": 200, "message": "获取新闻列表成功", "data": {
        "list": news_list,
        "total": total,
        "hasMore": has_more,
    }}
```

> **重难点：** 分页响应里 `list` 和 `total` 不是同一个查询得到的。`list` 受 `offset/limit` 限制，`total` 是当前筛选条件下的全部数量；`hasMore` 才能据此判断是否还有下一页。

新闻详情接口包含三个动作：按 id 查询新闻，浏览量加 1，查询同分类的相关新闻。这个流程比“查详情并返回”多一步副作用，因此要检查更新是否命中数据。

```python
@router.get("/detail")
async def get_news_detail(
    news_id: int = Query(..., alias="id"),
    db: AsyncSession = Depends(get_db),
):
    news_detail = await news.get_news_detail(db, news_id)
    if not news_detail:
        raise HTTPException(status_code=404, detail="新闻不存在")

    views_res = await news.increase_news_views(db, news_id)
    if not views_res:
        raise HTTPException(status_code=404, detail="新闻不存在")

    related_news = await news.get_related_news(
        db, news_detail.id, news_detail.category_id
    )
    return {"code": 200, "message": "success", "data": {
        "id": news_detail.id,
        "title": news_detail.title,
        "content": news_detail.content,
        "image": news_detail.image,
        "author": news_detail.author,
        "publishTime": news_detail.publish_time,
        "categoryId": news_detail.category_id,
        "views": news_detail.views,
        "relatedNews": related_news,
    }}
```

相关新闻的查询按同分类、排除当前新闻、浏览量降序、发布时间降序来取前几条：

```python
stmt = (
    select(News)
    .where(News.category_id == category_id, News.id != news_id)
    .order_by(News.views.desc(), News.publish_time.desc())
    .limit(5)
)
```

## 四、用户模块：注册登录、Token、统一响应和异常

### 本部分记忆导图

```mermaid
flowchart TB
    A["用户模块主线：身份从注册登录开始，在每次请求中被重新证明"]:::root
    A --> B["注册：查用户名 -> 哈希密码 -> 创建用户 -> 生成 Token"]:::auth
    A --> C["登录：查用户 -> verify 明文/哈希 -> 生成 Token"]:::auth
    B --> D["passlib bcrypt：hash / verify"]:::security
    C --> D
    B --> E["UserToken：token + expires_at"]:::token
    C --> E
    E --> F["Authorization: Bearer <token>"]:::token
    F --> G["get_current_user：Header -> token 表 -> user 表"]:::dep
    G --> H["受保护接口：info/update/password/favorite/history"]:::dep

    A --> I["统一成功响应：code/message/data"]:::resp
    I --> I1["jsonable_encoder：ORM/Pydantic 转 JSON 安全结构"]:::resp
    A --> J["全局异常处理：HTTPException/IntegrityError/SQLAlchemyError/Exception"]:::except
    J --> J1["顺序：具体异常在前，兜底异常在后"]:::except
    A --> K["Pydantic alias：后端 snake_case，前端 camelCase"]:::schema

    classDef root fill:#263238,stroke:#111,color:#fff;
    classDef auth fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20;
    classDef security fill:#FFF3E0,stroke:#EF6C00,color:#E65100;
    classDef token fill:#E3F2FD,stroke:#1565C0,color:#0D47A1;
    classDef dep fill:#E0F2F1,stroke:#00796B,color:#004D40;
    classDef resp fill:#F3E5F5,stroke:#7B1FA2,color:#4A148C;
    classDef except fill:#FFEBEE,stroke:#C62828,color:#B71C1C;
    classDef schema fill:#FCE4EC,stroke:#C2185B,color:#880E4F;
```

用户模块最核心的问题是：HTTP 是无状态的，服务器不会天然记得“谁已经登录”。所以登录或注册成功后，后端生成 Token 返回给前端；前端保存 Token，之后每次访问需要登录的接口时，把 Token 放在请求头中。

```http
Authorization: Bearer <token>
```

密码不能明文存储。项目使用 `passlib` 的 bcrypt 上下文完成哈希和校验。

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def get_hash_password(password: str):
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str):
    return pwd_context.verify(plain_password, hashed_password)
```

注册流程是：先按用户名查用户，如果已存在则抛出 409；否则哈希密码、创建用户、生成 Token、返回用户信息。

```python
@router.post("/register")
async def register(user_data: UserRequest, db: AsyncSession = Depends(get_db)):
    existing_user = await users.get_user_by_username(db, user_data.username)
    if existing_user:
        raise HTTPException(status_code=409, detail="用户已存在")

    user = await users.create_user(db, user_data)
    token = await users.create_token(db, user.id)
    response_data = UserAuthResponse(
        token=token,
        user_info=UserInfoResponse.model_validate(user),
    )
    return success_response(message="注册成功", data=response_data)
```

登录流程类似，但它不创建用户，而是先认证用户名和密码。认证失败返回 `None`，路由层再转成 401。

```python
async def authenticate_user(db: AsyncSession, username: str, password: str):
    user = await get_user_by_username(db, username)
    if not user:
        return None
    if not security.verify_password(password, user.password):
        return None
    return user
```

Token 的生成在项目中使用 `uuid.uuid4()`，并写入 `user_token` 表，过期时间设置为 7 天。如果用户已有 Token，则更新旧 Token；否则新增一条。

```python
async def create_token(db: AsyncSession, user_id: int):
    token = str(uuid.uuid4())
    expires_at = datetime.now() + timedelta(days=7)

    query = select(UserToken).where(UserToken.user_id == user_id)
    result = await db.execute(query)
    user_token = result.scalar_one_or_none()

    if user_token:
        user_token.token = token
        user_token.expires_at = expires_at
    else:
        db.add(UserToken(user_id=user_id, token=token, expires_at=expires_at))

    await db.flush()
    return token
```

> **易忘点：** Token 本身不是用户对象，只是“身份凭证”。真正确认当前用户时，仍要用 Token 查 `user_token` 表，确认未过期，再查 `user` 表。

项目把“当前登录用户”封装成依赖项。这样任何需要登录的接口，只要声明 `user: User = Depends(get_current_user)`，就能拿到当前用户对象。

```python
from fastapi import Header, Depends, HTTPException

async def get_current_user(
    authorization: str = Header(..., alias="Authorization"),
    db: AsyncSession = Depends(get_db),
):
    token = authorization.replace("Bearer ", "")
    user = await users.get_user_by_token(db, token)
    if not user:
        raise HTTPException(status_code=401, detail="无效的或者过期的令牌")
    return user
```

> **易错点：** 请求头字段名是 `Authorization`，而不是普通查询参数。真实项目里还应更严格检查 `Bearer ` 前缀，避免传入格式异常时被误处理。

统一成功响应封装在 `utils/response.py` 中。它的价值是让前端始终拿到 `{code, message, data}`，并用 `jsonable_encoder` 处理 ORM、Pydantic、日期等 JSON 不直接支持的对象。

```python
from fastapi.encoders import jsonable_encoder
from fastapi.responses import JSONResponse

def success_response(message: str = "success", data=None):
    content = {"code": 200, "message": message, "data": data}
    return JSONResponse(content=jsonable_encoder(content))
```

Pydantic 模型还负责“前后端命名风格转换”。Python 后端常用 `snake_case`，前端接口常用 `camelCase`，可以用 `Field(alias=...)` 和 `ConfigDict(populate_by_name=True)` 兼容。

```python
from pydantic import BaseModel, ConfigDict, Field

class UserAuthResponse(BaseModel):
    token: str
    user_info: UserInfoResponse = Field(..., alias="userInfo")

    model_config = ConfigDict(
        populate_by_name=True,
        from_attributes=True,
    )
```

全局异常处理器用于把业务异常、数据库约束异常、数据库异常和兜底异常统一返回给前端。注册时要从具体到抽象：`HTTPException`、`IntegrityError`、`SQLAlchemyError`、`Exception`。

```python
def register_exception_handlers(app):
    app.add_exception_handler(HTTPException, http_exception_handler)
    app.add_exception_handler(IntegrityError, integrity_error_handler)
    app.add_exception_handler(SQLAlchemyError, sqlalchemy_error_handler)
    app.add_exception_handler(Exception, general_exception_handler)
```

> **重难点：** 异常处理器的注册顺序体现“子类在前、父类在后”。如果过早用 `Exception` 兜底，具体异常就失去精确处理机会。

用户信息修改和修改密码都依赖当前用户。修改资料时，项目用 `model_dump(exclude_unset=True, exclude_none=True)`，只更新前端真正传来的非空字段；修改密码时先校验旧密码，再把新密码哈希后保存。

```python
query = (
    update(User)
    .where(User.username == username)
    .values(**user_data.model_dump(exclude_unset=True, exclude_none=True))
)
```

## 五、收藏、浏览历史、缓存和模型调用：把项目做完整

### 本部分记忆导图

```mermaid
flowchart TB
    A["项目后半段主线：用户行为数据 + 性能优化 + AI 能力"]:::root
    A --> B["收藏模块：用户主动保存新闻"]:::fav
    B --> B1["check：查 Favorite(user_id, news_id)"]:::fav
    B --> B2["add：新增收藏；唯一约束防重复"]:::fav
    B --> B3["remove/clear：delete 后检查 rowcount"]:::fav
    B --> B4["list：Favorite join News，带 favorite_time"]:::fav

    A --> C["浏览历史：用户访问行为轨迹"]:::his
    C --> C1["add：已有则更新时间，未有则新增"]:::his
    C --> C2["list：History join News，按 view_time 倒序"]:::his
    C --> C3["delete/clear：只删除当前用户数据"]:::his

    A --> D["Redis 缓存：读多写少数据加速"]:::cache
    D --> D1["Key：news:categories / news:list:{category}:{page}:{size}"]:::cache
    D --> D2["Value：JSON 字符串"]:::cache
    D --> D3["Cache-Aside：先缓存，未命中查库并写缓存"]:::cache
    D --> D4["TTL：分类更久，列表/详情更短"]:::cache

    A --> E["QWen 模型调用：前端配置 API 地址、Key、模型"]:::ai
    E --> E1["学习阶段可前端直连；生产环境应避免暴露 Key"]:::warn

    B2 --> F["数据库约束兜底：UniqueConstraint(user_id, news_id)"]:::warn
    C1 --> G["历史不是简单重复插入，而是刷新最近浏览时间"]:::warn
    D3 --> H["写操作后要更新或删除缓存，否则读到旧数据"]:::warn

    classDef root fill:#263238,stroke:#111,color:#fff;
    classDef fav fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20;
    classDef his fill:#E3F2FD,stroke:#1565C0,color:#0D47A1;
    classDef cache fill:#FFF3E0,stroke:#EF6C00,color:#E65100;
    classDef ai fill:#F3E5F5,stroke:#7B1FA2,color:#4A148C;
    classDef warn fill:#FFEBEE,stroke:#C62828,color:#B71C1C;
```

收藏模块和浏览历史模块都依赖登录用户，所以路由层都声明了 `user: User = Depends(get_current_user)`。两者区别在业务语义：收藏是用户主动保存，不能重复；历史是用户访问行为，重复访问应刷新时间。

收藏表用唯一约束保证同一个用户不能重复收藏同一条新闻：

```python
from sqlalchemy import UniqueConstraint, Index

class Favorite(Base):
    __tablename__ = "favorite"
    __table_args__ = (
        UniqueConstraint("user_id", "news_id", name="user_news_unique"),
        Index("fk_favorite_user_idx", "user_id"),
        Index("fk_favorite_news_idx", "news_id"),
    )
```

检查收藏状态只需要查 `Favorite` 表是否存在记录。

```python
async def is_news_favorite(db: AsyncSession, user_id: int, news_id: int):
    query = select(Favorite).where(
        Favorite.user_id == user_id,
        Favorite.news_id == news_id,
    )
    result = await db.execute(query)
    return result.scalar_one_or_none() is not None
```

收藏列表需要展示新闻详情，所以使用联表查询：主体是 `News`，再连接 `Favorite`，同时把收藏时间和收藏 id 取出来。

```python
query = (
    select(
        News,
        Favorite.created_at.label("favorite_time"),
        Favorite.id.label("favorite_id"),
    )
    .join(Favorite, Favorite.news_id == News.id)
    .where(Favorite.user_id == user_id)
    .order_by(Favorite.created_at.desc())
    .offset((page - 1) * page_size)
    .limit(page_size)
)
rows = (await db.execute(query)).all()
```

> **易错点：** `select(News, Favorite.created_at.label(...))` 返回的不是单个 `News` 对象列表，而是包含多个元素的行结果。路由层需要按 `(news, favorite_time, favorite_id)` 解包。

浏览历史的新增逻辑更像“存在则更新，不存在则插入”。这能让历史列表按最近浏览排序，而不是一个用户看同一新闻十次就插入十条重复记录。

```python
async def add_history(db: AsyncSession, user_id: int, news_id: int):
    query = select(History).where(
        History.user_id == user_id,
        History.news_id == news_id,
    )
    result = await db.execute(query)
    history = result.scalar_one_or_none()

    if history:
        history.view_time = datetime.now()
        await db.flush()
        return history

    history = History(user_id=user_id, news_id=news_id)
    db.add(history)
    await db.flush()
    return history
```

历史列表和收藏列表结构相似，只是排序字段变成 `History.view_time.desc()`，响应字段变成 `historyId`、`viewTime`。注意项目路由为删除单条历史定义了 `/delete/{history_id}`，但 CRUD 函数参数命名为 `news_id` 并按 `History.news_id == news_id` 删除。复习时要明确接口语义：如果路径叫 `history_id`，删除条件应优先匹配历史记录 id；如果传的是新闻 id，就应在接口文档和变量名上保持一致。

Redis 缓存用于解决高频读取带来的数据库压力。项目使用 `redis.asyncio` 创建客户端，并封装 `get_cache`、`get_json_cache`、`set_cache`。

```python
import json
import redis.asyncio as redis

redis_client = redis.Redis(
    host="localhost",
    port=6379,
    db=0,
    decode_responses=True,
    protocol=2,
)

async def get_json_cache(key: str):
    data = await redis_client.get(key)
    if data:
        return json.loads(data)
    return None

async def set_cache(key: str, value, expire: int = 3600):
    if isinstance(value, (dict, list)):
        value = json.dumps(value, ensure_ascii=False)
    await redis_client.setex(key, expire, value)
    return True
```

资料中重点强调旁路缓存策略，也就是 Cache-Aside：读请求先查缓存，有数据直接返回；没有数据再查数据库，并把结果写入缓存。写请求则应该更新数据库后删除或更新对应缓存。

```python
async def get_categories(db: AsyncSession, skip: int = 0, limit: int = 100):
    cached_categories = await get_categories_cache()
    if cached_categories:
        return cached_categories

    stmt = select(Category).offset(skip).limit(limit)
    result = await db.execute(stmt)
    categories = result.scalars().all()

    if categories:
        categories = jsonable_encoder(categories)
        await set_categories_cache(categories)
    return categories
```

项目为新闻列表设计的缓存 Key 是：

```python
NEW_LIST_PREFIX = "news:list:"
CATEGORIES_KEY = "news:categories"

key = f"news:list:{category_id}:{page}:{size}"
```

缓存时间要按数据变化频率设计：分类、配置类数据更稳定，可以缓存更久；列表、详情变化更频繁，应缓存更短；验证码等安全数据要非常短。可以记成一句话：**越稳定，TTL 越长；越敏感，TTL 越短。**

> **重难点：** 缓存是临时高速数据，数据库才是权威数据。只加“读缓存”还不够，任何修改新闻、收藏、历史等写操作后，都要考虑是否需要删除或刷新相关缓存，否则前端会读到旧数据。

项目中的新闻列表缓存还有一个很容易忘的细节：从 Redis 取回的是 JSON 字典，不是 SQLAlchemy ORM 对象。代码先用 `NewsItemBase` 把 ORM 转成适合缓存的字典，读取缓存时再构造为 `News(**item)` 供后续代码使用。

```python
news_data = [
    NewsItemBase.model_validate(item).model_dump(mode="json", by_alias=False)
    for item in news_list
]
await set_cached_news_list(category_id, page, limit, news_data)
```

最后是模型调用。资料中把 QWen 调用放在前端配置里：进入前端项目的 `src/config/api.js`，替换大模型 HTTP 请求地址、API Key 和模型名称。学习阶段这样做方便联调，但生产环境不要把 API Key 直接暴露给浏览器，推荐由后端提供代理接口，再由后端安全地读取环境变量调用模型。

## 复习收束：一条接口从前端到数据库再回前端

把所有内容串起来，可以用新闻收藏列表作为完整链路：

1. 前端请求 `GET /api/favorite/list?page=1&pageSize=10`，请求头带 `Authorization: Bearer <token>`。
2. 路由层通过 `Query` 校验分页参数，通过 `Depends(get_current_user)` 校验登录态，通过 `Depends(get_db)` 获取数据库会话。
3. 认证依赖从 Header 取 Token，查 `user_token` 是否存在且未过期，再查 `user` 表得到当前用户。
4. CRUD 层先统计收藏总数，再通过 `Favorite join News` 查询当前用户收藏的新闻列表。
5. Schema 层用 Pydantic alias 把 `favorite_time`、`has_more` 等字段转成前端需要的 `favoriteTime`、`hasMore`。
6. `success_response` 统一包装成 `{code, message, data}`，并用 `jsonable_encoder` 转成 JSON 安全结构。
7. 如果中途出现未登录、资源不存在、唯一约束冲突、数据库异常，就由 `HTTPException` 或全局异常处理器返回统一错误格式。

> **最终记忆锚点：** FastAPI 项目不是“写很多装饰器”，而是把职责放对位置：路由处理 HTTP，Schema 校验数据，Depends 注入上下文，CRUD 管数据库，Utils 管统一能力，Cache 减轻读压力，异常处理守住失败边界。
