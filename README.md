# query-builder
Laravel package used to work with eloquent query. 

1. Search
2. Scopes
3. Filter
4. Includes
5. Fields
6. Sort
7. Total Records
8. Pagination



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

## Use includes
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->applyIncludes();
}
```

You can specify relationships that should be fetched with model records. That can be done via includes[] URL parameter. Inside includes should be specified desired relationships, in following manner: ```inludes[]=relationship```. To fetch desired relationship, Model has to have ```public $relationship = [] ``` property. This proprety has to be array and has to specify known relationships of Model. Example of ```$relationship``` property:

```php
public $relationships = [
  'nameOfRelationshipMethod' => [
    'class' => 'classOfRelatedModel',
    'type' => 'typeOfRelationship',
  ]
];
```

```php
public $relationships = [
  'tags' => [
    'class' => \App\Tag::class,
    'type' => 'belongsToMany',
  ],
  'notes' => [
    'class' => \App\Note::class,
    'type' => 'belongsToMany',
  ],
  'files' => [
    'class' => \App\File::class,
    'type' => 'hasMany',
  ],
];
```

After that QueryBuilder will automatically include all relationships.

As addition to includes, properties that are defined in allowed fields (more information in Use fields) are included automatically.

If includes are not defined in URL param. QueryBuilder will try to find default includes in Model. Default includes are defined as public property ```protected $defaultIncludes = []```. Values inside that array should be defined in same manner as in URL param described above. Same as with URL params all values has to be defined inside relationship property on model, or relationship will be skipped.

## Use filters
```php
public function index (\Illuminate\Http\Request $request) 
{
  $query = new QueryBuilder(\App\Contact::class, $request);
  $query->applyFilters();
}
```

Filters can be applied to QueryBuilder. Mainly will QueryBuilder try to fetch filters from request URL param ```filter```. Filter URL param is defined in following way:

```php
filter[type]=OR&filter[values][0][type]=AND&filter[values][0][values][0][field]=first_name&filter[values][0][values][0][comparison]=IS&filter[values][0][values][0][value]=John+Doe&filter[values][0][values][1][field]=email&filter[values][0][values][1][comparison]=HAS_ANY_VALUE&scopes[]=person
```

parsed to array:

```php
$filter = [
  'type' => 'OR',
  'values' => [
     'type' => 'AND', 
     'values' => [
          0 => [
              'field' => 'first_name',
              'comparison' => 'IS',
              'value' => 'John',
          ],
          1 => [
              'field' => 'email',
              'comparison' => 'HAS_ANY_VALUE',
          ],
      ],
  ],
];
```

As field you can specify model fields, relationship fields and custom filters from model.

If you want to specify your own custom filtering logic, you need to add folowing to your model:
  1. Add method ```public function shouldApplyCustomFilter($fieldName) {}```. This method will recieve $fieldName param that contains name of field that is desired to filter. In that function you can specify your own logic to determinate, if custom filter should be called. Example:

```php
public function shouldApplyCustomFilter($fieldName): bool
{
    // If Model has method with customFieldNameFitler
    if (method_exists($this, 'custom' . $fieldName . 'Filter')) {
        return true;
    }
    
    // If $fieldName contains 'custom_field.'
    if (substr($fieldName, 0, strlen('custom_field.')) === 'custom_field.') {
        return true;
    }
    
    // If $fieldName is defined inside property
    if (array_key_exists($fieldName, $this->systemProperties)) {
        return true;
    }
}
```
  2. If ```shouldApplyCustomFilter()``` returns ```true```, instead of QueryBuilder filter is called custom method from Model. QueryBuilder need sto know what is name of that custom method, so it is needed to define ```public function getCustomFilteringFunction($fieldName):string {}``` that returns method name. That will be called. Example:
  
```php
public function shouldApplyCustomFilter($fieldName): bool
{
    if (method_exists($this, 'custom' . $fieldName . 'Filter')) {
        return 'custom' . $fieldName . 'Filter';
    }

    if (substr($fieldName, 0, strlen('custom_field.')) === 'custom_field.') {
        return 'filterInCustomFieldProperties';
    }

    if (array_key_exists($fieldName, $this->systemProperties)) {
        return 'filterInSystemProperties';
    }
}
```
  3. After that is called that custom function. Function will recieve $query, $whereClauseType and $filter (can be used to apply filter logic on modified value and so...). Examples:
```php
public function filterInCustomFieldProperties($fieldName): bool
{
  $name = explode('.', $filter->getField())[1];
        $query->{$whereClauseType . 'Has'}('properties', function ($q) use ($name, $filter) {
            $q->where('type', 'CUSTOM')
                ->where('name', $name);

            $filterTypeClass = '\\App\\Utils\\QueryBuilder\\FilterTypes\\' . ucfirst(\Illuminate\Support\Str::camel(strtolower($filter->getComparison())));
            $filterType = new $filterTypeClass('value', $filter->getValue());
            $filterType($q, 'where');
        });
}
```
```php
/**
 *  Custom filter for name
 *
 *  @param
 *  @return void
 */
public function customNameFilter($query, $whereClauseType, $filter): void
{
    list($firstNamePart, $secondNamePart) = explode(' ', $filter->getValue());
    $query->{$whereClauseType}(function ($q) use ($firstNamePart, $secondNamePart, $filter) {
        $q->whereHas('properties', function ($propertiesQuery) use ($firstNamePart, $filter) {
            $propertiesQuery->where('type', 'SYSTEM')
                ->where('name', 'first_name');

            $filterTypeClass = '\\App\\Utils\\QueryBuilder\\FilterTypes\\' . ucfirst(\Illuminate\Support\Str::camel(strtolower($filter->getComparison())));
            $filterType = new $filterTypeClass('value', $firstNamePart);
            $filterType($propertiesQuery, 'where');
        })->whereHas('properties', function ($propertiesQuery) use ($secondNamePart, $filter) {
            $propertiesQuery->where('type', 'SYSTEM')
                ->where('name', 'last_name');

            $filterTypeClass = '\\App\\Utils\\QueryBuilder\\FilterTypes\\' . ucfirst(\Illuminate\Support\Str::camel(strtolower($filter->getComparison())));
            $filterType = new $filterTypeClass('value', $secondNamePart);
            $filterType($propertiesQuery, 'where');
        });
    });
}
```
```php
public function filterInSystemProperties($query, $whereClauseType, $filter)
{
    $query->{$whereClauseType . 'Has'}('properties', function ($q) use ($filter) {
        $q->where('name', $filter->getField());

        $filterTypeClass = '\\App\\Utils\\QueryBuilder\\FilterTypes\\' . ucfirst(\Illuminate\Support\Str::camel(strtolower($filter->getComparison())));
        $filterType = new $filterTypeClass('value', $filter->getValue());
        $filterType($q, 'where');
    });
}
```
  
