<p align="center">
  <img src="assets/brand/ants-infrastructure-mark-clean-six-legs-v02.png" alt="Antonic ant infrastructure logo" width="220">
</p>

# antonic

MongoDB persistence for Pydantic v2 AntDocs.

Antonic `0.1.1b1` is a beta pre-release. The public package is named `antonic`, while the
core model and metadata classes keep the Ant vocabulary: `AntDoc`,
`AntConnector`, `AntIndex`, `ant_collection`, and `ant_indexes`.

## Install

```bash
pip install antonic
```

To install the current beta pre-release explicitly:

```bash
pip install antonic==0.1.1b1
```

Pre-release versions are not selected by plain `pip install antonic` when a
stable release is available. To allow pip to consider beta releases:

```bash
pip install --pre antonic
```

AntDocs describe data and local persistence metadata. `AntConnector` owns the
MongoDB connection and document behavior:

```python
from typing import ClassVar, Sequence

from antonic import ASCENDING, DESCENDING, AntConnector, AntDoc, AntIndex


class User(AntDoc):
    email: str
    name: str
    status: str = "active"

    ant_collection: ClassVar[str] = "users"
    ant_indexes: ClassVar[Sequence[AntIndex]] = (
        AntIndex([("email", ASCENDING)], unique=True, name="uniq_user_email"),
        AntIndex([("status", ASCENDING), ("created_at", DESCENDING)]),
    )


class Project(AntDoc):
    owner_id: str
    slug: str
    title: str

    # Collection defaults to "projects".
    ant_indexes: ClassVar[Sequence[AntIndex]] = (
        AntIndex([("owner_id", ASCENDING), ("slug", ASCENDING)], unique=True),
    )


async def main() -> None:
    async with AntConnector("mongodb://localhost:27017/app") as db:
        await db.ensure_indexes(User, Project)

        user = await db.save(User(email="a@b.com", name="Alice"))
        found = await db.get(User, user.id)
        active = await db.find(User, status="active", sort=[("created_at", -1)], limit=25)

        await db.patch(User, {"name": "Alice Cooper"}, id=user.id)
        await db.delete(user)

        raw_users = db.collection(User)
        await raw_users.find_one({"email": "a@b.com"})
```

If no connection string is passed, `AntConnector()` reads `MONGODB_URI`. The
database name can come from the URI path, from `database=`, or from
`MONGODB_DATABASE` when the URI has no database path.

`filter={...}` accepts Ant's Mongo-like query DSL. Extra keyword arguments are
equality filters, so use `filter={"limit": 10}` for document fields that collide
with connector options.

Plain `AntDoc` ids default to MongoDB `ObjectId` values. You can pass either
an `ObjectId` or its string form to connector methods that filter by `id`.

Antonic requires Python 3.12 or newer.

## License

Apache-2.0. See [LICENSE](LICENSE).
