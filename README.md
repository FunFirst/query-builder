# query-builder
Laravel package used to work with eloquent query. 

1. Search
2. Pagination
3. Scopes
4. Filter
5. Includes
6. Fields
7. Total Records
8. Sort


Because QueryBuilder extends \Illuminate\Database\Eloquent\Builder, you can use query builder as you are used to in Laravel application.
Optimalized to work with FromRequests.

## Initialize query
#### Via Model name:
```php
$query = new QueryBuilder(\App\Contact::class);
```

#### With existing query:

```php
$query = \App\Contact::query();
$query = new QueryBuilder($query);
```

#### With model & request
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
}

```

## Use search
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->applySearch();
}
```

Query builder automatically fetche's from request url params and tries to find with where LIKE query in all searchable fields and relationship fields. 

By default will QueryBuilder tries to search inside Fillable fields of Model
```php
protected $fillable = [
    'name',
    'type',
    'source',
    'created_by',
    'created_at',
    'updated_at',
];
```

or you can specify searchable fields on model with ```$searchableFields``` property:
```php
protected $searchableFields = [
  'name',
  'type',
];
```

To tell QueryBuilder to search inside relationships, model has to contain ```$relationships``` property.
```php
public $relationships = [
  'tags' => [
    'class' => \App\API\Tags\Tag::class,
    'type' => 'belongsToMany',
  ],
  'notes' => [
    'class' => \App\API\Notes\Note::class,
    'type' => 'belongsToMany',
  ],
];
```
QueryBuilder will than try to search inside relationship's fields. Relationship fields are defined on relationship model with same way as before -> defautly with ```$fillable``` property or can be customize with ```$searchableFields``` property

If you need to specify custom search query for some searchable fields, you can define custom query scopes that will be applied on QueryBuilder. Custom query scopes are defined by adding methods starting with 'scope' folowed by searchable field name. That method has to recieve two parameters $query (current QueryBuilder query) and $value (Searched value). 

Define custom searchable field:
```php
protected $searchableFields = [
  'name' // Name is not column inside models table.
];
```

Define custom scope:
```php
public function scopeName($query, $value): void
{
    $explodedName = explode(' ', $value);
    $firstName = $explodedName[0];
    $lastName = $explodedName[1];
    
    $query->where('first_name', $firstName)
      ->where('last_name', $lastName);
}
```

## Use scopes
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->applyScopes();
}
```

Scopes can be used to apply custom model scopes on query. QueryBuilder will automatically fetch value of scopes url param. Then will QueryBuilder try to apply them on query. Scopes from url param has to be defined on model. Custom query scopes are defined by adding methods starting with 'scope' folowed by searchable field name. That method has to recieve two parameters $query (current QueryBuilder query). If scope method is not found on model, QueryBuilder will skip that scope.

#### For example passing ```?scopes[]=woman``` via request:
Define custom scope:
```php
public function scopeWoman($query): void
{
    $query->where('gender', 'WOMAN')
}
```
After that all contacts left in query will have gender === 'WOMAN'.

## Use pagination
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->applyPagination();
}
```

QueryBuilder will defautly take first 25 records from query. But you can per page and page inside request url params that will be automatically fetched by QueryBuilder. 

For example setting per page = 10 and page number on 2 (Will return 11 - 20 record from query):
```?page[size]=10&page[number]=2```

## Use total records
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->getTotalRecords();
}
```

QueryBuilder will return number of records of current query. 
*Be careful to use getTotalRecords() before using applyPagination, or you will get only per page number of records not a total query records number*

## Use fields
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->applyAllowedFields();
}
```

Query builder can select just desired fields from query. You can specify allowed fields via request url param fields[]. Fields values has to exist in Model table, if not desired field value is skipped. As addition to that you can specify relationship fields, which means relationship will be fetched and only desired column will be returned, for example: ```relationship.name```. *Relationship fields does not need to be specify in includes, query will automaticlly resolve relationship include.* 

If url param fields[] is not passed via URL, QueryBuilder will try to find defaults on model. Defaults are defined in public model property with name ```php public $returnFields = []```. In defaults can be defined model fields and even relationship fields with same manner as in URL param. 

If not even defaults are defined all fields inside $fillable property of model will be returned.
