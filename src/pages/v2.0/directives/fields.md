---
path: "/docs/2.0/directives/fields"
date: "2018-03-31T20:37:30.954Z"
title: "Field Directives"
---

## Field Directives

Field directives can be attached to any field of an Object Type.

* [**@auth** directive](#auth) provides the currently authenticated user.
* [**@belongsTo** directive](#belongsTo) eager loads the eloquent relationship.
* [**@can** directive](#can) check authorizations against a model
* [**@complexity** directive](#complexity) calculates complexity 
* [**@create** directive](#create) its used to create new records.
* [**@delete** directive](#delete) used to delete.
* [**@event** directive](#event) allows you to fire a Laravel event.
* [**@field** directive](#field) points to a class and method used to resolve a field.
* [**@first** directive](#first) finds a single model according to given filters, similar to the `@paginate` directive
* [**@find** directive](#find) the same as the `@first` directive with the difference that if there are more then one result an error will be returned
* [**@globalId** directive](#globalId) converts a globalId field back to it's original id.
* [**@hasMany** directive](#hasMany) used to create a connection between two eloquent models.
* [**@inject** directive](#inject) can be used to inject a value from the context.
* [**@method** directive](#method) used to call a method on the model to resolve a field
* [**@middleware** directive](#middleware) TODO
* [**@paginate** directive](#paginate) similar to `@hasMany` in that it return a `paginator` or `relay` connection.
* [**@rename** directive](#rename) used to rename on argument on the server side.
* [**@update** directive](#update) TODO

<br />

<a name="auth"></a>

### @auth

The `@auth` directive provides the currently authenticated user. This comes in handy on the root query. For example:

```graphql
type Query {
  me: User @auth
}
```

Sending the following query will return the authenticated user, or if the request is not authenticated null will be returned.

```graphql
query Me {
  me {
    name
    email
  }
}
```

<a name="belongsTo"></a>

### @belongsTo

The `@belongsTo` directive eager loads the eloquent relationship so it should only be used on a type that resolved to an Eloquent model. The `@belongsTo` directive accepts a `relation` argument if your relationship has a different name than the field. For example, let's assume we have the following models:

```php
class User extends Model {
  // ...
}

class Post extends Model {
  // ...
  public function author()
  {
      return $this->belongsTo(User::class);
  }
}
```

We can express our GraphQL types like so:

```graphql
type User {
  # ....
}

type Post {
  # ...
  author @belongsTo

  # or we could use the `relation` argument and use
  # a different field name
  user @belongsTo(relation: "author")
}
```

<a name="can"></a>

### @can

The `@can` directive can be used to check authorization against a model. It requires a `model` argument which should be the namespace of the model, and a `if` argument which can be a string or an array. If an array is provided then **all** policies have to be granted.

Some examples:

```graphql
type Mutation {
  updateSubscriptionBilling(
    account: Int
  ) Subscription 
    @can(if: "updateBilling", model: "App\\Subscription")
}
```

```graphql
type User {
  jobs: [Job] @hasMany @can(if: ["view", "deep-look"], model: "App\\Job")
}
```

<a name="complexity"></a>

### @complexity

The `@complexity` directive can be attached to fields to calculate complexity (default calculation provided for relationships). It requires a `resolver` argument which should be the namespace of the model, and a `if` argument which can be a string or an array. If an array is provided then **all** policies have to be granted.

Some examples:

```graphql
type User {
    # default calculation
    posts: [Post!]! @hasMany @complexity

    # custom calculation
    posts: [Post!]! @hasMany @complexity(resolver: "App\\Http\\GraphQL\\Complexity@userPosts")
}
```

<a name="create"></a>

### @create

The `@create` directive can be used to create a new model. It requires a `model` argument which should be the namespace of the model you want to create. In the following example, the `createPost` mutation will autofill a new `Post` eloquent model with the `title` and `content` arguments.

Most of the time, you'll likely need to grab something like the authenticated user's `id` and inject it into the arguments (since we don't want the client to decide what user `id` to fill). In that case, use `@create` with the `@inject` directive (listed below).

```graphql
type Mutation {
  createPost(title: String!, content: String!): Post @create(model: "App\\Post")
}
```

<a name="delete"></a>

### @delete

The `@delete` directive can be used to delete a model with a given id field. The field must be an `ID` type.

```graphql
type Mutation {
  deletePost(id: ID!): Post @delete
  # If you use global ids, you can set the `globalId` argument to true like so:
  deletePost(id: ID!): Post @delete(globalId: true)
}
```

<a name="event"></a>

### @event

The `@event` directive allows you to fire an event after a mutation has taken place. It requires the `fire` argument that should be the class name of the event you want to fire.

```graphql
type Mutation {
  createPost(title: String!, content: String!): Post
    @event(fire: "App\\Events\\PostCreated")
}
```

<a name="field"></a>

### @field

The `@field` directive points to a class and method used to resolve a field. This can be use to resolve a query or mutation field, or you could also use it to manipulate the output of a field on a registered type (i.e., format a Carbon instance)

```graphql
type User {
  created_at: String!
    @field(resolver: "App\\Http\\GraphQL\\Types\\UserType@created_at")
}

type Mutation {
  createPost(title: String!): Post
    @field(resolver: "App\\Http\\GraphQL\\Mutations\\PostMutator@create")
  updatePost(title: String!): Post
    @field(resolver: "App\\Http\\GraphQL\\Mutations\\PostMutator@update")
  deletePost(title: String!): Post
    @field(resolver: "App\\Http\\GraphQL\\Mutations\\PostMutator@delete")
}
```

<a name="first"></a>

### @first

The `@field` directive finds a single model according to given filters, similar to the `@paginate` directive.

Searches for a user using either a given id or email:

```graphql
type Query {
    user(id: ID @eq, email: String @eq): User @find(model: "App\\User")
}
```

Or with scopes:

```graphql
type Query {
    user(name: String @eq, company: String!): User @find(model: "App\\User" scopes: [companyName])
}
```

... where your query could look like this:

```graphql
{
    user(company: "MyCompany") {
        id
        name
    }
}
```

.. and your scope code look like this:

```php
    public function scopeCompanyName($query, $args)
    {
        return $query->whereHas('company', function($q) use ($args){
            $q->where('name', $args['company']);
        });
    }
```

<a name="find"></a>

### @find

The `@field` directive works the same as the `@first` directive with the only difference that if there are more then one result an error will be returned.


<a name="globalId"></a>

### @globalId

The `@globalId` directive converts a globalId field back to it's original id. This can be useful for mutations that accept a global id, but you need the original id to query the database.

```graphql
type Mutation {
  # In your resolver the $args['id'] will be the original id of the post
  updatePost(id: ID! @globalId, title: String): Post
}
```

<a name="hasMany"></a>

### @hasMany

The `@hasMany` directive can be used to create a connection between two eloquent models. It also accepts a `relation` argument if the name of the relationship is different than the field. It also accepts a `type` argument that can be set to `paginator` or `relay` to convert the relationship to a pagination field.

```graphql
type User {
  posts: [Post!]! @hasMany
  # for a Paginator field
  posts: [Post!]! @hasMany(type: "paginator")
  # for a Relay connection field
  posts: [Post!]! @hasMany(type: "relay")
  # if the relationship on your user model was named `articles`
  posts: [Post!]! @hasMany(relation: "articles")
}
```

<a name="inject"></a>

### @inject

The `@inject` directive can be used to inject a value from the context object into the arguments. This is really useful with the `@create` directive that rely on the authenticated user's `id` that you don't want the client to fill in themselves.

```graphql
type Mutation {
  createPost(title: String!, content: String!): Post
    @create(model: "App\\Post")
    @inject(context: "user.id", name: "user_id")
}
```

<a name="paginate"></a>

### @paginate

The `@paginate` is similar to the `@hasMany` directive in that it return a `paginator` or `relay` connection. However, instead of using it on a `Type` you would instead use it on one of the `Query`'s fields. This directive requires a `model` argument.

```graphql
type Query {
  posts: [Post!]! @paginate(model: "App\\Post")
  # for a Paginator field
  posts: [Post!]! @paginate(model: "App\\Post", type: "paginator")
  # for a Relay connection field
  posts: [Post!]! @paginate(model: "App\\Post", type: "relay")
}
```

<a name="rename"></a>

### @rename

The `@rename` directive can be used to rename on argument on the server side. This comes in handy if you want to use snake_case on the server side but camelCase on the client side. It requires the `attribute` argument

```graphql
type User {
  createdAt: String! @rename(attribute: "created_at")
}
```

<a name="method"></a>

### @method

The `@method` directive can be used to call a method on the target model. This comes in handy if the data is not accessable via get attribute (e.x. getMyDataAttribute -> myData) but via a method like $myModel->myData(). It requires the `name` argument

```graphql
type User {
  mySpecialData: String! @method(name: "findMySpecialData")
}
```
