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
QueryBuilder will than try to search inside relationship's fields. Relationship fields are defined on relationship model with same was as before -> defautly with ```$fillable``` property or can be customize with ```$searchableFields``` property

```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->applySearch();
}

```
