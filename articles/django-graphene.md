---
title: "DjangoでGraphQL"
emoji: "🍌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Django", "GraphQL"]
published: true
---

## 概要

DjangoでGraphqlが使えないか調査していたら[graphene-django](https://github.com/graphql-python/graphene-django/)を見つけたので使ってみました


## ゴール

Djangoにgraphene-djangoをインストールしてシンプルなクエリを発行してみる


## 成果物

[サンプル](https://github.com/tabio/django-graphene-test)をGitHubにおいておきました

デバッグ用のGUIツールが用意されているので、Docker起動後テストデータを投入しサクッと確認ができました
![](/images/django-graphene/graphql_results.png)

## ポイント

以下のようにGraphQLのスキーマをコードベースで管理できる
Django側でGraphQLのIFを良しなに吸収して貰えるのもお手軽で良い

```py
import graphene
from graphene_django import DjangoObjectType

from .models import Staff, Store


class StoreType(DjangoObjectType):
    class Meta:
        model = Store


class StaffType(DjangoObjectType):
    class Meta:
        model = Staff


class Query(graphene.ObjectType):
    stores = graphene.List(StoreType)
    staffs = graphene.List(StaffType)

    def resolve_stores(self, info, **kwargs):
        return Store.objects.all()

    def resolve_staffs(self, info, **kwargs):
        return Staff.objects.all()


schema = graphene.Schema(query=Query)
```

### その他

django以外で使いたい場合は、[SQLAlchemy版](https://github.com/graphql-python/graphene-sqlalchemy/)や[Google App Engine版](https://github.com/graphql-python/graphene-gae/)もあるので用途に応じて選択も可能でした
