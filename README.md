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
#### via Model name:
```php
$query = new QueryBuilder(\App\Contact::class);
```

#### with existing query:

```php
$query = \App\Contact::query();
$query = new QueryBuilder($query);
```

### with model & request
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
}

```
