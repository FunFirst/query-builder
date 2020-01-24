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
QueryBuilder will than try to search inside relationship's fields. Relationship fields are defined on relationship model with same way as before -> defautly with ```$fillable``` property or can be customize with ```$searchableFields``` property

```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->applySearch();
}
```

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
