# Template FastAPI

Template base para criação de APIs em **FastAPI** seguindo um padrão modular, reutilizável e escalável para projetos com SQLAlchemy, Pydantic, CRUD abstrato, separação por camadas e rotas versionadas.

Este template foi pensado para manter o mesmo padrão usado nos projetos internos: separação clara entre `models`, `schemas`, `crud`, `api`, `core` e `db`, com suporte a reaproveitamento de lógica através de classes genéricas como `CRUDBase`.

---

## 1. Objetivo do template

Este projeto serve como ponto de partida para APIs FastAPI com:

- organização por módulos;
- rotas versionadas em `/api/v1`;
- conexão com banco usando SQLAlchemy;
- sessões assíncronas com `AsyncSession`;
- schemas Pydantic para entrada e saída de dados;
- CRUD abstrato reutilizável;
- endpoints separados por responsabilidade;
- configuração centralizada;
- suporte a filtros dinâmicos;
- suporte a relacionamentos em consultas;
- suporte a paginação, ordenação e agregações;
- facilidade para criação de novos módulos.

---

## 2. Estrutura do projeto

A estrutura identificada no template é semelhante a esta:

```text
TemplateFastAPI/
├── api/
│   ├── __init__.py
│   ├── deps.py
│   └── api_v1/
│       ├── __init__.py
│       ├── api.py
│       └── endpoints/
│           ├── __init__.py
│           └── cars.py
│
├── core/
│   ├── __init__.py
│   ├── example-config.py
│   ├── filters.py
│   ├── request.py
│   └── xml_render.py
│
├── crud/
│   ├── __init__.py
│   ├── baseAsync.py
│   ├── baseSync.py
│   └── crud_cars.py
│
├── db/
│   ├── __init__.py
│   ├── base.py
│   ├── base_class.py
│   └── session.py
│
├── models/
│   ├── __init__.py
│   └── car_model.py
│
├── schemas/
│   ├── __init__.py
│   └── car_schema.py
│
├── main.py
├── requirements.txt
├── utils.py
├── teste.py
└── README.md
```

---

## 3. Responsabilidade de cada pasta

### `main.py`

Arquivo principal da aplicação FastAPI.

Normalmente é responsável por:

- criar a instância do `FastAPI`;
- configurar CORS;
- incluir as rotas principais;
- definir eventos de inicialização, se necessário;
- expor a aplicação para o Uvicorn/IIS/Docker.

Exemplo conceitual:

```python
from fastapi import FastAPI
from api.api_v1.api import api_router

app = FastAPI(title="Template FastAPI")

app.include_router(api_router, prefix="/api/v1")
```

---

### `api/`

Camada responsável pelas rotas da API.

Ela não deve conter regra de negócio pesada. A função principal dos endpoints é:

1. receber a requisição;
2. validar dados com Pydantic;
3. obter dependências, como banco e API key;
4. chamar o CRUD ou uma camada core/service;
5. retornar a resposta.

---

### `api/deps.py`

Arquivo de dependências globais.

Aqui normalmente ficam funções como:

- `get_db`;
- validação de API key;
- validação de usuário autenticado;
- obtenção de sessão assíncrona;
- dependências reutilizadas nos endpoints.

Exemplo:

```python
from typing import AsyncGenerator
from sqlalchemy.ext.asyncio import AsyncSession
from db.session import async_session

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session() as session:
        yield session
```

---

### `api/api_v1/api.py`

Arquivo centralizador das rotas da versão 1 da API.

Sempre que um novo endpoint for criado dentro de `api/api_v1/endpoints/`, ele deve ser registrado aqui.

Exemplo:

```python
from fastapi import APIRouter
from api.api_v1.endpoints import cars

api_router = APIRouter()
api_router.include_router(cars.router, prefix="/cars", tags=["Cars"])
```

---

### `api/api_v1/endpoints/`

Pasta onde ficam os endpoints da aplicação.

Cada arquivo deve representar um domínio ou recurso.

Exemplos:

```text
endpoints/
├── cars.py
├── service_orders.py
├── travels.py
├── clients.py
└── tracking.py
```

Padrão recomendado para endpoint:

```python
from typing import Any, List
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession

from api import deps
from schemas.car_schema import CarCreate, CarUpdate, CarResponse
from crud.crud_cars import crud_car

router = APIRouter()


@router.post("/", response_model=CarResponse)
async def create_car(
    *,
    db: AsyncSession = Depends(deps.get_db),
    payload: CarCreate,
) -> Any:
    """
    Cria um novo carro.
    """
    return await crud_car.create(db, obj_in=payload)


@router.get("/", response_model=List[CarResponse])
async def list_cars(
    db: AsyncSession = Depends(deps.get_db),
) -> Any:
    """
    Lista carros cadastrados.
    """
    return await crud_car.get_multi(db, order_by="id")
```

---

### `core/`

Pasta para funcionalidades de apoio, configuração e serviços compartilhados.

No template existem arquivos como:

```text
core/
├── example-config.py
├── filters.py
├── request.py
└── xml_render.py
```

Uso sugerido:

- `config.py`: configurações do sistema;
- `filters.py`: helpers para filtros e períodos;
- `request.py`: cliente HTTP reutilizável;
- `xml_render.py`: funções para trabalhar com XML;
- services/cores específicos podem ser adicionados aqui quando a regra de negócio crescer.

---

### `core/example-config.py`

Arquivo de exemplo para configuração do projeto.

Recomendação: copiar este arquivo para `core/config.py` e ajustar as variáveis conforme o ambiente.

Exemplo:

```bash
cp core/example-config.py core/config.py
```

Em ambiente Windows PowerShell:

```powershell
Copy-Item core/example-config.py core/config.py
```

Exemplo esperado de configuração:

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    PROJECT_NAME: str = "Template FastAPI"
    API_V1_STR: str = "/api/v1"
    SQLALCHEMY_DATABASE_URI: str
    API_KEY: str | None = None

    class Config:
        env_file = ".env"

settings = Settings()
```

---

### `core/request.py`

Arquivo indicado para centralizar requisições HTTP externas.

Esse padrão evita repetir `httpx`, headers, timeout e tratamento de erro em vários pontos da aplicação.

Exemplo de uso recomendado:

```python
from core.request import APIClient

client = APIClient(
    base_url="https://api.externa.com.br",
    apikey="TOKEN",
    api_key_header_name="x-api-key",
)

response = await client.request(
    method="POST",
    endpoint="/orders",
    data={"order_number": "123"},
)
```

---

### `crud/`

Camada de acesso ao banco.

Ela concentra operações de persistência, consulta, filtros e atualizações.

Arquivos identificados no template:

```text
crud/
├── baseAsync.py
├── baseSync.py
└── crud_cars.py
```

Recomendação atual: usar o CRUD assíncrono como padrão principal em novos módulos.

---

## 4. CRUD abstrato recomendado

O template possui `baseAsync.py`, mas a versão consolidada recomendada é a versão definitiva do `CRUDBase`, que junta funcionalidades extras das versões anteriores.

Recursos recomendados no `CRUDBase` definitivo:

- `get`;
- `get_multi`;
- `get_multi_filter`;
- `get_multi_filters`;
- `get_multi_dynamic_filters`;
- `get_first_by_filters`;
- `get_last_by_filters`;
- `get_aggregates`;
- `create`;
- `create_multi`;
- `update`;
- `update_multi`;
- `remove`;
- filtros por relacionamento;
- filtros em JSON/JSONB;
- filtros agrupados com `and`, `or`, `not`;
- operadores extras;
- paginação;
- ordenação;
- `distinct_on_id`;
- retry em commit com `UniqueViolationError`.

---

## 5. Exemplo de CRUD específico

Para cada model, crie um arquivo específico em `crud/`.

Exemplo: `crud/crud_cars.py`

```python
from crud.baseAsync import CRUDBase
from models.car_model import Car
from schemas.car_schema import CarCreate, CarUpdate


class CRUDCar(CRUDBase[Car, CarCreate, CarUpdate]):
    pass


crud_car = CRUDCar(Car)
```

Quando precisar adicionar métodos específicos daquele domínio, crie dentro da classe:

```python
class CRUDCar(CRUDBase[Car, CarCreate, CarUpdate]):
    async def get_by_plate(self, db, *, plate: str):
        return await self.get_last_by_filters(
            db,
            filters={
                "plate": {"operator": "==", "value": plate}
            }
        )
```

---

## 6. Models

Os models ficam em `models/` e devem herdar de `Base`, definida em `db/base_class.py`.

Exemplo: `models/car_model.py`

```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime, func
from db.base_class import Base


class Car(Base):
    __tablename__ = "cars"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, nullable=False)
    plate = Column(String, nullable=True, index=True)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
```

Padrões recomendados:

- usar nomes de tabela no plural ou padrão do projeto;
- sempre criar `id` como chave primária;
- usar `nullable=False` em campos obrigatórios;
- criar `index=True` em campos usados para busca;
- evitar regra de negócio dentro do model;
- declarar relacionamentos quando necessário.

---

## 7. Schemas

Os schemas ficam em `schemas/` e devem representar entrada e saída da API.

Exemplo: `schemas/car_schema.py`

```python
from typing import Optional
from pydantic import BaseModel, ConfigDict


class CarBase(BaseModel):
    name: Optional[str] = None
    plate: Optional[str] = None
    is_active: Optional[bool] = True


class CarCreate(CarBase):
    name: str


class CarUpdate(CarBase):
    pass


class CarResponse(CarBase):
    id: int

    model_config = ConfigDict(from_attributes=True)
```

Padrões recomendados:

- `Base`: campos comuns;
- `Create`: campos obrigatórios para criação;
- `Update`: campos opcionais para alteração;
- `Response`: campos retornados pela API;
- usar `ConfigDict(from_attributes=True)` no Pydantic v2.

---

## 8. Banco de dados

A pasta `db/` centraliza a configuração do banco.

Arquivos principais:

```text
db/
├── base.py
├── base_class.py
└── session.py
```

### `db/base_class.py`

Define a base declarativa do SQLAlchemy.

Exemplo:

```python
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass
```

Ou, em versões mais antigas:

```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

---

### `db/base.py`

Este arquivo deve importar todos os models para que o SQLAlchemy/Alembic consiga reconhecê-los.

Exemplo:

```python
from db.base_class import Base
from models.car_model import Car
from models.service_order_model import ServiceOrder
from models.travel_model import Travel
```

Sempre que criar um novo model, importe ele em `db/base.py`.

---

### `db/session.py`

Arquivo responsável por criar engine e session.

Exemplo assíncrono:

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker
from core.config import settings

engine = create_async_engine(
    settings.SQLALCHEMY_DATABASE_URI,
    pool_pre_ping=True,
)

async_session = async_sessionmaker(
    bind=engine,
    autoflush=False,
    autocommit=False,
    expire_on_commit=False,
)
```

Exemplo de URI PostgreSQL async:

```env
SQLALCHEMY_DATABASE_URI=postgresql+asyncpg://usuario:senha@localhost:5432/minha_base
```

Exemplo de URI SQL Server com ODBC:

```env
SQLALCHEMY_DATABASE_URI=mssql+pyodbc://usuario:senha@servidor/banco?driver=ODBC+Driver+17+for+SQL+Server
```

---

## 9. Instalação do projeto

### 9.1. Criar ambiente virtual

Windows PowerShell:

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

Linux:

```bash
python -m venv venv
source venv/bin/activate
```

---

### 9.2. Instalar dependências

```bash
pip install -r requirements.txt
```

Recomendação: em projetos novos, usar Python 3.11 ou 3.12 para evitar conflitos com bibliotecas que ainda não suportam versões muito recentes.

---

### 9.3. Configurar variáveis de ambiente

Crie o arquivo `.env` na raiz do projeto:

```env
PROJECT_NAME=Template FastAPI
API_V1_STR=/api/v1
SQLALCHEMY_DATABASE_URI=postgresql+asyncpg://usuario:senha@localhost:5432/minha_base
API_KEY=sua-chave-interna
BACKEND_CORS_ORIGINS=["*"]
```

Caso o projeto use `core/example-config.py`, copie para `core/config.py` e ajuste os dados.

---

### 9.4. Rodar o projeto localmente

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

Acesse:

```text
http://127.0.0.1:8000/docs
```

ou:

```text
http://127.0.0.1:8000/redoc
```

---

## 10. Criando um novo módulo completo

Exemplo: criar módulo `service_orders`.

### Passo 1: criar o model

Arquivo: `models/service_order_model.py`

```python
from sqlalchemy import Column, Integer, String, DateTime, func
from sqlalchemy.dialects.postgresql import JSONB
from db.base_class import Base


class ServiceOrder(Base):
    __tablename__ = "service_orders"

    id = Column(Integer, primary_key=True, index=True)
    order_number = Column(String, nullable=False, index=True)
    external_order_number = Column(String, nullable=True, index=True)
    status = Column(String, nullable=False, default="PENDENTE")
    extra_information = Column(JSONB, nullable=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
```

---

### Passo 2: importar o model no `db/base.py`

```python
from models.service_order_model import ServiceOrder
```

---

### Passo 3: criar schemas

Arquivo: `schemas/service_order_schema.py`

```python
from typing import Any, Dict, Optional
from pydantic import BaseModel, ConfigDict


class ServiceOrderBase(BaseModel):
    order_number: Optional[str] = None
    external_order_number: Optional[str] = None
    status: Optional[str] = "PENDENTE"
    extra_information: Optional[Dict[str, Any]] = None


class ServiceOrderCreate(ServiceOrderBase):
    order_number: str


class ServiceOrderUpdate(ServiceOrderBase):
    pass


class ServiceOrderResponse(ServiceOrderBase):
    id: int

    model_config = ConfigDict(from_attributes=True)
```

---

### Passo 4: criar CRUD específico

Arquivo: `crud/crud_service_orders.py`

```python
from crud.baseAsync import CRUDBase
from models.service_order_model import ServiceOrder
from schemas.service_order_schema import ServiceOrderCreate, ServiceOrderUpdate


class CRUDServiceOrder(CRUDBase[ServiceOrder, ServiceOrderCreate, ServiceOrderUpdate]):
    async def get_by_order_number(self, db, *, order_number: str):
        return await self.get_last_by_filters(
            db,
            filters={
                "order_number": {"operator": "==", "value": order_number}
            }
        )


crud_service_order = CRUDServiceOrder(ServiceOrder)
```

---

### Passo 5: criar endpoint

Arquivo: `api/api_v1/endpoints/service_orders.py`

```python
from typing import Any, List
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.ext.asyncio import AsyncSession

from api import deps
from crud.crud_service_orders import crud_service_order
from schemas.service_order_schema import (
    ServiceOrderCreate,
    ServiceOrderUpdate,
    ServiceOrderResponse,
)

router = APIRouter()


@router.post("/", response_model=ServiceOrderResponse)
async def create_service_order(
    *,
    db: AsyncSession = Depends(deps.get_db),
    payload: ServiceOrderCreate,
) -> Any:
    return await crud_service_order.create(db, obj_in=payload)


@router.get("/", response_model=List[ServiceOrderResponse])
async def list_service_orders(
    db: AsyncSession = Depends(deps.get_db),
    limit: int = Query(100, ge=1, le=500),
    offset: int = Query(0, ge=0),
) -> Any:
    return await crud_service_order.get_multi_filters(
        db,
        filters=[],
        order_by="id",
        order_desc=True,
        limit=limit,
        offset=offset,
    )


@router.get("/{id}", response_model=ServiceOrderResponse)
async def get_service_order(
    *,
    db: AsyncSession = Depends(deps.get_db),
    id: int,
) -> Any:
    obj = await crud_service_order.get(db, id=id)
    if not obj:
        raise HTTPException(status_code=404, detail="Ordem de serviço não encontrada.")
    return obj


@router.put("/{id}", response_model=ServiceOrderResponse)
async def update_service_order(
    *,
    db: AsyncSession = Depends(deps.get_db),
    id: int,
    payload: ServiceOrderUpdate,
) -> Any:
    db_obj = await crud_service_order.get(db, id=id)
    if not db_obj:
        raise HTTPException(status_code=404, detail="Ordem de serviço não encontrada.")
    return await crud_service_order.update(db, db_obj=db_obj, obj_in=payload)


@router.delete("/{id}")
async def delete_service_order(
    *,
    db: AsyncSession = Depends(deps.get_db),
    id: int,
) -> Any:
    obj = await crud_service_order.remove(db, id=id)
    if not obj:
        raise HTTPException(status_code=404, detail="Ordem de serviço não encontrada.")
    return {"msg": "Registro removido com sucesso."}
```

---

### Passo 6: registrar rota no `api/api_v1/api.py`

```python
from fastapi import APIRouter
from api.api_v1.endpoints import service_orders

api_router = APIRouter()
api_router.include_router(
    service_orders.router,
    prefix="/service-orders",
    tags=["Service Orders"],
)
```

---

## 11. Usando filtros dinâmicos

O CRUD abstrato permite filtros com uma lista de dicionários.

Exemplo simples:

```python
orders = await crud_service_order.get_multi_filters(
    db,
    filters=[
        {"field": "status", "operator": "==", "value": "PENDENTE"},
        {"field": "order_number", "operator": "ilike", "value": "030"},
    ],
    order_by="id",
    order_desc=True,
    limit=100,
)
```

---

## 12. Operadores disponíveis

Operadores recomendados no CRUD definitivo:

| Operador | Descrição | Exemplo |
|---|---|---|
| `=` | Igual | `status = PENDENTE` |
| `==` | Igual | `status == PENDENTE` |
| `!=` | Diferente | `status != CANCELADO` |
| `>` | Maior que | `id > 10` |
| `>=` | Maior ou igual | `id >= 10` |
| `<` | Menor que | `id < 100` |
| `<=` | Menor ou igual | `id <= 100` |
| `like` | Busca textual case-sensitive | `%ABC%` |
| `ilike` | Busca textual case-insensitive | `%cielo%` |
| `not_like` | Não contém texto | `%erro%` |
| `not_ilike` | Não contém texto ignorando caixa | `%erro%` |
| `contains` | Contém valor | depende do tipo da coluna |
| `startswith` | Começa com | `030` |
| `endswith` | Termina com | `999` |
| `in` | Está em lista | `[1, 2, 3]` |
| `notin` | Não está em lista | `[4, 5]` |
| `between` | Entre dois valores | `[data_ini, data_fim]` |
| `not_between` | Fora do intervalo | `[data_ini, data_fim]` |
| `is_null` | É nulo | `None` |
| `is_not_null` | Não é nulo | `None` |
| `is_true` | Booleano verdadeiro | `True` |
| `is_false` | Booleano falso | `False` |

---

## 13. Filtros agrupados com AND, OR e NOT

Use `get_multi_dynamic_filters` quando precisar montar filtros mais complexos.

Exemplo:

```python
orders = await crud_service_order.get_multi_dynamic_filters(
    db,
    filters=[
        {"field": "status", "operator": "!=", "value": "CANCELADO"},
        {
            "logic": "or",
            "conditions": [
                {"field": "order_number", "operator": "ilike", "value": "030"},
                {"field": "external_order_number", "operator": "ilike", "value": "CLR"},
            ],
        },
    ],
    order_by="created_at",
    order_direction="desc",
    force_order_id=True,
    limit=100,
)
```

Esse exemplo representa:

```text
status != 'CANCELADO'
AND (
    order_number ILIKE '%030%'
    OR external_order_number ILIKE '%CLR%'
)
```

---

## 14. Filtros por relacionamento

O CRUD permite acessar campos relacionados usando ponto.

Exemplo:

```python
orders = await crud_service_order.get_multi_filters(
    db,
    filters=[
        {"field": "client.name", "operator": "ilike", "value": "CIELO"},
        {"field": "order_type.name", "operator": "==", "value": "REDESPACHO"},
    ],
    order_by="client.name",
)
```

Para funcionar, os relacionamentos precisam estar definidos no model SQLAlchemy.

Exemplo:

```python
class ServiceOrder(Base):
    __tablename__ = "service_orders"

    id = Column(Integer, primary_key=True)
    client_id = Column(Integer, ForeignKey("clients.id"))

    client = relationship("Client")
```

---

## 15. Filtros em JSON/JSONB

Para campos JSON/JSONB, use caminho com ponto.

Exemplo:

```python
orders = await crud_service_order.get_multi_filters(
    db,
    filters=[
        {
            "field": "extra_information.delivery_id",
            "operator": "==",
            "value": "abc-123",
        },
        {
            "field": "extra_information.end_customer.federal_tax_payer_id",
            "operator": "==",
            "value": "12345678000199",
        },
    ],
)
```

Esse padrão é útil para projetos que usam `extra_information` como campo flexível.

---

## 16. Ordenação e paginação

Exemplo com `limit` e `offset`:

```python
items = await crud_service_order.get_multi_filters(
    db,
    filters=[],
    order_by="id",
    order_desc=True,
    limit=50,
    offset=0,
)
```

Exemplo com ordenação dinâmica:

```python
items = await crud_service_order.get_multi_dynamic_filters(
    db,
    filters=[],
    order_by="created_at",
    order_direction="desc",
    force_order_id=True,
    limit=50,
    offset=0,
)
```

`force_order_id=True` adiciona o `id` como critério de desempate, útil quando vários registros possuem a mesma data.

---

## 17. Agregações

Use `get_aggregates` para montar consultas de soma, contagem, média, mínimo e máximo.

Exemplo:

```python
result = await crud_service_order.get_aggregates(
    db,
    filters=[
        {"field": "status", "operator": "!=", "value": "CANCELADO"},
    ],
    aggregations=[
        {"op": "count", "field": "id", "alias": "total_orders"},
    ],
    group_by=["status"],
)
```

Retorno esperado:

```json
[
  {"status": "PENDENTE", "total_orders": 10},
  {"status": "FINALIZADO", "total_orders": 25}
]
```

Exemplo com soma de campo dentro de JSON:

```python
result = await crud_service_order.get_aggregates(
    db,
    filters=[],
    aggregations=[
        {
            "op": "sum",
            "field": "extra_information.valor_total",
            "alias": "valor_total",
            "is_json": True,
        }
    ],
)
```

---

## 18. Padrão de resposta e tratamento de erro

Nos endpoints, mantenha mensagens claras e padronizadas.

Exemplo:

```python
obj = await crud_service_order.get(db, id=id)

if not obj:
    raise HTTPException(
        status_code=404,
        detail="Registro não encontrado."
    )
```

Para erros de validação de regra de negócio:

```python
raise HTTPException(
    status_code=400,
    detail="Não é possível finalizar uma viagem já finalizada."
)
```

Para conflitos:

```python
raise HTTPException(
    status_code=409,
    detail="Já existe um registro com esses dados."
)
```

---

## 19. Padrão para services/core de regra de negócio

Quando a lógica começar a ficar grande, evite colocar tudo no endpoint.

Crie uma classe em `core/` ou em uma pasta `services/`.

Exemplo:

```text
core/
└── service_order_core.py
```

```python
from fastapi import HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from crud.crud_service_orders import crud_service_order
from schemas.service_order_schema import ServiceOrderCreate


class ServiceOrderCore:
    async def create_order(self, db: AsyncSession, payload: ServiceOrderCreate):
        existing = await crud_service_order.get_by_order_number(
            db,
            order_number=payload.order_number,
        )

        if existing:
            raise HTTPException(
                status_code=409,
                detail="Ordem de serviço já cadastrada."
            )

        return await crud_service_order.create(db, obj_in=payload)
```

Endpoint usando core:

```python
@router.post("/")
async def create_service_order(
    *,
    db: AsyncSession = Depends(deps.get_db),
    payload: ServiceOrderCreate,
):
    core = ServiceOrderCore()
    return await core.create_order(db, payload)
```

---

## 20. Padrão para consumo de APIs externas

Use uma classe centralizada para requisições externas.

Exemplo:

```python
from core.request import APIClient

api_client = APIClient(
    base_url="https://api.parceiro.com.br",
    apikey="TOKEN",
    api_key_header_name="x-api-key",
    timeout=30,
)

response = await api_client.request(
    method="POST",
    endpoint="/v1/status",
    data={
        "order_number": "123",
        "status": "confirmed",
    },
)
```

Padrões recomendados:

- nunca repetir `httpx` diretamente em cada endpoint;
- centralizar headers;
- centralizar timeout;
- logar payload enviado em erro;
- retornar mensagem clara quando a API externa falhar;
- usar `jsonable_encoder` para serializar `datetime`, `Decimal` e schemas Pydantic.

---

## 21. Padrão para API Key

Em APIs internas, é comum proteger endpoints com API key.

Exemplo em `api/deps.py`:

```python
from fastapi import Header, HTTPException, status
from core.config import settings


async def validate_api_key(api_key: str = Header(None, alias="x-api-key")):
    if not api_key or api_key != settings.API_KEY:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="API key inválida ou ausente.",
        )
    return api_key
```

Uso no endpoint:

```python
@router.post("/webhook")
async def webhook(
    payload: dict,
    api_key: str = Depends(validate_api_key),
):
    return {"received": True}
```

---

## 22. Padrão para imports

Evite imports circulares.

Padrão recomendado:

```python
# endpoints
from api import deps
from crud.crud_service_orders import crud_service_order
from schemas.service_order_schema import ServiceOrderCreate

# crud específico
from crud.baseAsync import CRUDBase
from models.service_order_model import ServiceOrder
from schemas.service_order_schema import ServiceOrderCreate, ServiceOrderUpdate

# models
from db.base_class import Base
```

Evite:

```python
from main import app
```

ou importar endpoint dentro de CRUD/model.

---

## 23. Rodando com Docker

Exemplo simples de `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Exemplo de `docker-compose.yml`:

```yaml
services:
  api:
    build: .
    container_name: template-fastapi
    ports:
      - "8000:8000"
    env_file:
      - .env
    restart: unless-stopped
```

Subir:

```bash
docker compose up -d --build
```

Ver logs:

```bash
docker logs -f template-fastapi
```

---

## 24. Deploy em IIS com HttpPlatform

Para ambiente Windows Server com IIS, um exemplo de `web.config`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="httpPlatformHandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" />
    </handlers>
    <httpPlatform
      processPath="C:\inetpub\wwwroot\TemplateFastAPI\venv\Scripts\python.exe"
      arguments="-m uvicorn main:app --host 0.0.0.0 --port %HTTP_PLATFORM_PORT%"
      stdoutLogEnabled="true"
      stdoutLogFile="C:\inetpub\wwwroot\TemplateFastAPI\logs\python-stdout.log"
      startupTimeLimit="60"
      processesPerApplication="1">
      <environmentVariables>
        <environmentVariable name="PYTHONPATH" value="C:\inetpub\wwwroot\TemplateFastAPI" />
      </environmentVariables>
    </httpPlatform>
  </system.webServer>
</configuration>
```

Pontos importantes:

- instalar dependências dentro do `venv` do projeto;
- garantir permissão de escrita na pasta `logs`;
- usar `%HTTP_PLATFORM_PORT%`;
- configurar `PYTHONPATH` apontando para a raiz do projeto;
- testar localmente antes com `uvicorn main:app`.

---

## 25. Boas práticas do projeto

### Organização

- Um domínio por arquivo de endpoint.
- Um CRUD específico por model.
- Um schema por domínio.
- Um model por tabela principal.
- Regra de negócio grande deve ir para `core` ou `services`.

### Banco

- Sempre usar `get_db` como dependência.
- Não abrir sessão manualmente dentro do endpoint.
- Evitar commit espalhado fora do CRUD.
- Usar transações quando houver várias operações dependentes.

### Schemas

- Entrada e saída devem ser separadas.
- Nunca retornar diretamente campos sensíveis.
- Usar `exclude_unset=True` em updates.

### Endpoints

- Manter endpoints pequenos.
- Validar existência antes de update/delete.
- Usar status code correto.
- Mensagens de erro devem ser claras.

### Integrações

- Centralizar chamadas externas.
- Logar payload e resposta em erros.
- Usar timeout.
- Tratar falha externa sem quebrar o fluxo interno quando necessário.

---

## 26. Checklist para criar uma nova funcionalidade

Use este checklist sempre que criar um novo módulo:

```text
[ ] Criar model em models/
[ ] Importar model em db/base.py
[ ] Criar schemas em schemas/
[ ] Criar CRUD específico em crud/
[ ] Criar endpoint em api/api_v1/endpoints/
[ ] Registrar endpoint em api/api_v1/api.py
[ ] Criar migration, se usar Alembic
[ ] Testar criação via /docs
[ ] Testar listagem via /docs
[ ] Testar update e delete
[ ] Validar filtros necessários
[ ] Validar response_model
[ ] Validar mensagens de erro
[ ] Validar autenticação/API key, se necessário
```

---

## 27. Exemplo completo de consulta com filtros avançados

```python
filters = [
    {"field": "is_active", "operator": "is_true", "value": True},
    {
        "logic": "or",
        "conditions": [
            {"field": "status", "operator": "==", "value": "PENDENTE"},
            {"field": "status", "operator": "==", "value": "EM_ROTA"},
        ],
    },
    {
        "logic": "not",
        "conditions": [
            {"field": "status", "operator": "==", "value": "CANCELADO"},
        ],
    },
]

items = await crud_service_order.get_multi_dynamic_filters(
    db,
    filters=filters,
    order_by="created_at",
    order_direction="desc",
    force_order_id=True,
    offset=0,
    limit=100,
)
```

---

## 28. Exemplo de endpoint com filtros por query params

```python
@router.get("/buscar", response_model=list[ServiceOrderResponse])
async def buscar_service_orders(
    db: AsyncSession = Depends(deps.get_db),
    status: str | None = None,
    cliente: str | None = None,
    limit: int = Query(100, ge=1, le=500),
    offset: int = Query(0, ge=0),
):
    filters = []

    if status:
        filters.append({
            "field": "status",
            "operator": "==",
            "value": status,
        })

    if cliente:
        filters.append({
            "field": "client.name",
            "operator": "ilike",
            "value": cliente,
        })

    return await crud_service_order.get_multi_filters(
        db,
        filters=filters,
        order_by="id",
        order_desc=True,
        limit=limit,
        offset=offset,
    )
```

---

## 29. Recomendações para evolução do template

Para projetos maiores, recomenda-se adicionar:

```text
alembic/
services/
tests/
logging_config.py
middlewares/
exceptions/
```

Estrutura sugerida:

```text
TemplateFastAPI/
├── services/
│   └── service_order_service.py
├── exceptions/
│   └── handlers.py
├── middlewares/
│   └── request_logger.py
├── tests/
│   └── test_service_orders.py
└── alembic/
```

---

## 30. Comandos úteis

Rodar projeto:

```bash
uvicorn main:app --reload
```

Rodar em porta específica:

```bash
uvicorn main:app --reload --port 8001
```

Instalar dependências:

```bash
pip install -r requirements.txt
```

Gerar requirements atualizado:

```bash
pip freeze > requirements.txt
```

Testar importação principal:

```bash
python -c "from main import app; print(app.title)"
```

Ver rotas disponíveis:

```bash
python -c "from main import app; print([route.path for route in app.routes])"
```

---

## 31. Problemas comuns

### Erro: `ModuleNotFoundError`

Verifique se está rodando o comando na raiz do projeto:

```bash
uvicorn main:app --reload
```

E se o ambiente virtual está ativo.

---

### Erro: `No module named asyncpg`

Instale o driver PostgreSQL async:

```bash
pip install asyncpg
```

---

### Erro de conexão com banco

Verifique:

- host;
- porta;
- usuário;
- senha;
- nome do banco;
- driver da URI;
- firewall;
- se o banco aceita conexão externa.

---

### Erro em update com Pydantic v2

Use:

```python
payload.model_dump(exclude_unset=True)
```

No CRUD definitivo, essa compatibilidade já deve estar tratada.

---

### Erro no `create_multi`

Garanta que o commit está sendo executado com parênteses:

```python
await db.commit()
```

Não use:

```python
await db.commit
```

---

## 32. Padrão recomendado de desenvolvimento

Fluxo ideal:

```text
1. Criar model
2. Criar schema
3. Criar CRUD
4. Criar endpoint
5. Registrar rota
6. Testar via Swagger
7. Criar regra de negócio em core/service, se necessário
8. Criar migrations
9. Testar no ambiente local
10. Subir para homologação/produção
```

---

## 33. Resumo

Este template é uma base para APIs FastAPI com padrão limpo e reaproveitável.

A ideia principal é manter cada camada com sua responsabilidade:

```text
Endpoint  -> recebe e responde HTTP
Schema    -> valida entrada e saída
CRUD      -> acessa banco
Core      -> regra de negócio
Model     -> representa tabela
DB        -> sessão e base SQLAlchemy
```

Seguindo esse padrão, o projeto fica mais fácil de manter, evoluir, testar e reutilizar em novas APIs.
